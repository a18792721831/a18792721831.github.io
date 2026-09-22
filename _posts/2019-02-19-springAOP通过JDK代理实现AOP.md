---
layout: post
title: "springAOP通过JDK代理实现AOP"
date: 2019-02-19 19:13:01 +0800
categories: [jdk实现的springAOP, springAOP原生实现原理, springAOP如何直接使用, 代理实现的AOP, jdk代理实现AOP的优缺点]
description: "本文详细介绍了Spring框架中AOP（面向切面编程）的实现原理，特别是通过JDK代理来创建代理对象的过程。从创建目标类、增强类到代理类，再到最终的运行结果，一步步解析了JDK代理的运作机制。同时，文章还提供了完整的代码示例，包括Spring项目的搭建、Java类的定义、XML配置文件以及运行效果展示。"
keywords: jdk实现的springAOP, springAOP原生实现原理, springAOP如何直接使用, 代理实现的AOP, jdk代理实现AOP的优缺点
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87727284
> - 发布时间：2019-02-19 19:13:01
> - 阅读量：697
> - 分类：java同时被 3 个专栏收录, 订阅专栏, spring
> - 标签：#jdk实现的springAOP, #springAOP原生实现原理, #springAOP如何直接使用, #代理实现的AOP, #jdk代理实现AOP的优缺点

## 摘要

文章浏览阅读697次。本文详细介绍了Spring框架中AOP（面向切面编程）的实现原理，特别是通过JDK代理来创建代理对象的过程。从创建目标类、增强类到代理类，再到最终的运行结果，一步步解析了JDK代理的运作机制。同时，文章还提供了完整的代码示例，包括Spring项目的搭建、Java类的定义、XML配置文件以及运行效果展示。

---

#### Spring实现的AOP--JDK代理

  * 1.jdk代理实现AOP的原理
  * 2.例子
  *     * 2.1创建一个spring项目
    * 2.2创建Java文件
    * 2.3xml文件
    * 2.4运行结果
  * 3.总结

## 1.jdk代理实现AOP的原理

jdk代理：  
aop实现原理：  
首先创建一个目标类–正常类  
然后创建一个增强类–横向业务  
然后创建一个代理类–正常类+横向业务=新类  
最后调用新类  
jdk代理缺点：  
必须是模板模式或者抽象类实现类或者子父类具有相同的方法  
不能代理普通bean

## 2.例子

### 2.1创建一个spring项目

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/940a07990e1c371395553f52cd401ac5.png)

### 2.2创建Java文件

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d570ad54e4c68a8a3eb3226499a8ffb9.png)
    
    
    package aspect;
    
    public class ObjectAsp {
    
    	public void before(){
    		System.out.println(this);
    		System.out.println("before");
    	}
    	
    	public void after(){
    		System.out.println(this);
    		System.out.println("after");
    	}
    	
    }
    
    
    
    
    package client;
    
    import impl.AimImpl;
    import intf.AimIntf;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import proxy.ObjectProxy;
    
    public class Main {
    
    	public static void main(String[] args) {
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    				"resource/*.xml");
    		ObjectProxy objectProxy = (ObjectProxy) applicationContext
    				.getBean("objectProxy");
    		AimIntf aimIntf = (AimIntf) objectProxy
    				.createProxy((AimImpl) applicationContext
    						.getBean("aimImpl"));
    		aimIntf.say();
    	}
    
    }
    
    
    
    
    package impl;
    
    import intf.AimIntf;
    
    public class AimImpl implements AimIntf{
    
    	@Override
    	public void say() {
    		System.out.println(this);
    		System.out.println("say");
    	}
    
    }
    
    
    
    
    package intf;
    
    public interface AimIntf {
    
    	void say();
    	
    }
    
    
    
    
    package proxy;
    
    import intf.AimIntf;
    
    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.Method;
    import java.lang.reflect.Proxy;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import aspect.ObjectAsp;
    
    public class ObjectProxy implements InvocationHandler{
    
    	private AimIntf aimIntf;
    	
    	public Object createProxy(AimIntf aimIntf){
    		this.aimIntf = aimIntf;
    		ClassLoader classLoader = this.getClass().getClassLoader();
    		Class[] classes = aimIntf.getClass().getInterfaces();
    		return Proxy.newProxyInstance(classLoader, classes, this);
    				
    	}
    	
    	@Override
    	public Object invoke(Object proxy, Method method, Object[] args)
    			throws Throwable {
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    				"resource/*.xml");
    		ObjectAsp objectAsp = (ObjectAsp) applicationContext
    				.getBean("objectAsp");
    		objectAsp.before();
    		Object object = method.invoke(aimIntf, args);
    		objectAsp.after();
    		return object;
    	}
    
    }
    
    

### 2.3xml文件

beans.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans
    	http://www.springframework.org/schema/beans/spring-beans.xsd"
    	>
    	<bean id="objectProxy" class="proxy.ObjectProxy"></bean>
    	<bean id="aimImpl" class="impl.AimImpl"></bean>
    	<bean id="objectAsp" class="aspect.ObjectAsp"></bean>
    </beans>
    

### 2.4运行结果
    
    
    aspect.ObjectAsp@e25b2fe
    before
    impl.AimImpl@754ba872
    say
    aspect.ObjectAsp@e25b2fe
    after
    
    

## 3.总结

1.aop实现原理：  
首先创建一个目标类–正常类  
然后创建一个增强类–横向业务  
然后创建一个代理类–正常类+横向业务=新类  
2.jdk代理缺点：  
必须是模板模式或者抽象类实现类或者子父类具有相同的方法  
不能代理普通bean
