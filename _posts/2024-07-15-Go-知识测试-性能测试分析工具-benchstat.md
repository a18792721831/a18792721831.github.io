---
layout: post
title: "Go-知识测试-性能测试分析工具-benchstat"
date: 2024-07-15 09:53:28 +0800
categories: [golang, go, go 测试, benchmark, go 性能测试分析, benchstat]
description: "p=0.000表示结果的可信程度，p值越大可信程度越低，统计学中通常把p=0.05作为临界值，超过此值说明结果不可信，可能是样本过少等原因。benchstat 是 Go 官方推荐的一款命令行工具，可以针对一组或多组样本进行分析，如果同时分析两组样本，还可以给出性能变化结果。至此，go的测试结束，通过学习go测试，首先是会写各种测试case，其次是理解了各种测试的目的，最后则是学习了很多优秀的实现。自动计算出了平均值，在cpu=10的时候，每次操作2.534微妙，样本离散值(2%)如果没有设置，默认是。_golang benchmark测试结果怎么看"
keywords: golang, go, go 测试, benchmark, go 性能测试分析, benchstat
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/140430218
> - 发布时间：2024-07-15 09:53:28
> - 阅读量：1
> - 分类：Golang专栏收录该内容, 订阅专栏
> - 标签：#golang, #go, #go 测试, #benchmark, #go 性能测试分析, #benchstat

## 摘要

文章浏览阅读1.5k次，点赞29次，收藏16次。p=0.000表示结果的可信程度，p值越大可信程度越低，统计学中通常把p=0.05作为临界值，超过此值说明结果不可信，可能是样本过少等原因。benchstat 是 Go 官方推荐的一款命令行工具，可以针对一组或多组样本进行分析，如果同时分析两组样本，还可以给出性能变化结果。至此，go的测试结束，通过学习go测试，首先是会写各种测试case，其次是理解了各种测试的目的，最后则是学习了很多优秀的实现。自动计算出了平均值，在cpu=10的时候，每次操作2.534微妙，样本离散值(2%)如果没有设置，默认是。_golang benchmark测试结果怎么看

---

#### Go-知识测试-性能测试分析工具-benchstat

  * [benchmark 结果](<#benchmark__4>)
  * [benchstat](<#benchstat_18>)
  *     * [确认 `benchstat` 已安装](<#_benchstat__27>)
    * [确认 `GOPATH` 和 `GOBIN`](<#_GOPATH__GOBIN_35>)
    * [将 `$GOPATH/bin` 添加到 `PATH`](<#_GOPATHbin__PATH_47>)
    * [验证安装](<#_75>)
    * [检查安装路径](<#_85>)
  * [使用](<#_94>)

> 传送门：[Go-知识测试-性能测试](<https://blog.csdn.net/a18792721831/article/details/140094738>)

## benchmark 结果

benchmark 测试是实际项目中经常使用的测试方法，下面是一个执行的结果  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a7f6c931a357445cac2f0c02d768979c.png#pic_center)

输出结果的含义：  
BenchmarkMakeWithout 表示1个CPU，BenchmarkMakeWithout-2 表示使用了2个CPU  
552829 表示循环次数， 2129 ns/op 表示一次调用花费 2129 纳秒  
480.98MB/s表示1秒可以处理的数据大小  
25208B/op 表示每次调用占用的字节数  
12 allocs/op 表示每次调用分配的对象数  
分析测试的逻辑：  
BenchmarkMakeWithout 表示创建切片的时候不指定容量，使用自动扩容。  
BenchmarkMakeWith 表示创建切片的时候，指定容量，不使用自动扩容。  
从 12 allocs/op 和 3 allocs/op 可以明显的看出区别，指定容量，分配了更少的对象。

## benchstat

benchstat 是 Go 官方推荐的一款命令行工具，可以针对一组或多组样本进行分析，如果同时分析两组样本，还可以给出性能变化结果。  
使用`go get golang.org/x/perf/cmd/benchstat`可以安装benchstat，将被安装到$GOPATH/bin中。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/80111d09181642ffba96977793a64cf6.png#pic_center)

使用时需要把benchmark测试输出到文件中，benchstat会读取这些文件。  
如果你使用的是mac，并且在goland中执行，那么可能会提示  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1b13142a80d6440089f0f6575854e561.png#pic_center)

### 确认 `benchstat` 已安装

首先，确保 `benchstat` 已经被安装。可以再次运行以下命令来安装它：
    
    
    go install golang.org/x/perf/cmd/benchstat@latest
    

### 确认 `GOPATH` 和 `GOBIN`

默认情况下，Go 会将可执行文件安装到 `$GOPATH/bin` 目录下。如果没有设置 `GOPATH`，默认路径是 `$HOME/go`。你可以通过以下命令来确认你的 `GOPATH`：
    
    
    echo $GOPATH
    

如果没有输出，说明使用的是默认路径 `$HOME/go`。

还可以设置 `GOBIN` 环境变量来指定可执行文件的安装路径。如果没有设置，默认是 `$GOPATH/bin`。

### 将 `$GOPATH/bin` 添加到 `PATH`

确保的 `PATH` 环境变量包含 `$GOPATH/bin`。可以通过以下命令来检查：
    
    
    echo $PATH
    

如果 `$GOPATH/bin` 不在 `PATH` 中，需要将其添加到你的 shell 配置文件中（例如 `.zshrc` 或 `.bashrc`）。

对于 `zsh`，可以编辑 `~/.zshrc` 文件：
    
    
    nano ~/.zshrc
    

然后添加以下行：
    
    
    export PATH=$PATH:$(go env GOPATH)/bin
    

保存并关闭文件，然后重新加载配置：
    
    
    source ~/.zshrc
    

### 验证安装

现在可以再次尝试运行 `benchstat`：
    
    
    benchstat
    

如果一切正常，应该能够运行 `benchstat` 而不会遇到 `command not found` 错误。

### 检查安装路径

还可以手动检查 `benchstat` 是否安装在预期的目录中：
    
    
    ls $(go env GOPATH)/bin | grep benchstat
    

如果看到 `benchstat`，说明它已经正确安装。

## 使用

比如把 BenchmarkWithout 输出到 without 中  
`go test -v slice_test.go -bench MakeWithout -count 20 -benchmem > without`  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5317058deea64b019452c2dab8f1f329.png#pic_center)

然后使用`benchstat without`执行  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1fc41392a73d4d85b78144032e67c7ee.png#pic_center)

自动计算出了平均值，在cpu=10的时候，每次操作2.534微妙，样本离散值(2%)  
在执行一次，输出到withou1，然后一次性传入两个文件分析  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2b4f5e56cc0d4503af24a06ab59ebecb.png#pic_center)

因为两次执行几乎没有变化，所以会提示all samples are equal。  
如果将BenchmarkMakeWith 和BenchmarkMakeWithout做对比  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e678df0931a747f99c657875b6be22dc.png#pic_center)

p=0.000表示结果的可信程度，p值越大可信程度越低，统计学中通常把p=0.05作为临界值，超过此值说明结果不可信，可能是样本过少等原因。  
当只有两组样本是，benchstat 还会额外计算出差值，用正负表示变化的百分比。  
如果是不同的两组样本：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/95ce99f3dc9d4a0fbb404b53c2fc500c.png#pic_center)

> 至此，go的测试结束，通过学习go测试，首先是会写各种测试case，其次是理解了各种测试的目的，最后则是学习了很多优秀的实现。加油~