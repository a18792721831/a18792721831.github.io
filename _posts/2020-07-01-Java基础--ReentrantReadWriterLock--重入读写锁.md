---
layout: post
title: "Java基础--ReentrantReadWriterLock--重入读写锁"
date: 2020-07-01 14:26:09 +0800
categories: [可重入读写锁, 可重入读写锁源码解读, 带你阅读可重入读写锁, 可重入读写锁的锁降级, 可重入读写锁实现原理]
description: "本文详细剖析了ReentrantReadWriteLock的内部结构、工作原理及其读写锁的实现细节，包括锁状态管理、读写锁的获取与释放过程，以及锁升级与降级的概念。"
keywords: 可重入读写锁, 可重入读写锁源码解读, 带你阅读可重入读写锁, 可重入读写锁的锁降级, 可重入读写锁实现原理
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107026498
> - 发布时间：2020-07-01 14:26:09
> - 阅读量：535
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, 多线程
> - 标签：#可重入读写锁, #可重入读写锁源码解读, #带你阅读可重入读写锁, #可重入读写锁的锁降级, #可重入读写锁实现原理

## 摘要

文章浏览阅读535次。本文详细剖析了ReentrantReadWriteLock的内部结构、工作原理及其读写锁的实现细节，包括锁状态管理、读写锁的获取与释放过程，以及锁升级与降级的概念。

---

#### Java基础--ReentrantReadWriterLock--重入读写锁

  * [1\. ReentrantReadWriterLock的整体结构](<#1_ReentrantReadWriterLock_1>)
  *     * [1.1 ReentrantReadWriterLock的UML图](<#11_ReentrantReadWriterLockUML_2>)
    * [1.2 ReentrantReadWriterLock的属性、方法](<#12_ReentrantReadWriterLock_4>)
  * [2\. ReentrantReadWriterLock 实现ReadWriterLock接口](<#2_ReentrantReadWriterLock_ReadWriterLock_6>)
  *     * [2.1 readLock](<#21_readLock_11>)
    *       * [2.1.1 tryLock](<#211_tryLock_14>)
      * [2.1.2 tryLock(long,TimeUnit)](<#212_tryLocklongTimeUnit_18>)
      * [2.1.3 lock](<#213_lock_41>)
      * [2.1.4 lockInterruptibly](<#214_lockInterruptibly_44>)
      * [2.1.5 unlock](<#215_unlock_51>)
      * [2.1.6 newCondition](<#216_newCondition_71>)
    * [2.2 writerLock](<#22_writerLock_74>)
    *       * [2.2.1 tryLock](<#221_tryLock_77>)
      * [2.2.2 tryLock(long,TimeUnit)](<#222_tryLocklongTimeUnit_80>)
      * [2.2.3 lock](<#223_lock_89>)
      * [2.2.4 lockInterruptibly](<#224_lockInterruptibly_98>)
      * [2.2.5 unlock](<#225_unlock_104>)
      * [2.2.6 newCondition](<#226_newCondition_113>)
  * [3\. ReentrantReadWriterLock 内部类Sync继承了AQS](<#3_ReentrantReadWriterLock_SyncAQS_120>)
  *     * [3.1 tryReadLock](<#31_tryReadLock_125>)
    * [3.2 tryAcquireShared](<#32_tryAcquireShared_186>)
    * [3.3 fullTryAcquireShared](<#33_fullTryAcquireShared_234>)
    * [3.4 tryReleaseShared](<#34_tryReleaseShared_328>)
    * [3.5 tryWriteLock](<#35_tryWriteLock_383>)
    * [3.6 tryAcquire](<#36_tryAcquire_417>)
    * [3.7 tryRelease](<#37_tryRelease_458>)
  * [4\. 继承于Sync的FairSync](<#4_SyncFairSync_485>)
  * [5\. 继承于Sync的NonfairSync](<#5_SyncNonfairSync_493>)
  * [6\. ReadLock](<#6_ReadLock_498>)
  * [7\. WriteLock](<#7_WriteLock_502>)
  * [8\. HoldCounter](<#8_HoldCounter_505>)
  * [9\. ThreadLocalHoldCounter](<#9_ThreadLocalHoldCounter_512>)
  * [10\. ReentrantReadWriteLock的构造](<#10_ReentrantReadWriteLock_516>)
  *     * [10.1 ReentrantReadWriteLock的无参构造](<#101_ReentrantReadWriteLock_519>)
    * [10.2 ReentrantReadWriteLock的有参数构造](<#102_ReentrantReadWriteLock_522>)
  * [11\. 锁升级与降级](<#11__527>)
  *     * [11.1 锁降级](<#111__541>)
    * [11.2 锁升级](<#112__549>)
  * [12\. 总结](<#12__562>)

## 1\. ReentrantReadWriterLock的整体结构

### 1.1 ReentrantReadWriterLock的UML图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b90359f2596d2384da5e3db420bb695d.png)

### 1.2 ReentrantReadWriterLock的属性、方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a7143d49a665a4a913fe1fdb55e81a63.png)

## 2\. ReentrantReadWriterLock 实现ReadWriterLock接口

既然ReentrantReadWriteLock实现了ReadWriteLock接口，那么就需要实现ReadWriteLock的方法  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ac856b3f5549ecb2523a72d48f25782f.png)  
不管是readLock还是writeLock返回的都是Lock类型，Lock则必须实现这些方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82f2f5474c5a9daf089b1ad2cb685d0f.png)

### 2.1 readLock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88a37aef3e45a06e516fbcfe3e8e3cec.png)  
readLock方法直接返回局部变量readerLock的值。

#### 2.1.1 tryLock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4c979eb306ba03716e44466998b6ee1.png)  
直接调用Sync的tryReadLock方法，请跳转3.1.  
sync,tryReadLoack的方法总结就是看看当前线程有没有获取读锁，如果已经获取了读锁，那么将当前线程的重入层数++，如果当前线程没有获取读锁，且不能获取锁，那么尝试获取读锁失败。否则当前线程是第一次获取读锁，将当前线程加入读锁持有线程映射表中。

#### 2.1.2 tryLock(long,TimeUnit)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c0a80a1c4457f0699b487ddce2f74fb.png)  
带有超时时间的尝试获取锁，则是调用AQS的带有超时时间的尝试获取共享锁的方法。  
这是AQS的带有超时时间的尝试获取共享锁的方法的时序图如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/273ebffa5a39ec7e51a67a6812c350d4.png)  
巨复杂！！！不过，不要怕，我们一步一步看。
    
    
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

#### 2.1.3 lock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b3ba1541bd8f949f86816b2155547b41.png)  
直接调用Sync的acquireShared方法，请见3.2.

#### 2.1.4 lockInterruptibly

阻塞获取锁，响应中断。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/56a69bfe0823b4ff3c8256e5f790e55b.png)  
直接调用AQS的acquireSharedInterruptibly方法。  
AQS的acquireSharedInterruptibly方法请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
的5.6.4 小节

#### 2.1.5 unlock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/45e6b44a423dd34cb0af1e119e89a021.png)  
直接调用AQS的releaseShared方法。
    
    
    // 释放共享锁
    public final boolean releaseShared(int arg) {
    	// 调用AQS子类实现的方法，尝试释放共享锁
        if (tryReleaseShared(arg)) {
        	// 自旋释放共享锁
            doReleaseShared();
            // 返回共享锁释放成功
            return true;
        }
        // 释放共享锁失败
        return false;
    }
    

在ReentrantReadWriteLock中，Sync继承了AQS。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f1af22815fb66a17f82d078c35579222.png)  
Sync的tryReleaseShared请见3.4

#### 2.1.6 newCondition

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25fe8ce30aa58c7fedbafff33219c355.png)  
读锁不允许使用condition,强行使用会直接抛出异常。

### 2.2 writerLock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88e1cab614cd926d56aaaa1266eaa573.png)  
writeLock方法直接返回局部变量readerLock的值。

#### 2.2.1 tryLock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6a1aacc4d8900d64cd23ef3bb6964485.png)  
调用Sync的tryWriteLock方法，请见3.5

#### 2.2.2 tryLock(long,TimeUnit)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e23e680fc8462c5243898633f6f18ae.png)  
调用AQS的tryAcquireNanos方法  
AQS的tryAcquireNanos方法请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
的5.6.5小节  
AQS的tryAcquireNanos方法会调用子类实现的tryAcquire实现独占锁的获取。  
在ReentrantReadWriteLock的内部类Sync实现了tryAcquire方法  
请见3.6

#### 2.2.3 lock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b412bd533d22bcfdc0ee9599700fb73b.png)  
直接调用AQS的acquire方法  
AQS的acquire方法请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
的5.6.2小节  
在AQS的acquire方法中会调用tryAcquire方法获取独占锁(写锁)  
而ReentrantReadWriteLock中的Sync中实现了tryAcquire方法  
请见3.6

#### 2.2.4 lockInterruptibly

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/36c70918e0981c94805787ad90fe6e3a.png)  
调用AQS的acquireInterruptibly方法。  
AQS的acquireInterruptibly方法请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
的5.6.4 小节

#### 2.2.5 unlock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/415dfdfb1203fc97980321c24c78a22f.png)  
调用AQS的release方法  
AQS的release方法请看  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
的5.6.3 小节  
AQS的release方法会调用子类实现的tryRelease方法释放独占锁(写锁)  
ReentrantReadWriteLock中的Sync实现了tryRelease方法，  
请见3.7

#### 2.2.6 newCondition

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/17d03bf0884651d5ab7a515d2f3893cd.png)  
调用ReentrantReadWriteLock的Sync的newCondition方法  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c136ce450e18282bc2602fbd4ed272a8.png)  
ReentrantReadWriteLock的Sync的newCondition方法直接返回AQS的ConditionObject对象。  
请看  
[AQS的Condition源码解析](<https://blog.csdn.net/a18792721831/article/details/106889626>)

## 3\. ReentrantReadWriterLock 内部类Sync继承了AQS

Sync继承了AQS:  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44824ae2a0688be595ba65fe483e1570.png)  
Sync内部使用了HoldCounter和ThreadLocalHoldCounter  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8e1879fb64c7a00bc350b30473b41c0.png)

### 3.1 tryReadLock

这是tryReadLock的时序图  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3fd43810672c57f5b2bd57d9f9e436c7.png)
    
    
    // 尝试获取读锁
    final boolean tryReadLock() {
    	// 首先获取当前线程
        Thread current = Thread.currentThread();
        // 开始自旋
        for (;;) {
        	// 获取当前线程节点的状态(可以理解为现在持有读锁的线程数量，最大65535个线程)
            int c = getState();
            if (exclusiveCount(c) != 0 && // 如果读锁不空闲(只有线程节点的等待状态为0时，才能尝试获取锁，线程节点的等待状态为0表示锁空闲)
            // 这里获取的数量是读锁以独占模式使用时(其实就是锁现在是写锁)
            	// 如果现在锁是以写锁被使用时，读锁现在不可用
            	// 因为读锁每次重入，都会将锁状态的值增加65535，所以，上面判断现在锁是以写锁使用
                getExclusiveOwnerThread() != current) // 当前线程还未持有读锁
                // 如果写锁持有线程是当前线程，那么就表示在同一个线程内即获取写锁，又获取读锁
                // 根据串行语义一致性原则，同一个线程既要写，又要读，是完全没有问题的
                // 直接返回获取读锁失败
                return false;
            // 只有线程节点的等待状态为0，表示锁空闲，才能走到这里
            int r = sharedCount(c); // 获取持有读锁的线程数量(这里的数量是读锁以共享模式使用时)
            // 上面排除了锁是写锁使用的情况，现在是读锁
            // 这里可以理解为当前线程获取的读锁的充入层数，因为读写锁还是可重入锁
            if (r == MAX_COUNT) // 如果共享读锁现在持有锁的线程数量达到最大值31(锁现在是读锁)
                throw new Error("Maximum lock count exceeded"); // 那么直接抛出异常快速失败
            if (compareAndSetState(c, c + SHARED_UNIT)) { // 设置读锁的等待状态是c+65536 ，+65536是为了记录重入层数将c+SHARED_UNIT除以SHARED_UNIT就是重入层数
                if (r == 0) { // 如果锁的线程持有数量等于0，表示当前线程是第一个获取读锁的线程
                    firstReader = current; // 设置第一个获取读锁的线程是当前线程
                    // 这里不使用cas也没有关系，设置firstReader只是偏向哪个线程而已
                    firstReaderHoldCount = 1; // 设置第一个获取读锁的重入层数是1
                } else if (firstReader == current) { // 如果第一个获取读锁的线程重入
                	// 这里是偏向锁的实现，第一个获取读锁的线程，读锁释放后，再次获取读锁，不会加入线程映射表，不需要创建HoldCounter和ThreadLocalHoldCounter对象，会比较快
                    firstReaderHoldCount++; // 重入层数++
                } else {
                	// 当前线程不是第一个获取读锁的线程时，需要创建映射表
                    HoldCounter rh = cachedHoldCounter; // 获取缓存的线程重入数量对象
                    // HoldCounter是记录线程重入层数的，cachedHoldCounter是缓存对象
                    if (rh == null || rh.tid != getThreadId(current)) // 如果读锁持有线程为空或者读锁持有线程不是当前线程
                        cachedHoldCounter = rh = readHolds.get(); // 获取当前线程的重入数量对象，重入量对象是一个线程变量
                    else if (rh.count == 0) // 第一次进来，当前线程读锁重入次数为0
                    	// 将当前线程的重入数量对象初始化到线程变量
                        readHolds.set(rh);
                    // 每次获取读锁，都应该将重入层数++
                    rh.count++;
                }
                // 返回获取读锁成功
                return true;
            }
        }
    }
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0b953930c0b610f699f01a4211b269a.png)  
传入数量除以每一个独占线程的间隔进行与操作。可以理解为，返回c/65535  
为什么需要这样处理呢？  
因为在ReentrantReadWriteLock中state的高16位表示读锁，低16位表示写锁。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fcbfc2228526ee737eb1edb66a83626d.png)  
SHARED_SHIFT的值是16，那么EXCLUSIVE_MASK的值是65535  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6cda8dd8815f67b7ab52668d4f881d01.png)  
获取持有锁的线程。

### 3.2 tryAcquireShared

尝试获取共享锁  
时序图如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/79943cadd43b29348413d19af8b14ac0.png)
    
    
    // 尝试获取共享锁(读锁)
    protected final int tryAcquireShared(int unused) {
    	// 获取当前线程
        Thread current = Thread.currentThread();
        // 获取锁状态
        // 在AQS中有一个state，表示锁状态，state=0表示锁空闲，state>0表示锁占用
        // 在AQS中有两个队列：竞争等待队列，由AQS对象的head和tail引用；等待通知队列，由AQS内部类ConditionObject对象的fristWaiter和lastWaiter引用
        // AQS中两个队列的元素节点都是AQS内部的Node的实例对象
        // Node中的waitStatus是线程节点的等待状态(0:初始化;1:取消;-1:SIGNAL;-2:CONDITION;-3:PROPAGATE)
        int c = getState();
        if (exclusiveCount(c) != 0 && // 如果锁被以独占的方式持有(写锁)，那么共享锁(读锁)不能获取
            getExclusiveOwnerThread() != current) // 如果占有锁的线程不是当前线程(如果是当前线程表示写锁重入)
            // 返回尝试获取共享锁失败
            return -1;
        // 获取共享锁持有线程数(每有一个新线程进入，state就会加65536)
        int r = sharedCount(c); // c >>> 16 就等价于 c/65536
        if (!readerShouldBlock() && // 如果当前线程前面没有等待线程，那么当前线程就可以获取锁了，否则应该先处理当前线程前面的线程
            r < MAX_COUNT && // 共享锁持有线程数量没有超过最大值65535
            compareAndSetState(c, c + SHARED_UNIT)) { // 锁状态+65535
            // 如果锁持有线程数量等于0，表示当前线程是第一个获取锁的线程
            if (r == 0) { // 当前线程是第一个获取锁的线程
                firstReader = current; // 设置第一个获取读锁的线程是当前线程(这里是偏向锁，如果是第一个线程再次获取锁，那么会进行比较，相等会快速获得锁)
                firstReaderHoldCount = 1; // 第一个获取到读锁的线程第一次获取到读锁，这里理解是第一个获取到读锁的线程的重入层数是1
            } else if (firstReader == current) { // 第一个线程再次获取读锁(重入)
                firstReaderHoldCount++; // 重入层数++
            } else {
            	// 如果当前线程不是第一个获取读锁的线程，那么需要初始化当前线程的重入数量对象(线程变量)
                HoldCounter rh = cachedHoldCounter; // 获取缓存的线程重入数量对象
                if (rh == null || rh.tid != getThreadId(current)) // 如果缓存为空，或者缓存的数据不是当前线程的数据
                	// 那么更新缓存的重入线程为当前线程的重入数量对象
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0) // 如果重入层数是0，表示是完全释放读锁后，再次进入，此时缓存的重入数量对象是当前线程的重入数量对象
                	// 将缓存的重入数量对象设置为线程变量(为什么？存疑，应该与释放有关，释放后能会清理线程变量)
                    readHolds.set(rh);
                // 重入层数++
                rh.count++;
            }
            // 返回获取共享锁成功
            return 1;
        }
        return fullTryAcquireShared(current);
    }
    

### 3.3 fullTryAcquireShared

当当前线程前面还有等待读取的线程的时候，需要先处理前面的读线程，然后在处理后面的线程。  
自旋获取共享锁  
这是自旋获取共享锁的时序图：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/87fcca1f32a117c6b8bb9c6101b451aa.png)
    
    
    // 自旋获取共享锁(读锁)
    final int fullTryAcquireShared(Thread current) {
    	// 新建当前线程的重入数量对象
        HoldCounter rh = null;
        // 开始自旋
        for (;;) {
        	// 获取锁状态
            int c = getState();
            // 当前锁是读锁还是写锁
            if (exclusiveCount(c) != 0) { // 获取锁状态的低16位
            	// 如果是写锁，那么判断当前线程是否已经获取了写锁
                if (getExclusiveOwnerThread() != current)
                	// 如果当前线程没有获取写锁，而且锁还是写锁状态时，此时不允许获取读锁
                	// 因为写锁是独占锁
                    return -1;
                // else we hold the exclusive lock; blocking here
                // would cause deadlock.
            // 如果现在锁状态是读锁，那么判断当前线程前面还有没有等待的读取线程(读锁获取是公平锁,不允许插队)
            } else if (readerShouldBlock()) {
                // Make sure we're not acquiring read lock reentrantly
                // 如果当前线程是第一个获取读锁的线程
                if (firstReader == current) {
                	// 当前线程已经获取了读锁了，现在是重入，所以只需要将重入数量++即可
                    // assert firstReaderHoldCount > 0;
                } else {
                	// 当前线程不是第一个获取读锁的线程，那么需要初始化线程重入数量计数器
                	// 如果线程重入计数器为空，那么需要初始化
                    if (rh == null) {
                    	// 获取读锁缓存的线程重入数量计数器
                        rh = cachedHoldCounter;
                        // 如果缓存的计数器为空或者不是当前线程的计数器
                        if (rh == null || rh.tid != getThreadId(current)) {
                        	// 那么调用get方法获取计数器(如果不存在会初始化一个0的计数器(不是空对象))
                            rh = readHolds.get();
                            // 如果线程重入次数是0，那么将计数器从线程变量中移除，只放到缓存变量中
                            if (rh.count == 0)
                            	// 移除线程变量(线程id可能重复)
                                readHolds.remove();
                        }
                    }
                    // 如果线程重入计数器的数量为0，那么直接返回获取读锁失败
                    if (rh.count == 0)
                        return -1;
                    // 首先，当前线程前面还有等待的读取线程才会走到这里
                    // 其次，前面的读取线程会初始化缓存计数器
                }
            }
            // 获取读锁持有的线程数量
            // 如果读锁的持有线程数量达到最大值65535，那么抛出错误
            if (sharedCount(c) == MAX_COUNT)
                throw new Error("Maximum lock count exceeded");
            // cas 增加读锁持有的线程数量
            if (compareAndSetState(c, c + SHARED_UNIT)) {
            	// 如果读锁的线程持有数量为1
                if (sharedCount(c) == 0) {
                	// 那么将当前线程设置为读锁缓存线程
                    firstReader = current;
                    // 将读锁缓存线程重入数量设置为1
                    firstReaderHoldCount = 1;
                // 如果读锁缓存线程和当前线程相等
                } else if (firstReader == current) {
                	// 那么将读锁缓存线程的重入层数++
                    firstReaderHoldCount++;
                } else {
                // 如果当前线程与读锁缓存线程不是同一个线程(表示现在最少有2个线程同时持有读锁)
                	// 如果缓存线程重入数量计数器为空
                    if (rh == null)
                    	// 那么获取缓存数据(第一次自旋)
                        rh = cachedHoldCounter;
                    // 如果缓存线程重入数量计数器为空或者不是当前线程的重入数量计数器
                    if (rh == null || rh.tid != getThreadId(current))
                    	// 初始化线程重入数量计数器
                        rh = readHolds.get();
                    // 如果重入层数为0，表示是第一次获取锁
                    else if (rh.count == 0)
                    	// 将线程重入数量计数器放到线程变量中(在前面会将历史的计数器清空)
                        readHolds.set(rh);
                    // 线程重入计数器数值++
                    rh.count++;
                    // 设置缓存的线程重入计数器是当前线程的计数器
                    cachedHoldCounter = rh; // cache for release
                }
                // 返回自旋获取读锁成功
                return 1;
            }
        }
    }
    

### 3.4 tryReleaseShared

尝试释放共享锁(读锁)  
这是tryReleaseShared的时序图  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b6c52267a3bcf12b03aa26885968f5fe.png)
    
    
    // 尝试释放共享锁(读锁)
    protected final boolean tryReleaseShared(int unused) { // 传入的unused=1
    	// 获取当前线程
        Thread current = Thread.currentThread();
        // 如果当前线程是第一个获取读锁的线程(偏向线程)
        if (firstReader == current) {
            // assert firstReaderHoldCount > 0;
            // 当前线程读锁重入层数为1
            if (firstReaderHoldCount == 1)
            	// 清空偏向线程
            	// 第一个线程获取了一次锁，然后现在释放了，所以将偏向锁清空
                firstReader = null;
            else
            // 如果第一个获取读锁的线程的重入层数不为1，那么每次释放，将重入层数减小1
                firstReaderHoldCount--;
        } else {
        // 当前线程不是第一个获取读锁的线程
        	// 获取读锁重入数量缓存
            HoldCounter rh = cachedHoldCounter;
            // 读锁重入数量缓存为空，或者读锁重入数量缓存不是当前线程的缓存
            if (rh == null || rh.tid != getThreadId(current))
            	// 获取当前线程的读锁重入数量缓存
                rh = readHolds.get();
            // 得到当前线程读锁的重入层数
            int count = rh.count;
            if (count <= 1) { // 如果重入层数小于等于1，那么移除当前线程重入数量线程变量
            	// 当前线程如果重入，那么在重入时，如果线程重入数量的线程变量为空，会进行新建
                readHolds.remove();
                // 如果重入层数小于等于0，抛出异常(重入层数等于0表示当前线程不持有读锁，还去释放，一定报错了)
                if (count <= 0)
                    throw unmatchedUnlockException();
            }
            // 释放完成，需要将重入层数--
            --rh.count;
        }
        // 进入自旋，在自旋中保证修改state成功
        for (;;) {
        	// 获取state，锁状态
            int c = getState();
            // 读锁持有线程数量减1(重入会加，重入释放也需要减)
            int nextc = c - SHARED_UNIT;
            if (compareAndSetState(c, nextc))
                // Releasing the read lock has no effect on readers,
                // but it may allow waiting writers to proceed if
                // both read and write locks are now free.
                // 读锁是否空闲(是否完全释放)
                return nextc == 0;
        }
    }
    

### 3.5 tryWriteLock

尝试获取写锁。  
这是尝试获取写锁的时序图  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/908e836f0b841591c90c8fe01eca3c86.png)
    
    
    // 尝试获取写锁
    final boolean tryWriteLock() {
    	// 获取当前线程
        Thread current = Thread.currentThread();
        // 获取锁状态
        int c = getState();
        // 判断锁是否空闲
        if (c != 0) { // 锁不空闲
        	// 获取锁是否是写锁
            int w = exclusiveCount(c); // 取锁状态的低16位
            // 如果锁状态的低16位为0，表示锁现在是读锁
            // 如果当前线程不是锁持有线程
            if (w == 0 || current != getExclusiveOwnerThread())
            	// 那么直接返回尝试获取写锁失败
                return false;
            // 如果锁持有线程数量达到最大值，那么直接抛出错误
            if (w == MAX_COUNT)
                throw new Error("Maximum lock count exceeded");
        }
        // 锁持有数量加1(写锁低位)
        if (!compareAndSetState(c, c + 1))
        	// 如果cas 设置锁状态失败，那么尝试获取写锁失败
            return false;
        // 否则将当前线程设置为锁持有线程
        setExclusiveOwnerThread(current);
        // 返回尝试获取写锁成功
        return true;
    }
    

### 3.6 tryAcquire

这是tryAcquire的时序图：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b33f19a2282ce1be5b07324bad2b94cf.png)
    
    
    // 尝试获取独占锁
    protected final boolean tryAcquire(int acquires) {
    	// 获取当前线程
        Thread current = Thread.currentThread();
        // 获取锁状态
        int c = getState();
        // 获取锁状态的低16位
        int w = exclusiveCount(c);
        // 判断锁是不是空闲
        if (c != 0) {
            // (Note: if c != 0 and w == 0 then shared count != 0)
            // 锁不空闲
            // 持有锁的线程不是当前线程
            if (w == 0 || current != getExclusiveOwnerThread())
            	// 尝试获取独占锁(写锁)失败
                return false;
            // 锁持有的线程数量是否超过写锁可以持有的最大线程数量65535
            if (w + exclusiveCount(acquires) > MAX_COUNT)
            	// 如果锁持有的线程数量超过最大值，那么直接抛出错误
                throw new Error("Maximum lock count exceeded");
            // Reentrant acquire
            // 设置锁状态
            setState(c + acquires);
            // 返回尝试获取独占锁(写锁)成功
            return true;
        }
        // 如果锁不空闲，那么判断当前线程前面是否还有等待的写线程
        if (writerShouldBlock() || // 如果当前线程之前还有等待的写线程(是否可以插队)
            !compareAndSetState(c, c + acquires)) // 那么尝试使用cas 设置失败
            // 返回尝试获取独占锁(写锁)失败
            return false;
        // 设置锁持有线程是当前线程
        setExclusiveOwnerThread(current);
        // 返回尝试获取独占锁(写锁)成功
        return true;
    }
    

### 3.7 tryRelease

tryRelease是尝试释放独占锁的方法  
这是尝试释放独占锁的方法的时序图  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6f535f850c8ab41a3fd3ff7abb1a954f.png)
    
    
    // 尝试释放独占锁(写锁)
    protected final boolean tryRelease(int releases) {
    	// 判断写锁持有线程是不是当前线程
        if (!isHeldExclusively())
        	// 如果写锁持有的线程不是当前线程，那么当前线程尝试释放自己没有获取的独占锁，肯定不对啦
        	// 直接抛出异常，快速失败
            throw new IllegalMonitorStateException();
        // 获取锁状态释放后的锁状态
        int nextc = getState() - releases;
        // 取得锁状态的低16位，即写锁锁重入的层数
        // 如果写锁锁重入的层数等于0，表示当前线程已经完全释放持有的写锁了
        boolean free = exclusiveCount(nextc) == 0;
        // 如果写锁释放完全
        if (free)
        	// 那么将锁持有的线程清空
            setExclusiveOwnerThread(null);
        // 否则只是将锁重入层数减1
        setState(nextc);
        // 返回锁是否完全释放
        return free;
    }
    

## 4\. 继承于Sync的FairSync

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/94eea3818a52387f75fec7e07e337698.png)  
直接调用AQS的hasQueuedPredecessors方法  
获取当前线程在等待竞争队列中有没有前继节点。  
简单来说，就是获取当前线程前面还有没有等待线程。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c4ae510f0cda0423655eb7ad6e7b16b3.png)  
如果等待竞争队列不为空，那么头结点的后继节点为空或者等待线程不是当前现场，那么就表示当前线程前面还有等待的线程。  
(不会存在head != tail && head.next == null)

## 5\. 继承于Sync的NonfairSync

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dc1259a09755558af30a7bb26e244fc6.png)  
对于非公平锁，写锁直接返回false,读锁则调用apparentlyFirstQueuedIsExclusive方法  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/857695cf201e3bf05effc71bef9d3478.png)  
apparentlyFirstQueuedIsExclusive是判断等待竞争队列中的等待线程是不是共享模式。

## 6\. ReadLock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a5eddcbfbbf7bdb2b115deaf70cc60c.png)  
读锁内部冗余持有ReentrantReadWriteLock的Sync对象实例。  
读锁将Lock全部的方法代理到了ReentrantReadWriteLock内部的Sync的方法中。

## 7\. WriteLock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/310cddc94ab60a44782103aad775553f.png)  
写锁和读锁实现完全相同。

## 8\. HoldCounter

线程重入数量计数器  
两个属性：线程id和重入层数。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b2f20d6a696d726fc6b9441aa0bd2af.png)  
HoldCounter很简单，记录线程重入层数，并且调用getThreadId方法获取当前线程的线程id  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf93cf1240dbedad298e5baabbdd50c2.png)  
使用的是UNSAFE的方法获取线程id.

## 9\. ThreadLocalHoldCounter

线程变量。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1bf581c8358a11735cb9c4ef58fdb335.png)  
线程变量，存储的是线程重入数量计数器。

## 10\. ReentrantReadWriteLock的构造

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d4a91ae663ac3b1dc46a6738f52c7ae1.png)  
因为ReentrantReadWriteLock内部也实现了AQS的Sync类，而且也基于Sync实现了FairSync和NonfairSync，所以，构造方法就是决定，ReentrantReadWriteLock使用何种方式实现锁。

### 10.1 ReentrantReadWriteLock的无参构造

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f751278bb23bd935f4e8d413f246fa16.png)  
无参构造方法直接调用有参构造，传入false

### 10.2 ReentrantReadWriteLock的有参数构造

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d2933003e7190afa527fca0461596e58.png)  
有参构造的参数是一个boolean参数，boolean参数决定使用FairSync还是NonfairSync。

然后传入自己，创建ReadLock和WriteLock.

## 11\. 锁升级与降级

什么是锁升级？  
对于ReentrantReadWriteLock来说，在某一时刻，他要么是读锁，要么是写锁，要么空闲。  
不能即是读锁，又是写锁。  
对于读锁来说，锁是共享锁，可以允许多个线程同时持有。  
而写锁是独占锁，不允许多个线程同时持有。

对于既有读取，又有写入的线程来说，线程需要在读取的时候持有读锁，在写入的时候持有写锁。  
而锁的获取需要和其他线程进行竞争，所以很多时候持有写锁，在独占锁的模式下进行线程内读取和写入。  
这样做是没有问题的，但是这样做对于锁的资源利用很低。  
假设线程内90%的时间都是在读取，只有10%的时间在写入，而且写入在前，那么此时后90%的时间都是使用独占锁在读取数据。而对于其他需要读取的线程来说，此时因为写锁是独占锁，所以，其他读线程也无法读取数据。  
为了提高资源利用率，提出了锁降级。  
锁降级是指：当写锁写入数据完成后，将写锁变为读锁，此时其他读取的线程也能读取了，这样就提高了资源利用率。  
**不过需要注意，只能锁降级，不能锁升级。**

### 11.1 锁降级

锁降级是在线程持有写锁的情况下，获取读锁，获取到读锁后，释放写锁。此时锁就从写锁降级为读锁。  
在持有写锁的时候，获取读锁：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b6a70a0c53ee2d5077f074c2238a2af9.png)  
在同时持有读锁和写锁的时候释放写锁：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d864b5e1e24bf22434738adb64d12203.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/35f31adca68265f46c9385f07e35922f.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0574a45abeee86ad0dd94060018a504.png)

### 11.2 锁升级

锁升级是指，在持有读锁的时候，获取写锁。  
这是不正确的，因为锁降级是独占锁转为共享锁。  
而锁升级是由共享锁转为独占锁。  
锁升级首先就要求其他线程放弃共享锁，这几乎是不可能的。  
因为线程间不应该有依赖关系，线程间应该独立的。  
而且要求其他线程放弃共享锁也无法实现。**所以ReentrantReadWriteLock不支持锁升级。**  
不支持锁升级：  
在持有读锁的时候，获取写锁：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8d46996b6e2b48f4456669eb5cd9e47.png)

**在锁降级中，当前线程是锁持有线程就能获取锁。  
在锁升级中，锁持有线程是空的。**

## 12\. 总结

ReentrantReadWriteLock内部持有两个Lock对象，分别是读锁和写锁。  
ReentrantReadWriteLock的锁状态用高16位表示读锁，低16位表示写锁。锁持有线程最大是65535.  
ReentrantReadWriteLock的读锁是偏向锁，当同时有多个读线程同时持有锁，第一个线程会被记录，而不用加入到线程变量中。这也是偏向锁的体现。  
ReentrantReadWriteLock的读锁同时也是乐观锁，乐观认为大多数情况下只有一个线程获取读锁，所以使用了偏向锁提高性能。  
ReentrantReadWriteLock的读锁使用的是共享锁。  
ReentrantReadWriteLock的写锁使用的是独占锁。  
ReentrantReadWriteLock的写锁可以降级为读锁，不能读锁升级为写锁。  
ReentrantReadWriteLock的锁降级是独占锁转为共享锁。  
ReentrantReadWriteLock内部也有FairSync和NonfairSync实现公平和不公平方式获取锁。  
对于读锁，是否是公平的？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0c337034b8b5341d09802e42515e1755.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/069c03a3bfe1f2ee26e5d0f2e3ae5596.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0b018184ae2f808fb57b04e9cc180a50.png)  
对于公平锁，只要当前线程前面还存在等待读的线程，那么就需要阻塞。  
**换句话说，公平模式下的读锁不允许插队。**  
不公平模式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/89b4efbf60bc2a8c6bc2aed96a1b3d0f.png)  
**只要锁是共享锁，那么就不需要阻塞，也就是可以插队。**

对于写锁：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6f62bfaca0fb06949fa21de8f72d0465.png)  
公平模式：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3282163b98d983794b6cdeb8862cabc9.png)  
**写锁公平模式下不允许插队。**  
不公平模式：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/02a036ba152df601a64d01304905d2e6.png)  
**写锁不公平模式下一直允许插队。**