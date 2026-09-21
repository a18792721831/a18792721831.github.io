---
layout: post
title: "spark环境docker镜像二次开发--解决中文乱码"
date: 2020-02-12 17:39:45 +0800
categories: [spark镜像二次开发, 官网镜像本地化, 官网镜像定制化, spark如何启动, spark启动异常排查]
description: "本文介绍如何通过修改Spark Docker镜像的语言环境来解决中文乱码问题，包括准备环境、编写Dockerfile及异常排查步骤。"
keywords: spark镜像二次开发, 官网镜像本地化, 官网镜像定制化, spark如何启动, spark启动异常排查
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104282040
> - 发布时间：2020-02-12 17:39:45
> - 阅读量：556
> - 分类：spark同时被 2 个专栏收录, 订阅专栏, docker
> - 标签：#spark镜像二次开发, #官网镜像本地化, #官网镜像定制化, #spark如何启动, #spark启动异常排查

## 摘要

文章浏览阅读556次。本文介绍如何通过修改Spark Docker镜像的语言环境来解决中文乱码问题，包括准备环境、编写Dockerfile及异常排查步骤。

---

#### spark环境docker镜像二次开发

  * [0.前言](<#0_8>)
  * [1.准备](<#1_15>)
  * [2.编写dockerfiel](<#2dockerfiel_20>)
  * [3.启动](<#3_45>)
  * [4.异常排查](<#4_49>)

  
  
一种更加简单的方式：  
启动命令增加-e，将语言环境设置进去
    
    
    docker run -d -p 8081:8081 -v /tmp/docker-spark-worker/:/tmp/ -e LANG="en_US.utf8" -e LC_ALL="en_US.utf8" --name=spark-worker mattf/spark-worker:2.2.1 {IP}
    

## 0.前言

官网的spark环境是英文环境，如果我们的spark在国内运行，就会出现中文乱码的问题。  
所以二次开发可以在官网镜像的基础上，定制一些本地化的修改。

本次修改较小，只是解决中文乱码问题。  
中文乱码问题本质上是请求与返回的中文编码方式不一致的原因造成的。  
所以本次就是以修改镜像的语言环境作为最终目标，将语言编码修改为utf-8编码。

## 1.准备

因为我们是docker环境，所以必须有docker环境。  
基于官网镜像，所以，必须本地有官网镜像。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f4c0a5eac4e3ff0aa51856123038e439.png)  
以2.2.1为例

## 2.编写dockerfiel

在这个dockerfile里面就是我们需要定制化的命令。  
以官网镜像为基础  
from mattf/spark-worker:2.2.1  
比如我要在官网的基础上执行以下命令
    
    
    RUN yum install -y wget 
    RUN wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
    RUN yum clean all 
    RUN yum makecache 
    RUN yum update -y
    

当然一般的写法是最好写一个RUN.  
额外提一点：  
为什么一般建议只写一个RUN？  
因为在dockerfile里面，一个dockerfiel命令就会生成一个缓存容器(构建层数)。  
而docker在构建的过程中有最大缓存容器的限制(构建层数)，一般是48层(新版本不确定)。  
但是在我们这里，因为命令比较耗时，且因为网络波动等原因容易失败。  
所以不建议只写一个RUN,而是写多个RUN，这样能提高构建的成功率。而且失败了，能够从上次成功的地方继续，比较合理。  
上面的命令我们更新了语言包。  
接下来就是设置语言环境：  
ENV LANG=en_US.utf8

所以，完整的dockerfiel如下  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/be208bac522ae047d1b25b4c2fdfbd2c.png)

## 3.启动

spark的启动参数在docker-hub的use里介绍的很清楚  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f00d8ec9d90582bf2f4e50e82102b0b2.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e2b9ca41a304fb540113b9775d795439.png)

## 4.异常排查

启动失败一般有几种思路：  
1.网络不同  
2.端口映射  
3.防火墙  
4.ip异常  
等。