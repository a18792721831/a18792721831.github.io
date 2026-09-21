---
layout: post
title: "spring batch元数据"
date: 2020-11-07 17:46:47 +0800
categories: [batch元数据, batch数据存储, batch的核心数据, batch的原理, batch数据表详解]
description: "本文深入探讨SpringBatch元数据管理机制，包括Job、JobInstance、JobParameters等核心概念及其相互关系，介绍了如何利用这些元数据确保批处理作业的正确执行与重启。"
keywords: batch元数据, batch数据存储, batch的核心数据, batch的原理, batch数据表详解
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/109550018
> - 发布时间：2020-11-07 17:46:47
> - 阅读量：1
> - 分类：spring batch专栏收录该内容, 订阅专栏
> - 标签：#batch元数据, #batch数据存储, #batch的核心数据, #batch的原理, #batch数据表详解

## 摘要

文章浏览阅读1.3k次。本文深入探讨SpringBatch元数据管理机制，包括Job、JobInstance、JobParameters等核心概念及其相互关系，介绍了如何利用这些元数据确保批处理作业的正确执行与重启。

---

#### spring batch元数据

  * [准备](<#_29>)
  * [Job](<#Job_67>)
  *     * [Job Instance](<#Job_Instance_87>)
    * [Job Parameters](<#Job_Parameters_123>)
    * [Job Execution](<#Job_Execution_209>)
  * [Step](<#Step_249>)
  *     * [Step Execution](<#Step_Execution_259>)
  * [Execution Context](<#Execution_Context_277>)
  *     * [Job Execution Context](<#Job_Execution_Context_281>)
    * [Step Execution Context](<#Step_Execution_Context_289>)
    * [关系](<#_297>)
  * [Job Repository](<#Job_Repository_301>)
  *     * [Job Repository的属性](<#Job_Repository_307>)
    * [Job Repository类型](<#Job_Repository_311>)
    * [元数据表](<#_323>)
  * [Job Launcher](<#Job_Launcher_337>)
  * [ItemReader](<#ItemReader_341>)
  * [ItemProcessor](<#ItemProcessor_347>)
  * [ItemWriter](<#ItemWriter_355>)

  
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

## 准备

为了更好的理解spring batch元数据，我们使用常见的数据库，将元数据存储到数据库中，然后使用SQL查询出来，就可以 快速的理解spring的元数据了。

我们新建一个spring batch的项目，gradle项目：

![image-20201107150156677](https://i-blog.csdnimg.cn/blog_migrate/6db1dc50af94690c6379cb4524ac298e.png)

然后还是我们还是用hello world的代码进行学习。

我们将代码完全拷贝过来,需要连接数据库。

![image-20201107150524896](https://i-blog.csdnimg.cn/blog_migrate/d9302888604811da6e4be4e38542dc17.png)

接着在resource目录下创建application.yaml文件。

在application.yaml中配置数据库以及spring batch

![image-20201107150553541](https://i-blog.csdnimg.cn/blog_migrate/4fe261054a7542d99cf35f3a61f6a9df.png)
    
    
    spring:
      main:
        allow-bean-definition-overriding: true
      datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://10.0.228.117:13306/springbatch?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
        username: springbatch
        password: springbatch
        schema: classpath:org/springframework/batch/core/schema-mysql.sql
        initialization-mode: never
      batch:
        initialize-schema: never
        job:
          enabled: true
          names: study3
    

## Job

批处理作业Job由一组Step组成，同时是作业配置的顶层配置。

每个作业有自己的名字、可以定义Step执行的顺序，以及定义作业是否可以重启。

![image-20201105191052031](https://i-blog.csdnimg.cn/blog_migrate/4b89aae7d84eba6ff2b17233441fba20.png)

Job执行的时候会生成一个Job Instance,Job Instance 包含执行Job期间产生的数据已经Job执行的状态信息等；Job Instance通过Job Name和Job Parameter来区分；每次Job执行的时候都有一个Job Execution，Job Execution负责具体job的执行。

Job:作业

Job Instance：作业实例

Job Parameter：作业参数

Job Execution：作业执行器

![image-20201105191415181](https://i-blog.csdnimg.cn/blog_migrate/09f4a9c437bc5e94edbc7b36cda26771.png)

### Job Instance

Job Instance(作业实例)是一个运行期的感念，Job每执行一次都会涉及一个Job Instance。

Job Instance来源可能有两种：一种是根据设置的Job Parameters从Job Repository(作业仓库)中获取一个；如果根据Job Parameters从Job Repository没有获取Job Instance，则重新创建一个新的Job Instance。

**Job Instance定义**

![image-20201107155515911](https://i-blog.csdnimg.cn/blog_migrate/e136f46f01b98046b809806a65697710.png)

字段说明

![image-20201107172946603](https://i-blog.csdnimg.cn/blog_migrate/28309f1ef41853b43f1837060af2ff88.png)

我们修改job的名字为job-instance，然后启动一次。

![image-20201107155648252](https://i-blog.csdnimg.cn/blog_migrate/fa1c2f84b5bb4efecccf85f20c49bc3f.png)

然后在mysql数据库中查询

![image-20201107155903089](https://i-blog.csdnimg.cn/blog_migrate/42fb03369a4e61bf4eae4780db17c64b.png)

如果，我们将job的参数去掉，重复执行，version会增加吗？

第一次：

![image-20201107160046751](https://i-blog.csdnimg.cn/blog_migrate/20f155fd6c2d17327bd628698e51503f.png)

第二次：

![image-20201107160129781](https://i-blog.csdnimg.cn/blog_migrate/b7d4e58e486afae03593c803848fef51.png)

![image-20201107160137160](https://i-blog.csdnimg.cn/blog_migrate/434cbffaf1d0b10c80666789c2018090.png)

并不会增加。

### Job Parameters

Job 通过Job Parameters来区分不同的Job Instance。简单说，就是Job Name + Job Parameters来唯一地确定一个Job Instance。如果Job Name一样，则Job Parameters肯定不一样；但是对于不同的Job来说，允许有相同的Job Parameters。

![image-20201107161054325](https://i-blog.csdnimg.cn/blog_migrate/d9463b2fbf764b6e72288a4d86027af7.png)

**Job Parameters定义**

![image-20201107161139610](https://i-blog.csdnimg.cn/blog_migrate/012e501346ceab1c69ff0d50306ee8c7.png)

字段说明

![image-20201107173010064](https://i-blog.csdnimg.cn/blog_migrate/7c183759f8f7940a1f4fb4954b72eac3.png)

![image-20201107173019852](https://i-blog.csdnimg.cn/blog_migrate/19bed52c2c7e4ec4551d6d1a90dc5809.png)

查看下Job Parameters表

![image-20201107161221151](https://i-blog.csdnimg.cn/blog_migrate/838b00f73fb4d2bd74a410149439044c.png)

现在，有一个问题：Job Instance和Job Parameters的关系？

在Job Instance中有这样一个字段：JOB_KEY。

这个key就是替代了Job Parameter。

用一个比较容易理解的方式来说明Job和Job Parameters的关系。

首先从运行入手：

我们在启动一个Job的时候，需要传入Job和JobParameters:

![image-20201107161842709](https://i-blog.csdnimg.cn/blog_migrate/aa6d30c2584792f237cb3efb98029461.png)

![image-20201107161933731](https://i-blog.csdnimg.cn/blog_migrate/827d7642ca92c68146c29ac7dcc4359f.png)

第一步，根据我们传入的Job和JobParameter从数据库中查询执行结果。

我们重点看如何查询。

在查询执行结果的时候，需要查询JobInstance

![image-20201107162114549](https://i-blog.csdnimg.cn/blog_migrate/071c2332756ae7d69061458e1183b959.png)

jobExecutionDao的getJobInstance方法有两个实现

![image-20201107162212423](https://i-blog.csdnimg.cn/blog_migrate/3da28763973623e7c8f328284877f3cb.png)

通过查看源码，猜测，JdbcJobInstanceDao是外部的，非切入式的数据库使用的dao

而MapJobInsttanceDao是切入式的数据库使用的dao。(猜测，不一定准确)

所以，我们查看JdbcJobInstanceDao的方法

![image-20201107162351699](https://i-blog.csdnimg.cn/blog_migrate/6bb20524b979005d2ffc37ec1c3cd02f.png)

我们看下如何生成的jobKey

第一步获取JobParameter的map化对象

![image-20201107162541168](https://i-blog.csdnimg.cn/blog_migrate/f8a5b928dc222bd8690b1495baa2c6a9.png)

第二步，将第一步获取的字符串进行MD5编码，如果MD5编码失败了，那么去UTF-8格式的前32位。(16进制)

![image-20201107162611361](https://i-blog.csdnimg.cn/blog_migrate/f1a04bc794a95028dfb94c3b19e187d6.png)

拿到了jobKey，如何查询JobInstance呢?

![image-20201107162737642](https://i-blog.csdnimg.cn/blog_migrate/66c83935076b75c2ca2b7b9bfcde2180.png)

我们看下SQL

![image-20201107162806159](https://i-blog.csdnimg.cn/blog_migrate/b17a2bcb45f33aaa2f2d467ba6a5a9a3.png)

合并起来，简单修饰下：

![image-20201107162917924](https://i-blog.csdnimg.cn/blog_migrate/011444d24a7c590b07f3e409b67cde2b.png)

所以，Job Instance和Job Parameter的关系，在JobInstance表中存储，jobKey字段间接关联了JobParameter。

**Job Parameters类型**

Job Parameter只支持字符串，时间，长整型和双精度类型。

![image-20201107164156433](https://i-blog.csdnimg.cn/blog_migrate/c12f4b4d5c3aebda73e228bf876fb877.png)

### Job Execution

我们前面看到了，执行一个Job，会根据Job Name + Job Parameter查询Job Instance。

**Job Execution定义**

![image-20201107163423875](https://i-blog.csdnimg.cn/blog_migrate/cd55c039fb653c614146ff76015bc281.png)

字段说明

![image-20201107173040796](https://i-blog.csdnimg.cn/blog_migrate/b5c27d88385a46f5c5537b4381dc77bb.png)

![image-20201107173051712](https://i-blog.csdnimg.cn/blog_migrate/6482b6518ea86c98abb7a994e67a309a.png)

![image-20201107164341946](https://i-blog.csdnimg.cn/blog_migrate/607020959d9fa352ec62b5115c3b9430.png)

![image-20201107163528775](https://i-blog.csdnimg.cn/blog_migrate/b9c6ddd25a41889a8ddb018344dad0e1.png)

在Job Execution中记录了Job Instance执行的情况，执行结果，退出信息等。

然后根据JobInstance查询Job Execution

![image-20201107163158867](https://i-blog.csdnimg.cn/blog_migrate/93982e9e843aa76a29a9bbb19c0c6e9b.png)

Job Execution的查询也是非常的简单

![image-20201107163250212](https://i-blog.csdnimg.cn/blog_migrate/a6c3e116bb2f2dfb10a0982ca0c90614.png)

这就是SQL

![image-20201107163322898](https://i-blog.csdnimg.cn/blog_migrate/c84c1cbbd1b9b6f5a442df64655339b7.png)

如果Job Instance的最后一次执行成功了，那么抛出异常；如果最后一次执行失败了，那么验证是否有正在运行的，当前Job Instance的任务，或者是，最后一次还未结束。

![image-20201107163937126](https://i-blog.csdnimg.cn/blog_migrate/d7af7c6c7d35dcbfc11ca9d8f825bf10.png)

如果没有找到最后一个Job Execution，那么就验证Job Parameter的类型等是否符合规则。如果验证通过，那么就创建Job Execution

![image-20201107164314028](https://i-blog.csdnimg.cn/blog_migrate/f1699a3b3225628ae8c137bc858b2378.png)

## Step

Step表示作业中的一个完整步骤，一个Job可以由一个或者多个Step组成。Step包含一个世纪运行的批处理任务中的所有必须的信息，Step可以是非常简单的业务实现，比如打印字符串；也可以是非常复杂的业务处理。

一个Job可以拥有一到多个Step；一个Step可以有一到多个Step Execution(当一个Step执行失败，下次重新执行该任务的时候，会为该Step重新生成一个Step Execution)；一个Job Execution可以有一到多个Step Execution(当一个Job由多个Step组成时，每个Step执行都会生成一个新的Step Execution，则一个Job Execution会拥有多个Step Execution)。

Step中可以配置tasklet,partition,job,flow。

![image-20201107165922395](https://i-blog.csdnimg.cn/blog_migrate/dc97b52837b1ceca77859ea084492170.png)

### Step Execution

Step Execution是Step执行的句柄，一次Step执行可能成功，也可能失败。

**Step Execution定义**

![image-20201107170202420](https://i-blog.csdnimg.cn/blog_migrate/43089aa6d630da288007e12f5022bc41.png)

字段说明

![image-20201107170223899](https://i-blog.csdnimg.cn/blog_migrate/6555c2cb71274a4cc7fadf5c37629f79.png)

![image-20201107170232520](https://i-blog.csdnimg.cn/blog_migrate/888206d124f7445e2f70a2d136085a99.png)

具体的记录

![image-20201107170319843](https://i-blog.csdnimg.cn/blog_migrate/c73cc92194510ab8a19d5e768aa0a335.png)

## Execution Context

Execution Context是spring batch框架提供的持久化与控制的key/value对，能够让开发者在Step Execution或Job Execution中保存需要进行持久化的状态。框架会每次commit后记录当前提交记录数以及读取的记录数，这样当Step发生错误时，下次重启Job会根据Execution Context中的数据恢复状态，保证继续从上次 失败的点重新执行。

### Job Execution Context

**Job Execution Context定义**

![image-20201107170726812](https://i-blog.csdnimg.cn/blog_migrate/5dc7bd7ee63cc0e883c837788c08c481.png)

![image-20201107170805715](https://i-blog.csdnimg.cn/blog_migrate/23369297209e30938c5baa6440b1eb39.png)

### Step Execution Context

**Step Execution Context定义**

![image-20201107170849777](https://i-blog.csdnimg.cn/blog_migrate/47bc5395f3aa16977f5d49ba35421a3e.png)

![image-20201107170923472](https://i-blog.csdnimg.cn/blog_migrate/309df5f2a2a5848a62189a3368ef63ac.png)

### 关系

两类上下文之间的关系：一个Job Execution对应一个Job Execution的上下文；每个Step Execution对应一个Step Execution上下文；同一个Job中的Step Execution共用Job Execution的上下文。因此如果同一个Job的不同Step间需要共享数据时，则可以通过Job Execution的上下文共享数据。

## Job Repository

spring batch框架提供Job Repository来存储Job执行期的元数据(这里的元数据是指Job Instance,Job Execution,Job Parameters,Step Execution,Execution Context等数据)，并提供两种默认实现。一种是存放在内存中，另一种是将元数据存放在数据库中。将元数据存放在数据库中，可以随时地监控批处理Job的执行状态，查看Job执行结果是成功还是失败，并且在Job失败的情况下重新启动Job.

spring batch框架的Job Repostory支持如下的数据库：DB2,Derby,H2,HSQLDB,Mysql,Oracle,PostgreSQL,SQLServer,Sybase.

### Job Repository的属性

![image-20201107172020643](https://i-blog.csdnimg.cn/blog_migrate/88bea36d733f8e071e6e9d7c45b5d89e.png)

### Job Repository类型

前面我们说过，Job Repository有两种实现方式，一种是基于数据库的，一种是基于内存的。

使用数据库：

![image-20201107172408040](https://i-blog.csdnimg.cn/blog_migrate/41d0c6aae3fde51edae875b9aa493e79.png)

使用内存：

![image-20201107172602725](https://i-blog.csdnimg.cn/blog_migrate/e55b278641b86e234994fbb579cfd67a.png)

### 元数据表

spring batch框架进行元数据管理共有九张表，其中有三张表(后缀为SEQ)是用来分配主键的。

![image-20201107172832547](https://i-blog.csdnimg.cn/blog_migrate/9c09ebdca930275b1f898416753931ee.png)

![image-20201107172847856](https://i-blog.csdnimg.cn/blog_migrate/8dd036ca9d90bae22e8d5d8123f3a0f9.png)

![image-20201107172857804](https://i-blog.csdnimg.cn/blog_migrate/fc144fd25d6b70e1dd0d13bf77f0addf.png)

er关系：

![image-20201107172915306](https://i-blog.csdnimg.cn/blog_migrate/748809fd6ab7a7144cb3efb22140e593.png)

## Job Launcher

Job Launcher(作业调度器)是spring batch框架基础设施层提供的运行job的能力。通过给定的job名称和作业参数Job Parameters，可以通过Job Launcher执行Job。通过Job Launcher可以在Java 程序中调用批处理任务，也可以在通过命令行或者其他框架(xxl-job或者quartz等)。spring batch框架提供了Job Launcher的简单实现SimpleJobLauncher。

## ItemReader

ItemReader是Step中对资源的读处理，spring batch框架已经提供了多种类型的读实现，包括对文本文件，xml文件，数据库，jms消息等读的处理。直接使用spring batch框架提供的读组件可以快速地完成批处理应用的开发和搭建。

![image-20201107173657074](https://i-blog.csdnimg.cn/blog_migrate/9a394bac4dbbfd8117b8a7bd4cce7f66.png)

## ItemProcessor

ItemProcessor阶段表示 对读取的数据进行处理，开发者可以实现自己的业务操作对数据进行处理。

![image-20201107173802306](https://i-blog.csdnimg.cn/blog_migrate/c4bb0615464063be54e20fffe9aa0bb3.png)

process方法中，参数item是ItemReader读取的数据，返回值是交给ItemProcessor交给ItemWriter写的数据，在process方法中可以修改到的数据的值。如果返回值是null,表示忽略这次的数据。

## ItemWriter

ItemWriter是Step中对资源的写处理，spring batch框架已经提供了多种类型的写实现，包括对文本文件，xml文件，DB等写的处理。直接使用spring batch框架提供的写组件可以快速地完成用用的开发和搭建。

![image-20201107174134021](https://i-blog.csdnimg.cn/blog_migrate/3938532ea67ec1e9d6c6366bfe400a7a.png)