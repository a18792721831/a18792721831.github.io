---
layout: post
title: "Java基础--Semaphore--计数信号量锁"
date: 2020-07-04 13:29:16 +0800
categories: [信号量锁, Semaphore源码, Semaphore如何使用, Semaphore原理, 常用线程同步锁的对比]
description: "本文深入解析Java中的Semaphore类，包括其UML结构、属性、构造方法、核心方法如acquire和release的工作原理，以及如何用于资源管理和线程同步。通过示例代码展示了Semaphore在实际应用中的作用。"
keywords: 信号量锁, Semaphore源码, Semaphore如何使用, Semaphore原理, 常用线程同步锁的对比
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107071225
> - 发布时间：2020-07-04 13:29:16
> - 阅读量：652
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, 多线程
> - 标签：#信号量锁, #Semaphore源码, #Semaphore如何使用, #Semaphore原理, #常用线程同步锁的对比

## 摘要

文章浏览阅读652次。本文深入解析Java中的Semaphore类，包括其UML结构、属性、构造方法、核心方法如acquire和release的工作原理，以及如何用于资源管理和线程同步。通过示例代码展示了Semaphore在实际应用中的作用。

---

#### Java基础--Semaphore--计数信号量锁

  * 1\. Semaphore 简介
  *     * 1.1 Semaphore 的 UML 结构
    * 1.2 Semaphore的属性和方法
  * 2\. Semaphore 的构造
  *     * 2.1 Samephore(int)
    * 2.2 Samephore(int,boolean)
  * 3\. Semaphore 的方法
  *     * 3.1 acquire
    * 3.2 acquire(int)
    * 3.3 acquireUninterruptibly
    * 3.4 acquireUninterruptibly(int)
    * 3.5 availablePermits
    * 3.6 drainPermits
    * 3.7 getQueuedThreads
    * 3.8 getQueueLength
    * 3.9 hasQueuedThreads
    * 3.10 isFair
    * 3.11 reducePermits
    * 3.12 release
    * 3.13 release(int)
    * 3.14 tryAcquire
    * 3.15 tryAcquire(int)
    * 3.16 tryAcquire(int,long,TimeUnit)
    * 3.17 tryAcquire(long,TimeUnit)
  * 4\. Semaphore 的Sync
  *     * 4.1 Sync的构造
    * 4.2 nonfairTryAcquireShared
    * 4.3 reducePermits
    * 4.4 tryReleaseShared
  * 5\. Semaphore 的FairSync
  *     * 5.1 FairSync 构造
    * 5.2 tryAcquireShared
  * 6\. Semaphore 的NonfairSync
  *     * 6.1 NonfairSync的构造
    * 6.2 tryAcquireShared
  * 7\. AQS实现的方法
  *     * 7.1 getQueuedThreads
    * 7.2 getQueueLength
    * 7.3 tryAcquireShharedNanos
  * 8\. Semaphore 示例
  * 9\. 总结

## 1\. Semaphore 简介

一个计数信号量。从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。  
Semaphore 通常用于限制可以访问某些资源（物理或逻辑的）的线程数目。

### 1.1 Semaphore 的 UML 结构

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b2bb91d5551e21645c981ad511da3615.png)  
Semaphore和ReentrantLock的结构相同。  
都是内部的Sync继承了AQS，内部还有FairSync和NonfairSync继承了Sync。

### 1.2 Semaphore的属性和方法

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/10203af0059056b543bf0e79e2ef871e.png)

## 2\. Semaphore 的构造

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c66f5222a664a040907dd2aaea2f5d66.png)

### 2.1 Samephore(int)

创建指定信号量的，不公平的信号量锁。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/150aa5c277a8a0762fac9a971e24e16b.png)  
根据传入的值，调用不公平的信号量同步锁。

### 2.2 Samephore(int,boolean)

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/68c77fed3dcd4794f60d0cbe47411742.png)  
根据传入的boolean值，选择是否是公平的处理方式。  
然后调用FairSync或者是NonfaireSync类的构造。

## 3\. Semaphore 的方法

接下来看下Semaphore提供的方法。

### 3.1 acquire

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f363ec35bbe6dc7e53334cde5fb20125.png)  
直接调用AQS的accquireSharedInterruptibly,传入1.  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fd7686ac3486b920d7ae171739b4a394.png)  
AQS的acquireSharedInterruptibly方法会调用AQS子类实现的tryAcquireShared方法。

### 3.2 acquire(int)

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/537c89e198da2cfb02b5f7b55642dd1b.png)  
如果请求的数量小于0，那么抛出参数异常。  
否则调用AQS的acquireSharedInterruptibly方法。  
最终还是调用Sync的nonfairTryAcquireShared和FairSync的tryAcquireShared方法。

### 3.3 acquireUninterruptibly

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/56a5c248344e6adff2833478d8c9afad.png)  
调用AQS的acquireShared方法。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a1497928f79985769854b1f597b11639.png)  
AQS的acquireShared方法会调用子类实现的tryAcquireShared方法。  
最终调用的还是Sync的nonfairTryAcquireShared和FairSync的tryAcquireShared方法。

### 3.4 acquireUninterruptibly(int)

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5cd26b8802a61f9a4c44352777f0c23d.png)  
尝试获取共享锁，不响应中断。获取请求的资源。  
如果请求的资源数量小于0，那么抛出参数异常。  
如果参数校验通过，调用就AQS的acquireShared方法。  
最终调用的还是Sync的nonfairTryAcquireShared和FairSync的tryAcquireShared方法。

### 3.5 availablePermits

返回此信号量中当前可用的许可数。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/287675151e0b4d44812e839f225b7380.png)  
调用Sync的getPermits方法。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a4e94b15382d7a7e2fddc80a0b162f53.png)  
直接返回锁状态(资源现在空闲的数量)

### 3.6 drainPermits

获取并返回立即可用的所有许可。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8dbe315659d250552415f7de52757360.png)  
调用Sync的drainPermits方法  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/23682e4c8a1d9ed19263ee6e308cd3c7.png)  
强制将锁状态设置为0.即可用资源数量为0.

### 3.7 getQueuedThreads

返回一个 collection，包含可能等待获取的线程。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f55139c9b65cb57db0ca33bdf16a450a.png)  
调用AQS的getQueuedThreads方法

### 3.8 getQueueLength

返回正在等待获取的线程的估计数目。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/18c3ef2e910eb9eb49fe361e0068c321.png)

### 3.9 hasQueuedThreads

查询是否有线程正在等待获取。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/11534853713e59d3801fbd6e5de817df.png)  
调用的是AQS的hasQueuedThreads方法  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/13101ba8a59ad567353ba66e49f5b79f.png)  
直接判断等待竞争队列是否为空。

### 3.10 isFair

如果此信号量的公平设置为 true，则返回 true。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/119fc4a261b9455fdf706699de7edf53.png)  
根据全局的Sync的对象，判断sync对象是否是FairSync的实例对象。

### 3.11 reducePermits

根据指定的缩减量减小可用许可的数目。(减少可用资源数量)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b56cd7e2220bdedc40f4b1a38b598297.png)  
如果传入的指定的缩减量小于0，那么抛出参数异常。  
通过参数校验后，调用Sync的reducePermits方法

### 3.12 release

释放一个许可，将其返回给信号量。(释放资源，将可用的资源数量增加)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2fc370e548abee72807e3a862b459996.png)  
调用AQS的releaseShared方法  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/14f568d1516cd49245e1184aa4f4b01c.png)  
调用的是AQS的子类实现的tryReleaseShared方法。  
也就是Semaphore的Sync的tryReleaseShared方法。

### 3.13 release(int)

释放指定个许可，将其返回给信号量。(释放资源，将可用的资源数量增加)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/60e52d09351d360c9dadbbc93e9949be.png)  
先进行参数校验，如果释放的资源个数小于0，那么抛出参数异常。(你不能打着还钱的幌子借钱)  
然后调用的是AQS的realeaseShared方法。  
和3.12相同，最终调用的是Semaphore的Sync的tryReleaseShared方法。

### 3.14 tryAcquire

从此信号量获取一个许可，在提供这些许可前一直将线程阻塞，或者线程已被中断。 （简单来说就是借钱）  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/03489b6ccb6739e4522640dcb329909b.png)  
调用的是Semaphore的Sync的nonfairTryAcquireShared方法。  
如果调用Sync的nonfairTryAcquireShared方法成功后，返回目前可用的资源数量，大于0表示获取成功。  
(你向地主借钱，不能把地主借的地主负债了，地主也不会自己借钱然后在借给你)

### 3.15 tryAcquire(int)

从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞，或者线程已被中断。 （简单来说就是借钱）  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fdc4a42807d1d453ad2296919c0f1307.png)  
先会进行参数校验，如果请求的资源数量小于0，那么抛出参数异常。  
（没有会打着借钱的幌子给你钱）  
调用的也是Sync的nonfairTryAcquireShared方法。

### 3.16 tryAcquire(int,long,TimeUnit)

如果在给定的等待时间内此信号量有可用的所有许可，并且当前线程未被中断，则从此信号量获取给定数目的许可。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/11d70980db8d799787b1fc794cb23162.png)  
第一个一定是参数校验，参数都不合法，后面也没有继续的必要了。  
调用的是AQS的tryAcquireSharedNanos方法。  
最终调用的也是FairSync或者NonfairSync的tryAcquireShared方法

### 3.17 tryAcquire(long,TimeUnit)

如果在给定的等待时间内此信号量有可用的所有许可，并且当前线程未被中断，则从此信号量获取给定数目的许可。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f6c781b813bcdaeb999d90308e6bc741.png)

## 4\. Semaphore 的Sync

### 4.1 Sync的构造

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/33b7dcef270fab5883f0aa3d03ce9245.png)  
设置锁状态为传入的值。

### 4.2 nonfairTryAcquireShared
    
    
    // 尝试获取共享锁
    final int nonfairTryAcquireShared(int acquires) {
    	// 自旋
        for (;;) {
        	// 获取锁状态(现有资源数量)
            int available = getState();
            // 锁状态减去请求的数量(剩余资源数量)
            int remaining = available - acquires;
            if (remaining < 0 || // 锁状态小于0(剩余资源数量小于0，表示不够请求的数量)
                compareAndSetState(available, remaining)) // 如果够，则更新锁状态(设置剩余资源数量)
               	// 返回锁状态(剩余资源数量)
                return remaining;
        }
    }
    

### 4.3 reducePermits
    
    
    // 以指定的缩减量减少可用资源数量
    final void reducePermits(int reductions) {
    	// 开始自旋
        for (;;) {
        	// 获取现有的可用的资源数量(锁状态)
            int current = getState();
            // 计算出减去请求后的可用资源数量(锁状态)
            int next = current - reductions;
            // 如果减完发现大于原有值(传入值是负数)，那么抛出错误(因为前面已经进行参数校验了，此时参数小于0，确实是错误，而不是异常)
            if (next > current) // underflow
                throw new Error("Permit count underflow");
            // 使用CAS进行更新锁状态
            if (compareAndSetState(current, next))
            	// 更新成功直接结束，否则进行自旋
                return;
        }
    }
    

### 4.4 tryReleaseShared

释放资源。将资源归还给信号量锁。
    
    
    // 尝试释放共享锁
    protected final boolean tryReleaseShared(int releases) {
    	// 进入自旋
        for (;;) {
        	// 获取当期锁状态(可用的资源数量)
            int current = getState();
            // 当前资源数量加上释放的资源数量(可用资源增加(归还))
            int next = current + releases;
            // 做检测，如果归还完反而可用资源变少 ，那么就是发生错误
            if (next < current) // overflow
                throw new Error("Maximum permit count exceeded");
            // 使用cas更新锁状态(可用资源)
            if (compareAndSetState(current, next))
            	// 返回释放成功(归还成功)
                return true;
        }
    }
    

## 5\. Semaphore 的FairSync

### 5.1 FairSync 构造

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6a05f6d4557b4fde079ebb36d9239dfd.png)  
调用父类的构造方法，传入permits.

### 5.2 tryAcquireShared
    
    
    // 尝试获取共享锁
    protected int tryAcquireShared(int acquires) {
    	// 自旋
        for (;;) {
        	// 当前线程前面是否还有等待的线程
            if (hasQueuedPredecessors())
            	// 如果前面还有线程在等待，那么当前线程尝试获取失败
                return -1;
            // 获取锁状态
            int available = getState();
            // 锁状态与请求数量的差值(看看是不是够)
            int remaining = available - acquires;
            if (remaining < 0 || // 差值小于0，表示不够。即使将现有的全部给你，也不够
                compareAndSetState(available, remaining)) // 如果够，则将可用的数量减去请求的个数
                return remaining; // 返回剩余的个数
        }
    }
    

获取当前线程在等待竞争队列中有没有前继节点。  
简单来说，就是获取当前线程前面还有没有等待线程。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/639f89047979d8a69dd7dd2fea2d8724.png)  
如果等待竞争队列不为空，那么头结点的后继节点为空或者等待线程不是当前现场，那么就表示当前线程前面还有等待的线程。  
(不会存在head != tail && head.next == null)

## 6\. Semaphore 的NonfairSync

### 6.1 NonfairSync的构造

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/dea7a17281d0cd62d8c525b03239aebf.png)  
调用父类的构造方法，传入permits.

### 6.2 tryAcquireShared

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3d570a2dac26e9eed0ea1686e946678a.png)  
直接调用Sync的nonfairTryAcquireShared方法。

## 7\. AQS实现的方法

### 7.1 getQueuedThreads

获取等待竞争的线程集合。（不准确，因为等待竞争队列是试试变化的）
    
    
    // 获取等待竞争队列的线程集合
    public final Collection<Thread> getQueuedThreads() {
    	// 构造返回集合对象
        ArrayList<Thread> list = new ArrayList<Thread>();
        // 从等待竞争队列的尾节点开始遍历，只要线程节点不为空，那么就加入返回集合中。
        for (Node p = tail; p != null; p = p.prev) {
            Thread t = p.thread;
            if (t != null)
                list.add(t);
        }
        // 返回线程集合
        return list;
    }
    

### 7.2 getQueueLength

获取等待竞争的线程的数量
    
    
    // 获取等待竞争的线程的数量
    public final int getQueueLength() {
    	// 初始化数量
        int n = 0;
        // 从等待竞争队列的尾节点开始遍历，只要线程节点不为空，那么就将线程数量++
        for (Node p = tail; p != null; p = p.prev) {
            if (p.thread != null)
                ++n;
        }
        // 返回线程数量
        return n;
    }
    

### 7.3 tryAcquireShharedNanos
    
    
    // 尝试获取共享锁，有超时时间，响应中断
    public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout) // arg = 1
            throws InterruptedException {
        // 获取线程中断标志，并重置中断标志
        if (Thread.interrupted())
        	// 如果线程已经中断，那么直接抛出中断异常，快速结束
            throw new InterruptedException();
        // 否则就会调用tryAcquireShared 和 doAcquireSharedNanos方法
        return tryAcquireShared(arg) >= 0 ||
            doAcquireSharedNanos(arg, nanosTimeout);
    }
    
    

tryAcquireShared方法请看3.2  
doAcquireSharedNanos请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
的5.6.2.4和5.6.5小节

## 8\. Semaphore 示例

我们以地主为例。  
假设村子里有一个地主，他有100元。  
村子里面每个人都需要用钱进行交易，有些人足够，有些人则不够。  
足够的人可以顺利完成交易。  
不够的人需要向地主借钱完成交易。
    
    
    public class MySemaphore {
    
        public static void main(String[] args) {
        	// 定义资源总量为100
            int count = 100;
            // 创建公平的信号量锁
            Semaphore semaphore = new Semaphore(count, true);
            Runnable runnable = () -> {
            	// 获取当前线程的线程名字
                String name = Thread.currentThread().getName();
                try {
                	// 随机生成当前线程需要的数量
                    int need = (int) (Math.random() * 100);
                    // 阻塞获取资源
                    semaphore.acquire(need);
                    // 等到资源后输出剩余资源数量
                    System.out.println(name + " get " + need + "\t, now semaphore have " + semaphore.availablePermits());
                    // 线程占用资源一定的是时间
                    Thread.sleep((long) (Math.random() * 10000)); // 睡眠10秒内
                    // 释放资源
                    semaphore.release(need);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
            for (int i = 0; i < 10; i++) {
            	// 创建线程
                new Thread(runnable, "村名" + i).start();
            }
        }
    
    }
    

执行结果  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ece666e893c9d471363df876872aceeb.png)  
从执行结果中可以看出村名5和村名1是一起去借钱的。因为借完钱后，剩余的钱相同。  
同理，村名3和村名2也是一起去的。  
如果我们将每个村民的需要的钱数设置为10以内。  
也就是地主完全有钱。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7a678b973406a4b3b56e75dd94dc7ebc.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/adb623fd218f2278ffe210d96c8b59a6.png)  
因为每个村名借钱的钱数都满足，所以不存在线程等待的问题，所以看起来就很顺滑。

## 9\. 总结

Semaphore是一个很有用的同步辅助类，但是感觉更多的使用在资源管理等方面。  
看了这么多的同步类：  
[ReentrantLock](<https://blog.csdn.net/a18792721831/article/details/107005826>)  
[ReentrantReadWriteLock](<https://blog.csdn.net/a18792721831/article/details/107026498>)  
[CountDownLatch](<https://blog.csdn.net/a18792721831/article/details/107070198>)  
[Cyclicbarrier](<https://blog.csdn.net/a18792721831/article/details/107071166>)  
个人感觉：  
ReentrantLock适合大多数的普遍的线程同步，对于读多还是写多不确定时。  
ReentrantReadWriteLock适合较高性能要求的线程同步，较为适合读多写少的场景。  
CountDownLatch适合倒计时类的线程同步，而且达到条件后，做的操作是一次性的那种。  
CyclicBarrier适合做线程间统一切面处理的操作，一批线程，所有线程都到达指定的点后，做一个预定的操作，然后各个线程在继续运行。  
Semaphore则适合资源管理，线程占用资源，就将可用资源减少，线程归还资源，就将可用资源增加。

我感觉，在理解Semaphore的时候，结合借钱还钱可能比较好理解。`<~^v^~>`
