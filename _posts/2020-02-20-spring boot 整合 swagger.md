---
layout: post
title: "spring boot 整合 swagger"
date: 2020-02-20 20:20:12 +0800
categories: [spboot集成swagger, jpa+swagger2, post+get, swagger2注解, spring boot日志]
description: "本文详细介绍如何在SpringBoot项目中整合Swagger2，包括引入依赖、配置数据源、枚举映射、实体类定义、DAO、Service及Controller的实现，同时讲解了日志配置、测试流程及Swagger2注解的使用。"
keywords: spboot集成swagger, jpa+swagger2, post+get, swagger2注解, spring boot日志
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104399638
> - 发布时间：2020-02-20 20:20:12
> - 阅读量：542
> - 分类：微服务同时被 3 个专栏收录, 订阅专栏, spring boot, JPA
> - 标签：#spboot集成swagger, #jpa+swagger2, #post+get, #swagger2注解, #spring boot日志

## 摘要

文章浏览阅读542次。本文详细介绍如何在SpringBoot项目中整合Swagger2，包括引入依赖、配置数据源、枚举映射、实体类定义、DAO、Service及Controller的实现，同时讲解了日志配置、测试流程及Swagger2注解的使用。

---

#### spring boot 整合 swagger

  * 1\. swagger 简介
  * 2\. 创建
  * 3\. 配置
  * 4\. 配置数据源
  * 5\. 枚举
  * 6\. 枚举映射
  * 7\. 实体
  * 8\. dao
  * 9\. service
  * 10\. controller
  * 11\. 配置日志级别
  * 12\. dao 测试
  * 13\. service测试
  * 14\. controller测试
  * 15\. swagger2注解
  * 16\. 启动

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. swagger 简介

Swagger，中文“拽”的意思，它是一个功能强大的在线 API文档的框架，目前它的版本  
为2.x，所以称为 Swagger2。Swagger2提供了在线文档的查阅和测试功能。利用 Swagger2很容易构建RESTful 风格的API，在Spring Boot 中集成 Swagger2。

## 2\. 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/590921b052f26d489144606df8102c49.png)  
引入依赖  
implementation ‘io.springfox:springfox-swagger2:2.6.1’  
implementation ‘io.springfox:springfox-swagger-ui:2.6.1’

## 3\. 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c6bd410a6b238b43470507b45f0b67e0.png)

## 4\. 配置数据源

首先增加编码集依赖  
implementation ‘cn.easyproject:orai18n:12.1.0.2.0’  
然后配置数据源  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aa4a18c645266f51462257d8a0a796eb.png)

## 5\. 枚举

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ddb53d0bfd9b2be91a33e7160c3c64ef.png)

## 6\. 枚举映射

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1f06a63820cf275746b8beeea86346e.png)

## 7\. 实体

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/28fb494528d1d5199da0d25cfdaf8d0a.png)

## 8\. dao

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce1239c6fd44ac3c6b89ac0ac28e51d8.png)

## 9\. service

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6ac7f787a697660baf1be7221fd9cdcb.png)

## 10\. controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/49ec98c20847aa3e9426cf07dddc5bc5.png)

## 11\. 配置日志级别
    
    
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
    
    

## 12\. dao 测试

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/389217298c73db742efb188eb32a4ad6.png)

## 13\. service测试

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4dcc8cc2ef8926e3d16b2e7c72847d65.png)

## 14\. controller测试

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aacc02ada91ca3cec2c9fac4a764350e.png)

## 15\. swagger2注解

（3）写生成文档的注解  
Swagger2通过注解来生成API接口文档，文档信息包括接口名、请求方法、参数、返回信息  
等。通常情况下用于生成在线API文档，以下的注解能够满足基本需求，注解及其描述如下。

  * @Api：修饰整个类，用于描述 Controller类。
  * @ApiOperation：描述类的方法，或者说一个接口。
  * @ApiParam:单个参数描述。
  * @ApiModel：用对象来接收参数。
  * @ApiProperty：用对象接收参数时，描述对象的一个字段。
  * @ApiResponse：HTTP响应的一个描述。
  * @ApiResponses：HTTP响应的整体描述。
  * @Apilgnore：使用该注解，表示Swagger2忽略这个API。
  * @ApiError：发生错误返回的信息。
  * @ApiParamImplicit：一个请求参数。
  * @ApiParamsImplicit：多个请求参数。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/01ecab3e737bd6f054a9c4a4524b0f38.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6232229a5d436206aae76bae1957f361.png)

## 16\. 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a114e253da444bc9ba3a9d39fbef3a90.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/94756d3a4d219bf18a2a44d7d2e5a351.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/57891ddfa2ec15c4aa248c90de6975eb.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dd30e43b7daa2481952a7b88a3ec9062.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c0854349299b9967ba7958a27b63ffaf.png)
