+++
title = "mysqldump 导出 数据+结构+函数+存储过程"
date = 2021-03-24 08:19:00
url = "archives/52346"
tags = ["MySQL"]
categories = ["工作记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner4.jpg'
+++

## mysqldump 导出 数据+结构+函数+存储过程

### MySQL导出：

导出单个数据库：结构 无数据

````shell
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt --no-data db_name >~/db_name.sql
````

导出单个数据库：有数据 无结构

````shell
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt --no-create-info db_name >~/db_name.sql
````

导出单个数据库：结构+数据
````shell
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt db_name >~/db_name.sql
````

导出单个数据库：结构+数据+函数+存储过程
````shell
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt -R db_name >~/db_name.sql
````

导出多个数据库：结构
````shell
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt --databases db_name1 db_name2 >~/db_name.sql
````

导出单个表：结构 无数据
````shell
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt --no-data -d db_name table>~/table_name.sql
````

导出单个表：结构 包含数据
````shell
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt dbname  tablename>～/tablename.sql
````

### MySQL导入：

````shell
[root@localhost ~]# mysql -u root -p 
[root@localhost ~]# use  要导入的数据库
[root@localhost ~]#source /home/1.sql
````
或者

````shell
[root@localhost ~]# mysql -u root -p<  /home/1.sql
````
