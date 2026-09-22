---
layout: post
title: "spring Bean的作用域--单例&原型"
date: 2019-02-18 19:44:04 +0800
categories: [springBean作用域, 单例作用域, 原型作用域, Bean的全部作用域, springBean的作用范围控制]
description: "本文详细介绍了Spring框架中Bean的六种作用域：singleton、prototype、request、session、globalSession和websocket，通过实例展示了单例和原型作用域的使用方式，并对比了它们的区别。"
keywords: springBean作用域, 单例作用域, 原型作用域, Bean的全部作用域, springBean的作用范围控制
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87640850
> - 发布时间：2019-02-18 19:44:04
> - 阅读量：970
> - 分类：java同时被 2 个专栏收录, 订阅专栏, spring
> - 标签：#springBean作用域, #单例作用域, #原型作用域, #Bean的全部作用域, #springBean的作用范围控制

## 摘要

文章浏览阅读970次。本文详细介绍了Spring框架中Bean的六种作用域：singleton、prototype、request、session、globalSession和websocket，通过实例展示了单例和原型作用域的使用方式，并对比了它们的区别。

---

#### springBean的作用域

  * 1.Bean的作用域
  * 2.常用的作用域
  * 3.例子
  *     * 3.1新建一个空的spring项目
    * 3.2创建java文件
    * 3.3xml文件
    * 3.4运行结果
  * 4.总结

## 1.Bean的作用域

1.单例-singleton  
2.原型-prototype  
3.request  
4.session  
5.globalSession  
6.websocket

## 2.常用的作用域

单例和原型是通用的作用域，其余的在网络编程中能够用到。

## 3.例子

### 3.1新建一个空的spring项目

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f567c031de8267da5c923773d9313247.png)

### 3.2创建java文件

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8d53e5bdcb273549a649e8dbadb9d712.png)
    
    
    package bean;
    
    public class Bean {
    
    	public void say(){
    		System.out.println(this);
    	}
    	
    }
    
    
    
    
    package client;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import bean.Bean;
    
    public class Main {
    
    	public static void main(String[] args) {
    
    		String path = "resource/beans-";
    		@SuppressWarnings("resource")
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    				path + "singleton.xml");
    		Bean bean = (Bean)applicationContext.getBean("bean");
    		bean.say();
    		bean = (Bean)applicationContext.getBean("bean");
    		bean.say();
    		applicationContext = new ClassPathXmlApplicationContext(path
    				+ "prototype.xml");
    		bean = (Bean)applicationContext.getBean("beanp");
    		bean.say();
    		bean = (Bean)applicationContext.getBean("beanp");
    		bean.say();
    	}
    
    }
    
    

### 3.3xml文件

beans-prototype.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<bean id="beanp" class="bean.Bean" scope="prototype">
    	</bean>
    </beans>
    

beans-singleton.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<bean id="bean" class="bean.Bean" scope="singleton">
    	</bean>
    </beans>
    

### 3.4运行结果
    
    
    bean.Bean@ae45eb6
    bean.Bean@ae45eb6
    bean.Bean@6a4f787b
    bean.Bean@685cb137
    

## 4.总结

spring的Bean的作用域  
1.Singleton  
2.Prototype
