---
layout: post
title: "【转载】理解scala中的Symbol"
date: 2019-11-19 19:59:10 +0800
categories: [scala, java, 符号字面量, java同名同对象, java快速比较]
description: "本文深入探讨Scala中的Symbol类型，解析其节省内存和快速比较的特性，对比Java中String的intern方法，阐述Symbol在Map类型中的应用优势。"
keywords: scala, java, 符号字面量, java同名同对象, java快速比较
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/103149824
> - 发布时间：2019-11-19 19:59:10
> - 阅读量：349
> - 分类：java同时被 2 个专栏收录, 订阅专栏, scala
> - 标签：#scala, #java, #符号字面量, #java同名同对象, #java快速比较

## 摘要

文章浏览阅读349次。本文深入探讨Scala中的Symbol类型，解析其节省内存和快速比较的特性，对比Java中String的intern方法，阐述Symbol在Map类型中的应用优势。

---

#### 【转载】理解scala中的Symbol

相信很多人和我一样，在刚接触Scala时，会觉得Symbol类型很奇怪，既然Scala中字符串都是不可变的，那么Symbol类型到底有什么作用呢？

简单来说，相比较于String类型，Symbol类型有两个比较明显的特点:节省内存和快速比较。在进入正题之前，让我们先来了解一下Java中String的intern()方法。

**一、String的****intern方法介绍**

Oracle的开发文档上讲解的很详细：String类内部维护一个字符串池(strings pool)，当调用String的intern()方法时，如果字符串池中已经存在该字符串，则直接返回池中字符串引用，如果不存在，则将该字符串添加到池中，并返回该字符串对象的引用。执行过intern()方法的字符串，我们就说这个字符串被拘禁了(interned)。默认情况下，代码中的字符串字面量和字符串常量值都是被拘禁的，例如：
    
    
    String s1 = "abc";
    String s2 =new String("abc");
    

//返回true  
System.out.println(s1 == s2.intern());  

同值字符串的intern()方法返回的引用都相同，例如：
    
    
    String s2 = new String("abc");
    String s3 = new String("abc");
    

//返回true  
System.out.println(s2.intern() == s3.intern());  
//返回false  
System.out.println(s2 == s3);  

**二、****Symbol类型的主要特点**

下面接着介绍Symbol类型的两个特点。

**1\. 节省内存**

在Scala中，Symbol类型的对象是被拘禁的(interned)，任意的同名symbols都指向同一个Symbol对象，避免了因冗余而造成的内存开销。而对于String类型，只有编译时确定的字符串是被拘禁的(interned)。Scala测试代码如下：
    
    
    val s = 'aSymbol
    //输出true
    println( s == 'aSymbol)
    //输出true
    println( s == Symbol("aSymbol"))
    

**2\. 快速比较**

由于Symbol类型的对象是被拘禁的(interned)，任意的同名symbols都指向同一个Symbol对象，而不同名的symbols一定指向不同的Symbol对象，所以symbols对象之间可以使用操作符==快速地进行相等性比较，常数时间内便可以完成，而字符串的equals方法需要逐个字符比较两个字符串，执行时间取决于两个字符串的长度，速度很慢。(实际上，String.equals方法会先比较引用是否相同，但是在运行时产生的字符串对象，引用一般是不同的)

**三、Symbol类型的应用**

Symbol类型一般用于快速比较，例如用于Map类型：Map<Symbol, Data>,根据一个Symbol对象，可以快速查询相应的Data, 而Map<String, Data>的查询效率则低很多。

**四、小结**

利用String的intern方法也可以实现Map<String, Data>的键值快速比较，但是由于需要显式地调用intern()方法，在编码时会造成很多的麻烦，而且如果忘了调用intern()方法，还会造成难以寻找的bug。从这个角度看，Scala的Symbol类型不仅有效率上的提升，而且也简化了编码的复杂度。