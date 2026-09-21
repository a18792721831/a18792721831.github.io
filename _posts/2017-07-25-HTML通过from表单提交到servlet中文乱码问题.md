---
layout: post
title: "HTML通过from表单提交到servlet中文乱码问题"
date: 2017-07-25 18:53:01 +0800
categories: [servlet, 表单, 乱码, 数据]
description: "本文介绍了解决HTML表单提交中文字符时出现乱码的方法。通过在Servlet中设置请求编码为UTF-8，可以有效避免中文乱码问题。"
keywords: servlet, 表单, 乱码, 数据
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/76094106
> - 发布时间：2017-07-25 18:53:01
> - 阅读量：4
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#servlet, #表单, #乱码, #数据

## 摘要

文章浏览阅读4.2k次，点赞2次，收藏2次。本文介绍了解决HTML表单提交中文字符时出现乱码的方法。通过在Servlet中设置请求编码为UTF-8，可以有效避免中文乱码问题。

---

在HTML文件中，通过from表单提交到servlet类中，可能会发生中文乱码问题：

比如输入一下信息：

![image](https://img-blog.csdn.net/20170725185603511?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

结果显示的内容：

![image](https://img-blog.csdn.net/20170725185731850?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

在servlet中进行调试：

发现在servlet中得到的数据就是乱码的数据：

![image](https://img-blog.csdn.net/20170725185835151?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

这个问题是因为在传值过程中，编码方式引起的问题，所以，在servlet中，获取数据之前，就因该设置HTML的编码方式，即在servlet中获取数据的语句之前添加：
    
    
    request.setCharacterEncoding("UTF-8");

  
重新启动服务器， 

进行测试：

![image](https://img-blog.csdn.net/20170725190130642?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

显示如下：

![image](https://img-blog.csdn.net/20170725190247176?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

完美解决HTML通过from表单提交的内容中文乱码问题！