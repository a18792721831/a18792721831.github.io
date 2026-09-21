---
layout: post
title: "idea中java swing项目打开无法加载类"
date: 2022-01-07 20:49:09 +0800
categories: [intellij-idea, java, intellij idea]
description: "当使用Idea开发Javaswing项目并遇到类无法识别的问题时，可能是由于form文件导致IDE解析异常。解决步骤包括：移除form文件，清理缓存，重启Idea，再将form文件放回原位置。"
keywords: intellij-idea, java, intellij idea
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/122372134
> - 发布时间：2022-01-07 20:49:09
> - 阅读量：1
> - 分类：Idea专栏收录该内容, 订阅专栏
> - 标签：#intellij-idea, #java, #intellij idea

## 摘要

文章浏览阅读1.2k次。当使用Idea开发Javaswing项目并遇到类无法识别的问题时，可能是由于form文件导致IDE解析异常。解决步骤包括：移除form文件，清理缓存，重启Idea，再将form文件放回原位置。

---

idea中java swing项目打开无法加载类

如果你使用Idea开发Java swing项目，而且使用了可视化界面开发，那么当你再次打开项目，就会发现Java类无法识别了，到处报红。

![image-20220107204136253](https://i-blog.csdnimg.cn/blog_migrate/0f2f91e719d6e605148ad64da8385483.png)

问题原因：个人猜测是可视化界面开发的`form`文件导致ide解析java类异常。

解决方案：先把`form`文件从项目中剪切出来，然后清除缓存，重启idea刷新整个项目，接着在把`form`文件拷贝回原来的位置。

![image-20220107204635038](https://i-blog.csdnimg.cn/blog_migrate/7f8db74a96bae22d12dc22ce9a80bc0f.png)