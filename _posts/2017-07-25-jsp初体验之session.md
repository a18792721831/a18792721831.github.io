---
layout: post
title: "jsp初体验之session"
date: 2017-07-25 18:14:24 +0800
categories: [session, cookie, jsp, 体验, 密码]
description: "本文通过四个JSP页面的示例，逐步介绍了如何使用Session对象在Web应用程序中存储和检索用户信息。"
keywords: session, cookie, jsp, 体验, 密码
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/76092600
> - 发布时间：2017-07-25 18:14:24
> - 阅读量：600
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#session, #cookie, #jsp, #体验, #密码

## 摘要

文章浏览阅读600次。本文通过四个JSP页面的示例，逐步介绍了如何使用Session对象在Web应用程序中存储和检索用户信息。

---

这个项目主要就是体验session对象，所以，首先，来看看session对象是什么？

![image](https://img-blog.csdn.net/20170725181904401?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

有人可能会问，cookie和session的区别是什么？

![image](https://img-blog.csdn.net/20170725182038108?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

基于这些原因，所以做了一个小例子，体验一下session对象：

![image](https://img-blog.csdn.net/20170725182210106?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

session对象的概要：

![image](https://img-blog.csdn.net/20170725182245652?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

首先，我们在第一个页面获取用户名和密码，

在第二个页面读取用户名和密码，将其显示在页面上，并进行session保存，并且继续获取性别和年龄，

在第三个页面读取性别和年龄，将其显示在页面上，并进行session中的用户名密码读取，并且存入性别和年龄信息，

在第四个页面，直接进行session读取，并且输出：

首先，我们来写第一个jsp文件：Login.jsp，位于JSP文件夹下：
    
    
    <%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
    <%
    String path = request.getContextPath();
    String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
    %>
    
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
    <html>
      <head>
        <base href="<%=basePath%>">
        
        <title>Loginjsp page</title>
        
    	<meta http-equiv="pragma" content="no-cache">
    	<meta http-equiv="cache-control" content="no-cache">
    	<meta http-equiv="expires" content="0">    
    	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    	<meta http-equiv="description" content="This is my page">
    	<!--
    	<link rel="stylesheet" type="text/css" href="styles.css">
    	-->
    
      </head>
      
      <body>
        <form action="JSP/second.jsp">
        	<table>
        		<tr>
        			<td align="center">
        				<b style="font-family: 楷体;font-size: 25px;color: green;">用户名:</b>
        			</td>
        			<td>
        				<input type="text" name = "user">
        			</td>
        		</tr>
        		<tr align="center">
        			<td>
        				<b style="font-family: 楷体;font-size: 25px;color: red;">密  码:</b>
        			</td>
        			<td>
        				<input type="password" name = "pswd">
        			</td>
        		</tr>
        		<tr>
        			<td>
        				<input type="reset" value = "重置">
        			</td>
        			<td>
        				<input type="submit" value="提交">
        			</td>
        		</tr>
        	</table>
        </form>
      </body>
    </html>
    

  
第二个jsp文件：second.jsp，位于JSP文件夹下： 
    
    
    <%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
    <%
    String path = request.getContextPath();
    String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
    %>
    
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
    <html>
      <head>
        <base href="<%=basePath%>">
        
        <title>secondjsp page</title>
        
    	<meta http-equiv="pragma" content="no-cache">
    	<meta http-equiv="cache-control" content="no-cache">
    	<meta http-equiv="expires" content="0">    
    	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    	<meta http-equiv="description" content="This is my page">
    	<!--
    	<link rel="stylesheet" type="text/css" href="styles.css">
    	-->
    
      </head>
      
      <body>
        客户信息：<br>
        <%
        String nmString = request.getParameter("user");
        String pdString = request.getParameter("pswd");
        String nameString = new String(nmString.getBytes("ISO-8859-1"),"UTF-8");
        String pswdString = new String(pdString.getBytes("ISO-8859-1"),"UTF-8");
        session.putValue("name", nameString);
        session.putValue("pswd", pswdString);
         %>
         名字：<%=nameString %><br>
         密码：<%=pswdString %><br>
         继续输入：<br>
         <form action="JSP/threed.jsp">
         	<table>
         		<tr>
         			<td align="center">
         				<b style="font-family: 楷体;font-size: 25px;color: green;">性别:</b>
         			</td>
         			<td align="center">
         				<input type="text" name = "sex">
         			</td>
         		</tr>
         		<tr>
         			<td align="center">
         				<b style="font-family: 楷体;font-size: 25px;color: green;">年龄:</b>
         			</td>
         			<td align="center">
         				<input type="text" name = "age">
         			</td>
         		</tr>
         		<tr>
         			<td align="center">
         				<input type="reset" value="重置">
         			</td>
         			<td align="center">
         				<input type="submit" value="提交">
         			</td>
         		</tr>
         	</table>
         </form>
      </body>
    </html>

第三个页面：threed.jsp，位于JSP文件夹下： 
    
    
    <%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
    <%
    String path = request.getContextPath();
    String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
    %>
    
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
    <html>
      <head>
        <base href="<%=basePath%>">
        
        <title>threedjsp page</title>
        
    	<meta http-equiv="pragma" content="no-cache">
    	<meta http-equiv="cache-control" content="no-cache">
    	<meta http-equiv="expires" content="0">    
    	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    	<meta http-equiv="description" content="This is my page">
    	<!--
    	<link rel="stylesheet" type="text/css" href="styles.css">
    	-->
    
      </head>
      
      <body>
        
        <%
        String sString = request.getParameter("sex");
        String aString = request.getParameter("age");
        String sexString = new String(sString.getBytes("ISO-8859-1"),"UTF-8");
        String ageString = new String(aString.getBytes("ISO-8859-1"),"UTF-8");
        session.putValue("sex", sexString);
        session.putValue("age", ageString);
        String nameString = (String)session.getValue("name");
        String pswdString = (String)session.getValue("pswd");
         %>
         客户信息：<br>
         用户名：<%=nameString %><br>
         密码：<%=pswdString %><br>
         性别：<%=sexString %><br>
         年龄：<%=ageString %><br>
         <a href="http://localhost:8080/htmlday1/JSP/four.jsp">跳转到第4个页面</a>
      </body>
    </html>

第四个页面，four.jsp,位于JSP文件夹下： 
    
    
    <%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
    <%
    String path = request.getContextPath();
    String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
    %>
    
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
    <html>
      <head>
        <base href="<%=basePath%>">
        
        <title>fourjsp page</title>
        
    	<meta http-equiv="pragma" content="no-cache">
    	<meta http-equiv="cache-control" content="no-cache">
    	<meta http-equiv="expires" content="0">    
    	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    	<meta http-equiv="description" content="This is my page">
    	<!--
    	<link rel="stylesheet" type="text/css" href="styles.css">
    	-->
    
      </head>
      
      <body>
      <%
      	String nameString = (String)session.getValue("name");
      	String pswdString = (String)session.getValue("pswd");
      	String sexString = (String)session.getValue("sex");
      	String ageString = (String)session.getValue("age");
       %>
        客户信息：<br>
        姓名：<%=nameString %><br>
        密码：<%=pswdString %><br>
        性别：<%=sexString %><br>
        年龄：<%=ageString %>
      </body>
    </html>
    

  
  

OK，来看效果：

![image](https://img-blog.csdn.net/20170725183010781?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

![image](https://img-blog.csdn.net/20170725183041933?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

![image](https://img-blog.csdn.net/20170725183108645?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

![image](https://img-blog.csdn.net/20170725183135287?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

![image](https://img-blog.csdn.net/20170725183158724?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

OK，现在，不关闭浏览器，跳转到其他界面，然后输入第四个页面的url地址：  

![image](https://img-blog.csdn.net/20170725183542469?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

OK，现在，关闭浏览器，重新启动浏览器，输入第四个页面的url地址：

  

![image](https://img-blog.csdn.net/20170725183409844?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

发现浏览器如果没有断开与服务器的连接，那么，session就会保存数据，但是如果浏览器与服务器断开连接，那么，session会被销毁并重新定义。

根据这个情况，可以说明，session是保存在服务器上的一个用来保存数据的对象；

当客户端与服务器断开链接，那么，服务器会销毁session对象；

当客户端与服务器端连接成功，不管会不会使用到session对象，服务器都会创建这个对象；

所以，session是一个存储于服务器端的对象，但是安全，因为不保存在本地，所以，用户不能接触到session，无法分析session的数据，而且，session不能永久保存，

因为没有自动的序列化，所以，session保存数据有生命周期。

session的生命周期以浏览器链接到服务器端开始，以浏览器断开与服务器的连接结束。

![image](https://img-blog.csdn.net/20170725184441318?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)