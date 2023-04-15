+++
title = "Linux下Mysql每天自动备份的实现"
date = 2021-01-05 09:51:00
url = "archives/75275"
tags = ["Linux", "MySQL"]
categories = ["工作记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner1.jpg'
+++

## MacOS Linux下Mysql每天自动备份的实现

#### 新建目录

````shell
mkdir -p /data/mysqlbak/data
mkdir -p /data/mysqlbak/scripts
mkdir -p /data/mysqlbak/logs
````

#### 创建备份脚本

````shell
cd /data/mysqlbak/scripts
vi backup.sh
````

````shell
#!/bin/bash

#备份目录
BACKUP_ROOT=/data/mysqlbak
BACKUP_FILEDIR=$BACKUP_ROOT/data

#当前日期
DATE=$(date +%Y%m%d)

######备份######

#查询所有数据库
#-uroot -p123456表示使用root账号执行命令，且root账号的密码为:123456
DATABASES=$(mysql -uroot -p123456 -e "show databases" | grep -Ev "Database|sys|information_schema|performance_schema|mysql")
#循环数据库进行备份
for db in $DATABASES
do
echo
echo ----------$BACKUP_FILEDIR/${db}_$DATE.sql.gz BEGIN----------
mysqldump -uroot -p123456 --default-character-set=utf8 -q --lock-all-tables --flush-logs -E -R --triggers -B ${db} | gzip > $BACKUP_FILEDIR/${db}_$DATE.sql.gz
echo ----------$BACKUP_FILEDIR/${db}_$DATE.sql.gz COMPLETE----------
echo
done

echo "done"
````

#### 设置脚本的执行权限

````shell
chmod 777 backup.sh
````

#### 将备份操作加入到定时任务(每天凌晨2点定时执行)

````shell
crontab -e

00 2 * * * /data/mysqlbak/scripts/backup.sh > data/mysqlbak/logs/backup.log 2>&1
````

#### 创建删除脚本(定时删除7天前的备份数据)

````shell
vi backup_clean.sh
````

````shell
#!/bin/bash
echo ----------CLEAN BEGIN----------
find /data/mysqlbak/data -mtime +7 -name "*.gz" -exec rm -rf {} \;
echo ----------CLEAN COMPLETE----------
````

#### 设置脚本的执行权限

````shell
chmod 777 backup_clean.sh
````

#### 将删除操作加入到定时任务(每天凌晨1点定时执行)

````shell
00 1 * * * /data/mysqlbak/scripts/backup_clean.sh > /data/mysqlbak/logs/backup_full_clean.log 2>&1
````

#### 查看定时任务

````shell
crontab -l
````

#### 如果需要备份到另外一台机器，可以备份完scp到另外一台机器

首先服务器需要安装export，yum安装：
````shell
yum  install expect
````

或者源码安装，参考：https://www.cnblogs.com/operationhome/p/9154055.html

脚本修改：
````shell
#!/bin/bash

#备份目录
BACKUP_ROOT=/data/mysqlbak
BACKUP_FILEDIR=$BACKUP_ROOT/data

#当前日期
DATE=$(date +%Y%m%d)

######备份######

#查询所有数据库
#-uroot -p123456表示使用root账号执行命令，且root账号的密码为:123456
DATABASES=$(mysql -uroot -p123456 -e "show databases" | grep -Ev "Database|sys|information_schema|performance_schema|mysql")
#循环数据库进行备份
for db in $DATABASES
do
echo
echo ----------$BACKUP_FILEDIR/${db}_$DATE.sql.gz BEGIN----------
mysqldump -uroot -p123456 --default-character-set=utf8 -q --lock-all-tables --flush-logs -E -R --triggers -B ${db} | gzip > $BACKUP_FILEDIR/${db}_$DATE.sql.gz
echo ----------$BACKUP_FILEDIR/${db}_$DATE.sql.gz COMPLETE----------
echo ----------scp 226  begin----------
expect -c "
    spawn scp -r /data/mysqlbak/data/${db}_$DATE.sql.gz root@xxx.xxx.xxx.226:/data/mysqlbak/data225/
    expect {
        \"*assword\" {set timeout 300; send \"此处是scp的密码\r\"; exp_continue;}
        \"yes/no\" {send \"yes\r\";}
    }
expect eof"
echo ----------scp 226  end----------
echo
done

echo "done"
````