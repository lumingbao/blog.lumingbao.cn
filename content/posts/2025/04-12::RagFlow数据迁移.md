+++
title = "RagFlow数据迁移"
date = 2025-04-12 21:44:00
url = "archives/f33c0oan"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/images/20250412/81c9942d89d44941a6dafb1544821254.png'
+++

# RagFlow数据迁移

![ragflow.png](https://oss.lumingbao.cn/images/20250412/81c9942d89d44941a6dafb1544821254.png?imageView2/0/interlace/1/q/50|imageslim)


#### 【背景】在实际开发过程,项目环境有开发环境,测试环境,生产环境;如何将开发环境中的RAGFlow迁移到测试环境,甚至上到生产环境呢

## 前言

如何完整的将RAGFlow项目迁移到新机器呢?

迁移到新机器时,直接把项目拷贝过来还不可以

要把每个容器的数据也拷贝过去(dify是直接放在项目下的,很方便)


## 一：先查看一个mysql容器的数据放在哪里(ragflow-mysql:容器名)
````shell
docker inspect ragflow-mysql | grep Mounts -A 10
````
命令返回结果如下:
````json
"Mounts": [
                {
                    "Type": "volume",
                    "Source": "ragflow_mysql_data",
                    "Target": "/var/lib/mysql",
                    "VolumeOptions": {}
                }
            ],
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
--
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/home/scia/ragflow/docker/init.sql",
                "Destination": "/data/application/init.sql",
                "Mode": "rw",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "volume",
````

## 二：实际物理存储路径(docker管理卷:ragflow_mysql_data)
````shell
docker volume inspect ragflow_mysql_data | grep Mountpoint
````

命令返回的结果

````json
"Mountpoint": "/var/lib/docker/volumes/ragflow_mysql_data/_data",
````

## 三：进入到 /var/lib/docker/volumes/

````shell
cd /var/lib/docker/volumes/
ll
````

注意到一下目录
````shell
ragflow_esdata01
ragflow_infinity_data
ragflow_minio_data
ragflow_mysql_data
ragflow_redis_data
````

对以上目录进行逐一打包
````shell
tar -cvf ragflow_esdata01.tar ragflow_esdata01
tar -cvf ragflow_infinity_data.tar ragflow_infinity_data
tar -cvf ragflow_minio_data.tar ragflow_minio_data
tar -cvf ragflow_mysql_data.tar ragflow_mysql_data
tar -cvf ragflow_redis_data.tar ragflow_redis_data
````

## 四：将RagFlow项目目录也进行打包
````shell
tar -cvf ragflow.tar ragflow
```` 

## 五：通过scp命令等方式将所有压缩后的文件拷贝到新服务器，并解压


## 六：启动新项目
````shell
docker-compose up -d
````

这样就完整将项目迁移过去了
