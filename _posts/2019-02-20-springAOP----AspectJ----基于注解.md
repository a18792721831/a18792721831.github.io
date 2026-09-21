---
layout: post
title: "springAOP----AspectJ----基于注解"
date: 2019-02-20 19:37:45 +0800
categories: [springAOP AspectJ, 基于注解的AspectJ, springAOP注解介绍, 如何用注解实现AspectJ的SpringAOP, 注解声明式AspectJ]
description: "本文探讨了AspectJ与Spring AOP的结合使用，详细介绍了AspectJ注解配置方式的优点，如@Aspect、@Pointcut、@Before等，并通过实例展示了如何在Spring环境中配置和运行AspectJ切面。"
keywords: springAOP AspectJ, 基于注解的AspectJ, springAOP注解介绍, 如何用注解实现AspectJ的SpringAOP, 注解声明式AspectJ
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87817309
> - 发布时间：2019-02-20 19:37:45
> - 阅读量：741
> - 分类：java同时被 3 个专栏收录, 订阅专栏, spring
> - 标签：#springAOP AspectJ, #基于注解的AspectJ, #springAOP注解介绍, #如何用注解实现AspectJ的SpringAOP, #注解声明式AspectJ

## 摘要

文章浏览阅读741次。本文探讨了AspectJ与Spring AOP的结合使用，详细介绍了AspectJ注解配置方式的优点，如@Aspect、@Pointcut、@Before等，并通过实例展示了如何在Spring环境中配置和运行AspectJ切面。

---

#### 基于注解的AspectJ

  * 1.AspectJ方式的优缺点
  * 2.注解介绍
  * 3.例子
  *     * 3.1准备
    * 3.2创建一个spring工程
    * 3.3创建Java文件
    * 3.4xml文件
    * 3.5运行结果
  * 4.总结

## 1.AspectJ方式的优缺点

与基于代理类的AOP相比，基于XML的声明式AspectJ要方便的多。但是也存在一些问题：要在Spring文件中配置大量的代码信息。  
注解配置方式可以取代spring配置文件中未实现AOP功能所配置的臃肿代码。

## 2.注解介绍

@Aspect用于定义一个切面  
@Pointcut用于定义切入点表达式：在使用时还需要定义哥包含名字和任意参数的方法签名来表示切入点名称。实际上，这个方法签名就是一个返回值为void，切方法体为空的普通方法。  
@Before用于定义前置通知，相当于BeforeAdvice，使用时通常需要制定一个value属性，值用来制定一个切入点表达式。  
@AfterReturning用于定义后置通知，相当于AfterReturnAdvice，使用时可以制定pointcut/value,都用于制定切入点表达式，returning属性值用于表示Advice方法中可以定义榆次同名的形参，形参用于访问目标方法的返回值。  
@Around用于定义环绕通知，相当于MethodInterceptor,使用时需要指定一个value属性，用于指定被植入的切入点  
@AfterThrowing用于定义异常通知来处理程序中未处理的异常，相当于ThrowAdvice  
@After用于定义最终final通知，不管是否异常，该通知都会执行  
@DeclareParents用于定义引介通知，相当于IntroductiongInterceptor

## 3.例子

### 3.1准备

aspectjrt-1.8.10.jar  
aspectjweaver-1.8.10.jar

### 3.2创建一个spring工程

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fccefc3b0bb3f16179ccc6ac8319840f.png)

### 3.3创建Java文件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4387cc363399137c860f8573d81bda84.png)
    
    
    package aspect;
    
    import org.aspectj.lang.JoinPoint;
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.aspectj.lang.annotation.After;
    import org.aspectj.lang.annotation.AfterReturning;
    import org.aspectj.lang.annotation.AfterThrowing;
    import org.aspectj.lang.annotation.Around;
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Before;
    import org.aspectj.lang.annotation.Pointcut;
    import org.junit.Test;
    import org.springframework.stereotype.Component;
    
    @Aspect
    @Component("aspect")
    public class MyAspect {
    
    	@Test
    	@Pointcut("execution(* bean.*.*(..))")
    	public void pointCut(){
    		System.out.println("MyAspect#pointCut");
    	}
    	
    	@Before("pointCut()")
    	public void before(JoinPoint joinPoint){
    		System.out.println("MyAspect#before");
    		System.out.println(joinPoint.getSignature().getName());
    	}
    	
    	@After("pointCut()")
    	public void after(JoinPoint joinPoint){
    		System.out.println(joinPoint.getSignature().getName());
    		System.out.println("MyAspect#after");
    	}
    	
    	@Around("pointCut()")
    	public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable{
    		System.out.println("MyAspect#around#before");
    		Object object = proceedingJoinPoint.proceed();
    		System.out.println("MyAspect#around#after");
    		return object;
    	}
    	
    	@Test
    	@AfterThrowing(value="pointCut()",throwing="e")
    	public void throwing(){
    		System.out.println("MyAspect#throwing");
    	}
    	
    	@Test
    	@AfterReturning("pointCut()")
    	public void afterReturning(){
    		System.out.println("Myaspect#afterReturning");
    	}
    }
    
    
    
    
    package bean;
    
    import org.junit.Test;
    import org.springframework.stereotype.Component;
    
    @Component("people")
    public class People {
    
    	@Test
    	public void play(){
    		
    		System.out.println("People#play");
    		
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
    		People people = (People)applicationContext.getBean("people");
    		people.play();
    	}
    
    }
    
    

### 3.4xml文件

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
    	<!-- 指定需要扫描的包 -->
    	<context:component-scan base-package="bean,aspect"></context:component-scan>
    	<!-- 启动基于注解的声明式AspectJ支持 -->
    	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
    </beans>
    

### 3.5运行结果
    
    
    MyAspect#around#before
    MyAspect#before
    play
    People#play
    MyAspect#around#after
    play
    MyAspect#after
    Myaspect#afterReturning
    
    

## 4.总结

AspectJ 是一个基于Java的AOP框架，spring2.0之后  
SpringAOP引入对AspectJ的支持，并允许直接进行编程。

注意，扫描包路径不要忘记扫描增强类所在的包（即所有的bean都要保证被注册）
