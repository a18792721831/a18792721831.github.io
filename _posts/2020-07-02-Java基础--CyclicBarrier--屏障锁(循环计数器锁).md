---
layout: post
title: "Java基础--CyclicBarrier--屏障锁(循环计数器锁)"
date: 2020-07-02 21:52:16 +0800
categories: [CyclicBarrier源码, CyclicBarrier使用, CyclicBarrier原理, 栅格锁, 屏障同步锁]
description: "CyclicBarrier是一种同步工具，用于让一组固定大小的线程在达到某个公共屏障点时互相等待，之后继续执行。本文深入解析CyclicBarrier的工作原理，包括其构造、属性、操作及示例程序。"
keywords: CyclicBarrier源码, CyclicBarrier使用, CyclicBarrier原理, 栅格锁, 屏障同步锁
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107071166
> - 发布时间：2020-07-02 21:52:16
> - 阅读量：730
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, 多线程
> - 标签：#CyclicBarrier源码, #CyclicBarrier使用, #CyclicBarrier原理, #栅格锁, #屏障同步锁

## 摘要

文章浏览阅读730次。CyclicBarrier是一种同步工具，用于让一组固定大小的线程在达到某个公共屏障点时互相等待，之后继续执行。本文深入解析CyclicBarrier的工作原理，包括其构造、属性、操作及示例程序。

---

#### Java基础--CyclicBarrier--屏障同步锁

  * [1\. CyclicBarrier](<#1__CyclicBarrier_1>)
  *     * [1.1 CyclicBarrier 的UML图](<#11_CyclicBarrier_UML_11>)
  * [2\. CyclicBarrier 构造](<#2_CyclicBarrier__13>)
  *     * [2.1 CyclicBarrier(int)](<#21_CyclicBarrierint_16>)
    * [2.2 CyclicBarrier(int,Runnable)](<#22_CyclicBarrierintRunnable_20>)
  * [3\. CyclicBarrier 的属性](<#3_CyclicBarrier__23>)
  *     * [3.1 lock](<#31_lock_24>)
    * [3.2 trip](<#32_trip_28>)
    * [3.3 parties](<#33_parties_32>)
    * [3.4 count](<#34_count_36>)
    * [3.5 barrierCommand](<#35_barrierCommand_40>)
    * [3.6 generation](<#36_generation_45>)
  * [4\. Generation](<#4_Generation_49>)
  * [5\. CyclicBarrier 的操作](<#5_CyclicBarrier__53>)
  *     * [5.1 await](<#51_await_54>)
    * [5.2 await(long, TimeUnit)](<#52_awaitlong_TimeUnit_58>)
    * [5.3 getNumberWaiting](<#53_getNumberWaiting_62>)
    * [5.4 getParties](<#54_getParties_67>)
    * [5.5 isBroken](<#55_isBroken_70>)
    * [5.6 reset](<#56_reset_75>)
    * [5.7 nextGeneration](<#57_nextGeneration_80>)
    * [5.8 breakBarrier](<#58_breakBarrier_83>)
    * [5.9 dowait(boolean, long)](<#59_dowaitboolean_long_87>)
  * [6\. 示例程序](<#6__205>)
  * [7\. 总结](<#7__288>)

## 1\. CyclicBarrier

一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。

举个例子理解屏障锁：  
学校开运动会，有短跑比赛。短跑参加的人数比较多，可能需要多次比较。  
首先将参与的运动员分组，比如10个一组。  
每一次10开始之前都需要运动员举手向裁判示意准备完成。  
当裁判收到这10个运动员的准备示意后，裁判向发令员发出命令，可以随时发令。

上述整个过程中，裁判做的工作就是CyclicBarrier的原理实现。

### 1.1 CyclicBarrier 的UML图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a27b89117286de2cd59998b45f5e5105.png)

## 2\. CyclicBarrier 构造

CyclicBarrier有两个构造方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/77dc55674c33d4ac3d6b55dbfec42653.png)

### 2.1 CyclicBarrier(int)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/42aeebba89aee08026d2d68550e3f7ec.png)  
根据传入的值，创建一个parties的线程组屏障。  
换句话说，就是循环计数器锁的值是parties.

### 2.2 CyclicBarrier(int,Runnable)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54a4de656cdb7e894456ef5c947dbfc9.png)  
设置线程数量是parites个，同时设置线程到达屏障点后，执行Runnable的操作。

## 3\. CyclicBarrier 的属性

### 3.1 lock

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/229e13781c08ebd1fd4260e864adf5fe.png)  
持有一个重入锁，使用的是不公平的重入锁。  
一次只能有一个运动员向裁判示意。

### 3.2 trip

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4a9daa72f68d32a8a4dd5a057689a31e.png)  
持有不公平的重入锁的Condition对象。  
示意准备完成的运动员需要等待。

### 3.3 parties

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aba21d019407326b10d9ce004d043436.png)  
需要等待的线程数量。  
一次有10个运动员比赛。

### 3.4 count

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b7c5cb16938549fd184733f91c9bd473.png)  
还未达成条件的线程数量。还需要继续等待的线程数量。  
当前还有多少个运动员没有准备完成。

### 3.5 barrierCommand

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76a3d18d7a54a594b00da33bdeb38051.png)  
线程到达屏障点后执行的操作，可空。  
当所有运动员准备完成后，裁判需要向指令员发信号。  
有些小型比赛，可能裁判自己发令。

### 3.6 generation

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44c7c8f482bbdb00a26f92b6c3d9e35a.png)  
当前运行的线程。(还未到达屏障处)  
当前正在示意的运动员。

## 4\. Generation

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b630e090a62b624025dfb9c85fbc16ec.png)  
是否全部等待的线程到达屏障点。默认没有到达。  
默认全部运动员都没有准备好。

## 5\. CyclicBarrier 的操作

### 5.1 await

等待全部线程到达屏障点。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ed2701ae3dfbe43acbc3f9e9fce0541c.png)  
直接调用dowait方法。

### 5.2 await(long, TimeUnit)

带有超时时间的等待全部线程到达屏障点。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d15d7101ae10a9ea1ee8b83b89bf5ca.png)  
也是调用doawait方法

### 5.3 getNumberWaiting

获取已经到达屏障点的线程数量  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c96a80d3481e2d6f9a559ba75221a2c.png)  
先获取锁，然后上锁，获取全部数量，获取还需要等待的线程数量。这两个的差就是已经到达的数量。  
最后释放锁。

### 5.4 getParties

获取屏障内线程总数  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/26b994b24dfec9b01e0fe93cb3e7752b.png)

### 5.5 isBroken

获取是否全部的线程到达屏障点。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8803eb939f2444b7518d263c8d404231.png)  
先上锁，然后获取全部线程到达屏障点的状态。  
最后释放锁。

### 5.6 reset

重置所有信息。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3453ae102c10bd0349e7318150f6c09a.png)  
先上锁，然后唤醒已经到达屏障点的线程，重置需要等待的线程为线程总数，最后重置是否全部线程到达屏障点的状态。  
最后释放锁。

### 5.7 nextGeneration

唤醒已经到达屏障点的全部线程，然后设置需要等待到达屏障点的线程为线程总数，最后重置是否全部线程到达屏障点的状态。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/718d26fb0f4edcb97d9c5987033ed5bf.png)

### 5.8 breakBarrier

是否异常。  
设置全局的异常状态为true，然后设置需要等待线程数量为线程总数，然后唤醒所有已经到达屏障点的线程。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/414019d9f5bcf470786427927bb75420.png)

### 5.9 dowait(boolean, long)

线程到达屏障处进行的操作。  
判断线程是否到达屏障处，到达屏障处做什么操作。  
运动员示意自己准备完成，应该面向裁判，看着裁判，同时举起右手，裁判点头后，放下右手，等待比赛开始。
    
    
    	// 真正实现屏障的方法
    	// await传入false,0
        private int dowait(boolean timed, long nanos)
            throws InterruptedException, BrokenBarrierException,
                   TimeoutException {
            // 获得全局的重入锁
            final ReentrantLock lock = this.lock;
            // 上锁
            lock.lock();
            try {
            	// 获取当前线程
                final Generation g = generation;
    			// 如果线程已经到达屏障处，抛出异常
    			// 运动员没有向裁判举手示意，就认为准备好了(流程不合法)
                if (g.broken)
                    throw new BrokenBarrierException();
    			// 获取线程中断状态，并重置中断标志
                if (Thread.interrupted()) {
                	// 唤醒全部已经到达屏障处的线程
                	// 因为有线程被中断了，也就是说，永远不会有足够的线程到达屏障点
                	// 一场比赛10个运动员，结果比赛的时候有一个运动员没来，那么这一组就无法以原有规则进行比赛
                    breakBarrier();
                    // 当前线程抛出中断异常
                    // 某个运动员举手示意的时候，得到了有一个运动员没来，那么他也不用继续完成示意了
                    throw new InterruptedException();
                }
    
    			// 处理的线程索引
    			// 举手示意需要从一遍到另一边，中间不能跳过，不能重复
                int index = --count;
                // 线程索引为0
                // 全部运动员都已经举手示意
                if (index == 0) {  // tripped
                	// 是否需要唤醒已到达屏障点的线程
                	// 是否需要重新指定比赛(默认不需要重新制定，默认所有运动员都会来)
                    boolean ranAction = false;
                    try {
                    	// 获取屏障点操作
                    	// 裁判示意发令员
                        final Runnable command = barrierCommand;
                        if (command != null) // 操作不为空
                        	// 执行屏障点操作
                        	// 发令员发令
                            command.run();
                        // 不需要唤醒已经到达屏障点的线程
                        // 不需要重新制定比赛
                        ranAction = true;
                        // 唤醒全部线程，准备下一次屏障
                        // 运动员开跑，同时下一组运动员开始就位
                        nextGeneration();
                        // 返回等待成功
                        // 比赛成功
                        return 0;
                    } finally {
                    	// 是否需要唤醒已经到达屏障点的线程
                    	// 是否需要重新制定比赛
                        if (!ranAction)
                        	// 唤醒已经到达屏障点的线程
                        	// 有运动员没到场，那么就需要重新制定比赛
                            breakBarrier();
                    }
                }
    
                // loop until tripped, broken, interrupted, or timed out
                for (;;) {
                    try {
                    	// 是否有超时时间
                    	// 运动员准备工作是否有时间限制
                        if (!timed)	// 没有时间限制
                        	// 无限期等待运动员准备
                            trip.await();
                        else if (nanos > 0L) // 有时间限制
                        	// 运动员只能准备指定的时间，超时取消比赛
                            nanos = trip.awaitNanos(nanos);
                            
                    } catch (InterruptedException ie) { // 发生中断异常
                        if (g == generation && ! g.broken) { // 当前线程中断
                            breakBarrier(); // 唤醒所有已经到达屏障点的线程 
                            // 抛出中断异常
                            throw ie;
                        } else {
                            // We're about to finish waiting even if we had not
                            // been interrupted, so this interrupt is deemed to
                            // "belong" to subsequent execution.
                            // 不是当前线程中断
                            Thread.currentThread().interrupt(); // 中断当前线程
                        }
                    }
    
    				// 是否异常(这个值一直是false才是正常的，只要索引到0，自动全部线程到达屏障点)
    				// 所有的运动员示意完成
                    if (g.broken) // 错误开始比赛
                    	// 因为index还不到0，所以还不能发令
                        throw new BrokenBarrierException();
                    
                    // 如果屏障点和当前线程进入的屏障点不同，那么返回还未到达屏障点的线程个数
                    if (g != generation)
                        return index;
                    // 是否超时
                    // 超过运动员的准备时间
                    if (timed && nanos <= 0L) {
                    	// 唤醒全部线程
                        breakBarrier();
                        // 抛出超时异常
                        throw new TimeoutException();
                    }
                }
            } finally {
            	// 释放锁
                lock.unlock();
            }
        }
    

## 6\. 示例程序

下面由运动员短跑比赛开始做为一个小例子演示CyclicBarrier.
    
    
    public class MyCyclicBarrier {
    
        public static void main(String[] args) throws InterruptedException {
            int count = 10;
            Runnable barrierAction = () -> {
                System.out.println("裁判:开始比赛--------");
            };
            CyclicBarrier cyclicBarrier = new CyclicBarrier(count, barrierAction);
            Runnable runner = () -> {
                String name;
                System.out.println("运动员" + (name = Thread.currentThread().getName()) + ":准备完成");
                try {
                    Thread.sleep(300);//运动员需要300毫秒准备比赛
                    cyclicBarrier.await();
                    System.out.println("远动员" + name + "开跑");
                } catch (InterruptedException e) {
                    System.out.println("ie");
                } catch (BrokenBarrierException e) {
                    System.out.println("比赛取消！！！！");
                }
            };
            for (int i = 0; i < 5; i++) {
                List<Thread> threads = new ArrayList<>(count);
                for (int j = 0; j < count; j++) {
                    threads.add(new Thread(runner,j+""));
                }
                for (Thread thread: threads){
                    thread.start(); // 运动员就位
                }
                for (Thread thread: threads){
                    thread.join(); // 等待运动员开跑
                }
                cyclicBarrier.reset(); // 重置
                System.out.println(); // 分割
            }
        }
    
    }
    

执行结果
    
    
    运动员0:准备完成
    运动员8:准备完成
    运动员6:准备完成
    运动员4:准备完成
    运动员3:准备完成
    运动员9:准备完成
    运动员2:准备完成
    运动员1:准备完成
    运动员5:准备完成
    运动员7:准备完成
    裁判:开始比赛--------
    远动员1开跑
    远动员4开跑
    远动员0开跑
    远动员8开跑
    远动员3开跑
    远动员9开跑
    远动员7开跑
    远动员5开跑
    远动员6开跑
    远动员2开跑
    运动员9完成
    运动员2完成
    运动员6完成
    运动员1完成
    运动员7完成
    运动员0完成
    运动员3完成
    运动员4完成
    运动员8完成
    运动员5完成
    
    运动员0:准备完成
    运动员3:准备完成
    ......
    
    Process finished with exit code 0
    
    

## 7\. 总结

CyclicBarrier是一个很有用的同步工具。  
它可以让一批线程在特定的地方等待所有线程都执行到这里，然后在继续执行。  
内部实现也很简单，使用AQS的ConditonObject使得到达屏障点的线程等待，然后等所有线程都等待后，执行预定的操作，执行完预定的操作后，在唤醒等待的所有线程。  
这个和CountDownLatch在只使用一次的时候，功能相同。但是CountDownLatch只能使用一次，而CyclicBarrier能重复使用。