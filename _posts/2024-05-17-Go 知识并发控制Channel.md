---
layout: post
title: Go 知识并发控制Channel
categories: [go]
description: Go 知识并发控制Channel
keywords: golang, go, go 并发管理, chan, channel
---

Go 知识并发控制Channel


channel 一般用于协程之间的通信，不过channel也可以用于并发控制。 比如主协程启动N个子协程，主协程等待所有子协程退出后再继续后续流程，这种并发控制的场景，channel也可以实现。
> channel 的用法： [Go 知识chan](https://jiayq.blog.csdn.net/article/details/135885482)

# 1. 无缓冲 chan 实现 并发控制
假设要在主协程中创建10个子协程，主协程等待子协程执行完成后主协程在退出。  
[Go 知识for-range](https://jiayq.blog.csdn.net/article/details/135886263)
```Go
func TestCc(t *testing.T) {
    // 定义子协程个数
	sum := 10
	// 创建10个 chan 用于通信
	chs := make([]chan struct{}, sum)
	for i := 0; i < sum; i++ {
	    // 创建 chan
		ch := make(chan struct{})
		chs[i] = ch
		// 因为在 子协程中用到了 chan 和 index ，所以使用临时变量拷贝一下， for - range 
		// 更多细节： https://jiayq.blog.csdn.net/article/details/135886263
		ti := i
		go func() {
		    // 子协程打印消息
			name := fmt.Sprintf("goroutine %d", ti)
			fmt.Println(name, "start")
			// 子协程睡眠模拟耗时逻辑
			time.Sleep(3 * time.Second)
			fmt.Println(name, "end")
			// 发送子协程完成和退出信号
			ch <- struct{}{}
			// 关闭资源
			close(ch)
		}()
	}
	// 在主协程中读取子协程的退出信号，阻塞从 chan 中读取数据
	for _, ch := range chs {
		<-ch
	}
	fmt.Println("all goroutine done")
}
```
执行结果如下：  
![img.png](/images/posts/2024-05-17-Go%20知识并发控制Channel/img.png)


和预期一致。
# 2. 有缓冲 chan 实现 并发控制
还是和1相同的逻辑，区别在于上面是一个子协程就有1个 chan ，假设有1w的子协程，就需要创建1w的chan。  
而有缓冲的 chan 则只需要一个即可:  
[Go-知识协程](https://jiayq.blog.csdn.net/article/details/135886263)
```Go
func TestCc1(t *testing.T) {
    // 主协程中创建10个子协程
	sum := 10
	// 创建 带有 10 个 缓冲空间的 chan
	// 缓冲空间的大小和需要管理的子协程的大小应该相同，否则子协程如果执行比较快，就会因为缓冲满而阻塞
	// 其实子协程因为缓冲满阻塞，也没有关系，依赖 go 的高效 调度，当出现因为 chan 阻塞时，长时间无法得到资源，
	// 就会让 绑定 G 的 P 脱离 M , 等到 chan 有空间了在继续执行
	// 更多细节： https://jiayq.blog.csdn.net/article/details/137087910
 	ch := make(chan struct{}, sum)
	for i := 0; i < sum; i++ {
	    // 闭包使用
		ti := i
		go func() {
			name := fmt.Sprintf("goroutine %d", ti)
			fmt.Println(name, "start")
			time.Sleep(3 * time.Second)
			fmt.Println(name, "end")
			ch <- struct{}{}
		}()
	}
	for i := 0; i < sum; i++ {
		<-ch
	}
	close(ch)
	fmt.Println("all goroutine done")
}
```
执行结果  
![img_1.png](/images/posts/2024-05-17-Go%20知识并发控制Channel/img_1.png)


上面的程序即使不带缓冲，也是能实现并发控制的：  
![img_2.png](/images/posts/2024-05-17-Go%20知识并发控制Channel/img_2.png)


# 3. 总结
channel 作为协程间通信的类型，用来在协程间传递信息是基本能力，而通过灵活的使用 channel ，可以实现各种各样复杂的并发控制。  
