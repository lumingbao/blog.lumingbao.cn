+++
title = ":Ollama导入本地模型"
date = 2025-02-25 18:00:00
url = "archives/0yfr7ixs"
tags = ["技巧"]
categories = ["日常记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/images/20250225/5728ee723c2748b5a111156f9921a859.png'
+++

# Ollama 导入本地模型

![](https://oss.lumingbao.cn/images/20250225/5728ee723c2748b5a111156f9921a859.png?imageView2/0/interlace/1/q/50|imageslim)


#### 当 Ollama 模型库没有提供你需要的模型时，我们可以通过 HuggingFace(https://huggingface.co) 或者 modelscope(https://huggingface.co) 来下载模型进行导入，下面将讲解详细的过程

## 一：Ollama下载安装

### 1.1：下载 Ollama 安装包

```shell
访问 https://ollama.com/download 下载自己系统对应的安装包安装即可
```
### 1.2：验证是否安装成功

````shell
安装完成，浏览器打开 localhost:11434 页面看到“ollama is running”内容表示安装成功；
````

## 二：模型下载导入
### 2.1：下载
#### 从 HuggingFace 或者 modelscope 下载模型，并解压到指定目录下，如果下载模型直接提供 GGUF 格式的模型文件，直接导入即可

### 2.2：导入
#### 2.2.1：创建模型配置文件
进入到解压后的模型目录中
````shell
vi Modelfile
````
加入如下内容：
````shell
FROM ./模型名称.gguf
````
#### 2.2.2：模型导入
执行命令：
````shell
ollama create 自定义模型名称 -f ./Modelfile
````

导入后执行：
````shell
ollama list
````
验证是否导入成功

## 三：模型转换

#### 大部分模型下载后，不会直接提供 GGUF 格式的模型文件，需要将 safetensors 格式的模型文件转换为 GGUF 格式

### 3.1：所需文件下载

下载 从 https://github.com/ggml-org/llama.cpp/releases 下载对应自己系统的包，并解压到特定目录
同时下载 https://github.com/ggml-org/llama.cpp 源代，并和上面下载的编译好的文件放到同一目录中

### 3.2：安装所有依赖库
进入 llama.cpp 目录下，执行命令：
````shell
pip install -r requirements.txt
````

### 3.3：模型转换
执行命令：
````shell
python convert_hf_to_gguf.py [模型文件夹位置]
````

### 3.4：导入模型
按照 2.2 步骤导入模型即可
