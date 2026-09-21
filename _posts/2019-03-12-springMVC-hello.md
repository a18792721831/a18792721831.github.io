---
layout: post
title: "springMVC-hello"
date: 2019-03-12 21:12:29 +0800
categories: [springMVC入门, Java-web请求的详细过程, 如何创建一个JavaWeb项目, JavaWeb入门, tomcat容器中发布sprigmvc的Javaweb项目]
description: "本文详细介绍SpringMVC框架的特点，包括其与Spring框架的无缝集成、支持国际化、多种视图技术及自动绑定用户输入等功能。通过实例演示如何创建项目、配置前端控制器、映射处理器并实现数据展示。"
keywords: springMVC入门, Java-web请求的详细过程, 如何创建一个JavaWeb项目, JavaWeb入门, tomcat容器中发布sprigmvc的Javaweb项目
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88429389
> - 发布时间：2019-03-12 21:12:29
> - 阅读量：397
> - 分类：java同时被 3 个专栏收录, 订阅专栏, springMVC
> - 标签：#springMVC入门, #Java-web请求的详细过程, #如何创建一个JavaWeb项目, #JavaWeb入门, #tomcat容器中发布sprigmvc的Javaweb项目

## 摘要

文章浏览阅读397次。本文详细介绍SpringMVC框架的特点，包括其与Spring框架的无缝集成、支持国际化、多种视图技术及自动绑定用户输入等功能。通过实例演示如何创建项目、配置前端控制器、映射处理器并实现数据展示。

---

#### springMVC入门

  * [1.springmvc特点](<#1springmvc_1>)
  * [2.spring核心点](<#2spring_10>)
  *     * [1.前端控制器DispatcherServlet](<#1DispatcherServlet_11>)
    * [2.支持国际化](<#2_33>)
    * [3.多种视图技术](<#3_35>)
    * [4.基于xml配置以及注解配置](<#4xml_37>)
    * [5.自动绑定用户](<#5_41>)
  * [2.例子](<#2_43>)
  *     * [1.创建一个Javaweb项目](<#1Javaweb_44>)
    * [2.导入tomcat容器](<#2tomcat_47>)
    * [3.导入jar包](<#3jar_54>)
    * [4.创建web.xml文件](<#4webxml_56>)
    * [5.创建前端控制器配置](<#5_83>)
    * [6.创建映射器目标类--controller](<#6controller_104>)
    * [7.创建视图](<#7_138>)
    * [8.添加到容器](<#8_156>)
    * [9.启动容器](<#9_158>)
    * [10.访问结果](<#10_169>)
  * [3.总结](<#3_171>)

## 1.springmvc特点

1.springmvc是web项目；  
2.与spring无缝集成；  
3.因为springmvc基于spring,所以springmvc继承了spring的与其他框架容易结合的特点；  
4.提供了一个前端控制器DispatcherServlet.这个控制器做了许多的事情，后面会说明；  
5.支持国际化；  
6.支持多种视图技术；  
7.使用基于xml的配置文件（也可以基于注解配置）；  
8.自动绑定用户输入，正确的转换数据类型。

## 2.spring核心点

### 1.前端控制器DispatcherServlet

想要知道前端控制器做了哪些事情，就需要知道一个Javaweb中发起一个请求到得到一个回应需要经历哪些过程。  
①用户发起请求；  
②前端控制拦截到请求；  
③前端控制器调用映射器；  
④映射器返回映射目标类；  
⑤前端控制器调用处理适配器；  
⑥处理适配器调用目标类实现的处理器；  
⑦目标类实现的处理器返回ModeAndView对象；  
⑧处理适配器返回ModeAndView对象给前端控制器；  
⑨前端控制器调用视图解析器解析ModeAndView对象；  
⑩视图解析器解析ModeAndView对象返回具体的视图；  
11前端控制器渲染视图（数据填充工作）；  
12渲染后的视图返回给用户。

前端控制器主要做以下几件事：  
1.拦截用户请求；  
2.调用映射器；  
3.调用处理适配器；  
4.调用视图解析器；  
5.渲染视图；  
6.返回渲染后的视图。

### 2.支持国际化

国际化就是前端控制器在渲染视图填充数据时，有选择的填充，这样就完成了国际化，只需要准备不同的语言文件即可。

### 3.多种视图技术

因为springmvc基于spring开发，所以只需要指定其他的视图解析器就会做到支持其他的视图技术。

### 4.基于xml配置以及注解配置

还是同样的原因，因为spring就有两种配置方式：  
xml和注解。  
所以springmvc也有两种配置方式。

### 5.自动绑定用户

spring可以自动的寻找bean：使用反射处理bean。

## 2.例子

### 1.创建一个Javaweb项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ae31373f1540a80efc247ffdcc0ae5d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/97dd6f29460ae885b00875b3233d37e3.png)

### 2.导入tomcat容器

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0269a8299141e19aa18dd15e2bef4f19.png)  
如果没有server，就去Other中搜索。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/92343918a617c00f8ed9f9925c8fa902.png)  
在这个窗口中添加。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d47c69d4ad6881a2f2048dfdf2c9246.png)  
添加后。

### 3.导入jar包

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/537b01ffb91aeec67a865bc78f5e113c.png)

### 4.创建web.xml文件

web.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://java.sun.com/xml/ns/javaee"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    	http://java.sun.com/xml/ns/javaee/web-app.xsd">
    	<servlet>
    		<!-- 配置前端过滤器 -->
    		<servlet-name>springmvc</servlet-name>
    		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    		<!-- 初始化时加载配置文件 -->
    		<init-param>
    			<param-name>contextConfigLocation</param-name>
    			<param-value>classpath:springmvc-config.xml</param-value>
    		</init-param>
    		<!-- 容器启动时加载servlet -->
    		<load-on-startup>1</load-on-startup>
    	</servlet>
    	<servlet-mapping>
    		<servlet-name>springmvc</servlet-name>
    		<url-pattern>/</url-pattern>
    	</servlet-mapping>
    </web-app>
    
    

### 5.创建前端控制器配置

springmvc-config.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<!-- 配置处理器，映射/hello -->
    	<bean id="/hello" class="controller.FirstController">
    	</bean>
    	<!-- 配置映射器,url和class进行映射 -->
    	<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
    	</bean>
    	<!-- 配置处理器，调用controller中实现的方法(使用了适配器模式) -->
    	<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>
    	<!-- 解析视图(mav) -->
    	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"></bean>
    </beans>
    

只配置第一个也可以。

### 6.创建映射器目标类–controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7db31365b4cf6368138046c22fb2148.png)
    
    
    package controller;
    
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    import org.springframework.web.servlet.ModelAndView;
    import org.springframework.web.servlet.mvc.Controller;
    
    
    
    public class FirstController implements Controller{
    
    	@Override
    	public ModelAndView handleRequest(HttpServletRequest arg0,
    			HttpServletResponse arg1) throws Exception {
    		//1.创建mav对象
    		ModelAndView mav = new ModelAndView();
    		//2.mav添加数据
    		mav.addObject("message", "hello springMVC");
    		//3.设置mav访问路径
    		mav.setViewName("/WEB-INF/jsp/first.jsp");
    		//4.返回mav对象
    		return mav;
    	}
    
    
    }
    
    

其中handleRequest方法就是目标类具体的处理器。  
ModeAndView不仅仅可以添加数据，也可以设置View

### 7.创建视图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf38e057e02873034f234945f89d9fc1.png)  
first.jsp
    
    
    <%@ page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="utf-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Hello Spring MVC</title>
    </head>
    <body>
    	${message}
    </body>
    </html>
    

其中${message}是jsp中的el表达式。

### 8.添加到容器

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a75e7bd12e70f8b38ceb7156cdae738.png)

### 9.启动容器

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b015117964a00a1e0e8e2f86a115c182.png)  
没有异常且看到Server startup in 1929ms即说明容器启动正常，此时访问我们配置的url  
就是在前端控制器中配置的url：  
<http://localhost:8080/springmvc_hello_/hello>  
说明：  
http表示访问方式  
localhost表示目标主机  
8080表示访问的端口  
springmvc_hello_表示访问的项目  
hello表示访问的请求

### 10.访问结果

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/91b8c81fe2d54fa7a1625d7d2e9eb8e1.png)

## 3.总结

学习springmvc可以深刻的了解Javaweb中请求的过程。  
更深入的理解为什么，而不是怎么做。