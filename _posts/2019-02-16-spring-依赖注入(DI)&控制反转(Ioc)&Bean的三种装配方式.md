---
layout: post
title: "spring-依赖注入(DI)&控制反转(Ioc)&Bean的三种装配方式"
date: 2019-02-16 18:10:49 +0800
categories: [依赖注入DI, 控制反转Ioc, springBean的装配方式, set设值注入, 构造方法注入]
description: "本文深入解析Spring框架中的依赖注入和控制反转概念，介绍Bean的装配方式，包括XML配置、注解和自动装配，并通过示例代码展示不同注入方式的具体应用。"
keywords: 依赖注入DI, 控制反转Ioc, springBean的装配方式, set设值注入, 构造方法注入
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87462107
> - 发布时间：2019-02-16 18:10:49
> - 阅读量：589
> - 分类：java同时被 2 个专栏收录, 订阅专栏, spring
> - 标签：#依赖注入DI, #控制反转Ioc, #springBean的装配方式, #set设值注入, #构造方法注入

## 摘要

文章浏览阅读589次。本文深入解析Spring框架中的依赖注入和控制反转概念，介绍Bean的装配方式，包括XML配置、注解和自动装配，并通过示例代码展示不同注入方式的具体应用。

---

#### 依赖注入&控制反转&bean装配方式

  * [1.依赖注入和控制反转的介绍](<#1_1>)
  * [2.spring中bean的注入方式](<#2springbean_38>)
  * [3.spring中bean的装配方式](<#3springbean_42>)
  * [4.例子](<#4_47>)
  * [5.总结](<#5_466>)

## 1.依赖注入和控制反转的介绍

spring一个很重要的概念就是依赖注入和控制反转。  
那么，什么是依赖注入，什么又是控制反转呢？  
首先依赖注入：  
依赖是什么？  
打个比方，有如下的类：
    
    
    class A{
    	private B b = new B();
    }
    

在上面的类A中，有一个B的属性，在Java中，一个对象在被使用之前，首先需要创建这个对象的实例，然后才能被使用。  
一般的创建方式就是使用关键字new来创建，这样要new一个A的实例对象，隐含条件就是B也会被创建。因此，类A依赖B，如果B在创建的时候出现异常，那么，A也会创建失败。  
这里有一个Java虚拟机关于类的加载的方式（这一部分了解的不是很透彻）；在加载B类时，发现B类依赖于A类，那么就会先加载A类，然后加载B类；A类加载失败，B类也不会加载成功。  
如果一个类的依赖于其他的类的数量非常的多时，Java虚拟机就可能采用多线程的方式进行加载，采用多线程会造成线程之间的不可预知，导致类加载失败；另一方面，如果类的依赖较多，会造成加载时间长，虚拟机内容占用大等缺点。  
依赖注入的原理是这样的：在B类与A类有依赖时，不使用new的方式去创建，而是使用JavaBean的方式：
    
    
    class A{
    private B b;
    public void setB(B b){
     this.b = b;
    }
    public B getB(){
     return this.b;
    }
    }
    

上面这个修改后的类A，虽然与B类有依赖关系，但是当B类创建失败时，A类依然可以成功的创建。  
可能有人会想，此时b这个对象是一个null，那么在使用的时候会发生空指针异常，这个怎么办？  
接下来就是注入：  
对象b有两个方法，分别是get方法和set方法，在使用之前需要通过set方法传入一个已经实例化的b对象，这样b在使用时，就不会有空指针异常了。  
或者在构造方法中传入。  
但是如果只是单纯的想new一个A对象，现在并没有已经实例的b对象，没发set怎么办？  
接下来就是控制反转：  
A类依赖B类，普通方式是new一个对象出来，这样控制权在使用者身上，谁要使用B类对象就new一个。  
但是spring是使用注入的方式解决依赖，而注入的操作由spring容器来完成，也就是spring容器来set或者构造传入；这样呢控制权就不在使用者身上了，而是转移到了spring容器中。  
这就是控制反转。

## 2.spring中bean的注入方式

spring中Bean有两种注入方式：  
1.set方法  
2.构造方法

## 3.spring中bean的装配方式

spring中Bean有三种装配方式：  
1.xml文件；  
2.注解；  
3.自动装配；

## 4.例子

首先创建一个spring的项目，并且导入需要的jar包：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d0956f8a14899aefa18d96ad2fc7b0e.png)  
其中：bean是实体类的包，client是主方法入口的包，resource是xml文件的包。  
readme是对这个工程简单的介绍，作为备注，否则过一段时间，再次打开这个项目，连这个项目做什么都忘记了，还需要看代码才能知道。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dce004ca7877ea88217febf46908248e.png)
    
    
    package bean;
    
    import java.io.Serializable;
    
    public class APeople implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 4129787061227512007L;
    
    	private GoodPeople goodPeople;
    	
    	private Man man1;
    	
    	private People people2;
    
    	public void say(){
    		System.out.println(this);
    		if(goodPeople != null)
    			goodPeople.say();
    		if(man1 != null)
    			man1.say();
    		if(people2 != null)
    			people2.say();
    	}
    	
    	public GoodPeople getGoodPeople() {
    		return goodPeople;
    	}
    
    	public void setGoodPeople(GoodPeople goodPeople) {
    		this.goodPeople = goodPeople;
    	}
    
    	public Man getMan() {
    		return man1;
    	}
    
    	public void setMan(Man man) {
    		this.man1 = man;
    	}
    
    	public People getPeople() {
    		return people2;
    	}
    
    	public void setPeople(People people) {
    		this.people2 = people;
    	}
    	
    }
    
    
    
    
    package bean;
    
    import java.io.Serializable;
    
    import javax.annotation.Resource;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Repository;
    
    @Repository("goodPeople")
    public class GoodPeople implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 1318010264676836624L;
    
    	@Autowired
    	private Man man;
    	
    	@Resource(name="people1")
    	private People otherPeople;
    	
    	@Resource(name="people2")
    	private People people;
    
    	public void say(){
    		System.out.println(this);
    		man.say();
    		people.say();
    		otherPeople.say();
    	}
    	
    	public GoodPeople(){
    		super();
    		System.out.println("GoodPeople#默认构造");
    	}
    	
    	public GoodPeople(Man man,People people,People otherPeople){
    		this.man = man;
    		this.people = people;
    		this.otherPeople = otherPeople;
    		System.out.println("GoodPeople#全参构造");
    	}
    	
    	public Man getMan() {
    		return man;
    	}
    
    	public void setMan(Man man) {
    		this.man = man;
    	}
    
    	public People getPeople() {
    		return people;
    	}
    
    	public void setPeople(People people) {
    		this.people = people;
    	}
    
    	public People getOtherPeople() {
    		return otherPeople;
    	}
    
    	public void setOtherPeople(People otherPeople) {
    		this.otherPeople = otherPeople;
    	}
    	
    }
    
    
    
    
    package bean;
    
    import java.io.Serializable;
    
    import org.springframework.stereotype.Repository;
    
    @Repository("man")
    public class Man implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 8352563082926893360L;
    
    	private String sex;
    	
    	private People people;
    	
    	public void say(){
    		System.out.println(this);
    		System.out.println(sex);
    		if(people != null){
    			System.out.println(people.getName() == null ? "null" : people
    					.getName());
    			System.out.println(people.getMessage() == null ? "null" : people
    					.getMessage());
    			System.out.println(people.getAge() == null ? "null" : people
    					.getAge());
    		}
    	}
    	
    	public Man(String sex,People people){
    		this.people = people;
    		this.sex = sex;
    		System.out.println("Man#全参构造");
    	}
    	
    	public Man(){
    		super();
    		System.out.println("Man#默认构造");
    	}
    
    	public String getSex() {
    		return sex;
    	}
    
    	public void setSex(String sex) {
    		this.sex = sex;
    	}
    
    	public People getPeople() {
    		return people;
    	}
    
    	public void setPeople(People people) {
    		this.people = people;
    	}
    	
    }
    
    
    
    
    package bean;
    
    import java.io.Serializable;
    
    public class People implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 5213848205077480400L;
    
    	private String name;
    	
    	private String message;
    	
    	private Integer age;
    
    	public People(String name,String message,Integer age){
    		this.name = name;
    		this.message = message;
    		this.age = age;
    		System.out.println("People#全参构造");
    	}
    	
    	public People(){
    		System.out.println("People#默认构造");
    	}
    	
    	public void say(){
    		System.out.println(this);
    		System.out.println(this.name);
    		System.out.println(this.message);
    		System.out.println(this.age);
    	}
    	
    	public String getName() {
    		return name;
    	}
    
    	public void setName(String name) {
    		this.name = name;
    	}
    
    	public String getMessage() {
    		return message;
    	}
    
    	public void setMessage(String message) {
    		this.message = message;
    	}
    
    	public Integer getAge() {
    		return age;
    	}
    
    	public void setAge(Integer age) {
    		this.age = age;
    	}
    	
    }
    
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d959417c5d0908e8d8120b90e1ffedcc.png)
    
    
    package client;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import bean.APeople;
    import bean.GoodPeople;
    import bean.Man;
    import bean.People;
    
    public class Main {
    
    	public static void main(String[] args) {
    		
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    				"resource/*.xml");
    		System.out.println();
    		People people = (People)applicationContext.getBean("people1");
    		people.say();
    		System.out.println();
    		people = (People)applicationContext.getBean("people2");
    		people.say();
    		
    		Man man = (Man)applicationContext.getBean("man");
    		man.say();
    		System.out.println();
    		man = (Man)applicationContext.getBean("man1");
    		man.say();
    		System.out.println();
    		
    		APeople aPeople = (APeople) applicationContext.getBean("apeople");
    		aPeople.say();
    		
    		
    		GoodPeople goodPeople = (GoodPeople) applicationContext
    				.getBean("goodPeople");
    		goodPeople.say();
    		
    	}
    
    }
    
    
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context  
        http://www.springframework.org/schema/context/spring-context.xsd"
    	>
    	<!-- 使用命名空间 -->
    	<context:annotation-config/>
    	<!-- 使用设值注入 -->
    	<bean id="people1" class="bean.People">
    		<property name="name">
    			<value>张三</value>
    		</property>
    		<property name="message">
    			<value>你好，张三</value>
    		</property>
    		<property name="age">
    			<value>22</value>
    		</property>
    	</bean>
    	<!-- 构造注入 -->
    	<bean id="people2" class="bean.People">
    		<constructor-arg name="name">
    			<value>李四</value>
    		</constructor-arg>
    		<constructor-arg name="message">
    			<value>hello</value>
    		</constructor-arg>
    		<constructor-arg name="age">
    			<value>45</value>
    		</constructor-arg>
    	</bean>
    	<!-- 使用注解注入 -->
    	<!-- 
    	<bean id="man" class="bean.Man">
    		<property name="sex">
    			<value>男</value>
    		</property>
    		<property name="people">
    			<ref="people1"/>
    		</property>
    	</bean>
    	 -->
    	 <!-- 使用构造注入（引用其他bean） -->
    	<bean id="man1" class="bean.Man">
    		<property name="sex">
    			<value>人妖</value>
    		</property>
    		<property name="people">
    			<ref bean="people2"/>
    		</property>
    	</bean>
    	<context:component-scan base-package="bean"/>
    	<!-- 自动装配 -->
    	<bean id="apeople" class="bean.APeople" autowire="byName">
    	</bean>
    </beans>
    

运行结果：
    
    
    People#默认构造
    People#全参构造
    Man#默认构造
    GoodPeople#默认构造
    Man#默认构造
    
    bean.People@71a794e5
    张三
    你好，张三
    22
    
    bean.People@76329302
    李四
    hello
    45
    bean.Man@5e25a92e
    null
    
    bean.Man@4df828d7
    人妖
    李四
    hello
    45
    
    bean.APeople@b59d31
    bean.GoodPeople@62fdb4a6
    bean.Man@5e25a92e
    null
    bean.People@76329302
    李四
    hello
    45
    bean.People@71a794e5
    张三
    你好，张三
    22
    bean.Man@5e25a92e
    null
    bean.GoodPeople@62fdb4a6
    bean.Man@5e25a92e
    null
    bean.People@76329302
    李四
    hello
    45
    bean.People@71a794e5
    张三
    你好，张三
    22
    
    

## 5.总结

依赖注入-控制反转  
Bean的装配方式：  
1.基于xml的装配;  
1.1设值注入;  
1.2构造注入;  
2.基于注解的装配;  
3.自动装配;