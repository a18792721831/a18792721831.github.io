---
layout: post
title: "spring boot负载均衡Feign"
date: 2020-02-26 20:35:19 +0800
categories: [Feign入门, boot整合Feign, 如何定位Feign, Feign与Ribbon的关系, FeignClient默认配置]
description: "springboot负载均衡Feign1. 创建 Feign2. 配置gradle3. Feign配置4. 开启Feign5. 创建Feign配置6. 创建FeignDao7. 创建FeignService8. 创建FeignController9. 启动&验证10. Feign与Ribbon的关系11. FeignClient12. FeignClient配置git地址https:/..._springboot feign 负载均衡"
keywords: Feign入门, boot整合Feign, 如何定位Feign, Feign与Ribbon的关系, FeignClient默认配置
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104523325
> - 发布时间：2020-02-26 20:35:19
> - 阅读量：2
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#Feign入门, #boot整合Feign, #如何定位Feign, #Feign与Ribbon的关系, #FeignClient默认配置

## 摘要

文章浏览阅读2.3k次。springboot负载均衡Feign1. 创建 Feign2. 配置gradle3. Feign配置4. 开启Feign5. 创建Feign配置6. 创建FeignDao7. 创建FeignService8. 创建FeignController9. 启动&验证10. Feign与Ribbon的关系11. FeignClient12. FeignClient配置git地址https:/..._springboot feign 负载均衡

---

#### springboot负载均衡Feign

  * 1\. 创建 Feign
  * 2\. 配置gradle
  * 3\. Feign配置
  * 4\. 开启Feign
  * 5\. 创建Feign配置
  * 6\. 创建FeignDao
  * 7\. 创建FeignService
  * 8\. 创建FeignController
  * 9\. 启动&验证
  * 10\. Feign与Ribbon的关系
  * 11\. FeignClient
  * 12\. FeignClient配置

  
git地址   
https://github.com/a18792721831/studySpringCloud 

## 1\. 创建 Feign

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/27536c3f5a2c065a28b254e9a30403cf.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bfe1c1f4b28d8871b9aba573fe3816cf.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/adeab9465fe12224bc96d33158650e79.png)

## 2\. 配置gradle

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c1c282b47e5f7f557d22bde62216c73.png)
    
    
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
    

## 3\. Feign配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2dba143a7929ab9b58b494e4bdc44dbd.png)
    
    
    server:
      port: 8000
    
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
        name: eureka-feign-client
    

## 4\. 开启Feign

在SpringbootfeignApplication类增加注解  
EnableFeignClients–开启Feign  
EnableEurekaClient–开启Eureka  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f1b2a929ce4f9f05165e08035068e492.png)

## 5\. 创建Feign配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d03b0b33d9894fa38afcc0c0a06d84b.png)

## 6\. 创建FeignDao

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/764cfc68e2798749d557d28cb4549511.png)  
其中，  
EUREKA-CLIENT表示eureka-client服务提供者的服务名字  
FeignConfig.class指定Feign的配置  
注意：  
1.这是接口  
2.GetMapping中是Eureka-client服务提供者暴露的接口的url请求地址  
说明：  
我个人比较喜欢将Feign定位于Dao层，当然Service层也可。

## 7\. 创建FeignService

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d4afec73a135dc33e915b3c6fe46ec1.png)

## 8\. 创建FeignController

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7565d7755e7c6b64c45d638706874f76.png)

## 9\. 启动&验证

首先启动eureka-server  
接着启动eureka-client服务提供者（多实例启动）  
最后启动feign-client服务消费者  
全部启动后是5个实例  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fcb14435409d04e7a30692334333c9b4.png)  
访问eureka-server的主面板  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/15bd72ba5afb7c9c771de93e32125992.png)  
接着验证eureka-client服务提供者的接口  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c1266e3de08ca0162d237be322f1d36e.png)  
然后请求feign的接口  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f8aba652aca4eb7d59c47115d38dd45.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e9dca5f604f70fdfc0e2ced4c75eca25.png)  
发现其效果与Ribbon一致，都是轮询访问的。

## 10\. Feign与Ribbon的关系

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6a4dd642d5a92a43bd8a69d23a0fbb08.png)  
从github看，Feign维护很好，Issues处理比较及时。  
而且从readme中看，feign的计划什么的都有，且分短期、中期和长期计划  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f3d7d9a2167bc5a1a29f62a1cf46f8a.png)  
而且给的例子也很多。赞。

从Feign的jar包依赖看  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/40141de9c0fafecc4fca7f8ea0919058.png)  
其依赖了ribbon.  
对于Feign有书上是这样介绍的

> Feign 受 Retrofit、JAXRS-2.0 和 WebSocket 的影响，采用了声明式 API 接口的风格，将Java Http 客户端绑定到它的内部。Feign 的首要目标是将JavaHttp客户端调用过程变得简单。Feign 的源码地址：https://github.com/OpenFeign/feign.

所以可以看到Feign的重点应该不是实现与Ribbon相同的功能，而是在Ribbon的基础上开发新的功能的。

## 11\. FeignClient

我们在FeignDao上加上FeignClient注解的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/51d2693453e991e8def5d1ca14c2aa28.png)  
注解的属性：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d63484714d01f884876eee03cece6347.png)  
看到了与Ribbon有关的包  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/092981b448e3bef626ac03499da3bf5b.png)

  * FeignClient 注解被@Target(ElementType.TYPE)修饰，表示 FeignClient 注解的作用目标在接口上。
  * @Retention(RetentionPolicv.RUNITINF)注解表明该注解会在Class字节码文件中存在，在运行时可以通过反射获取到。
  * @Documented 表示该注解将被包含在 Javadoc中。
  * @FeignClient 注解用于创建声明式API接口，该接口是RESTful风格的。Feign被设计成插拔式的，可以注入其他组件和Feign一起使用。最典型的是如果Ribbon可用，Feign会和Ribbon 相结合进行负载均衡。
  * value()和 name()一样，是被调用的服务的ServiceId。
  * url()直接填写硬编码的Url 地址。
  * decode404即404是被解码，还是抛异常。
  * configuration()指明FeignClient的配置类，默认的配置类为FeignClientsConfiguration类，在缺省的情况下，这个类注入了默认的 Decoder、Encoder 和 Contract 等配置的 Bean。
  * fallback(）为配置熔断器的处理类。

## 12\. FeignClient配置

Feign Client 默认的配置类为 FeignClientsConfiguration，这个类在 spring-cloud-netflix-core的jar包下。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d1bcd0b2f8cb82ba0d9dea8e9a683abc.png)  
打开这个类,可以发现这个类注入了很多Feign相关的配置Bean，包括FeignRetryer、FeignLoggerFactory 和FormattingConversionService 等。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/97392bc810bcff06d15b055afd123707.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de6083134b00e87f47cbf86ed871ee1a.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/905399d392ea94b7cdd98e7cdf1c6815.png)  
另外，Decoder、Encoder 和 Contract这3个类在没有Bean被注入的情况下，会自动注入默认配置的Bean，即ResponseEntityDecoder、SpringEncoder 和 SpringMvcContract。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/72e53883086166b9e3dba8811e9e1c49.png)

默认注入的配置如下。

  * Decoder feignDecoder:ResponseEntityDecoder。
  * Encoder feignEncoder：SpringEncoder
  * Logger feignLogger:SIf4jLogger。
  * Contract feignContract:SpringMvcContract。
  * Feign.Builder feignBuilder:HystrixFeign.Builder。
  * FcignClientsConfiguration 的配置类部分代码如下，
  * @ConditionalOnMissingBean 注解表示如果没有注入该类的Bean就会默认注入一个Bean。
