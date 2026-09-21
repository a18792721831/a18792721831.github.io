---
layout: post
title: "spring Bean初始化方法：无参构造、有参构造、set注入"
date: 2019-02-18 17:57:34 +0800
categories: [springBean初始化, 构造, spring xml文件bean配置, set注入, 隐式构造与显示构造]
description: "本文深入探讨了Spring框架中Bean的三种初始化方式：无参构造、有参构造和set方法，并通过实例展示了不同XML配置文件下Bean的初始化过程及运行结果。"
keywords: springBean初始化, 构造, spring xml文件bean配置, set注入, 隐式构造与显示构造
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87634086
> - 发布时间：2019-02-18 17:57:34
> - 阅读量：1
> - 分类：java同时被 2 个专栏收录, 订阅专栏, spring
> - 标签：#springBean初始化, #构造, #spring xml文件bean配置, #set注入, #隐式构造与显示构造

## 摘要

文章浏览阅读1.4w次，点赞6次，收藏16次。本文深入探讨了Spring框架中Bean的三种初始化方式：无参构造、有参构造和set方法，并通过实例展示了不同XML配置文件下Bean的初始化过程及运行结果。

---

#### Bean初始化

  * [1.Bean的初始化方式：](<#1Bean_1>)
  * [2.例子](<#2_5>)
  *     * [2.1新建一个空的spring项目](<#21spring_6>)
    * [2.2类文件创建](<#22_13>)
    * [2.3java文件](<#23java_15>)
    * [2.4xml文件](<#24xml_96>)
    * [2.5运行结果](<#25_148>)
  * [3.总结](<#3_159>)

## 1.Bean的初始化方式：

1.无参构造（默认构造方法）；  
2.有参构造（自定义构造方法）；  
3.set方法

## 2.例子

### 2.1新建一个空的spring项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4df1b4bea35b2996ddc7c4e14cfbe1fe.png)  
其中：  
client包是主方法所在类的包；  
domain包是实体Bean的包；  
resource包是xml配置文件所在的包；  
readme是项目介绍。

### 2.2类文件创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1683e327bd53df4e30cd61483b74d030.png)

### 2.3java文件
    
    
    package client;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import domain.People;
    
    public class Main {
    
    	public static void main(String[] args) {
    		
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    				"resource/*With.xml");
    		People people = (People) applicationContext.getBean("people");
    		people.say();
    		
    		applicationContext = new ClassPathXmlApplicationContext(
    				"resource/*No.xml");
    		people = (People) applicationContext.getBean("people");
    		people.say();
    		
    		applicationContext = new ClassPathXmlApplicationContext(
    				"resource/*Pro.xml");
    		people = (People) applicationContext.getBean("people");
    		people.say();
    	}
    
    }
    
    
    
    
    package domain;
    
    import java.io.Serializable;
    
    public class People implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = -3290777865940655640L;
    
    	private String name;
    	
    	private String age;
    
    	public void say(){
    		System.out.println("say");
    	}
    	
    	public People(String name,String age){
    		this.name = name;
    		this.age = age;
    		System.out.println(this.getClass()+"有参构造");
    	}
    	
    	public People(){
    		System.out.println(this.getClass()+"无参构造");
    	}
    	
    	public String getName() {
    		return name;
    	}
    
    	public void setName(String name) {
    		this.name = name;
    	}
    
    	public String getAge() {
    		return age;
    	}
    
    	public void setAge(String age) {
    		this.age = age;
    	}
    	
    }
    
    

### 2.4xml文件

beanNo.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	
    	<bean id="people" class="domain.People">
    	</bean>
    	
    </beans>
    

beanPro.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	
    	<bean id="people" class="domain.People">
    		<property name="name">
    			<value>xiaojua</value>
    		</property>
    		<property name="age">
    			<value>23</value>
    		</property>
    	</bean>
    	
    </beans>
    

beanWith.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	
    	<bean id="people" class="domain.People">
    		<constructor-arg name="name" type="java.lang.String">
    			<value>xiaoming</value>
    		</constructor-arg>
    		<constructor-arg name="age" type="java.lang.String">
    			<value>12</value>
    		</constructor-arg>
    	</bean>
    	
    </beans>
    

### 2.5运行结果
    
    
    class domain.People有参构造
    say
    class domain.People无参构造
    say
    class domain.People无参构造
    say
    
    

## 3.总结

JavaBean的初始化默认使用默认的构造方法，所以，如果某些操作必须在类加载时完成，可以使用显示的默认无参构造方法中完成，否则会使用隐式的默认无参构造。

这个项目实现不同的bean文件配置以及使用不同的方式初始化  
使用无参构造方法，  
有参构造  
set方法