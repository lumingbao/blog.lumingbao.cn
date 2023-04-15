+++
title = "Extjs 添加拦截器，对后台异常做统一的前台处理"
date = 2015-10-30 11:41:00
url = "archives/14181"
tags = ["Extjs"]
categories = ["工作记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner1.jpg'
+++

### 1.创建拦截器基类

```javascript
/**
 * 抽象类
 * 任何要实现拦截器都需要继承自这个类，并实现其中的interceptor方法，并添加至拦截器池中，就可以实现拦截功能
 */
Ext.define('system.interceptor.BaseInterceptor',{
	
	alternateClassName:['system.BaseInterceptor'],
	
	statics:{
		BEFORE:'before', //在请求前拦截
		AFTER:'after',   //在请求过后拦截
		AROUND:'around'  //在请求前后都拦截
	},
	
	mode:'after',
	
	isInterceptor:true,
	
	//不包含的URL
	excludes:[],
	
	//仅包含URL，优先级大于excludes
	includes:[],
	
	/**
	 * 拦截方法，执行拦截及验证过程
	 * @param {Object} options The options config object passed to the request method.
	 * @param {Object} response The XHR object containing the response data. See The XMLHttpRequest Object for details.
	 * @return {Boolean}
	 */
	interceptor:Ext.emptyFn(),
	
	constructor:function(interceptorFn){
		var me = this;
		
		if(Ext.isObject(interceptorFn)){
			//me.interceptor = interceptorFn;
			Ext.apply(me,interceptorFn);
		}else if(Ext.isFunction(interceptorFn)){
			me.interceptor = interceptorFn;
		}
	},
	
	getId:function(){
		return this.id;
	},
	/**
	 * 处理拦截过程
	 * @param {} options
	 */
	handler:function(options,response){
		if (this.validationUrl(options.url)) {
			return this.interceptor(options,response);
		}
	},
	/**
	 * 验证URL是否需要拦截
	 * @param {String} url 需要验证的地址
	 */
	validationUrl:function(url){
		var me = this,
			intercept = false;
		
		//如果配置了include就仅验证包含的URL
		//如果配置了excludes就仅不包含excludes中的URL
		//如果都没有配置就拦截所有URL
		if(me.includes.length == 0 && me.excludes.length == 0){
			intercept = true;
		}else if(me.includes.length>0){
			//满足条件说明需要拦截
			Ext.Array.each(this.includes,function(reg){
				//url.match(reg)
				var reg = new RegExp(reg);
				if(reg.test(url))intercept = true;
				return false;
			});
		}else{
			intercept = true;
			Ext.Array.each(this.excludes,function(reg){
				var reg = new RegExp(reg);
				if(reg.test(url))intercept = false;
				return false;
			});
		}
		
		return intercept;
	}
});
```

### 2.创建异常拦截器。继承前面创建的拦截器基类

```javascript
/**
 * 异常拦截器
 * 处理后台返回的有异常信息的请求(包括代码异常，业务异常，session超时)
 */
Ext.define('system.interceptor.ExceptionInterceptor',{
	extend:'system.interceptor.BaseInterceptor',
	
	interceptor:function(options,response){
		
		var resultData = Ext.decode(response.responseText);
		
		/**
		 * 处理代码异常和业务异常
		 */
		if(resultData.isException){
			Ext.MessageBox.alert(resultData.exName,resultData.message);
			return false;
		}
		
		/**
		 * 处理Session超时
		 */
		if(resultData.isSessionOut){
			Ext.MessageBox.alert('Session超时','Session超时，请重新登录！',function(){
				window.location = Ext.CONTEXT+'login';
			});
			return false;
		}
		
		return true;
	}
	
});
```
### 3.重写Ext.Ajax

```javascript
    /**
	 * 重写Ext.Ajax的意思在于：添加拦截器，对后台异常、session超时信息在前台进行统一的处理。
	 */
	Ext.define('Ext.Ajax',{
		extend: 'Ext.data.Connection',
		
		singleton: true,
	    
		autoAbort: false,
		//是否允许在请求前拦截
		enableBeforeInterceptor:false,
	    
		interceptors:Ext.create('Ext.util.MixedCollection'),
		
		//执行拦截操作
		invokeInterceptor:function(options,response,mode){
			var me = this;
			
			this.interceptors.each(function(interceptor){
				//判断拦截器类型
				if(interceptor.mode == mode || 
					interceptor.mode == Xzr.AbstractInterceptor.AROUND ){
					//执行拦截操作，如果没有通过拦截器则返回false
					if(interceptor.handler(options,response) === false){
						return false;
					};
				}
			});
			
			return true;

		},
		//通过listener实现对Ajax访问的拦截
		listeners :{
			//拦截器处理，如果没有通过拦截器，则取消请求
			beforerequest:function( conn, options, eOpts ){
				if(this.enableBeforeInterceptor){
					return this.invokeInterceptor(options,null,'before');
				}
				return true;
			},
			//请求完成后对数据验证，无法中断后续的操作，具体请研究ExtJs源码。
			requestcomplete:function(conn, response, options, eOpts){
				return this.invokeInterceptor(options,response,'after');
			}
		},
		
		/**
		 * 添加拦截器
		 * @param {String} interceptorId
		 * @param {Xzr.web.AbstractInterceptor} interceptor
		 */
		addInterceptor:function(interceptor){
			if(!interceptor)return;
			
			if(Ext.isString(interceptor)){
				interceptor = Ext.create(interceptor);
			}
			
			this.interceptors.add(interceptor.getId(),interceptor);
		}
	});
```

### 4.将异常拦截器添加到拦截池中

```javascript
Ext.onReady(function() {
	Ext.Ajax.addInterceptor('system.interceptor.ExceptionInterceptor');
});
```
### 5.后台统一捕获代码异常和业务异常

```java
package com.fsd.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.log4j.Logger;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import com.fsd.bean.ExceptionInfo;
import com.fsd.common.BusinessException;

public class BaseController {
	
	private static final Logger log = Logger.getLogger(BaseController.class);
	
	/** 基于@ExceptionHandler异常处理 */
	@ExceptionHandler
	public @ResponseBody Object resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
		ExceptionInfo exinfo = new ExceptionInfo();
		if(ex instanceof BusinessException) {
			exinfo.setIsException(true);
			exinfo.setExTtype("bizexception");
			exinfo.setExName("业务异常");
			exinfo.setExDetails(ex.toString());
			exinfo.setMessage(ex.getMessage());
		}else {
			exinfo.setIsException(true);
			exinfo.setExTtype("exception");
			exinfo.setExName("代码异常");
			exinfo.setExDetails(ex.toString());
			exinfo.setMessage(ex.getMessage());
			ex.printStackTrace();
		}
		log.error(ex);
		return exinfo;
	}
}
```


