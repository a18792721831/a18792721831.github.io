---
layout: post
title: "Command line is too long."
date: 2020-03-26 20:07:00 +0800
categories: ["2020"]
description: "本文详细介绍了在使用IDEA运行项目时遇到的错误：命令行过长。通过修改.idea文件夹下的workspace.xml文件，增加一行代码，即可解决该问题，确保项目的正常运行。"
keywords: 2020
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/105126686
> - 发布时间：2020-03-26 20:07:00
> - 阅读量：269
> - 分类：java同时被 3 个专栏收录, 订阅专栏, 微服务, Idea

## 摘要

文章浏览阅读269次。本文详细介绍了在使用IDEA运行项目时遇到的错误：命令行过长。通过修改.idea文件夹下的workspace.xml文件，增加一行代码，即可解决该问题，确保项目的正常运行。

---

idea运行报错：
    
    
    下午 8:00	Error running 'ServiceAdminApplication': Command line is too long. Shorten command line for ServiceAdminApplication or also for Spring Boot default configuration.
    

解决方式:  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/60c0c4cc5dd1b3890ee22dcac54b6bbc.png)  
在`.idea`文件夹下的`workspace.xml`中，找到 `PropertiesComponent`的`component`，增加一行`<property name="dynamic.classpath" value="true" />`