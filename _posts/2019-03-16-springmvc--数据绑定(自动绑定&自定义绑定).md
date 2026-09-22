---
layout: post
title: "springmvc--数据绑定(自动绑定&自定义绑定)"
date: 2019-03-16 17:02:53 +0800
categories: [springmvc-数据绑定, springmvc数据传输, springmvc数据格式化, springmvc自定义数据绑定, Javaweb请求与后台数据传输]
description: "本文深入探讨SpringMVC框架中的数据绑定机制，包括默认数据类型绑定、简单类型绑定、POJO类型绑定、嵌套POJO类型绑定、自定义数据绑定（Converter与Formatter）、以及数组绑定等核心内容。"
keywords: springmvc-数据绑定, springmvc数据传输, springmvc数据格式化, springmvc自定义数据绑定, Javaweb请求与后台数据传输
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88601280
> - 发布时间：2019-03-16 17:02:53
> - 阅读量：2
> - 分类：java同时被 3 个专栏收录, 订阅专栏, springMVC
> - 标签：#springmvc-数据绑定, #springmvc数据传输, #springmvc数据格式化, #springmvc自定义数据绑定, #Javaweb请求与后台数据传输

## 摘要

文章浏览阅读2.4k次，点赞4次，收藏10次。本文深入探讨SpringMVC框架中的数据绑定机制，包括默认数据类型绑定、简单类型绑定、POJO类型绑定、嵌套POJO类型绑定、自定义数据绑定（Converter与Formatter）、以及数组绑定等核心内容。

---

#### springmvc--数据绑定自动绑定&自定义绑定

  * 1.什么是数据绑定
  * 2.实例准备
  *     * 2.1创建项目
    * 2.2导入jar包
    * 2.3springmvc配置
    * 2.4创建springmvc-config.xml
    * 2.5创建controller
  * 3.绑定默认数据类型
  *     * 3.1 在controller中新增方法
    * 3.2创建jsp文件
    * 3.3启动项目访问
  * 4.绑定简单类型
  *     * 4.1在controller中增加方法
    * 4.2测试
  * 5.参数别名
  *     * 5.1在controller中增加方法
    * 5.2测试
  * 6.绑定POJO类型
  *     * 6.1增加实体类
    * 6.2在controller中增加方法
    * 6.3增加jsp
    * 6.4测试
    * 注意
  * 7.嵌套POJO类型
  *     * 7.1创建实体
    * 7.2在controller中增加方法
    * 7.3增加jsp
    * 7.4测试
  * 8.自定义数据绑定-Converter
  *     * 8.1Converter
    * 8.2增加自定义的Converter类
    * 8.3springmvc-config.xml配置
    * 8.4创建jsp
    * 8.5在controller中增加方法
    * 8.6测试
  * 9.自定义数据绑定-Formatter
  *     * 9.1增加自定义的Formatter类
    * 9.2springmvc-config.xml配置
    * 9.3创建jsp
    * 9.5在controller中增加方法
    * 9.6测试
  * 10数组绑定
  *     * 10.1在controller中增加方法
    * 10.2创建jsp
    * 10.3测试

## 1.什么是数据绑定

springmvc项目启动后，客户端发起的请求有时需要传输数据，那么这个传输的过程中如何准确的发出与接收数据？

  * 1.springmvc把ServletRequest对象传递给DataBinder;
  * 2.将处理方法的入参对象传递给DataBinder;
  * 3.DataBinder调用ConversionService组件进行数据类型转换、数据格式化，并将ServletRequest对象中的消息填充到参数对象中；
  * 4.调用Validator组件对已经绑定了请求数据的参数对象进行数据合法性检测；
  * 5.校验完成后会生成数据绑定结果BindingResult对象，springmvc会将BindingResult对象中的内容赋给处理方法的对应参数；

## 2.实例准备

### 2.1创建项目

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/23efb85e9f5507de7f9f710a623f67a8.png)

### 2.2导入jar包

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/428f8343e8216131d01e996485714daa.png)

### 2.3springmvc配置

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
    
    

### 2.4创建springmvc-config.xml

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
    </beans>
    

### 2.5创建controller

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2ef1651dbe35539bc80d3f9c85f7b398.png)  
使用注解。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8eda5d30664b0ffa790868ce6bd09337.png)

## 3.绑定默认数据类型

常用的默认数据类型：

  * 1.HttpServletRequest:通过request对象获取请求信息
  * 2.HttpServletResponse:通过response处理响应信息
  * 3.HttpSession:通过session对象得到session中存储的对象
  * 4.Model/ModelMap:Model是一个接口，ModelMap是一个接口实现，作用是将model数据填充到request  
例子：

### 3.1 在controller中新增方法
    
    
    @RequestMapping("/request")
       public String singleBingding(HttpServletRequest httpServletRequest,Model model){
       	model.addAttribute("message", httpServletRequest.getParameter("id"));
       	return "first";
       }
    

### 3.2创建jsp文件

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
    

### 3.3启动项目访问

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c17189da59004093c45ebaa547612ac7.png)  
因为我们从HttpServletReques中获取参数的名字为id，所以传输参数的形参名必须为id.(类型为基本类型)

## 4.绑定简单类型

简单数据类型不用做任何处理，发起请求是什么简单类型，接收的时候直接接收简单类型即可：

### 4.1在controller中增加方法
    
    
    @RequestMapping("/integer")
    	public String singleBindingInteger(Integer integer,Model model){
    		model.addAttribute("message", integer);
    		return "first";
    	}
    

### 4.2测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b38caadb197967e9226c368b2db9a4dc.png)  
其他类型则会出现异常  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cc9d1277e4a14ce2da969dba6179dc9b.png)

## 5.参数别名

在发起请求的时候，有时候前台不知道后台接收的参数名字是什么，为了前后台完全解耦，会让前台定一个参数名字列表，后台接收的时候需要对参数名字进行转换。  
springmvc提供了一个注解用来控制参数：  
@RequestParam:  
value:参数的别名  
name:请求头绑定的别名  
required:是否必须传输  
defaultValue:参数中没有此项时默认的值

### 5.1在controller中增加方法
    
    
    @RequestMapping("/parameter")
    	public String singleBindingComment(
    			@RequestParam(value = "v_id", required = true) Integer id,
    			Model model) {
    		model.addAttribute("message", id);
    		return "first";
    	}
    

### 5.2测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d67cb047dc437c479378ce88083d1e78.png)  
可以看到前台传输的参数和后台接收的名字并不相同。

## 6.绑定POJO类型

POJO类型就是一个简单的JavaBean对象：私有的属性，get,set方法。

### 6.1增加实体类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e3544f457c479e17d65e3572f398b7e9.png)
    
    
    package domain;
    
    import java.io.Serializable;
    
    public class People implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = -1364237643930318187L;
    
    	private Long id;
    	
    	private String name;
    	
    	private Integer age;
    	
    	private Integer sex;
    
    	public Long getId() {
    		return id;
    	}
    
    	public void setId(Long id) {
    		this.id = id;
    	}
    
    	public String getName() {
    		return name;
    	}
    
    	public void setName(String name) {
    		this.name = name;
    	}
    
    	public Integer getAge() {
    		return age;
    	}
    
    	public void setAge(Integer age) {
    		this.age = age;
    	}
    
    	public Integer getSex() {
    		return sex;
    	}
    
    	public void setSex(Integer sex) {
    		this.sex = sex;
    	}
    
    	@Override
    	public String toString() {
    		return "[id=" + this.id + ",name=" + this.name + ",age=" + this.age
    				+ ",sex" + this.sex + "]";
    	}
    	
    }
    
    

### 6.2在controller中增加方法
    
    
    @RequestMapping("/getPeople")
    	public String pojoBinding(People people,
    			Model model){
    		System.out.println(people);
    		model.addAttribute("message", people.toString());
    		return "first";
    	}
    	
    	@RequestMapping("/toPeople")
    	public String toPeopeJSP(){
    		return "people";
    	}
    

因为POJO对象需要使用from 表单提交，所以需要一个跳转的方法，其中toPeople就是访问提交POJO的jsp页面，getPeople接收POJO对象。

### 6.3增加jsp

people.jsp
    
    
    <%@ page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="utf-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>POJO people</title>
    </head>
    <body>
    	<form action="${pageContext.request.contextPath}/test/getPeople"  method="post">
    		id:<input type="text" name="id"><br/>
    		姓名:<input type="text" name="name"><br/>
    		年龄:<input type="text" name="age"><br/>
    		性别:<input type="text" name="sex"><br/>
    		<input type="reset" value="重置"><input type="submit" value="提交">
    		
    	</form>
    </body>
    </html>
    

jsp中input的名字必须与POJO的属性名字相同，否则会出现找不到对应属性的get,set方法的异常。

### 6.4测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a09aee785ef1743b415afcc75183c933.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4adab9b5dc20733bd20bc0fe5fb6c314.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/00020bd3e94ad12085d062a0d13b509e.png)  
说明：参数的类型也必须一致，在传输的过程中会隐式的做类型转换。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/49c2b2356894675831faab4bf09564cb.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e13b0d57f303e02b950652169bfc7f14.png)

### 注意

如果出现中文乱码，可以使用以下方式解决：  
1.在web.xml中增加
    
    
    <!-- 配置编码过滤器 -->
    <filter>
    	<filter-name>CharacterEncodingFilter</filter-name>
    	<filter-class>
    		org.springframework.web.filter.CharacterEncodingFilter
    	</filter-class>
    	<init-param>
    		<param-name>encoding</param-name>
    		<param-value>UTF-8</param-value>
    	</init-param>
    </filter>
    <filter-mapping>
    	<filter-name>CaracterEncodingFilter</filter-name>
    	<url-pattern>/*</url-pattern>
    </filter-mapping>
    

2.使用utf-8的jsp（本人使用utf-8的jsp没有遇到中文乱码）  
3.自定义过滤器  
[Servlet过滤器](<https://blog.csdn.net/a18792721831/article/details/76464659>)  
[HTML中文乱码](<https://blog.csdn.net/a18792721831/article/details/76094106>)

## 7.嵌套POJO类型

有时候数据过于复杂，需要使用嵌套的POJO类型，就是另一个POJO作为本POJO的属性

### 7.1创建实体
    
    
    package domain;
    
    import java.io.Serializable;
    import java.util.List;
    
    public class Server implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = -3793545656487562942L;
    
    	private Long id;
    	
    	private String name;
    	
    	private String remark;
    	
    	private List<People> peoples;
    
    	public Long getId() {
    		return id;
    	}
    
    	public void setId(Long id) {
    		this.id = id;
    	}
    
    	public String getName() {
    		return name;
    	}
    
    	public void setName(String name) {
    		this.name = name;
    	}
    
    	public String getRemark() {
    		return remark;
    	}
    
    	public void setRemark(String remark) {
    		this.remark = remark;
    	}
    
    	public List<People> getPeoples() {
    		return peoples;
    	}
    
    	public void setPeoples(List<People> peoples) {
    		this.peoples = peoples;
    	}
    
    	@Override
    	public String toString() {
    		return "id=" + this.id + ",name=" + this.name + ",remark="
    				+ this.remark + ",peoples=" + this.peoples + "]";
    	}
    	
    }
    
    

### 7.2在controller中增加方法
    
    
    @RequestMapping("/toServer")
    	public String toServer(){
    		return "server";
    	}
    	
    	@RequestMapping("/getServer")
    	public String pojoBindingServer(Server server,
    			Model model){
    		System.out.println(server);
    		model.addAttribute("message", server.toString());
    		return "first";
    	}
    

### 7.3增加jsp

server.jsp
    
    
    <%@ page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="utf-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>POJO server</title>
    </head>
    <body>
    	<form action="${pageContext.request.contextPath}/test/getServer"  method="post">
    		id:<input type="text" name="id"><br/>
    		名称:<input type="text" name="name"><br/>
    		备注:<input type="text" name="remark"><br/>
    		people.id:<input type="text" name="peoples[0].id"><br/>
    		people姓名:<input type="text" name="peoples[0].name"><br/>
    		people年龄:<input type="text" name="peoples[0].age"><br/>
    		people性别:<input type="text" name="peoples[0].sex"><br/>
    		<input type="reset" value="重置"><input type="submit" value="提交">
    		
    	</form>
    </body>
    </html>
    

注意:  
输入框的名字与POJO的类型一致，如果是属性为POJO类型需要用.来调用，如果是数组的POJO属性，使用下标调用。

### 7.4测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9cb28952df88588b7d64c2c664e5c2e5.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9ef52bf3f9bb0e4de245adeb330f90c3.png)

## 8.自定义数据绑定-Converter

### 8.1Converter

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fc7b08ae0f1441f13e905edfdbb04374.png)  
能够将任意类型转换为指定的任意类型：  
S是源类型，T是目标类型。  
比如字符串->日期

### 8.2增加自定义的Converter类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c18f147462509c73856879f8136cf5dc.png)
    
    
    package my_convert;
    
    import java.text.ParseException;
    import java.text.SimpleDateFormat;
    import java.util.Date;
    
    import org.springframework.core.convert.converter.Converter;
    
    public class DateConverter implements Converter<String, Date>{
    
    	private String pattern = "yyyy-MM-dd HH:mm:ss,s";
    	
    	@Override
    	public Date convert(String arg0) {
    		SimpleDateFormat simpleDateFormat = new SimpleDateFormat(pattern);
    		try{
    			return simpleDateFormat.parse(arg0);
    		} catch(ParseException parseException) {
    			throw new IllegalArgumentException("this pattern"+pattern);
    		}
    	}
    
    }
    
    

### 8.3springmvc-config.xml配置
    
    
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
    	<!-- 显示的装配自定义类型转换器 -->
    	<mvc:annotation-driven conversion-service="conversionService">
    	</mvc:annotation-driven>
    	<!-- 自定义类型转换器配置 -->
    	<bean id="conversionService"
    		class="org.springframework.context.support.ConversionServiceFactoryBean">
    		<property name="converters">
    			<set>
    				<bean class="my_convert.DateConverter"></bean>
    			</set>
    		</property>
    	</bean>
    </beans>
    

### 8.4创建jsp

date.jsp
    
    
    <%@ page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="utf-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>converter Date</title>
    </head>
    <body>
    	<form action="${pageContext.request.contextPath }/test/getDate" method="get">
    		<input type="text" name="date"><br/>
    		<input type="submit" value="提交">
    	</form>
    </body>
    </html>
    

### 8.5在controller中增加方法
    
    
    @RequestMapping("/toDate")
    	public String toDate(){
    		return "date";
    	}
    	
    	@RequestMapping("/getDate")
    	public String converterDate(Date date,Model model){
    		model.addAttribute("message", date);
    		return "first";
    	}
    

### 8.6测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/584eff7181d2c040e20e751e2681c003.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6f34c63fa7eaa727b3ae6041d56a4fb1.png)  
字符串必须符合在自定义Converter中定义的格式  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bd871acbad2c74ef3930e09886b77e62.png)  
否则会出现异常  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/44b7ffb37bff4d353d9137bc86695773.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c567d65b2df933df231d7eefe0617a73.png)  
异常信息：
    
    
    警告: Resolved [org.springframework.web.method.annotation.MethodArgumentTypeMismatchException: Failed to convert value of type 'java.lang.String' to required type 'java.util.Date'; nested exception is org.springframework.core.convert.ConversionFailedException: Failed to convert from type [java.lang.String] to type [java.util.Date] for value '2019/03/15 19:48:55,63155'; nested exception is java.lang.IllegalArgumentException: this patternyyyy-MM-dd HH:mm:ss,s]
    

## 9.自定义数据绑定-Formatter

Formatter与Converter的作用相同，但是Formatter的源类型必须是字符串。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c235c771aaf9bfacb89a56b7f75ab4f1.png)

### 9.1增加自定义的Formatter类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8a6191276c6ea43ff2908343b67ef645.png)
    
    
    package my_formatter;
    
    import java.text.ParseException;
    import java.util.Locale;
    
    import org.springframework.format.Formatter;
    
    import domain.People;
    
    public class PeopleFormatter implements Formatter<People>{
    
    	@Override
    	public String print(People arg0, Locale arg1) {
    		return "people:" + arg0 + ",Locale:" + arg1.toString();
    	}
    
    	@Override
    	public People parse(String arg0, Locale arg1) throws ParseException {
    		//id=12,name=你好,age=23,sex=25
    		String[] pStrings = arg0.split(",");
    		People people = new People();
    		people.setId(Long.valueOf(pStrings[0].split("=")[1]));
    		people.setName(pStrings[1].split("=")[1]);
    		people.setAge(Integer.valueOf(pStrings[2].split("=")[1]));
    		people.setSex(Integer.valueOf(pStrings[3].split("=")[1]));
    		return people;
    	}
    
    }
    
    

### 9.2springmvc-config.xml配置
    
    
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
    	<!-- 显示的装配自定义类型转换器 -->
    	<!-- <mvc:annotation-driven conversion-service="conversionService">
    	</mvc:annotation-driven> -->
    	<!-- 自定义类型转换器配置 -->
    	<!-- <bean id="conversionService"
    		class="org.springframework.context.support.ConversionServiceFactoryBean">
    		<property name="converters">
    			<set>
    				<bean class="my_convert.DateConverter"></bean>
    			</set>
    		</property>
    	</bean> -->
    	<mvc:annotation-driven conversion-service="peopleConversionService">
    	</mvc:annotation-driven>
    	<bean id="peopleConversionService"
    	class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    		<property name="formatters">
    			<set>
    				<bean class="my_formatter.PeopleFormatter"></bean>
    			</set>
    		</property>
    	</bean>
    </beans>
    

### 9.3创建jsp

formatPeople.jsp
    
    
    <%@ page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="utf-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"><title>formatter people</title>
    </head>
    <body>
    	<form action="${pageContext.request.contextPath }/test/getFormatPeople"
    		method="get">
    		eg:id=12,name=小华,age=12,sex=0<br/> 
    		<input type="text" name="people">
    		<input type="submit" value="提交">
    	</form>
    </body>
    </html>
    

### 9.5在controller中增加方法
    
    
    @RequestMapping("/toFormatPeople")
    	public String toFormatPeople(){
    		return "formatPeople";
    	}
    	
    	@RequestMapping("getFormatPeople")
    	public String getFormatPeople(People people,Model model){
    		model.addAttribute("message", people.toString());
    		return "first";
    	}
    

### 9.6测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0d782ed5f5644a6c1583de12222996cc.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e90c6418b47e2aa1f5476c118da22e2d.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b7d3aa1252779f10ce9b450ad13b1169.png)

## 10数组绑定

### 10.1在controller中增加方法
    
    
    @RequestMapping("/toCheckBox")
    	public String toCheckBox(){
    		return "checkBox";
    	}
    	
    	@RequestMapping("/getCheckBox")
    	public String getCheckBox(Integer[] array,Model model){
    		String message = "";
    		for(Integer integer : array){
    			message += "\n"+integer;
    		}
    		model.addAttribute("message", message);
    		return "first";
    	}
    

### 10.2创建jsp

checkBox.jsp
    
    
    <%@ page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="utf-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>check box</title>
    </head>
    <body>
    	<form action="${pageContext.request.contextPath }/test/getCheckBox" method="get">
    		<table border="1" width="20%">
    			<%
    				for (int i = 0; i < 10; i++) {
    			%>
    			<tr>
    				<td><input type="checkbox" name="array" value="<%=i%>"><br />
    				</td>
    				<td><%=i%></td>
    			</tr>
    			<%
    				}
    			%>
    		</table>
    		<input type="submit" value="提交">
    	</form>
    </body>
    </html>
    

### 10.3测试

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cdcfbe12c450f5b4ab927da7ec1261cc.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3fbf56f68247aee8284b4fd6b006ebb1.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/83d68563a1dc792317e6328421ddf602.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d1da7d796e86784d524c2dd84b6a4250.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f33e2f01bf609ef5a8a69c2caad85e50.png)
