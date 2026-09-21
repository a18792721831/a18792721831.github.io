---
layout: post
title: "spring AOP介绍"
date: 2019-02-19 19:05:51 +0800
categories: [springAOP, AOP术语, AOP流行的框架, 三种编程思想, 什么是AOP]
description: "Spring AOP介绍1.什么是AOP2.AOP框架3.AOP术语1.什么是AOPAspect-Oriented Programming:面向切面编程。面向过程编程：C语言，适合算法等；面向对象编程：绝大数编程语言，抽象，封装；面向切面编程：大型，超大型企业级应用，封装业务，逻辑等；AOP的全称是Aspect-Oriented Programming,面向切面编程。面向切面编程是对..._以下关于spring aop的介绍错误的是"
keywords: springAOP, AOP术语, AOP流行的框架, 三种编程思想, 什么是AOP
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87726859
> - 发布时间：2019-02-19 19:05:51
> - 阅读量：495
> - 分类：java同时被 2 个专栏收录, 订阅专栏, spring
> - 标签：#springAOP, #AOP术语, #AOP流行的框架, #三种编程思想, #什么是AOP

## 摘要

文章浏览阅读495次。Spring AOP介绍1.什么是AOP2.AOP框架3.AOP术语1.什么是AOPAspect-Oriented Programming:面向切面编程。面向过程编程：C语言，适合算法等；面向对象编程：绝大数编程语言，抽象，封装；面向切面编程：大型，超大型企业级应用，封装业务，逻辑等；AOP的全称是Aspect-Oriented Programming,面向切面编程。面向切面编程是对..._以下关于spring aop的介绍错误的是

---

#### Spring AOP介绍

  * [1.什么是AOP](<#1AOP_1>)
  * [2.AOP框架](<#2AOP_13>)
  * [3.AOP术语](<#3AOP_19>)

## 1.什么是AOP

Aspect-Oriented Programming:面向切面编程。

面向过程编程：C语言，适合算法等；  
面向对象编程：绝大数编程语言，抽象，封装；  
面向切面编程：大型，超大型企业级应用，封装业务，逻辑等；

AOP的全称是Aspect-Oriented Programming,面向切面编程。  
面向切面编程是对面向对象编程的一种补充，也是一种比较成熟的编程方式。

面向切面编程主要可以解决业务处理中冗余的代码，比如日志记录代码，在类似的类中，记录日志的代码是相同的，如果不采用面向切面编程，那么这些相同的代码需要在每一个类中进行写。  
但是如果采用面向切面编程，那么，这些相同的代码只需要编写一次即可。

## 2.AOP框架

目前流行的AOP框架有两个：  
1.springAOP  
2.AspectJ  
spring AOP使用纯Java实现，运行期间通过代理织入。  
AspectJ是基于Java实现的AOP框架。

## 3.AOP术语

Aspect-切面：系统横向业务，逻辑；  
Joinpoint-连接点：方法的调用或者异常的抛出；  
Pointcut-切入点：类名或者方法名；  
Advice-通知、增强处理：横向的业务，逻辑代码；  
Target Object - 目标对象：需要增加横向业务，逻辑的对象；  
Proxy-代理：将横向代码应用到目标对象后组装的新的动态创建的对象；  
Weaving-植入：Proxy的过程；