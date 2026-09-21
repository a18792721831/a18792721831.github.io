---
layout: post
title: "Java基础--ThreadPoolExecutor--线程池和锁"
date: 2020-07-11 17:17:23 +0800
categories: [线程池, 线程池使用, Executors使用, 线程池源码解析, 线程池是如何保证线程安全]
description: "本文详细解析Java线程池ThreadPoolExecutor的内部机制，包括线程池的创建、任务分配、线程调度、异常处理策略及核心组件Worker的工作流程。探讨了线程池如何保证线程安全，并分析了四种任务执行失败的处理方式。"
keywords: 线程池, 线程池使用, Executors使用, 线程池源码解析, 线程池是如何保证线程安全
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107071280
> - 发布时间：2020-07-11 17:17:23
> - 阅读量：1
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, 多线程
> - 标签：#线程池, #线程池使用, #Executors使用, #线程池源码解析, #线程池是如何保证线程安全

## 摘要

文章浏览阅读1.2k次。本文详细解析Java线程池ThreadPoolExecutor的内部机制，包括线程池的创建、任务分配、线程调度、异常处理策略及核心组件Worker的工作流程。探讨了线程池如何保证线程安全，并分析了四种任务执行失败的处理方式。

---

#### Java基础--ThreadPoolExecutor--线程池和锁

  * 1\. Executors
  *     * 1.1 Executors构造
    * 1.2 newFixedThreadPool(int nThreads)
    * 1.3 newCachedThreadPool
    * 1.4 newSingleThreadExecutor
    * 1.5 newScheduledThreadPool
    * 1.6 newSingleThreadScheduledExecutor
    * 1.7 newWorkStealingPool
    * 1.8 总结
  * 2\. ThreadPoolExecutor
  * 3 Executor
  * 4 ExecutorService
  * 5 RejectedExecutionHandler
  * 6 ThreadPoolExecutor
  * 7 Worker
  *     *       * tryAcquire
      * tryRelease
      * run
      *         * 构造
        * runWorker
        * getTask
        * decrementWorkerCount
        * processWorkerExit
        * tryTerminate
        * interruptIdleWorkers
  * 8\. execute
  *     *       * addWorker
      * addWorkerFailed
  * 9\. submit(AbstractExecutorService)
  * 10\. AbortPolicy
  * 11\. CallerRunsPolicy
  * 12\. DiscardOldestPolicy
  * 13\. DiscardPolicy
  * 14\. 总结

  
使用多线程的时候，经常会使用Executors创建线程池，然后使用线程池。从而达到复用线程，减少线程切换，从而增加性能。 

## 1\. Executors

### 1.1 Executors构造

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/658b5fb73d4b3409ebab9712d524ffe2.png)  
Executors是一个工具类，不能被实例化。

### 1.2 newFixedThreadPool(int nThreads)

创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。  
创建指定个线程的线程池，而且不能扩容，在刚开始就会初始化指定线程池。  
创建线程池本身会比较耗时，而且传入的数值过大时，可能造成OOM。  
参数

  * 核心线程数
  * 最大线程数
  * 非核心空闲线程存活时间
  * 非核心空闲线程存活时间单位
  * 任务存储队列  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb6c798b5eb8792ec3ef682b7e154deb.png)  
调用的是threadPoolExecutor。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/05a2c37307dd8d1b14977fd063fb9722.png)  
调用的是threadPoolExecutor。

### 1.3 newCachedThreadPool

创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。  
创建一个初始为0的可缓存的线程池。非核心空闲线程存活时间为60s.  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f243ed5a4497d1942e991c6ce74b4d56.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d8f935b828a1ba188aa417c3f17d6baf.png)

### 1.4 newSingleThreadExecutor

创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/27f185c04bd9766f3d4b7053c4e8101c.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c4f0fc5ec3bce2850507c29f7ac198c1.png)

### 1.5 newScheduledThreadPool

创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/850897b581c0bb0593863167c90a572b.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/51ec982040fcc536f3bc34cb4835f919.png)

### 1.6 newSingleThreadScheduledExecutor

创建一个单线程执行程序，它可安排在给定延迟后运行命令或者定期地执行。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0828cb154cccf4d01995a739b38cb5e4.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/061db70cc5b9d2240a1da9fa26a6d929.png)

### 1.7 newWorkStealingPool

具有抢占式操作的线程池（1.8增加）  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4f811ea5bfe039f23537334e2120523a.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99923ed10c713a920102631b327cc5de.png)

### 1.8 总结

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f5de91f48a9f54ac59f70770e2fc07a.png)

## 2\. ThreadPoolExecutor

这是ThreadPoolExecutor的结构  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/11124bdfe5ddcfd97ed8adf4903df3c8.png)

## 3 Executor

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c357a2fd98dddfec2ba8593e86f5ce3d.png)  
Executor接口只有一个方法，execute，参数是一个Runnable。  
传入一个Runnable任务，调用execute执行，其执行的时间是不确定的。

## 4 ExecutorService

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3ed5008c29e2b3f198c19f17f942706d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8cae879dfd9d3f62b8f5835a4d59c626.png)

## 5 RejectedExecutionHandler

无法由 ThreadPoolExecutor 执行的任务的处理程序。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b37c0b3607d3e8240e07d97d98379645.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f3669e170ac7b3f675ddf7e0d9204478.png)

## 6 ThreadPoolExecutor

看到这里，ThreadPoolExecutor的结构就比较明显了。  
首先ThreadPoolExecutor实现了Executor接口，传入任务，在未来时间执行。  
ThreadPoolExecutor实现了ExecutorService接口，可以用于控制ThreadPoolExecutor是否接受任务，接受有返回的任务，以及ThreadPoolExecutor的停止等。  
ThreadPoolExecutor继承AbstractExecutorServe抽象类。抽象类封装了提交任务，执行任务的通用封装。  
ThreadPoolExecutor内部聚合了Worker类，Worker类实现了Runnable接口，可以实现任务的执行。  
而且Wroker继承了AbstractQueuedSynchronizer类，用于实现多线程间数据的安全，以及相对应的线程调度。  
ThreadPoolExecutor内部还组合了多个Policy的内部类，用于当ThreadPoolExector出现异常时的处理。

## 7 Worker

在Worker里面主要看下Worker继承AQS，实现AQS要求实现的tryAcquire,tryRelease,tryAcquireShared,tryReleaseShard和isHeldExclusively方法。  
其他的请看：  
[Java基础–AQS原理](<https://blog.csdn.net/a18792721831/article/details/106730738>)  
[Java基础–AQS的Condition源码解析](<https://blog.csdn.net/a18792721831/article/details/106889626>)

#### tryAcquire

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3db1e9e23ddd31f786e3427978a6aa9a.png)  
ThreadPoolExecutor内部的Worker是独占锁，而且是二进制锁，其锁状态只有两个状态：0,1  
0表示空闲，1表示占用。

#### tryRelease

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ac3af0504aa4e2309590e8b5e8fa7890.png)  
释放锁，清空锁持有线程，然后设置锁状态为0.  
因为是独占锁，所以Worker没有实现tryAcquireShared和tryReleaseShared方法。

#### run

Wroker还实现了Runnable接口，Runnable接口只有一个方法，就是定义任务的run方法。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/94dfeb9f3eced2b64a1e520109857bcf.png)  
这是run方法的时序图，我们看下run方法都干了什么：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9b5d7883fe67e4f056a129eae8fe121d.png)  
看起来挺复杂的。

##### 构造

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ced01fa46ac7a4a661d558f24a8222ce.png)  
构造方法需要有一个Runnable 的任务来初始化。  
创建Worker时，会设置Worker的锁状态是-1,然后将传入的Runnable任务初始化任务。  
然后将当前Runnable任务传入，从ThreadFactory获取线程。

##### runWorker
    
    
    final void runWorker(Worker w) { // 传入是Worker对象(this)
    	// 获取当前线程
        Thread wt = Thread.currentThread();
        // 获取任务
        Runnable task = w.firstTask;
        // 将Worker的任务设置为空
        w.firstTask = null;
        // 将Worker的锁释放，允许Worker接收新的任务
        w.unlock(); // allow interrupts
        // 
        boolean completedAbruptly = true;
        // 尝试执行任务
        try {
        	// 任务不为空，或者调用getTask还能得到任务
            while (task != null || (task = getTask()) != null) {
            	// Worker上锁
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                // 判断线程池的状态
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    // 线程被中年高端
                    !wt.isInterrupted())
                    // 当前线程中断
                    wt.interrupt();
                try {
                	// 线程执行前准备，在ThreadPoolExecutor中是空
                    beforeExecute(wt, task);
                    // 定义一个异常超类，Exception和Error都是Thowable的子类
                    Throwable thrown = null;
                    try {
                    	// 执行任务
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                    	// 线程执行后操作，在ThreadPoolExecutor中也是空
                        afterExecute(task, thrown);
                    }
                } finally {
                	// 将任务置空，准备下一次执行任务
                    task = null;
                    // 将线程执行的任务数量++
                    w.completedTasks++;
                    // 释放锁
                    w.unlock();
                }
            }
            // 设置worker意外的标志为false
            completedAbruptly = false;
        } finally {
        	// 调用processWorkerExit进行worker退出
            processWorkerExit(w, completedAbruptly);
        }
    }
    

##### getTask
    
    
    // 获取任务
    private Runnable getTask() {
    	// 超时标志设置为false
        boolean timedOut = false; // Did the last poll() time out?
    	// 死循环
        for (;;) {
        	// 获取线程池状态
            int c = ctl.get();
            // 获取状态
            int rs = runStateOf(c);
            // Check if queue empty only if necessary.
            // 检查线程池状态和线程池任务队列
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            	// 如果线程池状态不是RUNNING或者线程池状态是SHUTDOWN但是线程池任务队列空
            	// 那么执行decrementWorkerCount
                decrementWorkerCount(); // 将线程池状态设置为0
                // 不返回任务
                return null;
            }
    		// 获取线程池现在运行的线程数量(Worker数量)
            int wc = workerCountOf(c);
    
            // Are workers subject to culling?
            // 检查线程池是否允许非核心线程超时，以及线程数量是否超过核心现场数量
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
    		// 如果线程池不允许非核心线程超时，而且线程数量超过核心线程允许数量，那么当前线程不允许在执行任务
    		// 假设允许10个核心线程，但是现在线程池已经有11个线程了，而且不允许非核心线程超时
    		// 那么就需要终止一个线程，保证核心线程有10个。但是应该终止哪个线程呢？
    		// 既然当前线程的任务已经执行完毕了，现在在调用getTask获取下一个执行的任务，
    		// 那么终止的线程就是你了
            if ((wc > maximumPoolSize || (timed && timedOut))
            	// 如果线程池线程数量小于允许的核心线程数量，但是任务队列已经空了，那么返回null
                && (wc > 1 || workQueue.isEmpty())) {
                // 因为当前线程已经不再执行任务了，所以执行任务的线程数量需要减1
                if (compareAndDecrementWorkerCount(c))
                	// 线程池线程数量减1，返回null
                    return null;
                // 如果线程池数量减1失败，那么，重试，直到线程池数量减1成功
                continue;
            }
    		// 尝试从任务队列中获取任务
            try {
            	// 如果允许超时
                Runnable r = timed ?
                	// 那么就等待指定时间，从任务队列中获取一个任务
                	// 难道多个线程池的线程同时从任务队列中获取一个任务，这里不会发生并发问题吗？(存疑)
                	// 非核心线程
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) : // 如果超时获取null
                    // 否则阻塞获取任务
                    // 核心线程
                    workQueue.take();
                // 如果任务不是null
                if (r != null)
                	// 返回任务
                    return r;
                // 否则设置超时
                timedOut = true;
            } catch (InterruptedException retry) {
            	// 出现中断异常，重试
                timedOut = false;
            }
        }
    }
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce9f52c8e8e82fbd2344aaaec63e065a.png)  
一个线程安全的整形变量。它有这几个值：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d699178f55550097902b57335f23cd42.png)  
RUNNING,SHUTDOWN,STOP,TIDYING,TERMINATED.

  * RUNNING:Worker可以接收新的任务执行
  * SHUTDOWN:不在接收新的任务，但是会将已经接收的任务执行完
  * STOP:不在接收新的任务，而且已经接收的任务也不会再运行，会给正在运行中的线程发送中断。
  * TIDYING:Worker任务列表清空，准备执行异常方法
  * TERMINATED:Worker的异常方法已经执行。

这几个状态的状态转换图：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18df7045063db2d388053a74b2094827.png)

##### decrementWorkerCount

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0f9ca52d469f6796b6dfc47f740162f0.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d9415145d1a9a512c775818efc07376a.png)  
每次将线程安全的ctl的值减1，直到为0.

##### processWorkerExit

worker意外退出
    
    
    private void processWorkerExit(Worker w, boolean completedAbruptly) {
    	// 判断worker是否发生意外
        if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        	// 如果发生意外，那么清空线程池worker的数量(相当于线程池发生意外)
            decrementWorkerCount();
    	// 获取线程池的锁
        final ReentrantLock mainLock = this.mainLock;
        // 上锁
        mainLock.lock();
        try {
        	// 线程池执行任务数量同步worker的执行成功数量
            completedTaskCount += w.completedTasks;
            // 将出现意外的worker从worker的set中移除
            workers.remove(w);
        } finally {
            mainLock.unlock();
        }
    	// 线程池进入终止阶段，尝试执行线程池终止的操作
        tryTerminate();
    
        int c = ctl.get();
        if (runStateLessThan(c, STOP)) {
            if (!completedAbruptly) {
                int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
                if (min == 0 && ! workQueue.isEmpty())
                    min = 1;
                if (workerCountOf(c) >= min)
                    return; // replacement not needed
            }
            addWorker(null, false);
        }
    }
    

##### tryTerminate

执行线程池终止的操作
    
    
    final void tryTerminate() {
    	// 死循环
        for (;;) {
        	// 获取线程池的状态
            int c = ctl.get();
            // 判断线程池的状态是否是RUNNING
            if (isRunning(c) ||
            	// 判断线程池的状态是不是TIDYING
                runStateAtLeast(c, TIDYING) ||
                // 判断线程池的状态是不是SHUTDOWN而且任务队列空(需要转换状态为TIDYING)
                (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
                // 只有TERMINATED状态的线程池才会执行异常方法
                return;
            // 线程池中还有正在运行的任务
            if (workerCountOf(c) != 0) { // Eligible to terminate
            	// 终止一个worker
                interruptIdleWorkers(ONLY_ONE);
                // 结束
                return;
            }
    		// 线程池中没有任务在执行
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
            	// 设置线程池的状态是TIDYING
                if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                    try {
                    	// 执行线程池异常操作
                        terminated();
                    } finally {
                    	// 线程池状态设置为TERMINATED
                        ctl.set(ctlOf(TERMINATED, 0));
                        // 唤醒等待的线程(等待的线程是什么呢？(存疑))
                        termination.signalAll();
                    }
                    return;
                }
            } finally {
                mainLock.unlock();
            }
            // else retry on failed CAS
        }
    }
    

##### interruptIdleWorkers

// 中断worker
    
    
    private void interruptIdleWorkers(boolean onlyOne) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            for (Worker w : workers) {
                Thread t = w.thread;
                if (!t.isInterrupted() && w.tryLock()) {
                    try {
                        t.interrupt();
                    } catch (SecurityException ignore) {
                    } finally {
                        w.unlock();
                    }
                }
                // 只中断一个worker
                if (onlyOne)
                    break;
            }
        } finally {
            mainLock.unlock();
        }
    }
    

## 8\. execute

提交任务，没有返回值
    
    
    public void execute(Runnable command) {
    	// 检测任务
        if (command == null)
            throw new NullPointerException();
        // 获取线程池worker数量
        int c = ctl.get();
        // 如果线程池中线程数量小于核心线程数量
        if (workerCountOf(c) < corePoolSize) {
        	// 增加worker
            if (addWorker(command, true))
            	// 结束
                return;
            // 增加失败，重新获取线程池中线程数量
            c = ctl.get();
        }
        // 如果线程池的状态是RUNNING,而且任务队列中加入任务成功
        if (isRunning(c) && workQueue.offer(command)) {
        	// 获取线程池线程数量
            int recheck = ctl.get();
            // 如果线程池的状态不是RUNNING，那么将当前任务从任务队列中移除
            if (! isRunning(recheck) && remove(command))
            	// 执行任务失败操作
                reject(command);
            // 如果线程池的状态还是RUNNING或者任务队列中移除当前任务失败
            else if (workerCountOf(recheck) == 0)
            	// 增加一个空的非核心worker
                addWorker(null, false);
        }
       	// 以当前任务增加worker，woeker不是核心worker
        else if (!addWorker(command, false))
        	// 执行任务失败操作
            reject(command);
    }
    

#### addWorker

增加worker
    
    
    private boolean addWorker(Runnable firstTask, boolean core) {
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);
    
            // Check if queue empty only if necessary.
            // 检测线程池状态和任务以及任务队列
            if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN &&
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                // 未通过检测
                return false;
    
            for (;;) {
                int wc = workerCountOf(c);
                if (wc >= CAPACITY || // worker数量超过允许的最大数量
                    wc >= (core ? corePoolSize : maximumPoolSize)) // worker数量小于核心worker数量
                    return false;
                // worker数量++
                if (compareAndIncrementWorkerCount(c))
                    break retry;
                c = ctl.get();  // Re-read ctl
                if (runStateOf(c) != rs)
                    continue retry;
                // else CAS failed due to workerCount change; retry inner loop
            }
        }
    
        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
        	// 新创建 worker
            w = new Worker(firstTask);
            // 获取worker的线程
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());
    
                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        // worker加入到workers队列(set)
                        workers.add(w);
                        int s = workers.size();
                        if (s > largestPoolSize)
                        	// 更新线程池worker数量
                            largestPoolSize = s;
                        // worker增加成功
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                // worker增加成功
                if (workerAdded) {
                	// 启动worker的线程
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {
        	// 如果worker启动失败
            if (! workerStarted)
            	// worker启动失败，将worker从workers队列中移除
                addWorkerFailed(w);
        }
        return workerStarted;
    }
    

#### addWorkerFailed

worker增加失败，将worker移除
    
    
    private void addWorkerFailed(Worker w) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (w != null)
                workers.remove(w);
            // 线程池worker数量--
            decrementWorkerCount();
            // 执行线程池异常操作(空)
            tryTerminate();
        } finally {
            mainLock.unlock();
        }
    }
    

## 9\. submit(AbstractExecutorService)

提交任务，有返回值  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/594c78a6f480c0ef47d7feb511e1a3e0.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a5058ab021195b2125a6f4ba276f100.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/347bc68915e4d76aaaf2ab36fa49521d.png)

## 10\. AbortPolicy

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c63bb210d93b92560384dfa9dca74e1f.png)  
任务执行失败，抛出异常。

## 11\. CallerRunsPolicy

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1cfea474ff1da354ec0644d8e08dcce6.png)  
任务被拒绝，那么在当前线程调用(不能保证是多线程，可能是主线程直接调用run方法(串行))

## 12\. DiscardOldestPolicy

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f6b81aafb489f25fbbeb002eafe6fdce.png)  
移除任务队列头的任务。  
可以理解为，任务队列满了，在加入任务的时候，会将队列头部的挤掉。

## 13\. DiscardPolicy

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f0d79c2bb00a795ae52421603bb933c.png)  
任务增加失败，什么也不做。

## 14\. 总结

ThreadPoolExecutor继承了AbstractExecutorService,AbstractExecutorService实现了任务的提交，任务的增加等方法。但是实际上调用的是ThreadPoolExecutor的execute方法。  
在ThreadPoolExecutor的execute方法中会判断线程池是否可用，如果可用，就会获取线程池的锁(ReentrantLock)，然后将任务加入任务队列。  
**线程池是如何保证线程安全的？  
1.线程池任务调度使用ReentrantLock保证任务不会被重复执行  
任务队列必须是BlockQueue类型的，BlockQueue的子类保证队列的出入的线程安全。  
2.线程池的worker节点继承了AbstractQueueSynchronizer()  
当worker在运行任务前上锁，在任务运行结束后解锁。上锁后，不会响应中断，保证开始运行的任务不会被其他线程中断。只有任务结束，才会被中断。  
3.worker的锁是不可重入的锁  
防止线程池操作调整大小，获取数量等操作时，中断线程，导致任务没有完整的被执行。**  
任务加入线程池失败有4中处理方式

  1. AbortPolicy: 抛出异常
  2. CallerRunsPolicy:主线程调用run运行
  3. DiscardOldestPolicy:抛弃任务队列头的任务，将失败的任务加入任务队列
  4. DiscardPolicy:抛弃失败的任务
