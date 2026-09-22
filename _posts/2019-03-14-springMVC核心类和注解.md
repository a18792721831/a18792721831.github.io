---
layout: post
title: "springMVC核心类和注解"
date: 2019-03-14 20:26:55 +0800
categories: ["springmvc注解", "@Controller注解@RequestMapping注解", "springMVC请求方式注解", "springmvc配置自动扫描注解", "springMVC请求的方法可返回的类型"]
description: "本文详细介绍了SpringMVC框架的核心组件，包括DispatcherServlet的作用及配置，Controller和RequestMapping注解的使用，以及请求处理流程和返回类型。通过实例展示了如何搭建SpringMVC项目。"
keywords: springmvc注解, @Controller注解@RequestMapping注解, springMVC请求方式注解, springmvc配置自动扫描注解, springMVC请求的方法可返回的类型
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88560582
> - 发布时间：2019-03-14 20:26:55
> - 阅读量：1
> - 分类：java同时被 3 个专栏收录, 订阅专栏, springMVC
> - 标签：#springmvc注解, #@Controller注解@RequestMapping注解, #springMVC请求方式注解, #springmvc配置自动扫描注解, #springMVC请求的方法可返回的类型

## 摘要

文章浏览阅读1.5k次，点赞2次，收藏7次。本文详细介绍了SpringMVC框架的核心组件，包括DispatcherServlet的作用及配置，Controller和RequestMapping注解的使用，以及请求处理流程和返回类型。通过实例展示了如何搭建SpringMVC项目。

---

#### springMVC核心类和注解

  * 1.DispatcherServlet
  * 2.Controller注解
  * 3.RequestMapping注解
  * 4.组合注解
  * 5.请求方法参数类型
  * 6.返回结果类型
  * 7.例子
  *     * 7.1新建一个Javaweb项目
    * 7.2导入jar包
    * 7.3写web.xml
    * 7.4写页面
    * 7.5写配置
    * 7.6写Controller
    * 7.7运行，使用postMan测试

## 1.DispatcherServlet

DispatcherServlet配置为：  
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
    
    

其中
    
    
    <servlet-mapping>
    		<servlet-name>springmvc</servlet-name>
    		<url-pattern>/</url-pattern>
    	</servlet-mapping>
    
    
    
    配置了前端控制器的过滤规则，/表示过滤所有的的请求。
    
    
    
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
    
    
    
    注册了一个前端控制器
    
    
    
    	<!-- 配置前端过滤器 -->
    		<servlet-name>springmvc</servlet-name>
    		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    

前端控制器引用的是springmvc这个过滤规则，同时注册这个前端控制器去寻找配置文件时的默认名字。  
springmvc-config.xml
    
    
    <!-- 初始化时加载配置文件 -->
    		<init-param>
    			<param-name>contextConfigLocation</param-name>
    			<param-value>classpath:springmvc-config.xml</param-value>
    		</init-param>
    		<!-- 容器启动时加载servlet -->
    		<load-on-startup>1</load-on-startup>
    

## 2.Controller注解

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9250ff4becc75388209b0d89863f2a9a.png)  
如何配置  
springmvc-config.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd
    	">
    	<!-- 指定需要扫描的包 -->
    	<context:component-scan base-package="controller"></context:component-scan>
    	<!-- 定义视图解析器 -->
    	<bean id="viewResolver" 
    		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    		<!-- 设定前缀 -->
    		<property name="prefix" value="/WEB-INF/jsp/"></property>
    		<!-- 设定后缀 -->
    		<property name="suffix" value=".jsp"></property>
    	</bean>
    </beans>
    

如何使用  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9041eff3c456a26dec547581ee7997aa.png)  
在类上增加注解

## 3.RequestMapping注解

spring通过Controller注解表示这是一个实现了目标方法的适配类，但是还需要映射器，把请求的url与具体的适配处理进行绑定。  
RequestMapping就是实现请求url与具体的适配方法进行绑定的映射器。  
如何使用  
RequestMapping可以用在类上，也可以用在方法上。  
用在类上表示调用该类型所有的方法都必须先请求配置的方法。  
举个例子：  
A类上注解的是/asd  
a方法上注解的是/nnn  
那么访问a方法的请求url就是  
/asd/nnn  
A类除了a方法还有b方法，配置的是/mmm  
那么访问b方法的请求url是  
/asd/mmm  
当然RequestMapping除了控制映射url之外，还可以进行以下处理：  
1.访问别名:name  
2.映射url:value  
3.请求方式:method  
4.参数:params  
5.请求头:headers  
6.请求内容类型:consumes  
7.返回内容类型:produces

## 4.组合注解

@GetMapping:Get请求；  
@PostMapping:Post请求；  
@PutMapping:Put请求;  
@DeleteMapping:Delete请求；  
@PatchMapping:Patch请求；

## 5.请求方法参数类型

## 6.返回结果类型

ModelAndView  
Model  
View  
String  
void  
HttpEntity  
Callable  
DeferedResult

## 7.例子

### 7.1新建一个Javaweb项目

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a5b8bbb1941e9cdf87a45d26b4a9e690.png)

### 7.2导入jar包

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1451d9968aa073fa977e313e942a31fb.png)

### 7.3写web.xml

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
    
    

### 7.4写页面

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3235b7361e3a8b15e7a76110dcfccdd7.png)  
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
    

### 7.5写配置

springmvc-config.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd
    	">
    	<!-- 指定需要扫描的包 -->
    	<context:component-scan base-package="controller"></context:component-scan>
    	<!-- 定义视图解析器 -->
    	<bean id="viewResolver" 
    		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    		<!-- 设定前缀 -->
    		<property name="prefix" value="/WEB-INF/jsp/"></property>
    		<!-- 设定后缀 -->
    		<property name="suffix" value=".jsp"></property>
    	</bean>
    </beans>
    

### 7.6写Controller
    
    
    package controller;
    
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    
    @Controller
    @RequestMapping("/hello")
    public class FirstController{
    
    	@RequestMapping("/first")
    	public String handleRequest(HttpServletRequest httpServletRequest,
    			HttpServletResponse httpServletResponse,Model model){
    		model.addAttribute("message", "springmvc_comment");
    		return "first";
    	}
    	
    	@RequestMapping("/second")
    	public String second(Model model){
    		model.addAttribute("message", "second");
    		return "first";
    	}
    	
    }
    
    

### 7.7运行，使用postMan测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2d62d747f1fb8595b268fd8b41bf9309.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9a394c018dee91fc972e4b9a1f0b88fc.png)
