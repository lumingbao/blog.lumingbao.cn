+++
title = "MacOS 移动硬盘中的文件灰色不可用处理办法"
date = 2021-09-18 10:32:00
url = "archives/57323"
tags = ["MacOS"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner3.jpg'
+++

## MacOS 移动硬盘中的文件灰色不可用处理办法

有时候在苹果电脑上插入移动硬盘，会发现有些文件灰色状态不可用，也不能进行复制，可以用这个办法进行暂时解决

打开终端执行如下命令：

````shell
xattr -d com.apple.FinderInfo *
````

注意：*号部分替换为文件目录，或者文件所在目录