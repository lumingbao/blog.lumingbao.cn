+++
title = "MacOS下制作种子图片、将种子隐藏至图片内"
date = 2022-01-10 13:00:00
url = "archives/35543"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner3.jpg'
+++

## MacOS下制作种子图片、将种子隐藏至图片内


开始动手~

1. 准备： 环境: Mac os 10.13.5 、工具:iTerm2 、 命令:zip和unzip
2. 第一步 压缩t.mp4文件为test.zip包: zip test.zip t.mp4
3. 第二步 将压缩包写入图片并生成新的图片: cat test.jpg test.zip > private.jpg
4. 第三步 解压图片: unzip private.jpg