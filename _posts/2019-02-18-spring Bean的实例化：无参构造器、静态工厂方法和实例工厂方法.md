---
layout: post
title: "spring Bean的实例化：无参构造器、静态工厂方法和实例工厂方法"
date: 2019-02-18 19:36:10 +0800
categories: [springBean的实例化方式, 无参构造器实例化, 静态工程方法实例化, 实例工厂实例化, 实例化之间的区别]
description: "本文深入解析Spring框架中Bean的三种实例化方式：无参构造器实例化、静态工厂方法实例化及实例工厂方法实例化。通过示例代码和XML配置展示每种方式的具体应用，并总结了各自的使用场景。"
keywords: springBean的实例化方式, 无参构造器实例化, 静态工程方法实例化, 实例工厂实例化, 实例化之间的区别
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87640342
> - 发布时间：2019-02-18 19:36:10
> - 阅读量：2
> - 分类：java同时被 2 个专栏收录, 订阅专栏, spring
> - 标签：#springBean的实例化方式, #无参构造器实例化, #静态工程方法实例化, #实例工厂实例化, #实例化之间的区别

## 摘要

文章浏览阅读2.9k次。本文深入解析Spring框架中Bean的三种实例化方式：无参构造器实例化、静态工厂方法实例化及实例工厂方法实例化。通过示例代码和XML配置展示每种方式的具体应用，并总结了各自的使用场景。

---

#### Bean的实例化

  * 1.spring的三种实例化方式
  * 2.例子
  *     * 2.1创建一个空的spring工程
    * 2.2新建Java类
    * 2.3xml文件
    * 2.4运行结果
  * 3.总结

## 1.spring的三种实例化方式

1.默认无参的构造器实例化  
2.静态工厂方法实例化  
3.实例工厂方法实例化  
构造器实例化是指spring容器通过Bean对应类中默认的无参构造方法来实例化。  
静态工厂要求开发者创建一个静态工厂的方法来创建Bean的实例，器Bean配置中的class属性所指定的不在是Bean实例的实现类，而是静态工程类，同时还需要使用factory-method属性来制定所创建的静态工程方法。  
实例工厂不在使用静态方法创建Bean实例，而是采用直接创建Bean实例的方式；在配置文件中，需要实例化的Bean也不是通过class属性直接指向的实例化类，而是通过factory-bean属性指向配置的实例工厂，然后使用factory-method属性确定使用工厂中的哪个方法。

## 2.例子

### 2.1创建一个空的spring工程

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f739f37e162ef7c186bb42061210e075.png)

### 2.2新建Java类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e7419b037fc2877eb9adeb4090850380.png)
    
    
    package client;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import domain.def.DefaultBean;
    import domain.inst.InstanceBean;
    import domain.stat.StaticBean;
    
    public class Main {
    
    	@SuppressWarnings("resource")
    	public static void main(String[] args) {
    
    		String defaultPath = "default";
    		String staticPath = "static";
    		String instancePath = "instance";
    		String before = "resource/beans-";
    		String after = ".xml";
    		
    		ApplicationContext applicationContext = null;
    		
    		applicationContext = new ClassPathXmlApplicationContext(before
    				+ defaultPath + after);
    		DefaultBean defaultBean = (DefaultBean) applicationContext
    				.getBean(defaultPath + "Bean");
    		System.out.println(defaultBean);
    		
    		applicationContext = new ClassPathXmlApplicationContext(before
    				+ staticPath + after);
    		StaticBean staticBean = (StaticBean) applicationContext
    				.getBean(staticPath + "Bean");
    		System.out.println(staticBean);
    		
    		applicationContext = new ClassPathXmlApplicationContext(before
    				+ instancePath + after);
    		InstanceBean instanceBean = (InstanceBean) applicationContext
    				.getBean(instancePath + "Bean");
    		System.out.println(instanceBean);
    		
    	}
    
    }
    
    
    
    
    package domain.def;
    
    public class DefaultBean {
    
    }
    
    
    
    
    package domain.inst;
    
    public class InstanceBean {
    
    }
    
    
    
    
    package domain.inst.factory;
    
    import domain.inst.InstanceBean;
    
    public class InstanceBeanFactory {
    
    	public InstanceBean createInstanceBean(){
    		return new InstanceBean();
    	}
    	
    }
    
    
    
    
    package domain.stat;
    
    public class StaticBean {
    
    }
    
    
    
    
    package domain.stat.factory;
    
    import domain.stat.StaticBean;
    
    public class StaticBeanFactory {
    
    	public static StaticBean createStaticBean(){
    		return new StaticBean();
    	}
    	
    }
    
    

### 2.3xml文件

beans-default.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<bean id="defaultBean" class="domain.def.DefaultBean"></bean>
    </beans>
    

beans-instance.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<bean id="instanceBeanFactory" 
    	class="domain.inst.factory.InstanceBeanFactory"></bean>
    	<bean id="instanceBean" factory-bean="instanceBeanFactory"
    	factory-method="createInstanceBean"></bean>
    </beans>
    

beans-static.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd">
    	<bean id="staticBean" 
    	class="domain.stat.factory.StaticBeanFactory"
    	factory-method="createStaticBean"></bean>
    </beans>
    

### 2.4运行结果
    
    
    domain.def.DefaultBean@ae45eb6
    domain.stat.StaticBean@4dfa3a9d
    domain.inst.InstanceBean@6500df86
    

## 3.总结

spring的三种实例化方式  
1.默认无参的构造器实例化  
2.静态工厂方法实例化  
3.实例工厂方法实例化
