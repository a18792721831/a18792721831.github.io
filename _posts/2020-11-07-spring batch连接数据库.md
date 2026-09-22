---
layout: post
title: "spring batch连接数据库"
date: 2020-11-07 14:50:05 +0800
categories: [batch连接数据库, batch跳过job, batch无法执行job, batch执行了空命令, batch根据教程无法启动]
description: "本文详细介绍了如何使用SpringBatch框架搭建批处理任务，包括配置数据库、创建Job及各组件、配置Job参数等步骤。特别关注了Oracle数据库环境下遇到的问题及解决方案。"
keywords: batch连接数据库, batch跳过job, batch无法执行job, batch执行了空命令, batch根据教程无法启动
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/109546988
> - 发布时间：2020-11-07 14:50:05
> - 阅读量：2
> - 分类：spring batch专栏收录该内容, 订阅专栏
> - 标签：#batch连接数据库, #batch跳过job, #batch无法执行job, #batch执行了空命令, #batch根据教程无法启动

## 摘要

文章浏览阅读2k次，点赞6次，收藏9次。本文详细介绍了如何使用SpringBatch框架搭建批处理任务，包括配置数据库、创建Job及各组件、配置Job参数等步骤。特别关注了Oracle数据库环境下遇到的问题及解决方案。

---

#### spring batch连接数据库

  * 创建项目
  * 创建配置
  * 创建job
  *     * 创建job配置类
    * 创建ItemReader
    * 创建ItemProcess
    * 创建ItemWriter
    * 组装Step
    * 配置Job
  * 配置数据库
  *     * 覆盖接口jobRepository
  * 注意

  
github地址： 

https://github.com/a18792721831/studybatch.git

文章列表：

[spring batch 入门](<https://blog.csdn.net/a18792721831/article/details/109514091>)

[spring batch连接数据库](<https://blog.csdn.net/a18792721831/article/details/109546988>)

[spring batch元数据](<https://blog.csdn.net/a18792721831/article/details/109550018>)

[spring batch Job详解](<https://blog.csdn.net/a18792721831/article/details/109691602>)

[spring batch step详解](<https://blog.csdn.net/a18792721831/article/details/110200113>)

[spring batch ItemReader详解](<https://blog.csdn.net/a18792721831/article/details/110637684>)

[spring batch itemProcess详解](<https://blog.csdn.net/a18792721831/article/details/110681958>)

[spring batch itemWriter详解](<https://blog.csdn.net/a18792721831/article/details/110832092>)

[spring batch 作业流](<https://blog.csdn.net/a18792721831/article/details/111083504>)

[spring batch 健壮性](<https://blog.csdn.net/a18792721831/article/details/111404284>)

[spring batch 扩展性](<https://blog.csdn.net/a18792721831/article/details/111408460>)

## 创建项目

![image-20201107134817228](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/28725f3682da444e3e67c2adfec9830c.png)

选择数据库和spring batch

![image-20201107134850794](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3e561e5e154aa28ce4382d263bb1513d.png)

下载完依赖后，目录结构如下

![image-20201107134924932](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a3d8bea8558030b8042fcc15531cc4c9.png)

## 创建配置

在resources目录下创建application.yaml配置文件

![image-20201107135012111](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3f7c64399fa4fdec4cc601f5428fb733.png)

接着配置数据库
    
    
    spring:
      main:
        allow-bean-definition-overriding: true
      datasource:
        username: springbatch
        password: springbatch
        url: jdbc:oracle:thin:@10.0.250.178:1521/starboss
        driver-class-name: oracle.jdbc.OracleDriver
        initialization-mode: embedded
        schema: classpath:/org/springframework/batch/core/schema-oracle10g.sql
      batch:
        initialize-schema: embedded
        job:
          enabled: true
          names: study2-oracle
    

## 创建job

### 创建job配置类

![image-20201107135116623](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a821450ba8d4d3c46ff170ca320217ea.png)

### 创建ItemReader

我们不使用spring batch提供的已经实现的包装好的数据读取器，而是直接实现接口。

![image-20201107135241687](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e5af46d7aff70d8cf5624b00ccc94229.png)

### 创建ItemProcess

![image-20201107135304534](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/681b7d83bbffe3de951dbc87f314dbbb.png)

### 创建ItemWriter

![image-20201107135327646](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6a78962a3d51358dde2cb110393cc9a5.png)

### 组装Step

![image-20201107135409954](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/743f76e1fd2a429b837a8a96e546c3f6.png)

### 配置Job

![image-20201107135431435](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d1b69a496317865ccd7471863f9f12f8.png)

## 配置数据库

到目前为止，我们已经创建了一个标准的job。创建job的过程中，需要使用到jobBuilderFactory和stepBuilderFactory.

我们使用的jobBuilderFactory和stepBuilderFactory是spring batch注入的。

spring batch注入的这两个接口，使用的是默认的配置。

而我们在application.yaml中配置的oracle数据库，却没有被用到。

所以，还需要配置数据库。

![image-20201107135713704](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a1d10bd8e073187c46567c5132dc5af9.png)

我们在容器中注入了相关的配置，没有注入使用我们想要的配置的接口。

### 覆盖接口jobRepository

![image-20201107135843277](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2bdd7710224b11e58430d600ab2638d2.png)

## 注意

这里面有一个很大坑。

在spring batch的quick start中 ，并没有需要我们自己去执行job，而是只需要将job注入容器，然后就会执行。

理所当然的，我们配置了oracle数据库后，也会认为，我们只需要将job注入容器，然后将使用了数据库配置的jobRepository注入容器，就可以了。

当然，我也是这么想的，所以，我将这一切配置好后，启动，发现了一个很有意思的事情：

![image-20201107140209999](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6d44fc7bc7e783bfa5dbc782f916bb80.png)

就像这样，什么都没有。

提示`Running default command line with: []`

什么鬼，我们不是已经将job注入容器了吗，为什么不执行？

**数据库驱动问题吗**

网上有很多的资料说，数据库的驱动，版本问题，很容易出现问题。

于是，我就创建了另外一个项目：mysql的。

相比来说，oracle数据库驱动，因为不容易下载等原因，问题总是比mysql多些。

可惜的是，换成了mysql数据库后，还是相同的。

我更换了网上常见的ojdbc8,12,14等等，各种版本。

发现还是不行。

**开启debug级别**

想着开启debug之后，打印的信息更多，可能会看出问题的原因来：

![image-20201107140704297](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ba1dbf60b75834b6993444b9198b7d01.png)

开启debug级别的日志后，确实打印的信息更多了

![image-20201107140755667](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/32c895f61a1a09c0a22a395306e7ccf3.png)

从日志中可以看出来，我们的job配置是没有任何问题的。

容器扫描到了job，但是却被spring batch跳过了。

原因是从注册元数据中没有找到这个job。

什么情况，我们使用jobBuilderFactory.get(“hello-job”)不就是想注册这个job么。

**是不是数据库就没有连上？**

于是，我创建了一个新的oracle数据库用户，接着配置了oracle用户，在其他数据不修改的情况下，启动。

发现还是这样。

而且，数据库中，根本就没有执行init-schema脚本。

数据库真的没有连上。(还是太年轻)

但是数据库连接就是这样配置啊，驱动，配置等等都有了。还需要怎么做呢？

**重新阅读quick start**

确实不甘心，我要学习spring batch，怎么在第一步上就放弃呢。

重新阅读quick start。没有发现什么有用的信息。

我强迫自己忘记现在已经做的全部事情，从头开始。

避免惯性思维。

于是从头开始，重新按照教程做了一遍。

发现和我的项目的区别就是数据库不同。

于是我将我之前的oracle数据库在换回hsql数据库。

神奇的一幕出现了：

在hsql中，我们将job注入容器就可以了，启动项目，spring batch中会默认调度我们的job。

在oracle中，我们将job注入容器，一点屁用都没有，甚至，初始化脚本都不会执行。

**破局**

我发现这个差异，就去github,stackoverflow中搜索spring batch oracle。发现没有什么有用的回答。

继续回想看到的资料，说pring batch是一个批处理的框架，需要配合其他调度框架使用。

我可能明白了，需要我们自己调度。

![image-20201107143319098](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8c3e0cc09728aac672fa4f5c79f49b87.png)

于是，我在job的配置类中，写了调度方法

![image-20201107143509622](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b36d6873bb6488e96397bd12fb8b21d3.png)

接着启动

![image-20201107143541490](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b4efddafe061f0511e01bc9faad20faa.png)

成功

**另一个坑**

在spring batch的配置中，还有一个地方有坑。

![image-20201107143639613](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a85afc8b298df06cf6c8979cdaad93ad.png)

在数据库初始化策略中，有三种模式：

![image-20201107143714944](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/aedc5eb787a7c4383eec314ca2d7abfd.png)

从不初始化，每次都初始化，根据需要初始化。

所以，配置的是embedded.

结果，并不会初始化，oracle数据库。或许，是我自己理解有误，只有切入式的数据库，才使用embedded。

所以，第一次启动oracle数据库，我们需要将初始化策略配置为always，或者将初始化脚本手动执行。

然后将初始化策略配置为never。

这部分在github的issus中也得到了侧面验证。初始化脚本和数据库用户的权限问题。配置的数据库用户只有数据的读写权限，没有数据表操作权限。

也就是，dml和ddl是分离的。

如果使用oracle数据库，那么使用embedded和never是一样的效果。

**另一种不执行的情况**

job instance执行成功后，就不在重复执行。

刚开始，我启动job的时候，配置的jobParameter是空的。

第一次执行没有问题，第二次执行，就是跳过这个job instance了。

因为在spring batch中，识别一个job instance = job name + job parameter

我们两次执行的parameter是相同的，也就是同一个job instance，因为job instance已经执行成功了，就不在重复执行了。

所以，最好是job parameter中传入时间，即使我们不会用到。

这样保证可以重复执行。

这个问题，耗费了我好几天的学习时间才解决，希望可以帮到你。
