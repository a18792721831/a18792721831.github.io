---
layout: post
title: Go 知识并发控制Context
categories: [go]
description: Go 知识并发控制Context
keywords: golang, go, go 并发管理, context, go 上下文
---

Go 知识并发控制Context

# 1. 介绍
Go 语言的 context 是应用开发常用的并发控制技术，它与 WaitGroup 最大的不同点是 context
对于派生 goroutine 有更强的控制力，可以控制多级的 goroutine 。  
context 翻译成中文是 上下文  ，即可以控制一组呈树状结构的 goroutine ，每个 goroutine 拥有相同的上下文。  
![img.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img.png)


上图中由于 goroutine 派生出子 goroutine ，而子 goroutine 又继续派生出新的 goroutine ,
这种情况下使用 WaitGroup 就不太容易，因为子 goroutine 的个数不太容易确定，而使用 context 就很容易实现。
# 2. 实现原理
context 实际上只定义了接口，凡是实现该接口的类都可以称为一种 context ,官方包中实现了几个常用的 context ，分别用于不同的场景。
## 2.1 接口定义
源码包中的 `src/context/context.go`定义了接口：  
![img_1.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_1.png)


基础的 context 接口只定义了4个方法。
## 2.2 Deadline()
`Deadline() (deadline time.Time, ok bool)`  
该方法返回一个 deadline 和标识是否已设置 deadline 的 bool 值，如果没有设置 deadline ,
则 ok == false ，此时 deadline 是一个初始值的 time.Time 值。
## 2.3 Done()
`Done() <-chan struct{}`  
该方法返回一个用于探测 Context 是否取消的 channel ， 当 Context 取消时会自动将该 channel 关闭。  
对于不支持取消的 Context ，该方法可能会返回 nil。
## 2.4 Err()
`Err() error`  
该方法描述 context 关闭的原因。关闭原因由 context 实现控制，不需要用户设置。比如 Deadline context,
关闭原因可能是因为 deadline ,也可能是提前被主动关闭，那么关闭原因就会不同：
- 因 deadline 关闭：context deadline exceeded.
- 因主动关闭： context canceled.

当 context 关闭后，Err()返回 context 的关闭原因；当 context 还未关闭时，Err() 返回nil.

## 2.5 Value()
`Value(key any) any`  
有一种 context ，它不是用于控制呈树状分部的 goroutine ，而是用于在树状分部的 goroutine 间传递信息。  
Value() 方法就是用于此种类型的 context ，该方法根据 key 值查询 map 中的 value 。
# 3. 空 context
context 包中定义了一个空的 context ，名为 emptyCtx ，用于 context 的根节点，空 context 只是简单地实现了 Context，
本身也不包含任何值，仅用于其他 context 的父节点
![img_2.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_2.png)



context 包中还定义了一个公用的 emptyCtx 全局变量，名为 background ，可以使用
`context.Background()`获取。

![img_3.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_3.png)


context 包提供了四个方法创建不同类型的 context ，使用这四个方法时，如果没有父 context，
则都需要传入 background ，即将 background 作为其父节点：
- WithCancel()
- WithDeadline()
- WithTimeout()
- WithValue()

context 包中实现了 Context 接口的 struct，除了 emptyCtx ，还有 cancelCtx,timerCtx和valueCtx三种，
基于这三种 context 实例，实现了上述四种额理性的 context.  
context 包中各 context 类型之间的关系：

![img_4.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_4.png)


# 4. cancelCtx
源码包中的 `src/context/context.go:cancelCtx`定义了该类型 context:  
![img_5.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_5.png)


children 中记录了由此context派生的所有child , 此 context 被 cancel 时，会把其中所有的 child 都 cancel 。  
cancelCtx 与 deadline 和 value 无关，所以只需要实现 Done() 和 Err() 接口即可。
## 4.1 Done()
按照 Context 的定义，Done()方法只需要返回一个 channel 即可，对于 cancelCtx 来说
只需要返回成员变量 done 即可。
```Go
func (c *cancelCtx) Done() <-chan struct{} {
    // 获取 成员变量 done , 在老版本中，done 直接是 类型 chan struct{} 类型，不安全，新版本是 atomic.Value
    d := c.done.Load()
    // 如果 不为 nil 表示已经被初始化过了，直接返回
	if d != nil {
		return d.(chan struct{})
	}
	// 如果还未初始化，那么加锁
	c.mu.Lock()
	defer c.mu.Unlock()
	// 加锁重新获取数据，防止并发导致的泄露，加了锁之后再重新获取数据，一定是最新的，加锁之前获取的数据，不一定是最新的
	d = c.done.Load()
	if d == nil {
		// 初始化 done 变量 的 channel 
		d = make(chan struct{})
		c.done.Store(d)
	}
	return d.(chan struct{})
}
```
在老版本中 Done 每次都必须加锁，然后获取值，解锁，返回。  
![img_6.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_6.png)


在 Done 方法中，只有第一次需要初始化，后面都是直接返回即可，所以只有第一次的加锁是有效的，后面加锁都是无效的(假设 channel 不会被修改)
因此在新版本中，done 变量使用 atomic.Value 存储，并发安全，在Done 里面也是乐观获取，如果获取得到为空，那么在加锁初始化。
> 既然只有第一次才初始化，为何不在 init 方法中初始化呢？

在 cancelCtx 中 ，done 变量就是 channel ，channel 会经历 nil -> make -> close 三个阶段。  
如果直接 close 值为 nil 的 channel ，会引发 panic 。
## 4.2 Err()
按照 Context 定义，Err() 需要返回一个 error 告知 context 被关闭的原因。 对于 cancelCtx 来说
返回成员变量 err 即可。
```Go
func (c *cancelCtx) Err() error {
	c.mu.Lock()
	err := c.err
	c.mu.Unlock()
	return err
}
```
Err()也是加锁然后获取值，解锁。
## 4.3 cancel()
在 Context 接口定义中并没有 cancel 方法，所以 cancel方法是在接口canceler中定义的。  
![img_7.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_7.png)


cancelCtx 和 timerCtx 实现了这个接口。  
cancelCtx 接口和 Context 接口都定义了 Done() 方法，而且完全相同。  
cancel 方法是 cancelCtx 的关键方法，作用是关闭自己和其后代，其后代存储在 cancelCtx.children 的map中，
其实可以理解是 Set 中，因为map的value没有任何意义，数据本身是存储在key中的。
```Go
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
	// 必须给出 cancel 的原因
	if err == nil {
		panic("context: internal error: missing cancel error")
	}
	// 加锁
	c.mu.Lock()
	// 读取 err 
	if c.err != nil {
		c.mu.Unlock()
		// 已经 cancel 了
		return // already canceled
	}
	// 否则将 error 设置到 成员变量中
	c.err = err
	// 读取 done 成员变量
	d, _ := c.done.Load().(chan struct{})
	// 如果 done 成员变量为空，表示还未调用 Done() 就调用 cancel，那么需要先初始化 done 成员变量
	// 因为 close channel 时，channel 不能nil 否则触发 panic
	if d == nil {
	    // 这里的 closedchan 是一个全局变量，并且在 init 中就 close 了
		c.done.Store(closedchan)
	} else {
	    // 如果是已经初始化的 channel ，那么调用 close 
		close(d)
	}
	// 将 cannel 扩散到全部的后代中
	// 需要注意的是，这里是递归扩散，如果 children 也有 children ，那么孙子也会执行 cannel
	for child := range c.children {
		// 注意: 获取孩子的锁，同时持有父母的锁。
		// 移出 后代，不需要移出自己本身
		child.cancel(false, err)
	}
	// 同时将 后代移出 map
	c.children = nil
	// 解锁
	c.mu.Unlock()
    // 如果需要将自己从 parent 中删除
	if removeFromParent {
		removeChild(c.Context, c)
	}
}
```
这是 `closedchan`的源码，在初始化的时候就 close 了。
![img_8.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_8.png)


对于 `removeChild`的实现，这里先留一下，要不不太好理解。
## 4.4 WithCancel
WithCancel 方法做了三件事：
- 初始化一个 cancelCtx 实例
- 将 cancelCTx 实例添加到其父节点的 children 中(如果父节点也可以被`cancel`)
- 返回 cancelCtx 实例和 cancel() 方法

```Go
// WithCancel返回带有新的Done通道的父副本。返回的
// 当调用返回的cancel函数时，上下文的Done通道关闭
// 或当父上下文的Done通道关闭时，以先发生者为准。
//
// 取消此上下文会释放与其关联的资源，因此代码应该
// 在此上下文中运行的操作完成后，立即调用cancel。
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	c := withCancel(parent)
	return c, func() { c.cancel(true, Canceled, nil) }
}
```
这里引入了 `CancelFunc` 类型
```Go
// CancelFunc告诉操作放弃它的工作。
// CancelFunc不等待工作停止。
// 一个CancelFunc可以被多个goroutines同时调用.
// 第一次调用后，对CancelFunc的后续调用不执行任何操作。
type CancelFunc func()
```
前面看到 `cancel` 方法是包内可见，但是 cancel 方法又需要包外的调用者触发，而且在触发的时候，入参又有一定的要求。  
基于这种需要，这里使用 func 类型包装，在获取 func 的时候，将入参做个处理，最后将 入参 经过处理的 cancel 用 func 变量的方式
返回到调用者，至于触发 cancel 的时机，由调用者决定。 换句话来说，调用者只能决定 cancel 的时机和部分受限的参数。  
这种使用方式很巧妙，值得好好学学。  
首先看 `withCancel(parent)`调用
```Go
func withCancel(parent Context) *cancelCtx {
    // 父节点不能是 nil
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	// 创建当前节点
	c := &cancelCtx{}
	// 将当前节点加入到树中，也就是加入到 父节点 的 children map中
	c.propagateCancel(parent, c)
	return c
}
```
OK，接着看下 `propagateCancel`方法
```Go
func (c *cancelCtx) propagateCancel(parent Context, child canceler) {
    // 将 父节点设置为 parent
	c.Context = parent
	// 获取父节点的 channel ，因为整颗树使用一个 channel 
	done := parent.Done()
	// 如果父节点的 channel 是空的，说明 父节点永远不会 cancel，也就是根节点(emptyCtx?)
	// 那也就是说 当前 节点是 一代节点，因为父节点 可能是 emptyCtx
	if done == nil {
		return // parent is never canceled
	}
	// 使用 select 阻塞获取 channel 的值，但是因为有 default
	// 所以这里可以理解为 非阻塞获取 channel 值，如果没有获取到，什么也不做
	select {
	// 如果 channel 有值，表示 父节点执行了 cancel，那么子节点也需要执行 cancel
	case <-done:
		// parent is already canceled
		// 这里需要注意，扩散的时候，都是不把自己从父节点列表中删除，也就是不把自己从树里面移出
		child.cancel(false, parent.Err(), Cause(parent))
		return
	default:
	}
    // 如果父节点没有 cancel ，那就就把当前节点加入父节点的 children
    // 如果父节点已经 cancel ，那么当前节点也 cancel ，并且 不加入父节点的 children
    // parentCancelCtx 是查找祖辈节点是否是 cancelCtx (包括包装的cancelCtx)
    // 祖辈节点的查找通过 Value实现的
	if p, ok := parentCancelCtx(parent); ok {
		// parent is a *cancelCtx, or derives from one.
		p.mu.Lock()
		// 判断 祖辈节点是否 cancel
		if p.err != nil {
			// parent has already been canceled
			// 祖辈节点存在 cancel，那么 当前节点也 cancel
			// 当前节点理论上还未加入到 children中，执行 cancel 是为了保证 cancel 节点的数据完整性
			child.cancel(false, p.err, p.cause)
		} else {
		    // 如果祖辈节点没有 cancel ，而且祖辈节点还没有 children
		    // 那么初始化
			if p.children == nil {
				p.children = make(map[canceler]struct{})
			}
			// 将当前节点加入到 children中
			// 如果不加入，那么当 祖辈节点 cancel 的时候，就无法通知到当前 节点
			p.children[child] = struct{}{}
		}
		p.mu.Unlock()
		return
	}
	// 如果 父节点 不是 cancelCtx ,而是实现了 afterFuncer 接口的 stopCtx
	// stopCtx 需要实现 AfterFunc(func()) func() bool 方法，获取 cancel 后需要执行的 func
	if a, ok := parent.(afterFuncer); ok {
		// parent implements an AfterFunc method.
		c.mu.Lock()
		// 如果 父节点是 stopCtx，那么当前节点也 包装成 stopCtx
		// 但是需要注意的是，当前节点本质上还是 cancelCtx
		stop := a.AfterFunc(func() {
			child.cancel(false, parent.Err(), Cause(parent))
		})
		// 将当前节点包装为 stopCtx
		c.Context = stopCtx{
			Context: parent,
			stop:    stop,
		}
		c.mu.Unlock()
		return
	}
    // 表示整个树中有多少个节点，简单记录，包内可见，用于 test 
	goroutines.Add(1)
	// 如果从当前节点的父节点不是 cancelCtx,那么启动一个协程
	// 协程的作用是等待 父节点 结束，然后结束当前节点。
	// context 一个很重要的作用就是可扩散
	// 如果 父节点不是 cancelCtx(stopCtx 可以认为是包装的 cancelCtx)
	// 但是有 channel ，那么就需要扩散传播
	// 那么就需要格外的协程去 canncel 
	// 因为当前节点是 cancelCtx，所以在当前这个路径下继续增加节点，就不会在创建协程了。
	go func() {
		select {
		case <-parent.Done():
			child.cancel(false, parent.Err(), Cause(parent))
		case <-child.Done():
		}
	}()
}
```
在上面的实现中，`parentCancelCtx`也是一个关键方法，看看如何找到祖辈的 cancelCtx
```Go
func parentCancelCtx(parent Context) (*cancelCtx, bool) {
    // 获取父节点的 channel
	done := parent.Done()
	// done 为空表示父节点不传播扩散或是到了根节点
	// done 为 closedchan 表示父节点是 cancelCtx，但是很可惜，父节点 cancelCtx 已经 cancel 了
	if done == closedchan || done == nil {
	    // 父节点已经调用了 cancel ，那么当前节点就等待 父节点 cancel ,然后当前节点 cancel
	    // 逐层 扩散？
		return nil, false
	}
	// 找祖辈节点
	// 因为任何 Context 都需要实现 Value 方法
	// cancelCtx 的 Value 方法就是将当前节点放入 指定的 key 
	// 因为在 cancelCtx 中，每个 节点的 key 都是 cancelCtxKey，所以这里就像是循环查找，直到根节点
	p, ok := parent.Value(&cancelCtxKey).(*cancelCtx)
	// 如果 父节点不是 cancelCtx ，那么返回未找到
	if !ok {
		return nil, false
	}
	// 获取 cancelCtx 的 done 变量
	pdone, _ := p.done.Load().(chan struct{})
	// 二次确认，Done() 返回的就是 done 变量
	if pdone != done {
		return nil, false
	}
	// 返回找到了
	return p, true
}
```
接着看一下 Value方法：
```Go
func (c *cancelCtx) Value(key interface{}) interface{} {
    // 假设 r -> a -> b -> c
    // a,b,c 三个节点的 key 都是 同样的值
    // 所以这里就相当于是从当前节点一直向上查找，直到根节点
	if key == &cancelCtxKey {
		return c
	}
	// 父节点的 Value 方法
	return c.Context.Value(key)
}
```
简单说一下 WithCancel 的逻辑：
- 初始化一个cancelCtx实例
- 将cancelCtx 实例添加到父节点的 children中(如果父节点也可以被 cancel )
- 返回cancelCtx实例和 cancel方法

在添加和查找父节点时，逻辑如下：
- 如果父节点支持 cancel ，那么父节点有 children 变量，那么将当前节点也加入到 children中
- 如果父节点不支持 cancel ，那么就说明父节点没有 children 变量，就不能将当前节点加入到 children 中
- 如果该路径中没有 children 变量，那么就新增一个协程，等待父节点(cancelCtx)结束

之前存疑的`removeChild`就是将触发节点从整个路径中移除。那么被移除节点的子节点自然就从整个树上面移除了。  
其实严格来说，不一定是树形结构，也有可能是图形结构。
## 4.5 例子
单链传播
```Go
func TestCtxEmpty(t *testing.T) {
	ctx := context.Background()
	ctxr, cancelr := context.WithCancel(ctx)
	// root -> n1 -> n2 -> c3 -> c4 -> n5 -> c6
	go func() {
		ctx1, _ := context.WithCancel(ctxr)
		fmt.Println("ctx1 <- ctxr")
		go func() {
			ctx2, _ := context.WithCancel(ctx1)
			fmt.Println("ctx2 <- ctx1")
			go func() {
				fmt.Println("ctx3 <- ctx2")
				select {
				case <-ctx2.Done():
					fmt.Println("ctx3 <- ctx2 done")
					return
				}
			}()
			select {
			case <-ctx1.Done():
				fmt.Println("ctx2 <- ctx1 done")
				return
			}
		}()
		select {
		case <-ctxr.Done():
			fmt.Println("ctx1 <- ctxr done")
			return
		}
	}()
	time.Sleep(time.Second)
	cancelr()
	time.Sleep(time.Second)
}
```
执行如下：
![img_9.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_9.png)


树形传播
```Go
func TestCtxEmpty(t *testing.T) {
	ctx := context.Background()
	// r -> 1a -> 2a
	//         -> 2b
	//   -> 1b
	ctxr, cancelr := context.WithCancel(ctx)
	go func() {
		fmt.Println("ctx1a <- ctxr")
		ctx1, _ := context.WithCancel(ctxr)
		go func() {
			fmt.Println("ctx2a <- ctx1")
			select {
			case <-ctx1.Done():
				fmt.Println("ctx2a <- ctx1 done")
				return
			}
		}()
		go func() {
			fmt.Println("ctx2b <- ctx1")
			select {
			case <-ctx1.Done():
				fmt.Println("ctx2b <- ctx1 done")
				return
			}
		}()
		select {
		case <-ctxr.Done():
			fmt.Println("ctx1a <- ctxr done")
			return
		}
	}()
	go func() {
		fmt.Println("ctx1b <- ctxr")
		select {
		case <-ctxr.Done():
			fmt.Println("ctx1b <- ctxr done")
			return
		}
	}()
	time.Sleep(time.Second)
	cancelr()
	time.Sleep(time.Second)
}
```
执行结果  
![img_10.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_10.png)


## 4.6 总结
cancelCtx 实现了 Context 和 canceler 接口。  
Context 接口定义了 Done, Value, Deadline, Err 方法  
canceler 接口定义了 Done, cancel 方法

emptyCtx 是一个空值实现 Context 的 结构体，主要用作根节点。

cancelCtx 的结构体组合了 Context 接口，相当于是隐含成员变量，作为父节点。  
cancelCtx 的成员变量 done 存储了 Context 接口中 Done 方法返回的 channel 作为传播变量。  
cancelCtx 通过 WithCancel 方法创建，通过传入 Context 类型的 parent 作为父节点。  
cancelCtx 的成员变量 children 存储了子 canceler 节点，当发生 cancel 的时候，会进行传播调用。  
cancelCtx 通过 WithCancel 进行组织，可以是链表，可以是树，可以是图，但是关系越复杂，越容易出现环形死锁。  
cancelCtx 通过 WithCancel 方法创建的时候，返回的第二个值是触发cancel的func，调用会触发该路径下全部子节点的 cancel。  
cancelCtx 的cancel方法通过func值传递给调用者，由调用者确定时机，这是一种很巧妙的将内部方法交由外部调用的方式，值得学习。  
cancelCtx 实现了Value 方法，通过将全部的 cancelCtx 的 key 设置成相同的，以此实现从当前节点查找整个路径。
# 5. timerCtx
源码包中的`src/context/context.go:timerCtx`定义了timerCtx:
```Go
// timerCtx携带一个计时器和一个截止时间。它将cancelCtx嵌入到
// 实现Done和Err。它通过停止其计时器来实现取消，然后
// 委托t o cancelCtx.ca ncel。
type timerCtx struct {
	cancelCtx
	timer *time.Timer // Under cancelCtx.mu.

	deadline time.Time
}
```
timerCtx 在cancelCtx 的基础上增加了 deadline ，用于标识自动 cancel 的最终时间，
而timer就是一个触发自动cancel 的定时器。  
并由此衍生出来WithDeadline() 和 WithTimeout()。   
实现上这两种类型原理相同，但是在使用的时候存在区别。
- deadline: 指定最后期限，比如 context 将在指定时间结束
- Timeout: 指定最大存活时间，比如 context 将在 2s 后结束

## 5.1 Deadline
```Go
func (c *timerCtx) Deadline() (deadline time.Time, ok bool) {
	return c.deadline, true
}
```
直接返回 timerCtx 结构体中的 deadline 。
## 5.2 cancel
```Go
func (c *timerCtx) cancel(removeFromParent bool, err error) {
    // 调用 cancelCtx 的 cancel 方法，并且当前节点不从父节点删除
	c.cancelCtx.cancel(false, err)
	if removeFromParent {
		removeChild(c.cancelCtx.Context, c)
	}
	c.mu.Lock()
	// 如果存在计时器，那么停止计时器
	// 这是还未到时间，就触发 cancel 了，调用者触发的
	if c.timer != nil {
		c.timer.Stop()
		c.timer = nil
	}
	c.mu.Unlock()
}
```
对于 timerCtx ，不一定只有 timer 触发 cancel ， 因为 timerCtx 组合了 cancelCtx ，所以还有可能是 调用者手动 cancel.
## 5.3 WithDeadline
```Go
// WithDeadline返回父上下文的副本，并调整了截止日期
// 不晚于d。如果父母的截止日期已经早于d，
// WithDeadline(parent，d) 在语义上等同于parent。返回的
// 上下文的完成通道在截止日期到期时关闭，当返回的
// 调用cancel函数，或者当父上下文的Done通道为
// 关闭，以先发生者为准。
//
// 取消此上下文会释放与其关联的资源，因此代码应该
// 在此上下文中运行的操作完成后，立即调用cancel。
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
    // 如果父节点为空，那么异常，父节点不能为空
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	// 判断父节点的时间和当前节点的时间
	if cur, ok := parent.Deadline(); ok && cur.Before(d) {
		// The current deadline is already sooner than the new one.
		// 父节点的时间早于当前节点
		// 那么只需要保证父节点 cancel ，当前节点也 cancel 
		// 当前节点的定时器没有意义，直接认为当前节点是 cancelCtx
		return WithCancel(parent)
	}
	// 当前节点的时间比父节点的时间晚，需要创建 timerCtx
	c := &timerCtx{
		cancelCtx: newCancelCtx(parent),
		deadline:  d,
	}
	// 将当前节点加入父节点
	propagateCancel(parent, c)
	// 计算时间差
	dur := time.Until(d)
	// 如果时间差小于等于0，表示时间到了(可能加入节点或创建当前节点比较耗时)
	if dur <= 0 {
	    // 当前节点从父节点移除，同时设置 error 是 到时间 了
		c.cancel(true, DeadlineExceeded) // deadline has already passed
		// 否则返回 timerCtx 和 cancelFunc
		return c, func() { c.cancel(false, Canceled) }
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	// 加锁读取 err ， err 为空表示还未取消或到时间
	if c.err == nil {
	    // 增加定时器执行 func
		c.timer = time.AfterFunc(dur, func() {
		    // 触发 cancel 
			c.cancel(true, DeadlineExceeded)
		})
	}
	// 返回 timerCtx 和 cancelFunc
	return c, func() { c.cancel(true, Canceled) }
}
```
## 5.4 WithTimeout
WithTimeout实际上是复用了WithDeadline: deadline = now + timeout  
![img_11.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_11.png)


## 5.5 例子
timeout
```Go
func TestTimerCtx(t *testing.T) {
	ctxr := context.Background()
	ctx1, cancel1 := context.WithTimeout(ctxr, time.Second)
	go func() {
		fmt.Println("timerCtx ctx1 1 second timeout")
		select {
		case <-ctx1.Done():
			fmt.Println("timerCtx ctx1 1 second done")
			return
		}
	}()
	defer cancel1()
	ctx2, cancel2 := context.WithTimeout(ctxr, time.Second*3)
	defer cancel2()
	select {
	case <-ctx2.Done():
		fmt.Println("main timeout 3")
		return
	}
}
```
执行如下：  
![img_12.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_12.png)


## 5.6 总结
timerCtx 组合了 cancelCtx ，并在其基础上增加了 time 定时器，用于触发 cancel ，从而实现了 deadline 和 timeout 两种能力。

timerCtx 实现了 Context 的 Deadline 方法，直接返回了timerCtx 的成员变量 deadline.  
timerCtx 组合了 cancelCtx ，如果父节点的时间早于子节点，子节点会退化成 cancelCtx；如果子节点的deadline 大于父节点，那么子节点会增加 time 定时器，用于触发 cancel

timerCtx 有两种方式触发，一种时间计时器触发，一种是调用者触发；计时器触发，原因为 context deadline exceeded,手动取消触发为 context canceled

timerCtx 的 timeout 复用了 deadline 实现方式： deadline = now + timeout
# 6. valueCtx
源码包`src/context/context.go:valueCtx`定义了valueCtx:  
![img_13.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_13.png)


valueCtx 只是在 Context 的基础上增加了一个 key-value 对，用于在各级协程间传递一些数据。  
由于 valueCtx 即不需要 cancel ,也不需要 deadline ，那么只需要实现 Value 方法即可。
## 6.1 Value
valueCtx 的结构非常简单，只是在 Context 的基础上增加了 key 和 value 两个成员变量，所以 Value 方法也很简单：
```Go
func (c *valueCtx) Value(key interface{}) interface{} {
	if c.key == key {
		return c.val
	}
	return c.Context.Value(key)
}
```
如果当前节点的 key 等于请求的 key，那么返回当前节点的 value 。  
如果当前节点的 key 不等于请求的 key, 那么会递归向上查找 ，直到找到根节点。 找不到就返回 interface{}.
## 6.2 WithValue
WithValue也是非常的简单,创建节点，然后设置 key, value 然后将节点挂载树上。
```Go
// WithValue返回parent的副本，其中与key关联的值为
// 瓦尔。
//
// 仅对请求范围内的数据使用上下文值，这些数据传输进程和
// Api，不用于将可选参数传递给函数。
//
// 提供的键必须具有可比性，并且不应为类型
// 字符串或任何其他内置类型，以避免之间的冲突
// 使用上下文的包。WithValue的用户应该定义自己的
// 键的类型。以避免在分配给
// interface {}, 上下文键通常具有具体类型
// struct{}。或者，导出的上下文键变量的静态
// 类型应该是指针或接口。
func WithValue(parent Context, key, val interface{}) Context {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	if key == nil {
		panic("nil key")
	}
	// 需要注意的一点，key 必须是可比较的
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	// 将新增的节点加入，并设置 key, value 
	return &valueCtx{parent, key, val}
}
```
## 6.3 例子
```Go
func TestValueCtx(t *testing.T) {
	ctx := context.Background()
	ctxV1 := context.WithValue(ctx, "hi", "hi")
	ctxV2 := context.WithValue(ctxV1, "hello", "hello")
	go func() {
		fmt.Println(ctxV2.Value("hello").(string))
		fmt.Println(ctxV2.Value("hi").(string))
	}()
	time.Sleep(time.Second)
}
```
执行结果  
![img_14.png](/images/posts/2024-06-01-Go%20知识并发控制Context/img_14.png)


其实更好的用法应该是这样
```Go
func TestValueCtx(t *testing.T) {
	ctx := context.Background()
	ctx = context.WithValue(ctx, "hi", "hi")
	ctx = context.WithValue(ctx, "hello", "hello")
	go func() {
		fmt.Println(ctx.Value("hello").(string))
		fmt.Println(ctx.Value("hi").(string))
	}()
	time.Sleep(time.Second)
}
```
从始至终都使用一个变量 ctx ，像是 ctx 是一个 map 一样。
## 6.4 总结
valueCtx 主要是在 Context 的基础上实现了 Value 方法，其结构体增加了 key 和 value 两个成员变量。

valueCtx 只能增加不能减少键值对。  
valueCtx 用树形结构不断做加法来存储多个键值对。  
valueCtx 的树形结构中，一个节点只能存储一个键值对。  
valueCtx 的key必须是可比较的。  
valueCtx 查找的时候，从当前节点向根节点遍历查询，如果比较深的话，可能会存在性能问题。  
valueCtx 的实现中无锁。
# 7. afterFuncCtx
定义：
```Go
type afterFuncCtx struct {
	cancelCtx
	once sync.Once // either starts running f or stops f from running
	f    func()
}
```
afterFuncCtx 在 cancelCtx 的基础上增加了附加的收尾操作，以及执行标记。  
afterFuncCtx 在触发 cancel 后执行一次附加操作，用于类似清理回收资源等操作。
## 7.1 cancel
```Go
func (a *afterFuncCtx) cancel(removeFromParent bool, err, cause error) {
    // 执行 cancelCtx 的 cancel 操作
	a.cancelCtx.cancel(false, err, cause)
	if removeFromParent {
		removeChild(a.Context, a)
	}
	// 执行一次 给定的 func 
	a.once.Do(func() {
		go a.f()
	})
}
```
afterFuncCtx 重写了 cancelCtx 的 cancel 方法，在基础上进行了增强。
## 7.2 AfterFunc
AfterFunc用于创建 afterFuncCtx .
```Go
// AfterFunc安排在ctx完成后在自己的goroutine中调用f
// (取消或超时)。
// 如果ctx已经完成，AfterFunc会立即在自己的goroutine中调用f。
//
// 在一个上下文上多次调用AfterFunc独立操作；
// 一个不取代另一个。
//
// 调用返回的stop函数停止ctx与f的关联。
// 如果调用停止运行f，则返回true。
// 如果stop返回false，
// 要么上下文已完成，并且f已在其自己的goroutine中启动；
// 或f已经停止。
// 停止函数不等待f完成后返回。
// 如果调用者需要知道f是否完成，
// 它必须明确地与f协调。
//
// 如果ctx有一个 “AfterFunc(func()) func() bool” 方法，
// AfterFunc将使用它来安排调用。
func AfterFunc(ctx Context, f func()) (stop func() bool) {
    // 创建 afterFuncCtx
	a := &afterFuncCtx{
		f: f,
	}
	// 加入父节点中
	a.cancelCtx.propagateCancel(ctx, a)
	// 构造 终止函数
	return func() bool {
	    // 是否停止的标志
		stopped := false
		// 使用 once.Do 执行，如果 f 函数已经启动，那么 once.Do 就不会执行， stopped 是 false 
		// 如果 f 函数还未执行，那么 once.Do 执行，会将 stopped 设置为 true
		a.once.Do(func() {
			stopped = true
		})
		// 如果 stopped 为 true ,表示 f 函数还未执行，而且因为 once.Do 已经执行了 上面的 stoppped = true ，所以也就不会执行 f 了
		if stopped {
		    // f 还未执行，直接触发 cancel 
			a.cancel(true, Canceled, nil)
		}
		// 返回停止标志
		return stopped
	}
}
```
## 7.3 总结
afterFuncCtx 在 cancelCtx 的基础上增加了 sync.Once 和 调用者指定 func 的成员变量，在 cancel 触发之后执行一次 指定的 func .

afterFuncCtx 重写了 cancelCtx 的 cancel 函数，增强了cancel 的能力，以便执行附加操作。  
afterFuncCtx 利用 sync.Once 的只执行一次的特性，用于调度 stop 或 附加func  
afterFuncCtx 如果已经开始执行 附加func ，那么stop操作无法终止 附加func

afterFuncCtx 非常像给 cancelCtx 打了个补丁，增强了能力。
# 8. withoutCancelCtx
上面那几种实现都是基于 cancelCtx 的实现，除此之外，还有不依赖 cancelCtx 的实现
```Go
type withoutCancelCtx struct {
	c Context
}
```
其 Context 的实现和 emptyCtx 的实现几乎相同。
## 8.1 WithoutCancel
```Go
// WithoutCancel返回父级的副本，当父级被取消时，它不会被取消。
// 返回的上下文不返回Deadline或Err，其Done channel为nil。
// 在返回的上下文上调用 [原因] 返回nil。
func WithoutCancel(parent Context) Context {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	return withoutCancelCtx{parent}
}
```
使用 WithoutCancel 进行创建节点，加入树形结构，再无其他能力。
# 9. 总结
说了这么多的 Ctx ，其实只是 SDK 包实现的几种通用的实现，而且加上每种实现都是 Context 的子节点，
这就能让上述全部的 Ctx 进行相互自由组合，以便应用到不同的场景需要中。  
学习 Context 不仅仅学习 Context ,更学习 SDK 的一些实现方式。  
通过 返回 func ，实现让调用者调用内部的方法，甚至可以让调用者控制调用时机，但是参数相对受限。  
学习如何更好的利用接口和 struct 实现多变组合。  
学习组合增强的方式，在不改变原逻辑的前提下，增强能力。  


