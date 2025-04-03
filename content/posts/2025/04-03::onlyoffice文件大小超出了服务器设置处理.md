+++
title = "onlyoffice文件大小超出了服务器设置处理"
date = 2025-04-03 13:00:00
url = "archives/8ayofa2e"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/images/20250403/2726b9e2fc5a4d769ebbcf782c74db86.png'
+++

# onlyoffice文件大小超出了服务器设置处理

#### 【摘要】本文针对使用onlyoffice在线编辑大文件出现的“文件大小超出服务器设置限制”问题，总结了有效的解决方案，并附带处理步骤。

## 一：前言

使用owncloud搭建局域网的网盘服务，使用onlyoffice插件实现word文档在线编辑预览，正常文件打开没有问题。产品同学突然整了两百多兆的world文档，上传到owncloud问我为啥不能打开了。

![](https://oss.lumingbao.cn/images/20250403/ee901caad2974d828221e5d55477e30b.png?imageView2/0/interlace/1/q/50|imageslim)

200MB+ word真的太夸张了。本着服务精神，但是还是给整下。发现网上很多解除onlyoffice大小限制方法，都不能用了，可能旧版本是可以，浪费了很多时间。

本文使用解决方案适用于docker容器安装的onlyoffice服务。镜像版本:onlyoffice/documentserver:7.2


## 二：修改服务器文件大小限制方案
### 2.1：旧方案-7.2版本不可行
#### 在 OnlyOffice Document Server 的 Docker 容器中，你可以通过修改 Document Server 的配置文件来调整默认可打开文件的大小限制, 网上大部分采用的方案，亲测在7.2版本是不可以用的。更低版本应该可以，如果不关心可以跳过，这里只做记录。

#### 2.1.1：进入 OnlyOffice Document Server 容器

````shell
docker exec -it your_document_server_container_name /bin/bash
````
#### 2.1.2：编辑配置文件
使用编辑器（例如 nano 或 vi）编辑配置文件，该文件位于 /etc/onlyoffice/documentserver/default.json。
````shell
nano /etc/onlyoffice/documentserver/default.json
````
onlyoffice容器中是没有vi和vim命令的，这个nano命令可以修改。第一次用应急很方便

#### 2.1.3：找到并修改文件大小限制
在配置文件中，找到 files.docservice.filetypes.maxFileSize 部分，并将其值修改为你期望的文件大小限制，以字节为单位。例如，将其修改为 50 MB：
````json
"files.docservice.filetypes.maxFileSize": 52428800,
````

#### 2.1.4：保存并退出编辑器
保存修改后的配置文件并退出编辑器。

#### 2.1.5：重启 OnlyOffice Document Server 容器
````shell
docker restart your_document_server_container_name
````

### 2.2：可行方案

适用镜像版本:onlyoffice/documentserver:7.2

#### 2.2.1：进入 OnlyOffice Document Server 容器
````shell
docker exec -it  onlyoffice_simple bash
````
#### 2.2.2：在容器内执行修改配置
在容器内复制以下脚本执行
````shell
#!/usr/bin/env bash

sed -i -e 's/104857600/10485760000/g' /etc/onlyoffice/documentserver-example/production-linux.json

sed -i '9iclient_max_body_size 1000M;' /etc/onlyoffice/documentserver-example/nginx/includes/ds-example.conf
sed -i '16iclient_max_body_size 1000M;' /etc/nginx/nginx.conf

sed -i -e 's/104857600/10485760000/g' /etc/onlyoffice/documentserver/default.json
sed -i -e 's/50MB/5000MB/g' /etc/onlyoffice/documentserver/default.json
sed -i -e 's/300MB/3000MB/g' /etc/onlyoffice/documentserver/default.json

service nginx restart
supervisorctl restart all
````

因为需要修改的配置比较多。这里可以看到是采用sed命令直接去替换配置，和补充nginx中文件上传大小的配置。旧方法修改了文件带下不生效,应该是漏补充ngnix和production-linux.json 中的配置。
