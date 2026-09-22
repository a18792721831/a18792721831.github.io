---
layout: post
title: "Java基础--ReentrantLock--重入锁"
date: 2020-06-29 19:01:22 +0800
categories: [ReentrantLock源码, ReentrantLock结构, ReentrantLock解析, 理解ReentrantLock, 学习ReentrantLock]
description: "Java基础--ReentrantLock--重入锁1. ReentrantLock的整体结构1.1 ReentrantLock的UML图1.2 ReentrantLock的属性、方法2. ReentrantLock 实现Lock接口2.1 tryLock2.2 tryLock(long,TimeUnit)2.3 lock2.4 lockInterruptibly2.5 unlock2.6 newCondition3. ReentrantLock 内部Sync实现了AQS3.1 lock3.2 nonfai_reentrantlock重入锁"
keywords: ReentrantLock源码, ReentrantLock结构, ReentrantLock解析, 理解ReentrantLock, 学习ReentrantLock
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107005826
> - 发布时间：2020-06-29 19:01:22
> - 阅读量：529
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, 多线程
> - 标签：#ReentrantLock源码, #ReentrantLock结构, #ReentrantLock解析, #理解ReentrantLock, #学习ReentrantLock

## 摘要

文章浏览阅读529次。Java基础--ReentrantLock--重入锁1. ReentrantLock的整体结构1.1 ReentrantLock的UML图1.2 ReentrantLock的属性、方法2. ReentrantLock 实现Lock接口2.1 tryLock2.2 tryLock(long,TimeUnit)2.3 lock2.4 lockInterruptibly2.5 unlock2.6 newCondition3. ReentrantLock 内部Sync实现了AQS3.1 lock3.2 nonfai_reentrantlock重入锁

---

#### Java基础--ReentrantLock--重入锁

  * 1\. ReentrantLock的整体结构
  *     * 1.1 ReentrantLock的UML图
    * 1.2 ReentrantLock的属性、方法
  * 2\. ReentrantLock 实现Lock接口
  *     * 2.1 tryLock
    * 2.2 tryLock(long,TimeUnit)
    * 2.3 lock
    * 2.4 lockInterruptibly
    * 2.5 unlock
    * 2.6 newCondition
    * 2.7 其他方法
  * 3\. ReentrantLock 内部Sync实现了AQS
  *     * 3.1 lock
    * 3.2 nonfairTryAcquire--不公平的尝试获取锁
    * 3.3 tryRelease
    * 3.4 isHeldExclusively
    * 3.5 newCondition
    * 3.6 getOwner
    * 3.7 getHoldCount
    * 3.8 isLocked
    * 3.9 readObject
  * 4\. ReentrantLock 内部NonfairSync和FairSync继承Sync
  *     * 4.1 FairSync--lock
    * 4.2 FairSync--tryAcquire
    * 4.3 NonfairSync--lock
    * 4.4 NonfairSync--tryAcquire
  * 5\. ReentrantLock 的构造
  *     * 5.1 ReentrantLock
    * 5.2 ReentrantLock(boolean)
  * 6\. 总结

## 1\. ReentrantLock的整体结构

### 1.1 ReentrantLock的UML图

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f18ebc062c98bc394b6bb694ef360cad.png)  
感觉有些抽象，稍微调整下布局：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8090072961deb0327742c328a678779f.png)  
这是一个巨复杂的类。  
不过，仔细看，ReentrantLock上面直接连接的关系并不多：ReentrantLock实现了Lock接口；其拥有一个内部类Sync，Sync类继承于AQS；同时ReentrantLock内部继承Sync实现了NonfairSync和FairSync。  
也就是说，ReentrantLock其实可以分为三部分：  
第一部分，ReentrantLock实现了Lock方法，所以需要实现Lock的接口；  
第二部分，ReentrantLock内部的Sync继承于AQS，在一定程度上实现了AQS要求实现的方法；  
第三部分，ReentrantLock内部的NonfairSync和FairSync继承内部的Sync，增强了Sync的实现。

### 1.2 ReentrantLock的属性、方法

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b8f69923d6ad5bcb56ceee25dffcc62e.png)

## 2\. ReentrantLock 实现Lock接口

既然ReentrantLock实现了Lock接口，那么，我们就从Lock入手。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6ca967b47d2d22d6b59b287f4476a3ca.png)

### 2.1 tryLock

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2c985dbe004c0c489d192e7bbc1f41e1.png)  
可以看到，ReentrantLock的tryLock方法只是内部Sync的nonfairTryAcquire的代理，请从目录跳转到3.2.  
这里传入的1表示每次重入，锁的重入层数加1.

### 2.2 tryLock(long,TimeUnit)

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5aefaefec7930de6dee3e0dfcbe5c225.png)  
这个方法也是做了代理。代理的目标方法是AQS的方法：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5696470b01b0a8541198acb4821bf1bf.png)  
tryAcquireNanos的方法的时序图如下：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f6286ac3f1534f99b76f8d6663d3f47f.png)  
首先是调用ReentrantLock中FairSync和NonfairSync实现的tryAcquire方法。  
如果tryAcquire方法失败，就会调用doAcquireNanos方法。  
tryAcquire方法请跳转4.2,4.4
    
    
    // 自旋指定纳秒获取锁，响应中断
    private boolean doAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        // 如果等待的纳秒小于等于0，那么直接失败
        if (nanosTimeout <= 0L)
            return false;
        // 根据刚进入方法时，系统的纳秒时间和等待时间计算出，截止时间
        final long deadline = System.nanoTime() + nanosTimeout;
        // 将当前线程以独占的方式加入等待竞争队列
        final Node node = addWaiter(Node.EXCLUSIVE);
        // 定义获取锁失败状态的标志
        boolean failed = true;
        try {
        	// 进入自旋
            for (;;) {
            	// 获取node的前继线程节点
                final Node p = node.predecessor();
                // 如果node的前继线程节点是头节点(表示node是第一个等待的线程)
                // tryAcquire尝试获取锁成功
                if (p == head && tryAcquire(arg)) {
                	// 将node设置为头结点(设置头结点会将线程节点的线程和前继清空)
                    setHead(node);
                    // 将原头节点从等待竞争队中移除
                    p.next = null; // help GC
                    // 设置获取锁成功
                    failed = false;
                    // 返回获取锁成功
                    return true;
                }
                // 每次自旋都进行判断剩余自旋时间
                nanosTimeout = deadline - System.nanoTime();
                // 如果剩余自旋时间小于等于0，那么需要结束自旋
                if (nanosTimeout <= 0L)
                	// 结束自旋的时候还没有获取到锁，所以获取锁失败，返回获取锁失败
                    return false;
                // 调用shouldParkAfterFailedAcquire方法，将node前继节点的等待状态设置为SIGNAL
                if (shouldParkAfterFailedAcquire(p, node) &&
                	// 如果剩余自旋时间大于1000纳秒，那么阻塞线程，否则长时间自旋耗费CPU资源
                    nanosTimeout > spinForTimeoutThreshold)
                    // 阻塞线程指定纳秒数
                    LockSupport.parkNanos(this, nanosTimeout);
                // 获取线程中断状态，并且重置中断标志
                if (Thread.interrupted())
                	// 如果线程已经被中断，那么直接抛出中断异常
                    throw new InterruptedException();
            }
        } finally {
        	// 如果线程获取锁失败，那么需要清理等待竞争队列中，node节点前面的已经获取锁的节点
        	// 换句话说就是清理node前面已经获取到锁的线程
        	// 同时如果node前面的线程节点的等待状态是0或者1(已经获取锁，但是线程还是阻塞)，那么就唤醒线程
        	// 可以理解为：唤醒已经获得锁或者将要获得锁的线程干活
            if (failed)
                cancelAcquire(node);
        }
    }
    
    
    
    private void setHead(Node node) {
        head = node;
        // 清空头结点的线程
        node.thread = null;
        // 清空头结点的前继
        node.prev = null;
    }
    

### 2.3 lock

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/450f3ab7f2326405047d0c4864ded4a3.png)  
ReentrantLock中的lock方法代理指向了ReentrantLock内部实现的Sync类的lock方法，请跳转到3.1.

### 2.4 lockInterruptibly

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/322fa3da818abf44ad11a5b3313374f3.png)  
ReentrantLock的lockInterruptibly方法代理指向ReentrantLock内部类Sync的acquireInterruptibliy方法。  
在ReentrantLock的内部类Sync中并没有对AQS的acquireInterruptibly方法做任何修改。所以调用的实际上是AQS的acquireInterruptibly方法。  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
中的5.6.4小节

### 2.5 unlock

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f688a66c8246c044fecd2b5f115acdc6.png)  
ReentrantLock的unlock方法，先是调用AQS的release方法，在release方法中调用ReentrantLock内部Sync的tryRelease方法。  
首先AQS的release方法请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
的5.6.3小节。  
在AQS的release中会调用tryRelease方法，而ReentrantLock的内部类Sync实现了tryRelease方法，请跳转本文3.3。

### 2.6 newCondition

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/eb5a83a286436e558b7af8ab4fb7aeae.png)  
newCondition代理指向ReentrantLock内部类的newCondition方法，请跳转3.5.

### 2.7 其他方法

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0ce00c36ac294e34a809ed547ea0996a.png)

## 3\. ReentrantLock 内部Sync实现了AQS

这是ReentrantLock内部Sync的结构：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/db0efb85144711872482f61cba7e1a55.png)

### 3.1 lock

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/dbbd77da8b144acb76ece264ec9f5bb9.png)  
在ReentrantLock内部类Sync中的lock方法是抽象方法，要求子类实现的，所以真正的实现是ReentrantLock内部Sync的子类NonfairSync和FairSync的lock方法。  
请跳转4.1,4.3.

### 3.2 nonfairTryAcquire–不公平的尝试获取锁

这是nonfairTryAcquire的时序图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3610025139d41df27a5d05d0781799c6.png)
    
    
    // 不公平的尝试获取锁，在ReentrantLock的tryLock方法中，传入的值是1
    final boolean nonfairTryAcquire(int acquires) {
    	// 获取当前线程
        final Thread current = Thread.currentThread();
        // 获取线程节点的等待状态
        int c = getState();
        // 如果等待状态等于0，表示锁空闲
        if (c == 0) {
        	// 设置线程节点的等待状态是取消(换种理解方向就是线程节点的线程已经获取到了锁，不需要再进行竞争了)
            if (compareAndSetState(0, acquires)) {
            	// 设置锁持有的线程是当前线程(锁持有线程是AOS的属性)
                setExclusiveOwnerThread(current);
                // 返回尝试获取锁成功
                return true;
            }
        }
        // 如果等待状态不等于0，表示锁已经被线程持有了
        // 如果持有锁的线程和当前线程相同，表示当前线程已经获取了锁，现在正在重入
        else if (current == getExclusiveOwnerThread()) {
        	// 得到重入次数(或者说重入层数)
            int nextc = c + acquires;
            // 重入次数应该大于等于0
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            // 重入次数加1
            setState(nextc);
            // 可重入，所以返回尝试获取锁成功
            return true;
        }
        // 否则获取锁失败
        return false;
    }
    

### 3.3 tryRelease

这是Sync的tryRelease方法的时序图：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d890cf6c60bd2226ff7a4c99f27f4f89.png)
    
    
    // 这里传入的值是1，表示每释放一次，锁重入层数减小1层
    protected final boolean tryRelease(int releases) { // releases = 1
        int c = getState() - releases; // 获取当前锁重入层数，减去释放的层数，得到锁释放后的层数
        if (Thread.currentThread() != getExclusiveOwnerThread()) // 如果不是当前线程持有锁，那么锁释放直接抛出异常(释放不属于自己的锁（或者自己还未获取锁，而是直接越权释放其他线程的锁）)
            throw new IllegalMonitorStateException();
        // 定义锁释放的状态，初始化为失败
        boolean free = false;
        // 如果锁释放后重入层数为0，表示当前线程持有的锁已经全部释放，重入层数为0
        if (c == 0) {
        	// 修改锁释放状态为释放
            free = true;
            // 清空锁持有线程(当前线程已经用完，且完全释放了锁)
            setExclusiveOwnerThread(null);
        }
        // 更新线程节点的等待状态为0（表示锁空闲）(此时线程节点是头结点
        // (头结点存储线程，因为在独占模式下，只有头结点的后继节点才是锁接下来持有的线程；
        // 当抬头节点的后继获取到锁后，会更新头结点为后继节点))
        // 当头结点的锁释放后，头结点的后继节点的线程会自旋，
        // 自旋执行ReentrantLock中NonfairSync和FairSync实现的tryAcquire方法
        setState(c);
        // 返回锁释放状态
        return free;
    }
    

### 3.4 isHeldExclusively

是否是独占模式，只有condition才会用到  
这个方法在ReentrantLock中主要可以判断当前线程是不是获取到了锁：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/72af95f32c62d0963eb2c53f9b1e0306.png)  
将当前线程与锁持有线程进行比较，如果相等，则返回true，表示当前线程已经获取到了锁，否则返回false，表示当前线程没有获取到锁。

### 3.5 newCondition

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5942848d2d8f17eb21ff46e04363c978.png)  
newCondition的方法很简单，直接new 一个AQS的内部类ConditionObject。  
AQS的ConditionObject请看  
[AQS的Condition源码解析](<https://blog.csdn.net/a18792721831/article/details/106889626>)

### 3.6 getOwner

这个方法获取线程节点的线程的方法。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b23066bb74fd63a00128a9a1ec0199e4.png)  
这个方法返回线程节点存储的线程。  
线程节点的状态等于0表示锁空闲，那么就表示线程节点对应的线程已经被清空了(头结点)。

### 3.7 getHoldCount

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/64d5cf059443f38fc40f92cb7a64158e.png)  
如果是独占模式，那么返回当前线程持有锁的层数。

### 3.8 isLocked

查询此锁是否由任意线程保持。此方法用于监视系统状态，不用于同步控制。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/48468eaa4b23fd2658dd624d64af26de.png)

### 3.9 readObject

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/53f69d742baad8a912e7a9c892b240db.png)  
锁在从持久化数据中读取的时候，所有线程节点都会被设置为初始化状态。

## 4\. ReentrantLock 内部NonfairSync和FairSync继承Sync

### 4.1 FairSync–lock

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8b507200bb8ef4c4e7bb8a24316b47e2.png)  
FairSync的lock方法，使用的是AQS的acquire方法。  
AQS的acquire方法请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
中的第5.6.2小节  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d42fa42ed0defa87aee00cc6f910e896.png)  
lock的公平性是由FairSync的tryAcquire方法实现的。

### 4.2 FairSync–tryAcquire

这是tryAcquire的时序图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/83c9e364872dd1f2fe07545442e1dce1.png)
    
    
    // 公平尝试获取锁，这里传入的值是1
    protected final boolean tryAcquire(int acquires) {
    	// 获取当前线程
        final Thread current = Thread.currentThread();
        // 获取线程节点的等待状态
        int c = getState();
        // 如果线程节点的等待状态为0表示锁空闲
        if (c == 0) {
        	// 当前线程前面是否还有等待的线程，没有返回false
            if (!hasQueuedPredecessors() &&
            	// 如果当前线程前面没有等待的线程，那么就当前线程的节点的等待状态设置为取消，表示当前线程节点
            	// 已经获取到了锁，不需要再参与调度了
                compareAndSetState(0, acquires)) {
                // 设置锁持有线程是当前线程
                setExclusiveOwnerThread(current);
                // 返回公平获取锁成功
                return true;
            }
        }
        // 如果线程节点的等待状态不是0，表示锁不空闲，锁已经被线程持有了
        // 如果持有锁的线程和当前线程相等，那么表示锁重入
        else if (current == getExclusiveOwnerThread()) {
        	// 将锁重入层数加1
            int nextc = c + acquires;
            // 锁重入层数应该大于等大于0
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            // 每重入一次，锁的重入层数加1
            setState(nextc);
            // 返回公平获取锁成功
            return true;
        }
        // 公平获取锁失败
        return false;
    }
    
    
    
    // 这个方法是AQS的方法;在等待竞争队列中当前线程之前是否还有其他线程
    // 只需要看等待竞争队列中第一个有效的节点的线程和当前线程是不是相等即可，因为第一个有效的和当前不等，或者队列空，那么就是没有
    // 没有返回false,否则返回true
    public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        // 获取等待竞争队列的队列尾
        Node t = tail; // Read fields in reverse initialization order
        // 获取等待竞争队里的队列头
        Node h = head;
        // 定义头结点的后继节点
        Node s;
        return h != t && // 如果头结点和尾节点重合，那么表示等待竞争队列是一个空队列(里面有头结点，但是头结点不存储线程，不是线程节点)
            ((s = h.next) == null || s.thread != Thread.currentThread());
            // 只有头结点，也是相当于等待竞争队列中没有其他线程了
            // 或者是当前线程与第一个有效的线程节点的线程不相等
    }
    

### 4.3 NonfairSync–lock

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1aaa58f115867d8536cc7254b5213164.png)  
如果当前线程的线程节点已经获取到锁了，或者说锁空闲，那么就设置当前线程节点的等待状态是取消（即不需要参与竞争）。  
然后设置锁持有的线程是当前线程。  
如果当前线程的线程节点还没有获取锁，那么调用AQS的acquire自旋获取锁。  
获取锁是否公平由tryAcquire方法决定。

### 4.4 NonfairSync–tryAcquire

![这是不公平尝试获取锁的时序图](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/73e756db8f048df4139f777f17cd980c.png)  
不公平尝试获取锁的方法直接代理使用ReentrantLock内部Sync的nonfairTryAcquire方法。  
详细请见3.2.  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f14d2fcbe00bf26fd6fa078919f16d67.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b44898ac1f9b58ea6778b3ab6874e841.png)

## 5\. ReentrantLock 的构造

我们知道ReentrantLock内部类Sync，且Sync有两个实现NonfairSync和FairSync。那么我们怎么决定使用FairSync还是NonfairSync呢？  
答案就在ReentrantLock的构造方法中。

### 5.1 ReentrantLock

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/62fca4513e672db07491029f56a28f47.png)  
使用无参数的构造方法，调用的是不公平重入锁。

### 5.2 ReentrantLock(boolean)

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/01df756f3a75ebca9afb15d8ec365dbb.png)  
使用有参数的构造方法，则根据传入的boolean值，决定使用哪种重入锁。  
true:公平重入锁  
false:不公平重入锁

## 6\. 总结

通篇阅读了ReentrantLock的源码后，不仅仅了解了ReentrantLock的实现结构，以及ReentrantLock的核心原理。  
更加重要的是，现在，你可以自由的使用ReentrantLock，不用再模仿使用。  
不同的场景，根据ReentrantLock的原理，很容易找到最合适的方法调用。
