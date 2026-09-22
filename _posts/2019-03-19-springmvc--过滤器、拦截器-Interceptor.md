---
layout: post
title: "springmvc--过滤器、拦截器:Interceptor"
date: 2019-03-19 21:35:21 +0800
categories: [springMVC-拦截器-Interceptor, spring mvc拦截器&amp;拦截链, springmvc实现请求的AOP, springmvc拦截器的配置与定义, springMVC拦截器实现aop业务控制]
description: "本文围绕SpringMVC拦截器展开，介绍了编写拦截器的方式，如实现HandlerInterceptor或WebRequestInterceptor接口等。阐述了拦截器的配置方法及注意事项，分析了拦截器的执行顺序，包括单个和多个拦截器的情况。最后通过实例展示了创建SpringMVC项目使用拦截器的过程，并总结可采用责任链模式设计拦截器链。"
keywords: springMVC-拦截器-Interceptor, spring mvc拦截器&amp;拦截链, springmvc实现请求的AOP, springmvc拦截器的配置与定义, springMVC拦截器实现aop业务控制
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88674234
> - 发布时间：2019-03-19 21:35:21
> - 阅读量：867
> - 分类：java同时被 3 个专栏收录, 订阅专栏, springMVC
> - 标签：#springMVC-拦截器-Interceptor, #spring mvc拦截器&amp;拦截链, #springmvc实现请求的AOP, #springmvc拦截器的配置与定义, #springMVC拦截器实现aop业务控制

## 摘要

文章浏览阅读867次。本文围绕SpringMVC拦截器展开，介绍了编写拦截器的方式，如实现HandlerInterceptor或WebRequestInterceptor接口等。阐述了拦截器的配置方法及注意事项，分析了拦截器的执行顺序，包括单个和多个拦截器的情况。最后通过实例展示了创建SpringMVC项目使用拦截器的过程，并总结可采用责任链模式设计拦截器链。

---

#### springmvc--过滤器、拦截器:Interceptor

  * 1.如何写一个拦截器
  * 2.配置拦截器
  * 3.拦截器的执行顺序
  * 4.多个拦截器配置后执行的顺序
  * 5.例子
  *     * 5.1创建一个springmvc项目
    * 5.2导入jar包
    * 5.3配置web.xml
    * 5.4创建实体
    * 5.5创建自定义的拦截器
    * 5.6配置拦截器
    * 5.7创建controller
    * 5.8导入js
    * 5.9编写jsp
    * 5.10测试
  * 6.总结

## 1.如何写一个拦截器

  * 1.实现HandlerInterceptor接口
  * 2.继承HandlerInterceptor的实现类
  * 3.实现WebRequestInterceptor接口
  * 4.继承WebRequestInterceptor实现类

HandlerInterceptor接口的方法：  
default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)  
throws Exception {
    
    
    	return true;
    }
    

default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,  
@Nullable ModelAndView modelAndView) throws Exception {  
}  
default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,  
@Nullable Exception ex) throws Exception {  
}  
default是Java8的一个新增特性，接口定义的default方法就相当于接口提供了方法的默认实现，而且所有实现接口或者实现接口继承接口的方法都自动拥有了默认的方法。  
当然一般希望被覆盖。

preHandler方法在前端控制器方法前执行，返回值控制是否执行前端控制器以及之后的方法。  
postHandler方法在前端控制器方法之后，视图解析之前执行。  
afterCompletion方法会在整个请求完成，视图渲染结束之后调用。

疑问：  
postHandler方法和controller这个处理器的实现方法哪个先执行？  
我们知道前端控制器拦截到所有的请求后会调用映射器获取到与映射器匹配的处理适配器，然后调用处理适配器中的目标方法，最后调用视图解析器进行解析。

那么在视图解析器之前，前端控制器之后有好几个控制点可以调用postHandler方法，那么postHandler和适配器目标方法谁先谁后？  
（留个疑问，后面实例中解析）

## 2.配置拦截器

拦截器定义后还需要配置才能生效。  
配置:
    
    
    <!-- 配置拦截器 -->
    	<mvc:interceptors>
    		<!-- 使用bean直接定义(全局拦截) -->
    		<bean class="interceptor.MyInterceptor1">
    		</bean>
    		<!-- 拦截器1 -->
    		<mvc:interceptor>
    			<!-- 目标路径 -->
    			<mvc:mapping path="/**"/>
    			<!-- 释放路径 -->
    			<mvc:exclude-mapping path=""/>
    			<!-- 局部拦截(全局拦截，目标路径匹配所有的路径) -->
    			<bean class="interceptor.MyInterceptor2"></bean>
    		</mvc:interceptor>
    		<!-- 拦截器2 -->
    		<mvc:interceptor>
    			<!-- 目标路径(以hello结尾的拦截) -->
    			<mvc:mapping path="/*/hello"/>
    			<!-- 释放路径(以ex结尾的不拦截) -->
    			<mvc:exclude-mapping path="/*/ex"/>
    			<!-- 局部拦截 -->
    			<bean class="interceptor.MyInterceptor3"></bean>
    		</mvc:interceptor>
    		<!-- 拦截器3 -->
    		<mvc:interceptor>
    			<mvc:mapping path="/*/test"/>
    			<bean class="interceptor.MyInterceptor4"></bean>
    		</mvc:interceptor>
    		<mvc:interceptor>
    			<mvc:mapping path="/user/*"/>
    			<bean class="interceptor.LoginInterceptor"></bean>
    		</mvc:interceptor>
    	</mvc:interceptors>
    

注意：  
全局拦截器可以配多个。  
路径的匹配规则：
    
    
    /*匹配/+后面至下一个/之间的字符
    /**匹配/后面所有的字符
    eg:
    匹配:/test/people
    /**
    /test/*
    
    /people
    匹配项目路径+/people
    匹配/people结尾
    /**/people
    

局部拦截器的配置顺序：  
mvc:mapping  
mvc:exclude-mapping(可以缺失不写)  
bean  
必须以上面的先后顺序配置。

## 3.拦截器的执行顺序

还记得之前的疑问吗？  
postHandler和controler中的方法哪个先执行？  
答案是controller中的方法先执行。  
因为preHandler方法在controller前执行，所以在controller方法前就没有其他方法的必要了，放在preHandler方法中就好了。  
在controller方法执行之后，就会执行posthandler方法然后执行视图解析器，然后向客户端发送返回数据，最后执行afterCompletion  
为什么postHandler和afterCompletion方法不能放在一起?  
因为当controller方法执行完毕后，还会执行视图解析，以及发送返回数据，这两个操作都是会出现异常的情况，所以postHandler是在controller方法后，试图解析前对视图可进行操作的方法。  
而当返回数据发送成功后，认为整个请求的流程全部走完，此时本次请求所占用的资源已经可以释放（作用范围是本次请求的资源）。

## 4.多个拦截器配置后执行的顺序

多个拦截器的执行顺序是堆栈的执行过程：  
假设有A，B两个拦截器，那么执行的顺序是这样的：  
A.preHandler->B.preHandler->controller->B.postHandler->A.postHandler->send Data(返回数据给客户端)->B.afterCompletion->A.afterCompletion  
记忆技巧：  
ABBABA  
最外层的永远在外层，先配置的后执行。

## 5.例子

### 5.1创建一个springmvc项目

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/43371e455be7517dc303780b501a97db.png)

### 5.2导入jar包

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/da63324aca53035af8489c01e96e5a2b.png)

### 5.3配置web.xml

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
    
    

### 5.4创建实体

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ddcc48b9fc0a47ffb898b1230b7c0d4d.png)
    
    
    package domain;
    
    import java.io.Serializable;
    
    public class People implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = -1364237643930318187L;
    
    	private Long id;
    
    	private String username;
    	
    	private String password;
    	
    	public Long getId() {
    		return id;
    	}
    
    	public void setId(Long id) {
    		this.id = id;
    	}
    
    	public String getUsername() {
    		return username;
    	}
    
    	public void setUsername(String username) {
    		this.username = username;
    	}
    
    	public String getPassword() {
    		return password;
    	}
    
    	public void setPassword(String password) {
    		this.password = password;
    	}
    	
    }
    
    
    

### 5.5创建自定义的拦截器

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0a39b852f9dcf26efc98e74ba2feda7e.png)
    
    
    package interceptor;
    
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import javax.servlet.http.HttpSession;
    
    import org.springframework.web.servlet.HandlerInterceptor;
    import org.springframework.web.servlet.ModelAndView;
    
    import domain.People;
    
    public class LoginInterceptor implements HandlerInterceptor{
    
    	@Override
    	public boolean preHandle(HttpServletRequest request,
    			HttpServletResponse response, Object handler) throws Exception {
    		String url = request.getRequestURI();
    		if(url.indexOf("/login") >= 0){
    			return true;
    		}
    		HttpSession session = request.getSession();
    		People people = (People) session.getAttribute("people");
    		if(people != null){
    			return true;
    		} 
    		request.setAttribute("message", "not login,please login");
    		request.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(request, response);
    		return false;
    	}
    
    	@Override
    	public void postHandle(HttpServletRequest request,
    			HttpServletResponse response, Object handler,
    			ModelAndView modelAndView) throws Exception {
    		System.out.println(this.getClass().getName()+"postHandle");
    	}
    
    	@Override
    	public void afterCompletion(HttpServletRequest request,
    			HttpServletResponse response, Object handler, Exception ex)
    			throws Exception {
    		System.out.println(this.getClass().getName()+"afterCompletion");
    	}
    
    	
    	
    }
    
    
    
    
    package interceptor;
    
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    import org.springframework.web.servlet.HandlerInterceptor;
    import org.springframework.web.servlet.ModelAndView;
    
    public class MyInterceptor1 implements HandlerInterceptor{
    
    	@Override
    	public boolean preHandle(HttpServletRequest request,
    			HttpServletResponse response, Object handler) throws Exception {
    		System.out.println(this.getClass().getName()+"preHandle");
    		request.setCharacterEncoding("utf-8");
    		return true;
    	}
    
    	@Override
    	public void postHandle(HttpServletRequest request,
    			HttpServletResponse response, Object handler,
    			ModelAndView modelAndView) throws Exception {
    		System.out.println(this.getClass().getName()+"postHandle");
    	}
    
    	@Override
    	public void afterCompletion(HttpServletRequest request,
    			HttpServletResponse response, Object handler, Exception ex)
    			throws Exception {
    		System.out.println(this.getClass().getName()+"afterCompletion");
    	}
    
    }
    
    
    
    
    package interceptor;
    
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    import org.springframework.web.servlet.ModelAndView;
    import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
    
    public class MyInterceptor2 extends HandlerInterceptorAdapter{
    
    	@Override
    	public boolean preHandle(HttpServletRequest request,
    			HttpServletResponse response, Object handler) throws Exception {
    		System.out.println(this.getClass().getName()+"preHandle");
    		return true;
    	}
    
    	@Override
    	public void postHandle(HttpServletRequest request,
    			HttpServletResponse response, Object handler,
    			ModelAndView modelAndView) throws Exception {
    		System.out.println(this.getClass().getName()+"postHandle");
    	}
    
    	@Override
    	public void afterCompletion(HttpServletRequest request,
    			HttpServletResponse response, Object handler, Exception ex)
    			throws Exception {
    		System.out.println(this.getClass().getName()+"afterCompletion");
    	}
    
    	@Override
    	public void afterConcurrentHandlingStarted(HttpServletRequest request,
    			HttpServletResponse response, Object handler) throws Exception {
    		System.out.println(this.getClass().getName()+"afterConcurrentHandlingStarted");
    		super.afterConcurrentHandlingStarted(request, response, handler);
    	}
    
    }
    
    
    
    
    package interceptor;
    
    import org.springframework.ui.ModelMap;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.context.request.WebRequestInterceptor;
    
    public class MyInterceptor3 implements WebRequestInterceptor{
    
    	@Override
    	public void afterCompletion(WebRequest arg0, Exception arg1)
    			throws Exception {
    		System.out.println(this.getClass().getName()+"afterCompletion");
    	}
    
    	@Override
    	public void postHandle(WebRequest arg0, ModelMap arg1) throws Exception {
    		System.out.println(this.getClass().getName()+"postHandle");
    	}
    
    	@Override
    	public void preHandle(WebRequest arg0) throws Exception {
    		System.out.println(this.getClass().getName()+"preHandle");
    	}
    
    }
    
    
    
    
    package interceptor;
    
    import org.springframework.ui.ModelMap;
    import org.springframework.web.context.request.WebRequest;
    
    public class MyInterceptor4 extends MyInterceptor3{
    
    	@Override
    	public void afterCompletion(WebRequest arg0, Exception arg1)
    			throws Exception {
    		System.out.println(this.getClass().getName()+"afterCompletion");
    	}
    
    	@Override
    	public void postHandle(WebRequest arg0, ModelMap arg1) throws Exception {
    		System.out.println(this.getClass().getName()+"postHandle");
    	}
    
    	@Override
    	public void preHandle(WebRequest arg0) throws Exception {
    		System.out.println(this.getClass().getName()+"preHandle");
    	}
    
    }
    
    

### 5.6配置拦截器

springmvc-config.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xmlns:mvc="http://www.springframework.org/schema/mvc"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd
    	http://www.springframework.org/schema/mvc
    	http://www.springframework.org/schema/mvc/spring-mvc.xsd
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
    	<!-- json的转换器注解模式 -->
    	<mvc:annotation-driven></mvc:annotation-driven>
     	<!-- 静态资源访问映射 -->
    	<mvc:resources location="/WEB-INF/js/" mapping="/js/**"></mvc:resources>
    	<!-- 配置拦截器 -->
    	<mvc:interceptors>
    		<!-- 使用bean直接定义(全局拦截) -->
    		<bean class="interceptor.MyInterceptor1">
    		</bean>
    		<bean class="interceptor.MyInterceptor4">
    		</bean>
    		<!-- 拦截器1 -->
    		<mvc:interceptor>
    			<!-- 目标路径 -->
    			<mvc:mapping path="/**"/>
    			<!-- 释放路径 -->
    			<mvc:exclude-mapping path=""/>
    			<!-- 局部拦截(全局拦截，目标路径匹配所有的路径) -->
    			<bean class="interceptor.MyInterceptor2"></bean>
    		</mvc:interceptor>
    		<!-- 拦截器2 -->
    		<mvc:interceptor>
    			<!-- 目标路径(以hello结尾的拦截) -->
    			<mvc:mapping path="/*/hello"/>
    			<!-- 释放路径(以ex结尾的不拦截) -->
    			<mvc:exclude-mapping path="/*/ex"/>
    			<!-- 局部拦截 -->
    			<bean class="interceptor.MyInterceptor3"></bean>
    		</mvc:interceptor>
    		<!-- 拦截器3 -->
    		<!-- <mvc:interceptor>
    			<mvc:mapping path="/*/test"/>
    			<bean class="interceptor.MyInterceptor4"></bean>
    		</mvc:interceptor> -->
    		<mvc:interceptor>
    			<mvc:mapping path="/user/*"/>
    			<bean class="interceptor.LoginInterceptor"></bean>
    		</mvc:interceptor>
    	</mvc:interceptors>
    </beans>
    

### 5.7创建controller

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/screenshot/thp_20260104_192032.png)
    
    
    package controller;
    
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    
    @Controller
    @RequestMapping("/test")
    public class FirstController{
    	
    	@RequestMapping("/toJson")
    	public String toJson(){
    		return "json";
    	}
    	
    	@RequestMapping("/hello")
    	public String hello(Model model){
    		model.addAttribute("message", "hello");
    		return "first";
    	}
    	
    	@RequestMapping("/test")
    	public String test(Model model){
    		model.addAttribute("message", "test");
    		return "first";
    	}
    	
    	@RequestMapping("/ex")
    	public String ex(Model model){
    		model.addAttribute("message", "ex");
    		return "first";
    	}
    }
    
    
    
    
    package controller;
    
    import javax.servlet.http.HttpSession;
    
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.bind.annotation.RequestMapping;
    
    import domain.People;
    
    
    @Controller
    @RequestMapping("/user")
    public class UserController {
    
    	@GetMapping("/login")
    	public String toLogin(){
    		return "login";
    	}
    	
    	@PostMapping("/login")
    	public String login(People people,Model model,HttpSession httpSession){
    		if (people != null && people.getUsername().equals("张三")
    				&& people != null && people.getPassword().equals("password")) {
    			httpSession.setAttribute("people", people);
    			return "redirect:main";
    		} else {
    			model.addAttribute("message", "username or password is error,please try again");
    			return "login";
    		}
    	}
    	
    	@RequestMapping("/main")
    	public String toMain(){
    		return "main";
    	}
    	
    	@RequestMapping("/logout")
    	public String logout(HttpSession session){
    		session.invalidate();
    		return "redirect:login";
    	}
    }
    
    

### 5.8导入js

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/77db0a6d36cf7d3db3598957b70136a7.png)

### 5.9编写jsp

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/33d79c05058732610a65162d44d8f5a0.png)
    
    
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
    
    
    
    <%@ page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="utf-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Test Json</title>
    <script type="text/javascript" src="${pageContext.request.contextPath }/js/jquery.js">
    </script>
    <script type="text/javascript">
    $(document).ready(function (){
    	$("input[type='button'][value='json提交']").click(function (){
    		$.ajax({
    			url:"${pageContext.request.contextPath}/test/json",
    			type:"post",
    			data:JSON.stringify({id:$("input[name='id']").val(),
    				name:$("input[name='name']").val(),
    				age:$("input[name='age']").val(),
    				sex:$("input[name='sex']").val()
    			}),
    			contentType:"application/json;charset=utf-8",
    			dataType:"json",
    			success:function(data){
    				if(data != null){
    					$("form").append("<B>"+"id="+data.id+",name="+
    						data.name+",age="+data.age+",sex="+data.sex+"</B>");
    				}
    			}});
    	});
    	
    	$("input[type='button'][value='restful修改']").click(function (){
    		$.ajax({
    			url:"${pageContext.request.contextPath}/test/restful",
    			type:"post",
    			data:JSON.stringify({id:$("input[name='id']").val(),
    				name:$("input[name='name']").val(),
    				age:$("input[name='age']").val(),
    				sex:$("input[name='sex']").val()
    			}),
    			contentType:"application/json;charset=utf-8",
    			dataType:"json",
    			success:function(data){
    				if(data != null){
    					$("form").append("<B>Modify:"+"id="+data.id+",name="+
    						data.name+",age="+data.age+",sex="+data.sex+"</B><br/>");
    				}
    			}
    		});
    	});
    	
    	$("input[type='button'][value='restful增加']").click(function (){
    		$.ajax({
    			url:"${pageContext.request.contextPath}/test/restful",
    			type:"put",
    			data:JSON.stringify({id:$("input[name='id']").val(),
    				name:$("input[name='name']").val(),
    				age:$("input[name='age']").val(),
    				sex:$("input[name='sex']").val()
    			}),
    			contentType:"application/json;charset=utf-8",
    			dataType:"json",
    			success:function(data){
    				if(data != null){
    					$("form").append("<B>Add:"+"id="+data.id+",name="+
    						data.name+",age="+data.age+",sex="+data.sex+"</B><br/>");
    				}
    			}
    		});
    	});
    	
    	$("input[type='button'][value='restful查询']").click(function (){
    		$.ajax({url:"${pageContext.request.contextPath}/test/restful/"+$("input[name='query']").val(),
    			type:"get",
    			contentType:"application/json;charset=utf-8",
    			dataType:"json",
    			success:function(data){
    				if(data != null){
    					$("form").append("<B>Query:"+"id="+data.id+",name="+
    						data.name+",age="+data.age+",sex="+data.sex+"</B><br/>");
    				}
    			},
    			error:function(){
    				$("form").append("<B>Query:not have id="+$("input[name='query']").val()+"</B><br/>");
    			}
    		});
    	});
    	
    	$("input[type='button'][value='restful删除']").click(function (){
    		$.ajax({url:"${pageContext.request.contextPath}/test/restful/"+$("input[name='delete']").val(),
    			type:"delete",
    			contentType:"application/json;charset=utf-8",
    			dataType:"json",
    			success:function(data){
    				if(data != null){
    					$("form").append("<B>delete:"+"id="+data.id+",name="+
    						data.name+",age="+data.age+",sex="+data.sex+"</B><br/>");
    				}
    			},
    			error:function(){
    				$("form").append("<B>Delete:not have id="+$("input[name='delete']").val()+"</B><br/>");
    			}
    		});
    	});
    	
    });
    </script>
    </head>
    <body>
    	<form action="">
    		<table border="1px">
    			<tr>
    				<td>
    					id:
    				</td>
    				<td>
    					<input type="text" name="id">
    				</td>
    			</tr>
    			<tr>
    				<td>
    					name:
    				</td>
    				<td>
    					<input type="text" name="name">
    				</td>
    			</tr>
    			<tr>
    				<td>
    					age:				
    				</td>
    				<td>
    					<input type="text" name="age">
    				</td>
    			</tr>
    			<tr>
    				<td>
    					sex:
    				</td>
    				<td>
    					<input type="text" name="sex">
    				</td>
    			</tr>
    			<tr>
    				<td>
    					<input type="reset" value="重置">
    				</td>
    				<td>
    					<input type="button" value="json提交">
    				</td>
    			</tr>
    			<tr>
    				<td colspan="2">
    					<input type="button" value="restful增加">
    				</td>
    			</tr>
    			<tr>
    				<td colspan="2">
    					<input type="button" value="restful修改">
    				</td>
    			</tr>
    			<tr>
    				<td>
    					<input type="text" name="query">
    				</td>
    				<td>
    					<input type="button" value="restful查询">
    				</td>
    			</tr>
    			<tr>
    				<td>
    					<input type="text" name="delete">
    				</td>
    				<td>
    					<input type="button" value="restful删除">
    				</td>
    			</tr>
    		</table>
    	</form>
    </body>
    </html>
    
    
    
    <%@ page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="utf-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>login page!</title>
    </head>
    <body>
    	<form action="${pageContext.request.contextPath }/user/login" method="post">
    		<table border="1px">
    			<tr>
    				<td>
    					id:
    				</td>
    				<td>
    					<input type="text" name="id">
    				</td>
    			</tr>
    			<tr>
    				<td>
    					name:
    				</td>
    				<td>
    					<input type="text" name="username">
    				</td>
    			</tr>
    			<tr>
    				<td>
    					password:
    				</td>
    				<td>
    					<input type="password" name="password">
    				</td>
    			</tr>
    			<tr>
    				<td>
    					<input type="reset" value="重置">
    				</td>
    				<td>
    					<input type="submit" value="提交">
    				</td>
    			</tr>
    		</table>
    		<font style="size=8px;color: red">
    			${message }
    		</font>
    	</form>
    </body>
    </html>
    
    
    
    <%@ page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="utf-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Main page!</title>
    </head>
    <body>
    	this people is ${people.username },password is ${people.password }<br/>
    	<a href="${pageContext.request.contextPath }/user/login" >logout</a>
    </body>
    </html>
    

### 5.10测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2c4cc9d203ea01edd82aa3a295ed5808.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c131a9523d3267a6795e7b2397c16617.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/476709059fa96572c7ded10f249e783c.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/da1cf64fa3d5ac1593c859c093715cc4.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/826184a8edc9bddc4315a1de28399ada.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b0fa97c2d95dcb236e9797d9814ca07a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5c0719f245d991c5882cbf598370481d.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/63b63f873fb4d7f72a3b19cf28cf3ca5.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bfca25dcf18589d8d7428a16282f8728.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/069a7aa0f84f2dcf0fa9672f38afd77f.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/182ee851fbaeb29adf7ce6516fc45ba7.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b2ed10fca4db88beedb0eb13ccee6cc3.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9b1edb2fdf96fb766ffc423be0606fc8.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7716bdf1d57c663f2a0019c0353e7d3c.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1651cc1ed09268be259e757a52420c03.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e40384933b9ba1c3119697f5abd09a2f.png)  
session还在有效  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8d6c190e918b523783294b89cc41a94a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c46062bc5aeb7991e6270b23a35f723f.png)

## 6.总结

多个拦截器或形成拦截器链，可以使用责任链的设计模式设计多个拦截器。  
比如第一个拦截器设计传输编码；  
第二个拦截器过滤敏感词；  
第三个持久化数据；  
第四个session持久化；  
第五个释放资源；  
第六个记录访问日志。。。。。。  
类似于spring的aop。
