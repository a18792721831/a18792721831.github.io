---
layout: post
title: "springBean的生命周期"
date: 2019-02-18 20:09:13 +0800
categories: [springBean的生命周期, spring如何创建Bean, springAOP核心实现方法, springBean创建于销毁, 单例与原型Bean的本质区别]
description: "本文深入解析Spring框架中Bean的生命周期，从实例化到销毁的全过程，包括BeanNameAware、BeanFactoryAware、ApplicationContextAware等接口的应用，以及init-method和destroy-method的使用。通过示例代码和运行结果，详细展示了Spring如何管理Bean的生命周期。"
keywords: springBean的生命周期, spring如何创建Bean, springAOP核心实现方法, springBean创建于销毁, 单例与原型Bean的本质区别
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87642498
> - 发布时间：2019-02-18 20:09:13
> - 阅读量：621
> - 分类：java同时被 2 个专栏收录, 订阅专栏, spring
> - 标签：#springBean的生命周期, #spring如何创建Bean, #springAOP核心实现方法, #springBean创建于销毁, #单例与原型Bean的本质区别

## 摘要

文章浏览阅读621次。本文深入解析Spring框架中Bean的生命周期，从实例化到销毁的全过程，包括BeanNameAware、BeanFactoryAware、ApplicationContextAware等接口的应用，以及init-method和destroy-method的使用。通过示例代码和运行结果，详细展示了Spring如何管理Bean的生命周期。

---

#### Bean的生命周期

  * [1.Bean的生命周期的管理](<#1Bean_1>)
  * [2.bean的生命周期](<#2bean_4>)
  * [3.Bean的生命周期的描述](<#3Bean_6>)
  * [4.例子](<#4_18>)
  *     * [4.1创建一个spring的空工程](<#41spring_19>)
    * [4.2创建Java文件](<#42Java_21>)
    * [4.3xml文件](<#43xml_163>)
    * [4.4运行结果](<#44_178>)
  * [5.总结](<#5_200>)

## 1.Bean的生命周期的管理

spring容器可以管理singleton作用域的Bean的生命周期，当Bean的作用域为singleton时，Spring容器能够精确的知道Bean合适被创建，何时初始化完成，何时被销毁。  
对于prototype作用域的bean，spring只负责创建，创建的实例交由客户端代码来管理。

## 2.bean的生命周期

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf9c487507546f70269f1564b4ce11f3.jpeg)

## 3.Bean的生命周期的描述

1.实例化bean；  
2.依赖注入；  
3.如果实现BeanNameAware接口，spring调用setBeanName方法传入id;  
4.如果实现BeanFactoryAware接口，spring调用setBeanFactory方法传入工厂实例的引用；  
5.如果实现ApplicationContextAware接口，spring调用setApplicationContext方法传入ApplicationContext引用；  
6.如果BeanPostProcessor和Bean关联，调用postProcessBeforeInitialzation进行加工，（Spring的AOP就是用此方法实现）；  
7.如果实现InitializingBean接口，spring调用afterPropertiesSet方法；  
8.如果配置文件中指定init-method方法，spring调用配置的方法；  
9.如果BeanPostProcessor和Bean关联，spring调用postProcessAfterInitialization方法；（此时系统可以使用Bean）；  
10如果Bean的作用域是singleton则将Bean放入Spring IOC的缓存池中；如果作用域是prototype,则交给调用者。（spring将对IOC缓存池中的Bean进行生命周期管理）  
11.如果实现DisposableBean接口，spring调用destory方法将Bean销毁；如果在配置文件中指定了destory-method方法，那么会调用指定的方法进行销毁。

## 4.例子

### 4.1创建一个spring的空工程

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b086318e08ba23c344a09e9cea33fd5d.png)

### 4.2创建Java文件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/63998d67b75571a49c49e431f5a54888.png)
    
    
    package bean;
    
    import org.springframework.beans.BeansException;
    import org.springframework.beans.factory.BeanFactory;
    import org.springframework.beans.factory.BeanFactoryAware;
    import org.springframework.beans.factory.BeanNameAware;
    import org.springframework.beans.factory.DisposableBean;
    import org.springframework.beans.factory.InitializingBean;
    import org.springframework.beans.factory.config.BeanPostProcessor;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.ApplicationContextAware;
    
    public class Bean implements DisposableBean, InitializingBean,
    		BeanPostProcessor, BeanNameAware, ApplicationContextAware,
    		BeanFactoryAware {
    
    	private String name;
    	
    	public String getName() {
    		System.out.println("getName");
    		return name;
    	}
    
    	public void setName(String name) {
    		System.out.println("setName");
    		this.name = name;
    	}
    
    	@Override
    	public void setBeanName(String arg0) {
    		System.out.println("BeanNameAware:"+arg0);
    		System.out.println();
    	}
    
    	public void say(){
    		System.out.println("say");
    		System.out.println();
    	}
    	
    	@Override
    	public void setApplicationContext(ApplicationContext arg0)
    			throws BeansException {
    		System.out.println("ApplicationContextAware"+arg0);
    		System.out.println();
    	}
    
    	@Override
    	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
    		System.out.println("BeanFactoryAware"+beanFactory);
    		System.out.println();
    	}
    	
    	@Override
    	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    		System.out.println("postProcessBeforeInitialization:"+bean+beanName);
    		System.out.println();
    		return bean;
    	}
    	
    	@Override
    	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    		System.out.println("postProcessAfterInitialization"+bean+beanName);
    		System.out.println();
    		return bean;
    	}
    
    	@Override
    	public void afterPropertiesSet() throws Exception {
    		System.out.println("afterPropertiesSet");
    		System.out.println();
    	}
    
    	@Override
    	public void destroy() throws Exception {
    		System.out.println("destroy");
    		System.out.println();
    	}
    	
    	public void initMethod(){
    		System.out.println("initMethod");
    		System.out.println();
    	}
    	
    	public void destoryBean(){
    		System.out.println("destoryBean");
    		System.out.println();
    	}
    	
    }
    
    
    
    
    package beanFactory;
    
    import org.springframework.beans.BeansException;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.ApplicationContextAware;
    
    import bean.Bean;
    
    public class BeanFactory implements ApplicationContextAware{
    
    	@Override
    	public void setApplicationContext(ApplicationContext applicationContext)
    			throws BeansException {
    		System.out.println("ApplicationContextAware"+applicationContext);
    		System.out.println();
    	}
    
    	public Bean createBean(){
    		System.out.println("createBean");
    		System.out.println();
    		return new Bean();
    	}
    	
    }
    
    
    
    
    package client;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import bean.Bean;
    
    public class Main {
    
    	public static void main(String[] args) {
    		@SuppressWarnings("resource")
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    				"resource/beans.xml");
    		Bean bean = (Bean) applicationContext.getBean("bean");
    		bean.say();
    	}
    
    }
    
    

### 4.3xml文件

beans.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<bean id="bean" factory-bean="beanFactory" factory-method="createBean"
    	init-method="initMethod" destroy-method="destoryBean" scope="singleton">
    	</bean>
    	<bean id="beanFactory" class="beanFactory.BeanFactory">
    	</bean>
    </beans>
    

### 4.4运行结果
    
    
    ApplicationContextAwareorg.springframework.context.support.ClassPathXmlApplicationContext@6193b845, started on Mon Feb 18 20:07:12 CST 2019
    
    二月 18, 2019 8:07:13 下午 org.springframework.context.support.PostProcessorRegistrationDelegate$BeanPostProcessorChecker postProcessAfterInitialization
    信息: Bean 'beanFactory' of type [beanFactory.BeanFactory] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
    createBean
    
    BeanNameAware:bean
    
    BeanFactoryAwareorg.springframework.beans.factory.support.DefaultListableBeanFactory@4411d970: defining beans [bean,beanFactory]; root of factory hierarchy
    
    ApplicationContextAwareorg.springframework.context.support.ClassPathXmlApplicationContext@6193b845, started on Mon Feb 18 20:07:12 CST 2019
    
    afterPropertiesSet
    
    initMethod
    
    say
    
    
    

## 5.总结

Bean的生命周期  
1.BeanNameAware  
2.BeanFactoryAware  
3.ApplicationContextAware  
4.BeanPostProcessor  
5.InitializingBean#postProcessBeforeInitialization  
6.init-method  
7.BeanPostProcessor#postProcessAfterInitialization  
8.DisposableBean  
9.destorey-method