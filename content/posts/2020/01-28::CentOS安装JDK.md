+++
title = "CentOS安装JDK"
date = 2020-01-28 19:46:00
url = "archives/93072"
tags = ["CentOS", "JDK"]
categories = ["工作记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner3.jpg'
+++

## Centos下安装JDK
查看是否配置了jdk：java -version  
查看是否安装openjdk命令 ： rpm -qa | grep jdk  
卸载jdk ： yum -y remove ~~java-1.6.0-openjdk-1.6.0.0-1.45.1.11.1.el6.i686~~
（后面部分为使用查看命令得到的名称）

查看环境变量：echo $PATH

解压：
````shell
tar -zxvf jdk-8u131-linux-x64.gz
````

JAVA环境变量：
````shell
vi /etc/profile

export JAVA_HOME=/usr/java/jdk1.8.0_131
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$CLASSPATH
export PATH=$JAVA_HOME/bin:$PATH
````
按Esc键盘，输入 :wq保存退出后  
在输入指令：source /etc/profile     使设置的环境变量生效。  
再输入 ： java -version 查看版本信息，验证jdk是否安装成功。
