+++
title = "09-02::Docker部署Zabbix"
date = 2022-09-02 15:58:00
url = "archives/mv2jaib0"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner1.jpg'
+++

# Docker部署Zabbix


## 一：安装 Zabbix 服务

创建docker网络
````shell
docker network create -d bridge zabbix_net
````

创建zabbix服务，注意数据库配置改为自己的数据库信息，启动时会自动创建 zabbix 数据库
````shell
docker run \
--name zabbix-server \
-p 10051:10051 \
-e DB_SERVER_HOST="数据库服务Ip" \
-e MYSQL_USER="数据库用户名" \
-e MYSQL_PASSWORD="数据库密码" \
--network zabbix_net -d zabbix/zabbix-server-mysql
````

创建zabbix web 服务，注意数据库配置改为自己的数据库信息
````shell
docker run --name zabbix-web \
-p 8089:8080 \
--link zabbix-server:zabbix-server \
-e DB_SERVER_HOST="数据库服务Ip" \
-e MYSQL_USER="数据库用户名" \
-e MYSQL_PASSWORD="数据库密码" \
-e ZBX_SERVER_HOST="zabbix-server" \
-e PHP_TZ="Asia/Shanghai" \
--network zabbix_net -d zabbix/zabbix-web-nginx-mysql
````

## 二：安装 Zabbix agent

### docker 安装形式
````shell
docker run -d --name zabbix-agent \
-e ZBX_HOSTNAME="172.19.0.2" \
-e ZBX_SERVER_HOST="172.19.0.2" \
-p 10050:10050 \
--link zabbix-server:zabbix-server \
--network zabbix_net zabbix/zabbix-agent:latest
````

### rpm包安装形式

rmp 包下载地址：https://repo.zabbix.com/zabbix/

安装 rpm 包
````shell
rpm -Uvh openssl11-libs-1.1.1k-2.el7.x86_64.rpm
rpm -Uvh zabbix-agent-5.4.9-1.el8.x86_64.rpm
````

修改 agent 配置为服务器IP
````shell
vi /etc/zabbix/zabbix_agentd.conf
````

agent 服务操作
````shell
systemctl status zabbix-agent.service
systemctl stop zabbix-agent.service
systemctl start zabbix-agent.service
````
