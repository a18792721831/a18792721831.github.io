---
layout: post
title: "Java基础--谈谈你对 Java 平台的理解？“Java 是解释执行”，这句话正确吗？"
date: 2020-04-17 18:44:48 +0800
categories: [Java平台, Java组成, Java生态系统, Java本质, 什么是Java]
description: "本文深入探讨Java平台的特点，包括其跨平台能力、垃圾回收机制、JVM内部工作原理及编译执行模式。揭示了Java不仅是解释执行，还通过JIT编译器优化热点代码，提升运行效率。"
keywords: Java平台, Java组成, Java生态系统, Java本质, 什么是Java
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/105585686
> - 发布时间：2020-04-17 18:44:48
> - 阅读量：636
> - 分类：java核心专栏收录该内容, 订阅专栏
> - 标签：#Java平台, #Java组成, #Java生态系统, #Java本质, #什么是Java

## 摘要

文章浏览阅读636次。本文深入探讨Java平台的特点，包括其跨平台能力、垃圾回收机制、JVM内部工作原理及编译执行模式。揭示了Java不仅是解释执行，还通过JIT编译器优化热点代码，提升运行效率。

---

#### java基础-谈谈你对 Java 平台的理解？“Java 是解释执行”，这句话正确吗？

  * 1.典型回答
  * 2.java平台组成

## 1.典型回答

Java 本身是一种面向对象的语言，最显著的特性有两个方面，一是所谓的“**书写一次，到处运行** ”（Write once, run anywhere），能够非常容易地获得跨平台能力；另外就是**垃圾收集** （GC, Garbage Collection），Java 通过垃圾收集器（Garbage Collector）回收分配内存，大部分情况下，程序员不需要自己操心内存的分配和回收。我们日常会接触到 JRE（Java Runtime Environment）或者 JDK（Java Development Kit）。 JRE，也就是 Java 运行环境，包含了 JVM 和 Java 类库，以及一些模块等。而 JDK 可以看作是 JRE 的一个超集，提供了更多工具，比如编译器、各种诊断工具等。对于“Java 是解释执行”这句话，这个说法不太准确。我们开发的 Java 的源代码，首先通过 Javac 编译成为字节码（bytecode），然后，在运行时，通过 Java 虚拟机（JVM）内嵌的解释器将字节码转换成为最终的机器码。但是常见的 JVM，比如我们大多数情况使用的 Oracle JDK 提供的 Hotspot JVM，都提供了 JIT（Just-In-Time）编译器，也就是通常所说的动态编译器，JIT 能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就属于**编译执行** ，而不是**解释执行** 了。

## 2.java平台组成

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25456e096040bc08daa62cea379e31b1.png)  
我们通常把 Java 分为编译期和运行时。这里说的 Java 的编译和 C/C++ 是有着不同的意义的，Javac 的编译，编译 Java 源码生成“.class”文件里面实际是字节码，而不是可以直接执行的机器码。Java 通过字节码和 Java 虚拟机（JVM）这种跨平台的抽象，屏蔽了操作系统和硬件的细节，这也是实现“一次编译，到处执行”的基础。

在运行时，JVM 会通过类加载器（Class-Loader）加载字节码，解释或者编译执行。就像我前面提到的，主流 Java 版本中，如 JDK 8 实际是解释和编译混合的一种模式，即所谓的混合模式（-Xmixed）。通常运行在 server 模式的 JVM，会进行上万次调用以收集足够的信息进行高效的编译，client 模式这个门限是 1500 次。Oracle Hotspot JVM 内置了两个不同的 JIT compiler，C1 对应前面说的 client 模式，适用于对于启动速度敏感的应用，比如普通 Java 桌面应用；C2 对应 server 模式，它的优化是为长时间运行的服务器端应用设计的。默认是采用所谓的分层编译（TieredCompilation）。

Java 虚拟机启动时，可以指定不同的参数对运行模式进行选择。 比如，指定“-Xint”，就是告诉 JVM 只进行解释执行，不对代码进行编译，这种模式抛弃了 JIT 可能带来的性能优势。毕竟解释器（interpreter）是逐条读入，逐条解释运行的。与其相对应的，还有一个“-Xcomp”参数，这是告诉 JVM 关闭解释器，不要进行解释执行，或者叫作最大优化级别。那你可能会问这种模式是不是最高效啊？简单说，还真未必。“-Xcomp”会导致 JVM 启动变慢非常多，同时有些 JIT 编译器优化方式，比如分支预测，如果不进行 profiling，往往并不能进行有效优化。

除了我们日常最常见的 Java 使用模式，其实还有一种新的编译方式，即所谓的 AOT（Ahead-of-Time Compilation），直接将字节码编译成机器代码，这样就避免了 JIT 预热等各方面的开销，比如 Oracle JDK 9 就引入了实验性的 AOT 特性，并且增加了新的 jaotc 工具。利用下面的命令把某个类或者某个模块编译成为 AOT 库。
    
    
    jaotc --output libHelloWorld.so HelloWorld.class
    jaotc --output libjava.base.so --module java.base
    java -XX:AOTLibrary=./libHelloWorld.so,./libjava.base.so HelloWorld
    

另外，JVM 作为一个强大的平台，不仅仅只有 Java 语言可以运行在 JVM 上，本质上合规的字节码都可以运行，Java 语言自身也为此提供了便利，我们可以看到类似 Clojure、Scala、Groovy、JRuby、Jython 等大量 JVM 语言，活跃在不同的场景.
