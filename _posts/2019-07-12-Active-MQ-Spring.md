---
layout: post
title: ActiveMQ集成Spring
categories: [ActiveMQ, Spring]
description: ActiveMQ集成Spring
keywords: ActiveMQ, Spring, ActiveMQ集成到Spring, 如何简化访问ActiveMQ
---

 ActiveMQ集成Spring

# 1.java访问ActiveMQ的过程
生产
1.创建连接工厂
2.创建连接
3.开启连接
4.创建会话
5.创建主题
6.创建生产者
7.创建消息
8.发布
9.关闭资源
消费
6.创建消费者
7.注册监听
可以看到，对于很多的操作，对于生产者或者消费者来说，都是相同，甚至对于不用的生产者来说，也是相同的，
所以，将这些相同的处理逻辑，交给spring去管理，我们配置好相关的信息，然后将ActiveMQ的操作，
当做一个bean去操作。
这样可以在很大程度上简化使用ActiveMQ的流程。
# 2.ActiveMQ集成Spring
## 2.1新建一个gradle项目
![](/images/posts/2019-07-12-Active-MQ-Spring/1.jpeg)
## 2.2添加依赖
这里有一个小坑：
使用哪个仓库中心，就在哪个仓库中心搜索jar.
之前我使用的是阿里云的仓库中心，但是却在maven的仓库中心搜索的
版本号和名字，导致jar包一直无法下载。
为了防止这种问题，可以配置多个仓库中心。
![](/images/posts/2019-07-12-Active-MQ-Spring/3.png)
![](/images/posts/2019-07-12-Active-MQ-Spring/2.jpeg)