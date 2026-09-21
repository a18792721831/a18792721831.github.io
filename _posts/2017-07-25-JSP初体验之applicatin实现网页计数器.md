---
layout: post
title: "JSP初体验之applicatin实现网页计数器"
date: 2017-07-25 20:04:43 +0800
categories: [体验, jsp, servlet, appliction]
description: "本文介绍了一个简单的JSP应用示例，通过使用application对象来记录网站访问者的数量，并展示了如何在每次页面刷新时更新计数器。"
keywords: 体验, jsp, servlet, appliction
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/76098045
> - 发布时间：2017-07-25 20:04:43
> - 阅读量：602
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#体验, #jsp, #servlet, #appliction

## 摘要

文章浏览阅读602次。本文介绍了一个简单的JSP应用示例，通过使用application对象来记录网站访问者的数量，并展示了如何在每次页面刷新时更新计数器。

---

首先，什么是application？

![image](https://img-blog.csdn.net/20170725200444580?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

![image](https://img-blog.csdn.net/20170725200523812?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

所以，我们直接在jsp文件中使用：
    
    
    <%
    if(application.getAttribute("counter") == null ){
    	application.setAttribute("counter", "1");
    }else{
    	String strnum = null;
    	strnum = application.getAttribute("counter").toString();
    	int icount = 0;
    	icount = Integer.valueOf(strnum).intValue();
    	icount++;
    	application.setAttribute("counter", Integer.toString(icount));
    }
     %>
     <font style="font-family: 楷体;font-size: 30px;color: blue;">您是第<%=application.getAttribute("counter") %>位访问者!</font>
    

  
接下来，我们在界面看看效果： 

![image](https://img-blog.csdn.net/20170725200803216?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

刷新下：

![image](https://img-blog.csdn.net/20170725200833237?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
  

  

OK，这个小例子就完成了！