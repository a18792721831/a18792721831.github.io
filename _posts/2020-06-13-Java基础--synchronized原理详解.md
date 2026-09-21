---
layout: post
title: "Java基础--synchronized原理详解"
date: 2020-06-13 13:19:39 +0800
categories: [HappenBefore原则, 锁存在的意义, synchronized原理, synchrionzed使用, synchronized优化]
description: "本文深入探讨Java中的synchronized关键字，涵盖其多线程特性、锁的定义与意义、synchronized的使用场景及原理，对比类对象与实例对象的锁区别，分析synchronized的缺陷与优化，以及其内部处理过程。"
keywords: HappenBefore原则, 锁存在的意义, synchronized原理, synchrionzed使用, synchronized优化
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/106589671
> - 发布时间：2020-06-13 13:19:39
> - 阅读量：570
> - 分类：java核心同时被 3 个专栏收录, 订阅专栏, Java8, 多线程
> - 标签：#HappenBefore原则, #锁存在的意义, #synchronized原理, #synchrionzed使用, #synchronized优化

## 摘要

文章浏览阅读570次。本文深入探讨Java中的synchronized关键字，涵盖其多线程特性、锁的定义与意义、synchronized的使用场景及原理，对比类对象与实例对象的锁区别，分析synchronized的缺陷与优化，以及其内部处理过程。

---

#### Java基础--synchronized原理详解

  * [1\. 多线程特性](<#1__1>)
  *     * [1.1 原子性(Atomicity)](<#11_Atomicity_2>)
    * [1.2 可见性(Visibility)](<#12_Visibility_4>)
    * [1.3 有序性(Ordering)](<#13_Ordering_6>)
    * [1.4 Happen-Before原则](<#14_HappenBefore_34>)
  * [2\. 锁定义](<#2__44>)
  *     * [2.1 为什么需要锁](<#21__45>)
    * [2.2 锁存在的意义](<#22__154>)
  * [3\. synchronized](<#3_synchronized_159>)
  *     * [3.1 synchronized的使用场景](<#31_synchronized_160>)
    * [3.2 synchronized原理](<#32_synchronized_168>)
    *       * [3.2.1 Java对象在JVM中的结构](<#321_JavaJVM_172>)
      * [3.2.2 monitor指令](<#322_monitor_182>)
      * [3.2.3 monitor指令过程](<#323_monitor_192>)
  * [4\. synchronized 对类对象和实例对象的区别](<#4_synchronized__198>)
  *     * [4.1 static修饰和没有static修饰的区别](<#41_staticstatic_199>)
    * [4.2 synchronized 不同使用场景](<#42_synchronized__288>)
    * [4.3 不使用synchronized 同步](<#43_synchronized__291>)
    * [4.4 synchronized 同步代码块--类对象](<#44_synchronized__357>)
    * [4.5 synchronized 同步代码块--实例对象](<#45_synchronized__367>)
    * [4.6 synchronized 同步代码块--任意实例对象](<#46_synchronized__373>)
    * [4.7 synchronized 同步方法--类方法](<#47_synchronized__385>)
    * [4.8 synchronized 同步方法--实例方法](<#48_synchronized__397>)
  * [5\. synchronized 的缺陷](<#5_synchronized__407>)
  * [6\. synchronized的锁处理](<#6_synchronized_418>)
  * [7\. synchronized 处理过程](<#7_synchronized__438>)

## 1\. 多线程特性

### 1.1 原子性(Atomicity)

原子性是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一单开始，就不会被其他线程干扰。

### 1.2 可见性(Visibility)

可见性是指当一个线程修改了某一个共享变量的值，其他线程是否能够立即知道这个修改。

### 1.3 有序性(Ordering)

程序在执行时，编译器可能会进行指令重排，重排后的指令原指令的顺序未必一致。  
**指令重排可以保证串行语义一致，但是没有义务保证多线程间的语义也一致。**  
为什么要进行指令重排？  
CPU执行指令，需要进行这几步(不同的指令集可能不同，一般认知中是这样的)

  * 取指
  * 译码和取操作数
  * 执行或计算
  * 存储器访问
  * 写回  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a800ee659db54dffe523f3a0dac4d9eb.png)  
CPU执行是一个一个的周期进行的，CPU组成中需要有晶振，晶振产生固定频率的脉冲，每一次脉冲就是一次时钟周期。CPU的一个时钟周期内只能进行一个操作。  
那么完成一次指令需要5个时钟周期。  
仔细观察这5个操作，分别都是CPU不同的区域。所以，在一个时钟周期内，可以进行多个不同的操作。  
比如：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f25f7c16cc251343287c30561cd89036.png)  
在上图中执行了3条指令，如果是串行的，那么需要15个时钟周期才能执行完成。  
这就是CPU执行指令流水线执行，指令的执行效率高。  
CPU流水线执行指令，虽然效率高，但是依然存在问题。  
假设蓝色的指令计算的数据，依赖绿色指令的计算结果，在第5个时钟周期进行计算时，绿色的计算结果，还没有写到寄存器，此时就需要蓝色指令等待绿色指令的计算结果写入寄存器才能继续进行。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec779e069d38959449f1086ab4d363f5.png)  
发现因为蓝色需要等待绿色指令执行完毕，才能执行蓝色指令。  
但是在代码逻辑中，绿色后面就是蓝色，而蓝色后面是橙色。  
如果橙色和蓝色没有强烈的先后关系，那么可以调整指令执行顺序。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5a30dff6b419aae957ed9d6cacfb529f.png)  
就可以避免CPU指令执行的中断停顿。  
指令重排提高了CPU执行效率，但是也带来了指令乱序的问题。  
相比之下，指令乱序的问题是可以接受的。

### 1.4 Happen-Before原则

Happen-Before原则是不进行指令重排的规则：

  * 程序顺序原则：一个线程内保证语义的串行性
  * volatile规则：volatile变量的写，先发生于读，这保证了volatile变量的可见性
  * 锁规则：解锁(unlock)先发生于加锁(lock)
  * 传递性：A先于B，B先于C，那么A一定先于C
  * 线程的start方法先于线程的任务
  * 线程的任务先于线程的终止
  * 线程的中断先于中断前的代码
  * 对象的构造函数先于对象finalize方法

## 2\. 锁定义

### 2.1 为什么需要锁

因为在CPU执行指令的时候，会进行指令重排，指令重排在串行上可以保证程序语义一致，但是在多线程情况下，就无法保证语义一致了。  
举个例子：  
一个全局变量，每个线程对全局变量进行1w次自增操作。如果有10个线程，那么最终的全局变量的值应该是10W。  
串行：
    
    
    public class Main {
    
        private static Long sum = 0L;
    
        public static void main(String[] args) {
    
            System.out.println("main start sum = " + sum);
    
            ExecutorService service = new ThreadPoolExecutor(
                    10, 10, 0L, TimeUnit.SECONDS, new LinkedBlockingQueue<>());
    
            for (int i = 0; i < 10; i++) {
                new Add().run();
            }
            service.shutdown();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                System.out.println("main interrupt exception");
            }
            System.out.println("main end sum = " + sum);
        }
    
        static class Add implements Runnable {
    
            @Override
            public void run() {
                Thread thread = Thread.currentThread();
                System.out.println(thread.getName() + thread.getId() + " start add!");
                for (int i = 0; i < 10000; i++) {
                    sum++;
                }
                System.out.println(thread.getName() + thread.getId() + " add over!");
            }
        }
    }
    

执行结果：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2b6e9e7ed4a8a55af3884e8e7813a91e.png)  
并发：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/355e85abc2b51bddf9fc597c30443d0b.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f6accd0713d3bf2565d36cb77a1e9d41.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8d79f1497b9ac105992513f880c533de.png)  
每次执行的结果都是不确定的。  
所以，在并发情况下，对同一个变量的操作，会出现语义不一致的并发问题。  
那么，如何解决这个问题呢？  
加锁。  
一般来说，Java中锁的实现有两种方式：synchronized和Lock.  
我们先用synchronized修改  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bd371a587898f23a17e7f3e90dfac111.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1f0f54af89b630006c25dd2f2c3ccb2c.png)  
接下来使用Lock进行修改：
    
    
    public class Main {
    
        private static volatile Long sum = 0L;
    
        private static volatile Lock lock = new ReentrantLock();
    
        public static void main(String[] args) {
    
            System.out.println("main start sum = " + sum);
    
            ExecutorService service = new ThreadPoolExecutor(
                    10, 10, 0L, TimeUnit.SECONDS, new LinkedBlockingQueue<>());
    
            for (int i = 0; i < 10; i++) {
                service.execute(new Add());
            }
            service.shutdown();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                System.out.println("main interrupt exception");
            }
            System.out.println("main end sum = " + sum);
        }
    
        static class Add implements Runnable {
    
            @Override
            public void run() {
                Thread thread = Thread.currentThread();
                System.out.println(thread.getName() + thread.getId() + " start add!");
                try {
                    while(!lock.tryLock()){
                        TimeUnit.MILLISECONDS.sleep(200);
                    }
                    for (int i = 0; i < 10000; i++) {
                        sum++;
                    }
                }catch (InterruptedException e){
                    System.out.println(e);
                } finally {
                    lock.unlock();
                }
                System.out.println(thread.getName() + thread.getId() + " add over!");
            }
        }
    }
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/67ec3b09f82c660943af3a5810879534.png)  
我们增加了一个全局变量，这个全局变量就是sum的锁，只有获取到了锁的线程，才能进行累加操作。如果没有获取锁，那么就线程sleep200毫秒。然后重新获取锁，直到获取了锁，否则就一直循环。

### 2.2 锁存在的意义

在2.1 中，我们可以很明显的看到，这是因为并发情况下，多个线程对全局变量的读写，造成语义不一致。  
说简单点，就是第一个线程和第二个线程等多个线程读取到了相同的sum初始值，然后对sum初始值进行递增操作，导致多个线程递增和一个线程的一次递增的结果相同。（不考虑时间先后问题）（线程间指令重排问题）  
还有就是第一个线程可能计算的快，已经计算到了 9++=10了，但是第二个线程还是比较慢，才计算到了1++=2。（线程执行，多个核心执行，每个核心的寄存器里面都有sum的一个副本）（内存可见性问题）  
锁的存在就是为了解决这些问题。

## 3\. synchronized

### 3.1 synchronized的使用场景

| 分类  | 具体场景         | 被锁的对象      | 伪代码                                        |
|-----|--------------|------------|--------------------------------------------|
| 方法  | 实例方法         | 类的实例对象     | public synchronized void method(){}        |
| 方法  | 静态方法         | 类对象        | public static synchronized void method(){} |
| 代码块 | 实例对象         | 类的实例对象     | synchronized (this){}                      |
| 代码块 | class对象      | 类对象        | synchronized(Main.class){}                 |
| 代码块 | 任意实例对象Object | 实例对象Object | String x = “”; synchronized(x){}           |

### 3.2 synchronized原理

首先我们将2.1中的synchronized实现的代码进行编译`javac Main.java`,然后使用`javap -v`进行反编译  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8308ed9832bc0d0fb3cc8fa5411cece.png)  
这里比较好找，先找递增操作，ladd的指令，在ladd的指令前后有monitorenter指令。

#### 3.2.1 Java对象在JVM中的结构

> ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7f2c61061ffb1821f04a7518ab1b4b65.png)  
>  ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad4bdc340fa949ef4535e1af6c535c20.png)

来源：<https://blog.csdn.net/z_ssyy/article/details/103737553>

通过上面两张图片，可以很直观的知道，对象在jvm中分为三块区域：对象头，对象实际数据，填充数据。

> ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a732605e5ae6724e70eeed87ea9f3053.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9728e857d6e37f3ebbb67dc639e03b41.png)

来自：<https://blog.csdn.net/javazejian/article/details/72828483>

#### 3.2.2 monitor指令

monitor指令分为两个：monitorenter和monitorexit。  
分别代码开始同步和结束同步。或者开始加锁，结束加锁。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb7572b8784254576c5f520d653ce2f9.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/27729c7255c6aee576ac6834e56f56ac.png)  
可以理解为：在遇到monitorenter指令的时候，进行加锁，进入同步代码后，每次进行操作前后，都需要获取最新的数据，执行完毕，及时的写回。(这是个人理解)  
在执行过程中，遇到monitorenter指令，设置对象的锁标志以及线程id（重入锁的核心实现）。  
因为第一个争夺到锁的线程已经将锁标志置1了，其他线程就无法获取锁了（无法在增加了）。  
当执行完同步操作后，遇到monitorexit指令，设置对象的锁标志为0，线程id清空(网上的资料没有指明不过从重入锁的定义来分析，应该是清空id的)  
这样其他线程就可以获取锁了。

#### 3.2.3 monitor指令过程

在2.3.2.2小节中知道，每一个对象都有自己的对象头，而在对象头中有一个锁标志，只有线程修改锁标志成功，才是获取到了锁，其他线程只能等待。  
所以，如果有若干线程同时获取一个对象的锁，其中某一个线程得到锁之后，执行线程的任务，而其他锁则会进入同步队列，线程也会进入BLOCKED的状态。

> ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7dc0e32d0b7ba957e5cc202119fe2925.png)

图片来自<https://www.jianshu.com/p/d53bf830fa09>

## 4\. synchronized 对类对象和实例对象的区别

### 4.1 static修饰和没有static修饰的区别

首先理解一个关键字static，这个关键字是区分一个属性变量是否是类变量，还是实例属性变量。  
同样的，一个方法如果有static就是说，这个这个方法是类方法；如果以一个方法没有static 就认为这个方法是实例方法。  
当然，最明显的是：类方法和类变量，可以直接通过类名调用；而实例方法和实例变量，必须先创建类的实例，然后通过实例调用。  
还有一点需要注意：非static方法可以调用static方法和非static方法，而static方法只能调用static方法。

从对象的角度来看：  
类对象，类方法不需要使用new实例化对象，就可以调用类方法和类变量。  
因为类对象在内存中只会存储一个。  
还记得前面说的JVM中对象的结构吗，在对象头中，就会存储类元数据：

> ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6d88fa8b86eb64b205705cf03619a4cc.png)

来自：<https://blog.csdn.net/z_ssyy/article/details/103737553>  
实例对象的对象头中存储的这个类元数据就是类对象的地址。  
也就是说，在内存中，这个类的所有实例对象的对象头都会存储类元信息，也就是类对象。而且这些实例对象的类对象都是相同的。

用最直白的话说：类对象，内存中只有一份；实例对象，每new一次，就会有一个。

因为内存中只有一个，所以不管是类变量还是类属性，，都是同一个，怎么调用都行。

而实例方法或者实例对象调用类方法或者类属性：因为实例对象和类对象是多对1的关系，所以实例方法调用类方法或者类属性就是互斥的。在同一时刻，只能有一个实例对象可以调用成功(有锁，或者有同步逻辑的)。如果是不需要同步的，那无所谓了。  
比如：
    
    
    public class Student {
    
        public static void say() {
            System.out.println("static method");
            Thread thread = Thread.currentThread();
            System.out.println(thread.getName() + thread.getId());
            while (!thread.isInterrupted()) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    break;
                }
            }
            System.out.println("static end");
        }
    
        public void sing() {
            System.out.println("nomal method");
            Thread thread = Thread.currentThread();
            System.out.println(thread.getName() + thread.getId());
            while (!thread.isInterrupted()) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    break;
                }
            }
            System.out.println("nomal method end");
        }
    }
    
    
    
    public class StudentMain {
    
        public static void main(String[] args) {
            Student student = new Student();
            Thread t1 = new Thread(() -> {
                Student.say();
            });
            t1.start();
            Thread t2 = new Thread(() -> {
                student.sing();
            });
            t2.start();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                System.out.println("main interrupt exception");
            }
            System.out.println("main method , static method and nomal method is BLOCKED");
            new Thread(() -> Student.say()).start();
            new Thread(() -> student.sing()).start();
            new Thread(() -> new Student().sing()).start();
        }
    
    }
    

执行结果：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/502979b83c55a24eb258225e7b4feaac.png)  
即使第一次调用的线程现在在方法内阻塞，但是，因为方法不是同步方法，所以，后面创建的线程依然可以访问，依然可以进入。  
那么，把方法修改成需要同步的呢？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/571d6e6e2dbe5aeecf9c2c3fd8869f6a.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2725be5ed55b81f4648090c1ffc58af5.png)  
这个时候，实例同步方法可以进入，但是类同步方法不可以进入。  
从这里也进一步说明，类对象，类方法在内存中是一份的。而实例方法是每new一次，就会产生一个的。  
然后实例对象的类元信息就是类对象的地址。

### 4.2 synchronized 不同使用场景

这个时候，我们返回去看下2.3.1的使用场景![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fccb25adbe01c1041eb5c5e24d2567a3.png)  
其实就是可以分为2类，一种是实例对象锁，一种是类对象锁。

### 4.3 不使用synchronized 同步

接下来，在看一个例子：  
在多线程的情况下，多个线程对同一个属性进行操作，会发生并发问题。  
我们通过实例查看：
    
    
    public class People {
    
        private Long sum = 0L;
    
        private static Long all = 0L;
    
        public People(){}
    
        public Long getSum(){
            return sum;
        }
    
        public void setSum(Long sum){
            this.sum = sum;
        }
    
        public Long getAll(){
            return all;
        }
    
        public void setAll(Long all){
            People.all = all;
        }
    }
    
    
    
    public class Main {
    
        public static void main(String[] args) {
            People people = new People();
            Runnable runnable = () -> {
                Thread thread = Thread.currentThread();
                System.out.println(thread.getName() + thread.getId() + " start ");
                for (int i = 0; i < 10000; i++) {
                    people.setAll(people.getAll() + 1);
                    people.setSum(people.getSum() + 1);
                }
                System.out.println(thread.getName() + thread.getId() + " end ");
            };
            System.out.println("main thread sum = " + people.getSum() + " , all = " + people.getAll());
            ExecutorService service = new ThreadPoolExecutor(10, 10, 0, TimeUnit.SECONDS, new LinkedBlockingQueue<>());
            for (int i = 0; i < 10; i++) {
                service.execute(runnable);
            }
            service.shutdown();
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException e) {
                System.out.println("main interrupt exception");
            }
            System.out.println("main end sum = " + people.getSum() + " , all = " + people.getAll());
        }
    
    }
    

我们创建了一个类，类里面有两个属性，一个是static的，另一个是非static的。  
也就是说，all是类变量，sum是实例变量。

在主线程中，我们一个线程将People的属性值增加1W，那么，10个线程就是10W。  
我们预期的目标是all和sum都是10W。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/be6f9b3625aa56bd55a522e177ac24ea.png)

### 4.4 synchronized 同步代码块–类对象

为了解决这个问题，有两种解决方式：加锁(Lock)或者同步(synchronized)  
在这里只考虑同步的实现方式。  
你可能注意到了，我们的People中的两个属性，一个是类属性，一个是实例属性。  
首先，我们使用代码块同步类的方式，进行同步：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/962d374bb78a302c13de03333b91b7bd.png)  
运行结果：  
预期分析：因为使用的是类同步，对于每一个实例对象来说，对应的都是同一个类对象。所以当这10个线程的其中某一个线程获取了类同步的锁，其他线程就无法获取类同步的锁了，其他线程就会被阻塞了。  
这样就保证了同一时间只会有一个线程操作类变量和实例变量。就不存在并发问题了。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fca0e9afa7566f908e65368e4dc3b112.png)

### 4.5 synchronized 同步代码块–实例对象

接下来，我们使用对象同步呢？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/02336c66aa2492fb3f77bf93de9e1cab.png)  
预期分析：经过上面的例子，这个可以很轻松的分析出来，这个例子也能达到我们的目的。  
因为这10个线程使用的是同一个实例对象，所以使用实例对象，也就是10个线程在竞争一个实例对象的同步锁。  
也能够保证同一时间内，只有一个线程操作类变量和实例变量。

### 4.6 synchronized 同步代码块–任意实例对象

上面两个小例子是synchronized同步对象的例子，在同步代码块的场景中，还有一种，同步任意实例对象。  
其实同步任意实例对象和同步某一个实例对象的原理是一样的：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7ad6c45ab9a47a9edb9c6131efff0326.png)  
在这种写法下，10个线程竞争同一个实例对象的同步锁，当然可以保证同一时间内只有一个线程进行操作。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d9ad7448fb3c914b41f3534d2ff3cf7.png)  
可是，如果每一个线程使用的都是自己线程内创建的实例对象呢？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a42a78363277fded96bfe4eff68f77c.png)  
预期分析：因为我们将实例对象放到了线程内，那么首先这个10个线程对应的是10个实例对象，每一个线程同步的都是自己线程内创建的对象，这当然每一个线程都能够获取到实例对象锁了，也就是每一个线程在任意时间都可以操作类变量和实例变量。  
也就无法达到预期目标了。  
换个角度想，当我们将实例对象的创建移到线程内的时候，对于每一个单个的线程来说，其同步的都是自己线程内的局部变量。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1670736a16eac0a876db85d2cf6a1cb6.png)

### 4.7 synchronized 同步方法–类方法

我们看完了synchronized同步代码块，接下来看看synchronized同步方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d03e14119e27b04001aed13978559c6.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7efe05ff1f4881f2edde7ece6407b56e.png)  
预期分析：  
因为方法是类方法，在整个内存中只有一个，所以，可以保证同一时间只有一个线程能够获取锁。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c3c71212c2ea135cdfe797516b51b1ee.png)  
这个可能不太好对比：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c330d68586dbeef4f736ee70e38ffa6.png)  
我们新增了两个方法，一个是类方法，一个是实例方法。  
因为我们在类方法上进行同步，所以类变量符合预期结果，而实例方法因为没有进行同步，所以，实例变量不符合预期结果：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/019b0bee519c21b5f97579bd2e5f854b.png)

### 4.8 synchronized 同步方法–实例方法

接下来我们根据上面的例子，同步实例方法，然后不同步类方法，以作对比：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/258f7e6c7e0c6c78407e3641fd015ecb.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/413fcf27412e6771558293cee2ee425b.png)  
预期分析：因为类方法没有进行同步，所以类方法应该不符合预期结果。  
而实例方法进行同步，那么同步方法应该是符合预期的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0b8906dd00102f9f71448bd0994a1a8.png)  
即使这样调用，类方法也不同步的：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f31657548e6d5a778cf6b9d0a5f35a66.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c17718c8123327b9ca481286a0ad048.png)

## 5\. synchronized 的缺陷

  1. synchronized效率低  
如果在同步方法或者同步代码所需时间较长，那么其他线程就必须阻塞等待， 而且可能会发生无限等待的情况，synchronized非常影响程序执行效率(Lock有多种锁，可以根据需要同步的操作选择不同的锁)
  2. synchronized资源利用率低  
一般来说，写操作和写操作有冲突，写操作和读操作有冲突。  
但是读操作和读操作是没有冲突的。  
在程序中，我们较多的操作是读取计算，写入只占一部分。  
使用synchronized的时候，如果是读操作，同步了，其他线程也无法进行读操作。  
也就造成资源的浪费。
  3. synchronized无法知道是否成功获取到锁  
使用synchronized我们可以实现同步，但是线程是否成功获取锁，这是一个不确定事件。

## 6\. synchronized的锁处理

在jdk5之后，jvm对synchronized做了优化。

  * 默认开启偏向锁
  * 会进行锁升级  
在jdk5之前synchronized是重量级锁。  
在jdk5之前，使用synchronized是比较耗费资源的。  
因为在jdk5之前，synchronized是需要调用OperatorSystem的一些操作，实现锁的。  
这就涉及到线程需要从用户态切换到内核态。  
这个切换过程非常耗费时间。所以，在jdk5之前，synchronized是重量级锁，耗费性能。

在jdk5之后，jvm对synchronized进行了优化。  
在jdk5之后，线程使用synchronized进行同步，首先会使用偏向锁，如果有第二个线程竞争锁，此时锁会升级为轻量级锁，多个线程竞争轻量级锁，未竞争到锁的线程进行自旋等待。如果自旋超过10次还未获取到锁，那么锁就会升级为重量级锁。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9728e857d6e37f3ebbb67dc639e03b41.png)

偏向锁的机制也比较简单，在对象的对象头中写入了一个线程的id，那么此时，如果这个线程再次获取锁，jvm将对象的对象头中的线程id与竞争锁的线程id进行对比，如果是一样的，那么这个线程就直接获取锁。  
如果有多于1个线程进行竞争锁，此时偏向锁只能记录一个线程id，就不合适了，此时会升级为轻量级锁。

偏向锁的设计思想是：在大多数程序中，我们还是串行处理占多数；并发处理的时间或者操作占比比较低。  
轻量级锁的设计思想是：在大多数并发中，我们需要同步加锁的操作是比较简单，快速的操作，占整个线程处理时间的占比很小。所以，每一个线程获取到锁之后，大多数是在很短的时间内就会释放。  
重量级锁的设计思想是：即使并发冲突的概率比较小，但是并发冲突的造成的后果非常的严重。当并发冲突无法避免的时候，我们就需要保证并发的安全。

## 7\. synchronized 处理过程

> ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d8f74d79169897525c66b0485c0f934.png)  
>  当有多个线程一起访问某个对象的monitor对象的时候，对象监视器会将这些线程存储在不同的容器中：
> 
>   1. Contention List：竞争队列，所有请求锁的线程首先被放在这个竞争队列中；
>   2. Entry List：Contention List中那些有资格成为候选资源的线程被移动到Entry List中；
>   3. Wait Set：哪些调用wait方法被阻塞的线程被放置在这里；
>   4. OnDeck：任意时刻，最多只有一个线程正在竞争锁资源，该线程被成为OnDeck；
>   5. Owner：当前已经获取到所资源的线程被称为Owner；
>   6. !Owner：当前释放锁的线程。
> 

> 
>   
> JVM每次从队列的尾部取出一个数据用于锁竞争候选者（OnDeck），但是并发情况下，ContentionList会被大量的并发线程进行CAS访问，为了降低对尾部元素的竞争，JVM会将一部分线程移动到EntryList中作为候选竞争线程。Owner线程会在unlock时，将ContentionList中的部分线程迁移到EntryList中，并指定EntryList中的某个线程为OnDeck线程（一般是最先进去的那个线程）。Owner线程并不直接把锁传递给OnDeck线程，而是把锁竞争的权利交给OnDeck，OnDeck需要重新竞争锁。这样虽然牺牲了一些公平性，但是能极大的提升系统的吞吐量，在JVM中，也把这种选择行为称之为“竞争切换”。  
>   
>  OnDeck线程获取到锁资源后会变为Owner线程，而没有得到锁资源的仍然停留在EntryList中。如果Owner线程被wait方法阻塞，则转移到WaitSet队列中，直到某个时刻通过notify或者notifyAll唤醒，会重新进去EntryList中。  
>   
>  处于ContentionList、EntryList、WaitSet中的线程都处于阻塞状态，该阻塞是由操作系统来完成的（Linux内核下采用pthread_mutex_lock内核函数实现的）。  
>   
>  **Synchronized是非公平锁。** Synchronized在线程进入ContentionList时，等待的线程会先尝试自旋获取锁，如果获取不到就进入ContentionList，这明显对于已经进入队列的线程是不公平的，还有一个不公平的事情就是自旋获取锁的线程还可能直接抢占OnDeck线程的锁资源。

来自：<https://blog.csdn.net/zqz_zqz/article/details/70233767>