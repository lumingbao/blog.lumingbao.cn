+++
title = "04-20::Docker搭建Ngrok内网穿透"
date = 2022-04-20 17:00:00
url = "archives/a010ac2d"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner1.jpg'
+++

# Docker搭建Ngrok内网穿透

## 一：准备
要有云服务器一台;  
公网域名一个，可以是一级域名也可以是二级域名，我这里使用二级域名ngrok.mydomain.com进行举例；

## 二：解析域名

A记录 解析 ngrok.mydomain.com到服务器ip  
CNAME 解析 *.ngrok.mydomain.com 到 ngrok.mydomain.com

## 三：开始搭建
### 1.下载镜像
````shell
docker pull lumingbao/ngrok
````

### 2.启动一个容器生成ngrok客户端,服务器端和CA证书
````
docker run --rm -it -e DOMAIN="ngrok.mydomain.com" -v /root/ngrok:/myfiles lumingbao/ngrok /bin/sh /build.sh
````
### 3.启动Ngrok server
````
docker run -idt --name ngrok-server -v /root/ngrok:/myfiles -p 80:80 -p 443:443 -p 4443:4443 -e DOMAIN='ngrok.mydomain.com' --restart=always lumingbao/ngrok /bin/sh /server.sh
````
### 4.创建本地config.yml配置文件，文件内容：
````properties
server_addr: "ngrok.mydomain.com:4443"
trust_host_root_certs: false
tunnels:
  client:
    subdomain: "test"
    proto:
      https: 80
````
````shell
说明：
1：server_addr 后面的域名为你的域名，端口为启动 ngrok 服务时指定的 4443 映射端口
2：subdomain 为内网穿透域名的前缀
3：https 部分为内网穿透域名的协议，可选  http 和 https
4：https 后面的 80 为你要映射的本地端口
````

### 5.根据平台下载对应的客户端工具并启动
根据你的客户端版本，在 /root/ngrok/bin 目录下，下载对应的客户端文件

### 6.启动
````shell
./ngrok -config=config.yml start client
````


注意：如果linux 启动客户端报错：  
./ngrok: /lib/ld-musl-x86_64.so.1: bad ELF interpreter: No such file or directory  
解决办法：
````shell
wget https://copr.fedorainfracloud.org/coprs/ngompa/musl-libc/repo/epel-7/ngompa-musl-libc-epel-7.repo -O /etc/yum.repos.d/ngompa-musl-libc-epel-7.repo
yum install -y musl-libc-static
````
### 7.Linux下ngrok后台运行
nohup 不支持ngrok的后台运行，这里用 screen

安装screen：
````shell
yum install screen -y
````
创建任务:
```shell
screen -S myngrok
```
“myngrok” 是自定义名称，执行后会自动清屏，不要慌,没事,继续。。。

然后正常启动ngrok，关闭窗口，注意：不要 Ctrl + C，直接关闭当前回话窗口就好了，此时ngrok已经在后台运行了，验证一下就好了。

打开screen任务：
```shell
screen -r  myngrok
```

(myngrok 这个名称就是前面自定义的) ，可以在这选择关闭，Ctrl +C 或者查找进程,直接 kill 进程
```shell
ps -ef | grep myngrok
```

### 8.nginx 代理配置例子
一般情况下，80 端口和 443 端口不会直接给 ngrok 服务使用，需要用 Nginx 进行代理
```xml
server {
        listen 80;
        listen [::]:80;

        keepalive_timeout 70;

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name ngrok.lzfsd.com *.ngrok.lzfsd.com;
        location / {
            proxy_pass http://localhost:88/;
            proxy_redirect off;
            proxy_connect_timeout 1;
            proxy_send_timeout 120;
            proxy_read_timeout 120;

            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #千万不能带80，后面我会告诉你为何
            #proxy_set_header Host  $http_host:80;
            proxy_set_header Host  $http_host;
            proxy_set_header X-Nginx-Proxy true;
            proxy_set_header Connection "";
        }
    }


    server {
        listen 443 ssl;
        server_name  *.ngrok.lzfsd.com;
        ssl_certificate /etc/letsencrypt/live/ngrok.lzfsd.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/ngrok.lzfsd.com/privkey.pem;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        location / {
            proxy_pass https://localhost:444/;
            proxy_redirect off;
            proxy_connect_timeout 1;
            proxy_send_timeout 120;
            proxy_read_timeout 120;

            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #千万不能带443，后面我会告诉你为何
            #proxy_set_header Host  $http_host:443;
            proxy_set_header Host  $http_host;
            proxy_set_header X-Nginx-Proxy true;
            proxy_set_header Connection "";
        }
    }
```

END

