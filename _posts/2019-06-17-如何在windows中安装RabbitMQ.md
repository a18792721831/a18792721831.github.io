---
layout: post
title: "如何在windows中安装RabbitMQ"
date: 2019-06-17 19:56:54 +0800
categories: [如何在windos中安装RabbitMQ, RabbitMQ和erlang, RabbitMQ的几种管理方式, 如何图形化管理RabbitMQ, 如何判断RabbitMQ是否安装成功]
description: "本文介绍在Windows中安装RabbitMQ的方法，包括下载文件，中途需安装erlang。还说明了验证安装成功的方式，如在计算机服务中查看、使用命令行工具。此外，讲解了开启可视化管理插件的步骤，以及处理端口占用问题的办法。"
keywords: 如何在windos中安装RabbitMQ, RabbitMQ和erlang, RabbitMQ的几种管理方式, 如何图形化管理RabbitMQ, 如何判断RabbitMQ是否安装成功
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/92693437
> - 发布时间：2019-06-17 19:56:54
> - 阅读量：455
> - 分类：RabbitMQ专栏收录该内容, 订阅专栏
> - 标签：#如何在windos中安装RabbitMQ, #RabbitMQ和erlang, #RabbitMQ的几种管理方式, #如何图形化管理RabbitMQ, #如何判断RabbitMQ是否安装成功

## 摘要

文章浏览阅读455次。本文介绍在Windows中安装RabbitMQ的方法，包括下载文件，中途需安装erlang。还说明了验证安装成功的方式，如在计算机服务中查看、使用命令行工具。此外，讲解了开启可视化管理插件的步骤，以及处理端口占用问题的办法。

---

#### 如何在windows中安装RabbitMQ

  * [1.下载文件](<#1_1>)
  * [2.验证](<#2_17>)
  * [3.开启可视化管理](<#3_33>)

## 1.下载文件

<https://www.rabbitmq.com>  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b59b1a346bce9e67751abd9625f8762d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/170971e9b4d16677ff0cbef35a85741c.png)  
下载可能速度较慢，可以使用下载工具下载。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f4e03acdf455d34704cbb53b8a814b46.png)  
双击安装。  
中途会要求安装erlang  
<https://www.erlang-solutions.com/resources/download.html>  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71fbc1ed874ea0d7e863098fb3136ad8.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fbbff5830206b3f3ec9679ce8796e917.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/928f271e776d382d65e5927605af786d.png)  
下载。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b494e540bbf9c117be41c269e18df60e.png)  
双击安装。  
然后在重新安装RabbitMQ

## 2.验证

RabbitMQ安装完成后有几个方式进行验证是否安装成功。

  1. 首先RabbitMQ是一个消息处理中心，所以就是服务端。服务端是服务，所以在计算机的服务中寻找。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d0cac26559c49555d4f65eaedf3cc62a.png)  
发现已启动，ok,安装成功。
  2. 在Windows任务管理器中查看服务：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5783a5c9c127029f2dffb3cbb1bb2500.png)
  3. RabbitMQ有命令行工具：  
使用cmd切换到RabbitMQ安装目录  
然后切换到sbin目录  
最后运行:rabbitmqctl status  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a6e1dc407707d41fef532c9a4c803ae0.png)  
当出现PID时，即可说明安装成功了  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/402d9ae9bad70521e60e1aa53eb5b381.png)  
此时你的应该与我的不同。  
因为你未开启可视化管理插件。

## 3.开启可视化管理

首先用浏览器打开localhost:15672发现无法访问。  
cmd切换到Rabbitmq的安装目录的sbin目录：  
然后执行  
rabbitmq-plugins enable rabbitmq_management  
cmd会提示你具体启用了哪些插件。  
然后重新访问localhost:15672  
（ps：有些可能存在端口占用的情况，导致启用失败：首先使用netstat -aon|find "15672"查询哪个进程在占用此端口，然后使用taskkill /f /pid 进程号 终止进程（一般不会冲突，也可以修改rabbitMQ的端口）重启MQ服务）  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0684acd86cacf89d904b78ebcf7ec9b1.png)  
声明一个队列（可以用可视化管理，客户端以及命令行进行）  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df1ccd0f89d6dea9ddbf777817ad224d.png)