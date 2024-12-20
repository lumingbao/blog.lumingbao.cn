+++
title = "搭建 sonarqube 代码质量工具"
date = 2024-12-20 08:50:00
url = "archives/4y70teum"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/pasteimageintomarkdown/2024-12-20/297928175369583.png'
+++

# 搭建 sonarqube 代码质量工具

![](https://oss.lumingbao.cn/pasteimageintomarkdown/2024-12-20/297928175369583.png)

#### SonarQube是管理代码质量一个开放平台，可以快速的定位代码中潜在的或者明显的错误，下面将会介绍一下这个工具的安装、配置以及使用。

## 一：Docker 安装sonarqube

### 1.1：拉取 SonarQube 镜像

```shell
docker pull sonarqube
```
### 1.2：运行 SonarQube 容器

````shell
docker run -d --name sonarqube -p 9000:9000 sonarqube

生产环境推荐使用外置数据库
docker run -d --name sonarqube \
  --link my-postgres:db \
  -e SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar \
  -e SONAR_JDBC_USERNAME=postgres \
  -e SONAR_JDBC_PASSWORD=password \
  -p 9000:9000 \
  sonarqube
````

### 1.3：访问 SonarQube

````shell
打开浏览器并访问 http://localhost:9000
默认登录凭据是用户名 admin 和密码 admin
````
## 二：sonarqube基本配置
### 2.1：汉化
#### 从应用市场中安装中文语言包即可完成汉化
![](https://oss.lumingbao.cn/pasteimageintomarkdown/2024-12-20/316796466761666.png)

## 三：推送项目到 sonarqube
### 3.1：生成令牌
#### 点击右上角用户头像，选择我的账户，在安全中生成令牌

![](https://oss.lumingbao.cn/pasteimageintomarkdown/2024-12-20/302325246646541.png)

### 3.2：java项目配置
#### 在pom.xml中，加入如下代码：
````xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.sonarsource.scanner.maven</groupId>
            <artifactId>sonar-maven-plugin</artifactId>
            <version>3.9.1.2184</version>
        </plugin>
    </plugins>
</build>
````

### 3.3：执行命令
````shell
mvn sonar:sonar -Dsonar.projectKey=项目标识 -Dsonar.host.url=http://127.0.0.1:9000 -Dsonar.login=上一步生成的令牌
````