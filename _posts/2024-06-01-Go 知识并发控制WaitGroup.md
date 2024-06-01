---
layout: post
title: Go 知识并发控制WaitGroup
categories: [go]
description: Go 知识并发控制WaitGroup
keywords: golang, go, go 并发管理, WaitGroup, go 等待组
---

Go 知识并发控制WaitGroup

# 1. 认识 WaitGroup
WaitGroup 是Go 应用开发过程中经常使用的并发控制技术。  
WaitGroup 可理解为 Wait-Goroutine-Group,即等待一组 goroutine 结束。  
比如主协程需要等待启动的N个子协程完成，主协程在退出，那么可以认为子协程就是一组。  
比如主协程创建10个子协程，子协程睡眠1秒，然后子协程退出，主协程等待子协程退出后再退出:
```Go
func TestWaitGroup(t *testing.T) {
	sum := 10
	// 创建一个 waitGroup
	wg := sync.WaitGroup{}
	// 设置组内需要等待的数量是 10
	wg.Add(sum)
	for i := 0; i < sum; i++ {
		ti := i
		go func() {
			name := fmt.Sprintf("goroutine %d", ti)
			fmt.Println(name, "start")
			time.Sleep(3 * time.Second)
			fmt.Println(name, "end")
			// 每执行完一个，就将数量减少1
			wg.Done()
		}()
	}
	// 阻塞等待数量减少到0
	wg.Wait()
	fmt.Println("all goroutine done")
}
```
执行如下：  
![img.png](/images/posts/2024-06-01-Go%20知识并发控制WaitGroup/img.png)


在上面程序中wg内部维护了一个计数器：
- 启动goroutine前通过Add(sum)方法将计数器设置为待启动的goroutine的个数
- 启动goroutine后，使用Wait方法阻塞自己，等待计数器变为0
- 每个goroutine执行结束后通过Done方法将计数器减1
- 计数器变为0后，阻塞的goroutine呗唤醒

WaitGroup也可以嵌套调用，以实现更复杂的并发管理逻辑，不过复杂就意味着难实现。
# 2. 基本原理
## 2.1 信号量
信号量是UNIX系统提供的一种保护共享资源的机制，用于防止多个线程同时访问某个资源。  
信号量可简单理解为一个数值：
- 当信号量 > 0 时，表示资源可用，获取信号量时系统自动将信号量减1
- 当信号量 == 0 时，表示资源不可用，获取信号量时，当前线程会进入睡眠，当信号量为正时被唤醒。

## 2.2 数据结构
在源码包`src/sync/waitgroup.go`定义了数据结构：  
![img_1.png](/images/posts/2024-06-01-Go%20知识并发控制WaitGroup/img_1.png)


上面的 noCopy 是内部结构，防止拷贝 WaitGroup 进行使用  
![img_2.png](/images/posts/2024-06-01-Go%20知识并发控制WaitGroup/img_2.png)


而 state 分为两部分，高32位是当前还未执行结束的 goroutine 计数器 counter ，低32位是等待 goroutine-group 结束的 goroutine 数量 waiter count，即有多少个等候者。  
sema 则是信号量。  
同时 WaitGroup 对外暴露了三个方法：  
Add(delta int): 将 delta 值加到 state 的高 32 位，也就是未执行结束的 goroutine 数量加 x  
Wait: waiter 递增1，并阻塞等待信号 sema ，表示等候者+1
Done: counter 递减1，按照 waiter 数值释放相应次数信号量
## 2.3 Add
Add方法做了两件事，第一是把传入的值累加到 counter 中，因为传入的值可以为负值，也就是说 counter 有可能变成 0 或者负值。  
第二就是当 counter 值变为 0 时，根据 waiter count 的值释放等量的信号量，把等待的 goroutine 全部唤醒。 如果 counter 值变为负值，则触发 panic 。
```Go
// Add将增量 (可能为负) 添加到WaitGroup计数器。
// 如果计数器变为零，则释放在等待时阻塞的所有goroutines。
// 如果计数器为负，则添加panics。
//
// 请注意，当计数器为零时发生的具有正增量的调用
// 必须在等待之前发生。具有负delta的呼叫，或具有
// 当计数器大于零时开始的正增量，可能会发生
// 任何时候。
// 通常，这意味着对Add的调用应在语句之前执行
// 创建要等待的goroutine或其他事件。
// 如果一个WaitGroup被重用等待几个独立的事件集，
// 新的添加调用必须在所有以前的等待调用返回后发生。
// 请参见WaitGroup示例。
func (wg *WaitGroup) Add(delta int) {
    // 将传入的值，加到 state 的 高 32 位
    // 这里也就是 counter 的值，当前还未结束的 goroutine 的值，也就是执行 goroutine 的值
	state := wg.state.Add(uint64(delta) << 32)
	// counter
	v := int32(state >> 32)
	// waiter count 等待 goroutine 结束的 值
	// waiter count 是由 Wait 方法 递增的，所以  waiter count 一定是 大于等于0
	w := uint32(state)
	// 如果 counter 小于 0 触发 panic 
	if v < 0 {
		panic("sync: negative WaitGroup counter")
	}
	// 如果 waiter count 大于 0 表示已经有 goroutine 在等待了，而且 counter 等待执行的数量和设置的相同，
	// 此时在 调用 Add ，表示 Add 与 Wait 并发调用了 
	if w != 0 && delta > 0 && v == int32(delta) {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
	// 如果 counter 大于 0 ，但是 waiter count 等于 0 ，表示 有 counter 个 goroutine 待执行，
	// 但是没有 goroutine 阻塞等待，所以直接设置 counter 就结束了
	if v > 0 || w == 0 {
		return
	}
	// 当waiters> 0时，此goroutine已将counter设置为0。
	// 现在不能有状态的并发突变:
	//-Adds不能与Wait同时发生，
	//-如果看到counter == 0，Wait不会增加waiters。
	// 仍然做一个便宜的健全性检查来检测WaitGroup滥用。
	if wg.state.Load() != state {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
	// Reset waiters count to 0.
	// 到了这里 counter 一定等于 0 ， 而且 waiter count 大于 0
	// 将 counter 设置为 0 
	wg.state.Store(0)
	// 根据 waiter count 的数量，释放信号量
	for ; w != 0; w-- {
		runtime_Semrelease(&wg.sema, false, 0)
	}
}
```
如果按照串行的执行方式，一般都是先 Add 在 Wait ，然后在 Done ，但是在并发的环境中，执行的先后顺序存在不确定性，所以，在Add里面会对各种情况做判断。
## 2.4 Wait
Wait 方法也是两个操作，第一个是累加 waiter count ，第二个是阻塞并等待信号量。
```Go
// 等待块，直到WaitGroup计数器为零。
func (wg *WaitGroup) Wait() {
    // 死循环，死循环的目的是 做 cas 的时候，如果失败，不断重试
    // 这也是 Java 中乐观锁的实现，死循环 cas
	for {
	    // 获取 counter 和 waiter count 
		state := wg.state.Load()
		// counter 
		v := int32(state >> 32)
		// waiter count
		w := uint32(state)
		// 如果 counter 等于 0 ，表示等待执行的 goroutine 已经全部都执行完了，那么就无需等待了
		if v == 0 {
			// Counter is 0, no need to wait.
			return
		}
		// 增加 waiter count 计数。
		// 等待 信号的数量加1
		// 使用 cas 累加，如果累加失败，那么就 for 循环重试
		if wg.state.CompareAndSwap(state, state+1) {
		    // 阻塞并等待信号量
			runtime_Semacquire(&wg.sema)
			// 如果 得到了信号量，但是 waiter count 和 counter 都不为0 那么失败
			// 这里可以认为是使用成本比较低的方式做了健壮性检查
			if wg.state.Load() != 0 {
				panic("sync: WaitGroup is reused before previous Wait has returned")
			}
			// 跳出循环
			return
		}
	}
}
```
使用死循环的CAS保证并发的时候，也能正确的累加 waiter count.
## 2.5 Done
Done 方法只做一件事，就是把 counter 减 1，但是需要注意的是， counter 减 1 的操作是通过 Add 方法实现的。  
所以 Done 等价于  Add(-1)
![img_3.png](/images/posts/2024-06-01-Go%20知识并发控制WaitGroup/img_3.png)


在回过头看看 Add 的逻辑：
```Go
// Add将增量 (可能为负) 添加到WaitGroup计数器。
// 如果计数器变为零，则释放在等待时阻塞的所有goroutines。
// 如果计数器为负，则添加panics。
//
// 请注意，当计数器为零时发生的具有正增量的调用
// 必须在等待之前发生。具有负delta的呼叫，或具有
// 当计数器大于零时开始的正增量，可能会发生
// 任何时候。
// 通常，这意味着对Add的调用应在语句之前执行
// 创建要等待的goroutine或其他事件。
// 如果一个WaitGroup被重用等待几个独立的事件集，
// 新的添加调用必须在所有以前的等待调用返回后发生。
// 请参见WaitGroup示例。
func (wg *WaitGroup) Add(delta int) {
    // 将传入的值，加到 state 的 高 32 位
    // 这里也就是 counter 的值，当前还未结束的 goroutine 的值，也就是执行 goroutine 的值
	state := wg.state.Add(uint64(delta) << 32)
	// counter
	v := int32(state >> 32)
	// waiter count 等待 goroutine 结束的 值
	// waiter count 是由 Wait 方法 递增的，所以  waiter count 一定是 大于等于0
	w := uint32(state)
	// 如果 counter 小于 0 触发 panic 
	if v < 0 {
		panic("sync: negative WaitGroup counter")
	}
	// 如果 waiter count 大于 0 表示已经有 goroutine 在等待了，而且 counter 等待执行的数量和设置的相同，
	// 此时在 调用 Add ，表示 Add 与 Wait 并发调用了 
	if w != 0 && delta > 0 && v == int32(delta) {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
	// 如果 counter 大于 0 ，但是 waiter count 等于 0 ，表示 有 counter 个 goroutine 待执行，
	// 但是没有 goroutine 阻塞等待，所以直接设置 counter 就结束了
	// 在 Done 递减的时候，如果 counter 不等于 0 ，那么就结束了, 如果 counter > 0 同时 waiter count 也大于0呢
	if v > 0 || w == 0 {
		return
	}
	// 当waiters> 0时，此goroutine已将counter设置为0。
	// 现在不能有状态的并发突变:
	//-Adds不能与Wait同时发生，
	//-如果看到counter == 0，Wait不会增加waiters。
	// 仍然做一个便宜的健全性检查来检测WaitGroup滥用。
	if wg.state.Load() != state {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
	// Reset waiters count to 0.
	// 到了这里 counter 一定等于 0 ， 而且 waiter count 大于 0
	// 将 counter 设置为 0 
	wg.state.Store(0)
	// 根据 waiter count 的数量，释放信号量
	for ; w != 0; w-- {
		runtime_Semrelease(&wg.sema, false, 0)
	}
}
```
可以这么简单的理解，初始化 x 个 counter 等待数量，需要等待的 goroutine 通过 Wait 方法阻塞等待信号量，并且将 waiter count 加1，
执行 Done 的 goroutine 将 counter 数量减1，当 counter 减少到 0  的时候，释放 waiter count 个信号量，唤醒 goroutine 继续执行。
# 3. 小例子
## 3.1 主协程等待子协程执行完成
```Go
func TestWaitGroup(t *testing.T) {
	sum := 10
	wg := sync.WaitGroup{}
	wg.Add(sum)
	for i := 0; i < sum; i++ {
		ti := i
		go func() {
			name := fmt.Sprintf("goroutine %d", ti)
			fmt.Println(name, "start")
			time.Sleep(3 * time.Second)
			fmt.Println(name, "end")
			wg.Done()
		}()
	}
	wg.Wait()
	fmt.Println("all goroutine done")
}
```
主协程等待10个子协程执行完成后，继续执行。
## 3.2 子协程等待主协程信号
```Go
func TestWaitGroup(t *testing.T) {
	sum := 10
	wg := sync.WaitGroup{}
	wg.Add(1)
	for i := 0; i < sum; i++ {
		go func() {
			fmt.Println("goroutine wait")
			// 子协程等待资源信号
			wg.Wait()
			fmt.Println("goroutine done")
		}()
	}
	time.Sleep(time.Second)
	// 主协程资源准备好了，发出信号，让子协程继续处理
	wg.Done()
	time.Sleep(time.Second)
}
```
这种使用方式不多，考虑如下场景：  
主协程查询到100条数据，子协程需要先准备一下，执行一段逻辑，然后在执行分配的数据。  
如果是主协程数据准备好了，在启动子协程，处理数据，那么如果处理数据之前的操作比较耗时，那么性能就比较差。  
这个时候就可以使用上面这种用法，主协程先创建子协程，子协程先执行逻辑，到达从主协程获取数据前，使用 wait 进行等待。  
主协程数据准备好了，在使用 done 通知 子协程继续。
## 3.3 GetFirst
```Go
func TestWaitGrou(t *testing.T) {
	sum := 10
	wg := sync.WaitGroup{}
	wg.Add(sum)
	for i := 0; i < sum; i++ {
		idx := i
		go func() {
			fmt.Printf("goroutine %d start\n", idx)
			if idx == 6 {
				// 因为 Done 就是 调用  Add(-1) , 所以如果某个 goroutine 执行成功，那么可以使用 Add(-sum) 一次性解除 主协程的等待
				// 其他子协程的执行实际上就没有意义了，因为只需要执行成功的任意一个就行了
				time.Sleep(time.Second)
				fmt.Printf("goroutine %d success\n", idx)
				wg.Add(0 - sum)
			}
			// 模拟超时
			time.Sleep(3 * time.Second)
			wg.Done()
			fmt.Println("goroutine done")
		}()
	}
	fmt.Println("wait some one ok")
	wg.Wait()
	fmt.Println("master done")
}
```
这种用法可以用在查询中：  
比如知道一个资源id，但是不知道这个资源id是哪个部门的，假设每个部门一个数据库，那么就需要将所有的部门都遍历一遍，才能知道是哪个部门的。  
如果是串行，那么当某个部门的数据库，查询这个资源id有数据返回，那么查询结束，否则就继续查询，直到全部查询完成。  
如果是并行，那么当某个子协程返回数据，那么就可以结束了，因为隐含了一个逻辑，就是当资源在A部门的时候，就不可能在其他部门，因为一个资源不能多次分配。  
但是使用这种方法的时候需要注意，其他子协程在 done 的时候，因为 counter 会变成负数，导致 panic 。
# 4. 总结
WaitGroup 通常用于等待一组“工作协程”结束的场景，其内部维护两个计数器，可以称为“工作协程”计数器和“等待协程”计数器。WaitGroup 对外提供的三个方法分工明确：
- Add(delta int) 方法用于增加“工作协程”计数，通常在启动新的“工作协程”之前调用。
- Done 方法用于减少“工作协程”计数，每次调用递减1，通常在“工作协程”内部且在临近返回之前调用。
- Wait 方法用于增加“等待协程”计数，通常在所有“工作协程”全部启动之后调用。

Done 方法除了负责递减“工作协程”计数，还会在“工作协程”计数变为0时检查“等待协程”计数器，并把“等待协程”唤醒。需要注意，Done 方法递减“工作协程”计数后，如果“工作协程”
计数变为负数，则会触发panic，这就要求 Add 方法调用要早于 Done 方法。  
此外，通过 Add 方法累加的“工作协程”计数要与实际需要等待的“工作协程”数量一致，否则 实际“工作协程”调用 Done 递减“工作协程”计数，无法使计数等于0，那么“等待协程”永远不会被唤醒，就发生了死锁。
Go运行时检测到死锁触发panic。

WaitGroup 如果遇到派生的 goroutine ，就需要将 WaitGroup 不断的传递，还要保证数量能对上，否则就会产生死锁，对于存在派生的，多层级的 goroutine 的时候，WaitGroup 的使用难度大大增加。
