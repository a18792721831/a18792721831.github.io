---
layout: post
title: "spring boot集成Eureka服务注册与发现"
date: 2020-02-22 16:24:01 +0800
categories: [boot集成eureka, eureka server, eureka client, eureka机制, eureka入门]
description: "本文介绍如何在SpringBoot中集成Eureka实现服务注册与发现，涵盖EurekaServer搭建、Gradle配置、EurekaClient注册流程及服务发布。Eureka简化了微服务架构中的服务发现过程。"
keywords: boot集成eureka, eureka server, eureka client, eureka机制, eureka入门
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104444686
> - 发布时间：2020-02-22 16:24:01
> - 阅读量：952
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#boot集成eureka, #eureka server, #eureka client, #eureka机制, #eureka入门

## 摘要

文章浏览阅读952次。本文介绍如何在SpringBoot中集成Eureka实现服务注册与发现，涵盖EurekaServer搭建、Gradle配置、EurekaClient注册流程及服务发布。Eureka简化了微服务架构中的服务发现过程。

---

#### springboot集成Eureka服务注册与发现

  * 1\. Eureka简介
  *     * 1.1 什么是Eureka
    * 1.2 Eureka的基本架构
  * 2\. Eureka Server
  *     * 2.1 创建Eureka Server
    * 2.2 配置gradle
    * 2.3 配置Eureka
    * 2.4 启动eureka server
  * 3\. Eureka Client
  *     * 3.1 创建 Eureka Client
    * 3.2 配置Eureka
    * 3.3 启动eureka client
    * 3.4 eureka client 服务发布者
  * 4\. eureka的一点思考

  
git 地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. Eureka简介

### 1.1 什么是Eureka

和 Consul、Zookeeper类似，Eureka 是一个用于服务注册和发现的组件，最开始主要应用于亚马逊公司旗下的云计算服务平台 AWS。Eureka 分为 Eureka Server和 Eureka Client,Eureka Server 为Eureka 服务注册中心，Eureka Client 为 Eureka客户端。

### 1.2 Eureka的基本架构

Eureka的基本架构主要包括以下3种角色。

  * Register Service：服务注册中心，它是一个Eureka Server，提供服务注册和发现的功能。
  * Provider Service：服务提供者，它是一个 Eureka Client，提供服务。
  * Consumer Service：服务消费者，它是一个Eureka Cient，消费服务。

服务消费的基本过程如下：首先需要一个服务注册中心 Eureka Server，服务提供者 Eureka  
Client 向服务注册中心 Eureka Server注册，将自己的信息（比如服务名和服务的IP地址等）  
通过 REST API 的形式提交给服务注册中心 Eureka Server。同样，服务消费者 Eureka Client 也  
向服务注册中心 Eureka Server注册，同时服务消费者获取一份服务注册列表的信息，该列表  
包含了所有向服务注册中心 Eureka Server注册的服务信息。获取服务注册列表信息之后，服  
务消费者就知道服务提供者的IP地址，可以通过Http远程调度来消费服务提供者的服务。

## 2\. Eureka Server

### 2.1 创建Eureka Server

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9e5b0facc230a918e4d42663dffc4867.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/db6945df35533c790272c5525de4f511.png)

### 2.2 配置gradle

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e9b83c2640b361783283b52039f841d9.png)  
其项目结构如上图，.gradle和build的文件夹不需要进行手动创建。  
我们使用gradle warpper  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8d4a1a2fc18086ee82c7c7db01a61d74.png)  
然后修改maven仓库地址
    
    
    repositories {
        maven{
            url 'https://maven.aliyun.com/'
        }
        maven{
            url 'http://maven.aliyun.com/nexus/content/groups/public/'
        }
        maven{
            url 'https://repo1.maven.org/maven2/'
        }
        mavenCentral()
    }
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cf1587d6e3b24ebb587fadfdf5e8fad3.png)  
然后等待重新构建(刚创建成功，gradle会从maven的默认仓库下载依赖，此时下载非常慢，可以手动终止，等待我们添加了其他的仓库后重新刷新下载依赖。效果很明显，使用默认仓库下载一个jar在十几几十秒，但是使用国内的仓库，下载一个jar包只需要不到1秒)

### 2.3 配置Eureka

创建配置文件application.yml,在resources下  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e1123d3dc9bea8428da8890eb40f5a69.png)
    
    
    server:
      port: 8761
    
    eureka:
      instance:
        hostname: 127.0.0.1
      client:
        register-with-eureka: false
        fetch-registry: false
        service-url:
          defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      server:
        enable-self-preservation: false
    
    logging:
      level:
        org:
          springframework:
            web:
              servlet:
                mvc:
                  method:
                    annotation:
                      RequestMappingHandlerMapping: trace
    
    spring:
      freemarker:
        template-loader-path: classpath:/templates/
        prefer-file-system-access: false
    

  * 8761是端口(tomcat启动的端口，对外服务的端口)
  * 127.0.0.1表示eureka访问的域名
  * 因为我们构建的是eureka服务端，所以，eureka Server不需要进行注册，而是eureka Client向eureka Server 进行注册的，所以需要关闭注册。即 register-with-eureka和fetch-registry为false
  * defaultZone是eureka主面板访问地址。其值进行变量替换后就是http://127.0.0.1:8761/eureka/
  * enable-self-preservation设置为false是关闭其自我保护机制(后面有说明)
  * logging是配置tomcat日志打印级别，默认打印信息较少，无法打印tomcat容器发布了哪些接口，但是设置为较详细的日志级别，可以打印发布哪些接口，这样就可以从日志中看出我们的controller是否发布成功
  * feemarker是重中之重，因为eureka刚创建成功时，去访问主面板是无法访问的，从官网的issues看，是认为gradle的缓存问题。不过网上有人说增加这些配置，可以解决这一问题。我没有深入，只是配置这个之后，重新刷新gradle构建，确实可以访问了。  
接下来在启动类增加eureka server的注解  
@EnableEurekaServer  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/04e6269602241ac6d7eb6cabcf708d85.png)

### 2.4 启动eureka server

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3cdabdbb946cebc67b71909ff0004c7d.png)  
然后在浏览器验证  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a41c9a552af0599cce8753b7e2ac3ade.png)  
提示没有开启自我保护机制，而且，其注册的eureka client也是空的。

## 3\. Eureka Client

### 3.1 创建 Eureka Client

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6dbf5da679ebd06cd66272e784a89aae.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4f71b7c6eede48bf23dfaca364329235.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/39190aebae5779616c9c9207d108f41d.png)  
然后与2.2同样进行配置gradle(这里其实可以将仓库配置到root的gradle中，但是貌似不生效，不知道为什么，存疑，后续研究gradle时解决。)

### 3.2 配置Eureka

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b0de7143f4ea7f43db4834fb8723017b.png)
    
    
    server:
      port: 8762
    
    eureka:
      client:
    #    register-with-eureka: false
    #    fetch-registry: false
        service-url:
          defaultZone: http://127.0.0.1:8761/eureka/
    
    
    logging:
      level:
        org:
          springframework:
            web:
              servlet:
                mvc:
                  method:
                    annotation:
                      RequestMappingHandlerMapping: trace
    
    spring:
      freemarker:
        template-loader-path: classpath:/templates/
        prefer-file-system-access: false
      application:
        name: eureka-client-test
    

  * 在 eureka client中需要注销注册，默认开启，因为eureka client需要向eureka server进行注册的。
  * defaultZone就是eureka server的注册地址
  * application:name是eureka client在eureka server面板中展示的名称  
其余配置与eureka server配置相同。  
当然，其注解是client  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7a3de2fdc313eb2a726c81996abab094.png)

### 3.3 启动eureka client

请注意，如果需要同时启动多个tomcat容器在一个idea中，需要在run dash board面板中。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7d85cf941e6ad56eb2aa8c0004921ce6.png)  
正常情况下，会自动弹出提示，配置是否展示run dashboard。如果没有弹出，请百度。  
此时刷新eureka面板  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2a5a818646e0d20cdd257f96b24a8ddb.png)

### 3.4 eureka client 服务发布者

在eureka中有三个角色：  
eureka server  
eureka client 服务发布者（服务提供者）  
eureka client 服务消费则（服务调用者）

我们创建了eureka server和一个eureka client，并且需要将这个eureka client作为服务提供者，对外提供接口。  
所以，我们需要创建controller  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/dc12a68514cc5911b3b9393bd7996623.png)  
controller提供了两个接口，分别是hi和hello接口，返回String，并且get访问  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3a500ad91bbbfc4bc0cb6383add76823.png)  
接口自测  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2cdc6996284a0ffa4964f38b2e36255d.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1c62290520b5cb209020857aa5fd8ff5.png)

## 4\. eureka的一点思考

在学习eureka的时候，我学习到这里，其实有一个疑问，我们创建了eureka server，eureka client并且提供了两个接口。那么，eureka cleint的调用者呢。  
就是我们创建了对外接口，并且将对外接口以及接口服务器的信息放到了eureka server中，且所有的eureka集群都有这些信息，那么是怎么调用的呢？  
这一块在后面的一个框架中，这个框架暂时实现了，提供一个平台，用于服务器的注册，以及相关信息的记录，并保证所有的服务器信息共享。  
调用的是Ribbon，Ribbon使用服务器信息进行远程调用，当然，远远不止，一些网关路由，负载均衡，熔断机制，都和服务注册与发现有关。  
这里存疑。
