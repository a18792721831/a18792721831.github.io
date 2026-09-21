---
layout: post
title: "【转载】ArrayBlockingQueue"
date: 2019-06-12 18:41:57 +0800
categories: ["2019"]
description: "因ArrayBlockingQueue使用不当，导致线上系统故障及数百万经济损失。文章深入解析ArrayBlockingQueue工作原理，揭示不当使用导致的问题，并提出正确实践。"
keywords: 2019
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/91589727
> - 发布时间：2019-06-12 18:41:57
> - 阅读量：239
> - 分类：java专栏收录该内容, 订阅专栏

## 摘要

文章浏览阅读239次。因ArrayBlockingQueue使用不当，导致线上系统故障及数百万经济损失。文章深入解析ArrayBlockingQueue工作原理，揭示不当使用导致的问题，并提出正确实践。

---

转自：<https://www.xttblog.com/?p=3686>  
我们为什么要招高级程序员呢？因为高级程序员写的 bug 可能更少，在调用 api 的时候，犯错的概率更小。但是并不意味这高级程序员就不犯错。今天我们就一起来分享一个由于 ArrayBlockingQueue 使用不当，导致公司损失几百万的案例！  
根据 ArrayBlockingQueue 的名字我们都可以看出，它是一个队列，并且是一个基于数组的阻塞队列。  
ArrayBlockingQueue 是一个有界队列，有界也就意味着，它不能够存储无限多数量的对象。所以在创建 ArrayBlockingQueue 时，必须要给它指定一个队列的大小。  
我们先来熟悉一下 ArrayBlockingQueue 中的几个重要的方法。

  * add(E e)：把 e 加到 BlockingQueue 里，即如果 BlockingQueue 可以容纳，则返回 true，否则报异常 
  * offer(E e)：表示如果可能的话，将 e 加到 BlockingQueue 里，即如果 BlockingQueue 可以容纳，则返回 true，否则返回 false 
  * put(E e)：把 e 加到 BlockingQueue 里，如果 BlockQueue 没有空间，则调用此方法的线程被阻断直到 BlockingQueue 里面有空间再继续
  * poll(time)：取走 BlockingQueue 里排在首位的对象，若不能立即取出，则可以等 time 参数规定的时间,取不到时返回 null 
  * take()：取走 BlockingQueue 里排在首位的对象，若 BlockingQueue 为空，阻断进入等待状态直到 Blocking 有新的对象被加入为止 
  * remainingCapacity()：剩余可用的大小。等于初始容量减去当前的 size  
我们再来看一下 ArrayBlockingQueue 使用场景。
  * 先进先出队列（队列头的是最先进队的元素；队列尾的是最后进队的元素）
  * 有界队列（即初始化时指定的容量，就是队列最大的容量，不会出现扩容，容量满，则阻塞进队操作；容量空，则阻塞出队操作）
  * 队列不支持空元素  
ArrayBlockingQueue 进队操作采用了加锁的方式保证并发安全。源代码里面有一个 while() 判断：

    
    
    public void put(E e) throws InterruptedException {
        checkNotNull(e); // 非空判断
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly(); // 获取锁
        try {
            while (count == items.length) {
                // 一直阻塞，知道队列非满时，被唤醒
                notFull.await();
            }
            enqueue(e); // 进队
        } finally {
            lock.unlock();
        }
    }
    public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {
        checkNotNull(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length) {
            // 阻塞，知道队列不满
            // 或者超时时间已过，返回false
                if (nanos <= 0)
                    return false;
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }
    }
    

通过源码分析，我们可以发现下面的规律：

  * 阻塞调用方式 put(e)或 offer(e, timeout, unit)
  * 阻塞调用时，唤醒条件为超时或者队列非满（因此，要求在出队时，要发起一个唤醒操作）
  * 进队成功之后，执行notEmpty.signal()唤起被阻塞的出队线程  
出队的源码类似，我就不贴了。ArrayBlockingQueue 队列我们可以在创建线程池时进行使用。

    
    
    new ThreadPoolExecutor(1, 1,
      0L, TimeUnit.MILLISECONDS,
      new ArrayBlockingQueue<Runnable>(2));
     
    new ThreadPoolExecutor(1, 1,
      0L, TimeUnit.MILLISECONDS,
      new LinkedBlockingQueue<Runnable>(2));
    

了解了这些后，再来看看我们开发人员的使用。  
当时线上系统出故障后，导致所有的请求都处理不了。给人的感觉就是，界面上一直在转圈。  
于是不得不 dump 线程，然后重启机器，先恢复使用。每次一故障，客服电话就被打爆了，投诉率疯升，当天订单大幅下滑。前前后后发生几次故障，领导都气疯了。几百万就这样没了，所以给我们的压力非常的大。  
dump 下来后，我分析发现线程都 Block 在写日志的地方。然后，我前前后后怕查，发现了 block 在了 ArrayBlockingQueue.put 这个方法。检查源码，发现创建了 ArrayBlockingQueue(250) 个长度的队列。当队列超过 250 时，put 就一定会被 block 住。  
业务代码抽象如下：
    
    
    if (blockingQueue.remainingCapacity() < 1) {
        //todo
    }
    blockingQueue.put(...)
    

这里两个悲催的问题，一是这个 if 判断完后，还是会进行 put 操作，应该是 else 中进行 put 操作；二是满了之后，还在 todo，做其他事情。  
其实我们这里可以完全没必要进行 if (blockingQueue.remainingCapacity() < 1) 判断，使用 blockingQueue.offer 不就完事了嘛。如果 BlockingQueue 可以容纳，则返回 true，否则返回 false。  
所以说，除了技术本身外，代码的细节功力是非常非常重要的。  
学习并不是为了解决 bug，而是预防 bug 产生！