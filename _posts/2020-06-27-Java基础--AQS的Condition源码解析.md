---
layout: post
title: "Java基础--AQS的Condition源码解析"
date: 2020-06-27 18:26:00 +0800
categories: [Condition源码解读, AQS中的Condition, ConditionObject, 解读Condition的源码, Condition中的知识]
description: "深入解析AQS中Condition的实现机制，包括Condition的谱系图、接口定义、ConditionObject存储结构，以及核心方法如addConditionWaiter、doSignal、doSignalAll等的详细流程。阐述了如何在等待队列中添加线程、信号通知、清理取消等待线程等关键步骤。"
keywords: Condition源码解读, AQS中的Condition, ConditionObject, 解读Condition的源码, Condition中的知识
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/106889626
> - 发布时间：2020-06-27 18:26:00
> - 阅读量：731
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, 多线程
> - 标签：#Condition源码解读, #AQS中的Condition, #ConditionObject, #解读Condition的源码, #Condition中的知识

## 摘要

文章浏览阅读731次。深入解析AQS中Condition的实现机制，包括Condition的谱系图、接口定义、ConditionObject存储结构，以及核心方法如addConditionWaiter、doSignal、doSignalAll等的详细流程。阐述了如何在等待队列中添加线程、信号通知、清理取消等待线程等关键步骤。

---

#### Java基础--AQS的Condition源码解析

  * 1\. Condition 谱系图
  * 2\. Condition 接口
  * 3\. AQS 中实现的Condition接口
  *     * 3.1 ConditionObject 存储结构
    * 3.2 addConditionWaiter
    * 3.3 doSignal
    * 3.4 doSignalAll
    * 3.5 unlinkCancelledWaiters
    * 3.6 signal
    * 3.7 signalAll
    * 3.8 awaitUninterruptibly
    * 3.9 await
    * 3.10 awaitNanos
    * 3.11 awaitUntil
    * 3.12 await(long time, TimeUnit unit)
    * 3.13 getWaitQueueLength

## 1\. Condition 谱系图

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4d381a7fc9426699c8ef9030e76c574b.png)

## 2\. Condition 接口

  * await  
当前线程在接到信号或被中断前处于等待状态
  * await(long time,TimeUnit unit)  
当前线程在接到信号、被中断或者到达指定等待时间之前一直处于等待状态
  * awaitNanos(long nanosTimeout)  
当前线程在接到信号、被中断或者到达指定等待时间之前一直处于等待状态
  * awaitUninterruptibly()  
当前线程在接到信号之前一直处于等待状态
  * awaitUntil(Date deadline)  
当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态
  * signal()  
唤醒一个等待线程
  * signalAll()  
唤醒所有等待线程

## 3\. AQS 中实现的Condition接口

### 3.1 ConditionObject 存储结构

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b91a7196175d45b6ed9632e6d296fa76.png)  
在ConditionObject中，保存了条件队列的第一个等待节点，以及条件队列的最后一个等待节点。  
  
请注意，在AQS中，有两个双向队列，在AQS外部有一个，分别用head和tail进行引用双向列表的头结点和尾节点；在ConditionObject中也有一个双向列表，用firstWaiter和lastWaiter进行引用双向列表的头节点和尾节点。  

### 3.2 addConditionWaiter
    
    
    // 新增当前线程节点到双向列表中
    private Node addConditionWaiter() {
        Node t = lastWaiter; // 获取双向列表的尾节点
        // If lastWaiter is cancelled, clean out.
        // 如果双向列表的尾节点不是CONDITION节点，那么调用unlinkCancelledWaiters方法清理双向列表
        if (t != null && t.waitStatus != Node.CONDITION) {
            unlinkCancelledWaiters();
            // 在unlinkCancelledWaiters方法中已经更新了尾节点
            t = lastWaiter;
        }
        // 根据当前线程创建当前线程的CONDITION的线程节点
        Node node = new Node(Thread.currentThread(), Node.CONDITION);
        // 如果t为空，表示尾节点为空，也就是说，双向列表中现在是空的
        if (t == null)
        	// 将当前新增的线程节点设置为双向列表的头节点
            firstWaiter = node;
        else
        	// 否则将新增的线程节点连接到双向列表的尾节点上
            t.nextWaiter = node;
        // 更新尾节点
        // 如果双向列表中只有一个线程节点，此时头结点和尾节点相同
        // 如果双向列表中一个线程节点都没有，那么此时尾节点为空，头节点初始化的是空节点
        lastWaiter = node;
        return node;
    }
    
    
    
    // 清理双向列表中不是CONDITION的节点
    private void unlinkCancelledWaiters() {
    	// 得到双向列表的头，作为循环的第一个节点
        Node t = firstWaiter;
        // 定义当前循环节点的前继节点
        Node trail = null;
        // 循环节点不为空，那么就一直循环
        while (t != null) {
        	// 获取循环节点的后继节点
            Node next = t.nextWaiter;
            // 如果当前循环节点的等待状态不是CONDITION
            if (t.waitStatus != Node.CONDITION) {
            	// 设置当前循环的后继节点为空 
            	// 当前循环节点与双向列表做后继断裂
                t.nextWaiter = null;
                // 如果当前循环节点的前继为空，表示当前节点是头结点(头结点没有前继节点)
                if (trail == null)
                	// 头结点设置为当前循环节点的后继节点
                    firstWaiter = next;
                else
                // 如果当前循环节点的前继不为空，那么设置当前循环节点的前继节点的后继节点为当前循环节点的后继节点
                // a -> b -> c => a -> c
                    trail.nextWaiter = next;
                // 如果当前循环节点的后继节点的后继为空，那么表示当前循环节点是尾节点(尾节点没有后继节点)
                // 类似于 a -> b -> c ： t<=> c
                if (next == null)
                	// 如果尾节点不存在表示当前节点已经是尾节点，那么将当前循环节点的前继节点设置为尾节点
                	// 类似于 a-> b -> c: t <=> c;trail <=> b
                    lastWaiter = trail;
            }
            // 如果当前循环节点是CONDITION节点，那么设置循环节点的前继节点为当前循环节点(准备下一次循环)
            else
                trail = t;
            // 偏移当前循环节点
            t = next;
        }
    }
    

### 3.3 doSignal
    
    
    // 执行通知操作，通知等待通知的线程节点。将CONDITION节点转换为SIGNAL节点
    private void doSignal(Node first) {
        do {
        	// 因为头节点是一个空节点，所以在通知的时候需要跳过头结点
            if ( (firstWaiter = first.nextWaiter) == null)
            	// 如果头结点的后继节点为空，那么表示双向列表为空
            	// 设置尾节点为空
                lastWaiter = null;
            // 如果first的后继节点不为空，那么将first的后继节点设置为空
            first.nextWaiter = null;
        } while (!transferForSignal(first) &&
                 (first = firstWaiter) != null);
    }
    
    
    
    // 将CONDITION节点转为SIGNAL节点
    final boolean transferForSignal(Node node) {
        /*
         * If cannot change waitStatus, the node has been cancelled.
         */
        // 将CONDITION节点的等待状态设置为初始化状态0，如果设置失败，那么说明当前节点不是CONDITION节点了
        // 那么就跳过这个节点，结束，进行doSignal的下一次循环
        // 如果设置成功，那么，这个节点需要从等待通知状态转移到等待竞争锁的双向列表中了，那么就继续执行下面的代码
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;
    
        /*
         * Splice onto queue and try to set waitStatus of predecessor to
         * indicate that thread is (probably) waiting. If cancelled or
         * attempt to set waitStatus fails, wake up to resync (in which
         * case the waitStatus can be transiently and harmlessly wrong).
         */
       	// 将node加到双向列表的尾节点处
       	// 请注意，在AQS中有两个双向列表：AQS等待竞争锁双向列表;ConditionObject等待通知双向队列
       	// 这里将节点node从等待通知双向队列转移到了等待竞争锁双向列表
       	// 等待通知双向列表中的节点的等待状态都是CONDITION状态
        Node p = enq(node);
        // 获取node的等待状态
        int ws = p.waitStatus;
        // 如果node的状态是取消状态或者设置线程节点的等待状态为SIGNAL失败
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        	// 唤醒线程
            LockSupport.unpark(node.thread);
    	// 结束循环
        return true;
    }
    

### 3.4 doSignalAll
    
    
    private void doSignalAll(Node first) {
    	// 因为需要将等待通知列表中的节点全部转换，所以直接清空等待通知列表的头结点和尾节点
        lastWaiter = firstWaiter = null;
        do {
        	// 等待通知双向列表的头结点已经保存在了first中了
        	// 获得双向列表中头结点的后继节点
            Node next = first.nextWaiter;
            // 将当前循环节点的后继节点设置为空，后继断裂
            first.nextWaiter = null;
            // 将当前CONDITION节点转换为SIGNAL节点
            transferForSignal(first);
            // 偏移当前节点为后继节点
            first = next;
        } while (first != null);// 循环节点不为空
    }
    

### 3.5 unlinkCancelledWaiters
    
    
    private void unlinkCancelledWaiters() {
    	// 获取等待通知双向列表的头结点
        Node t = firstWaiter;
        // 定义循环节点的前继节点
        Node trail = null;
        while (t != null) { // 循环节点不为空
            // 获取循环节点的后继节点
            Node next = t.nextWaiter;
            // 循环节点不是CONDITION节点
            if (t.waitStatus != Node.CONDITION) {
            	// 循环节点的后继节点设置为空
                t.nextWaiter = null;
                // 如果循环节点的前继节点为空，那么说明循环节点就是头节点
                if (trail == null)
                	// 设置头节点为循环节点的后继节点
                    firstWaiter = next;
                else
                	// 如果循环节点的前继节点不为空，那么说明循环节点是中间节点
                	// 设置循环节点的前继节点的后继是循环节点的后继节点
                	// 简单来说就是跳过循环节点，将循环节点前面和后面连接起来
                    trail.nextWaiter = next;
                // 如果循环节点的后继节点为空，表示循环节点就是尾节点
                if (next == null)
                	// 设置尾节点是循环节点的前继节点(跳过循环节点)
                    lastWaiter = trail;
            }
            else
            	// 循环节点是CONDITION节点
            	// 偏移循环节点的前继节点是循环节点
                trail = t;
            // 偏移循环节点是循环节点的后继节点
            t = next;
        }
    }
    

### 3.6 signal
    
    
    public final void signal() {
    	// 是否是独占模式，只有condition才会用到
        if (!isHeldExclusively())
        	// 如果不是独占模式，那么抛出异常
            throw new IllegalMonitorStateException();
        // 获得等待通知双向列表的头节点
        Node first = firstWaiter;
        // 如果头节点不为空，那么通知头结点
        if (first != null)
            doSignal(first);
    }
    

这也是为什么在doSignal中头节点需要向后偏移一次。  
因为这个方法只会处理第一个节点

### 3.7 signalAll
    
    
    public final void signalAll() {
    	// 是否是独占模式
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        // 获取等待通知双向列表
        Node first = firstWaiter;
        // 头结点不为空,意味着等待通知双向列表不为空
        if (first != null)
        	// 从第一个节点开始，通知全部节点
            doSignalAll(first);
    }
    

因为通知了全部节点，所以头结点和尾节点就没有必要再引用旧的不属于等待通知的节点了，所以在doSignalAll中清空了头结点和尾节点

### 3.8 awaitUninterruptibly

这个方法的作用是阻塞等待通知，阻塞期间不响应中断  
其时序图如下：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/43d7a3e2cbb3ba2f22b69126c7d2f4fd.png)  
这个方法看上去很复杂，不过不要怕，我们一步一步看
    
    
    // 等待通知，不响应中断
    public final void awaitUninterruptibly() {
    	// 将当前线程添加到等待通知双向队列中
        Node node = addConditionWaiter();
        // 调用fullyRelease方法，传入的是等待通知队列中的尾节点
        int savedState = fullyRelease(node); // 这里得到的等待状态是 -2
        boolean interrupted = false;
        // 如果不是等待竞争队列节点，那么一直循环，直到找到等待竞争队列的节点
        while (!isOnSyncQueue(node)) { 
            LockSupport.park(this); // 如果等待竞争队列中不存在等待竞争节点，那么阻塞当前线程，进入自旋
            if (Thread.interrupted()) // 获取线程的中断标志，并重置中断标志，这里是不响应中断，只是记录了中断状态
                interrupted = true;
        }
        // 如果等待竞争队列中存在等待竞争节点，那么就自旋获取锁
        if (acquireQueued(node, savedState) || interrupted) // 如果自旋获取锁失败或者线程被中断，那么就中断线程
            selfInterrupt();
    }
    
    
    
    final int fullyRelease(Node node) {
    	// 完全尝试释放锁
        boolean failed = true;
        try {
        	// 获取线程节点的等待状态(这个节点是传入的等待通知队列的尾节点)
            int savedState = getState();
            // 调用独占锁实现的释放锁的方法,如果释放失败会抛出异常
            // 如果调用定义的tryRelease尝试释放锁成功，那么就会获取等待竞争队列的头结点
            // 然后唤醒等待竞争队列的第二个节点(等待竞争锁队列的头结点是空节点)
            if (release(savedState)) { // 此时savedState 的值是 -2
            	// 等待通知队列的尾节点释放锁成功
                // 是否取消等待通知队列的尾节点
                failed = false;
                // 返回等待通知
                return savedState;
            } else {
                throw new IllegalMonitorStateException();
            }
        } finally {
            if (failed)
                node.waitStatus = Node.CANCELLED;
        }
    }
    
    
    
    // 释放锁
    public final boolean release(int arg) {
    	// 尝试释放锁 ，此时arg = -2
        if (tryRelease(arg)) {
            Node h = head; // 获取等待竞争线程节点队列
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h); // 唤醒队列最前面的线程节点的线程
            return true;
        }
        return false;
    }
    
    
    
    //唤醒线程，请注意，这里真正唤醒的是传入节点的后继节点
    private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        // 只要节点不是取消的和已经唤醒的，那么就设置节点的状态为已唤醒
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
    
        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next; // 这里唤醒的是后继节点，因为传入的永远是头结点或者已唤醒的节点
        // 如果后继节点已经取消了，那么需要找到队列中最前面的非取消节点(遍历的范围是传入节点后面的队列节点)
        if (s == null || s.waitStatus > 0) {
            s = null;
            // 倒序遍历
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread); // 唤醒线程
    }
    
    
    
    // 是否是等待竞争队列的节点
    final boolean isOnSyncQueue(Node node) {
    	// 等待通知队列节点和头结点不是等待等待竞争队列的节点
        if (node.waitStatus == Node.CONDITION || node.prev == null)
            return false;
        // 如果有后继节点，那么此节点一定在等待通知队列中间位置
        if (node.next != null) // If has successor, it must be on queue
            return true;
        /*
         * node.prev can be non-null, but not yet on queue because
         * the CAS to place it on queue can fail. So we have to
         * traverse from tail to make sure it actually made it.  It
         * will always be near the tail in calls to this method, and
         * unless the CAS failed (which is unlikely), it will be
         * there, so we hardly ever traverse much.
         */
        return findNodeFromTail(node);
    }
    
    
    
    // 从为节点开始，遍历等待通知队列，查找等待通知队列中是否存在传入的节点
    private boolean findNodeFromTail(Node node) {
        Node t = tail;
        for (;;) {
            if (t == node)
                return true;
            if (t == null)
                return false;
            t = t.prev;
        }
    }
    
    
    
    // 自旋获取锁
    final boolean acquireQueued(final Node node, int arg) { // 这里传入的节点是等待通知队列的节点，arg = -2
        boolean failed = true; // 获取锁成功的状态
        try {
            boolean interrupted = false; // 是否中断状态
            for (;;) {
                final Node p = node.predecessor(); // 获取传入节点的前继节点
                if (p == head && tryAcquire(arg)) { // 如果传入节点是头节点，且尝试获取锁成功
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    

### 3.9 await

这是await的时序图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d52b14e1bbd8601c1df8431acfabfafc.png)  
这些方法基本逻辑都差别不大，详细阅读了一个方法源码，根据源码一步一步往下看，除了一些细节，大致上差不多。
    
    
    // 等待通知，响应中断
    public final void await() throws InterruptedException {
    	// 获取线程的中断标志，并重置中断标志
        if (Thread.interrupted())
        	// 如果线程已经中断，那么直接抛出中断异常快速失败
            throw new InterruptedException();
        // 将当前节点加入等待通知队列
        Node node = addConditionWaiter();
        // 尝试获取锁
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        // 是否是等待竞争队列节点，这里其实是自旋，如果等待通知队列节点一直没有收到通知，那么这个循环一直成立，一直在自旋等待线程节点从等待通知队列转移到等待竞争队列
        while (!isOnSyncQueue(node)) {
            LockSupport.park(this);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0) // 如果线程节点已经被中断，那么直接结束自旋
                break;
        }
        // 因为等待通知线程节点已经被转移到了等待竞争队列，所以这里调用自旋等待获取锁即可
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE) // 自旋过程中线程被中断，那么就设置中断标志
            interruptMode = REINTERRUPT;
        // 如果有后继节点，那么对后继节点进行清理等待通知队列
        if (node.nextWaiter != null) // clean up if cancelled 
            unlinkCancelledWaiters();
        // 如果线程已经被中断，那么就根据中断模式，判断是否是当前线程在自旋过程中被中断还是其他线程已经中断了线程，然后进行中断或者抛出异常
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }
    
    
    
    // 检测中断状态
    private int checkInterruptWhileWaiting(Node node) {
        return Thread.interrupted() ? // 获取线程的中断标志，且重置
            (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) : // 返回true表示这个节点是由当前线程进行的转移，否则就
            // 是其他线程进行的转移，那么哪个线程负责的转移，就由哪个线程负责抛出中断异常
            // 其他线程只是获取下状态
            0; // 未中断，直接返回0
    }
    
    
    
    final boolean transferAfterCancelledWait(Node node) {
    	// cas将等待通知节点的等待状态设置为0，表示已经得到通知，可以将等待通知线程节点转移到等待竞争队列
        if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) { // 这一步是为了将node设置为新建的等待竞争队列节点，这个0表示这个节点可以当做等待竞争队列的新建节点
        	// 将node加入到等待竞争队列中
            enq(node);
            return true;
        }
        /*
         * If we lost out to a signal(), then we can't proceed
         * until it finishes its enq().  Cancelling during an
         * incomplete transfer is both rare and transient, so just
         * spin.
         */
         // 表示node的等待状态已经被修改，那么就自旋等待转移完成，此时可以理解为其他线程正在转移，但是还未转移完成
        while (!isOnSyncQueue(node))
            Thread.yield(); // 如果在等待竞争队列中没有找到这个节点，那么就自旋等待转移完成
        return false;
    }
    
    
    
    // 中断处理模式，到底是直接抛出异常还是直接线程中断(这里需要注意，抛出异常，但是线程并未中断)
    // 为什么这么做？当前线程处理的节点和其他线程处理的节点可能是同一个节点，那么，在其他线程中已经进行中断
    // 那么当前线程就是对中断线程进行中断，所以抛出中断异常
    private void reportInterruptAfterWait(int interruptMode)
        throws InterruptedException {
        if (interruptMode == THROW_IE)
            throw new InterruptedException();
        else if (interruptMode == REINTERRUPT)
            selfInterrupt();
    }
    

### 3.10 awaitNanos

这是awaitNanos的时序图：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fcc23e7b4dbcc10363b3b8eac7d782b1.png)  
这个等待一定的时间的方法和前面的等待大同小异吧
    
    
    // 等待指定的时间，如果时间到了，还未收到通知，那么就抛出结束等待，同时在等待的期间，响应中断异常
    public final long awaitNanos(long nanosTimeout)
            throws InterruptedException {
        // 首先检测线程中断，且重置
        if (Thread.interrupted())
            throw new InterruptedException(); // 如果线程已经被中断，直接抛出中断异常
        Node node = addConditionWaiter(); // 将当前线程加入到等待通知队列尾
        // 尝试释放锁
        int savedState = fullyRelease(node); // 此时获取的状态是-2
        final long deadline = System.nanoTime() + nanosTimeout; // 获取当前时间，当前时间加上等待时间，就是截止时间
        int interruptMode = 0; // 中断模式
        while (!isOnSyncQueue(node)) { // 自旋等待线程节点从等待通知队列转移到等待竞争队列
            if (nanosTimeout <= 0L) { // 如果等待时间小于等于0，那么就取消当前节点
                transferAfterCancelledWait(node);
                break; // 结束自旋
            }
            // 如果剩余等待时间大于100纳秒，那么就进行线程阻塞，否则进行自旋等待时间结束
            if (nanosTimeout >= spinForTimeoutThreshold)
            	// 阻塞当前线程指定时间
                LockSupport.parkNanos(this, nanosTimeout);
            // 检查等待时间内的中断标志，并重置
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            	// 如果已经被中断，那么结束自旋
                break;
            // 更新剩余等待时间
            nanosTimeout = deadline - System.nanoTime();
        }
        // 此时node节点已经从等待通知队列转移到了等待竞争队列，或者剩余等待时间为0
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE) // 如果不是当前线程处理node，那么获取中断标志且重置
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null) // 后继节点不为空，表示node后面还有节点
            unlinkCancelledWaiters(); // 清理等待通知队列中等待状态不是CONDITION的节点
        if (interruptMode != 0) // 如果中断模式不是0，表示线程已经被中断，那么需要根据中断模式判断是不是当前线程处理的中断
            reportInterruptAfterWait(interruptMode); // 如果是当前线程在自旋期间被中断，那么抛出中断异常，否则将当前线程进行中断
        return deadline - System.nanoTime(); // 成功获取锁，那么就返回剩余时间
    }
    
    
    
    final boolean transferAfterCancelledWait(Node node) {
    	// cas 将当前线程节点的等待状态设置为0，然后加入到等待竞争状态
        if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
            enq(node);
            return true; // 表示node节点是由当前线程进行转移的
        }
        /*
         * If we lost out to a signal(), then we can't proceed
         * until it finishes its enq().  Cancelling during an
         * incomplete transfer is both rare and transient, so just
         * spin.
         */
        // 如果当前节点不在等待竞争队列，那么自旋等待转移完成
        while (!isOnSyncQueue(node))
            Thread.yield(); // 让出cpu时间，给其他线程处理
        return false; // 表示node是由其他线程进行转移的
    }
    
    
    
    // 清理等待通知队列中等待状态不是CONDITION的节点
    private void unlinkCancelledWaiters() {
        Node t = firstWaiter; // 获取等待通知队列的队列头
        Node trail = null; // 设置临时变量存储循环节点的前继
        while (t != null) { // 如果等待通知队列的队列头不是空，表示等待通知队列不空
            Node next = t.nextWaiter; // 获取等待通知的队列头的后继
            if (t.waitStatus != Node.CONDITION) { // 如果等待通知队列的队列头的等待状态不是 -2
                t.nextWaiter = null; // 那么就需要将队列头从等待通知队列中移除，这里是断开队列头的后继链接
                if (trail == null) // 如果当前循环节点的前继是空，那么设置队列头是当前循环节点的后继，即队列头向后偏移
                    firstWaiter = next;
                else
                    trail.nextWaiter = next; // 否则设置当前循环节点的前继节点与当前循环节点的后继节点相连，即移除当前循环节点
                if (next == null) // 如果后继为空，那么就设置队列尾是当前循环节点的前继，即队列尾前移
                    lastWaiter = trail;
            }
            else // 如果当前循环节点的状态是 -2 ，那么偏移当前循环节点的前继
                trail = t;
            // 偏移当前循环节点是后继，进行下一次循环
            t = next;
        }
    }
    

### 3.11 awaitUntil

这个方法和3.10的方法区别在于：3.10等待时间较短，等待时间的单位是纳秒，这个等待的时间的单位是日期，范围上比3.10要大  
这是awaitUntil的时序图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d379030bedf17e5b8b7cccf53d4e7379.png)
    
    
    public final boolean awaitUntil(Date deadline)
            throws InterruptedException {
        long abstime = deadline.getTime();
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();
        int savedState = fullyRelease(node);
        boolean timedout = false;
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) { // 自旋等待node接收到通知，然后从等待通知队列转移到等待竞争队列
            if (System.currentTimeMillis() > abstime) { // 这里处理的时间单位是毫秒
                timedout = transferAfterCancelledWait(node);
                break;
            }
            LockSupport.parkUntil(this, abstime);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null)
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
        return !timedout;
    }
    

### 3.12 await(long time, TimeUnit unit)

这个比3.11更加灵活，由调用者传入时间和时间单位  
这是时序图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cac70cebae5c560aedd48fe90906d61a.png)
    
    
    public final boolean await(long time, TimeUnit unit)
            throws InterruptedException {
        long nanosTimeout = unit.toNanos(time); // 这里进行时间转换，转换为纳秒进行比较
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();
        int savedState = fullyRelease(node);
        final long deadline = System.nanoTime() + nanosTimeout;
        boolean timedout = false;
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            if (nanosTimeout <= 0L) {
                timedout = transferAfterCancelledWait(node);
                break;
            }
            if (nanosTimeout >= spinForTimeoutThreshold) // 如果大于1000纳秒进行阻塞
                LockSupport.parkNanos(this, nanosTimeout);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
            nanosTimeout = deadline - System.nanoTime();
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null)
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
        return !timedout;
    }
    

### 3.13 getWaitQueueLength

获取等待通知队列中节点的数量  
这是时序图  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/377f5b5007bca7d507c96254e424ca7c.png)
    
    
    protected final int getWaitQueueLength() {
        if (!isHeldExclusively()) // 获取当前线程是否是独占的，这个方法只有在ConditionObject中会调用
            throw new IllegalMonitorStateException();
        int n = 0;
        for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
            if (w.waitStatus == Node.CONDITION)
                ++n;
        }
        return n; // 返回等待通知队列中CONDITION的节点数量
    }
    
    
    
    // 在AQS中对内部类的方法做了代理
    public final int getWaitQueueLength(ConditionObject condition) {
        if (!owns(condition))
            throw new IllegalArgumentException("Not owner");
        return condition.getWaitQueueLength();
    }
