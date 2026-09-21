---
layout: post
title: "spring boot集成Hystrix"
date: 2020-03-02 19:47:18 +0800
categories: [hystrix, hystrix dashbd, hystrix-RestTP, hystix-feign, turbine]
description: "本文深入解析了Hystrix熔断器的工作原理及如何在SpringBoot中集成Hystrix，包括解决分布式系统中服务间依赖导致的故障级联问题，通过熔断机制、快速失败、故障恢复等策略增强系统稳定性。"
keywords: hystrix, hystrix dashbd, hystrix-RestTP, hystix-feign, turbine
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104595872
> - 发布时间：2020-03-02 19:47:18
> - 阅读量：968
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#hystrix, #hystrix dashbd, #hystrix-RestTP, #hystix-feign, #turbine

## 摘要

文章浏览阅读968次。本文深入解析了Hystrix熔断器的工作原理及如何在SpringBoot中集成Hystrix，包括解决分布式系统中服务间依赖导致的故障级联问题，通过熔断机制、快速失败、故障恢复等策略增强系统稳定性。

---

#### spring boot集成Hystrix

  * [1\. 什么是Hystrix](<#1_Hystrix_3>)
  * [2\. Hystrix解决了什么问题](<#2_Hystrix_5>)
  * [3\. Hystrix设计原则](<#3_Hystrix_10>)
  * [4\. Hystrix工作机制](<#4_Hystrix_17>)
  * [5\. RestTemplate和Ribbon使用Hystrix](<#5_RestTemplateRibbonHystrix_19>)
  *     * [5.1 创建项目](<#51__20>)
    * [5.2 配置](<#52__22>)
    * [5.3 添加注解](<#53__27>)
    * [5.4 创建Ribbon配置](<#54_Ribbon_29>)
    * [5.5 创建Ribbon Service](<#55_Ribbon_Service_31>)
    * [5.6 创建controller](<#56_controller_34>)
    * [5.7 验证](<#57__36>)
  * [6\. 在Feign上使用熔断器](<#6_Feign_52>)
  *     * [6.1 创建项目](<#61__53>)
    * [6.2 配置](<#62__55>)
    * [6.3 添加注解](<#63__57>)
    * [6.4 feign配置](<#64_feign_59>)
    * [6.5 feign调用](<#65_feign_61>)
    * [6.6 feign的hystrix处理](<#66_feignhystrix_63>)
    * [6.7 service](<#67_service_65>)
    * [6.8 controller](<#68_controller_67>)
    * [6.9 验证](<#69__75>)
  * [7\. RestTemplate和Feign对比](<#7_RestTemplateFeign_84>)
  * [8\. Hystrix Dashboard & RestTemplate](<#8_Hystrix_Dashboard__RestTemplate_91>)
  *     * [8.1 创建](<#81__92>)
    * [8.2 配置](<#82__94>)
    * [8.3 配置hystrix dashboard](<#83_hystrix_dashboard_96>)
    * [8.4 配置ribbon](<#84_ribbon_105>)
    * [8.5 service](<#85_service_107>)
    * [8.6 controller](<#86_controller_109>)
    * [8.7 注解](<#87__111>)
    * [8.8 启动](<#88__113>)
  * [9\. Hystrix Dashboard & Feign](<#9_Hystrix_Dashboard__Feign_126>)
  *     * [9.1 创建](<#91__127>)
    * [9.2 配置](<#92__129>)
    * [9.3 配置hystrix dashboard](<#93_hystrix_dashboard_131>)
    * [9.4 配置feign](<#94_feign_133>)
    * [9.5 dao.feign](<#95_daofeign_135>)
    * [9.6 hystrix.feign](<#96_hystrixfeign_137>)
    * [9.7 service](<#97_service_139>)
    * [9.8 controller](<#98_controller_141>)
    * [9.9 注解](<#99__143>)
    * [9.10 启动](<#910__145>)
  * [10\. Turbine聚合监控](<#10_Turbine_154>)
  *     * [10.1 创建](<#101__155>)
    * [10.2 配置](<#102__157>)
    * [10.3 启动](<#103__160>)

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. 什么是Hystrix

在分布式系统中，服务与服务之间的依赖错综复杂，一种不可避免的情况就是某些服务会出现故障，导致依赖于它们的其他服务出现远程调度的线程阻塞。Hystrix 是 Netflix公司开源的一个项目，它提供了熔断器功能，能够阻止分布式系统中出现联动故障。Hystrix 是通过隔离服务的访问点阻止联动故障的，并提供了故障的解决方案，从而提高了整个分布式系统的弹性。

## 2\. Hystrix解决了什么问题

在复杂的分布式系统中，可能有几十个服务相互依赖，这些服务由于某些原因，例如机房的不可靠性、网络服务商的不可靠性等，导致某个服务不可用。如果系统不隔离该不可用的服务，可能会导致整个系统不可用。  
在高并发的情况下，单个服务的延迟会导致整个请求都处于延迟状态，可能在几秒钟就使整个服务处于线程负载饱和的状态。  
某个服务的单个点的请求故障会导致用户的请求处于阻塞状态，最终的结果就是整个服务的线程资源消耗殆尽。由于服务的依赖性，会导致依赖于该故障服务的其他服务也处于线程阻塞状态，最终导致这些服务的线程资源消耗殆尽，直到不可用，从而导致整个问服务系统都不可用，即雪崩效应。  
为了防止雪崩效应，因而产生了熔断器模型。Hystrix 是在业界表现非常好的一个熔断器模型实现的开源组件，它是Spring Cloud 组件不可缺少的一部分。

## 3\. Hystrix设计原则

总的来说，Hystrix的设计原则如下。

  * 防止单个服务的故障耗尽整个服务的Servlet容器（例如Tomcat）的线程资源。
  * 快速失败机制，如果某个服务出现了故障，则调用该服务的请求快速失败，而不是线程等待。
  * 提供回退（fallback）方案，在请求发生故障时，提供设定好的回退方案。
  * 使用熔断机制，防止故障扩散到其他服务。
  * 提供熔断器的监控组件Hystrix Dashboard，可以实时监控熔断器的状态。

## 4\. Hystrix工作机制

首先，当服务的某个 API 接口的失败次数在一定时间内小于设定的阀值时，熔断器处于关闭状态，该 API接口正常提供服务。当该API 接口处理请求的失败次数大于设定的阀值时，Hystrix判定该API接口出现了故障，打开熔断器，这时请求该 API 接口会执行快速失败的逻辑（即 fallback 回退的逻辑），不执行业务逻辑，请求的线程不会处于阻塞状态。处于打开状态的熔断器，一段时间后会处于半打开状态，并将一定数量的请求执行正常逻辑。剩余的请求会执行快速失败，若执行正常逻辑的请求失败了，则熔断器继续打开；若成功了，则将熔断器关闭。这样熔断器就具有了自我像复的能力。

## 5\. RestTemplate和Ribbon使用Hystrix

### 5.1 创建项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9769dfe56a46f1b75e7d11732af02d46.png)

### 5.2 配置

配置服务名称，eureka server，日志等  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ea677e58d8d71952515f979613dba0b.png)  
配置eureka client服务提供者的连接名称  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6444559db9a0fd781ab0b3b389dd5783.png)

### 5.3 添加注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c25416908d9f70cb51c9c74e7fc3340c.png)

### 5.4 创建Ribbon配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/84f55e9392b5bcc7f99f35135e469f49.png)

### 5.5 创建Ribbon Service

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce88e8609c94cca5e5c8e7b74175d06f.png)  
当Hystrix认为eureka client的服务提供者提供的服务不可用时，就会访问fallbackMethod的方法

### 5.6 创建controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a47db7e814860affed226ee5190d9e9.png)

### 5.7 验证

首先启动eureka server  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/11a3dd530e39850c6d2c2ff613b13878.png)  
然后启动本项目  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b6b928437956851aa2630008be6e9a4d.png)  
注意，此时没有启动eureka client服务提供者，那么服务是不可用的。  
访问Hystrix的接口，会调用service里面的fallbackMethod的方法。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/49e08451a27dec876c96afcdce5af0a3.png)  
接下来启动eureka client 服务提供者。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a60cbcd2252da789e335e707cdf793be.png)  
此时服务可达，所以，此时应该能够正确的访问的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d989976c98c829fd283cf4df32793d25.png)  
当然，需要多试试，才能成功。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eedcd8a90bb98910f20e29deec22e50b.png)  
这就是Hystrix的自我修复。  
将熔断器设置为半开状态，尝试请求，成功就将服务设置可用，否则继续熔断。

## 6\. 在Feign上使用熔断器

### 6.1 创建项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fd0e5c884c921ab60c3f2785c5e95e89.png)

### 6.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2004d8069f38ccad75d27ea5d766e5bb.png)

### 6.3 添加注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/443844e34a2259ebe83e8d39955aa46b.png)

### 6.4 feign配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a9fb5ea61a4c06900b1734701d9a9c14.png)

### 6.5 feign调用

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/15b8a5ba1ad567c011bb345af3384c96.png)

### 6.6 feign的hystrix处理

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/94511a24286e9813eff3e36c1b4eebcd.png)

### 6.7 service

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bcdfd247f81c790047a2ab8ec78f4a04.png)

### 6.8 controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ede1fb653ba8359f5d8fb5b7cd286ef.png)

开启eureka client和feign  
因为feign已经引入了hystrix的依赖，所以我们这里开启就行。  
因为feign调用时写的是接口，而hystrix的熔断调用的方法就是实现了feign调用的接口的类。  
同时这些类和接口需要被spring管理。  
在feign调用的接口需要指定熔断处理类…

### 6.9 验证

启动，首先需要启动eureka-server和eureka client服务提供者  
接着启动feign-hystrix(也就是feign)  
访问：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cd98735f295efd2583176235453e57e7.png)  
关闭eureka client服务提供者  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/976706c7fdabdaf3281fe5f17f0ce383.png)  
就去调用熔断处理的方法了。

## 7\. RestTemplate和Feign对比

两者都能实现熔断处理。不过feign比RestTemplate更好。

  1. RestTemplate是使用硬编码指定熔断处理方法的，而feign则是指定类
  2. RestTemplate熔断处理方法没有限制，而feign则是实现接口，其方法已被定义
  3. RestTemplate请求单一，而feign有HttpUrlConnection,HttpClient,OkHttp多种方式
  4. RestTemplate需要自己增加依赖，而feign已集成，无需管理
  5. RestTemplate学习成本小于feign

## 8\. Hystrix Dashboard & RestTemplate

### 8.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0e05b751c91fa092b0e96c463572f12.png)

### 8.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a174acf34eee18d1ff12fcdd81c1b9c5.png)

### 8.3 配置hystrix dashboard

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2dab796437de34a2a9c8734fe284aede.png)  
注意：  
这里的urlMapping就是熔断器的元数据访问地址，如果不配置，会无法访问导致异常。  
当然也可以添加多个，比如  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eba45ec6009231e459735e9c72fbd2ee.png)  
因为内部是一个list，不存在覆盖的问题  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4563faecccd89ecf280cd0c6e5501591.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/530ddf7850c9f71e67b9b5eba1a1cbe3.png)

### 8.4 配置ribbon

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/79314bf83a7d9d611567eada7673e272.png)

### 8.5 service

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dfd537664f368f26949a2dd7b12a0814.png)

### 8.6 controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c0a930601a814d7017fd00d48b9a6aff.png)

### 8.7 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff3536233e094cede4987f4237a6455f.png)

### 8.8 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a6f0076d931479ea6df93a031ba905b9.png)  
注意，需要先启动eureka server以及eureka client服务提供者。  
刚开始没有访问任何服务，此时eureka client还未获取eureka server 服务列表。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f255543ac0e71b5f0d82e4f3749660f8.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b9bf88e86680f326328d1025e8bd25f.png)  
接着访问hystrix dashboard的主界面  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c995b4c5245478faec5c88156ae14721.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/26133fd4c2c53dd5f190f59e602ecdbf.png)  
接着访问：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d0551eb619e3a68cf1ada306120b0d78.png)  
其实就是配置的url实际上是等价的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/73c0ef886db43043934401e42319b2a8.png)

## 9\. Hystrix Dashboard & Feign

### 9.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/58478112c01895e5bae375c16b13f22c.png)

### 9.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5dba75ec1324489f8c99a80a8d3a4db1.png)

### 9.3 配置hystrix dashboard

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f60be8e32c55ec379f7d382427a8e3f9.png)

### 9.4 配置feign

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ae50b23dd39e48b62592003db9a891fa.png)

### 9.5 dao.feign

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0dd4e0cefaf941d5278e262234d22f5a.png)

### 9.6 hystrix.feign

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b10c5ed97933927125dac4fbc8036e1c.png)

### 9.7 service

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b6abf354f93d932b712a920a02929513.png)

### 9.8 controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5bc5a8106eab95dc410a54377769a4a4.png)

### 9.9 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3255dea8924a188879c2d2365e2d450b.png)

### 9.10 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c2a5e20b7e37c7ee6306100ab48bde54.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4ed7bcb92ca379de8e673eb3c5cbf19b.png)  
访问  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ba1e2a2868d93972d7f4777cc53f053.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/20dc795cc96e3d3a20eb335eaaf19e23.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec1db1cdaadd3a8886ad2d1e0886b13a.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6c6a604983e27837c910c3ffee60ef57.png)  
这个就是元数据访问不到。

## 10\. Turbine聚合监控

### 10.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ca26d223905e4e99c2dbeb1840cae747.png)

### 10.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d7bc17289e649228aab362277f228de.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0484441e33414d38d8098a01c35b4c38.png)

### 10.3 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e2a353ddcb0da1db0a65e67568e56420.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/84b3478cc7db4e2a9a8e163e8178db45.png)  
访问  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6685f7aaefeed908f002e7dce90d6e89.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6289260365694521bb3bd57d1060aead.png)  
将hystrix dashboard需要两页的监控图像放到了一个页面上。