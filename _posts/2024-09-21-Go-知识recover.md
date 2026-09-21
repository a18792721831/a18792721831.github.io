---
layout: post
title: "Go-知识recover"
date: 2024-09-21 13:03:40 +0800
categories: [golang, go, go recover, go 异常捕获, recover]
description: "内置函数recover用于消除panic并使程序恢复正常。recover的返回值是什么？执行recover之后程序将从哪里继续运行？recover为什么一定要在defer中使用？与panic函数一样，recover函数也是一个内置函数recover函数的返回值就是panic函数的参数，当程序产生panic时，recover函数就可用于消除panic，同时返回panic函数的参数，如果程序没有发生panic，则recover函数返回nil._go recover"
keywords: golang, go, go recover, go 异常捕获, recover
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/142415248
> - 发布时间：2024-09-21 13:03:40
> - 阅读量：1
> - 分类：Golang专栏收录该内容, 订阅专栏
> - 标签：#golang, #go, #go recover, #go 异常捕获, #recover

## 摘要

文章浏览阅读1.5k次，点赞16次，收藏13次。内置函数recover用于消除panic并使程序恢复正常。recover的返回值是什么？执行recover之后程序将从哪里继续运行？recover为什么一定要在defer中使用？与panic函数一样，recover函数也是一个内置函数recover函数的返回值就是panic函数的参数，当程序产生panic时，recover函数就可用于消除panic，同时返回panic函数的参数，如果程序没有发生panic，则recover函数返回nil._go recover

---

#### Go-知识recover

  * 1\. 介绍
  * 2\. 工作机制
  *     * 2.1 recover 定义
    * 2.2 工作流程
    * 2.3 总结
  * 3\. 原理
  *     * 3.1 recover函数的真正逻辑
    * 3.2 恢复逻辑
    * 3.3 生效条件
  * 4\. 总结
  *     * 4.1 recover的返回值是什么？
    * 4.2 执行recover之后程序将从哪里继续运行？
    * 4.3 recover为什么一定要在defer中使用？
    * 4.4 recover为什么必须被defer直接调用才生效

## 1\. 介绍

内置函数recover用于消除panic并使程序恢复正常。

> recover的返回值是什么？  
>  执行recover之后程序将从哪里继续运行？  
>  recover为什么一定要在defer中使用？

## 2\. 工作机制

### 2.1 recover 定义

与panic函数一样，recover函数也是一个内置函数  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bab68265d2b54c8e9d80ae5f4851856b.png#pic_center)

recover函数的返回值就是panic函数的参数，当程序产生panic时，recover函数就可用于消除panic，  
同时返回panic函数的参数，如果程序没有发生panic，则recover函数返回nil.  
如果panic函数参数为nil,那么仍然是一个有效的panic，此时recover函数仍然可以捕获panic，但返回值为nil。  
recover函数必须且直接位于defer函数中才有效。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/527a9a1ecaf445d2b7a4b8174a66f255.png#pic_center)

即使在defer函数中调用func，func中的recover也无法消除panic，必须是直接的defer中才行，而且不能嵌套：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/32e77cfe8322415397c9139788adc52c.png#pic_center)

### 2.2 工作流程

出现panic后，recover可以恢复程序正常的执行流程  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a84252a2baa34ccb97899b2bc303cbc2.png#pic_center)

黑色箭头代表征程执行流程，红色箭头表示出现了panic的执行流程。  
当出现panic后，panic后续的代码不在被执行。  
逐层执行defer，直到recover消除，或这到达最外层。  
如果panic被recover了，那么别的goroutine是不会感知panic的。

### 2.3 总结

recover的几个要点：

  * recover函数调用必须要位于defer函数中，且不能出现在另一个嵌套函数中
  * recover函数成功处理异常后，无法再次回到本函数发生panic的位置上继续执行
  * recover函数可以消除本函数产生或收到的panic，上游函数感知不到panic的发生

> 当函数中发生panic并用recover函数恢复后，当前函数仍然会继续返回，对于匿名函数返回值，函数将返回相应类型的零值，对于具名返回值，  
>  函数将返回当前已经存在的值。

## 3\. 原理

### 3.1 recover函数的真正逻辑

recover内置函数实际上调用的是gorecover函数  
在`src/runtime2.go`中  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/21161fa45bb24596888cb3c065337856.png#pic_center)

当panic不为空，而且panic没有被捕获，而且recover必须被defer直接调用，才会进行捕获处理。

### 3.2 恢复逻辑

gorecover函数通过协程数据结构中的_panic得到当前panic实例，如果当前panic的状态支持recover，则给该panic实例标记recoverd状态，  
最后返回panic函数的参数。  
当前执行的recover函数的defer函数是被gopanci执行的，defer函数执行结束后，在gopanic中会检查_panic的recoverd的状态，如果发现panic被恢复，  
则gopanic将结束当前panic流程，将程序流程回复正常。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/23477c4c5ef547f08a9a3015d483872d.png#pic_center)

### 3.3 生效条件
    
    
    if p != nil && !p.recovered && argp == uintptr(p.argp) {
    		p.recovered = true    
    		return p.arg
    }
    

当前协程没有产生panic时，协程结构体中_panic的链表为空，也就是 pnil，不满足恢复条件。  
假设函数包含多个defer函数，前面的defer函数通过recover函数消除panic后，函数中剩余的defer仍然会执行，但不能再次recover。也就是p.recoverdtrue  
内置函数recover没有参数，但是gorecover函数却有参数，gorecover中的参数为调用recover函数的参数地址，通常是defer 函数的参数地址。  
_panic实例中也保存了当前defer函数的参数地址，如果二者一致，说明recover被defer函数直接调用。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6738d0c1e0c945fdaa751830031d3133.png#pic_center)
    
    
    func TestTwelve(t *testing.T) {
    	defer func() { // 假设函数为 A
    		func() { // 假设函数为 B
    			// gorecover(B)
    			// (argp = B) != A
    			if err := recover(); err != nil {
    				fmt.Println(err)
    			}
    		}()
    	}()
    	panic("panic")
    }
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c5745735506b4a68a6a49831ea932809.png#pic_center)

## 4\. 总结

### 4.1 recover的返回值是什么？

recover的返回值是panic的参数。

### 4.2 执行recover之后程序将从哪里继续运行？

将从defer中继续执行(不管是否发生panic，defer 都会执行)

### 4.3 recover为什么一定要在defer中使用？

如果recover函数不在defer函数中，那么recover函数可能出现在panic之前，也可能出现在panic之后。  
出现在panic之前，因为找不到panic实例而无法生效。  
出现在panic之后，代码没有机会执行，所以recover函数必须存在于defer函数中才会生效。

### 4.4 recover为什么必须被defer直接调用才生效

这个问题等价于：  
为什么在defer函数中调用了函数A ,在A中的recover不会生效  
为什么recover只能处理本函数中或者传递给本函数的panic
    
    
    func TestTwelve(t *testing.T) {
    	defer func() { 
    		thirdPkg.Clean() // 调用第三方清理
    	}()
    	panic("panic")
    }
    

有时我们会在代码里显式的触发panic，同时还往往会在defer函数里调用第三方的包清理资源，如果第三方包也使用了recover，那么我们触发的panic将会被拦截，  
而且这种拦截可能是非预期。所以recover会被限定范围只能是defer直接调用才行。
