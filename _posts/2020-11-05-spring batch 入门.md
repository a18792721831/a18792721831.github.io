---
layout: post
title: "spring batch 入门"
date: 2020-11-05 16:00:43 +0800
categories: [Batch介绍, Batch更新历程, Batch特点, Batch实例, Batch原理]
description: "SpringBatch是一款专为构建高效、健壮的批处理应用而设计的框架。它提供了丰富的组件支持，如面向Chunk处理、事务管理及元数据管理等功能，同时还具备良好的监控、扩展性和流程定义能力。"
keywords: Batch介绍, Batch更新历程, Batch特点, Batch实例, Batch原理
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/109514091
> - 发布时间：2020-11-05 16:00:43
> - 阅读量：1
> - 分类：spring同时被 2 个专栏收录, 订阅专栏, spring batch
> - 标签：#Batch介绍, #Batch更新历程, #Batch特点, #Batch实例, #Batch原理

## 摘要

文章浏览阅读1.7k次。SpringBatch是一款专为构建高效、健壮的批处理应用而设计的框架。它提供了丰富的组件支持，如面向Chunk处理、事务管理及元数据管理等功能，同时还具备良好的监控、扩展性和流程定义能力。

---

#### spring batch 入门

  * spring batch 介绍
  *     * 批处理
    * spring batch
  * spring batch 原理
  *     * spring batch 架构
    * spring batch 优势
  * spring batch 发展历程
  *     * spring batch 2.X 特性
    * spring batch 2.2 新特性
    * spring batch 3.0 新特性
    * spring batch 4.1 新特性
    * spring batch 4.2 新特性
    * spring batch 3.0 新特性
  * spring batch hello world
  *     * 创建项目
    * 创建job
    * 启动

  
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

## spring batch 介绍

### 批处理

典型的批处理应该有以下几个特点：

  1. 自动执行，根据系统设定的工作步骤自动完成
  2. 数据量大，少则百万，多则千万甚至上亿
  3. 定时执行，如每天执行、每周或每月执行

从特点可以看出，批处理的流程可以明显的分为3个阶段：

  1. 读数据，读数据可能来自文件，数据库或消息队列
  2. 处理数据，处理读取的数据并下形成输出结果
  3. 写数据，将输出结果写入文件、数据库或消息队列

### spring batch

Spring batch是一个轻量级的、完善的批处理框架，旨在帮助企业建立健壮、高效的批处理应用。

Spring batch提供了打量可重用的组件，包括日志、追踪、事务、任务作业统计、任务重启、跳过、重复、资源管理。

对于大数据量和高性能的批处理任务，spring bach同样提供了高级功能和特性来支持，比如分区功能，远程功能。

spring batch是一个批处理应用框架，不是调度框架，但需要和调度框架合作来构建完成批处理任务。

## spring batch 原理

### spring batch 架构

spring batch核心架构分为三层：应用层、核心层、基础架构层。

![image-20201105135630995](https://i-blog.csdnimg.cn/blog_migrate/43a88c1f03d9aa5c98df42357289f4a2.png)

  * 应用层：包含所有的批处理作业，通过spring框架管理程序员的代码
  * 核心层：spring batch启动和控制所需要的核心类
  * 基础架构层：提供通用的读(ItemReader)，写(ItemWriter)和服务处理

### spring batch 优势

spring batch有如下优势：

  1. 丰富的开箱即用的组件：

读：支持文本文件读、xml文件读、数据库读、jms队列读。。

写：支持文本文件写、xml文件写、数据库写、jms队列写。。

  2. 面向Chunk的处理：

面向Chunk的处理，支持多次读，一次写，避免了多次对资源得到写入，增加批处理的处理效率。

  3. 事务管理：

spring batch框架默认采用spring提供的声明式事务管理模型。面向Chunk的操作支持事务管理，同时支持为每个tasklet操作设置细粒度的事务配置：隔离级别、传播行为、超时设置等。

  4. 元数据管理：

spring batch框架自动记录job的执行情况，包括job的执行成功、失败、失败的异常信息，step的执行成功、失败、失败的异常信息，执行次数，重试次数，跳过次数，执行时间等，方便后期的维护和查看。

  5. 易监控的批处理应用：

spring batch提供多种监控技术，支持对批处理操作的信息进行查看和管理，通过spring batch框架为批处理应用提供了灵活的监控模式：

     * 直接查看数据库
     * 通过spring batch提供的API查看
     * 通过spring batch admin查看
     * 通过jmx控制台查看
  6. 丰富的流程定义：

spring batch框架支持顺序任务，条件分支任务，基于顺序和条件任务可以组织复杂的任务流程。同时spring batch支持复用已经定义的job或者step，同时提供job和step的继承能力和抽象能力。

  7. 健壮的批处理应用：

spring batch框架支持作业的跳过，重试，重启能力，避免因错误导致批处理作业的异常中断：

     * 跳过(skip)：通常在发生非致命异常的情况下，应该不中断批处理应用；
     * 重试(retry)：发生瞬态异常情况下，应该能够通过重试操作避免该类异常，保证批处理应用的连续性和稳定性
     * 重启(restart)：当批处理应用因错误发生错误后，应该能够在最后执行失败的地方重新启动 Job实例
  8. 易扩展的批处理应用：

spring batch框架通过并发和并行技术实现应用的横向和纵向扩展机制，满足数据处理性能的需要。

扩展机制包括：

     * 多线程执行一个Step
     * 多线程并行执行多个Step
     * 远程执行作业
     * 分区执行
  9. 复用企业现有代码

spring batch框架提供多种Adapter能力，使得企业现有的服务可以方便集成到批处理应用中，避免重新开发。

## spring batch 发展历程

### spring batch 2.X 特性

  * 支持java 5

支持java5提供的增强特性

  * 非顺序的Step支持

![image-20201105144857231](https://i-blog.csdnimg.cn/blog_migrate/ca9c6e9a6d6770d6460d7640d4c0b825.png)

  * 面向Chunk处理

spring batch 1.X的执行时序图

![image-20201105144909922](https://i-blog.csdnimg.cn/blog_migrate/2e26761422476183f429f804d56f84cc.png)

spring batch 2.X的执行时序图

![image-20201105145006169](https://i-blog.csdnimg.cn/blog_migrate/f50bd03667dd01f69d072067b5f661f4.png)

  * 逻辑结构优化

spring batch 1.X的逻辑结构

![image-20201105145141636](https://i-blog.csdnimg.cn/blog_migrate/5354491226546e96fc8fb7e0faeb6b0c.png)

spring batch 1.X的代码组织结构

![image-20201105145234891](https://i-blog.csdnimg.cn/blog_migrate/3673f257762d16605225b55890a9549d.png)

spring batch 2.X的逻辑结构和代码结构

![image-20201105145306025](https://i-blog.csdnimg.cn/blog_migrate/73ca58e47a4d9962a77bb8d1d3688e73.png)

  * 强化元数据访问

spring batch 2.X中，新增了JobExplorer和JobOperator，整体的关系图

![image-20201105145403447](https://i-blog.csdnimg.cn/blog_migrate/7819ac29b3df474fd4edbeaedc7bc80e.png)

  * 增强扩展性

远程分块

分区

  * 可配置

spring batch 2.X增加了批处理的命名空间，简化了配置(对应的是xsd，也就是xml配置方式)

### spring batch 2.2 新特性

  * 支持spring data 集成
  * 支持java配置
  * 重试模块重构
  * 作业参数变化

### spring batch 3.0 新特性

  * JSR-352支持
  * 改进的Spring Batch Integration模块
  * 升级到支持Spring 4和Java 8
  * JobScope支持
  * SQLite支持

[Spring Batch参考文档中文版](<https://www.bookstack.cn/read/SpringBatchReferenceCN/README.md>)

### spring batch 4.1 新特性

  * 注解SpringBatchTest：用于更方便的测试batch组件
  * 注解EnableBatchIntegration：用于简化远程分块和分区配置
  * 支持json数据格式
  * 支持bean validation api验证项目
  * 支持jsr-305注解
  * `FlatFileItemWriterBuilder`API的增强功能

[Spring Batch 4.1 正式发布，带来了大量新特性](<https://zhuanlan.zhihu.com/p/48383390>)

### spring batch 4.2 新特性

  * 使用 [Micrometer](<https://micrometer.io/>) 来支持批量指标（batch metrics）
  * 支持从 [Apache Kafka](<https://kafka.apache.org/>) topics 读取/写入（reading/writing） 数据
  * 支持从 [Apache Avro](<https://avro.apache.org/>) 资源中读取/写入（reading/writing） 数据
  * 改进支持文档

[Spring Batch 4.2 新特性](<https://blog.csdn.net/caojiajin8990/article/details/100965193/>)

### spring batch 3.0 新特性

  * 新增`SynchronizedStreamWriter`
  * 新增`JpaQueryProvider`,`JpaCursorItemReader`
  * 新增`JobParameterIncrementer`
  * GraalVM支持
  * Java记录支持

[What’s New in Spring Batch 4.3](<https://docs.spring.io/spring-batch/docs/4.3.x/reference/html/whatsnew.html#whatsNew>)

## spring batch hello world

创建一个spring batch 的hello world非常的简单。

在spring batch的quick start 就有实例

[Creating a Batch Service](<https://spring.io/guides/gs/batch-processing/>)

### 创建项目

在ide中选择创建spring项目

选择gradle项目，jdk11

![image-20201105153527740](https://i-blog.csdnimg.cn/blog_migrate/634fbf85c02a8a898b8ad2ca74d8c06c.png)

选中spring batch

![image-20201105153544670](https://i-blog.csdnimg.cn/blog_migrate/93fcb495714e8426e82ae88d3fdbba53.png)

因为spring batch还需要存储元数据，所以，还需要一个数据库

![image-20201105153640539](https://i-blog.csdnimg.cn/blog_migrate/6e9593792a5f8b3265482d3196a5aa65.png)

### 创建job

项目创建完成后，等待依赖下载。

然后创建job

![image-20201105153801308](https://i-blog.csdnimg.cn/blog_migrate/f48312a845d92c9934a7e6a7cc5b1663.png)

在HelloJobConf上需要加上注解，并且注入操作spring batch元数据的接口

![image-20201105153845446](https://i-blog.csdnimg.cn/blog_migrate/6a9403b938f6260b477b7035d73f6db5.png)

创建ItemReader,因为批处理，如果一直有数据，会一直执行下去，所以，我们需要一个变量来标识退出批处理的时机

![image-20201105154412813](https://i-blog.csdnimg.cn/blog_migrate/1277584c9a67a6f53b1ae13a53e0265b.png)

![image-20201105154242396](https://i-blog.csdnimg.cn/blog_migrate/0093cce49e40e26c85695b73b53213b8.png)

可以看到，我们只想批处理20次，reader读取的是字符串。

创建ItemProcess，process里面我们什么都不做，打印reader的字符串。

![image-20201105154357374](https://i-blog.csdnimg.cn/blog_migrate/997138c0f0b2885a196860ae1362b29a.png)

创建ItemWriter,writer里面也只是打印

![image-20201105154447218](https://i-blog.csdnimg.cn/blog_migrate/8b57830f8222067196ee724c49a3ba8b.png)

创建执行步step

![image-20201105154616045](https://i-blog.csdnimg.cn/blog_migrate/f233b8b903c65ddfdc214f31996e8a08.png)

创建job

![image-20201105154707623](https://i-blog.csdnimg.cn/blog_migrate/a3837e8d8374c073cfbb42f50cbd73e4.png)

### 启动

直接启动spring boot main类即可

![image-20201105154802391](https://i-blog.csdnimg.cn/blog_migrate/ebc525c66b992d10687f81a446d5392f.png)

可以看出来，10个一批，和我们在chunk中定义的一样。

![image-20201105154820053](https://i-blog.csdnimg.cn/blog_migrate/19cc201701ea2c234bc7f369618d1a83.png)

第21个结束，共3批

![image-20201105154918264](https://i-blog.csdnimg.cn/blog_migrate/af443306234bda61eb36ed59aa4cf845.png)

怎么区分批？

chunk是多次读，一次写，所以，每次写读的分界线就是一批。

10个一批，集中写数据。
