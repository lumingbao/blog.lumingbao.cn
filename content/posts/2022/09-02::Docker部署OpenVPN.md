+++
title = "09-02::Docker部署OpenVPN"
date = 2022-09-02 16:58:00
url = "archives/l0bfx4kv"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner2.jpg'
+++

# Docker部署OpenVPN


### 拉取镜像、创建目录
````shell
docker pull kylemanna/openvpn:2.4
mkdir -p /data/openvpn
````

### 1、使用openvpn生成配置文件，注意目录映射和IP地址换成自己的
````shell
docker run \
-v /data/openvpn:/etc/openvpn \
--rm kylemanna/openvpn:2.4 ovpn_genconfig \
-u udp://127.0.0.1
````
执行完命令后可在目录/data/openvpn中看到相应的配置文件

### 2、初始化密钥文件
````shell
docker run \
-v /data/openvpn:/etc/openvpn \
--rm -it kylemanna/openvpn:2.4 ovpn_initpki
````
执行过程中需要先设置ca密码（如123456），Common Name可不设置直接按回车继续，接着需要输入ca密码更新密钥库以及生成crl文件

### 3、生成客户端证书
````shell
docker run \
-v /data/openvpn:/etc/openvpn \
--rm -it kylemanna/openvpn:2.4 easyrsa build-client-full xxxxxxx nopass
````
其中 xxxxxxx 为自定义名称，生成的过程需要输入ca密码

### 4、导出客户端的配置文件openvpn-client.ovpn
````shell
docker run \
-v /data/openvpn:/etc/openvpn \
--rm kylemanna/openvpn:2.4 ovpn_getclient xxxxxxx > /data/openvpn/openvpn-client.ovpn
````
注意openvpn-client名称需与第三步生成时命名一致，此时生成的配置文件openvpn-client.ovpn即可用于客户端连接

### 5、启动openvpn
````shell
docker run \
-v /data/openvpn:/etc/openvpn \
-d -p 1194:1194/udp \
--restart=always \
--name openvpn \
--cap-add=NET_ADMIN \
--sysctl net.ipv6.conf.all.disable_ipv6=0 \
--sysctl net.ipv6.conf.default.forwarding=1 \
--sysctl net.ipv6.conf.all.forwarding=1 \
kylemanna/openvpn:2.4
````

### 6、开启用户名密码认证

#### 6.1 修改 openvpn.conf 配置文件
修改 运行用户为 root  
加入如下内容：
````shell
### 用户名密码认证
script-security 3
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env    #指定用户认证脚本
username-as-common-name
client-cert-not-required
````

#### 6.2 创建用户名密码文件
````shell
vi /data/openvpn/psw-file
````
文件内容。注意：一行代表一个用户，前面是用户名后面是密码，用户名和密码用空格隔开
````shell
user1 123456
user2 123456
````

#### 6.3 创建用户认证脚本
````shell
vi /data/openvpn/checkpsw.sh
````
文件内容如下：
````shell
#!/bin/sh
###########################################################
# checkpsw.sh (C) 2004 Mathias Sundman <mathias@openvpn.se>
#
# This script will authenticate OpenVPN users against
# a plain text file. The passfile should simply contain
# one row per user with the username first followed by
# one or more space(s) or tab(s) and then the password.

PASSFILE="/etc/openvpn/psw-file"
LOG_FILE="/var/log/openvpn-password.log"
TIME_STAMP=`date "+%Y-%m-%d %T"`

###########################################################

if [ ! -r "${PASSFILE}" ]; then
  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >>   ${LOG_FILE}
  exit 1
fi

CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`

if [ "${CORRECT_PASSWORD}" = "" ]; then
  echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
  exit 1
fi

if [ "${password}" = "${CORRECT_PASSWORD}" ]; then
  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
  exit 0
fi

echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1
````
注意：PASSFILE 和 LOG_FILE 配置文件和自己的路径要匹配
````shell
chmod 777 checkpsw.sh
````

#### 6.4 重启 openvpn docker容器
````shell
docker restart xxx
````

#### 6.5 修改客户端配置文件，在文件的最后加入：
````shell
auth-user-pass
````