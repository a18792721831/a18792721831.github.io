---
layout: post
title: "Java基础--CountDownLatch--计数器锁(门闩锁)"
date: 2020-07-01 20:02:26 +0800
categories: [CountDownLatch, 门闩锁源码解读, 计数器锁源码解析, 带你一起阅读门闩锁源码, 门闩锁原理]
description: "@toc1. CountDownLatch一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。用给定的计数 初始化 CountDownLatch。由于调用了 countDown() 方法，所以在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await 的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 CyclicBarrier。1.1 CountDownLatch 的UML图1.2 C_java 计数器锁"
keywords: CountDownLatch, 门闩锁源码解读, 计数器锁源码解析, 带你一起阅读门闩锁源码, 门闩锁原理
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107070198
> - 发布时间：2020-07-01 20:02:26
> - 阅读量：1
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, 多线程
> - 标签：#CountDownLatch, #门闩锁源码解读, #计数器锁源码解析, #带你一起阅读门闩锁源码, #门闩锁原理

## 摘要

文章浏览阅读1.3k次。@toc1. CountDownLatch一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。用给定的计数 初始化 CountDownLatch。由于调用了 countDown() 方法，所以在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await 的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 CyclicBarrier。1.1 CountDownLatch 的UML图1.2 C_java 计数器锁

---

@[toc](<Java%E5%9F%BA%E7%A1%80--CountDownLatch--%E8%AE%A1%E6%95%B0%E5%99%A8%E9%94%81%28%E9%97%A8%E9%97%A9%E9%94%81%29>)

## 1\. CountDownLatch

一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

用给定的计数 初始化 CountDownLatch。由于调用了 countDown() 方法，所以在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await 的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 CyclicBarrier。

### 1.1 CountDownLatch 的UML图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c47497b1dac223eeeb68c5722e943adf.png)

### 1.2 CountDownLatch 的属性方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/050b790d78c2eb03b5f8f8c011d60155.png)

## 2\. CountDownLatch 构造

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a93d450cd2ab2ca4b0677b0a76ca312e.png)  
如果传入的值小于0，那么抛出异常。  
否则，初始化内部的Sync

### 2.1 Sync 构造

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a2445b68b282f6e9c624ab625f99bce0.png)  
CountDownLatch内部的Sync也是继承于AQS的。  
Sync的构造，传入的值是初始化AQS的锁持有线程数量的。

## 3\. await

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6bafc453881555a87a8fdb4c7942964e.png)  
CountDownLatch的await方法是等待通知，将当前线程阻塞，直到锁空闲。(或者说计数器倒数至0)  
AQS的acquireSharedInterruptibly方法请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
的5.6.4小节。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/78fa48bed8c217acc5c5ee25fcbbea59.png)  
调用AQS的acquireSharedInterruptibly方法。  
await方法响应中断，而且计数器锁是一个共享锁。  
是共享锁，就需要AQS的子类Sync实现tryAcquireShared和tryReleaseShared方法。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cbd3906806a53945b3b214da23346013.png)  
在自旋中尝试获取共享锁，尝试获取共享锁，调用的是CountDownLatch的Sync实现的tryAcquireShared方法。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5949e65d65ac35d68de3bb236ad84891.png)  
只有锁空闲，才允许等待竞争队列中的线程执行。  
换句话说，当计数器倒数没到0时，线程需要等待计数器归0.

## 4\. await(long,TimeUnit)

带有超时时间的等待，响应中断。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c9855873ba1d49d3cb862c1c6bcec612.png)  
调用了AQS的tryAcquireSharedNanos方法。  
AQS的tryAcquireSharedNanos方法请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
的5.6.5小节。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1f9b1b3aaa0aa870b191280269369c06.png)  
带有超时的等待，也是调用CountDownLatch的Sync实现的tryAcquireShared方法的。

## 5\. countDown

计数器锁值减1.  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e4cc874a620ad3167fe4c27e961aa63f.png)  
直接调用AQS的releaseShared方法。  
AQS的releaseShared方法请看  
[Java基础–ReentrantReadWriterLock–重入读写锁](<https://blog.csdn.net/a18792721831/article/details/107026498>)  
的2.1.5小节。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a5b72e17ea40b95ce159bdfb39b6c6fe.png)  
AQS的releaseShared方法会调用AQS子类实现的tryReleaseShard方法的。  
也就是CountDownLatch的Sync实现的tryReleaseShared方法。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d0b3ff861f36c374175e861c1913fdf.png)  
自旋将锁持有数量减1，也就是将计数器锁的值减1.  
如果锁状态已经是0，表示现在已经无法继续减下去了。不过也不会抛出异常，因为countDown是没有返回值的。  
如果锁状态不是0，那么将锁状态减1，最后返回锁是否空闲。  
如果锁空闲，那么等待竞争队列中的线程都可以再次调用tryAcquireShared方法尝试获取锁。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/05d135cd22ad4b1fc0ad1777059c92e1.png)  
此时锁空闲，锁状态为0，也就是，等待竞争队列中每一个线程都能获取锁。  
**请注意，CountDownLatch的计数器值为0之后，在无法恢复的。  
CountDownLatch只能做减法。**

## 6\. getCount

获取当前计数器锁的值。即获取锁状态值。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c0fd25b1dd03112def75108780ce611f.png)  
CountDownLatch调用其内部的Sync的getCount方法。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b128c18489f979d0654df116b7e44aa.png)  
CountDownLatch的Sync的getCount方法调用AQS的getState方法。

## 7\. 示例
    
    
    public class MyCountDownLatch {
    
        public static void main(String[] args) {
            // 创建一个值为count的计数器锁
            int count = 10;
            CountDownLatch countDownLatch = new CountDownLatch(count);
            Runnable runnable = () -> {
                Thread thread = Thread.currentThread();
                System.out.print(thread.getName() + " start count down : " + countDownLatch.getCount() + " -> ");
                countDownLatch.countDown();
                System.out.println(countDownLatch.getCount() + " time is " + System.currentTimeMillis());
            };
    
            Runnable runnable1 = () -> {
                Thread thread = Thread.currentThread();
                System.out.println(thread.getName() + " await count down latch , time is " + System.currentTimeMillis());
                try {
                    countDownLatch.await();
                } catch (InterruptedException e) {
                    System.out.println("ie");
                }
                System.out.println(thread.getName() + " await count down latch end, time is " + System.currentTimeMillis());
            };
    
            for (int i = 0; i < count; i++) {
                new Thread(runnable1, "awaiter" + i).start();
            }
            for (int i = 0; i < count; i++) {
                new Thread(runnable, "countDown" + i).start();
            }
            System.out.println("main end");
        }
    
    }
    

执行结果  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c80a5e3ef32fa23546aab780707f3fa7.png)  
开始后，await线程都被阻塞了，然后count down线程将计数器锁的值减小，直到为0.  
此时await线程就都被唤醒继续执行了。

需要注意一点，在输出中 `->`左边可能相等，但是右边一定不相等。  
因为右边使用cas进行设置的，保证线程安全。