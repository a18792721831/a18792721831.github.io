---
layout: post
title: "spring boot 运行时监控actuator"
date: 2020-02-17 20:35:22 +0800
categories: [spring boot 监控, actuator, boot actuator, springboot监控使用, spring boot监控配置]
description: "spring boot 运行时监控actuator1. 如何引入actuator2. 加载详细信息3. actuator api介绍git地址1. 如何引入actuator新建一个spring web项目，然后，在gradle依赖中增加	implementation 'org.springframework.boot:spring-boot-starter-actuator'    im..._implementation 'org.springframework.boot:spring-boot-starter-actuator"
keywords: spring boot 监控, actuator, boot actuator, springboot监控使用, spring boot监控配置
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104363642
> - 发布时间：2020-02-17 20:35:22
> - 阅读量：431
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#spring boot 监控, #actuator, #boot actuator, #springboot监控使用, #spring boot监控配置

## 摘要

文章浏览阅读431次。spring boot 运行时监控actuator1. 如何引入actuator2. 加载详细信息3. actuator api介绍git地址1. 如何引入actuator新建一个spring web项目，然后，在gradle依赖中增加	implementation 'org.springframework.boot:spring-boot-starter-actuator'    im..._implementation 'org.springframework.boot:spring-boot-starter-actuator

---

#### spring boot 运行时监控actuator

  * [1\. 如何引入actuator](<#1_actuator_3>)
  * [2\. 加载详细信息](<#2__18>)
  * [3\. actuator api介绍](<#3_actuator_api_24>)

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. 如何引入actuator

新建一个spring web项目，然后，在gradle依赖中增加
    
    
    	implementation 'org.springframework.boot:spring-boot-starter-actuator'
        implementation 'org.springframework.boot:spring-boot-configuration-processor'
        providedCompile 'org.projectlombok:lombok'
    

等gradle加载完依赖构建后，启动：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/513e2837033daf964b2e009178fc8e4a.png)  
然后访问http://localhost:8080/acruator/  
将会得到  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8e95c7047c9f7ade75cce0b88a03c353.png)  
接着访问http://localhost:8080/actuator/health  
得到  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce7a8efbc15ad2742ca6eb8955be0cd2.png)

## 2\. 加载详细信息

在application.yml中增加  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fd1a4aef43459391fbd016361fb9753f.png)  
启动  
重新访问http://localhost:8080/actuator/health  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/62d00a1a4864b865c8755df4a85f0345.png)

## 3\. actuator api介绍

Actuator提供了13个api接口，用于监控运行状态的spring boot的状况。  
首先在application.yml配置文件中将所有的api全部暴露  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e52f9a87d0ce54ccfc1a29fffae1dfce.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c7dc33f666dde5d5f19bcb2f76ac271d.png)

| 类型  | API接口           | 描述                          | 示例                                                                                        |
|-----|-----------------|-----------------------------|-------------------------------------------------------------------------------------------|
| GET | /configprops    | 描述配置属性如何注入Bean              | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/873079282fdc43c5d3e163c1ecb096cd.png) |
| GET | /beans          | 描述ing用程序上下文里全部的Beanm以及他们的关系 | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f777969db212a5b17f54db7b6cc87aed.png) |
| GET | /heapdump       | 获取快照                        | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0860e30f00fe43aa5d624fa0fff0699c.png) |
| GET | /threaddump     | 获取线程活动的快照                   | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6929d78aa6a6aaa4d75d429f42a745f4.png) |
| GET | /env            | 获取全部环境属性                    | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7fc87aca67bb64e719a477932c50c24.png) |
| GET | /env/{name}     | 根据名称获取特定的环境属性               | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d988b19dedd6bcd83c480636f1e0df6a.png) |
| GET | /health         | 应用程序的健康指标                   | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6782c1174e604466bb8ae5e2b46a4a41.png) |
| GET | /info           | 获取应用程序的信息                   | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b2e08e5f216950f77c5204a3246ea549.png) |
| GET | /mappings       | 描述全部的url以及控制器               | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2bacf8f16b14aafbae4493145deedfb1.png) |
| GET | /metrics        | 获取应用程序的全部度量信息               | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c6b6db737d88cdebddd632af61cb31a.png) |
| GET | /metrics/{name} | 获取程序的指定名称的度量信息              | ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a69c9dca64b1b6e5e272c563a1a0c161.png) |
| GET | /shutdown       | 关闭应用程序                      | 需要将endpoints.shutown.enabled设置为true                                                       |