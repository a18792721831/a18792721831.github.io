---
layout: post
title: "spring boot整合Redis"
date: 2020-02-19 19:56:43 +0800
categories: [boot集成redis, Redis docker, redis常用, redis boot配置, redis dao测试与编写]
description: "本文详细介绍如何使用SpringBoot整合Redis，包括从Redis介绍、Docker启动Redis服务、创建SpringBoot项目、配置Redis、创建DAO层、编写测试用例到最后的测试验证过程。通过本文，读者可以快速掌握SpringBoot中Redis的集成与应用。"
keywords: boot集成redis, Redis docker, redis常用, redis boot配置, redis dao测试与编写
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104399126
> - 发布时间：2020-02-19 19:56:43
> - 阅读量：519
> - 分类：微服务同时被 3 个专栏收录, 订阅专栏, spring boot, Redis
> - 标签：#boot集成redis, #Redis docker, #redis常用, #redis boot配置, #redis dao测试与编写

## 摘要

文章浏览阅读519次。本文详细介绍如何使用SpringBoot整合Redis，包括从Redis介绍、Docker启动Redis服务、创建SpringBoot项目、配置Redis、创建DAO层、编写测试用例到最后的测试验证过程。通过本文，读者可以快速掌握SpringBoot中Redis的集成与应用。

---

#### spring boot整合Redis

  * 1\. Redis简介
  * 2\. docker 启动
  * 3\. 创建项目
  * 4\. 创建配置
  * 5\. 创建Dao
  * 6\. 创建Test
  * 7\. 测试
  * 8\. 验证

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. Redis简介

Redis 是一个开源的、先进的 key-value 存储系统，可用于构建高性能的存储系统。Redis  
支持数据结构有字符串、哈希、列表、集合、排序集合、位图、超文本等。NoSQL（Not Only SQL）泛指非关系型的数据库。Redis 是一种NoSQL，Redis 具有很多的优点，例如读写非常快速，支持丰富的数据类型，所有的操作都是原子的。

## 2\. docker 启动

使用命令下载docker镜像  
docker pull docker.io/redis  
然后启动  
docker run -d -P redis

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/974a260d89e4a49a1a229c69d713600d.png)

## 3\. 创建项目

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/798a24fed527246f609632e075f65e05.png)

## 4\. 创建配置

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/134b7391db1163f7654773b92dbb2280.png)

## 5\. 创建Dao

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0d7c1d0ea704bec5e5e19ce2174d3003.png)

## 6\. 创建Test

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7dd05e37199fa5becedca4f9b0cf06b0.png)

## 7\. 测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9c4065fddb933f46eb4b0c00dc8eca0c.png)

## 8\. 验证

使用  
docker exec -it boring_joliot /bin./bash  
进入容器  
然后使用 redis-cli进入redis命令行  
然后使用select 1切换到1数据库  
最后使用keys * 查看所有键值对  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bc4b75e47ff67a0fcffec18d679423af.png)
