---
layout: post
title: "Java基础--int和Integer有什么区别"
date: 2020-04-18 17:07:11 +0800
categories: [Java基本类型与包装类型, 自动装箱与自动拆箱, 基本类型的线程安全, 自动装箱与拆箱源码分析, 原始类型与引用类型]
description: "int和Integer有什么区别1.区别2.自动装箱、拆箱3.自动装箱、拆箱源码4.原始类型线程安全5. 原始数据类型和引用类型局限性1.区别int 是我们常说的整形数字，是 Java 的 8 个原始数据类型（Primitive Types，boolean、byte 、short、char、int、float、double、long）之一。Java 语言虽然号称一切都是对象，但原始数据类型是例..._java int和integer能塞0么"
keywords: Java基本类型与包装类型, 自动装箱与自动拆箱, 基本类型的线程安全, 自动装箱与拆箱源码分析, 原始类型与引用类型
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/105600023
> - 发布时间：2020-04-18 17:07:11
> - 阅读量：1
> - 分类：java核心专栏收录该内容, 订阅专栏
> - 标签：#Java基本类型与包装类型, #自动装箱与自动拆箱, #基本类型的线程安全, #自动装箱与拆箱源码分析, #原始类型与引用类型

## 摘要

文章浏览阅读1k次。int和Integer有什么区别1.区别2.自动装箱、拆箱3.自动装箱、拆箱源码4.原始类型线程安全5. 原始数据类型和引用类型局限性1.区别int 是我们常说的整形数字，是 Java 的 8 个原始数据类型（Primitive Types，boolean、byte 、short、char、int、float、double、long）之一。Java 语言虽然号称一切都是对象，但原始数据类型是例..._java int和integer能塞0么

---

#### int和Integer有什么区别

  * 1.区别
  * 2.自动装箱、拆箱
  * 3.自动装箱、拆箱源码
  * 4.原始类型线程安全
  * 5\. 原始数据类型和引用类型局限性

## 1.区别

int 是我们常说的整形数字，是 Java 的 8 个原始数据类型（Primitive Types，boolean、byte 、short、char、int、float、double、long）之一。Java 语言虽然号称一切都是对象，但原始数据类型是例外。

Integer 是 int 对应的包装类，它有一个 int 类型的字段存储数据，并且提供了基本操作，比如数学运算、int 和字符串之间转换等。在 Java 5 中，引入了自动装箱和自动拆箱功能（boxing/unboxing），Java 可以根据上下文，自动进行转换，极大地简化了相关编程。

关于 Integer 的值缓存，这涉及 Java 5 中另一个改进。构建 Integer 对象的传统方式是直接调用构造器，直接 new 一个对象。但是根据实践，我们发现大部分数据操作都是集中在有限的、较小的数值范围，因而，在 Java 5 中新增了静态工厂方法 valueOf，在调用它的时候会利用一个缓存机制，带来了明显的性能改进。按照 Javadoc，这个值默认缓存是 -128 到 127 之间。

## 2.自动装箱、拆箱

自动装箱实际上算是一种语法糖。什么是语法糖？可以简单理解为 Java 平台为我们自动进行了一些转换，保证不同的写法在运行时等价，它们发生在编译阶段，也就是生成的字节码是一致的。

像前面提到的整数，javac 替我们自动把装箱转换为 Integer.valueOf()，把拆箱替换为 Integer.intValue()，这似乎这也顺道回答了另一个问题，既然调用的是 Integer.valueOf，自然能够得到缓存的好处啊。

做个小实验：基于jdk8环境  
java源码：
    
    
    public class MyInteger {
    
        public static void main(String[] args){
            Integer integer = 1;
            int in = integer ++;
        }
    
    }
    

然后使用javac进行编译  
得到Java源码的字节码：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de423863a0a0ad5ddbe22cf930230926.png)  
使用JD GUI进行反编译，得到反编译后的代码，发现：  
1.对于值1进行自动装箱，得到value1  
2.将自动装箱得到的value1赋值给我们的目标变量interge（引用）  
3.自动拆箱2中的临时变量，赋值给临时引用，加1后得到的基本类型再次装箱，并赋值给原本自身  
4.将临时引用自动拆箱，赋值给in基本变量

可以看到原本2条语句，执行的3个操作：  
1.integer = 1  
2.in = integer  
3.interger++  
反编译后，进行了多次自动拆箱，自动装箱。还使用了临时引用变量(实现++)  
继续使用javap进行反编译：
    
    
    Classfile /G:/studyjdk/study/src/com/study/myinteger/MyInteger.class
      Last modified 2020-4-18; size 411 bytes
      MD5 checksum d7a54fc38353e1c163dcb19d4da14c1f
      Compiled from "MyInteger.java"
    public class com.study.myinteger.MyInteger
      minor version: 0
      major version: 52
      flags: ACC_PUBLIC, ACC_SUPER
    Constant pool:
       #1 = Methodref          #5.#14         // java/lang/Object."<init>":()V
       #2 = Methodref          #15.#16        // java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       #3 = Methodref          #15.#17        // java/lang/Integer.intValue:()I
       #4 = Class              #18            // com/study/myinteger/MyInteger
       #5 = Class              #19            // java/lang/Object
       #6 = Utf8               <init>
       #7 = Utf8               ()V
       #8 = Utf8               Code
       #9 = Utf8               LineNumberTable
      #10 = Utf8               main
      #11 = Utf8               ([Ljava/lang/String;)V
      #12 = Utf8               SourceFile
      #13 = Utf8               MyInteger.java
      #14 = NameAndType        #6:#7          // "<init>":()V
      #15 = Class              #20            // java/lang/Integer
      #16 = NameAndType        #21:#22        // valueOf:(I)Ljava/lang/Integer;
      #17 = NameAndType        #23:#24        // intValue:()I
      #18 = Utf8               com/study/myinteger/MyInteger
      #19 = Utf8               java/lang/Object
      #20 = Utf8               java/lang/Integer
      #21 = Utf8               valueOf
      #22 = Utf8               (I)Ljava/lang/Integer;
      #23 = Utf8               intValue
      #24 = Utf8               ()I
    {
      public com.study.myinteger.MyInteger();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
          stack=1, locals=1, args_size=1
             0: aload_0
             1: invokespecial #1                  // Method java/lang/Object."<init>":()V
             4: return
          LineNumberTable:
            line 7: 0
    
      public static void main(java.lang.String[]);
        descriptor: ([Ljava/lang/String;)V
        flags: ACC_PUBLIC, ACC_STATIC
        Code:
          stack=2, locals=5, args_size=1
             0: iconst_1
             1: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
             4: astore_1
             5: aload_1
             6: astore_3
             7: aload_1
             8: invokevirtual #3                  // Method java/lang/Integer.intValue:()I
            11: iconst_1
            12: iadd
            13: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
            16: dup
            17: astore_1
            18: astore        4
            20: aload_3
            21: invokevirtual #3                  // Method java/lang/Integer.intValue:()I
            24: istore_2
            25: return
          LineNumberTable:
            line 10: 0
            line 11: 5
            line 12: 25
    }
    SourceFile: "MyInteger.java"
    

可以得出：  
自动装箱调用Integer.valueOf()方法  
自动拆箱调用Integer.intValue()方法  
(通过GD GUI反编译也能够看出)

那么，如果都是基本类型，相同的操作，其编译和反编译后的区别有多大？  
源码：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6f96ef55d6ede26d068086b05dde75e4.png)  
通过JD GUI反编译：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ac7e170695cb036f7185a11058e8aec6.png)  
通过Javap反编译
    
    
    Classfile /G:/studyjdk/study/src/com/study/myinteger/MyInteger.class
      Last modified 2020-4-18; size 298 bytes
      MD5 checksum 6178c8b915c04575fa90ab4900a9d6b4
      Compiled from "MyInteger.java"
    public class com.study.myinteger.MyInteger
      minor version: 0
      major version: 52
      flags: ACC_PUBLIC, ACC_SUPER
    Constant pool:
       #1 = Methodref          #3.#12         // java/lang/Object."<init>":()V
       #2 = Class              #13            // com/study/myinteger/MyInteger
       #3 = Class              #14            // java/lang/Object
       #4 = Utf8               <init>
       #5 = Utf8               ()V
       #6 = Utf8               Code
       #7 = Utf8               LineNumberTable
       #8 = Utf8               main
       #9 = Utf8               ([Ljava/lang/String;)V
      #10 = Utf8               SourceFile
      #11 = Utf8               MyInteger.java
      #12 = NameAndType        #4:#5          // "<init>":()V
      #13 = Utf8               com/study/myinteger/MyInteger
      #14 = Utf8               java/lang/Object
    {
      public com.study.myinteger.MyInteger();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
          stack=1, locals=1, args_size=1
             0: aload_0
             1: invokespecial #1                  // Method java/lang/Object."<init>":()V
             4: return
          LineNumberTable:
            line 7: 0
    
      public static void main(java.lang.String[]);
        descriptor: ([Ljava/lang/String;)V
        flags: ACC_PUBLIC, ACC_STATIC
        Code:
          stack=1, locals=3, args_size=1
             0: iconst_1
             1: istore_1
             2: iload_1
             3: iinc          1, 1
             6: istore_2
             7: return
          LineNumberTable:
            line 10: 0
            line 11: 2
            line 12: 7
    }
    SourceFile: "MyInteger.java"
    

完全没有自动装箱和自动拆箱。

基于此，我们似乎可以得出：  
基本类型的++操作和装箱类型的++操作的处理是不同的。

换句话说：  
JVM支持基本类型的++操作(应该是CPU实现ADD操作，汇编中有直接的指令实现ADD)  
JVM不支持装箱类型的++操作，因为装箱类型实际上是对象。CPU可以实现ADD数值，但是不能ADD对象的属性值。

## 3.自动装箱、拆箱源码

还有一个问题：**-128~127是怎么来的**  
我们已经知道了自动装箱的方法是valueOf()  
那么valueOf的方法的源码是怎么样的？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/62e670f8c17004f067e073eb35487d38.png)  
Integer是final类，不可扩展  
继承Number类  
Number类是一个抽象类，定义了包装类到基本类型转换的所有方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5bee127d047f92e4d96b4bf8db42c469.png)  
当然也实现了序列化接口。  
Integer还实现了Comparable接口，实现了Comparable接口，意味着，一些数组等的排序，对于包装类也可以使用(各个包装类之间可能存在不同)  
Integer数据存储范围：  
-2^31  
2^31  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/12c9bc1fd72bd04765ba812ab56b3cfa.png)  
32位长度，最高为符号。  
0x8000 0000 <=> 0b 1000 0000 0000 0000 0000 0000 0000 0000  
0x7fff ffff <=> 0b 0111 1111 1111 1111 1111 1111 1111 1111  
接下来看下valueOf方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1592d9cfce081ba33f5db6aadd288223.png)  
在Integer内部有一个私有类  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/680f012c461c2f567f9160132d3704cf.png)  
也就是说，Integer默认范围是-128 ~ 127,其在内部实现了-128 ~ 127之间包装类数据的缓存。

## 4.原始类型线程安全

基本类型不是线程安全的。  
举个常见的例子。  
多线程计数：不安全：
    
    
    public class MyInteger {
    
        int sum;
    
        public static void main(String[] args) throws InterruptedException {
    
            MyInteger myInteger = new MyInteger();
            Thread thread1 = new Thread(myInteger.new MyRunnable());
            Thread thread2 = new Thread(myInteger.new MyRunnable());
            Thread thread3 = new Thread(myInteger.new MyRunnable());
            Thread thread4 = new Thread(myInteger.new MyRunnable());
            thread1.start();
            thread2.start();
            thread3.start();
            thread4.start();
            thread1.join();
            thread2.join();
            thread3.join();
            thread4.join();
            //目标是4W
            System.out.println(myInteger.sum);
    
        }
    
        class MyRunnable implements Runnable{
    
            @Override
            public void run() {
                for(int i = 0;i < 10000;i++){
                    sum++;
                }
            }
        }
    
    }
    

每次输出结果都不同，但是都不会等于4W。  
在来看个线程安全的：
    
    
    public class MyInteger1 {
    
        AtomicInteger sum = new AtomicInteger();
    
        public static void main(String[] args) throws InterruptedException {
    
            MyInteger1 myInteger1 = new MyInteger1();
            Thread thread1 = new Thread(myInteger1.new MyRunnable());
            Thread thread2 = new Thread(myInteger1.new MyRunnable());
            Thread thread3 = new Thread(myInteger1.new MyRunnable());
            Thread thread4 = new Thread(myInteger1.new MyRunnable());
            thread1.start();
            thread2.start();
            thread3.start();
            thread4.start();
            thread1.join();
            thread2.join();
            thread3.join();
            thread4.join();
            //目标是4W
            System.out.println(myInteger1.sum);
    
        }
    
        class MyRunnable implements Runnable{
    
            @Override
            public void run() {
                for(int i = 0;i < 10000;i++){
                    sum.incrementAndGet();
                }
            }
        }
        
    }
    

这个每次执行都必须是4W  
**那么，使用基本类型能否实现线程安全呢？**
    
    
    public class MyInteger2 {
    
    
        volatile int sum = 0;
        AtomicIntegerFieldUpdater<MyInteger2> updater = AtomicIntegerFieldUpdater.newUpdater(MyInteger2.class, "sum");
        public static void main(String[] args) throws InterruptedException {
    
            MyInteger2 myInteger2 = new MyInteger2();
            Thread thread1 = new Thread(myInteger2.new MyRunnable());
            Thread thread2 = new Thread(myInteger2.new MyRunnable());
            Thread thread3 = new Thread(myInteger2.new MyRunnable());
            Thread thread4 = new Thread(myInteger2.new MyRunnable());
            thread1.start();
            thread2.start();
            thread3.start();
            thread4.start();
            thread1.join();
            thread2.join();
            thread3.join();
            thread4.join();
            //目标是4W
            System.out.println(myInteger2.sum);
    
        }
    
        class MyRunnable implements Runnable{
    
            @Override
            public void run() {
                for(int i = 0;i < 10000;i++){
                    updater.incrementAndGet(MyInteger2.this);
                }
            }
        }
    
    }
    

其输出也是4W  
使用内存可见，以及使用update进行更新。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a20f8473e51d3147e638a48e4a20e2d6.png)  
其内部是使用死循环的CAS实现的。  
所以，基本类型多线程可能存在以下问题：

  * 原始数据类型的变量，显然要使用并发相关手段，才能保证线程安全，这些我会在专栏后面的并发主题详细介绍。如果有线程安全的计算需要，建议考虑使用类似 AtomicInteger、AtomicLong 这样的线程安全类。
  * 特别的是，部分比较宽的数据类型，比如 float、double，甚至不能保证更新操作的原子性，可能出现程序读取到只更新了一半数据位的数值！

## 5\. 原始数据类型和引用类型局限性

  * 原始数据类型和 Java 泛型并不能配合使用  
这是因为 Java 的泛型某种程度上可以算作伪泛型，它完全是一种编译期的技巧，Java 编译期会自动将类型转换为对应的特定类型，这就决定了使用泛型，必须保证相应类型可以转换为 Object。
  * 无法高效地表达数据，也不便于表达复杂的数据结构，比如 vector 和 tuple  
我们知道 Java 的对象都是引用类型，如果是一个原始数据类型数组，它在内存里是一段连续的内存，而对象数组则不然，数据存储的是引用，对象往往是分散地存储在堆的不同位置。这种设计虽然带来了极大灵活性，但是也导致了数据操作的低效，尤其是无法充分利用现代 CPU 缓存机制。Java 为对象内建了各种多态、线程安全等方面的支持，但这不是所有场合的需求，尤其是数据处理重要性日益提高，更加高密度的值类型是非常现实的需求。

原始数据类型和 Java 泛型并不能配合使用，也就是Primitive Types 和Generic 不能混用，于是JAVA就设计了这个auto-boxing/unboxing机制，实际上就是primitive value 与 object之间的隐式转换机制，否则要是没有这个机制，开发者就必须每次手动显示转换，那多麻烦是不是？但是primitive value 与 object各自有各自的优势，primitive value在内存中存的是值，所以找到primitive value的内存位置，就可以获得值；不像object存的是reference，找到object的内存位置，还要根据reference找下一个内存空间，要产生更多的IO，所以计算性能比primitive value差，但是object具备generic的能力，更抽象，解决业务问题编程效率高。于是JAVA设计者的初衷估计是这样的：如果开发者要做计算，就应该使用primitive value如果开发者要处理业务问题，就应该使用object，采用Generic机制；反正JAVA有auto-boxing/unboxing机制，对开发者来讲也不需要注意什么。然后为了弥补object计算能力的不足，还设计了static valueOf()方法提供缓存机制，算是一个弥补。

Java对象内存结构：

  * 基本数据类型
  * 对象类型 
    * 对象头（Header） 
      * MarkWord，4字节
      * Class对象指针，4字节
    * 实例数据（Instance Data）
    * 对齐数据（Padding）, 按8个字节对齐
  * 数组类型  
-对象头（Header）  
\- MarkWord，4字节  
\- Class对象指针，4字节 
    * 数组长度，4字节
    * 实例数据（Instance Data）
    * 对齐数据（Padding）, 按8个字节对齐  
如何获取对象大小：
  * Instrumentation + premain实现工具类：Instrumentation.getObjectSize()来获取
  * Unsafe，Unsafe.objectFieldOffset()来获取
