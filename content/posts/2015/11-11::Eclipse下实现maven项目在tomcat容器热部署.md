+++
title = "Eclipse下实现maven项目在tomcat容器热部署"
date = 2015-11-11 17:41:00
url = "archives/70419"
tags = ["Maven","Eclipse","Maven"]
categories = ["工作记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner4.jpg'
+++

## Eclipse下实现maven项目在tomcat容器热部署

#### 1.修改tomcat目录下:conf/tomcat-users.xml 文件，在<tomcat-users>节点中加入如下内容：

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="admin" password="password" roles="manager-gui, manager-script"/>
```
其中用户名和密码根据自己情况自定义。

#### 2.修改Maven的settings.xml，在<servers>节点中加入如下代码：

```xml
<server>
	<id>tomcat7</id>
	<username>admin</username>
	<password>password</password>
</server>
```
其中 id 根据自己情况自定义，username 和 password 必须和第一步中配置的用户名密码相同。

#### 3.修改Maven项目的pom.xml文件，在<build>节点加入如下代码：

```xml
<plugins>
	<!-- tomcat7的maven插件 -->
	<plugin>
		<groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
		<configuration>
			<url>http://localhost:8080/manager/text</url> <!-- 此处是tomcat部署地址 -->
			<server>tomcat7</server> <!-- 此处的名字必须和setting.xml中配置的ID一致 -->
			<path>/SpringMVC</path> <!-- 此处的名字是项目发布的工程名 -->
		</configuration>
	</plugin>
</plugins>
```
其中 <server>节点对应的是第二步中的id

#### 4.部署
右键要部署的项目选择 Run as –> Run Configurations… 弹出如下界面：
![示例](https://oss.lumingbao.cn/images/20210927/64d849d7e8294942bd92a4522f119361.png)  
如果所示：   
* ##### 4.1 在1处右键点击Maven Build 选择New
* ##### 4.2 在2处输入自定义的Name
* ##### 4.3 点击3处的按钮，选择要部署的项目
* ##### 4.4 在4处输入：tomcat7:redeploy&#160;&#160;&#160;&#160; 其中 tomcat7为你之前定义的server id
* ##### 4.5 点击Run运行即可

