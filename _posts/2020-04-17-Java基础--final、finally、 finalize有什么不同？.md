---
layout: post
title: "Java基础--final、finally、 finalize有什么不同？"
date: 2020-04-17 19:57:01 +0800
categories: [java 三F, finalize, finnally, final, 可变不可变]
description: "本文详细解析了Java中final、finally、finalize的区别及应用，包括final的不可变性误解、finally的资源回收机制与最佳实践，以及finalize的替代方案。"
keywords: java 三F, finalize, finnally, final, 可变不可变
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/105587239
> - 发布时间：2020-04-17 19:57:01
> - 阅读量：361
> - 分类：java核心专栏收录该内容, 订阅专栏
> - 标签：#java 三F, #finalize, #finnally, #final, #可变不可变

## 摘要

文章浏览阅读361次。本文详细解析了Java中final、finally、finalize的区别及应用，包括final的不可变性误解、finally的资源回收机制与最佳实践，以及finalize的替代方案。

---

#### final、finally、 finalize有什么不同？

  * [1\. 语法和使用实践角度的不同](<#1__1>)
  * [2.final](<#2final_8>)
  * [3.finally](<#3finally_20>)
  * [4.finalize](<#4finalize_22>)
  * [5\. final 不是 immutable](<#5_final__immutable_26>)
  * [6\. finalize 真的那么不堪?](<#6_finalize__35>)
  * [7.有什么机制可以替换 finalize 吗？](<#7_finalize__40>)

## 1\. 语法和使用实践角度的不同

final 可以用来修饰类、方法、变量，分别有不同的意义，final 修饰的 class 代表不可以继承扩展，final 的变量是不可以修改的，而 final 的方法也是不可以重写的（override）。

finally 则是 Java 保证重点代码一定要被执行的一种机制。我们可以使用 try-finally 或者 try-catch-finally 来进行类似关闭 JDBC 连接、保证 unlock 锁等动作。

finalize 是基础类 java.lang.Object 的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize 机制现在已经不推荐使用，并且在 JDK 9 开始被标记为 deprecated。

## 2.final

推荐使用 final 关键字来明确表示我们代码的语义、逻辑意图，这已经被证明在很多场景下是非常好的实践，比如：

我们可以将方法或者类声明为 final，这样就可以明确告知别人，这些行为是不许修改的。

Java 核心类库的定义或源码， 有没有发现 java.lang 包下面的很多类，相当一部分都被声明成为 final class？在第三方类库的一些基础类中同样如此，这可以有效避免 API 使用者更改基础功能，某种程度上，这是保证平台安全的必要手段。

使用 final 修饰参数或者变量，也可以清楚地避免意外赋值导致的编程错误，甚至，有人明确推荐将所有方法参数、本地变量、成员变量声明成 final。

final 变量产生了某种程度的不可变（immutable）的效果，所以，可以用于保护只读数据，尤其是在并发编程中，因为明确地不能再赋值 final 变量，有利于减少额外的同步开销，也可以省去一些防御性拷贝的必要。

final 也许会有性能的好处，很多文章或者书籍中都介绍了可在特定场景提高性能，比如，利用 final 可能有助于 JVM 将方法进行内联，可以改善编译器进行条件编译的能力等等。坦白说，很多类似的结论都是基于假设得出的，比如现代高性能 JVM（如 HotSpot）判断内联未必依赖 final 的提示，要相信 JVM 还是非常智能的。类似的，final 字段对性能的影响，大部分情况下，并没有考虑的必要。

## 3.finally

对于 finally，明确知道怎么使用就足够了。需要关闭的连接等资源，更推荐使用 Java 7 中添加的 try-with-resources 语句，因为通常 Java 平台能够更好地处理异常情况，编码量也要少很多，何乐而不为呢。

## 4.finalize

对于 finalize，我们要明确它是不推荐使用的，业界实践一再证明它不是个好的办法，在 Java 9 中，甚至明确将 Object.finalize() 标记为 deprecated！如果没有特别的原因，不要实现 finalize 方法，也不要指望利用它来进行资源回收。  
为什么呢？简单说，你无法保证 finalize 什么时候执行，执行的是否符合预期。使用不当会影响性能，导致程序死锁、挂起等。  
通常来说，利用上面的提到的 try-with-resources 或者 try-finally 机制，是非常好的回收资源的办法。如果确实需要额外处理，可以考虑 Java 提供的 Cleaner 机制或者其他替代方法。接下来，我来介绍更多设计考虑和实践细节。

## 5\. final 不是 immutable
    
    
     final List<String> strList = new ArrayList<>();
     strList.add("Hello");
     strList.add("world");  
     List<String> unmodifiableStrList = List.of("hello", "world");
     unmodifiableStrList.add("again");
    

final 只能约束 strList 这个引用不可以被赋值，但是 strList 对象行为不被 final 影响，添加元素等操作是完全正常的。如果我们真的希望对象本身是不可变的，那么需要相应的类支持不可变的行为。在上面这个例子中，List.of 方法创建的本身就是不可变 List，最后那句 add 是会在运行时抛出异常的。

## 6\. finalize 真的那么不堪?

finalize 的执行是和垃圾收集关联在一起的，一旦实现了非空的 finalize 方法，就会导致相应对象回收呈现数量级上的变慢，有人专门做过 benchmark，大概是 40~50 倍的下降。  
因为，finalize 被设计成在对象被垃圾收集前调用，这就意味着实现了 finalize 方法的对象是个“特殊公民”，JVM 要对它进行额外处理。finalize 本质上成为了快速回收的阻碍者，可能导致你的对象经过多个垃圾收集周期才能被回收。  
有人也许会问，我用 System.runFinalization​() 告诉 JVM 积极一点，是不是就可以了？也许有点用，但是问题在于，这还是不可预测、不能保证的，所以本质上还是不能指望。实践中，因为 finalize 拖慢垃圾收集，导致大量对象堆积，也是一种典型的导致 OOM 的原因。  
从另一个角度，我们要确保回收资源就是因为资源都是有限的，垃圾收集时间的不可预测，可能会极大加剧资源占用。这意味着对于消耗非常高频的资源，千万不要指望 finalize 去承担资源释放的主要职责，最多让 finalize 作为最后的“守门员”，况且它已经暴露了如此多的问题。这也是为什么我推荐，资源用完即显式释放，或者利用资源池来尽量重用。

## 7.有什么机制可以替换 finalize 吗？

Java 平台目前在逐步使用 java.lang.ref.Cleaner 来替换掉原有的 finalize 实现。Cleaner 的实现利用了幻象引用（PhantomReference），这是一种常见的所谓 post-mortem 清理机制。利用幻象引用和引用队列，我们可以保证对象被彻底销毁前做一些类似资源回收的工作，比如关闭文件描述符（操作系统有限的资源），它比 finalize 更加轻量、更加可靠。  
吸取了 finalize 里的教训，每个 Cleaner 的操作都是独立的，它有自己的运行线程，所以可以避免意外死锁等问题。  
实践中，我们可以为自己的模块构建一个 Cleaner，然后实现相应的清理逻辑。  
注意，从可预测性的角度来判断，Cleaner 或者幻象引用改善的程度仍然是有限的，如果由于种种原因导致幻象引用堆积，同样会出现问题。所以，Cleaner 适合作为一种最后的保证手段，而不是完全依赖 Cleaner 进行资源回收，不然我们就要再做一遍 finalize 的噩梦了。  
很多第三方库自己直接利用幻象引用定制资源收集，比如广泛使用的 MySQL JDBC driver 之一的 mysql-connector-j，就利用了幻象引用机制。幻象引用也可以进行类似链条式依赖关系的动作，比如，进行总量控制的场景，保证只有连接被关闭，相应资源被回收，连接池才能创建新的连接。  
另外，这种代码如果稍有不慎添加了对资源的强引用关系，就会导致循环引用关系，前面提到的 MySQL JDBC 就在特定模式下有这种问题，导致内存泄漏。上面的示例代码中，将 State 定义为 static，就是为了避免普通的内部类隐含着对外部对象的强引用，因为那样会使外部对象无法进入幻象可达的状态。