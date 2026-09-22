---
layout: post
title: "spring boot 集成Zuul"
date: 2020-03-05 11:44:43 +0800
categories: [Zuul集成, Zuul自定义转发列表, Zuul熔断器, Zuul服务降级, Zuul自定义过滤器]
description: "本文详细介绍了Zuul作为路由网关组件在微服务架构中的重要作用，包括智能路由、负载均衡、API聚合与保护、用户认证、监控及流量控制等功能。深入解析Zuul的工作原理，涵盖PRE、ROUTING、POST和ERROR四种过滤器类型，以及它们在请求处理流程中的角色。同时，提供了Spring Boot集成Zuul的具体步骤，包括创建、配置、注解和启动过程，以及如何配置API接口版本号、熔断器和自定义转发列表。"
keywords: Zuul集成, Zuul自定义转发列表, Zuul熔断器, Zuul服务降级, Zuul自定义过滤器
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104635350
> - 发布时间：2020-03-05 11:44:43
> - 阅读量：4
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#Zuul集成, #Zuul自定义转发列表, #Zuul熔断器, #Zuul服务降级, #Zuul自定义过滤器

## 摘要

文章浏览阅读4k次。本文详细介绍了Zuul作为路由网关组件在微服务架构中的重要作用，包括智能路由、负载均衡、API聚合与保护、用户认证、监控及流量控制等功能。深入解析Zuul的工作原理，涵盖PRE、ROUTING、POST和ERROR四种过滤器类型，以及它们在请求处理流程中的角色。同时，提供了Spring Boot集成Zuul的具体步骤，包括创建、配置、注解和启动过程，以及如何配置API接口版本号、熔断器和自定义转发列表。

---

#### spring boot 集成Zuul

  * 1\. 为什么需要Zuul
  * 2\. Zuul的工作原理
  * 3\. spring boot集成
  *     * 3.1 创建
    * 3.2 配置
    * 3.3 注解
    * 3.4 启动
    * 3.5 指定url转发
    * 3.6 自定义转发列表
  * 4\. Zuul配置API接口版本号
  * 5\. 在Zuul上配置熔断器
  * 6\. 在Zuul中使用过滤器

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. 为什么需要Zuul

Zuul 作为路由网关组件，在微服务架构中有着非常重要的作用，主要体现在以下6个方面。

  * Zuul、Ribbon 以及Eureka 相结合，可以实现智能路由和负载均衡的功能，Zuul能够将请求流量按某种策略分发到集群状态的多个服务实例。
  * 网关将所有服务的API接口统一聚合，并统一对外暴露。外界系统调用API接口时，都是由网关对外暴露的 API 接口，外界系统不需要知道微服务系统中各服务相互调用的复杂性。微服务系统也保护了其内部微服务单元的 API 接口，防止其被外界直接调用，导致服务的敏感信息对外暴露。
  * 网关服务可以做用户身份认证和权限认证，防止非法请求操作API接口，对服务器起到保护作用。
  * 网关可以实现监控功能，实时日志输出，对请求进行记录。
  * 网关可以用来实现流量监控，在高流量的情况下，对服务进行降级。
  * API接口从内部服务分离出来，方便做测试。

## 2\. Zuul的工作原理

Zuul 是通过 Servlet 来实现的，Zuul 通过自定义的 ZuulServlet(类似于 Spring MVC 的DispatchServlet)米对请求进行控制。Zaul的核心是一系列过遗器，可以在Http请求的发起和响应返回期间执行一系列的过滤器。Zuu包括以下4种过滤器。

  * PRE 过滤器：它是在请求路由到具体的服务之前执行的，这种类型的过滤器可以做安全验证，例如身份验证、参数验证等。
  * ROUTING过滤器：它用于将请求路由到具体的微服务实例。在默认情况下，它使用Http Client 进行网络请求。
  * POST过滤器：它是在请求已被路由到微服务后执行的。一般情况下，用作收集统计信息、指标，以及将响应传输到客户端。
  * ERROR过滤器：它是在其他过滤器发生错误时执行的。  
Zuul采取了动态读取、编译和运行这些过滤器。过滤器之间不能直接相互通信，而是通过RequestContext 对象来共享数据，每个请求都会创建一个RequestContext对象。Zuul过滤器具有以下关键特性。
  * Type（类型）：Zuul过滤器的类型，这个类型决定了过滤器在请求的哪个阶段起作用，例如Pre、Post阶段等。
  * Execution Order（执行顺序）：规定了过滤器的执行顺序，Order的值越小，越先执行。
  * Criteria（标准）：Filter执行所需的条件。
  * Action（行动）：如果符合执行条件，则执行Action（即逻辑代码）。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7a5c0bf346a3c2f8bdcfc264fe3c323e.png)  
当一个客户端 Request请求进入Zuul网关服务时，网关先进入“prefilter”，进行一系列的验证、操作或者判断。然后交给“routing filter”进行路由转发，转发到具体的服务实例进行逻辑处理、返回数据。当具体的服务处理完后，最后由“post filter”进行处理，该类型的处理器处理完之后，将Response信息返回给客户端。  
ZuulServlet 是 Zuul 的核心 Servlet。ZuulServlet 的作用是初始化 ZuulFiher，并编排这些ZmulFiher的执行顺序。该类中有一个service(方法，执行了过滤器执行的逻辑。  
首先执行preRoute()方法，这个方法执行的是PRE类型的过滤器的逻辑。如果执行这个方法时出错了，那么会执行 error(e）和 postRoute(）。然后执行route(）方法，该方法是执行ROUTING类型过滤器的逻辑。最后执行 postRoute()，该方法执行了 POST类型过滤器的逻辑。

## 3\. spring boot集成

### 3.1 创建

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/48a5d377d9248afc148b6aac7637ce13.png)

### 3.2 配置

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/14738675b16ca37c873d715d340370f9.png)

### 3.3 注解

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/559cb996003df90e3c4b923d539b88d8.png)

### 3.4 启动

启动eureka-server、eureka-client多个、ribbon、feign、zuul。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3ab7641522bc7596156d18effe248346.png)  
访问直连  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1c4814ad994e090c5b091a5b516ff54b.png)  
再次请求  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7641aa8143db0bea032a5cec5fc2a9b7.png)  
说明zuul路由转发也做了负载均衡。  
访问ribbon  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6860016e4ef45ebe8d0ad1b00a2d5840.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bead2b8617af9427342153670aa785d7.png)  
访问feign  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ce504b8f67b282bd8de08459bdd3aa59.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3dc7103634221846d68c409355adbda4.png)

### 3.5 指定url转发

增加配置  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c8d4f150093aad11bd0f48aac82af59e.png)  
验证  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/46e88b3e72dcc21c30f78e3cfda79502.png)  
多次访问依然是同一个  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/998de9a86ceb1dc607f60dadd631b395.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ef78bca710c4abc6002967b6e68f72d2.png)

### 3.6 自定义转发列表

增加配置  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/607ce4c182af4ac9c22acd2d47a864cd.png)  
启动验证  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/021b77c551fe4cdb1efaaef9c1f9c99a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ec606716eaea1ae1f2c95a8c9edabd2c.png)  
多次访问依然是指定的列表。

## 4\. Zuul配置API接口版本号

配置  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/008d4dc726cb077f429e7509d04e8cc6.png)  
启动验证  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7a0889291823332935f7b1e101ef2f9e.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c6405cf929f8552daa8cda3c0eee8425.png)  
不加版本号是无法访问的。

## 5\. 在Zuul上配置熔断器

在Zuul中实现熔断功能需要实现FallbackProvider的接口。该接口主要有两个方法：一个是匹配的url的getRounte()方法，用于指定熔断功能应用于哪些路由的服务；另一个是fallbackResponse()方法，是当目标方法不可用时的处理方法。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/caa330624ab9ee3d0545feb12580a416.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/10e98fd62c80433a773adbeafea4a0a2.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/623ce8552ff3fc13f1498e3104845734.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/905c9dcc2446a747d525111ce05f89ba.png)  
启动验证  
这里有一个坑，配置了url的路由无法进行熔断，会直接异常，截止文章发表，未找到直接证据，但是根据步骤可以复现。  
具体见  
https://blog.csdn.net/a18792721831/article/details/104655867  
https://github.com/Netflix/zuul/issues/737  
首先两个服务都能访问  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/93b0ab878f17b3b5193f96f91eea3477.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/02ee33f3fad90fcb2383f5f4bc98ac47.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2c3e17fde0e313943d03620aa93c6b5a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a6709112464403518149007b6706f0c0.png)  
接着关闭eureka-client、ribbon、feign。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d04391887a2d5001eef95410ff7647c2.png)  
然后访问  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/54509a9ce3ebc313192f4af167fe5876.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8b57d02c7166341773b1181d5232012f.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b6d5902a806c49705ff14dc30b0323f0.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6c03b12cedebf531037065328e714e79.png)

## 6\. 在Zuul中使用过滤器

如何实现自定义过滤，以及在自定义过滤中加入业务逻辑。  
实现过滤器很简单，只需要继承ZuulFilter，并实现ZuulFilter中的抽象方法，包括filterType()和filterOrder()，以及IZuulFilter的 shouldFilter()和Object run()的两个方法。其中，filterType()即过滤器的类型，它有4种类型，分别是“pre”“post”“routing”和“error”。filterOrder是过滤顺序，它为一个Int类型的值，值越小，越早执行该过滤器。  
shouldFilter()表示该过滤器是否过滤逻辑，如果为true，则执行run()方法；如果为false，则不执行run(）方法。run()方法写具体的过滤的逻辑。  
首先创建自定义的过滤器  
PRE过滤器  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/10f7c2ac4842b5e244de44243d07ad62.png)  
post  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9f201890adf6af115f5049cfdf64719b.png)  
ps:Routing和error未验证触发(error直接转发到/error了，routing直接进入eureka-client中了)  
注意：使用getOutputStream，如果使用getWriter会有异常getWriter() has already been called for this response  
启动验证  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4c6b2e0ea21407f3f6b56d2b76b90fc1.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b21496b89c3ac46c317d96ff3b74c93e.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5a8a61f6027be893849d3d92eca784d8.png)  
日志  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4a0285603e166ca6e27f93f04339eeda.png)
