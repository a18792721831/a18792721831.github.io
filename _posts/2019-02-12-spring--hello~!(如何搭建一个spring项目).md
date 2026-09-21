---
layout: post
title: "spring--hello~!(如何搭建一个spring项目)"
date: 2019-02-12 19:40:50 +0800
categories: [spring入门, spring框架搭建, spring hello word, spring核心jar包, spring项目]
description: "本文详细介绍如何从零开始搭建Spring框架项目，包括环境准备、工程创建、jar包导入及基本操作，适合初学者快速上手。"
keywords: spring入门, spring框架搭建, spring hello word, spring核心jar包, spring项目
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87110126
> - 发布时间：2019-02-12 19:40:50
> - 阅读量：551
> - 分类：spring同时被 2 个专栏收录, 订阅专栏, spring
> - 标签：#spring入门, #spring框架搭建, #spring hello word, #spring核心jar包, #spring项目

## 摘要

文章浏览阅读551次。本文详细介绍如何从零开始搭建Spring框架项目，包括环境准备、工程创建、jar包导入及基本操作，适合初学者快速上手。

---

#### spring入门

  * 1.准备
  * 2.创建工程
  * 3.jar包导入
  * 4.增加自己的操作

## 1.准备

Java–jdk1.8  
详细版本:8u111  
百度 spring jar 找到：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f5fa0559a3ae45fc9d86ca54a90ff414.png)  
点击如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b5a01f5e47c4bf05f07b0795e87bce91.png)  
传送门 ->[点我](<https://repo.spring.io/release/org/springframework/spring/>)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aa735936274718e191b4d37f5a17a0b1.png)  
这个是我自己用的。  
解压如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6c0a8c4f61cb7d67b322b08d66eda082.png)  
这些jar包是必须的，在spring项目中都需要用到：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a78411a080363895a97823a76bf4519.png)  
spring-core是spring框架的核心工具类，spring其他组件都要用到这个包里的类。  
spring-beans：所有应用都要用到的jar包，包含访问配置文件、创建和管理Bean以及进行Ioc或者DI操作相关的类。  
spring-context：提供了在基础Ioc功能上的扩展服务，提供企业级服务，如邮件服务、任务调度、JNDI定位、EJB集成、远程访问、缓存。  
spring-expression定义了spring表达式语言。

spring项目还需要一个第三方的jar----commons.logging![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/115d50b8a86be46b4f914724e46d15b8.png)

## 2.创建工程

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c232752bcc30d75fc4f106cdfcb4cf0f.png)

## 3.jar包导入

将上述jar包拷贝到工程下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a160ae01b7ee7387ad0417c078f5d22a.png)  
选中jar包->右键->bulid path ->add bulid path:  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d6188e78b8a044e7920eb19011df3a91.png)

## 4.增加自己的操作

选中src右键->new ->package:  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/94d9a1a185147d43bf920cddbbf02103.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/72a8901b620475811efeb171f5ce58e4.png)  
说明一下：  
resource放xml文件  
client放测试类（main）  
beans放javaBean  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/47f22bc6e1bd1af3f04a7af7972c7330.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/19525ff9175b8833282ac791bf860a5f.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b2963fd387097da19a89f489a2771bfd.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b4753665eec9882dd3570d87ec622af9.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4267fae7f374bf20a8a66fa436e9ed7b.png)  
会自动生成get,set方法  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44517512fadd5ab9cf04336baae7988c.png)  
注意JavaBean的属性最好不要以set或者get开头，小布尔不要以is开头  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5505698016527a07e0f6eead16711f8c.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df023167ab54c4738ef0efe38edaf519.png)
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<bean id="people" class="beans.People">
    	</bean>
    </beans>
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/942a0919a7361f2061b8dc674be10c45.png)  
注意，这里可以不用写版本：  
因为：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/97776598a7fc70acab08f19fd7a738a3.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/17afb55b44d4087d593550779de7e8db.png)  
在这个文件中指定了默认的配置，所以刚开始可以不用写版本号，直接使用默认的就行：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/372805b07364358b48ac44c37677c75a.png)  
class的路径对不对，一个小的验证方法，按住ctrl左键点击引号内容，如果可以跳转，说明路径正确。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/41d1eec67957daa3d8d84e1e59d9db33.png)
    
    
    package client;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import beans.People;
    
    public class Main {
    
    	public static void main(String[] args) {
    
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    				"resource/beans.xml");
    		People people = (People) applicationContext.getBean("people");
    		people.setAge(33);
    		people.setName("spring");
    		people.setMessage("hello");
    		System.out.println("name:"+people.getName());
    		System.out.println("age:"+people.getAge());
    		System.out.println("message:"+people.getMessage());
    	}
    
    }
    
    
    
    

运行：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/474db159a04fbe705cb0e994385fa250.png)  
注意：运行之前需要关闭.xml文件，否则因为文件占用，会重新创建一个xml文件。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b9f32d851bbbf4aacaa25d899e2408c2.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bc9843323fef4e725a600be4c9dd9746.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/28d39b7de6f9294aa3bc9b7f8b40b74c.png)  
选中main方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3fdfb6a45302da858095aada4b0a8c88.png)

这个spring hello就完成了。  
1.运行文件时，xml文件不能被占用，不能用eclipse打开xml文件  
2.spring和jdk有版本对应关系
