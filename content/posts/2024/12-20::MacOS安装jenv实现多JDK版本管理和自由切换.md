+++
title = "MacOS安装jenv实现多JDK版本管理和自由切换"
date = 2024-12-20 11:30:00
url = "archives/mhv016ey"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/pasteimageintomarkdown/2024-12-20/303753260481875.png'
+++

# MacOS安装jenv实现多JDK版本管理和自由切换

<img src="https://oss.lumingbao.cn/pasteimageintomarkdown/2024-12-20/303753260481875.png" width="400" />

#### 在MacOS上安装 jenv 来实现多JDK版本的管理和自由切换，可以按照以下步骤进行。jenv 是一个Java环境管理工具，它允许你方便地在不同的JDK版本之间切换。

## 一：安装 Homebrew

#### 如果你还没有安装Homebrew，请先安装Homebrew。Homebrew是一个流行的包管理工具，可以方便地安装和管理软件包。

````shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
````


## 二：使用 Homebrew 安装 jenv
### 2.1 使用以下命令安装 jenv：
````shell
brew install jenv
````
### 2.2 配置 jenv
#### 安装完 jenv 后，需要将它添加到你的Shell配置文件中，以便每次打开终端时都能使用 jenv
#### 对于 bash，添加以下内容到 ~/.bash_profile 或 ~/.bashrc 文件中：
````shell
export PATH="$HOME/.jenv/bin:$PATH"
eval "$(jenv init -)"
````

#### 对于 zsh（MacOS Catalina 及更新版本默认使用的shell），添加以下内容到 ~/.zshrc 文件中：
````shell
export PATH="$HOME/.jenv/bin:$PATH"
eval "$(jenv init -)"
````

#### 然后，重新加载配置文件：
#### 对于 bash：
````shell
source ~/.bash_profile
````

#### 对于 zsh：
````shell
source ~/.zshrc
````

### 2.3 安装和配置JDK
#### 你可以通过Homebrew来安装不同版本的JDK。以下是一些示例命令：
````shell
brew install openjdk@8
brew install openjdk@11
brew install openjdk@17
````

#### 安装完成后，你可以使用 jenv 将这些JDK版本添加到 jenv 的管理中。
#### 假设你安装了上面的JDK版本，可以按如下步骤添加它们：
````shell
# 添加JDK 8
jenv add /usr/local/opt/openjdk@8/libexec/openjdk.jdk/Contents/Home

# 添加JDK 11
jenv add /usr/local/opt/openjdk@11/libexec/openjdk.jdk/Contents/Home

# 添加JDK 17
jenv add /usr/local/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home
````

### 2.4 切换JDK版本
#### 你可以使用以下命令来查看已经添加的JDK版本：
````shell
jenv versions
````

#### 可以使用以下命令在全局（所有终端会话）范围内设置默认的JDK版本：
````shell
jenv global <version>
例如：
jenv global 11.0
````

#### 你也可以在特定的shell会话中设置JDK版本：
````shell
jenv shell <version>
例如：
jenv shell 17.0
````

#### 还可以在特定项目目录中设置JDK版本，这样当你进入该目录时会自动使用指定的JDK版本：
````shell
jenv local <version>
例如：
jenv local 8.0
````

### 2.5 验证配置
#### 你可以通过以下命令验证当前使用的JDK版本：
````shell
java -version
````

### 2.6 启用插件
#### jenv 还提供了一些有用的插件，比如 maven 和 export。可以启用这些插件以增强 jenv 的功能：
````shell
jenv enable-plugin maven
jenv enable-plugin export
````

#### 启用 export 插件后，jenv 会自动设置 JAVA_HOME 环境变量。
#### 通过以上步骤，你就可以在MacOS上安装并配置 jenv 来管理和切换多个JDK版本了。这将大大简化开发过程中需要频繁切换JDK版本的操作。