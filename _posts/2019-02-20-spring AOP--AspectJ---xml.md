---
layout: post
title: "spring AOP--AspectJ---xml"
date: 2019-02-20 19:21:30 +0800
categories: [springAOP AspectJ, 基于xm声明的AspectJ, spring切入点表达式, AspectJ通知类型, 如何配置一个AspectJ的SpringAOP]
description: "本文详细介绍了如何使用AspectJ与Spring框架集成实现面向切面编程(AOP)，包括配置切面、切入点、通知等关键概念及XML配置方法，通过实例展示了AspectJ的强大功能。"
keywords: springAOP AspectJ, 基于xm声明的AspectJ, spring切入点表达式, AspectJ通知类型, 如何配置一个AspectJ的SpringAOP
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87816295
> - 发布时间：2019-02-20 19:21:30
> - 阅读量：771
> - 分类：java同时被 3 个专栏收录, 订阅专栏, spring
> - 标签：#springAOP AspectJ, #基于xm声明的AspectJ, #spring切入点表达式, #AspectJ通知类型, #如何配置一个AspectJ的SpringAOP

## 摘要

文章浏览阅读771次。本文详细介绍了如何使用AspectJ与Spring框架集成实现面向切面编程(AOP)，包括配置切面、切入点、通知等关键概念及XML配置方法，通过实例展示了AspectJ的强大功能。

---

#### 基于xml的AspectJ

  * 1.AspectJ介绍
  * 2.aop:config
  * 3.如何配置
  *     * 3.1配置切面
    * 3.2配置切入点
    * 3.3配置通知
  * 4.例子
  *     * 4.1准备
    * 4.2创建一个spring工程
    * 4.3创建Java文件
    * 4.4xml文件
    * 4.5运行结果
  * 5.总结

## 1.AspectJ介绍

AspectJ是一个基于Java语言的AOP框架，他提供了强大的AOP功能。Spring2.0以后，SpringAOP引入了对AspectJ的支持。  
使用AspectJ实现AOP有两种方式：  
1.基于xml的声明式AspectJ；  
2.基于注解的声明式AspectJ；

## 2.aop:config

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0a01e1fce6a68b3ec6c2a8ab6989b1dd.jpeg)  
spring配置文件中的元素下可以包含多个aop:config元素，一个aop:config元素中又可以包含属性和子元素。  
其层级关系如上图所示。

## 3.如何配置

### 3.1配置切面

aop:aspect元素  
id用于定义该切面的唯一标识；  
ref用于引用普通的SpringBean；

### 3.2配置切入点

aop:pointcut元素  
当aop:pointcut是aop:config的直接子元素时，表示全局的切入点。  
如果aop:pointcut是aop:aspect的直接子元素时，表示局部切入点。  
id用于指定切入点的唯一标识；  
expression用于指定切入点关联的切入点表达式。  
切入点表达式：
    
    
    execution(* com.bean.*.*(..))
    

其中execution()是表达式的主体  
第一个*表示方法的返回类型
    
    
    com.bean.*.*
    

第一个 _表示XXX类  
第二个_表示的是XXX方法
    
    
    (..)
    

两个点表示的是任意参数

springAOP中切入点表达式的基本格式如下：
    
    
    execution(modifiers-pattern?ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern)
    

modifiers-pattern:表示定义的目标方法的访问修饰符：public,private…  
ret-type-pattern:表示定义的目标方法的返回值类型：void,String…  
declaring-type-pattern:表示定义的目标方法的类路径：com.bean。。。。  
name-pattern:表示具体需要被代理的目标方法：add()。。。  
param-pattern:表示需要被代理的目标方法包含的参数；  
throws-pattern:表示需要被代理的目标方法抛出的异常类型。  
其他带有？的标识都是选填项。

### 3.3配置通知

pointcut:该属性用于指定一个切入点表达式  
pointcut-ref该属性指定一个已经存在的植入点名称  
pointcut与pointcut-ref二选一  
method该属性指定一个方法名，指定将切面bean中的该方法转换为增强处理  
throwing该属性支队after-throwing元素有效，用于指定一个形参名，异常通知方法可以通过改形参访问目标方法所抛出的异常  
returning该属性支队after-returning元素有效，用于指定衣蛾形参名，后置通知方法可以通过该形参访问目标增强方法的返回值。

## 4.例子

### 4.1准备

aspectjrt-1.8.10.jar  
aspectjweaver-1.8.10.jar

### 4.2创建一个spring工程

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/be9c894d5defd9203d5e509a3f920ceb.png)

### 4.3创建Java文件

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9f3b48ce6a20effeda3129243e0a259d.png)
    
    
    package aspect;
    
    import org.aspectj.lang.JoinPoint;
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.aspectj.lang.annotation.Aspect;
    import org.springframework.stereotype.Component;
    //@Aspect
    //@Component("aspect")
    public class MyAspect {
    
    	public void before(){
    		System.out.println("前置通知");
    		System.out.println("MyAspect#before");
    	}
    	
    	public void after(JoinPoint joinpoint){
    		System.out.println("后置通知");
    		System.out.println("MyAspect#after");
    		System.out.println(joinpoint.getSignature().getName());
    	}
    	
    	public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable{
    		System.out.println("MyAspect#around#before");
    		System.out.println("环绕通知#前置通知");
    		Object object = proceedingJoinPoint.proceed();
    		System.out.println("环绕通知#后置通知");
    		System.out.println("MyAspect#around#after");
    		return object;
    	}
    
    	public void afterThrow(JoinPoint joinPoint,Throwable e){
    		System.out.println("异常通知");
    		System.out.println("MyAspect#afterThrow");
    		System.out.println(e.getMessage());
    	}
    	
    	public void afterAfter(){
    		System.out.println("最终通知");
    		System.out.println("MyAspect#after#after");
    	}
    }
    
    
    
    
    package bean;
    
    import org.springframework.stereotype.Component;
    
    //@Component("people")
    public class People{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 6664089908413976436L;
    
    	public void say(){
    		System.out.println("People#say");
    //		int a = 3/0;
    	}
    	
    }
    
    
    
    
    package client;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import bean.People;
    
    
    public class Main {
    
    	public static void main(String[] args) {
    
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    				"resource/*.xml");
    		People people = (People) applicationContext.getBean("people");
    		people.say();
    		
    	}
    
    }
    
    

### 4.4xml文件

bean.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
       <!-- 目标bean -->
       <context:annotation-config/>
       <!-- 指定需要扫描的包 -->
       <!-- 
       <context:component-scan base-package="bean,aspect"></context:component-scan>
       -->
       <bean id="people" class="bean.People"></bean> 
       <!-- 增强类 -->
       <bean id="aspect" class="aspect.MyAspect"></bean>
       <!-- aop编程 -->
       <aop:config>
       	<!-- 配置切面 -->
       	<aop:aspect ref="aspect">
       		<!-- 配置切入点 -->
       		<aop:pointcut expression="execution(* bean.*.*(..))" id="aspectPoint"/>
       		<!-- 关联切入点和增强类 -->
       		<!-- 前置通知 -->
       		<aop:before method="before" pointcut-ref="aspectPoint"/>
       		<!-- 后置通知 -->
       		<aop:after-returning method="after" pointcut-ref="aspectPoint" returning="returnVal"/>
       		<!-- 环绕通知 -->
       		<aop:around method="around" pointcut-ref="aspectPoint"/>
       		<!-- 异常通知 -->
       		<aop:after-throwing method="afterThrow" pointcut-ref="aspectPoint" throwing="e"/>
       		<!-- 最终通知 -->
       		<aop:after method="afterAfter" pointcut-ref="aspectPoint"/>
       	</aop:aspect>
       </aop:config>
    </beans>
    

### 4.5运行结果
    
    
    前置通知
    MyAspect#before
    MyAspect#around#before
    环绕通知#前置通知
    People#say
    最终通知
    MyAspect#after#after
    环绕通知#后置通知
    MyAspect#around#after
    后置通知
    MyAspect#after
    say
    
    

## 5.总结

AspectJ 是一个基于Java的AOP框架，spring2.0之后  
SpringAOP引入对AspectJ的支持，并允许直接进行编程。
