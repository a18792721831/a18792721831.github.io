---
layout: post
title: "Java使用jdbc链接数据库的MySQL 5.5.45+以及SSL错误解决办法"
date: 2017-08-01 10:37:51 +0800
categories: [java, mysql, 数据库, ssl, url]
description: "本文介绍了在使用Java JDBC连接数据库时遇到SSL相关错误的解决方案。通过在URL中添加特定参数useSSL=true来启用安全连接。"
keywords: java, mysql, 数据库, ssl, url
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/76508021
> - 发布时间：2017-08-01 10:37:51
> - 阅读量：2
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#java, #mysql, #数据库, #ssl, #url

## 摘要

文章浏览阅读2.1k次。本文介绍了在使用Java JDBC连接数据库时遇到SSL相关错误的解决方案。通过在URL中添加特定参数useSSL=true来启用安全连接。

---

在Java中使用jdbc连接数据库时，有时会出现以下错误：   
![这里写图片描述](https://img-blog.csdn.net/20170801103422430?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
和：   
![这里写图片描述](https://img-blog.csdn.net/20170801103442071?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
以及：   
![这里写图片描述](https://img-blog.csdn.net/20170801103504117?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
修改方法：在写好的链接url后面加入：   
![这里写图片描述](https://img-blog.csdn.net/20170801103544656?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
首先，问号表示url后面传递的参数，useSSL = true表示使用安全链接。