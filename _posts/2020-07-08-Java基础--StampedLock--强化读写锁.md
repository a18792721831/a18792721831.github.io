---
layout: post
title: "Java基础--StampedLock--强化读写锁"
date: 2020-07-08 19:59:39 +0800
categories: [StampedLock源码, StampedLock原理, StampedLock入门, StampedLock教程, 学会StampedLock]
description: "本文深入解析了Java中的StampedLock，一种基于能力的锁机制，提供了读、写和乐观读三种模式，支持锁升级和降级，适用于低争用场景，可显著提高并发性能。"
keywords: StampedLock源码, StampedLock原理, StampedLock入门, StampedLock教程, 学会StampedLock
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107071195
> - 发布时间：2020-07-08 19:59:39
> - 阅读量：1
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, 多线程
> - 标签：#StampedLock源码, #StampedLock原理, #StampedLock入门, #StampedLock教程, #学会StampedLock

## 摘要

文章浏览阅读1.1k次，点赞2次，收藏2次。本文深入解析了Java中的StampedLock，一种基于能力的锁机制，提供了读、写和乐观读三种模式，支持锁升级和降级，适用于低争用场景，可显著提高并发性能。

---

#### Java基础--StampedLock--强化读写锁

  * 1\. StampedLock 介绍
  *     * 1.1 StampedLock 的 UML
    * 1.2 StamptedLock 的 方法和属性
  * 2\. StampedLock 的构造
  * 3\. StampedLock 的方法
  *     * 3.1 tryReadLock
    * 3.2 tryReadLock(long,TimeUnit)
    * 3.3 readLock
    * 3.4 readLockInterruptibly
    * 3.5 tryWriteLock
    * 3.6 tryWriteLock(long,TimeUnit)
    * 3.7 writeLock
    * 3.8 writeLockInterruptibly
    * 3.9 tryUnlockRead
    * 3.10 unlock
    * 3.11 unlockRead
    * 3.12 unlockWrite
    * 3.13 asReadWriteLock
    * 3.14 asReadLock
    * 3.15 asWriteLock
    * 3.16 getReadLockCount
    * 3.17 isReadLocked
    * 3.18 isWriteLocked
    * 3.19 tryConvertToOptimisticRead
    * 3.20 tryConvertToReadLock
    * 3.21 tryConvertToWriteLock
    * 3.22 tryOptimisticRead
    * 3.23 validate
  * 4\. StampedLock 的属性
  *     * 4.1 StampedLock 的属性
    * 4.2 StampedLock 的私有方法
    *       * 4.2.1 tryIncReaderOverflow
      * 4.2.2 acquireRead
      * 4.2.3 cancelWaiter
      * 4.2.4 release
      * 4.2.5 acquireWrite
      * 4.2.6 tryDecReaderOverflow
      * 4.2.7 unstampedUnlockWrite
      * 4.2.8 unstampedUnlockRead
  * 5\. WNode
  * 6\. ReadWriteLockView
  * 7\. WriteLockView
  * 8\. ReadLockView
  * 9\. 示例
  *     * 9.1 读锁
    * 9.2 写锁
    * 9.3 写锁 => 读锁--锁降级
    * 9.4 读锁 => 写锁--锁升级
    * 9.5 ReadLock
    * 9.6 WriteLock
    * 9.7 ReadWriteLock
  * 10\. 总结

## 1\. StampedLock 介绍

一种基于能力的锁，具有三种模式用于控制读/写访问。 StampedLock的状态由版本和模式组成。 锁定采集方法返回一个表示和控制相对于锁定状态的访问的印记; 这些方法的“尝试”版本可能会返回特殊值为零以表示获取访问失败。 锁定释放和转换方法要求邮票作为参数，如果它们与锁的状态不匹配则失败。 这三种模式是：

  * 写锁。 方法writeLock()可能阻止等待独占访问，返回可以在方法unlockWrite(long)中使用的邮票来释放锁定。 不定时的和定时版本tryWriteLock ，还提供。 当锁保持写入模式时，不能获得读取锁定，并且所有乐观读取验证都将失败。
  * 读锁。 方法readLock()可能阻止等待非独占访问，返回可用于方法unlockRead(long)释放锁的戳记 。 不定时的和定时版本tryReadLock ，还提供。
  * 乐观读锁。 方法tryOptimisticRead()只有当锁当前未保持在写入模式时才返回非零标记。 方法validate(long)返回true，如果在获取给定的邮票时尚未在写入模式中获取锁定。 这种模式可以被认为是一个非常弱的版本的读锁，可以随时由作家打破。 对简单的只读代码段使用乐观模式通常会减少争用并提高吞吐量。 然而，其使用本质上是脆弱的。 乐观阅读部分只能读取字段并将其保存在局部变量中，以供后验证使用。 以乐观模式读取的字段可能会非常不一致，因此只有在熟悉数据表示以检查一致性和/或重复调用方法validate()时，使用情况才适用。 例如，当首次读取对象或数组引用，然后访问其字段，元素或方法之一时，通常需要这样的步骤。

此类还支持有条件地在三种模式下提供转换的方法。 例如，方法tryConvertToWriteLock(long)尝试“升级”模式，如果（1）在读取模式下已经在写入模式（2）中并且没有其他读取器或（3）处于乐观模式并且锁可用，则返回有效写入戳记。 这些方法的形式旨在帮助减少在基于重试的设计中出现的一些代码膨胀。

StampedLocks设计用作线程安全组件开发中的内部实用程序。 他们的使用依赖于他们保护的数据，对象和方法的内部属性的知识。 它们不是可重入的，所以锁定的机构不应该调用其他可能尝试重新获取锁的未知方法（尽管您可以将戳记传递给可以使用或转换它的其他方法）。 读锁定模式的使用依赖于相关的代码段是无副作用的。 未经验证的乐观阅读部分不能调用不知道容忍潜在不一致的方法。 邮票使用有限表示，并且不是加密安全的（即，有效的邮票可能是可猜测的）。 邮票值可以在连续运行一年后（不早于）回收。 不超过此期限使用或验证的邮票可能无法正确验证。 StampedLocks是可序列化的，但总是反序列化为初始解锁状态，因此它们对于远程锁定无用。

StampedLock的调度策略不一致优先于读者，反之亦然。 所有“尝试”方法都是尽力而为，并不一定符合任何调度或公平政策。 用于获取或转换锁定的任何“try”方法的零返回不携带关于锁的状态的任何信息; 随后的调用可能会成功。

### 1.1 StampedLock 的 UML

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e556bf4f3c7fb569a589e1b00e669812.png)  
这是StampedLock的引用关系图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d09ad9e27a8de5996009656ebbecfec2.png)

### 1.2 StamptedLock 的 方法和属性

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a1b3235196cdbca0d8675fe5b0a7cbcc.png)

## 2\. StampedLock 的构造

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8bf343340240e0115796f36ea77e164f.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f7f851c290b073ec3699b4a94bd9389b.png)  
初始化锁状态为ORIGIN `1 0000 0000`

## 3\. StampedLock 的方法

### 3.1 tryReadLock

尝试获取读锁  
这是尝试获取读锁的时序图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3936e1db151377cfa8b88d99dfdb5e34.png)
    
    
    // 尝试获取读锁
    public long tryReadLock() {
    	// 自旋
        for (;;) {
        	// 定义变量
            long s, m, next;
            // s = state // 获取锁状态
            // m 获取读写锁状态(第8位为1表示写锁，为0表示读锁)
            if ((m = (s = state) & ABITS) == WBIT)
            	// 现在是写锁，获取读锁失败 
                return 0L;
            // RFULL = 0111 1110 , state最多纪录126个读锁(溢出需要做溢出处理：类似分数的整数位和分数位)
            else if (m < RFULL) {
            	// RUNIT = 1L
            	// 更新state的值为next
            	// next = state + 1
                if (U.compareAndSwapLong(this, STATE, s, next = s + RUNIT))
                	// 返回next
                    return next;
            }
            // 如果现在持有读锁的线程数量大于等于126，那么需要读锁溢出处理
            else if ((next = tryIncReaderOverflow(s)) != 0L)
            	// 如果溢出处理成功,next = 0111 1111 127
            	// 返回127
                return next;
            // 否则进行下一次自旋
        }
    }
    

### 3.2 tryReadLock(long,TimeUnit)

尝试获取读锁，带有超时时间。  
这是尝试获取读锁，带有超时时间的时序图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/634a24c5405d48f239fc08154246038e.png)  
真心恐怖
    
    
    // 尝试获取读锁，带有超时时间
    public long tryReadLock(long time, TimeUnit unit)
        throws InterruptedException {
        // 定义变量
        long s, m, next, deadline;
        // 将超时时间转换为纳秒
        long nanos = unit.toNanos(time);
        // 如果线程没有被中断
        if (!Thread.interrupted()) {
        	// 检测锁状态不是写锁
            if ((m = (s = state) & ABITS) != WBIT) {
            	// 读锁持有线程数量小于最大值
                if (m < RFULL) {
                	// 将读锁持有线程数量++
                    if (U.compareAndSwapLong(this, STATE, s, next = s + RUNIT))
                    	// 返回新的读锁持有线程数量
                        return next;
                }
                // 如果读锁持有线程数量达到或者大于最大值
                else if ((next = tryIncReaderOverflow(s)) != 0L)// 调用读锁溢出处理
                	// 溢出处理成功
                    return next;
            }
            // 如果锁状态是写锁，那么尝试获取读锁需要等待写锁释放
            // 如果超时时间小于等于0
            if (nanos <= 0L)
            	// 尝试立刻获取读锁，但是当前锁状态是写锁，返回获取失败
                return 0L;
            // 获取截止时间：截止时间=当前时间+超时时间
            if ((deadline = System.nanoTime() + nanos) == 0L)
                deadline = 1L;
            // 调用acquireRead自旋获取读锁(获取读锁需要等待，表示当前锁是写锁)
            // acquoreRead请跳转4.2.2
            if ((next = acquireRead(true, deadline)) != INTERRUPTED)
            	// 返回自旋获取锁结果
                return next;
        }
        // 如果线程被中断，那么抛出中断异常
        throw new InterruptedException();
    }
    

### 3.3 readLock

阻塞获取共享锁，读锁。
    
    
    // 阻塞获取共享锁，读锁
    public long readLock() {
    	// 定义变量
        long s = state, next;  // bypass acquireRead on common uncontended case
        // 线程等待队列的头结点等于尾节点(线程等待队列为空)
        // 锁状态是读锁，且读锁线程数量小于最大读锁线程持有数量
        return ((whead == wtail && (s & ABITS) < RFULL &&
        		// 锁状态++
                 U.compareAndSwapLong(this, STATE, s, next = s + RUNIT)) ?
                 // 返回更新后的锁状态，或者调用acquireRead方法
                next : acquireRead(false, 0L));
    }
    

### 3.4 readLockInterruptibly

获取共享锁，读锁，响应读锁。
    
    
    // 获取共享锁，响应中断
    public long readLockInterruptibly() throws InterruptedException {
    	// 定义变量
        long next;
        // 判断当前线程的中断状态
        if (!Thread.interrupted() &&
        	// 调用acquireRead方法获取共享锁
            (next = acquireRead(true, 0L)) != INTERRUPTED)
            // 返回锁状态(成功、失败)
            return next;
        // 抛出中断异常
        throw new InterruptedException();
    }
    

### 3.5 tryWriteLock

尝试获取写锁。
    
    
    // 尝试获取写锁
    public long tryWriteLock() {
    	// 定义变量
        long s, next;
        // 获取锁状态，只有锁空闲，才能取到写锁
        return ((((s = state) & ABITS) == 0L &&
                 U.compareAndSwapLong(this, STATE, s, next = s + WBIT)) ?
                next : 0L);
    }
    

### 3.6 tryWriteLock(long,TimeUnit)

尝试获取写锁，带有超时时间。
    
    
    // 尝试获取写锁，带有超时时间
    public long tryWriteLock(long time, TimeUnit unit)
        throws InterruptedException {
        // 获取超时时间
        long nanos = unit.toNanos(time);
        // 线程未中断
        if (!Thread.interrupted()) {
        	// 定义 变量
            long next, deadline;
            // 调用tryWriteLock进行尝试获取写锁
            if ((next = tryWriteLock()) != 0L)
            	// 获取写锁成功
                return next;
            // 如果超时时间小于等于0
            if (nanos <= 0L)
            	// 不等待，立刻返回失败
                return 0L;
            // 获取截止时间
            if ((deadline = System.nanoTime() + nanos) == 0L)
            	// 如果截止时间为空，那么设置为1纳秒
                deadline = 1L;
            // 自旋1纳秒获取写锁
            if ((next = acquireWrite(true, deadline)) != INTERRUPTED)
            	// 自旋过程中，线程没有被中断，返回成功
                return next;
        }
        // 抛出中断异常
        throw new InterruptedException();
    }
    

### 3.7 writeLock

阻塞获取写锁。
    
    
    // 阻塞获取写锁
    public long writeLock() {
    	// 定义变量
        long s, next;  // bypass acquireWrite in fully unlocked case only
        // 获取并判断锁状态，如果锁空闲
        return ((((s = state) & ABITS) == 0L &&
        		// 那么将锁设置为写锁
                 U.compareAndSwapLong(this, STATE, s, next = s + WBIT)) ?
                 // 返回锁状态，或者调用acquireWrite获取写锁
                next : acquireWrite(false, 0L));
    }
    

### 3.8 writeLockInterruptibly

阻塞获取写锁，响应中断
    
    
    // 阻塞获取写锁，响应中断
    public long writeLockInterruptibly() throws InterruptedException {
    	// 定义变量
        long next;
        // 如果线程没有被中断
        if (!Thread.interrupted() &&
        	// 那么调用acquireWrite方法获取 写锁
            (next = acquireWrite(true, 0L)) != INTERRUPTED)
            // 返回锁状态
            return next;
        // 如果在阻塞获取锁期间线程被中断，那么抛出中断异常
        throw new InterruptedException();
    }
    

### 3.9 tryUnlockRead

尝试释放读锁
    
    
    // 尝试释放读锁
    public boolean tryUnlockRead() {
    	// 定义变量
        long s, m; WNode h;
        // 获取锁状态，并判断锁是否空闲，以及锁是否是写锁
        while ((m = (s = state) & ABITS) != 0L && m < WBIT) {
        	// 如果读锁的持有线程数量小于读锁最大线程数量
            if (m < RFULL) {
            	// 那么将读锁持有线程的数量--
                if (U.compareAndSwapLong(this, STATE, s, s - RUNIT)) {
                    if (m == RUNIT && (h = whead) != null && h.status != 0)
                    	// 调用release释放线程挤捏点
                        release(h);
                    // 返回释放读锁成功
                    return true;
                }
            }
          	// 否则读锁的线程持有数量大于最大值，需要执行tryDecReaderOverflow进行处理
            else if (tryDecReaderOverflow(s) != 0L)
            	// 尝试释放读锁成功
                return true;
        }
        // 尝试释放读锁失败
        return false;
    }
    

### 3.10 unlock

释放锁(不管是读锁，还是写锁)
    
    
    // 释放锁(不管是读锁，还是写锁)
    public void unlock(long stamp) {
    	// 定义变量，传入的值是锁状态的值
    	// 获取传入锁状态的模式
        long a = stamp & ABITS, m, s; WNode h;
        // 比较现在锁状态是否与传入的锁状态相等
        // 传入的锁状态是否是最新的
        while (((s = state) & SBITS) == (stamp & SBITS)) {
        	// 如果相同，那么获取锁模式
            if ((m = s & ABITS) == 0L)
            	// 如果锁空闲，那么直接结束
                break;
            // 如果锁是写锁
            else if (m == WBIT) {
            	// 如果传入的锁状态和现在的锁的状态不同，直接结束
                if (a != m)
                    break;
                // 如果是写锁，那么将锁状态清空
                // 1000 0000 + 1000 0000 = 1 0000 0000
               	// s 前面被赋值了写锁的反码，所以这里直接为0
               	// ORIGN = WBIT << 1;
               	// 1000 0000 => 1 0000 0000
                state = (s += WBIT) == 0L ? ORIGIN : s; // 保证stamp的低8位为0
                // 如果CLH队列不为空，或者CLH队列中存在有效的节点
                if ((h = whead) != null && h.status != 0)
                	// 那么调用release方法释放线程节点
                    release(h);
                // 直到CLH队列为空，结束
                return;
            }
            // 如果传入的锁空闲或者传入的锁的值超过有效位数
            else if (a == 0L || a >= WBIT)
            	// 直接结束
                break;
            // 如果锁状态是读锁，且读锁持有线程数量小于可记录的最大值
            else if (m < RFULL) {
            	// 那么锁状态--
                if (U.compareAndSwapLong(this, STATE, s, s - RUNIT)) {
                	// 如果读锁持有线程刚好只有一个
                	// 而且CLH队列不为空，CLH队列中存在有效的线程节点
                    if (m == RUNIT && (h = whead) != null && h.status != 0)
                    	// 那么调用release方法释放线程节点
                        release(h);
                    // 直到CLH队列为空，结束
                    return;
                }
            }
            // 如果传入的锁状态是读锁，且读锁持有线程数量大于读锁可记录的最大数量
            // 那么需要进行读锁溢出逆处理
            else if (tryDecReaderOverflow(s) != 0L)
            	// 读锁溢出逆处理成功后结束
                return;
        }
        // 否则抛出锁状态不一致异常
        throw new IllegalMonitorStateException();
    }
    

### 3.11 unlockRead

释放读锁
    
    
    // 释放读锁
    public void unlockRead(long stamp) {
    	// 传入的是锁状态
    	// 定义变量
        long s, m; WNode h;
        // 自旋释放读锁
        for (;;) {
        	// 传入锁状态和现在的锁状态不一致
            if (((s = state) & SBITS) != (stamp & SBITS) ||
            	// 传入锁状态是空闲
            	// 当前锁状态是空闲
            	// 当前锁状态是写锁
                (stamp & ABITS) == 0L || (m = s & ABITS) == 0L || m == WBIT)
                // 抛出锁状态异常
                throw new IllegalMonitorStateException();
            // 如果当前锁状态实时读锁，且读锁持有线程数量小于锁状态可记录的最大值
            if (m < RFULL) {
            	// 那么直接进行释放锁，锁状态--
                if (U.compareAndSwapLong(this, STATE, s, s - RUNIT)) {
                	// 如果读锁持有线程数量是1，且CLH队列不为空，而且CLH队列的头结点是有效节点
                    if (m == RUNIT && (h = whead) != null && h.status != 0)
                    	// 那么调用release释放头结点
                        release(h);
                    // 直到CLH线程为空或者不存在有效节点
                    break;
                }
            }
            // 如果当前锁状态是读锁，且读锁持有线程数量大于锁状态可记录的最大值
            // 需要进行读锁溢出逆处理
            else if (tryDecReaderOverflow(s) != 0L) 
            	// 读锁溢出逆处理成功结束
                break;
        }
    }
    

### 3.12 unlockWrite

释放写锁
    
    
    // 释放写锁
    public void unlockWrite(long stamp) {
    	// 定义变量
        WNode h;
        // 如果传入的锁状态和当前锁状态不同
        // 或者传入的锁状态是读锁或空闲
        // stamp & 1000 0000
        if (state != stamp || (stamp & WBIT) == 0L)
        	// 抛出锁状态异常
            throw new IllegalMonitorStateException();
        // stamp = 1000 0000 => 1000 0000 + 1000 0000 = 1 0000 0000 
        state = (stamp += WBIT) == 0L ? ORIGIN : stamp; // 保证stamp的低8位为0
        // 如果CLH队列不为空，且头结点有效
        if ((h = whead) != null && h.status != 0)
        	// 调用release释放头结点(实际上是将头结点的后继节点的线程唤醒)
            release(h);
    }
    

### 3.13 asReadWriteLock

获取StampedLock的一个读写的ReadWriteLock视图。  
获取读写锁视图
    
    
    // 获取ReadWriteLock
    public ReadWriteLock asReadWriteLock() {
    	// 定义变量
        ReadWriteLockView v;
        // 如果全局变量的readWriteLock不为空，那么就返回全局变量
        // 否则创建新的ReadWriteLockView对象
        return ((v = readWriteLockView) != null ? v :
                (readWriteLockView = new ReadWriteLockView()));
    }
    

### 3.14 asReadLock

获取StamptedLock的一个读取的Lock视图。  
获取读锁视图
    
    
    // 获取ReadLockView
    public Lock asReadLock() {
    	// 定义变量
        ReadLockView v;
        // 判断全局变量readLockView是否为空
        // 如果不为空，直接返回全局变量
        // 否则新建一个ReadLockView对象
        return ((v = readLockView) != null ? v :
                (readLockView = new ReadLockView()));
    }
    

### 3.15 asWriteLock

获取StamptedLock的一个写入的Lock视图。  
获取写锁视图
    
    
    // 获取写锁视图
    public Lock asWriteLock() {
    	// 定义变量
        WriteLockView v;
        // 判断全局变量writeLockView是否为空
        // 如果不为空，直接返回全局变量
        // 否则新建一个WriteLockView对象
        return ((v = writeLockView) != null ? v :
                (writeLockView = new WriteLockView()));
    }
    

### 3.16 getReadLockCount

查询为此锁持有的读取锁的数量。  
获取读锁持有线程数量，实际上是获取锁状态的低7位+读锁溢出数  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2d917131ddcae55c40c1e6329722d56e.png)
    
    
    // 获取读锁持有线程数量
    private int getReadLockCount(long s) { // 传入锁状态
    	// 定义变量
        long readers;
        // 判断是否溢出
        // 获取锁状态的低7位(锁状态记录的读锁持有线程数量)
        if ((readers = s & RBITS) >= RFULL)
        	// 如果溢出了，那么就需要加上溢出线程数量
            readers = RFULL + readerOverflow;
        // 否则直接返回锁状态上记录的读锁持有线程数量
        return (int) readers;
    }
    

### 3.17 isReadLocked

获取是否是读锁(共享锁)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9743348aab46c0ffaadd3a6080d252d0.png)  
获取锁状态的低7位，如果锁状态的低7位的值不为0，表示现在锁是读锁，值是读锁线程持有数量。  
锁空闲返回false

### 3.18 isWriteLocked

获取是否是写锁(独占锁)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c5812eff3ba5af6b9899bebb6d784733.png)  
获取锁状态的第8位，如果锁状态的第8位为1表示现在是写锁，否则不是写锁。  
锁空闲返回false

### 3.19 tryConvertToOptimisticRead

将当前线程持有的锁转换为乐观读。  
换句话说，当前线程持有的锁可能已经使用完了，而且当前线程后续对同步资源的使用，也几乎不会出现竞争。所以释放当前线程持有的锁，乐观认为后续是不会出现同步问题的
    
    
    // 转换为乐观读
    public long tryConvertToOptimisticRead(long stamp) { // 传入锁状态
    	// 定义变量，获取传入锁状态
        long a = stamp & ABITS, m, s, next; WNode h;
        /*
    	>>>>>>>>>>>> 内存屏障
    	包括了loadFence、storeFence、fullFence等方法。
    	这是在Java 8新引入的，用于定义内存屏障，避免代码重排序。
    	loadFence() 表示该方法之前的所有load操作在内存屏障之前完成。
    	storeFence()表示该方法之前的所有store操作在内存屏障之前完成。
    	fullFence()表示该方法之前的所有load、store操作在内存屏障之前完成。
    	抄自https://www.jianshu.com/p/b110b6465004
    	*/
        U.loadFence();
        // 自旋
        for (;;) {
        	// 传入锁和当前锁的其他位相同
        	// 获取锁状态的反码
            if (((s = state) & SBITS) != (stamp & SBITS))
            	// 如果传入锁状态和当前锁状态不同，那么直接结束循环
                break;
            // 当前锁空闲
            if ((m = s & ABITS) == 0L) {
            	// 传入锁不空闲
                if (a != 0L)
                	// 锁不一致(可能前面的位数有区别，反码比较只是比较低8位)
                    break;
                // 直接返回当前空闲的锁
                return s;
            }
            // 如果当前锁现在是写锁
            else if (m == WBIT) {
            	// 传入锁不是写锁
                if (a != m)
                	// 结束循环
                    break;
               	// 传入锁和当前锁都是写锁
               	// 锁降级(写锁 => 读锁4)
                state = next = (s += WBIT) == 0L ? ORIGIN : s;
                // 因为现在锁从独占锁，变成了共享锁，锁以CLH队列中的读线程都可以获取锁了
                // 遍历CLH队列(自旋循环)
                if ((h = whead) != null && h.status != 0)
                	// 释放节点(读锁获取锁)
                    release(h);
                // CLH队列空(等待的有效的读线程节点)
                // 返回锁状态(读锁)
                return next;
            }
            // 如果传入的锁是空闲或者是写锁
            else if (a == 0L || a >= WBIT)
            	// 那么结束循环
                break;
            // 当前锁状态是读锁，且没有溢出
            else if (m < RFULL) {
            	// 释放锁(读锁持有线程数量--)
                if (U.compareAndSwapLong(this, STATE, s, next = s - RUNIT)) {
                	// 如果读锁持有线程数量是1，那么需要释放CLH中读线程节点
                    if (m == RUNIT && (h = whead) != null && h.status != 0)
                    	// 调用release释放读线程节点
                        release(h);
                    // 返回读锁的记录的读锁持有线程数量(锁状态的低7位)
                    return next & SBITS;
                }
            }
            // 如果当前锁状态是读锁，而且溢出了
            else if ((next = tryDecReaderOverflow(s)) != 0L) // 进行读锁溢出逆处理
            	// 返回读锁的记录的读锁持有线程数量(锁状态的低7位)
                return next & SBITS;
        }
        // 返回失败
        return 0L;
    }
    

### 3.20 tryConvertToReadLock

将当前线程持有的锁转换为读锁  
转换为读锁(获取读锁)
    
    
    // 转换为读锁
    public long tryConvertToReadLock(long stamp) { // 传入锁状态
    	// 获取传入锁状态的模式，定义变量
        long a = stamp & ABITS, m, s, next; WNode h;
        // 传入锁和当前锁的其他位相同
        while (((s = state) & SBITS) == (stamp & SBITS)) {
        	// 当前锁空闲
            if ((m = s & ABITS) == 0L) {
            	// 传入锁不空闲
                if (a != 0L)
                	// 结束循环
                    break;
                // 当前锁是读锁，且没有溢出
                else if (m < RFULL) {
                	// 当前读锁持有线程数量++
                    if (U.compareAndSwapLong(this, STATE, s, next = s + RUNIT))
                    	// 返回锁状态
                        return next;
                }
                // 当前锁是读锁，且锁状态++后溢出，进行读锁持有线程数量溢出处理
                else if ((next = tryIncReaderOverflow(s)) != 0L)
                	// 返回锁状态
                    return next;
            }
            // 如果当前锁状态是写锁
            else if (m == WBIT) {
            	// 传入锁状态和当前锁状态不同
                if (a != m)
                	// 直接结束
                    break;
                // 1000 0001 + s(s=1000 0000)
                // (写锁) => (读锁)
                // 锁降级
                state = next = s + (WBIT + RUNIT);
                // 如果CLH队列中存在有效的等待的读线程节点，那么此时锁是从独占锁变为了共享锁，所以读线程都可以获取到锁了
                if ((h = whead) != null && h.status != 0)
                    // 调用release释放读线程节点
                    release(h);
                // 返回锁状态
                return next;
            }
            // 如果传入锁不空闲，且当前锁是读锁
            // 读锁 => 读锁
            else if (a != 0L && a < WBIT)
            	// 返回传入的锁状态(传入锁状态和当前锁状态相同)
                return stamp;
            else
            // 否则结束循环
                break;
        }
        // 返回失败
        return 0L;
    }
    

### 3.21 tryConvertToWriteLock

将当前线程持有的锁转换为写锁  
转换为写锁(获取写锁)
    
    
    // 转换为写锁(获取写锁)
    public long tryConvertToWriteLock(long stamp) { // 传入锁
    	// 获取传入锁的模式，定义变量
        long a = stamp & ABITS, m, s, next;
        // 传入锁和当前锁的其他位相同
        while (((s = state) & SBITS) == (stamp & SBITS)) {
        	// 当前锁空闲
            if ((m = s & ABITS) == 0L) {
            	// 传入锁不空闲
                if (a != 0L)
                	// 当前锁和传入锁状态不同，直接结束循环
                    break;
                // 否则直接获取锁(当前锁是空闲的，传入锁也是空闲的)
                // 直接获取写锁
                if (U.compareAndSwapLong(this, STATE, s, next = s + WBIT))
                	// 返回锁状态
                    return next;
            }
            // 当前锁是写锁
            else if (m == WBIT) {
            	// 传入锁不是写锁
                if (a != m)
                	// 期间有其他线程获取了独占锁，当前线程就不能获取了
                	// 直接结束循环
                    break;
                // 传入锁状态是写锁，当前锁状态是写锁，返回锁状态
                // 当前线程已经获取到了写锁，此时尝试重新获取
                return stamp;
            }
            // 当前锁是读锁，且只有一个线程在读，传入锁不空闲
            else if (m == RUNIT && a != 0L) {
            	// 当前线程持有的锁是读锁，且读锁只有当前线程在持有，那么，升级为写锁
            	// 读锁 => 写锁 (读锁当前线程已经持有)
            	// 锁升级
                if (U.compareAndSwapLong(this, STATE, s,
                						// 0000 0001 - 1 + 1000 0000 = 1000 0000
                                         next = s - RUNIT + WBIT))
                    // 返回锁状态
                    return next;
            }
            else
            	// 其他情况，结束循环
                break;
        }
        // 返回失败
        return 0L;
    }
    

### 3.22 tryOptimisticRead

获取乐观读锁，返回乐观读锁的锁状态的反码(凭证)  
后续其他的操作都会需要用到这个凭证。  
这个方法按照我的理解就是：  
线程在某个时间点，记录下锁状态，得到锁状态的凭证。  
后面根据记录的锁状态，尝试获取写锁，读锁，释放锁。  
在后续进行锁的获取、释放，都需要验证凭证，看看期间锁有没有被其他线程修改。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/85c7ca219d8060e991534f5b35194f31.png)  
写锁返回0

### 3.23 validate

验证凭证是否有效。  
使用3.22的方法可以获取凭证，在使用凭证之前应该使用validate验证凭证，凭证有效在调用其他的方法进行获取或者释放锁。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/41df4440b45ba5e2b8cb4d6c6dbd7a99.png)  
判断从调用获取凭证到调用验证方法期间，锁状态有没有发生变化。

## 4\. StampedLock 的属性

### 4.1 StampedLock 的属性
    
    
    // 获取JVM可用的CPU核心数
    private static final int NCPU = Runtime.getRuntime().availableProcessors();
    // 获取锁的自旋次数，单核0次，多核 100000 = 2 ^ 6 = 64 次
    private static final int SPINS = (NCPU > 1) ? 1 << 6 : 0;
    // 等待竞争队列的头结点，自旋次数，超过自旋次数，继续阻塞
    private static final int HEAD_SPINS = (NCPU > 1) ? 1 << 10 : 0;
    // 重新阻塞的自旋次数
    private static final int MAX_HEAD_SPINS = (NCPU > 1) ? 1 << 16 : 0;
    // 自旋等待溢出的次数 	0000 0111
    private static final int OVERFLOW_YIELD_RATE = 7; // must be power 2 - 1
    // 读锁的位数 7 (state 的低7位表示读锁，第8位表示写锁)
    private static final int LG_READERS = 7;
    // 一个单位的读锁 	0000 0001
    private static final long RUNIT = 1L;
    // 写锁标志位		1000 0000
    private static final long WBIT  = 1L << LG_READERS;
    // 读锁状态表示		0111 1111
    // 将state与读锁状态进行按位与，可以得到state的低7位
    private static final long RBITS = WBIT - 1L;
    // 读锁的最大线程数量	0111 1110
    private static final long RFULL = RBITS - 1L;
    // 获取读写状态		1111 1111
    // 将state与获取读写状态按位与，可以判断得出是读锁，还是写锁
    private static final long ABITS = RBITS | WBIT;
    // 1111.... 		1000 0000
    private static final long SBITS = ~RBITS; // note overlap with ABITS
    // 初始的state	  1 0000 0000 
    private static final long ORIGIN = WBIT << 1;
    // 中断标志 1
    private static final long INTERRUPTED = 1L;
    // 等待状态 -1
    private static final int WAITING   = -1;
    // 取消状态 1
    private static final int CANCELLED =  1;
    // 读锁模式 0
    private static final int RMODE = 0;
    // 写锁模式 1
    private static final int WMODE = 1;
    

### 4.2 StampedLock 的私有方法

#### 4.2.1 tryIncReaderOverflow
    
    
    // 读锁溢出处理
    private long tryIncReaderOverflow(long s) {
        // assert (s & ABITS) >= RFULL;
        // 传入的s是调用者调用时的state >= 126
        // 如果是读锁，且现在持有读锁的数量126
        if ((s & ABITS) == RFULL) { // 0111 1110
        	// 假设s = 127  0111 1111
        	// s|RBITs = 0111 1111 | 0111 1111 全满
            if (U.compareAndSwapLong(this, STATE, s, s | RBITS)) {
            	// 溢出线程++
                ++readerOverflow;
                // 将state 设置为 0111 1111 全满
                state = s;
                // 返回 0111 1111 = 127
                return s;
            }
        }
        // state > 126, 0111 1111 = 127
        else if ((LockSupport.nextSecondarySeed() & 
                  OVERFLOW_YIELD_RATE) == 0) // 自旋等待溢出
            Thread.yield(); // 让出CPU时间
        return 0L; // 返回失败，溢出
    }
    

#### 4.2.2 acquireRead

自旋获取读锁  
这是自旋获取读锁的时序图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c595ad4f434b968f289df434299e0a1b.png)  
好恐怖啊。
    
    
    // 自旋获取读锁
    private long acquireRead(boolean interruptible, long deadline) {
    	// 定义node,p
        WNode node = null, p;
        // 开始自旋,初始化自旋次数为-1(不自旋)
        // 自旋初始化线程等待队列
        for (int spins = -1;;) {
        	// 定义等待竞争队列(线程队列)的头节点
            WNode h;
            // 获取线程队列的队列头和队列尾，并且线程队列不为空
            if ((h = whead) == (p = wtail)) {
            	// 锁状态是读锁，直接自旋
                for (long m, s, ns;;) {
                	// 获取当前锁状态，如果当前锁是读锁，且读锁持有线程数量小于最大值
                    if ((m = (s = state) & ABITS) < RFULL ?
                    	// 将锁状态++
                        U.compareAndSwapLong(this, STATE, s, ns = s + RUNIT) : 
                        // 如果锁是读锁，但是现在读锁的线程持有数量大于最大值
                        // 那么就进行读锁数量溢出处理
                        (m < WBIT && (ns = tryIncReaderOverflow(s)) != 0L))
                        // 返回获取读锁成功
                        return ns;
                    // 如果当前锁是写锁
                    else if (m >= WBIT) {
                    	// 如果自旋次数大于0
                        if (spins > 0) {
                        	// 获取多线程的随机数
                            if (LockSupport.nextSecondarySeed() >= 0) 
                            	// 自旋次数--
                                --spins;
                        }
                        // 自旋次数小于等于0，不自旋
                        else {
                        	// 自旋次数已经进行计算(初始化是-1)
                        	// 判断是否是单核
                            if (spins == 0) {
                            	// 获取线程等待队列的头节点和尾节点
                                WNode nh = whead, np = wtail;
                                // 头结点和尾节点没有被其他线程改变
                                // 线程等待队列不为空(还存在等待的线程)
                                if ((nh == h && np == p) || (h = nh) != (p = np))
                                	// 跳出内层循环
                                    break;
                            }
                            // 自旋次数计算，如果是单核，自旋次数为0，如果是多核，自旋次数64
                            spins = SPINS;
                        }
                    }
                }
            }
            // 如果尾节点为空，线程等待队列为空
            // 初始状态，尾节点和头结点相同，尾节点为空，表示线程等待队列为空
            if (p == null) { // initialize queue
            	// 初始化线程等待队列，头结点不为空
                WNode hd = new WNode(WMODE, null);
                // 设置头结点
                if (U.compareAndSwapObject(this, WHEAD, null, hd))
                	// 设置尾节点
                    wtail = hd;
            }
            // 如果node为空(当前线程的等待线程节点还未创建)
            else if (node == null)
            	// 创建当前线程的等待线程节点，其模式为读
                node = new WNode(RMODE, p);
            // 如果头结点和尾节点相同(线程等待队列中只剩下一个节点)
            // 线程节点是写模式
            else if (h == p || p.mode != RMODE) {
            	// 新创建的线程节点的前继是等待线程队队列的尾节点(可能其他线程已经加入了，此时尾节点已经被修改
                if (node.prev != p)
                	// 重新设置新创建的线程节点的前继是线程等待队的尾节点
                    node.prev = p;
                // 没有其他线程修改线程等待队列的尾节点，那么设置线程等待队列的尾节点是新创建的节点
                else if (U.compareAndSwapObject(this, WTAIL, p, node)) {
                	// 新创建的线程节点加入到线程等待队列成功
                	// 更新原线程等待队列的尾节点的后继为新创建的线程节点
                    p.next = node;
                    // 退出新增线程节点的自旋循环
                    break;
                }
            }
            // 线程节点是读模式
            else if (!U.compareAndSwapObject(p, WCOWAIT, // 关联节点
                                             node.cowait = p.cowait, node))
                // 关联失败，关联节点为空
                node.cowait = null;
            // 读线程节点关联成功
            else {
            	// 循环将等待线程队列中的读节点的线程唤醒
                for (;;) {
                	// 定义线程等待队列变量
                    WNode pp, c; Thread w;
                    // 获取头节点和头结点的关联节点不为空
                    if ((h = whead) != null && (c = h.cowait) != null &&
                    	// 设置头节点的关联节点为关联节点的关联节点
                    	// 头结点不存储线程(头结点为写节点)
                        U.compareAndSwapObject(h, WCOWAIT, c, c.cowait) &&
                        // 获取头结点的后继节点的线程，且不为空
                        (w = c.thread) != null) // help release
                        // 唤醒头结点的后继的线程
                        U.unpark(w);
                    // pp 是等待线程队列的尾节点的前继
                    // h==pp表示等待线程队列中共有2个线程节点，且尾节点是读节点
                    // h == p 表示等待线程队列中只有一个节点
                    // pp == null 表示等待线程队列中只有一个节点(头结点没有前继)
                    if (h == (pp = p.prev) || h == p || pp == null) {
                    	// 定义变量
                        long m, s, ns;
                        // 自旋获取读锁
                        do {
                        	// 获取锁状态，获取锁模式
                        	// 如果锁是读锁，且读锁持有线程数量小于最大值
                            if ((m = (s = state) & ABITS) < RFULL ?
                            	// 读锁的持有线程数量++
                                U.compareAndSwapLong(this, STATE, s,
                                                     ns = s + RUNIT) :
                                // 读锁的持有数量等于最大值
                                (m < WBIT &&
                                	// 读锁持有线程数量溢出处理
                                 (ns = tryIncReaderOverflow(s)) != 0L))
                                // 返回结果
                                return ns;
                        } while (m < WBIT);
                    }
                    // 如果线程等待队列的头结点和尾节点的前继没有被其他线程修改
                    if (whead == h && p.prev == pp) {
                    	// 定义变量time
                        long time;
                        // 线程等待队列的尾节点的前继为空(线程等待队列中只有一个节点)
                        // 头结点和尾节点相同(线程等待队列中只有一个节点)
                        // 线程等待队列的尾节点的状态是取消
                        if (pp == null || h == p || p.status > 0) {
                        	// 清空node
                            node = null; // throw away
                           	// 结束度节点唤醒循环
                            break;
                        }
                        // 如果截止时间等于0
                        if (deadline == 0L)
                        	// 设置time为0
                            time = 0L;
                        // 如果截止时间减去当前时间小于等0，表示时间到了
                        else if ((time = deadline - System.nanoTime()) <= 0L)
                        	// 调用cancelWaiter，并且返回cancelWaiter的结果
                            return cancelWaiter(node, p, false);
                        // 获取当前线程
                        Thread wt = Thread.currentThread();
                        // 设置当前线程的阻塞调用者是自己
                        U.putObject(wt, PARKBLOCKER, this);
                        // 设置线程节点的线程是当前线程
                        node.thread = wt;
                        // 头结点不等于尾节点的前继(线程等待队列的节点数>2)
                        // 锁状态是写锁
                        if ((h != pp || (state & ABITS) == WBIT) &&
                        	// 头节点和尾节点的前继没有被其他线程改变
                            whead == h && p.prev == pp)
                            // 阻塞当前线程指定的时间
                            U.park(false, time);
                        // 设置线程节点的线程是空
                        node.thread = null;
                        // 取消当前线程的阻塞调用者
                        U.putObject(wt, PARKBLOCKER, null);
                        // 如果是支持中断的，而且中断标志是中断
                        if (interruptible && Thread.interrupted())
                        	// 调用cancelWaiter，并且返回cancelWaiter的结果
                            return cancelWaiter(node, p, true);
                    }
                }
            }
        }
    	// 
        for (int spins = -1;;) {
        	// 定义变量
            WNode h, np, pp; int ps;
            // 获取线程等待队列头结点，如果线程等待队列的头结点和尾节点相同(线程等待队列只有一个节点)
            if ((h = whead) == p) {
            	// 如果自旋次数还未计算
                if (spins < 0)
                	// 自旋次数初始化，如果是多核的，那么，自旋次数是1024，如果是单核的，那么自旋次数是0
                    spins = HEAD_SPINS; // HEAD_SPINS : 1 << 16
                // 如果自旋次数小于最大自旋次数，那么将自旋次数扩大1倍
                // MAX_HEAD_SPINS:单核0;多核1<<16
                else if (spins < MAX_HEAD_SPINS)
                	// 2048
                    spins <<= 1; 
                // 自旋
                for (int k = spins;;) { // spin at head
                	// 定义变量
                    long m, s, ns;
                    // 获取锁状态和锁模式
                    if ((m = (s = state) & ABITS) < RFULL ? // 如果是读锁，且读锁持有线程数量没有达到最大值
                    	// 读锁持有线程数量++
                        U.compareAndSwapLong(this, STATE, s, ns = s + RUNIT) :
                        // 如果是读锁，那么进行读锁持有线程数量溢出处理
                        (m < WBIT && (ns = tryIncReaderOverflow(s)) != 0L)) {
                        // 定义变量
                        WNode c; Thread w;
                        // 线程等待队列的头结点设置为新增的线程节点
                        // node需要保证是头结点才行
                        whead = node;
                        // 新增的线程节点的前继设置为空
                        node.prev = null;
                        // 获取线程节点的关联节点
                        while ((c = node.cowait) != null) { // 关联节点不为空
                        	// 设置node的关联节点为：关联节点的节点
                            if (U.compareAndSwapObject(node, WCOWAIT,
                                                       c, c.cowait) &&
                                // 关联节点的线程不为空
                                (w = c.thread) != null)  
                                // 唤醒关联节点的线程
                                U.unpark(w);
                        }
                        // 返回锁状态
                        return ns;
                    }
                    // 如果是写锁，那么进行自旋(自旋次数k需要用完)
                    else if (m >= WBIT &&
                             LockSupport.nextSecondarySeed() >= 0 && --k <= 0)
                        break;
                }
            }
            // 如果头结点不为空
            else if (h != null) {
            	// 定义变量
                WNode c; Thread w;
                // 获取线程等待队列的头结点的关联节点
                while ((c = h.cowait) != null) { // 头结点的关联节点不为空
                	// 设置头节点的关联节点为关联节点的关联节点
                    if (U.compareAndSwapObject(h, WCOWAIT, c, c.cowait) &&
                        (w = c.thread) != null) // 关联节点的线程不为空
                        // 唤醒关联节点的线程
                        U.unpark(w);
                }
            }
            // 如果线程等待队列的头节点没有被其他线程修改
            if (whead == h) {
            	// 获取node的前继，如果node的前继不等于线程等待队列的尾节点
                if ((np = node.prev) != p) {
                	// 如果node的前继不为空
                    if (np != null)
                    	// 那么p偏移到node的前继，同时node的前继的后继等于node
                        (p = np).next = node;   // stale
                }
                // 获取线程节点的等待状态，如果等待状态为0，表示初始化
                else if ((ps = p.status) == 0)
                	// 设置初始化的线程节点的状态为等待-1
                    U.compareAndSwapInt(p, WSTATUS, 0, WAITING);
                // 如果线程节点的等待状态是取消
                else if (ps == CANCELLED) {
                	// 获取前继节点，并且前继不为空
                    if ((pp = p.prev) != null) {
                    	// node的前继为前继的前继
                        node.prev = pp;
                        // node的前继的前继的后继是node
                        pp.next = node;
                    }
                }
                // 如果线程节点的等待状态是空闲
                else {
                	// 定义变量
                    long time;
                    // 如果截止时间为0
                    if (deadline == 0L)
                    	// 设置超时时间为0
                        time = 0L;
                    // 获取超时时间：截止时间减去当前系统时间，如果超时时间小于0
                    else if ((time = deadline - System.nanoTime()) <= 0L)
                    	// 调用cancelWaiter，并且返回cancelWaiter的结果
                        return cancelWaiter(node, node, false);
                    // 获取当前线程
                    Thread wt = Thread.currentThread();
                    // 设置当前线程阻塞调用者是自己
                    U.putObject(wt, PARKBLOCKER, this);
                    // 设置线程节点的线程是当前线程
                    node.thread = wt;
                    if (p.status < 0 && // 线程节点的等待状态是等待
                    	// 当前线程节点不是头结点
                    	// 锁状态是写锁
                        (p != h || (state & ABITS) == WBIT) && 
                        // 头结点和node的前继节点没有被其他线程修改
                        whead == h && node.prev == p)
                        // 阻塞当前线程指定时间
                        U.park(false, time);
                    // 设置线程节点的线程为空
                    node.thread = null;
                    // 清空线程阻塞调用者
                    U.putObject(wt, PARKBLOCKER, null);
                   	// 如果是支持中断的，而且中断标志是中断
                    if (interruptible && Thread.interrupted())
                    	// 调用cancelWaiter，并且返回cancelWaiter的结果
                        return cancelWaiter(node, node, true);
                }
            }
        }
    }
    

本来这里打算画下流程图，但是，尝试着画了一个分支，发现实在是没法画下去了。

#### 4.2.3 cancelWaiter

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/68031e743c844c3856769d22786d76ef.png)
    
    
    // 
    private long cancelWaiter(WNode node, WNode group, boolean interrupted) {
    	// 如果node 和group 都不为空
       if (node != null && group != null) {
       	   // 定义变量
           Thread w;
           // 设置node节点的状态是取消
           node.status = CANCELLED;
           // unsplice cancelled nodes from group
           // 从node开始循环，循环的是node的关联节点
           // 这里有一个思想：读线程的线程节点链表看做一个写线程节点
           // 这一批读线程和一个写线程在调度过程中是一样看待的。
           // 只是读线程多了链表遍历的操作 
           for (WNode p = group, q; (q = p.cowait) != null;) {
           		// 读线程链表中存在取消状态的读线程节点	
               if (q.status == CANCELLED) {
               		// 将取消的线程节点从读线程链表中移除
                   U.compareAndSwapObject(p, WCOWAIT, q, q.cowait);
                   // 从头开始，去除的是当前线程节点p
                   p = group; // restart
               }
               else
               		// 偏移
                   p = q;
           } 
           // 如果传入的两个节点是相等的，那么就不用循环遍历线程等待队列了
           if (group == node) {
           		// 读线程关联链表
               for (WNode r = group.cowait; r != null; r = r.cowait) {
               		// 线程节点中有线程
                   if ((w = r.thread) != null)
                   		// 唤醒线程
                       U.unpark(w);       // wake up uncancelled co-waiters
               }
               // 遍历线程等待队列，从node往前遍历
               for (WNode pred = node.prev; pred != null; ) { // unsplice
               		// 定义变量
                   WNode succ, pp;        // find valid successor
                   while ((succ = node.next) == null || // node 的下一个线程节点为空
                          succ.status == CANCELLED) { // 或者当前循环的线程节点是取消节点
                       WNode q = null;    // find successor the slow way
                       // 遍历线程等待队列，从后往前
                       // 这个遍历的目的是找到线程等待队列中最前面的取消的节点
                       for (WNode t = wtail; t != null && t != node; t = t.prev)
                       		// 如果线程节点不是取消的，就跳过
                           if (t.status != CANCELLED)
                               q = t;     // don't link if succ cancelled
                       if (succ == q ||   // ensure accurate successor
                       		// 如果循环的线程节点就是线程等待队列中最前面的，那么就交换
                           U.compareAndSwapObject(node, WNEXT,
                                                  succ, succ = q)) {
                           // 如果node的下一个线程节点是空的，而且node就是最后一个线程节点
                           if (succ == null && node == wtail)
                           		// 那么将当前线程节点的关联节点设置为当前节点的前缀
                               U.compareAndSwapObject(this, WTAIL, node, pred);
                           // 跳出循环
                           break;
                       }
                   }
                   // 如果当前循环的线程节点就是node节点
                   if (pred.next == node) // unsplice pred link
                   		// 那么将循环线程节点与有效节点关联
                       U.compareAndSwapObject(pred, WNEXT, node, succ);
                   // 如果有效节点不为空，且节点的线程不为空
                   if (succ != null && (w = succ.thread) != null) {
                   		// 将有效节点的线程清空
                       succ.thread = null;
                       // 唤醒有效节点的线程
                       U.unpark(w);       // wake up succ to observe new pred
                   }
                   / 如果当前循环的线程节点的状态不是取消状态而且当前循环的线程节点的前继节点为空
                   if (pred.status != CANCELLED || (pp = pred.prev) == null)
                   		// 停止循环
                       break;
                   // 偏移线程节点
                   node.prev = pp;        // repeat if new pred wrong/cancelled
                   // 将当前循环节点的后继节点循环交换
                   U.compareAndSwapObject(pp, WNEXT, pred, succ);
                   // 偏移线程节点
                   pred = pp;
               }
           }
       }
       // 定义变量
       WNode h; // Possibly release first waiter
       // 头结点不为空
       while ((h = whead) != null) {
       		// 定义变量
           long s; WNode q; // similar to release() but check eligibility
           // 获取头结点的后继
           // 后继节点的状态是取消状态
           if ((q = h.next) == null || q.status == CANCELLED) {
           		// 从尾节点开始遍历
               for (WNode t = wtail; t != null && t != h; t = t.prev)
               		// 找到线程等待队列中最前面的等待的线程节点
                   if (t.status <= 0)
                       q = t;
           }
           // 如果当前循环节点是头结点
           if (h == whead) {
           		// 如果头节点的后继节点不为空，且头结点的状态是空闲状态
               if (q != null && h.status == 0 &&
               		// 锁状态不是写锁
                   ((s = state) & ABITS) != WBIT && // waiter is eligible
                   // 锁空闲或者锁是读锁
                   (s == 0L || q.mode == RMODE))
                   // 调用release释放
                   release(h);
               // 跳出循环
               // 和AQS一模一样，从头结点开始处理，头结点不存储线程
               break;
           }
       }
       // 返回线程的中断状态，如果是不响应中断的，那么就返回失败
       return (interrupted || Thread.interrupted()) ? INTERRUPTED : 0L;
    }
    

#### 4.2.4 release
    
    
    // 释放线程节点持有的锁
    private void release(WNode h) {
    	// 线程节点不为空
        if (h != null) {
        	// 定义变量
            WNode q; Thread w;
            // 将当前循环节点的状态从等待设置为空闲
            U.compareAndSwapInt(h, WSTATUS, WAITING, 0);
            // 如果当前循环节点的后继节点为空
            // 或者当前循环节点的状态是取消的
            if ((q = h.next) == null || q.status == CANCELLED) {
            	// 那么从线程等待队列的尾节点开始循环
                for (WNode t = wtail; t != null && t != h; t = t.prev)
                	// 找到线程等待队列的最前的等待节点
                    if (t.status <= 0)
                        q = t;
            }
            // 如果找到的等待的线程节点不为空，而且线程节点的线程不为空
            if (q != null && (w = q.thread) != null)
            	// 那么唤醒线程
                U.unpark(w);
        }
    }
    

#### 4.2.5 acquireWrite

自旋获取写锁。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c68335f055fa8bd5f6de4e38785913b8.png)
    
    
    // 自旋获取写锁
    private long acquireWrite(boolean interruptible, long deadline) {
    	// 定义变量
        WNode node = null, p;
        // 开始自旋
        for (int spins = -1;;) { // spin while enqueuing
        	// 定义变量
            long m, s, ns;
            // 判断锁状态是否空闲
            if ((m = (s = state) & ABITS) == 0L) {
            	// 锁状态空闲，直接获得写锁
                if (U.compareAndSwapLong(this, STATE, s, ns = s + WBIT))
                	// 返回锁状态
                    return ns;
            }
            // 锁状态不空闲，自旋次数未初始化
            else if (spins < 0)
            	// 初始化自旋次数
            	// 如果是写锁，而且线程等待队列为空
            	// 如果是多核的，那么自旋次数是1 << 6 = 64,否则自旋次数是0
                spins = (m == WBIT && wtail == whead) ? SPINS : 0;
            // 锁状态不空闲，自旋次数已经初始化了
            else if (spins > 0) {
            	// 获取多线程下的随机数
                if (LockSupport.nextSecondarySeed() >= 0)
                	// 自旋次数--
                    --spins;
            }
         	// 如果线程等待队列为空
            else if ((p = wtail) == null) { // initialize queue
            	// 那么进行初始化线程等待队列，将线程等待队列的头节点设置为空的线程节点
                WNode hd = new WNode(WMODE, null);
                if (U.compareAndSwapObject(this, WHEAD, null, hd))
                	// 将线程等待队列的尾节点设置为初始化的头节点
                    wtail = hd;
            }
            // 如果node为空，那么进行初始化node
            else if (node == null)
            	// 新创建node后，放在p后面(将新建的线程节点放入线程等待列表中)
                node = new WNode(WMODE, p);
            // 如果node的前继不是p，表明有其他线程在操作线程等待队列中的节点
            else if (node.prev != p)
            	// 将node的前继节点设置为p
                node.prev = p;
            // 将线程等待队列的尾节点设置为node
            else if (U.compareAndSwapObject(this, WTAIL, p, node)) {
            	// 将p的后继设置为node
                p.next = node;
                // 结束循环(线程节点加入线程等待队列完成)
                break;
            }
        }
    	// 第二个循环，前面的循环只是为了将当前线程包装成线程节点，然后放到CLH队列中
    	// 这个循环才是真正的自旋获取锁
    	// 首先定义变量，自旋次数为负数
        for (int spins = -1;;) {
        	// 定义变量
            WNode h, np, pp; int ps;
            // 获取头结点，如果自旋时的头结点和增加线程节点时的头节点是一个节点，那么就表示没有线程修改CLH队列
            if ((h = whead) == p) {
            	// 如果自旋次数还未计算(-1)
                if (spins < 0)
                	// 设置自旋次数为头结点自旋次数(多核:1024;单核:0)
                    spins = HEAD_SPINS;
                // 如果自旋次数小于最大头结点自旋次数(多核:1<<16;单核:0)
                else if (spins < MAX_HEAD_SPINS)
                	// 那么将自旋次数左移一位(扩大2倍)
                    spins <<= 1;
                // 根据前面计算出来的自旋次数进行自旋
                for (int k = spins;;) { // spin at head
                	// 定义变量
                    long s, ns;
                    // 获取锁状态，并且锁空闲
                    if (((s = state) & ABITS) == 0L) {
                    	// 设置锁状态是写锁
                        if (U.compareAndSwapLong(this, STATE, s,
                                                 ns = s + WBIT)) {
                           	// 将头结点设置为node(相当于是CLH队列的节点进行处理)
                            whead = node;
                            // 设置头结点的前继节点为空(头结点没有前继)
                            node.prev = null;
                            // 返回锁状态
                            return ns;
                        }
                    }
                    // 锁不空闲，进行自旋
                    else if (LockSupport.nextSecondarySeed() >= 0 &&
                             --k <= 0) // 自旋次数--
                        // 结束循环(自旋次数用完)
                        break;
                }
            }
            // 如果头结点不为空，表示在当前线程节点前面，CLH中还有等待的线程节点
            else if (h != null) { // help release stale waiters
            	// 定义变量
                WNode c; Thread w;
               	// 线程节点关联的节点不为空
                while ((c = h.cowait) != null) {
                	// 将线程节点的关联节点设置为关联节点的关联节点
                    if (U.compareAndSwapObject(h, WCOWAIT, c, c.cowait) &&
                    	// 线程节点不为空
                        (w = c.thread) != null)
                        // 唤醒线程
                        U.unpark(w);
                }
            }
            // 如果刚进入循环到这里，没有其他线程修改CLH队列
            if (whead == h) {
            	// 获取node的前继节点，如果前继节点和头节点不相等
            	// 也就是说node的前继节点不是头结点
                if ((np = node.prev) != p) {
                	// 而且node的前继不是空
                    if (np != null)
                    	// 那么设置头结点的后继节点是node
                        (p = np).next = node;   // stale
                }
                // 如果线程节点的等待状态是初始化的
                else if ((ps = p.status) == 0)
                	// 那么将线程节点的等待状态设置为等待
                    U.compareAndSwapInt(p, WSTATUS, 0, WAITING);
                // 如果线程节点的等待状态是取消的
                else if (ps == CANCELLED) {
                	// 那么判断这个取消的线程节点的前继是否为空
                    if ((pp = p.prev) != null) {
                    	// 取消节点的前继 不为空，设置node的前继是取消节点
                        node.prev = pp;
                        // 取消节点的前继节点的后继节点是node
                        pp.next = node;
                    }
                }
                // 其他情况
                else {
                	// 定义变量
                    long time; // 0 argument to park means no timeout
                    // 超时时间等于0
                    if (deadline == 0L)
                    	// 设置截止时间为0
                        time = 0L;
                    // 否则计算截止时间
                    else if ((time = deadline - System.nanoTime()) <= 0L)
                    	// 调用cancelWaiter方法，并返回锁状态
                        return cancelWaiter(node, node, false);
                    // 获取当前线程
                    Thread wt = Thread.currentThread();
                    // 设置线程阻塞调用者是当前线程
                    U.putObject(wt, PARKBLOCKER, this);
                    // 设置node线程节点的线程是当前线程 
                    node.thread = wt;
                    // 如果ndoe的前继节点的状态是取消状态
                    // node的前继节点不是头结点
                    // 锁状态不空闲
                    // 头结点和h相同(CLH队列没有被其他线程修改)
                    // node的前继节点是p
                    if (p.status < 0 && (p != h || (state & ABITS) != 0L) &&
                        whead == h && node.prev == p)
                        // 阻塞当前线程指定的时间，不响应中断
                        U.park(false, time);  // emulate LockSupport.park
                    // 设置node线程节点的线程为空(此时node的线程已经得到了锁，或者超时了)
                    node.thread = null;
                    // 清空线程阻塞者
                    U.putObject(wt, PARKBLOCKER, null);
                    // 判断在阻塞期间线程是否被中断
                    if (interruptible && Thread.interrupted())
                    	// 调用cancelWaiter方法并返回结果
                        return cancelWaiter(node, node, true);
                }
            }
        }
    }
    

#### 4.2.6 tryDecReaderOverflow

读锁线程持有数量超过读锁能记录的最大值时，释放处理(溢出逆处理)
    
    
    // 读锁溢出逆处理
    private long tryDecReaderOverflow(long s) {
        // assert (s & ABITS) >= RFULL;
        // 传入的锁状态的值大于读锁能记录的数量
        // 如果锁状态中记录的读锁持有线程的数量刚好是最大值
        if ((s & ABITS) == RFULL) {
        	// 那么将可记录的最大值设置为锁状态的读状态最大值
        	// 0111 1110 => 0111 1111
            if (U.compareAndSwapLong(this, STATE, s, s | RBITS)) {
            	// 定义变量
                int r; long next;
                // 获取溢出线程数量
                // 如果溢出线程数量大于0
                if ((r = readerOverflow) > 0) {
                	// 那么将溢出线程数量--
                    readerOverflow = r - 1;
                    // 设置返回值为 0111 1111
                    next = s;
                }
                else
                	// 否则返回锁状态--
                    next = s - RUNIT;
                 state = next;
                 return next;
            }
        }
        // 如果读锁没有达到最大值，那么让出CPU时间，并且返回
        else if ((LockSupport.nextSecondarySeed() &
                  OVERFLOW_YIELD_RATE) == 0)
            Thread.yield();
        return 0L;
    }
    

#### 4.2.7 unstampedUnlockWrite

不使用凭证释放写锁
    
    
    // 不使用凭证释放写锁
    final void unstampedUnlockWrite() {
    	// 定义变量
        WNode h; long s;
        // 获取锁状态，如果锁不是写锁
        if (((s = state) & WBIT) == 0L)
        	// 抛出异常
            throw new IllegalMonitorStateException();
        // 释放写锁
        // 1000 0000 + 1000 0000 = 1 0000 0000
        state = (s += WBIT) == 0L ? ORIGIN : s;
        // 如果CLH队列不空
        if ((h = whead) != null && h.status != 0)
        	// 那么释放线程节点
            release(h);
    }
    

#### 4.2.8 unstampedUnlockRead

不使用凭证释放读锁
    
    
    // 不使用凭证释放读锁
    final void unstampedUnlockRead() {
    	// 自旋
        for (;;) {
        	// 定义变量
            long s, m; WNode h;
            // 锁空闲或者锁是写锁
            if ((m = (s = state) & ABITS) == 0L || m >= WBIT)
            	// 抛出异常
                throw new IllegalMonitorStateException();
            // 如果是读锁，且读锁持有线程数量没有溢出
            else if (m < RFULL) {
            	// 释放读锁
                if (U.compareAndSwapLong(this, STATE, s, s - RUNIT)) {
                	// CLH队列不空，释放线程节点
                    if (m == RUNIT && (h = whead) != null && h.status != 0)
                        release(h);
                    // 直到CLH队列空
                    break;
                }
            }
            // 如果是读锁，且读锁持有线程数量溢出，那么进行读锁溢出逆处理
            else if (tryDecReaderOverflow(s) != 0L)
            	// 结束
                break;
        }
    }
    

## 5\. WNode

线程等待队列节点
    
    
    static final class WNode {
    	// 前继节点(写)
        volatile WNode prev;
        // 后继节点(写)
        volatile WNode next;
        // 等待线程(读)
        volatile WNode cowait;    // list of linked readers
        // 线程节点的线程
        volatile Thread thread;   // non-null while possibly parked
        // 线程节点的状态
        volatile int status;      // 0, WAITING, or CANCELLED
        // 线程节点的模式
        final int mode;           // RMODE or WMODE
        // 构造方法：传入模式和位置
        // 表示构造一个m模式的线程节点，放在p后面
        // 后继交给持有者维护
        WNode(int m, WNode p) { mode = m; prev = p; }
    }
    

## 6\. ReadWriteLockView

读写视图
    
    
    // ReadWriteLockView
    final class ReadWriteLockView implements ReadWriteLock {
    	// readLock 调用asReadLock获取读锁视图对象
        public Lock readLock() { return asReadLock(); }
        // writeLock 调用asWriteLock获取写锁视图对象
        public Lock writeLock() { return asWriteLock(); }
    }
    

## 7\. WriteLockView

写视图
    
    
    // WriteLockView
    final class WriteLockView implements Lock {
    	// lock转向writeLock
        public void lock() { writeLock(); }
        // lockInterruptibly转向writeLockInterruptibly
        public void lockInterruptibly() throws InterruptedException {
            writeLockInterruptibly();
        }
        // tryLock转向tryWriteLock
        public boolean tryLock() { return tryWriteLock() != 0L; }
        // tryLock(long,TimeUnit)转向tryWriteLock
        public boolean tryLock(long time, TimeUnit unit)
            throws InterruptedException {
            return tryWriteLock(time, unit) != 0L;
        }
        // unlock转向unstampedUnlockWrite
        public void unlock() { unstampedUnlockWrite(); }
        // newCondition返回UnsupportedOperationException异常
        // 写锁也不支持condition
        public Condition newCondition() {
            throw new UnsupportedOperationException();
        }
    }
    

## 8\. ReadLockView

读视图  
readLockView
    
    
    // 定义ReadLockView
    final class ReadLockView implements Lock {
    	// lock方法转向readLock方法
        public void lock() { readLock(); }
        // lockInterruptibly方法转向readLockInterruptibly
        public void lockInterruptibly() throws InterruptedException {
            readLockInterruptibly();
        }
        // tryLock转向tryReadLock
        public boolean tryLock() { return tryReadLock() != 0L; }
        // tryLock转向tryLock(long,TimeUnit)
        public boolean tryLock(long time, TimeUnit unit)
            throws InterruptedException {
            return tryReadLock(time, unit) != 0L;
        }
        // unlock转向 unstampedUnlockRead
        public void unlock() { unstampedUnlockRead(); }
        // newCondition转向UnsupportedOperationException
        // 读锁不支持Condition,和ReentrantReadWriteLock
        public Condition newCondition() {
            throw new UnsupportedOperationException();
        }
    }
    

## 9\. 示例

### 9.1 读锁
    
    
    public class MyStampedLock {
    
        private static volatile int sum = 0;
    
        public static void main(String[] args) {
    
            StampedLock stampedLock = new StampedLock();
    
            Runnable reader = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取读锁
                state = stampedLock.readLock();
                // 线程睡眠100ms
                try {
                    System.out.println(name + " readLock , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                    Thread.sleep(100);
                } catch (InterruptedException ie) {
                    System.out.println(name + "ie");
                } finally {
                    // 释放锁
                    stampedLock.unlockRead(state);
                }
            };
    
            System.out.println("start ~!~");
            new Thread(reader, "reader-1").start();
            new Thread(reader, "reader-2").start();
            new Thread(reader, "reader-3").start();
            new Thread(reader, "reader-4").start();
        }
    
    }
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/25073af1267289afe1c13d699c15110d.png)

### 9.2 写锁
    
    
    public class MyStampedLock {
    
        private static volatile int sum = 0;
    
        public static void main(String[] args) throws InterruptedException {
    
            StampedLock stampedLock = new StampedLock();
    
            Runnable reader = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取读锁
                state = stampedLock.readLock();
                // 线程睡眠100ms
                try {
                    System.out.println(name + " readLock , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                    Thread.sleep(100);
                } catch (InterruptedException ie) {
                    System.out.println(name + "ie");
                } finally {
                    // 释放锁
                    stampedLock.unlockRead(state);
                }
            };
            Runnable writer = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取写锁
                state = stampedLock.writeLock();
                sum++;
                // 线程睡眠100ms
                try {
                    System.out.println(name + " writeLock , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                    Thread.sleep(100);
                } catch (InterruptedException ie) {
                    System.out.println(name + "ie");
                } finally {
                    // 释放写锁
                    stampedLock.unlockWrite(state);
                }
            };
            System.out.println("start ~!~");
            new Thread(reader, "reader-1").start();
            new Thread(reader, "reader-2").start();
            new Thread(writer, "writer").start();
        }
    
    }
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0f39f3bed51766c9d1147fad0339b1b1.png)

### 9.3 写锁 => 读锁–锁降级
    
    
    public class MyStampedLock {
    
        private static volatile int sum = 0;
    
        public static void main(String[] args) throws InterruptedException {
    
            StampedLock stampedLock = new StampedLock();
    
            Runnable reader = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取读锁
                state = stampedLock.readLock();
                // 线程睡眠100ms
                try {
                    System.out.println(name + " readLock , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                    Thread.sleep(100);
                } catch (InterruptedException ie) {
                    System.out.println(name + "ie");
                } finally {
                    // 释放锁
                    stampedLock.unlockRead(state);
                }
            };
            Runnable writer = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取写锁
                state = stampedLock.writeLock();
                sum++;
                // 线程睡眠100ms
                try {
                    System.out.println(name + " writeLock , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                    Thread.sleep(100);
                } catch (InterruptedException ie) {
                    System.out.println(name + "ie");
                } finally {
                    // 释放写锁
                    stampedLock.unlockWrite(state);
                }
            };
    
            Runnable writer2Reader = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取写锁
                state = stampedLock.writeLock();
                sum++;
                // 转换为读锁
                // 锁降级
                state = stampedLock.tryConvertToReadLock(state);
                // 线程睡眠200ms
                try {
                    System.out.println(name + " , writer2Reader , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                    Thread.sleep(200);
                } catch (InterruptedException ie) {
                    System.out.println(name + "ie");
                } finally {
                    // 释放锁
                    stampedLock.unlock(state);
                }
    
            };
            System.out.println("start ~!~");
            new Thread(reader, "reader-1").start();
            new Thread(reader, "reader-2").start();
            new Thread(writer, "writer").start();
    
            new Thread(reader, "reader-2").start();
            new Thread(writer2Reader, "writer2Reader-writer").start();
            new Thread(reader, "writer2Reader-reader").start();
        }
    
    }
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5d09b7b0fda82bb9ad28bbf9e9a96af2.png)

### 9.4 读锁 => 写锁–锁升级
    
    
    public class MyStampedLock {
    
        private static volatile int sum = 0;
    
        public static void main(String[] args) throws InterruptedException {
    
            StampedLock stampedLock = new StampedLock();
    
            Runnable reader = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取读锁
                state = stampedLock.readLock();
                // 线程睡眠100ms
                try {
                    System.out.println(name + " readLock , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                    Thread.sleep(100);
                } catch (InterruptedException ie) {
                    System.out.println(name + "ie");
                } finally {
                    // 释放锁
                    stampedLock.unlockRead(state);
                }
            };
            Runnable writer = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取写锁
                state = stampedLock.writeLock();
                sum++;
                // 线程睡眠100ms
                try {
                    System.out.println(name + " writeLock , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                    Thread.sleep(100);
                } catch (InterruptedException ie) {
                    System.out.println(name + "ie");
                } finally {
                    // 释放写锁
                    stampedLock.unlockWrite(state);
                }
            };
    
            Runnable writer2Reader = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取写锁
                state = stampedLock.writeLock();
                sum++;
                // 转换为读锁
                // 锁降级
                state = stampedLock.tryConvertToReadLock(state);
                // 线程睡眠200ms
                try {
                    System.out.println(name + " , writer2Reader , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                    Thread.sleep(200);
                } catch (InterruptedException ie) {
                    System.out.println(name + "ie");
                } finally {
                    // 释放锁
                    stampedLock.unlock(state);
                }
    
            };
    
            Runnable reader2Writer = () -> {
                String name = Thread.currentThread().getName();
                // 定义凭证
                long state = 0;
                // 获取读锁
                state = stampedLock.readLock();
                System.out.println(name + " , reader2Writer , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                // 锁升级
                try {
                    Thread.sleep(150);
                    if (stampedLock.validate(state)) {
                        state = stampedLock.tryConvertToWriteLock(state);
                    } else {
                        state = stampedLock.writeLock();
                    }
                    sum++;
                    System.out.println(name + " , reader2Writer , state = \t" + Long.toBinaryString(state) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
                } catch (InterruptedException ie) {
                    System.out.println(name + "id");
                } finally {
                    stampedLock.tryWriteLock();
                }
            };
            System.out.println("start ~!~");
    //        new Thread(reader, "reader-1").start();
    //        new Thread(reader, "reader-2").start();
    //        new Thread(writer, "writer").start();
    //        new Thread(reader, "reader-2").start();
    //        new Thread(writer2Reader, "writer2Reader-writer").start();
    //        new Thread(reader, "writer2Reader-reader").start();
            new Thread(reader, "reader2Writer-reader").start();
            new Thread(reader2Writer, "reader2Writer-writer").start();
        }
    
    }
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5d5f8062acc200dca3b5680516f9c09a.png)

### 9.5 ReadLock
    
    
    Runnable lockerReader = () -> {
        String name = Thread.currentThread().getName();
        Lock read = stampedLock.asReadLock();
        read.lock();
        try {
            System.out.println(name + " , locker , state = \t" + Long.toBinaryString(0) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
        } finally {
            read.unlock();
        }
    };
    

### 9.6 WriteLock
    
    
    Runnable lockerWriter = () -> {
        String name = Thread.currentThread().getName();
        Lock write = stampedLock.asWriteLock();
        write.lock();
        try {
            sum++;
            System.out.println(name + " , locker , state = \t" + Long.toBinaryString(0) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
        } finally {
            write.unlock();
        }
    };
    

### 9.7 ReadWriteLock
    
    
    Runnable readWriter = () -> {
        String name = Thread.currentThread().getName();
        ReadWriteLock readWriteLock = stampedLock.asReadWriteLock();
        Lock read = readWriteLock.readLock();
        Lock write = readWriteLock.writeLock();
        write.lock();
        try {
            sum++;
            System.out.println(name + " , locker , state = \t" + Long.toBinaryString(0) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
        } finally {
            write.unlock();
        }
    
        read.lock();
        try {
            System.out.println(name + " , locker , state = \t" + Long.toBinaryString(0) + " , sum = " + sum + " , time = " + System.currentTimeMillis());
        } finally {
            read.unlock();
        }
    };
    

## 10\. 总结

  *     1. StampedLock的方法比较乱 ，比较难懂，没有ReentrantLock那么清晰明了
  *     2. StampedLock的锁的状态的含义比较多，刚开始完全是空白的
  *     3. StampedLock的代码格式有些像C++的，经常是一行有多个操作
  *     4. 兼容并济，StampedLock既可以像ReadWriteLock那样简单使用，也可以使用stamp的高级使用
  *     5. StampedLock实现了ReentrantLock希望有的锁升级
  *     6. 可以提升性能

虽然看完了StampedLock的源码，但是最大的收获是会使用StampedLock。不在陌生，不在惧怕大段代码，熟悉高密度代码。
