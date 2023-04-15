+++
title = "SpringBoot 集成ElasticSearch"
date = 2020-01-28 19:42:00
url = "archives/47730"
tags = ["SpringBoot", "ElasticSearch"]
categories = ["SpringBoot"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner2.jpg'
+++

## SpringBoot 集成ElasticSearch

###### 一：安装ElasticSearch

>ElasticSearch简介：分布式索引。

> ElasticSearch与MySql 关系：  
> DB type –> Table
> Document –> row

ElasticSearch已有5.x版本，然而Spring Data 目前只支持ElasticSearch2.X版本，支持说明：https://github.com/spring-projects/spring-data-elasticsearch/wiki/Spring-Data-Elasticsearch---Spring-Boot---version-matrix

手动安装Elasticsearch

Linux下无法用root用户启动 ElasticSearch，创建 elasticsearch用户，命令如下：

````shell
[root@slave2 ~] useradd es 
[root@slave2 ~] passwd es
````

获取ElasticSearch，地址：https://www.elastic.co/downloads/past-releases

安装ElasticSearch：
解压，修改config下的elasticsearch.yml

配置 :  
cluster.name: elasticsearch #这是集群名字，我们起名为 elasticsearch。es启动后会将具有相同集群名字的节点放到一个集群下。  
node.name: node-1  
network.host: 192.168.20.110  
discovery.zen.minimum_master_nodes: 1  

修改用户权限，使用root用户执行如下命令：  
````shell
chown -R 用户名 elasticsearch-2.4.0
````
运行：  
进入 bin 目录，执行 ./elasticsearch

没有报错启动成功，访问 IP:9200，提示json信息，表示运行成功。

自动安装Elasticsearch

1．下载 elasticsearch rpm 安装包，上传至linux 相应目录下，执行安装命令：  
````shell
yum install elasticsearch.rmp
````
2.安装完毕以后查看安装目录，命令：
````shell
rpm -ql elasticsearch
````
3.修改配置文件
````shell
vi /etc/sysconfig/elasticsearch
````
加入 JAVA_HOME=” /usr/java/jdk1.8.0_131”  
指定jdk目录,因为service 脚本无法读取 JAVA_HOME环境变量

4.执行启动命令：
````shell
service elasticsearch start
````
查看启动状态：
````shell
service elasticsearch status
````

5.设置开机启动，命令：
````shell
chkconfig elasticsearch on
````
6.修改配置文件，使外部可以访问：
````shell
vi /etc/elasticsearch/elasticsearch.yml
````
###### 二：服务化Elasticsearch
完成了上述步骤以后，直接可以使用脚本启动服务.但是使用起来不是很方便.比如我们的日志是直接打印到了控制台上了.退出后 ElasticSearch 直接就退出了.不太好使.因此需要进行一些处理.

ElasticSearch 守护进程 java-service-wrapper 在 2.x 使用的解决方法。  
elasticsearch-1.x 版本直接使用：https://github.com/elastic/ela ... apper 则没什么问题，按照向导启动即可，最近在弄elasticsearch-2.0 时，直接把 1.x 下的守护程序 copy 过来后，启动出现问题。其中几个变化有：
1. es 不再使用 sigar 来进行监控系统资源了(这里对守护程序无影响)。  
2. elasticsearch 的启动类从 org.elasticsearch.bootstrap.ElasticsearchF 变更到 org.elasticsearch.bootstrap.Elasticsearch，并且在后续版本删除了 ElasticsearchF 类。  
3. 为了安全，不再建议使用 root  权限来运行 es。

这里我目前的解决方案是依然使用 root 权限来启动，非 root 用户下启动暂未验证。方法如下：
1. 既然 sigar 没了，先注释掉 sigar。
2. 改变启动类为:
````xml
wrapper.app.parameter.1=org.elasticsearch.bootstrap.Elasticsearch
wrapper.app.parameter.2=start
````
3. 允许 root 用户运行，并禁止掉类权限验证：
````xml
wrapper.java.additional.1=-Des.insecure.allow.root=true
wrapper.java.additional.2=-Des.security.manager.enabled=false
````
注：希望有非 root  用户下运行该守护程序的解决方案的同学提供下解决方法，在此不胜感激。
此方法降低了安全性，大家慎重考虑。
不喜欢折腾的同学直接使用 rpm 安装即可。

服务化 ElasticSearch 是使用的是一个 开源的 ServiceWrapper 脚本完成了.这个脚本的 ElasticSearch 的官方开发人员提供的.

下载插件

GitHub 地址: https://github.com/elastic/elasticsearch-servicewrapper

下载zip文件，解压到指定目录，命令：unzip service.zip

执行拷贝命令将service目录拷贝到 elasticsearch 安装目录下bin目录下的service

````shell
cp -r  ~/master/elasticsearch-servicewrapper-master/service/ /home/q/elasticsearch172/bin/service
````
修改elasticsearch.conf 设置 ES_HOME:/sur/elasticsearch/elasticsearch-2.4.5/

安装服务，进入service目录执行：

给 elasticsearch 文件给执行权限命令：
````shell
chmod a+x elasticsearch
````
执行：
````shell
elasticsearch install
````
启动服务：
````shell
service elasticsearch start
````
chmod命令详解：  
chmod [ugoa] [+(加入)-(除去)=(设定)] [rwx] 文件或目录  
u 表示“用户（user）”，即文件或目录的所有者。  
g 表示“同组（group）用户”，即与文件属主有相同组ID的所有用户。  
o 表示“其他（others）用户”。  
a 表示“所有（all）用户”。它是系统默认值。  

r 可读。  
w 可写。  
x 可执行。  

###### 三：插件及其安装
BigDesk Plugin : 对集群中es状态进行监控。  
Elasticsearch Head Plugin: 对ES进行各种操作，如查询、删除、浏览索引等。

1.安装head插件
1. https://github.com/mobz/elasticsearch-head下载zip 解压
2. 建立elasticsearch-2.4.0\plugins\head文件
3. 将解压后的elasticsearch-head-master文件夹下的文件copy到head
````shell
cp -r elasticsearch-head-master/* /usr/share/elasticsearch/plugins/head
````
4. 运行es  
访问：http://ip:9200/_plugin/head/


2.安装bigdesk

1. 下载bigdesk插件包 https://github.com/lukas-vlcek/bigdesk 下载master.zip后解压
2. 建立elasticsearch-2.x\plugins\bigdesk\_site文件夹
3. 将解压后的bigdesk-master文件夹下的文件copy到_site
4. 在plugin\bigdesk下创建 plugin-descriptor.properties
````shell
vim plugin-descriptor.properties
````
添加以下内容:
````shell
description=bigdesk - A web front end for an elastic search cluster
version=master
site=true
name=bigdesk
````
5. vim _site/js/store/BigdeskStore.js  定位到142行  
   去掉major == 1条件
6. 重启es，访问http://ip:9200/_plugin/bigdesk/

###### 四：Springboot集成ES

pom.xml 引用相关依赖：
````xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
    <version>2.1.4.RELEASE</version>
</dependency>
````

在application.properties配置elasticsearch 相关信息
````xml
spring.data.elasticsearch.cluster-name=elasticsearch
spring.data.elasticsearch.cluster-nodes=192.168.20.102:9300
spring.data.elasticsearch.local=false
spring.data.elasticsearch.repositories.enabled=true
````