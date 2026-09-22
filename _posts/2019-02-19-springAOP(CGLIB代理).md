---
layout: post
title: "springAOP(CGLIB代理)"
date: 2019-02-19 19:36:50 +0800
categories: [springAOP, CGLIB实现SpringAOP, CGLIB代理原理, CGLIB代理的优缺点, CGLIB代理未实现接口的类]
description: "本文深入探讨了Spring AOP中CGLIB代理的作用与原理，通过实例演示了如何使用CGLIB为未实现接口的类提供动态代理，适用于无法使用JDK代理的情况。"
keywords: springAOP, CGLIB实现SpringAOP, CGLIB代理原理, CGLIB代理的优缺点, CGLIB代理未实现接口的类
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87728747
> - 发布时间：2019-02-19 19:36:50
> - 阅读量：1
> - 分类：java同时被 3 个专栏收录, 订阅专栏, spring
> - 标签：#springAOP, #CGLIB实现SpringAOP, #CGLIB代理原理, #CGLIB代理的优缺点, #CGLIB代理未实现接口的类

## 摘要

文章浏览阅读1k次。本文深入探讨了Spring AOP中CGLIB代理的作用与原理，通过实例演示了如何使用CGLIB为未实现接口的类提供动态代理，适用于无法使用JDK代理的情况。

---

#### Spring AOP----CGLIB代理

  * 1.为什么要有CGLIB代理
  * 2.CGLIB代理的原理
  * 3.例子
  *     * 3.1创建一个spring的项目
    * 3.2创建Java类
    * 3.3xml文件
    * 3.4运行结果
  * 4.总结

## 1.为什么要有CGLIB代理

spring实现的AOP中，jdk代理是通过java.lang.reflect.Proxy类实现的，调用newProxyInstance方法创建代理对象。但是jdk代理有一个致命的缺点，那就是代理的目标类必须至少实现一个接口，如果一个接口也没有实现，那么就会出现异常。  
CGLIB代理正是可以解决这个问题，CGLIB代理没有实现任何接口的类。

## 2.CGLIB代理的原理

CGLIB代理的原理非常的简单：  
CGLIB采用字节码技术，对目标类进行继承，生成目标类的子类，然后对子类进行增强，实现代理的目的。  
spring核心包已经继承CGLIB。

## 3.例子

### 3.1创建一个spring的项目

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2bc7d644498520101dd2e34266652f1b.png)

### 3.2创建Java类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5f757957bdc5ee992f2ab1eda216a8de.png)
    
    
    package bean;
    
    public final class FinalPeople {
    	
    	public void say(){
    		System.out.println("final People");
    	} 
    	
    }
    
    
    
    
    package bean;
    
    public class People {
    
    	public void say(){
    		System.out.println("say");
    	}
    	
    }
    
    
    
    
    package cglib;
    
    import java.lang.reflect.Method;
    
    import org.springframework.cglib.proxy.Enhancer;
    import org.springframework.cglib.proxy.MethodInterceptor;
    import org.springframework.cglib.proxy.MethodProxy;
    
    public class MyCglib implements MethodInterceptor{
    
    	public Object createCglibProxy(Object target){
    		Enhancer enhancer = new Enhancer();
    		enhancer.setSuperclass(target.getClass());
    		enhancer.setCallback(this);
    		return enhancer.create();
    	}
    	
    	@Override
    	public Object intercept(Object arg0, Method arg1, Object[] arg2,
    			MethodProxy arg3) throws Throwable {
    
    		System.out.println("before");
    		Object object = arg3.invokeSuper(arg0, arg2);
    		System.out.println("after");
    		return object;
    	}
    
    }
    
    
    
    
    package client;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import cglib.MyCglib;
    import bean.FinalPeople;
    import bean.People;
    
    public class Main {
    
    	public static void main(String[] args) {
    		
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    				"resource/*.xml");
    		People people = (People) applicationContext.getBean("people");
    		MyCglib myCglib = (MyCglib)applicationContext.getBean("cglib");
    		People peopleProxy = (People)myCglib.createCglibProxy(people);
    		peopleProxy.say();
    		System.out.println();
    		FinalPeople finalPeople = (FinalPeople) applicationContext
    				.getBean("finalPeople");
    //		FinalPeople finalPeopleProxy = (FinalPeople) myCglib
    //				.createCglibProxy(finalPeople);
    //		finalPeopleProxy.say();
    	}
    
    }
    
    

### 3.3xml文件
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<bean id="people" class="bean.People"></bean>
    	<bean id="cglib" class="cglib.MyCglib"></bean>
    	<bean id="finalPeople" class="bean.FinalPeople"></bean>
    </beans>
    

### 3.4运行结果
    
    
    before
    say
    after
    
    
    

## 4.总结

如果需要对无法用jdk代理的bean进行代理  
可以使用CGLIB代理  
CGLIB : Code Generation Library  
一个高性能开源代码生成包  
采用非常底层的字节码技术，对目标类生成子类，并对子类进行增强。  
代理调用的是子类。

缺陷：  
如果目标类使用了final修饰，那么不可以使用CGLIB
