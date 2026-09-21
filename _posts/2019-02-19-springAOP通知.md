---
layout: post
title: "springAOP通知"
date: 2019-02-19 20:10:21 +0800
categories: [springAOP通知, AOP通知类型, 实现自定义AOP, jdk代理与cglib代理的环绕通知, AOP通知类型详解与练习]
description: "springAOP通知1.什么是AOP通知2.通知的类型3.原理4.例子4.1创建一个spring项目4.2创建Java文件4.3创建xml文件4.4运行结果5.总结1.什么是AOP通知AOP通知就是面向切面编程中冗余的代码。比如在每次对数据库进行操作之前，需要记录操作的SQL语句，或者调用方法前打印方法名，这些就是AOP的通知。用大白话说，就是在横向业务中相同的操作，比如日志，事物，异常..._aop的通知是什么意思"
keywords: springAOP通知, AOP通知类型, 实现自定义AOP, jdk代理与cglib代理的环绕通知, AOP通知类型详解与练习
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87730393
> - 发布时间：2019-02-19 20:10:21
> - 阅读量：1
> - 分类：java同时被 3 个专栏收录, 订阅专栏, spring
> - 标签：#springAOP通知, #AOP通知类型, #实现自定义AOP, #jdk代理与cglib代理的环绕通知, #AOP通知类型详解与练习

## 摘要

文章浏览阅读1.1k次。springAOP通知1.什么是AOP通知2.通知的类型3.原理4.例子4.1创建一个spring项目4.2创建Java文件4.3创建xml文件4.4运行结果5.总结1.什么是AOP通知AOP通知就是面向切面编程中冗余的代码。比如在每次对数据库进行操作之前，需要记录操作的SQL语句，或者调用方法前打印方法名，这些就是AOP的通知。用大白话说，就是在横向业务中相同的操作，比如日志，事物，异常..._aop的通知是什么意思

---

#### springAOP通知

  * 1.什么是AOP通知
  * 2.通知的类型
  * 3.原理
  * 4.例子
  *     * 4.1创建一个spring项目
    * 4.2创建Java文件
    * 4.3创建xml文件
    * 4.4运行结果
  * 5.总结

## 1.什么是AOP通知

AOP通知就是面向切面编程中冗余的代码。  
比如在每次对数据库进行操作之前，需要记录操作的SQL语句，或者调用方法前打印方法名，这些就是AOP的通知。  
用大白话说，就是在横向业务中相同的操作，比如日志，事物，异常等等。

## 2.通知的类型

1.环绕通知  
1.1 jdk代理  
1.2 cglib代理  
2.前置通知  
3.后置通知  
4.异常通知  
5.引介通知

## 3.原理

ProxyFactoryBean是FactoryBean接口的实现类，FactoryBean负责实例化一个Bean，而ProxyFactoryBean负责为其他Bean创建代理实例。在spring中，使用ProxyFactoryBean是创建AOP代理的基本方式。

## 4.例子

### 4.1创建一个spring项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fac6168f97d9379d75ff9035850e7ec4.png)

### 4.2创建Java文件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a5241fd961fcf15ad08e24530320128b.png)
    
    
    package aspect;
    
    import java.lang.reflect.Method;
    
    import org.springframework.aop.AfterReturningAdvice;
    
    public class AfterAspect implements AfterReturningAdvice{
    
       @Override
       public void afterReturning(Object returnValue, Method method,
       		Object[] args, Object target) throws Throwable {
       	
       	System.out.println("AfterAspect#afterReturning");
       	
       }
    
    }
    
    
    
    
    package aspect;
    
    import java.lang.reflect.Method;
    
    import org.springframework.aop.MethodBeforeAdvice;
    
    public class BeforeAspect implements MethodBeforeAdvice{
    
       @Override
       public void before(Method method, Object[] args, Object target)
       		throws Throwable {
       	
       	System.out.println("BeforeAspect#before");
       	
       }
    
    }
    
    
    
    
    package aspect;
    
    import intf.PswdDao;
    
    import org.aopalliance.intercept.MethodInvocation;
    import org.springframework.aop.IntroductionInterceptor;
    
    public class IntroductionAspect implements IntroductionInterceptor , PswdDao{
    
       @Override
       public Object invoke(MethodInvocation invocation) throws Throwable {
       	if(implementsInterface(invocation.getMethod().getDeclaringClass())){
       		return invocation.getMethod().invoke(this, invocation);
       	} else {
       		return invocation.proceed();
       	}
       }
    
       @Override
       public boolean implementsInterface(Class<?> intf) {
       	return intf.isAssignableFrom(PswdDao.class);
       }
    
       @Override
       public void sayPswd() {
       	System.out.println("Aspect#sayPswd");
       }
    
    
    }
    
    
    
    
    package aspect;
    
    import org.aopalliance.intercept.MethodInterceptor;
    import org.aopalliance.intercept.MethodInvocation;
    
    public class MyAspect implements MethodInterceptor{
    
       @Override
       public Object invoke(MethodInvocation arg0) throws Throwable {
       	before();
       	Object object = arg0.proceed();
       	after();
       	return object;
       }
    
    
       private void before(){
       	System.out.println("before");
       }
       
       private void after(){
       	System.out.println("after");
       }
       
    }
    
    
    
    
    package bean;
    
    public class People {
    
       public void say(){
       	System.out.println("People#say");
       }
       
    }
    
    
    
    
    package client;
    
    import intf.PswdDao;
    import intf.UserDao;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import bean.People;
    
    public class Main {
    
       public static void main(String[] args) {
       	
       	ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
       			"resource/*.xml");
       	UserDao userDao = (UserDao)applicationContext.getBean("userDaoInterceptorProxy");
       	userDao.add();
       	userDao.modify();
       	System.out.println();
       	People people = (People)applicationContext.getBean("peopleInterceptorProxy");
       	people.say();
       	System.out.println();
       	userDao = (UserDao)applicationContext.getBean("userDaoBeforeProxy");
       	userDao.add();
       	userDao.modify();
       	System.out.println();
       	userDao = (UserDao)applicationContext.getBean("userDaoAfterProxy");
       	userDao.add();
       	userDao.modify();
       	System.out.println();
       	userDao = (UserDao) applicationContext.getBean("pswdDaoInterceptorProxy");
       	userDao.add();
       	userDao.modify();
       	((PswdDao) userDao).sayPswd();
       }
    
    }
    
    
    
    
    package impl;
    
    import intf.UserDao;
    
    public class UserDaoImpl implements UserDao{
    
       @Override
       public void add() {
       	System.out.println("UserDaoImpl#add");
       }
    
       @Override
       public void modify() {
       	System.out.println("UserDaoImpl#modify");
       }
    
    }
    
    
    
    
    package intf;
    
    public interface PswdDao {
    
       void sayPswd();
       
    }
    
    
    
    
    package intf;
    
    public interface UserDao {
    
       void add();
       
       void modify();
       
    }
    
    

### 4.3创建xml文件

bean.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd">
       <!-- 目标类 -->
       <bean id="userDao" class="impl.UserDaoImpl"></bean>
       <!-- 增强类 -->
       <bean id="aspect" class="aspect.MyAspect"></bean>
       <!-- 代理对象 -->
       <bean id="userDaoInterceptorProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
       	<!-- 增强的接口 -->
       	<property name="proxyInterfaces">
       		<value>intf.UserDao</value>
       	</property>
       	<!-- 指定目标对象 -->
       	<property name="target" ref="userDao">
       	</property>
       	<!-- 指定增强类 -->
       	<property name="interceptorNames"> 
       		<value>aspect</value>
       	</property>
       	<property name="proxyTargetClass">
       		<value>false</value>
       	</property>
       </bean>
       <bean id="people" class="bean.People"></bean>
       <bean id="peopleInterceptorProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
       	<!-- 指定目标对象 -->
       	<property name="target" ref="people">
       	</property>
       	<!-- 指定增强类 -->
       	<property name="interceptorNames"> 
       		<value>aspect</value>
       	</property>
       	<property name="proxyTargetClass">
       		<value>true</value>
       	</property>
       </bean>
       <bean id="userDaoBeforeProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
       	<!-- 增强的接口 -->
       	<property name="proxyInterfaces">
       		<value>intf.UserDao</value>
       	</property>
       	<!-- 指定目标对象 -->
       	<property name="target" ref="userDao">
       	</property>
       	<!-- 指定增强类 -->
       	<property name="interceptorNames"> 
       		<value>beforeAspect</value>
       	</property>
       	<property name="proxyTargetClass">
       		<value>false</value>
       	</property>
       </bean>
       <!-- 增强类 -->
       <bean id="beforeAspect" class="aspect.BeforeAspect"></bean>
       <!-- 增强类 -->
       <bean id="afterAspect" class="aspect.AfterAspect"></bean>
       <bean id="userDaoAfterProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
       	<!-- 增强的接口 -->
       	<property name="proxyInterfaces">
       		<value>intf.UserDao</value>
       	</property>
       	<!-- 指定目标对象 -->
       	<property name="target" ref="userDao">
       	</property>
       	<!-- 指定增强类 -->
       	<property name="interceptorNames"> 
       		<value>afterAspect</value>
       	</property>
       	<property name="proxyTargetClass">
       		<value>false</value>
       	</property>
       </bean>
       <bean id="intriductionAspect" class="aspect.IntroductionAspect"></bean>
       <bean id="pswdDaoInterceptorProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
       	<!-- 增强的接口 -->
       	<property name="proxyInterfaces">
       		<value>intf.PswdDao</value>
       	</property>
       	<!-- 指定目标对象 -->
       	<property name="target" ref="intriductionAspect">
       	</property>
       	<!-- 指定增强类 -->
       	<property name="interceptorNames"> 
       		<value>intriductionAspect</value>
       	</property>
       	<property name="proxyTargetClass">
       		<value>false</value>
       	</property>
       </bean>
    </beans>
    

### 4.4运行结果
    
    
    before
    UserDaoImpl#add
    after
    before
    UserDaoImpl#modify
    after
    
    before
    People#say
    after
    
    BeforeAspect#before
    UserDaoImpl#add
    BeforeAspect#before
    UserDaoImpl#modify
    
    UserDaoImpl#add
    AfterAspect#afterReturning
    UserDaoImpl#modify
    AfterAspect#afterReturning
    
    Exception in thread "main" java.lang.ClassCastException: com.sun.proxy.$Proxy5 cannot be cast to intf.UserDao
       at client.Main.main(Main.java:32)
    
    

## 5.总结

spring的通知类型：  
1.环绕通知  
1.1 jdk代理  
1.2 cglib代理  
2.前置通知  
3.后置通知  
4.异常通知  
5.引介通知
