+++
title = "CentOS安装FastDFS"
date = 2018-03-12 00:00:00
url = "archives/16739"
tags = ["CentOS","FastDFS"]
categories = ["工作记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner3.jpg'
+++

## CentOS安装FastDFS

### FastDfs分布式文件系统

### 一：简介

> FastDFS是一个开源的轻量级分布式文件系统，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。
FastDFS为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务。

> 首先简单了解一下基础概念，FastDFS是一个开源的轻量级分布式文件系统，由跟踪服务器（tracker server）、存储服务器（storage server）和客户端（client）三个部分组成，主要解决了海量数据存储问题，特别适合以中小文件（建议范围：4KB < file_size <500MB）为载体的在线服务。FastDFS的系统结构图如下：

![](https://oss.lumingbao.cn/images/20211003/b01249ec1a464f199d9a0df9409e668d.png)

如上图，FastDFS的两个核心概念分别是：
* 1.Tracker（跟踪器）
* 2.Storage（存储节点）

`Tracker`主要做调度工作，相当于mvc中的controller的角色，在访问上起负载均衡的作用。跟踪器和存储节点都可以由一台或多台服务器构成，跟踪器和存储节点中的服务器均可以随时增加或下线而不会影响线上服务，其中跟踪器中的所有服务器都是对等的，可以根据服务器的压力情况随时增加或减少。Tracker负责管理所有的Storage和group，每个storage在启动后会连接Tracker，告知自己所属的group等信息，并保持周期性的心跳，tracker根据storage的心跳信息，建立group==>[storage server list]的映射表，Tracker需要管理的元信息很少，会全部存储在内存中；另外tracker上的元信息都是由storage汇报的信息生成的，本身不需要持久化任何数据，这样使得tracker非常容易扩展，直接增加tracker机器即可扩展为tracker cluster来服务，cluster里每个tracker之间是完全对等的，所有的tracker都接受stroage的心跳信息，生成元数据信息来提供读写服务。  

`Storage`采用了分卷[Volume]（或分组[group]）的组织方式，存储系统由一个或多个组组成，组与组之间的文件是相互独立的，所有组的文件容量累加就是整个存储系统中的文件容量。一个卷[Volume]（组[group]）可以由一台或多台存储服务器组成，一个组中的存储服务器中的文件都是相同的，组中的多台存储服务器起到了冗余备份和负载均衡的作用，数据互为备份，存储空间以group内容量最小的storage为准，所以建议group内的多个storage尽量配置相同，以免造成存储空间的浪费。更多原理性的内容可以参考这篇文章，介绍的很详细：分布式文件系统FastDFS设计原理http://www.linuxidc.com/Linux/2014-10/107591.htm。
### 二：安装FastDFS
#### 1：准备

下载地址：https://github.com/happyfish100/fastdfs/releases

由于FastDFS是纯C语言实现，只支持Linux、FreeBSD等UNIX系统，所以我们直接下载tar.gz的压缩包，同时FastDFS 5.0.5同以前版本相比将公共的一些函数等单独封装成了libfastcommon包，所以在安装FastDFS之前我们还必须安装libfastcommon，在GitHub首页可以看到：

![](https://oss.lumingbao.cn/images/20211003/b924ed5658054e4988233174a338cc8e.png)

将下载好的 fastdfs 的包和 libfastcommon 包上传到指定目录
(如:可新建/usr/software文件夹,用于存放上传的压缩包文件)

> 使用 tar -zxvf 命令解压 tar.gz 包  
> 使用 unzip 解压 zip 包  
> 如果unzip命令不存在，使用命令：yum -y install unzip zip  安装

#### 2：安装 libfastcommon
使用cd 命令进入解压好的包中，分别执行 ：./make.sh和./make.sh install
如果提示找不到gcc 命令，使用命令：yum -y install gcc-c++ 安装
````shell
./make.sh 为编译命令，没有error信息的话就说明编译成功了  
./make.sh install 为安装命令，看到类似如下提示信息就说明libfastcommon已安装成功
````

![](https://oss.lumingbao.cn/images/20211003/67202b4798f34682a9ff2591020a06e6.png)

至此libfastcommon就已经安装成功了，但注意一下上图中红色框标注的内容，libfastcommon.so 默认安装到了/usr/lib64/libfastcommon.so，但是FastDFS主程序设置的lib目录是/usr/local/lib，所以此处需要重新设置软链接（类似于Windows的快捷方式）：

分别执行：..
````shell
ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
````

#### 3：安装FastDFS
进入解压的 fastdfs 目录、依次执行：
````shell
./make.sh 编译命令
./make.sh install 安装命令
````
没有报错就说明安装成功了，在log中我们可以发现安装路径：  

![](https://oss.lumingbao.cn/images/20211003/ea083f185b0240f7b5d3ecd87466d254.png)

没错，正是安装到了/etc/fdfs中。

进入解压的fastdfs 目录下的 conf目录中，将 http.conf anti-steal.jpg mime.types 这三个文件拷贝到 /etc/fdfs目录下，命令如下：
````shell
cd fastdfs-5.10/conf
cp http.conf anti-steal.jpg mime.types /etc/fdfs/
````
创建配置文件：
````shell
cp -p /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
cp -p /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
cp -p /etc/fdfs/client.conf.sample /etc/fdfs/client.conf
````
至此FastDFS已经安装完毕，接下来的工作就是依次配置Tracker和Storage了。
#### 4：配置Tracker
在配置Tracker之前，首先需要创建Tracker服务器的文件路径，即用于存储Tracker的数据文件和日志文件等，我这里选择在/usr/fastdfs目录下创建一个fastdfs_tracker目录用于存放Tracker服务器的相关文件：
````shell
mkdir /usr/fastdfs/fastdfs_tracker
````
修改tracker进程的配置文件/etc/fdfs/tracker.conf
````xml
disabled=false #启用配置文件（默认启用）
port=22122 #设置tracker的端口号，通常采用22122这个默认端口
base_path=/usr/fastdfs/fastdfs_tracker #设置tracker的数据文件和日志目录
http.server_port=6666 #设置http端口号，默认为8080
````

这里需要配置防火墙例外 22122 端口，6666端口，具体情况视自己配置而定。

配置完成后就可以启动Tracker服务器了，但首先依然要为启动脚本创建软引用，因为fdfs_trackerd等命令在/usr/local/bin中并没有，而是在/usr/bin路径下：
````shell
ln -s /usr/bin/fdfs_trackerd /usr/local/bin
ln -s /usr/bin/stop.sh /usr/local/bin
ln -s /usr/bin/restart.sh /usr/local/bin
````
最后通过命令启动Tracker服务器：
````shell
service fdfs_trackerd start
````
命令执行后可以看到以下提示：  

![](https://oss.lumingbao.cn/images/20211003/24bc9ea7474441ec836d232cd0123c91.png)

如果启动命令执行成功，那么同时在刚才创建的tracker文件目录/usr/fastdfs/fastdfs_tracker中就可以看到启动后新生成的data和logs目录，tracker服务的端口也应当被正常监听，最后再通过netstat命令查看一下端口监听情况：  
`netstat -unltp|grep fdfs`

可以看到tracker服务运行的22122端口正常被监听：  

![](https://oss.lumingbao.cn/images/20211003/7adfb54092224776ba7b7c05a7222f45.png)

确认tracker正常启动后可以将tracker设置为开机启动:  
`chkconfig fdfs_trackerd on`

Tracker至此就配置好了，接下来就可以配置FastDFS的另一核心——Storage。

#### 5：配置Storage
同理，步骤基本与配置Tracker一致，首先是创建Storage服务器的文件目录，需要注意的是同Tracker相比我多建了一个目录，因为Storage还需要一个文件存储路径，用于存放接收的文件：  
````shell
mkdir /usr/fastdfs/fastdfs_storage
mkdir /usr/fastdfs/fastdfs_storage_data
````
接下来修改/etc/fdfs目录下的storage.conf配置文件，打开文件后依次做以下修改：
````xml
disabled=false #启用配置文件（默认启用）
group_name=group1 #组名，根据实际情况修改
port=23000 #设置storage的端口号，默认是23000，同一个组的storage端口号必须一致
base_path=/usr/fastdfs/fastdfs_storage #设置storage数据文件和日志目录
store_path_count=1 #存储路径个数，需要和store_path个数匹配
store_path0=/usr/fastdfs/fastdfs_storage_data #实际文件存储路径
tracker_server=192.168.20.110:22122 #tracker 服务器的 IP地址和端口号，如果是单机搭建，IP不要写127.0.0.1，否则启动不成功（此处的ip是我的CentOS虚拟机ip）
http.server_port=8888 #设置 http 端口号
````
这里需要配置防火墙例外 23000 端口，8888端口，具体情况视自己配置而定。

配置完成后同样要为Storage服务器的启动脚本设置软引用：  
`ln -s /usr/bin/fdfs_storaged /usr/local/bin`

接下来就可以启动Storage服务了：  
`service fdfs_storaged start`

命令执行后可以看到以下提示：  

![](https://oss.lumingbao.cn/images/20211003/e22ff099b3c34aa995e0133ee45f0ee6.png)

同理，如果启动成功，/usr/fastdfs/fastdfs_storage中就可以看到启动后新生成的data和logs目录，端口23000也应被正常监听，还有一点就是文件存储路径下会生成多级存储目录，那么接下来看看是否启动成功了：  
`netstat -unltp|grep fdfs`  

![](https://oss.lumingbao.cn/images/20211003/a2159adff77d4b3bbbdd785edbab0067.png)

说明启动成功。  

![](https://oss.lumingbao.cn/images/20211003/edcb21c7cb57416aa988216e9c9337b4.png)

如上图，可以看到/suer/fastdfs/fastdfs_storage/data目录下生成好的pid文件和dat文件，那么再看一下实际文件存储路径下是否有创建好的多级目录呢：  

![](https://oss.lumingbao.cn/images/20211003/5623abbb61b541a78f2db31b46359a6f.png)

如上图，没有任何问题，data下有256个1级目录，每级目录下又有256个2级子目录，总共65536个文件，新写的文件会以hash的方式被路由到其中某个子目录下，然后将文件数据直接作为一个本地文件存储到该目录中。

设置开机启动：  
`chkconfig fdfs_storaged on`

至此storage服务器就已经配置完成，确定了storage服务器启动成功后，还有一项工作就是看看storage服务器是否已经登记到 tracker服务器（也可以理解为tracker与storage是否整合成功），运行以下命令：

`/usr/bin/fdfs_monitor /etc/fdfs/storage.conf`

![](https://oss.lumingbao.cn/images/20211003/1f364652f4984e4f9c2f33475ee0d2b2.png)

如上所示，看到192.168.20.110 ACTIVE 字样即可说明storage服务器已经成功登记到了tracker服务器。

至此我们就已经完成了fastdfs的全部配置。

#### 6：测试文件上传

此时，由于目前还没有搭建完集群，因此我们暂且在tracker上使用client测试文件的上传。

* 1、进入到/etc/fdfs目录下, 可以看到client.conf.sample这个配置文件
* 2、使用命令：cp client.conf.sample client.conf复制一份该文件并命名为client.conf
* 3、编辑client.conf文件  
    需要修改的配置有:  
    `base_path=/fastdfs/tracker`  
    `tracker_server=192.168.1.11: 22122`  
  <font color=red>(注意:此处ip地址为我为自己机器设置的ip地址,具体配置时改为自己的即可,端口号为自己为Tracker服务器配置的端口号)</font>
* 4、下面我们来上传一张图片，我把/usr/local目录下一张3.jpg图片上传  
  使用的命令:  
  `/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/3.jpg`  
  可以看到这条命令由3部分组成:  
  * 第一部分是/usr/bin/fdfs_upload_file，意思是指定要进行上传文件操作;
  * 第二部分是/etc/fdfs/client.conf，意思是指定上传操作使用的配置文件，这个配置文件就是我们上面刚配置过的client.conf文件;
  * 第三部分是/usr/local/3.jpg，意思是指定要上传哪个目录下的哪个文件。
  
  按回车执行上传命令后会返回一个串：  
  <font color=red>group1/M00/00/00/wKgBC1qjdjKADXHiAABqAsOwqcU728.jpg</font>

  其中group1表示这张图片被保存在了哪个组当中;

  M00代表磁盘目录，如果电脑只有一个磁盘那就只有M00， 如果有多个磁盘，那就M01、M02...等等;  
  00/00代表磁盘上的两级目录，每级目录下是从00到FF共256个文件夹，两级就是256*256个;  
  wKicB1jjiFmAOUdkAAHk-VzqZ6w720.jpg表示被存储到storage上的3.jpg被重命名的名字;样做的目的是为了防止图片名字重复。


### 三：Nginx安装和配置
#### 1：准备

4.0.5版本开始移除了自带的HTTP支持（因为之前自带的HTTP服务较为简单，无法提供负载均衡等高性能服务），所以余大提供了nginx上使用FastDFS的模块fastdfs-nginx-module，下载地址如下：  
https://github.com/happyfish100/fastdfs-nginx-module

这样做最大的好处就是提供了HTTP服务并且解决了group中storage服务器的同步延迟问题，接下来就具体记录一下fastdfs-nginx-module的安装配置过程。

fastdfs-nginx-module

首先将nginx和fastdfs-nginx-module的安装包上传至CentOS

<font color=red>(注意:在此我们将压缩包上传至/usr/nginx文件夹下)</font>

在安装nginx之前需要先安装一些模块依赖的lib库
````shell
yum -y install pcre pcre-devel  
yum -y install zlib zlib-devel  
yum -y install openssl openssl-devel
````
依次装好这些依赖之后就可以开始安装nginx了。

#### 2：Storage Nginx安装配置

首先为storage服务器安装nginx

分别解压，进入nginx目录执行：

`./configure --prefix=/usr/nginx --add-module=/usr/nginx/fastdfs-nginx-module-master/src`

<font color=red>(注意: /usr/nginx/fastdfs-nginx-module-master/src 该路径为fastdfs-nginx-module-master.zip压缩包解压后所在的文件路径)</font>

配置成功后会看到如下信息：

![](https://oss.lumingbao.cn/images/20211003/4593d260c57646189c1abefbe281de43.png)

紧接着就可以进行编译安装了，依次执行以下命令：

````shell
make
make install
````

如果没有报错，安装正常结束。

安装完成后，我们在我们指定的目录/usr/nginx中就可以看到nginx的安装目录了：

![](https://oss.lumingbao.cn/images/20211003/679e602f157f4568abafa8907e021f4c.png)

`cp /usr/nginx/fastdfs-nginx-module-master/src/mod_fastdfs.conf /etc/fdfs/`

<font color=red>(注意: /usr/nginx/fastdfs-nginx-module-master/src/mod_fastdfs.conf 该文件为fastdfs-nginx-module-master.zip解压后文件所在的路径)
修改拷贝过去的mod_fastdfs.conf，命令：</font>

`vi /etc/fdfs/mod_fastdfs.conf`

````xml
base_path=/usr/fastdfs/fastdfs_storage
tracker_server=192.168.20.110:22122
group_name=group1
url_have_group_name = true
store_path0=/usr/fastdfs/fastdfs_storage_data
````

接下来要修改一下nginx的配置文件，进入conf目录并打开nginx.conf文件加入以下配置：

````xml
server {
        listen       80;
        server_name  localhost;
        location ~/group[0-9]/M00 {
            ngx_fastdfs_module;
        }	
}
````

进入nginx下的sbin目录，执行 ./nginx 启动nginx

Nginx常用命令:

* 查看进程:ps -ef | grep nginx
* 启动: /usr/nginx/sbin/nginx
* 重启: /usr/nginx/sbin/nginx -s reload
* 关闭: /usr/nginx/sbin/nginx -s stop

访问文件路径：（此处的文件是我刚才上传的文件哦!）http://192.168.1.11/group1/M00/00/00/wKgBC1qjdjKADXHiAABqAsOwqcU728.jpg

这样一来，下载来的文件名为生成的索引，而不是原文件名。

访问文件强制下载(传参自定义文件名字):

在nginx的 nginx.conf 的 location 节点下加入:

`add_header Content-Disposition "attachment;filename=$arg_attname";`  
如图所示:  

![](https://oss.lumingbao.cn/images/20211003/66bf1a1f282141c7958e96dfaa2db0d5.png)

访问文件地址的时候，在文件后缀加入?attname=filename.jpg 如：

http://192.168.1.11/group1/M00/00/00/wKgBC1qjdjKADXHiAABqAsOwqcU728.jpg?attname=new.jpg

Nginx 注册为服务

创建nginx启动命令脚本：

`vi /etc/init.d/nginx`

插入以下内容, 注意修改PATH和NAME字段, 匹配自己的安装路径

````
#! /bin/bash
# chkconfig: 2345 80 90
PATH=/usr/nginx
DESC="nginx daemon"
NAME=nginx
DAEMON=$PATH/sbin/$NAME
CONFIGFILE=$PATH/conf/$NAME.conf
PIDFILE=$PATH/logs/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
set -e
[ -x "$DAEMON" ] || exit 0
do_start() {
$DAEMON -c $CONFIGFILE || echo -n "nginx already running"
}
do_stop() {
$DAEMON -s stop || echo -n "nginx not running"
}
do_reload() {
$DAEMON -s reload || echo -n "nginx can't reload"
}
case "$1" in
start)
echo -n "Starting $DESC: $NAME"
do_start
echo "."
;;
stop)
echo -n "Stopping $DESC: $NAME"
do_stop
echo "."
;;
reload|graceful)
echo -n "Reloading $DESC configuration..."
do_reload
echo "."
;;
restart)
echo -n "Restarting $DESC: $NAME"
do_stop
do_start
echo "."
;;
*)
echo "Usage: $SCRIPTNAME {start|stop|reload|restart}" >&2
exit 3
;;
esac
exit 0
````
设置执行权限：

`chmod a+x /etc/init.d/nginx`

注册成服务：

`chkconfig --add nginx`

检查mysqld服务是否已经生效

`chkconfig --list nginx`

设置开机启动：

`chkconfig nginx on`

重启, 查看nginx服务是否自动启动

`netstat -apn|grep nginx`