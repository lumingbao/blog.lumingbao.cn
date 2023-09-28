+++
title = "Mac环境下安装Node.js多版本管理工具NVM"
date = 2023-09-28 14:40:00
url = "archives/nmcs5jwv"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner1.jpg'
+++

# Mac环境下安装Node.js多版本管理工具NVM


## 一：安装

日常开发过程中，手动进行Node.js的多版本管理非常不便。这里推荐使用nvm。对于Mac环境，不推荐使用brew进行安装。可先通过下述链接获取nvm安装脚本并执行脚本来实现
````shell
# nvm安装脚本
https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh
# 执行脚本
sh ./install.sh
````

这里的终端环境为zsh，故在～/.zshrc文件当中添加如下配置，并重新加载配置文件
````shell
# 在.zshrc文件中添加如下配置
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

# 重新加载配置文件
source ~/.zshrc
````

一切完毕后，可通过查看nvm版本确认是否安装成功
````shell
# 查看nvm版本
nvm -v
````

同时由于众所周知的原因，这里将NVM源切换为国内的镜像源。这里我们使用淘宝的镜像源。故在～/.zshrc文件当中添加相应的环境变量

````shell
# 添加环境变量, 值为国内的淘宝镜像源
export NVM_NODEJS_ORG_MIRROR=http://npmmirror.com/mirrors/node
export NVM_IOJS_ORG_MIRROR=http://npmmirror.com/mirrors/iojs

# 重新加载配置文件
source ~/.zshrc
````

## 二：常用命令

````shell
# 查看可安装版本
nvm ls-remote

# 查看已安装版本
nvm ls

# 安装指定版本
nvm install <Node.js的版本号>

# 卸载指定版本
nvm uninstall <Node.js的版本号>

# 查看当前版本
nvm current

# 切换至指定版本
nvm use <Node.js的版本号>
nvm use <别名>

# 设置默认版本
nvm alias default <Node.js的版本号>

# 对指定版本添加别名
nvm alias <别名> <Node.js的版本号>

# 删除已定义别名
nvm unalias <别名>
````

