---
layout: post
title: "spring boot 负载均衡Ribbon"
date: 2020-02-25 19:12:20 +0800
categories: [boot整合Ribbon, RestTemplate, eureka服务消费, LoadBanlcClt, Ribbon原理]
description: "本文详细介绍SpringBoot中Ribbon负载均衡组件的使用方法，包括RestTemplate与Ribbon结合实现服务消费，以及如何通过Ribbon配置负载均衡策略。通过实例演示了如何启动Eureka Server，多实例启动Eureka Client，验证Eureka服务，创建Ribbon模块，配置Ribbon，以及使用本地Server List。"
keywords: boot整合Ribbon, RestTemplate, eureka服务消费, LoadBanlcClt, Ribbon原理
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104482835
> - 发布时间：2020-02-25 19:12:20
> - 阅读量：2
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#boot整合Ribbon, #RestTemplate, #eureka服务消费, #LoadBanlcClt, #Ribbon原理

## 摘要

文章浏览阅读2.1k次。本文详细介绍SpringBoot中Ribbon负载均衡组件的使用方法，包括RestTemplate与Ribbon结合实现服务消费，以及如何通过Ribbon配置负载均衡策略。通过实例演示了如何启动Eureka Server，多实例启动Eureka Client，验证Eureka服务，创建Ribbon模块，配置Ribbon，以及使用本地Server List。

---

#### spring boot 负载均衡Ribbon

  * 1\. RestTemplate简介
  * 2\. Ribbon简介
  * 3\. 实例--使用RestTemplate和Ribbon消费服务
  *     * 3.1 启动eureka server
    * 3.2 多实例启动eureka client
    * 3.3 验证eureka
    * 3.4 创建 Ribbon模块
    * 3.5 配置
    * 3.6 创建Ribbon Config类
    * 3.7 创建service
    * 3.8 service test
    * 3.9 创建controller
    * 3.10 controller test
    * 3.11 启动
  * 4\. LoadBalancerClent
  * 5\. 本地serverList

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. RestTemplate简介

RestTemplate 是 Spring Resources 中一个访问第三方RESTful API 接口的网络请求框架。RestTemplate 的设计原则和其他 Spring Template（例如JdbcTemplate、JmsTemplate）类似，都是为执行复杂任务提供了一个具有默认行为的简单方法。  
RestTemplate 是用来消费 REST 服务的，所以 RestTemplate 的主要方法都与REST的 Http协议的一些方法紧密相连，例如 HEAD、GET、POST、PUT、DELETE和 OPTIONS 等方法，这些方法在 RestTemplate 类对应的方法为 headForHeaders()、getForObject()、postForObject()、put()和delete()等。  
我们在写测试方法时用到的TestRestTemplate  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/24c30421cb3c174d46c06ed7965b1df8.png)  
就是用RestTemplate实现的  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/75e2e25a8a4c05fdd89e94c12a0baa61.png)

## 2\. Ribbon简介

负载均衡是指将负载分摊到多个执行单元上，常见的负载均衡有两种方式。一种是独立进程单元，通过负载均衡策略，将请求转发到不同的执行单元上，例如Ngnix。另一种是将负载均衡逻辑以代码的形式封装到服务消费者的客户端上，服务消费者客户端维护了一份服务提供者的信息列表，有了信息列表，通过负载均衡策略将请求分摊给多个服务提供者，从而达到负载均衡的目的。  
Ribbon是 Netflix 公司开源的一个负载均衡的组件，它属于上述的第二种方式，是将负载均衡逻辑封装在客户端中，并且运行在客户端的进程里。Ribbon是一个经过了云端测试的IPC库，可以很好地控制HTTP和TCP客户端的负载均衡行为。  
在Spring Cloud 构建的微服务系统中，Ribbon作为服务消费者的负载均衡器，有两种使用方式，一种是和RestTemplate相结合，另一种是和 Feign相结合。  
Ribbon有很多子模块，但很多模块没有用于生产环境，目前Netilix 公司用于生产环境的Ribbon子模块如下。

  * ribbon-loadbalancer：可以独立使用或与其他模块一起使用的负载均衡器API。
  * ribbon-cureka:Ribbon 结合 Eureka 客户端的API，为负载均衡器提供动态服务注册列  
表信息。
  * ribbon-core：Ribbon的核心API。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1992cee39bc11fe4d7f9d39bcc14c062.png)  
不知是否还有人维护？  
积累了140多个issues没人处理  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/08e53614834662612fb195f5f5e0dd5f.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/627ba31a2513f367f54ac2a4744cd458.png)

## 3\. 实例–使用RestTemplate和Ribbon消费服务

### 3.1 启动eureka server

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/aa9fadc89d73265d87280d1b30b44382.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/22519fb9d28a186b88c678dfdc6abc5f.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a91d6d3a7cf6a21d9a35d05900994e95.png)

### 3.2 多实例启动eureka client

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/85a624db95df1d983564083b40728334.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/107caaacf1b9c1899d27bcee485763d6.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b1360b87619b7a31aca2313a33028010.png)

### 3.3 验证eureka

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5de192481f4dd8b7fecbb980692cc41a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/66f06f33b17c103d8fc97534a0627421.png)

### 3.4 创建 Ribbon模块

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e835b0707f61e43080f448527042eec2.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9ff3743597fedcf86b094377e044ec3e.png)  
项目结构  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/182e5c4ca83e34d90b29135abf037fce.png)

### 3.5 配置

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1b65880a1f5dd880618c084ef32ce98f.png)

### 3.6 创建Ribbon Config类

只需要在程序的IoC容器中注入一个restTemplate的Bean，并在这个Bean上加上@LoadBalanced注解，此时RestTemplate就结合Ribbon开启了负载均衡。(为什么？负载均衡策略有哪些？存疑)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/343e4eaf48796e6b12bed8dd996c98dc.png)

### 3.7 创建service

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0a76192ae40740f7d65a84f2c0bbb294.png)

### 3.8 service test

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/be88abb3ebdd5e00a8db5f3dc57ff28b.png)  
结果  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bbec5c36173caef5f80cc52965b29d9c.png)  
默认是轮询策略。

### 3.9 创建controller

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/36e582a9241edcc7c3052910d5481e69.png)

### 3.10 controller test

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/298d81baff5be0e8d7363481d30f58cd.png)  
结果  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/22c2ea574ebd2e738fc5bc5c400cf848.png)

### 3.11 启动

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/df932ebe0e85ad5ea4d60969f2c4ac59.png)  
访问  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/47a0af9409619f83d288a5d534dcf11f.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f126c3f0376f60f7cdd018457be854fc.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ca0d8ce82a3f02c7ae72b7576dd4a8ba.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6d31d383f32c17578efed442f1988f6b.png)

## 4\. LoadBalancerClent

负载均衡器的核心类为LoadBalancerClient,它可以获取负载均衡的服务提供者的实例信息。  
我们上述的项目基础上，在创建一个模块。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/76679c2127d69075b34429df2c7a026a.png)  
如下配置  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7913abddce6ce3b7a2105b2242e298f1.png)  
写一个controller  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c2c2da9af8bfd2868d42030646ee0ca6.png)  
打上断点，调试启动。  
不要忘记这里：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/dfcc480b5dfa87c5458c42d48fed4eda.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2ed2750b9d8214e9056e093fbefa902f.png)  
查看choose方法  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8db1051d5e8862c5c008f892d8600e72.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/388a6cf3d32a3ec39b112757f5bb01c3.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/50b4857d63e043483c21d374ed938710.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/74d4b97474353ba8146cea2202d20804.png)  
这里getserver，打上断点，继续。  
首先看getLoadBalancer  
进入打断点  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6cba60bcea5b467fb535b650cba3f92b.png)  
接下来  
getLoadBalancer  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/827fc2bd7e70e8353bb8b8c291ecfb38.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/202428dfc6245de69e99349605b24ea2.png)  
通过反射，获取IClientConfig.  
接下来是  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e344b4efccf7cb4f442c154f451c4712.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e798cf0a9b02c69d0c21166962a3c708.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0d037281211030150cd79c48b25c81a8.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fc85db191280d51f4b6540964efc0c70.png)  
loadBalancer有什么？  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4890d9168b43e49ef42647ff34e3507d.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/86d4fd17364b89faf5bf71f3fdc47b39.png)  
从上述图片看出，本次请求应该是8764  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/030b44d7d3062600f196088ea7d9f0cb.png)  
看下allServers  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/642f8ef721969057b5812c1057101dfa.png)  
是个接口，其实现有  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/174d3056be5a32af049b9197ce35415a.png)  
有四个实现类，本次使用从调试信息看是使用第二个实现类。猜测这里应该是不同的策略。

## 5\. 本地serverList

在4的基础上进行修改：  
设置不连接eureka server  
且定义stores，这个stores就是我们之前的eureka-client 的url  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/34d0f8dd23624c68de736ebb7f324505.png)  
接着修改controller  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1d6376f53cbcf009f11721e54f2a0ba4.png)  
然后运行：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/968738cd3e4ee7b0f04577cbb2b542b0.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/270a4f3c55489715c2203260ca95e2ff.png)  
这里的list就是我们配置的本地的list
