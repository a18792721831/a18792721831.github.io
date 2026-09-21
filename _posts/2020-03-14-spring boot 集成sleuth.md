---
layout: post
title: "spring boot 集成sleuth"
date: 2020-03-14 16:14:01 +0800
categories: [zipkin 原理, zipkin docker, zipkin rabbitmq, zipkin elsearch, zipkin kibana]
description: "本文详细介绍了SpringBoot中Sleuth组件的使用，以及如何与Zipkin进行集成，实现微服务间的链路追踪。覆盖了理论概念、实践步骤、Zipkin的组成部分及其与RabbitMQ、Elasticsearch的集成。"
keywords: zipkin 原理, zipkin docker, zipkin rabbitmq, zipkin elsearch, zipkin kibana
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104825746
> - 发布时间：2020-03-14 16:14:01
> - 阅读量：3
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#zipkin 原理, #zipkin docker, #zipkin rabbitmq, #zipkin elsearch, #zipkin kibana

## 摘要

文章浏览阅读3.1k次。本文详细介绍了SpringBoot中Sleuth组件的使用，以及如何与Zipkin进行集成，实现微服务间的链路追踪。覆盖了理论概念、实践步骤、Zipkin的组成部分及其与RabbitMQ、Elasticsearch的集成。

---

#### spring boot 集成sleuth

  * [1\. 理论](<#1__4>)
  *     * [1.1 sleuth是什么](<#11_sleuth_5>)
    * [1.2 sleuth有哪些](<#12_sleuth_8>)
    * [1.3 链路追踪的一些基本概念](<#13__10>)
    * [1.4 zipkin的组成](<#14_zipkin_18>)
  * [2\. zipkin 实例](<#2_zipkin__25>)
  *     * [2.1 zipkin server](<#21_zipkin_server_26>)
    * [2.2 zipkin client](<#22_zipkin_client_45>)
    *       * [2.2.1 创建](<#221__47>)
      * [2.2.2 配置](<#222__49>)
      * [2.2.3 注解](<#223__57>)
      * [2.2.4 对外接口 controller](<#224__controller_59>)
      * [2.2.5 启动](<#225__61>)
    * [2.3 gateway service](<#23_gateway_service_83>)
    *       * [2.3.1 创建](<#231__85>)
      * [2.3.2 配置](<#232__87>)
      * [2.3.3 注解](<#233__90>)
      * [2.3.4 启动](<#234__92>)
    * [2.4 自定义链路数据](<#24__110>)
  * [3\. zipkin 集成 rabbitmq](<#3_zipkin__rabbitmq_138>)
  *     * [3.1 rabbitmq的搭建](<#31_rabbitmq_140>)
    * [3.2 zipkin server rabbitmq](<#32_zipkin_server_rabbitmq_146>)
    * [3.3 zipkin client rabbitmq](<#33_zipkin_client_rabbitmq_177>)
    *       * [3.3.1 创建](<#331__178>)
      * [3.3.2 配置](<#332__180>)
      * [3.3.3 注解](<#333__183>)
      * [3.3.4 启动](<#334__185>)
  * [4\. zipkin 集成oracle](<#4_zipkin_oracle_192>)
  * [5\. zipkin集成 elasticsearch](<#5_zipkin_elasticsearch_197>)
  *     * [5.1 安装 elasticsearch](<#51__elasticsearch_198>)
    * [5.2 安装 kibana](<#52__kibana_213>)
    * [5.3 zipkin 使用 elasticsearch](<#53_zipkin__elasticsearch_225>)
    * [5.5 zipkin client 使用 elasticsearch](<#55_zipkin_client__elasticsearch_234>)
    *       * [5.5.1 创建](<#551__235>)
      * [5.5.2 配置](<#552__247>)
      * [5.5.3 注解](<#553__249>)
      * [5.5.4 启动](<#554__251>)
      * [5.5.5 验证](<#555__253>)
    * [5.6 kibana 连接 elasticsearch](<#56_kibana__elasticsearch_256>)

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. 理论

### 1.1 sleuth是什么

sleuth是spring cloud的一个组件，主要的功能是在分布式系统中提供服务链路追踪的解决方案。  
简单来讲，就是，展示微服务调用过程。

### 1.2 sleuth有哪些

常见的链路追踪组件有Google的Dapper，Twitter的Zipkin，以及阿里的Eagleeye

### 1.3 链路追踪的一些基本概念

  *     1. Span 基本工作单元，每次一个远程调度任务就会产生一个Span，Span是用一个64位ID唯一标识的。Span包含了摘要、时间戳标记、Span的ID以及进程ID.
  *     2. Trace: 有一系列的Span组成，呈现树状结构。在一个微服务请求中，每次一个新的调用都会产生一个Span，这些所有的Span组成了Trace.
  *     3. Annotation:用于记录一个事件，一些核心的注解用于定义一个请求的开始和结束：
    * cs-Client Sent:客户端发送一个请求，这个注解描述了Span的开始。
    * sr-Server Received:服务端获得请求并准备开始处理。sr-cs的时间差就是网络请求的时间。
    * ss-Server Sent:服务端发送响应，该注解表明请求处理的完成(请求的结果开始返回客户端)ss-sr的时间差就是服务器处理请求的真正时间
    * cr-Client Received:客户端接收响应，此时整个Span结束，cr-cs的时间差就是整个请求的时间。

### 1.4 zipkin的组成

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/111a8d21fcb5eb9438a12b1baaa44714.png)  
zipkin由4部分组成：

  * Collector：收集器组件，它主要用于处理从外部系统发送过来的跟踪信息，将这些信息转换为 Zipkin 内部处理的 Span 格式，以支持后续的存储、分析、展示等功能。
  * Storage：存储组件，它主要对处理收集器接收到的跟踪信息，默认会将这些信息存储在内存中，我们也可以修改此存储策略，通过使用其他存储组件将跟踪信息存储到数据库中。
  * RESTful API：API 组件，它主要用来提供外部访问接口。比如给客户端展示跟踪信息，或是外接系统访问以实现监控等。
  * Web UI：UI 组件，基于 API 组件实现的上层应用。通过 UI 组件用户可以方便而有直观地查询和分析跟踪信息。

## 2\. zipkin 实例

### 2.1 zipkin server

这里有一个坑，现在好多书上或者资料上说怎么怎么搭建zipkin server工程。  
但是你根据数据或者网上的步骤一步一步的完成，但是会出现各种各样的问题。  
这是因为从19年左右开始，zipkin server已经被集成到jar包里面了，不需要再搭建了，只需要java -jar启动就可以了。  
首先，zipkin server在搭建的时候都需要依赖zipkin-ui的，也就是说，我们搭建zipkin server只是为了展示数据。  
那么，数据的展示是稳定的，不需要频繁的更改，所以zipkin就将zipkin server（依赖的了ui）打成jar包，下载后启动即可使用。  
注意，jar包有jdk版本依赖。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e3ca20a56468009da67f68b462aef04a.png)  
[ latest传送门](<https://search.maven.org/remote_content?g=io.zipkin&a=zipkin-server&v=LATEST&c=exec>)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b3245796efdc5657090498442122e56b.png)  
下载完成后启动，会出现如下界面  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/83c9eb48edc9d7d322c758592a7d67de.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4315e4ab055e5c641cdaa0e1202503ec.png)  
访问9411端口，可以进入zipkin的界面  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d46f2adc5627428c4f5192d32254ded2.png)  
这个界面一般不会修改的，所以为了方便起见，将zipkin在服务器的docker上启动。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a57f973a2d52109afc2ea91ba82af31.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f8bc9ea24e3df2b11d1309e3fbdb731e.png)  
访问  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1fea43b0e1c85d245a88ee2c3e4d1dfb.png)

### 2.2 zipkin client

zipkin server是负责将数据展示出去，而zipkin client则是收集数据，并将收集到的数据传给zipkin server，然后完成数据的收集与展示。

#### 2.2.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f825f26b4fcbe17d876c40f50686458c.png)

#### 2.2.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/297a44eeefb5960b6ab5ce52940929f3.png)  
最上面配置eureka server的地址  
中间是日志级别  
然后指定服务名是zipkin-client-user-service  
然后指定zipkin server的地址  
最后的spring.sleuth.sampler.percentage是上传到zipkin server的信息的百分比  
1.0 表示将所有的链路信息全部上传。

#### 2.2.3 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b5d3b80ce050d4da2c3f1e6175f7d9ff.png)

#### 2.2.4 对外接口 controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c85e2e0eb3cef5d9169011f53b1d7c22.png)

#### 2.2.5 启动

首先启动eureka server，然后启动zipkin client.  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18f5d979f4cc5238a25309dd8ee3e1fb.png)  
然后尝试请求zipkin client对外开放的接口  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c4ae09a0909328e0d1c0c35d9c2bfaf.png)  
然后刷新zipkin server  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/95cd350013f7400cf3d34189d837aff0.png)  
我有一次地址写错了，所以这里是两条记录。  
点击其中一个还可以进去  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/05e64b8bdceffeffa684bc9be4ec87fa.png)  
这个图怎么看？  
1.系统的展示了微服务请求的时间以及微服务的深度和这次微服务调用的唯一标识traceID  
2.表示这次微服务调用的入口以及请求方式restful  
3.选择的请求的span id以及这个span的父亲。  
4.这个事件  
5.记录了微服务请求的结果以及来源信息。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/49296d5063ba3cabb178790ed4927145.png)  
对于事件还可以展示更多信息，记得我们在理论里面说事件有4个注解：cs、sr、ss、cr  
上面只展示了sr、ss  
我们再来看一个成功的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ae42b2ccc93e607797fabcab4a047fe.png)

### 2.3 gateway service

上面我们看了1层深度的微服务的调用，我们之前了解过zuul，基于zuul实现微服务的转发，进而实现多层微服务的调用。

#### 2.3.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/812860c62bed1aa9bf00265629fb24f5.png)

#### 2.3.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8dc65796f4f09d7bcfec1df60f6855a1.png)  
指定端口、eureka server的地址，日志级别，以及服务名称zipkin server的地址，上报比例以及zuul转发路由。

#### 2.3.3 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8786879bcfb16c9d3a64b7b2e89c084.png)

#### 2.3.4 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf98a9e54b28171eb7964a451d54f44b.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7a6c6d0286a91e16877f2b48b37d0ec.png)  
此时访问zipkin server  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce49b3bb708db2423acc8295e7f6b2fc.png)  
这个时候，搜索结果就有不一样了  
1.表示微服务调用入口微服务  
2.表示那些微服务参与，其顺序  
点击进去查看详细信息  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dffcbc0788f19c513714b2aaa19429dd.png)  
这是gateway-service的调用信息，可以看到这是调用入口微服务，因为其span的父亲是空的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0778ff30dca6c4b94a2ee826f7409fd.png)  
这是user-service的调用详细信息，也是gateway的下一级调用的服务。因为user-service的span的父亲是gateway的span.  
而且这个事件的节点信息就与我们理论中的所有都有了。  
当然，下面遮住的是user-service的返回信息。  
当然，这是微服务请求调用其他微服务比较少的时间，如果调用的微服务比较多的时候，查看起来就非常的不方便了。幸好，zipkin server还可以以更加简单明了的方式查看微服务调用链路  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2b4821f70126ee5355ca1cdb3c0e561d.png)  
而且这图还可以动，连接线上的小圆点会指明调用的方向的。

### 2.4 自定义链路数据

在了解zuul的时候，我们知道了zuul有4种filter![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/714e65ba77f79810f0d3cab648e54f96.png)  
我们在2.3种的gateway-service中实现post过滤器  
在过滤器中我们加入请求者信息(打印一些信息)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4a36fc3fb4e212adaab99ee5e894be02.png)  
注意导入的tracer包是zipkin的包里的类。  
我们通过之前看traceID和SpanID以及tag基本上就能知道其树状关系。  
接下来重启访问  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b3d16114bd7493efff6f2b7b931f99f1.png)  
查看zipkin server  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/91b35dd6e946afffe86e0cf0747c843c.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6466026aabe612d29769c99bbcecf91d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/af7204ce8c6c1ed973aa6fd9528885f3.png)  
很奇怪，为什么没有我们的tag，难道没有走posty过滤器》？  
我们在过滤器里面打印控制台输出  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/42e640f68e66df7040f3b7479bb72e37.png)  
然后重试  
但是还是没有打印我们想要的输出  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4e8a6df9a0cbe8c8c4a22e0d588005f.png)  
仔细查看，重新梳理，发现是因为没有将过滤器注册到spring容器中  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6be71c44986ab705c44418954b7d1d0f.png)  
重试  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2afe6c62d6c043006f08f6bc65a097f8.png)  
打印了。  
接下来看看zipkin server  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48f89b2f16d79e5eac2ce8d9b6a4a926.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ad2de952eeb2dea91eda13f3e18e3c4.png)  
与我们想象中的一样，至于访问不到主机名，然后异常的情况就不试了。

## 3\. zipkin 集成 rabbitmq

在我们之前的实例中，zipkin client收集到信息是通过http将数据传输到zipkin server的，使用http请求传输数据比较性能差，所以zipkin 可以集成rabbitmq进行数据的传输。

### 3.1 rabbitmq的搭建

rabbitmq的搭建参考这里：  
https://blog.csdn.net/a18792721831/article/details/104737243  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/77f7623c2892f3a0837634ae467401af.png)  
所以我们是有rabbitmq的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6a3266ff5aaa784d4220a72ecabb1e52.png)

### 3.2 zipkin server rabbitmq

我们可以通过环境变量让 Zipkin 从 RabbitMQ 中读取信息，就像这样：
    
    
    RABBIT_ADDRESSES=localhost java -jar zipkin.jar
    

可配置的环境变量如下表所示：

| 属性                                           | 环境变量                      | 描述                                                            |
|----------------------------------------------|---------------------------|---------------------------------------------------------------|
| zipkin.collector.rabbitmq.concurrency        | RABBIT_CONCURRENCY        | 并发消费者数量，默认为 1                                                 |
| zipkin.collector.rabbitmq.connection-timeout | RABBIT_CONNECTION_TIMEOUT | 建立连接时的超时时间，默认为 60000 毫秒，即 1 分钟                                |
| zipkin.collector.rabbitmq.queue              | RABBIT_QUEUE              | 从中获取 span 信息的队列，默认为 zipkin                                    |
| zipkin.collector.rabbitmq.uri                | RABBIT_URI                | 符合 RabbitMQ URI 规范 的 URI，例如 amqp://user:pass@host:10000/vhost |

如果设置了 URI，则以下属性将被忽略。

| 属性                                     | 环境变量                | 描述                                                    |
|----------------------------------------|---------------------|-------------------------------------------------------|
| zipkin.collector.rabbitmq.addresses    | RABBIT_ADDRESSES    | 用逗号分隔的 RabbitMQ 地址列表，例如 localhost:5672,localhost:5673 |
| zipkin.collector.rabbitmq.password     | RABBIT_PASSWORD     | 连接到 RabbitMQ 时使用的密码，默认为 guest                         |
| zipkin.collector.rabbitmq.username     | RABBIT_USER         | 连接到 RabbitMQ 时使用的用户名，默认为 guest                        |
| zipkin.collector.rabbitmq.virtual-host | RABBIT_VIRTUAL_HOST | 使用的 RabbitMQ virtual host，默认为 /                       |
| zipkin.collector.rabbitmq.use-ssl      | RABBIT_USE_SSL      | 设置为 true 则用 SSL 的方式与 RabbitMQ 建立链接                    |

但是我们是使用docker启动的，所以需要使用-e实现指定环境变量  
我们重新启动zipkinserver  
我们之前启动的是简化版的，只能用于内存和elasticsearch存储。  
如果要使用kafka或者rabbitmq,则需要启动完整版的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/85d37c352d7f026876f552307ebeec4a.png)
    
    
    docker run -d -p 9411:9411 -e "RABBIT_ADDRESSES=10.0.228.93:30672"  --name=zipkin-server openzipkin/zipkin
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e518b7c484a2ef632f0aa14c4e88a117.png)

### 3.3 zipkin client rabbitmq

#### 3.3.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b2ca3c6d896088c976c991bae75a593f.png)

#### 3.3.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f81a0332fc53a53c6681f47000d58ecc.png)  
我们注释掉了zipkin的base-url的配置

#### 3.3.3 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/718b1a784fc392b683ea2f1145978cd1.png)

#### 3.3.4 启动

首先启动eureka server,然后启动zipkin-rabbitmq  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d678c655a6e67891835cd1dd620ff2a8.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a0d66ca80c710f514cf11ecc3f3c345.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9461c1215792be2cbbba7a6a391a7b6c.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e9047079d98f23dc5eee7e913afc8e0f.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0c60cb5b52b71b972ecbefe39f78442c.png)

## 4\. zipkin 集成oracle

目前zipkin官方没有实现oracle的集成，只支持mysql，由于我本机没有安装mysql，所以这部分不做深入。  
详细介绍见 https://github.com/openzipkin/brave/tree/master/instrumentation  
不过，基于zipkin的实现原理，可以用其他方式实现对普遍数据库的支持，需要自己实现数据的存储于插入到span中。  
详细实现见 [Zipkin原理学习–日志追踪 MySQL 执行语句](<https://tianjunwei.blog.csdn.net/article/details/88559499>)

## 5\. zipkin集成 elasticsearch

### 5.1 安装 elasticsearch

我们需要在服务器上的docker环境下安装elasticsearch。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a04ff41fb05c0c87d07b281b3467a3b2.png)  
我们根据docker hub 上给出的教程，安装elasticsearch  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d394f44fe971b1027e01db7b64605ba1.png)  
使用的镜像如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/466572e15d96058528dc20b92cea4f10.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/782e2fab0c01d7f7d535cc67131a735c.png)  
访问验证  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/19ecdcb1965f653d94e5607472c6bb78.png)  
这里借用一张图片  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/80b86fc9f3ec641638989cec548bcf1d.png)  
很明白。  
我们安装的是单机版，如果需要集群安装，参考这里  
https://www.elastic.co/guide/en/elasticsearch/reference/7.5/docker.html

### 5.2 安装 kibana

elasticsearch只是存储了数据，那么数据的展示是通过kibana展示的。  
我们在docker hub上搜索elasticsearch时，kibana会一起展示，这两个基本上都是一块使用的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/97d7fd8e30a8a4aa247a92c05747856a.png)  
安装过程也是一样的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c573dba685a3e4a67a3dea9cd1a1d8cc.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe34b28548db7a4504b92f3ba722c7f9.png)  
访问验证  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d5c9c97f3d40e21e36d56374dbc62562.png)  
kibana启动比较慢  
要做好多的操作才能成功启动  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/01d99dcb2d75a1cddd0a0949fb9c2d34.png)

### 5.3 zipkin 使用 elasticsearch

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e22448173399b97641bf5e7f9dac729.png)  
基于此，我们需要重新创建一个docker 容器，之前创建的容器是基于rabbitmq的，现在重新创建一个基于elasticsearch的容器
    
    
    docker run -d -p 9412:9411 -e "STORAGE_TYPE=elasticsearch" -e "ES_HOSTS=10.0.251.180:9200"  --name=zipkin-elsearch openzipkin/zipkin
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a2a92ff78cbe4437cae4cc0ed1cc93cf.png)  
访问验证  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eda823b9c2dc8470d4a4fb090fec7bd9.png)

### 5.5 zipkin client 使用 elasticsearch

#### 5.5.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f825f26b4fcbe17d876c40f50686458c.png)  
和普通的一样，直接创建。  
我是这样理解，也不知道是否正确，大家当做参考就行。

  * zipkin server集成了elasticsearch,当zipkin client 通过http或者mq将链路信息上传至zipkin server后，zipkin server会将信息存储到elasticsearch而不是内存中，此时，elasticsearch就拥有了zipkin client的链路信息。
  * zipkin client通过http或者mq将链路信息上传到zipkin server.
  * elasticsearch接收到zipkin server的信息后，存储到elasticsearch然后提供给其他组件进行使用
  * kibana连接elasticsearch，并且根据定义的查询进行信息的展示。  
这里有一个误区：  
网上或者书上的许多资料，都是基于本地的zipkin server进行集成elasticsearch的。很容易给人一种错觉：zipkin client直接将链路信息上传给elasticsearch的。实际上，zipkin client也可以直接将链路信息上传给elasticsearch，只不过需要进行手动的上传。  
而将信息上传给zipkin server，且指定zipkin server的存储方式后，这些就都不需要了，zipkin server会将这些都处理完成的。  
(这一块花费了我大概4小时才意识到这个思想误区)

#### 5.5.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d83f0235a2e6b683c5a447c2cfa1e1f.png)

#### 5.5.3 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a3e4ae3b7936eb040126bab05ad65106.png)

#### 5.5.4 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fcf4dd34cbf2f5aefd2da37d562d7f09.png)

#### 5.5.5 验证

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a05c0b037449d5f550d963f79f26b74.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2556a24836f976f7c09bba2416a87d0e.png)

### 5.6 kibana 连接 elasticsearch

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/af37b0b6ba4af01e10c55406904ee3f1.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad5f40a5b429be115ba3e82db3fa9507.png)  
输入zipkin-*会显示匹配到一个index  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9e78a2155491a6947ff34e1da42f82d0.png)  
选择第一个就行，根据名字来看是和时间有关的，具体没有研究。。。。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4af1e33f0fd48614f39129f22854fcae.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a54ac1f6a85230351966cff9dfa29b39.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8682d364e3129918353d6e1f18435916.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25af7adad7882f78de7e869260d100fb.png)

这里有一个比较坑的地方  
点击展示面板可能不会展示上图，而是提示没有信息可以展示。(巨坑无比:因为展示的是默认的index，而不是我们需要的zipkin-*的index)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5cea7de888dbf49d4c47b8b21fb868ac.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/398d116fb58c43bd463782d636f122af.png)  
我们在请求一次，看看效果：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d7e03bc88cf6a413a1c18d45d80ee1c9.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3386c8d899d0098393e6a457377cd89d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c7dd9c46c1cf3b63da1af49f4ee506a7.png)  
当然kibana还提供更强大的展示面板  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a9d1f421eb06710027d5fa5d4777dd8f.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb07bdd0b3b916c95f41ba67bd7cdd18.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/26fc0f11dcc5ea7dbdd30b57dacd3f3e.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9de12ba195feccf1980f865c3881e770.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f58fd1db171a5e3c7db008c66e7c92b.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3a73f09adea41889d6a722b3967e7ce8.png)  
还有统计个数的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3ac7f22cd42f3ba42c4aed38548d06e2.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/afa96e5d14b5f671a7f10df790028cb7.png)  
随便定制。强大，还不太会用。。