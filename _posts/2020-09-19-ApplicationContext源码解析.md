---
layout: post
title: "ApplicationContext源码解析"
date: 2020-09-19 20:45:46 +0800
categories: [应用上下文源码解析, 解析常用的容器上下文, 上下文源码解析, 上下文实现原理, 注解配置原理解析]
description: "ApplicationContext源码解析接口继承关系--向上ApplicationContext资源ResourceInputStreamSource环境变量EnvironmentPropertyResolver事件消息国际化接口继承关系--向下WebApplicationContextServletConextRequestDispatcherServletRegistrationServletRegistrationFilterFilterChainFilterRegistrationEventLi_application-context 环境变量"
keywords: 应用上下文源码解析, 解析常用的容器上下文, 上下文源码解析, 上下文实现原理, 注解配置原理解析
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/108685530
> - 发布时间：2020-09-19 20:45:46
> - 阅读量：500
> - 分类：spring同时被 2 个专栏收录, 订阅专栏, springMVC
> - 标签：#应用上下文源码解析, #解析常用的容器上下文, #上下文源码解析, #上下文实现原理, #注解配置原理解析

## 摘要

文章浏览阅读500次。ApplicationContext源码解析接口继承关系--向上ApplicationContext资源ResourceInputStreamSource环境变量EnvironmentPropertyResolver事件消息国际化接口继承关系--向下WebApplicationContextServletConextRequestDispatcherServletRegistrationServletRegistrationFilterFilterChainFilterRegistrationEventLi_application-context 环境变量

---

#### ApplicationContext源码解析

  * 接口继承关系--向上
  *     * ApplicationContext
    * 资源
    *       * Resource
      * InputStreamSource
    * 环境变量
    *       * Environment
      * PropertyResolver
    * 事件
    * 消息国际化
  * 接口继承关系--向下
  *     * WebApplicationContext
    *       * ServletConext
      *         * RequestDispatcher
        * Servlet
        * Registration
        * ServletRegistration
        * Filter
        *           * FilterChain
        * FilterRegistration
        * EventListener
      * ServletRequest
      *         * AsyncContext
        * AsyncListener
        * DispatcherType
      * ServletResponse
    * WebServerApplicationContext
    *       * WebServer
    * ConfigurableApplicationContext
    *       * Lifecycle
      * Closeable
      * ConfigurableEnvironment
      * ProtocolResolver
    * ReactiveWebApplicationContext
    * ConfigurableWebApplicationContext
    *       * ServletConfig
    * ConfigurableWebServerApplicationContext
    * ApplicationContextAssertProvider
    *       * AssertProvider
    * ConfigurableReactiveWebApplicationContext
    * AssertableWebApplicationContext
    * AssertableApplicationContext
    * AssertableReactiveWebApplicationContext
  * 实现
  *     * AbstractApplicationContext
    *       * ApplicationEventMulticaster
      * DefaultResourceLoader
      * ResourceLoader
    * GenericApplicationContext
    * AbstractRefreshableApplicationContext
    * AbstractRefreshableConfigApplicationContext
    * StaticApplicationContext
    *       * StaticWebApplicationContext
    * GenericGroovyApplicationContext
    * GenericXmlApplicationContext
    * AnnotationConfigApplicationContext
    *       * AnnotationConfigRegiustry
      * AnnotatedBeanDefinitionReader
      * ClassPathBeanDefinitionScanner
      * AnnotationConfigReactiveWebApplicationContext
    * GenericWebAppliicationContext
    *       * AnnotationConfigServletWebApplicationContext
      * ServletWebServerApplicationContext
      *         * WebServerGracefulShutdownLifecycle
        *           * SmartLifecycle
          * Lifecycle
          * Phased
        * WebServerStartStopLifecycle
        * AnnotationConfigServletWebServerApplicationContext
        * XmlServletWebServerApplicationContext
    * GenericReactiveWebApplicationContext
    *       * ReactiveWebServerApplicationContext
      *         * WebServerManager
        * HttpHandler
        * ServletHttpRequest
        * HttpRequest
        * HttpMessage
        * HttpMethod
        * AnnotationConfigReactiveWebServerApplicationContext
    * AbstractRefreshableWebApplicationContext
    *       * GroovyWebApplicationContext
      * XmlWebApplicationContext
      * AnnotationConfigWebApplicationContext
    * AbstractXmlApplicationContext
    *       * ClassPathXmlApplicationContext
      * FileSystemXmlApplicationContext
  * 常用的注解

## 接口继承关系–向上

在这篇文章中，我们主要了解了`BeanFactory`的结构和功能，以及其关系，最终我们以`XmlBeanFactory`为例，了解了`BeanFactory`的实现。

不过，目前基于`BeanFactory`的可以使用的容器仅仅有`XmlBeanFactory`而且是被标注为废弃的。

我们日常工作中，使用的更多的是`ApplicationContext`。

这是`ApplicationContext`接口的关系：

![image-20200914193115968](https://i-blog.csdnimg.cn/blog_migrate/f610d4caf36786f54359da5fe4287578.png)

可以看到，`ApplicationContext`继承了`BeanFactory`的同时，还扩展了资源加载、环境变量、事件和消息国际化这四个比较大的功能。

### ApplicationContext

接口要求实现的操作：

  1. 获取id
  2. 获取名字
  3. 获取启动时间
  4. 获取父上下文
  5. 获取自动装配的容器

### 资源

`ResourcePatternResolver`继承了`ResourceLoader`接口，主要定义了这些操作：

`ResourceLoader`:

  1. classpath的url字符串
  2. 获取Resource
  3. 获取ClassLoader

`ResourcePatternResolver`:

  1. classpath*的url字符串
  2. 获取Resource[]

#### Resource

`Resource`继承了`InputStreamSource`.

`Resource`定义了这些操作：

  1. 资源是否存在
  2. 资源是否可读
  3. 资源是否打开
  4. 资源是否是文件
  5. 获取URL
  6. 获取URI
  7. 获取File
  8. 获取字节读信道
  9. 长度
  10. 最后修改时间
  11. 获取文件名字
  12. 获取资源描述

#### InputStreamSource

`InputStreamSource`就很简单了，直接返回`InputStream`即可。

![image-20200915191430822](https://i-blog.csdnimg.cn/blog_migrate/f285d1e22ce90fdd6f0f958eb6a76451.png)

### 环境变量

环境变量就很简单了，直接返回一个环境变量的实体即可：

![image-20200915190108662](https://i-blog.csdnimg.cn/blog_migrate/f69297174de02e1e6ea969ea26ad0876.png)

#### Environment

`Environment`继承了`PropertyResolver`。

`Environment`定义了这些功能：

  1. 获取当前环境中活动的配置
  2. 获取默认的配置
  3. 判断配置是否可用(可变参数)
  4. 判断配置是否可用

#### PropertyResolver

`PropertyResolver`定义了这些功能：

  1. 是否包含指定的配置的key
  2. 根据key获取配置
  3. 根据key获取配置，如果不存在，返回默认值(传入)
  4. 获取指定配置，返回指定类型
  5. 获取指定配置，返回指定类型，如果不存在，返回默认值
  6. 获取必须的配置，如果不存在，抛出异常
  7. 加载`${}`类型的配置

### 事件

`ApplicationContext`接口继承了`ApplicationEventPublisher`接口，实现了事件发布。

`ApplicationEventPublisher`使用了`@FunctionalInterface`注解，表示发布的事件的内容可以是函数。

![image-20200915191750303](https://i-blog.csdnimg.cn/blog_migrate/41a78f3168d727e22d784fbd7e95ab87.png)

时间发布接口共有两个方法：

  1. 默认方法，调用定义方法发布事件
  2. 定义方法，由子类实现，完成消息的发布。

### 消息国际化

`ApplicationContext`继承了`MessageSource`接口。

![image-20200915191946913](https://i-blog.csdnimg.cn/blog_migrate/e425c3424ad3df59a7c3b3851567944f.png)

`MessageSource`主要定义了`getMessage`方法。传入不同的参数，返回String格式的国际化信息。

## 接口继承关系–向下

`ApplicationContext`作为核心接口，不仅仅继承了一些接口，有更多的接口继承了`ApplicationContext`

![image-20200915194353588](https://i-blog.csdnimg.cn/blog_migrate/b72be99fbb91de54418691fd52873a1f.png)

### WebApplicationContext

`WebApplicationContext`接口继承了`ApplicationContext`。在`ApplicationContext`的基础上，定义了一些web相关的名字：

![image-20200916190740177](https://i-blog.csdnimg.cn/blog_migrate/332699234edb9f6d13d4de4d512c80f0.png)

这些名字包括：

  1. request
  2. session
  3. application
  4. servletContext
  5. contextParameters
  6. contextAttributes

以及一个获取`ServletContext`的方法。

#### ServletConext

`ServletContext`中定义了很多获取版本号的方法，也有获取path的，获取资源的获取`RequestDispatcher`的，获取`Servlet`的，以及一些获取初始化参数等等。

![image-20200916191629732](https://i-blog.csdnimg.cn/blog_migrate/df37aa07d1c32427c3a6ee6e325d2d65.png)

还有增加`Servlet`的：

![image-20200916191900780](https://i-blog.csdnimg.cn/blog_migrate/8713d39e73734b4983f0a5f9324dae2a.png)

增加`jsp`文件的：

![image-20200916191926803](https://i-blog.csdnimg.cn/blog_migrate/e4bc53393aa3ac7f94daab92f7309597.png)

创建`Servlet`的：

![image-20200916192103155](https://i-blog.csdnimg.cn/blog_migrate/498f31675b7c9f95aa6f4c8792565f11.png)

还有`ServletRegistration`相关的：

![image-20200916192138541](https://i-blog.csdnimg.cn/blog_migrate/b4e595f5fe6ca18d1019ddc7a70c06d7.png)

以及`Filter,FilterRegistration`相关的

![image-20200916192215306](https://i-blog.csdnimg.cn/blog_migrate/a44e4d023dc10f33e6153f5333d235a2.png)

有`EventListener`相关的：

![image-20200916192318961](https://i-blog.csdnimg.cn/blog_migrate/033a1b618433ad5f492e94e39edfbd00.png)

在`ServletContext`中引出了很多的类型：

##### RequestDispatcher

根据名字分析，这个类型应该与请求分发有关。

`RequestDispatcher`主要做个三个工作：

  1. 定义了一大堆的字符串
  2. 定义了forward操作
  3. 定义了include操作

![image-20200916192929415](https://i-blog.csdnimg.cn/blog_migrate/33955f0703aea3997cb133a68cdc0e0a.png)

总结来看，`RequestDispatcher`主要定义了转发，包含操作，以及一些包路径，这些包路径应该和转发或者包含的目标路径有关。

##### Servlet

`Servlet`定义了初始化方法，定义了service操作，其中的参数是`ServletRequest`和`ServletResponse`。当然还有销毁操作。

![image-20200916193301718](https://i-blog.csdnimg.cn/blog_migrate/cf3181641837182534769d3b666da6bb.png)

`Servlet`接口定义了初始化，service和销毁方法。Servlet是一类特殊的springBean，只有一个service方法。

或者可以说，Servlet就是控制器。

##### Registration

定义了注册器的操作：

获取一些信息

![image-20200916194744821](https://i-blog.csdnimg.cn/blog_migrate/afb8370043a8428bf12c9f313511cb29.png)

实现一个内部接口：

![image-20200916194802009](https://i-blog.csdnimg.cn/blog_migrate/417183a4e422176c17310b30a2537b4f.png)

也就是说，每一个注册器都是两层，内部持有一个多一个方法的注册器

##### ServletRegistration

控制器注册。

![image-20200916193740984](https://i-blog.csdnimg.cn/blog_migrate/ec7ab455080a5933e6643c766776ffcb.png)

核心的方法是存储了url模式和控制器的映射关系。

简单来说，就是请求地址和控制器的映射关系。

##### Filter

过滤器。这些在springMVC中都有。  
有默认的初始化方法，以及过滤逻辑。

![image-20200916193959027](https://i-blog.csdnimg.cn/blog_migrate/39e45f45d870082345034ade6e1af602.png)

###### FilterChain

过滤器链，上一个过滤操作完成后，启动下一个过滤器。

内部只有三个方法：初始化、过滤、销毁。

![image-20200916194125606](https://i-blog.csdnimg.cn/blog_migrate/a49e39328b455b1093b2f885cd6591ab.png)

所以，FilterChain也是一类比较特殊的springBean。

Filter和FilterChain是同一个东西，名字不同。

##### FilterRegistration

`FilterRegistration`持有了地址和过滤器的关系

![image-20200916194648831](https://i-blog.csdnimg.cn/blog_migrate/4fdc3b72a29f50bf26eb5761b5fab0ce.png)

##### EventListener

事件监听器。是java中的一个接口，标记性接口。不要求实现方法。

![image-20200916194953416](https://i-blog.csdnimg.cn/blog_migrate/f0c644b2fe8fb052f6bf9dad6b7c8b40.png)

#### ServletRequest

控制器请求。

定义了控制器的请求应该有什么信息。

  1. 属性
  2. 编码
  3. 长度
  4. 内容类型
  5. 控制器输入流
  6. 参数
  7. 协议
  8. 获取方式(http,https,ftp)
  9. 服务信息(名字，端口)
  10. 读缓存
  11. 获取控制器上下文
  12. 获取异步控制器上下文
  13. 获取分发类型

##### AsyncContext

异步上下文控制器。

首先还是定义了一些异步相关的包路径：

![image-20200916200524361](https://i-blog.csdnimg.cn/blog_migrate/640ac4cdd9cd6836d7c12490a7431f75.png)

然后定义域了获取`ServletRequest`和`ServletResponse`方法

![image-20200916200600724](https://i-blog.csdnimg.cn/blog_migrate/71e556ecb5f71efa5b215a788ddf1155.png)

还有dispatch方法

![image-20200916200741906](https://i-blog.csdnimg.cn/blog_migrate/b7976f669a120d8f7514c6b9134f4326.png)

以及执行、启动、增加监听创建监听，设置超时时间等等

![image-20200916200826231](https://i-blog.csdnimg.cn/blog_migrate/50eee8e096254ac337b5162e4d43c965.png)

##### AsyncListener

异步监听，主要是为异步控制器服务的。

监听器监听4种事件：

![image-20200916201001154](https://i-blog.csdnimg.cn/blog_migrate/38ef3e9602492472bc64514718584626.png)

启动、执行、超时和异常。

##### DispatcherType

![image-20200916201117483](https://i-blog.csdnimg.cn/blog_migrate/b9e4ac4e5e59876bfaa884eb229e93e2.png)

分发类型包含转发、包含、要求、异步和异常。

#### ServletResponse

控制器响应。

首先就是编码

![image-20200916201304932](https://i-blog.csdnimg.cn/blog_migrate/2cc0c78ee4a9fa23d36604ddfdc798e6.png)

然后是内容类型、长度和控制器输出流。

![image-20200916201346439](https://i-blog.csdnimg.cn/blog_migrate/5358a2cc1ffd2674086e7a1f1f263b6a.png)

以及响应缓存

![image-20200916201427606](https://i-blog.csdnimg.cn/blog_migrate/31c3e0937995e86db2f11e86a2c08bf8.png)

控制器响应有事务：

![image-20200916201508930](https://i-blog.csdnimg.cn/blog_migrate/4dd7d8d81851484159dcf55e8594da28.png)

总结：`WebApplicationContext`定义了完整的Web过程，包括请求，分发，过滤，监听以及响应。当然还有异步。

### WebServerApplicationContext

`WebServerApplicationContext`继承了ApplicationContext接口，除了继承了`ApplicationContext`接口的方法外，还定义了自己的操作：

  1. 获取命名空间
  2. 获取网页服务

![image-20200917185549048](https://i-blog.csdnimg.cn/blog_migrate/5fcbfe39c7f26638b18e1ed93926e9c0.png)

#### WebServer

WebServer定义了网页服务的操作：

  1. 开始
  2. 结束
  3. 获取端口

![image-20200917185919274](https://i-blog.csdnimg.cn/blog_migrate/4d01fb29f68fcb618fdd3fa7c893be07.png)

### ConfigurableApplicationContext

`ConfigurableApplicationContext`继承了`ApplicationContext`以及`Lifecycle`和`Closeable`接口。

`ConfigurableApplicationContext`定义了设置Context的属性的操作。

获取环境变量配置，添加`BeanFactoryPostProcessor`处理器(beanFactory后置处理器)，添加应用监听，添加协议解析器，定义了`refresh`操作，以及注册程序终止钩子，是否还存活，以及获取依赖处理的BeanFactory。

#### Lifecycle

声明周期接口，只定义了三个操作：

  1. start
  2. stop
  3. isRunning

![image-20200917191551134](https://i-blog.csdnimg.cn/blog_migrate/42495ceafc880f0946e0285b7a898603.png)

#### Closeable

资源关闭接口。

定义了资源关闭的方法

![image-20200917191637935](https://i-blog.csdnimg.cn/blog_migrate/bd0d12f26e9d60a7f678332d4bfdcf32.png)

#### ConfigurableEnvironment

可配置的环境变量。

定义了这些方法：

  1. 设置活动配置
  2. 增加活动配置
  3. 设置默认的配置
  4. 获取配置
  5. 合并配置

![image-20200917191914032](https://i-blog.csdnimg.cn/blog_migrate/abdeac54a899b567bb99ef97244ee2f9.png)

#### ProtocolResolver

协议解析接口。

定义了解析操作。

![image-20200917192022251](https://i-blog.csdnimg.cn/blog_migrate/c758d5302180f1a60ff7865e5873cbc3.png)

### ReactiveWebApplicationContext

`ReactiveWebApplicationContext`继承于`ApplicationContext`接口，不过，`ReactiveWebApplicatonContext`没有定义任何操作，应该是一个标记性接口。

![image-20200917192224118](https://i-blog.csdnimg.cn/blog_migrate/2c17ad8ebf6fb286999ed83f0c13a17c.png)

### ConfigurableWebApplicationContext

`ConfigurableWebApplicationContext`继承了`WebApplicationContext`接口，也继承了`ConfigurableApplicationContext`接口。

定义的操作：

  1. 设置ServletContext，控制器上下文
  2. 设置ServletConfig，控制器配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8a6521980535f81178ddc6462638240.png#pic_center)

#### ServletConfig

控制器配置。

定义了这些配置操作：

  1. 设置控制器名字
  2. 设置控制器上下文
  3. 获取初始化参数

![image-20200917192721935](https://i-blog.csdnimg.cn/blog_migrate/1fb6c8093cf0c3719c8b95552fa1a6eb.png)

### ConfigurableWebServerApplicationContext

`ConfigurableWebServerApplicationContext`继承了`ConfigurableApplicationContext`和`WebServerApplicationContext`接口。

它只定义了一个方法，就是设置服务命名空间的方法。

![image-20200917193238249](https://i-blog.csdnimg.cn/blog_migrate/5a165d8bf37739b32e709f1b92189dee.png)

### ApplicationContextAssertProvider

`ApplicationContextAssertProvider`继承了`ApplicationContext`和`AssertProvider`以及`Closeable`接口。

`ApplicationContextAssertProvider`用到了泛型，定义了类型上界是`ApplicationContext`。如果是一个BeanFactory就无法使用`ApplicationContextAssertProvider`。

定义的方法：

  1. 返回一个断言(这个断言用于AspectJ切面)
  2. 获取源程序上下文
  3. 获取源程序上下文，并转为指定的类型(做类型消除)
  4. 获取启动失败异常(返回值是Throable，意思是，不管是异常还是错误，都能返回)
  5. 关闭资源接口
  6. 获取代理类

![image-20200917194308997](https://i-blog.csdnimg.cn/blog_migrate/fe584fbcfcc3fcd3a7963223ee5f68fe.png)

#### AssertProvider

断言提供者

![image-20200917194350918](https://i-blog.csdnimg.cn/blog_migrate/06376284c24c757c239c97bae7068d25.png)

### ConfigurableReactiveWebApplicationContext

`ConfigurableReactiveWebApplicationContext`继承了`ConfigurableApplicationContext`和`ReactiveWebApplicationContext`接口

同时`ConfigurableReactiveWebApplicationContext`是一个标记性接口，没有定义任何操作。

![image-20200917200547268](https://i-blog.csdnimg.cn/blog_migrate/3cc37e627c43e8c7ecd08afdb5e6c6f9.png)

### AssertableWebApplicationContext

`AssertableApplicationContext`继承了`ApplicationContextAssertProvider`和`ConfigurableWebApplicationContext`接口。

重写了`ApplicationContextAssertProvider`中的get方法。重写了获取代理实例的方法。

![image-20200917201035145](https://i-blog.csdnimg.cn/blog_migrate/b80ef47dcd8c2b17fbf3f55b8f2f4bdb.png)

### AssertableApplicationContext

`AssertableApplicationContext`继承了`ApplicationContextAssertProvider`和`ConfigurableApplicationContext`接口。

重写了`ApplicationContextAssertProvider`中的get方法。重写了获取代理实例的方法。

![image-20200917201000214](https://i-blog.csdnimg.cn/blog_migrate/f5c8bdd0b1850ab5aedafa62f6a35106.png)

### AssertableReactiveWebApplicationContext

`AssertableApplicationContext`继承了`ApplicationContextAssertProvider`和`ConfigurableReactiveWebApplicationContext`接口。

重写了`ApplicationContextAssertProvider`中的get方法。重写了获取代理实例的方法。

![image-20200917200935910](https://i-blog.csdnimg.cn/blog_migrate/ed8579a67eb5dd98da50c153995b186e.png)

## 实现

![image-20200914194242755](https://i-blog.csdnimg.cn/blog_migrate/310bdb25a3aca09dd50b5d87aced6f2b.png)

我们采用自顶向下的方式阅读

### AbstractApplicationContext

`AbstractApplicationContext`继承了`DefaultResourceLoader`类，实现了`ConfigurableApplicationContext`接口。

`AbstractApplicationContext`内部定义了国际化bean的名字，生命周期处理器bean的名字以及应用处理器的bean的名字。

`AbstractApplicationContext`拥有其他父上下文`ApplicationContext`的属性。

还有环境变量的配置的属性，以及beanFactory后置处理器。

线程安全的布尔属性，用于标识当前上下文是否存活 ，是否关闭。

里面还定义了一个启动，终止的监视器。是一个Object对象，应该充当锁。

上下文被终止的钩子，因为上下文结束了，整个应用程序就结束了，所以这个钩子是独立于上下文之外的线程。用一个Thread保存。

上下文还持有了国际化的服务，程序事件传播器(`ApplicationEventMulticaster`)。

还有这个上下文的一堆的监听器。

当前上下文自己的一堆前置监听器。

以及当前上下文自己的一堆程序事件。

![image-20200918192346559](https://i-blog.csdnimg.cn/blog_migrate/ba28b1bc7ee946232e68aeeb60ab77f4.png)

发布事件：

![image-20200918192503461](https://i-blog.csdnimg.cn/blog_migrate/1de6ccbbfd1e8ab7db8c795d5570f081.png)

![image-20200918192518806](https://i-blog.csdnimg.cn/blog_migrate/4941a44d3cc413f789de34b4f3939ec3.png)

事件传播器要求子类提供，否则抛出异常。

![image-20200918192611271](https://i-blog.csdnimg.cn/blog_migrate/bdae954d304bee0c63854372539be493.png)

接下来就是非常经典的一个过程了：

![image-20200918192726222](https://i-blog.csdnimg.cn/blog_migrate/32789606ecc812f83ca318c869312400.png)

![image-20200918192854447](https://i-blog.csdnimg.cn/blog_migrate/51e1c3d97f18dd01bab8517e6e5fe3a4.png)

![image-20200918192906054](https://i-blog.csdnimg.cn/blog_migrate/1ee02ca7e28e40eb7c5f6ea35649105c.png)

刷新方法。

这些方法有哪些呢：

  1. prepareRefresh:上下文刷新准备操作。

![image-20200918200412450](https://i-blog.csdnimg.cn/blog_migrate/b3ebab0cea655fb0a09c1ae3e16f6745.png)

1.1 初始化配置信息

1.2 验证配置信息

  2. prepareBeanFactory:beanFactory刷新准备操作。

2.1 refreshBeanFactory:刷新beanFactory

2.2 getBeanFactory:获取beanFactory

  3. prepareBeanFactory:beanFactory刷新准备操作。主要是将bean依赖处理，忽略类型，bean合并等操作的类注册到beanFactory中。

![image-20200918193401849](https://i-blog.csdnimg.cn/blog_migrate/0111aad16b8969c56e58d4211dde3204.png)

  4. postProcessBeanFactory:beanFactory后置处理器。

  5. invokeBeanFactoryPostProcessors:执行beanFactory后置处理器，并且将beanFactory后置处理器注册到beanFactory中。

  6. regstoryBeanPostProcessors:注册前置处理器

![image-20200918194325962](https://i-blog.csdnimg.cn/blog_migrate/51f257039dd0440531a6819368267c2f.png)

  7. initMessageSource:初始化国际化

![image-20200918194429417](https://i-blog.csdnimg.cn/blog_migrate/e3beaaa42b65419fa9d3ce320a3f8731.png)

  8. initApplicationEventMulticaster:初始化程序事件传播器。(其实就是注册了一个特别的bean)

![image-20200918194600139](https://i-blog.csdnimg.cn/blog_migrate/1f0a56ec0791585feb8c9cf355d68245.png)

  9. onRefresh:刷新，初始化其他bean

  10. registerListeners:注册监听器。(包括事件传播器和被传播的bean的名字以及传播事件)

![image-20200918194749208](https://i-blog.csdnimg.cn/blog_migrate/f832256493e395c823e7a0626ac14143.png)

  11. finishBeanFactoryInitialization:剩余bean的初始化(非懒加载的)(剩余的bean包括环境变量值的加载，以及一些自动装配的类的加载)

![image-20200918195012921](https://i-blog.csdnimg.cn/blog_migrate/523f6bb33feba1645e5a41f7d71db815.png)

  12. finishRefresh:发布事件，告诉其他上下文，当前上下文刷新创建完毕。

12.1 clearResourceCaches:清楚资源缓存

12.2 initLifecycleProcessor:初始化生命周期处理器。(其实就是注册了一个特殊的bean)

![image-20200918195413465](https://i-blog.csdnimg.cn/blog_migrate/13b75115d99401509cc8b9fb674519ac.png)

12.3 getLifecycleProcessor().refresh():执行生命周期处理的操作。

12.4 publishEvent:发布上下文刷新的事件：

12.5 LiveBeanView.registory:存活的上下文注册(将当前Context注册到存活的Context集合中)

  13. destoryBeans:销毁bean.如果出现异常，就会执行这个方法，清空当前上下文创建的bean。

![image-20200918195931759](https://i-blog.csdnimg.cn/blog_migrate/26c5c8e28fb2e175de8982f2ae6ab0ff.png)

  14. cancelRefresh:取消刷新。(设置存储状态为否)

  15. resetCommonCaches:重置普通的缓存

![image-20200918200048643](https://i-blog.csdnimg.cn/blog_migrate/d90d5727391fd25d8864462c6910837b.png)

refresh核心的操作：刷新准备，前后置处理器，自定义初始化bean这几个操作。

上下文关闭需要做的操作：

  1. 将当前上下文从存活上下文中移除
  2. 发布上下文关闭的事件
  3. 销毁beans
  4. 关闭beanFactory
  5. 执行自定义的关闭操作
  6. 设置上下文存活标志为否

![image-20200918200807371](https://i-blog.csdnimg.cn/blog_migrate/cc0dfbde08090374b1ac68659a6f26ef.png)

#### ApplicationEventMulticaster

上下文时间传播器。是一个接口，里面定义了增加、移除以及传播上下文监听器的方法。

![image-20200918192122793](https://i-blog.csdnimg.cn/blog_migrate/b9524acf923970ea584db4ed7fd007bb.png)

#### DefaultResourceLoader

`DefaultResourceLoader`实现了`ResourceLoader`接口。

默认的资源加载器内部持有资源的缓存和协议解析器。同样还持有一个类加载器。

核心的方法，获取资源。就是从资源的缓存中查询，如果缓存中不存在，那么，就使用协议解析器，解析加载资源。

![image-20200918191142620](https://i-blog.csdnimg.cn/blog_migrate/8d66ff69ab4fd8c881499de1e9ea2e53.png)

#### ResourceLoader

`ResourceLoader`接口定义了两个操作：

![image-20200918190347812](https://i-blog.csdnimg.cn/blog_migrate/22aac0f32566cdbbf4cbc76f0f1e9f79.png)

一个是获取资源，另一个是获取类加载器。

### GenericApplicationContext

通用程序上下文。

`GenericApplicationContext`继承了`AbstractApplicationContext`抽象类，实现了`BeanDefinitionRegistry`接口。

通用上下文内部持有了可列举的beanFactory,以及资源加载器和是否客户化类加载的标志和是否进行刷新的标志。

![image-20200918201937433](https://i-blog.csdnimg.cn/blog_migrate/41eb7c17f8a2134acc213979778a9efd.png)

不过大多是调用抽象类的实现

![image-20200918202021895](https://i-blog.csdnimg.cn/blog_migrate/e83e3a18c5b7574261cf01f6b6ffb727.png)

通用程序上下文在获取bean的时候，会做通用的操作(设置序列化id)

![image-20200918202237640](https://i-blog.csdnimg.cn/blog_migrate/12fe0749f1cfa1f751e0955275217ddc.png)

以及通用程序上下文在取消刷新的时候，取消设置序列化id，以及调用抽象类的取消刷新。

![image-20200918202350305](https://i-blog.csdnimg.cn/blog_migrate/01bdc6fc44289d97bc54f066f84bfaea.png)

### AbstractRefreshableApplicationContext

抽象可刷新的应用上下文。

`AbstratcRefreshableApplicationContext`继承于`AbstractApplicationContext`。

`AbstracttRefreshableApplicationContext`内部持有三个属性：是否可覆盖类信息，允许循环引用，默认可列出的beanFactory

![image-20200919134740753](https://i-blog.csdnimg.cn/blog_migrate/c4369341eea963f4389aeb0e9421fef5.png)

抽象可刷新的应用上下文主要实现的操作也是refresh刷新beanFactory前的准备操作。

![image-20200919135219433](https://i-blog.csdnimg.cn/blog_migrate/527e85afbaceee82ef76ea8d33193a86.png)

核心的两个操作：客户化处理beanFactory，加载bean类信息。

客户化处理beanFactory，主要是根据上下文的属性，设置beanFactory的属性。

![image-20200919135505865](https://i-blog.csdnimg.cn/blog_migrate/12e8a6bedde82e2eb12d71f74818764a.png)

其中加载beanFactory类信息是一个抽象方法，要求子类实现。

![image-20200919135416465](https://i-blog.csdnimg.cn/blog_migrate/cfa19d1163702d65dbb044cea4777c68.png)

取消刷新和关闭beanFactory的操作类似：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/405a096c4204554df131b9c4f889aef7.png#pic_center)

将beanFactory的序列化id设置为空。

### AbstractRefreshableConfigApplicationContext

抽象可刷新可配置的应用上下文。

`AbstractRefreshableConfigApplicationContext`继承于`AbstractRefreshableApplicationContext`，同时实现了`BeanNameAware`和`InitializingBean`接口。

这个抽象的上下文的属性就很少了，也是非常的简单：

配置的参数(毕竟这个应用上下文是可配置的)，是否调用过setId方法标志。

![image-20200919140037134](https://i-blog.csdnimg.cn/blog_migrate/ff2fbfac7b1d503b951a6a00422e7db1.png)

OK，接下来看看实现的两个接口的实现操作：

`BeanNameAware`接口定义了了一个setBeanName方法。

![image-20200919140326196](https://i-blog.csdnimg.cn/blog_migrate/bbb9319423060c1ab72a38194921a55f.png)

给beanName加了一个前缀。

`InitializingBean`接口定义了一个afterPropertiesSet方法，这个方法在beanFactory创建完bean后调用，也就是beanFactory的后置处理器。

![image-20200919140504713](https://i-blog.csdnimg.cn/blog_migrate/9bec913357101850d165891ab8496c18.png)

这个方法的逻辑也是非常的简单，如果应用程序上下文存活，那么调用refresh方法刷新全部的bean.

这里体现的是可刷新这一个特性。

这也是到目前为止，我们找到的第一个调用referesh方法的地方。

### StaticApplicationContext

静态应用上下文。

`StaticApplicationContext`继承了`GenericApplicationContext`。

静态应用上下文继承了通用应用程序上下文。  
内部只有一个静态国际化的服务。

![image-20200919141046239](https://i-blog.csdnimg.cn/blog_migrate/42738afeb5718232b0ca981f9a843fa7.png)

静态应用上下文在构造的时候，向beanFactory中注册了静态国际化的bean。

![image-20200919141250218](https://i-blog.csdnimg.cn/blog_migrate/c60b8372846f4a4a2aeb785011f3b4b8.png)

还对外开放了单例注册方法。

![image-20200919141333657](https://i-blog.csdnimg.cn/blog_migrate/38d724c0b77d8feb14442b038a76700e.png)

#### StaticWebApplicationContext

静态网络应用上下文。

`StaticWebApplicationContext`继承于`StaticApplicationContext`，并且实现了`ConfigurableWebApplicationContext`和`ThemeSource`接口。

`StaticWebApplicationContext`的属性包含了`ServletContext`,`ServletConfig`,命名空间和主题服务。

![image-20200919143448299](https://i-blog.csdnimg.cn/blog_migrate/0af0bd2274e34534468de74519ed48c6.png)

主要实现的操作

首先是beanFactory后置处理器。

![image-20200919143706648](https://i-blog.csdnimg.cn/blog_migrate/aab995dc40f259f80b6a73e4f36c54f6.png)

在beanFactory后置处理器中，首先增加了beanFactory后置处理器，接着将`ServletContextAware`和`ServletConfigAware`这两个类设置为忽略的依赖类。

接着将该上下文的属性，注册到beanFactory中。

还实现了onRefresh方法：onRefresh中主要是设置当前上下文的主题服务。

![image-20200919144027366](https://i-blog.csdnimg.cn/blog_migrate/1e33c71fda2e9e1a2a84443493170ef1.png)

在实现的initPropertySources中，也是设置当前上下文的控制器上下文和控制器配置。

### GenericGroovyApplicationContext

通用Groovy应用上下文。

`GenericGroovyApplicationContext`继承于`GenericApplicationContext`，而且实现了`GroovyObject`接口。

其中GroovyObject是一个接口，因为没有源码，也没有找到相关的class类，所以不知道GroovyObject中定义了什么操作。

在网上搜了下，原来`GroovyObject`类似于java中`Object`的地位。

Groovy中所有的类都实现了`GroovyObject`接口。

于是我创建了一套Groovy环境，查看下GroovyObject接口定义了一些什么操作。

![image-20200919144839551](https://i-blog.csdnimg.cn/blog_migrate/2eabc1845a36cc8486e2caf1cbc3f44d.png)

定义的操作：

  1. 执行方法
  2. 获取属性
  3. 设置属性
  4. 获取类的元数据
  5. 设置类的元数据

`GenericGroovyApplicationContext`中的属性也有`GroovyBeanDefinitionReader`和上下文包装以及类的元数据(这个也是在Groovy中定义的)

![image-20200919145135377](https://i-blog.csdnimg.cn/blog_migrate/9ba1fc856ffdd3f1d0e2e28213f799de.png)

这个没啥好说的，是兼容Groovy实现的通用应用上下文。

内部实现的方法也是`GroovyObject`接口定义的方法。

### GenericXmlApplicationContext

通用xml应用上下文。

`GenericXmlApplicationContext`需要解析的是xml，在XmlBeanFactory中，就使用了`XmlBeanDefinitionReader`用于解析和设置Bean，所以在`GenericXmlApplicationContext`中也有一个属性就是`XmlBeanDefinitionReader`。

![image-20200919153741059](https://i-blog.csdnimg.cn/blog_migrate/b24b6e5181f922417270d2635b8d5319.png)

在`GenericXmlApplicationConext`的构造函数中，主要调用了load和refresh方法。

这是目前为止，遇到的第二个调用refresh的应用上下文。

![image-20200919153933894](https://i-blog.csdnimg.cn/blog_migrate/85bb192e28481ec5c395de494682dadc.png)

load方法就是直接调用`XmlBeanDefinitionReader`的load方法

![image-20200919154015293](https://i-blog.csdnimg.cn/blog_migrate/9375ca43f2ea07290616392913f07c44.png)

refresh方法就是调用`AbstractApplicationContext`中定义的refresh方法。

而且`GenericXmlApplicationContext`还开放了获取`XmlBeanDefinitionReader`的方法

![image-20200919154145170](https://i-blog.csdnimg.cn/blog_migrate/d1ea2fdd563cd0983aba12d1bedb5559.png)

### AnnotationConfigApplicationContext

注解配置应用上下文。

`AnnotationConfigApplicationContext`继承于`GenericApplicationContext`，实现了`AnnotationConfigRehistry`接口。

`AnnotationConfigApplicationContext`中有两个属性，用于支持解析注解。

分别是注解类信息解析器和类路劲类信息扫描器。

![image-20200919154623240](https://i-blog.csdnimg.cn/blog_migrate/5b92d65f6ce548d6f2424ed0077e9f23.png)

在`AnnotationConfigApplicationContext`的构造方法中，主要执行了扫描操作和刷新操作。

![image-20200919163440886](https://i-blog.csdnimg.cn/blog_migrate/bc20a8645ca47ae743897f02eacbfcbb.png)

当然，`AnnotationConfigApplicationContext`既可以支持包扫描，也支持类型扫描。

如果是包扫描，就调用扫描器，如果是类型扫描，就调用类型解析器。

这里调用了refresh方法。

#### AnnotationConfigRegiustry

注解配置注册器。

`AnnotationCOnfigRegistry`接口只定义了两个方法：注册和扫描包的方法

![image-20200919154508068](https://i-blog.csdnimg.cn/blog_migrate/5df982196a312e8f7e3fff90d64ad7b2.png)

#### AnnotatedBeanDefinitionReader

注解类信息解析器。

`AnnotatedBeanDefinitionReader`中有一些属性，帮助解析注解：bean信息注册器，beanName生成器，bean作用域解析器，条件执行器。

![image-20200919155038728](https://i-blog.csdnimg.cn/blog_migrate/03d33072bd785f79cef4e4ad24d88450.png)

作为一个解析器，最重要的方法就是解析。

传入一个注解类，要求解析某一类注解的bean。

![image-20200919155220245](https://i-blog.csdnimg.cn/blog_migrate/eb99a8dfaf06ad0f1c8492f367d50c66.png)

可以看到，注释中给出的例子是`@Configuration`注解。

内部实际调用的是registerBean方法。

![image-20200919155348199](https://i-blog.csdnimg.cn/blog_migrate/c48790329ce44fdee72afedc35ecc037.png)

真正的逻辑在doRegisterBean方法中：

![image-20200919155500998](https://i-blog.csdnimg.cn/blog_migrate/729a3158b31318bd4c946e9a7a0c088e.png)

![image-20200919155514968](https://i-blog.csdnimg.cn/blog_migrate/5ec2210bed39f6baff0739a01353e9d2.png)

doRegisterBean方法的参数很多，第一个是注解的class，第二个是指定的beanName，第三个是限定的注解，第四个是客户化的操作，第五个是自定义的类信息处理器。

可以看到常见的，如果有限定注解，在`AnnotatedBeanDefinitionReader`中是`@Primary`和`@Lazy`注解。

#### ClassPathBeanDefinitionScanner

前面的`AnnotatedBeanDefinitionReader`只是加载某一个注解的bean到容器内。

而`ClassPathBeanDefinitionScanner`则是扫描某一个包下全部的bean。

所以，`ClassPathBeanDefinitionScanner`需要做的比`AnnotatedBeanDefinitionReader`的多。

类路径类信息扫描器。

`ClassPathBeanDefinitionScanner`中有类信息注册器，默认的类信息，自动匹配的候选，beanName生成器，bean作用域解析器，是否包含注解配置标志。

![image-20200919160615660](https://i-blog.csdnimg.cn/blog_migrate/2f9956ff17a05374355d4c64fec5f607.png)

`ClassPathBeanDefinitionScanner`的构造器中，根据传入的参数，判断是否需要过滤排除的类信息。

![image-20200919161205742](https://i-blog.csdnimg.cn/blog_migrate/781209c7161bab010af13f533980ce38.png)

使用默认的过滤器进行解析类路径下的类信息。就会注册默认的类信息过滤器，这个注册方法是`ClassPathBeanDefinitionScanner`的父类的方法：

![image-20200919161535162](https://i-blog.csdnimg.cn/blog_migrate/0644a27dc01bffbbd400995b9ca1ecb0.png)

在这里将`@Component`注解注册到父类的倒入的过滤器中。(这里就是`@Component`注解的解析的逻辑)

![image-20200919161622856](https://i-blog.csdnimg.cn/blog_migrate/4fd98f8b1eeea954bf1d4de6aa39d045.png)

在父类中有两个过滤器，一个是倒入的，一个排除的过滤器。换句话说就是，在类路径下，一部分是需要的，一部分是不需要的。

需要的和不需要的通过注解类型进行区分。

这个方法返回默认候选的类信息。

作为一个扫描器，最重要的一定是扫描方法。

![image-20200919162359538](https://i-blog.csdnimg.cn/blog_migrate/5b162891492541d293a1b55aa8199293.png)

这里调用了doScan方法，传入的是一个String数组

![image-20200919162433479](https://i-blog.csdnimg.cn/blog_migrate/a02527a470effc8a1c2672eef859a4df.png)

对传入的每一个需要扫描的路径，首先获取默认候选的类信息(findCandidateComponts):

![image-20200919162535374](https://i-blog.csdnimg.cn/blog_migrate/d278c2720b32b43f5ec086d4fb343e36.png)

直接扫描，可能存在变种的注解，此时就需要手动指定变种的注解，或者是默认的注解(修改过，如果没有修改过，默认是`@Component`注解)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/546f19c903e074473043c1230d763336.png#pic_center)

另一个方法和这个差不多

![image-20200919162624347](https://i-blog.csdnimg.cn/blog_migrate/a060f0044ea2ce9f775ea08083c91cd3.png)

区别就在于是否存在`CandidateComponentsIndex`参数，也就是是否存在多个映射关系。（一般来说，这里获取的是指定路径下`@Component`注解的类）

接着就是设置作用域，注册类信息，以及类信息注册的beanFactory后置处理器(类信息注册到类信息BeanFactory中，说白了，类信息也是一个bean，也有自己的beanFactory后置处理器)

![image-20200919163054008](https://i-blog.csdnimg.cn/blog_migrate/97960295e0f6794396fc4b13367ccb25.png)

使用初始化设置进行更新，有些信息可能没有加载到，那么就需要使用默认只进行初始化。同时设置这些类都是自动装配的。换句话说，加了注解的bean都是自动装配的。

有了类信息，就可以根据类信息生成bean了。

#### AnnotationConfigReactiveWebApplicationContext

注解可配置的活性的网络应用上下文。

`AnnotationConfigReactiveWebApplicationContext`继承于`AnnotationConfigApplicationContext`类，实现了`ConfigurableReactiveWebApplicationContext`接口。

其中`ConfigurableReactiveWebApplicationContext`接口是一个标记性接口，没有定义任何操作。

`AnnotationConfigReactiveWebApplicationContext`没有实现什么比较特殊的操作，只是将类路径下的资源做了一层包装。

![image-20200919170944371](https://i-blog.csdnimg.cn/blog_migrate/f4775c2eb093887d7c097495b5270cde.png)

### GenericWebAppliicationContext

通用网络应用上下文。

`GenericWebApplicationContext`继承了`GenericApplication`,实现了`ConfigurableWebApplicationContext`和`ThemeSource`接口。

> ThemeSource其实就是将MessageSource进行了封装。

`ConfigurableWebAppliicationContext`接口定义了如何向上下文设置控制器上下文和控制器配置。

因此，通用网络应用上下文实现了设置控制器上下文和控制器配置。同时这部分也是与其他上下文不同的地方。

通用的网络应用上下文只有控制器上下文这一个属性，没有见到有控制器配置属性。

![image-20200919171443095](https://i-blog.csdnimg.cn/blog_migrate/47185581a927e40c404468c451d313df.png)

通用网络应用上下文的beanFactory后置处理器是将控制器上下文注册到beanFactory中，同时设置忽略控制器上下文相关的类型。

![image-20200919171600781](https://i-blog.csdnimg.cn/blog_migrate/915b57cab072211e608bf7a22092555e.png)

通用网络应用上下文实现了onRefresh方法，在onRefresh方法中主要是初始化了主题，也就是国际化。

![image-20200919171734362](https://i-blog.csdnimg.cn/blog_migrate/2012db21404ed307ab3f4ed4871bda9e.png)

因为通用网络应用上下文没有定义控制器配置属性，所以，实现的控制器配置相关的方法都是空的或者异常

![image-20200919171824126](https://i-blog.csdnimg.cn/blog_migrate/643ce1ef658d571bc49ec48bd19e7632.png)

命名空间也是如此，还有本地化配置。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb4762127b9d981d01bf07afbce74994.png#pic_center)

也就是说，通用网络应用上下文其实是一个半成品，不能直接通用。

#### AnnotationConfigServletWebApplicationContext

注解配置的控制器网络应用上下文。

`AnnotationConfigServletWebApplicationContext`继承了`GenericWebApplicationContext`同时实现了`AnnotationConfigRegistry`接口。

这个上下文其实就是将通用网络应用上下文和注解配置应用上下文缝合起来的。实现的`AnnotationConfigRegistry`接口其实就是说明，注解配置的控制器网络应用上下文既可以按照类型扫描，同时也可以按照路径扫描。

这是属性，也就是注解配置应用上下文的属性。（因为集成了通用网络应用上下文，所以通用网络应用上下文的属性就没有必要重写了，根据继承原则，可以直接使用）

![image-20200919172456546](https://i-blog.csdnimg.cn/blog_migrate/37dbf18ba78263f2c0e8ca4b679c40cd.png)

实现的方法也主要是注解配置应用上下文，因为通用网络上下文的可以直接使用

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cd85dea835f73c28a93bc3661537a7fa.png#pic_center)

这时候想着，如果Java中有多继承该有多好啊，这里就不用重新写一遍了，直接同时继承这两个上下文就好了。

唯一的例外是重写了刷新准备方法，在`AbstractApplicationContext`的准备的基础上，增加了扫描器缓存的清理操作。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e632541ad8400f0765a55479c33d024e.png#pic_center)

#### ServletWebServerApplicationContext

控制器网络服务应用上下文。

`ServletWebServerApplicationContext`继承于`GenericWebApplicationContext`，实现了`ConfigurableWebServerApplicationContext`接口。

因为控制器网络服务上下文继承了通用网络应用上下文，所以控制上下文就不用重写了，主要实现的是可配置的网络服务应用上下文接口中的方法。

所以在控制器网络服务应用上下文的侧重点就是服务相关的。网络相关的在通用网络应用上下文中都做了。

![image-20200919175959917](https://i-blog.csdnimg.cn/blog_migrate/50eab1099fcfdef1ee5e6cf69efa2bdd.png)

所以在控制器网络服务应用上下文的属性主要是网络服务，控制器配置，服务命名空间。

在父类通用网络应用上下文中，我们知道，没有实现控制器配置和服务命名空间，在这里实现的。

而网络服务主要是实现可配置的网络服务应用上下文中的接口的方法。

在控制器网络服务上下文的beanFactory后置处理中，主要是忽略控制器上下文相关的类，以及注册网络作用域(request,reponse,session,application,page…)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7344a88cca417313e60cfae03d6b1bee.png#pic_center)

网络作用域使用了内部类实现的

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/841857857478b948dc9f3fbc18c08876.png#pic_center)

初始化的时候，只初始化了request和session

![image-20200919180355284](https://i-blog.csdnimg.cn/blog_migrate/dac239a39bcbbdb5326416c938746cd4.png)

控制器网络服务应用上下文也重写了refresh方法，在refresh方法中捕获了父类refresh中的全部运行时异常，如果出现异常，需要将网络服务关闭。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1ad6ccbfb9901ea467dc6e3a52446928.png#pic_center)

控制器网络服务应用上下文还重写了onRefresh方法，在父类的onRefresh方法之后，创建了网络服务。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3110216f0d76cc44654c49527242189e.png#pic_center)

如果网络服务和控制器上下文都没有被初始化，那么就进行初始化，并注册相关的bean到beanFactory中，否则就启动控制器上下文，然后初始化配置。

![image-20200919180906819](https://i-blog.csdnimg.cn/blog_migrate/b073fb4d3e7746a3720f838ae5abde16.png)

初始化主要是创建了网络服务对象，然后将网络服务对象设置到属性，然后注册了网络服务终止的生命周期bean和网络服务启动停止的生命周期bean.

这两个bean分别是`WebServerGracefulShutdownLifecycle`和`WwebServerStartStopLifecycle`。

如果控制器上下文不为空，那么就会调用安全的初始化器，进行初始化启动控制器上下文。

![image-20200919182218097](https://i-blog.csdnimg.cn/blog_migrate/26a53a560a16d972e34f02866cd7cc18.png)

安全初始化主要是网络应用上下文准备操作和注册application的作用域。

应用上下文准备中主要是设置网络的父容器的属性以及设置控制器上下文的父容器的属性。

![image-20200919182734656](https://i-blog.csdnimg.cn/blog_migrate/6a526d0d2ea419694a12fdb0ee7b0b30.png)

application作用域注册，不仅需要注册到应用上下文中，还需要注册到控制器上下文中。

内部的每一个初始化器都需要执行。

最后就是获取环境变量配置器，然后使用环境变量配置器初始化属性。

![image-20200919182349953](https://i-blog.csdnimg.cn/blog_migrate/f8d18bf994e2b8d8df4d5a536e226c08.png)

在一个控制器网络服务应用上下文中只能存在一个网络服务的容器，多于1个少于1个都不行

![image-20200919182512492](https://i-blog.csdnimg.cn/blog_migrate/2a11022ba4f071ba697036c9961eacba.png)

##### WebServerGracefulShutdownLifecycle

网络服务正常关闭的生命周期。

正常关闭的生命周期实现了SmartLifecycle。

正常关闭的生命周期主要是需要实现接口定义的操作，包括启动，停止，判断是否正在运行等方法。

![image-20200919181837652](https://i-blog.csdnimg.cn/blog_migrate/31fb6b41e6f5bf288eb7af6c8b7a5ef4.png)

###### SmartLifecycle

智能生命周期接口。

`SmartLifecycle`继承了Lifecycle和Phased接口。

`SmartLifecycle`接口主要定义了一些通用的操作，比如停止操作。停止之后应该调用相关的回调。

![image-20200919181747711](https://i-blog.csdnimg.cn/blog_migrate/79f86bc349ae713d2cdb5de4cd6736bc.png)

###### Lifecycle

生命周期接口。

定义了三个方法：启动，停止，是否正在运行。

![image-20200919181601505](https://i-blog.csdnimg.cn/blog_migrate/d68ae6197fc3ba5f1ca48113d2f5ea92.png)

###### Phased

阶段接口。

定义了获取当前对象的阶段的方法。

![image-20200919181653702](https://i-blog.csdnimg.cn/blog_migrate/be58936925834876f3887094ec699911.png)

##### WebServerStartStopLifecycle

网络服务启动停止生命周期。

网络服务启动停止也是需要实现`SmartLifecycle`接口。

主要也是启动，停止和判断是否正在运行的方法。

![image-20200919182016980](https://i-blog.csdnimg.cn/blog_migrate/394a5d928dba9521ddcdae2838fcad9e.png)

和正常关闭的相比，启动之后需要发布事件，发布一个网络服务初始化的事件。在start方法中。

##### AnnotationConfigServletWebServerApplicationContext

注解配置控制器网络服务应用上下文。

`AnnotationConfigServletWebServerApplicationContext`继承了`ServletWebServerApplicationContext`，实现了`AnnotationConfigRegistry`接口。

注解配置控制器网络服务应用上下文继承了控制器网络服务应用上下文，实现了注解配置的接口。

这个上下文也没有什么新的操作，核心就是继承控制器网络服务的上下文，然后实现注解配置的方法。

因为是继承了控制器网络服务应用上下文，所以主要是将注解配置的相关方法拷贝即可。

所以注解配置控制机器网络服务上下文主要的属性都是注解配置相关的

![image-20200919183456710](https://i-blog.csdnimg.cn/blog_migrate/e360766b7e045366eb9c3b91d0ddd654.png)

属性包含了注解解析的注册器和扫描器。

构造方法中也和注解配置应用上下文中的构造方法相同

![image-20200919183549697](https://i-blog.csdnimg.cn/blog_migrate/f09cf1cd427d096e60cdc7a8ec700c7d.png)

你甚至可以理解为将注解应用上下文中的方法拷贝过来就行。

核心的方法完全相同：

![image-20200919183647928](https://i-blog.csdnimg.cn/blog_migrate/b2eee89a470428fac26b779f96ff7319.png)

##### XmlServletWebServerApplicationContext

xml控制器网络服务应用上下文。

`XmlServletWebServerApplicationContext`继承了`ServletWebServerApplicationContext`。

这个和注解配置控制器网络服务应用上下文类似，只是将注解配置换成了xml配置。

到目前为止，xml解析使用的都是同一个xml解析器`XmlBeanDefinitionReader`。

所以xml控制器网络服务应用上下文的唯一属性就是xml解析器。

![image-20200919184602823](https://i-blog.csdnimg.cn/blog_migrate/48027aa99f2b2e72b87cb65a10e5c34c.png)

也是在构造器调用了load方法(注解是扫描方法)和refresh方法

![image-20200919184640123](https://i-blog.csdnimg.cn/blog_migrate/566d2c6f6df07d3dd6775d1d03df93d9.png)

load方法主要是调用了xml解析器的load方法。

![image-20200919184716783](https://i-blog.csdnimg.cn/blog_migrate/2502cb0f8d363072a04e82c4de41aa16.png)

### GenericReactiveWebApplicationContext

通用活性网络应用上下文。

`GenericReactiveWebApplicationContext`继承了`GenericApplicationContext`实现了`ConfigurableReactiveWebApplicationContext`接口。

前面就看到过，reactive相关的都是加个壳，实际上没有新的操作。

所以，`GenericReactiveWebAppliiicationContext`主要是就是实现接口的方法。

而接口也是一个Reactive的方法也就是说，实际上实现还是接口的父类的方法。也就是接口`ConfigurableApplicationContext`的方法。

而`ConfiguracbleApplicationContext`中的方法，在`GenericApplicationContext`中也有实现，所以`GenericReactiveWebApplicationContext`实际上什么都没有增加。

![image-20200919190947788](https://i-blog.csdnimg.cn/blog_migrate/ab4a6f7ad3fd24808a0396dc42fb8cb3.png)

唯一的操作就是将资源重新做了包装。

#### ReactiveWebServerApplicationContext

活性网络服务应用上下文。

`ReactiveWebServerApplicationContext`继承了`GenericReactiveWebApplicationContext`实现了`ConfigurableWebServerApplicationContext`接口。

和通用活性网络应用上下文一样，活动网络服务应用上下文多了服务相关的操作，其余的都在通用应用上下文中实现了。

所以活性网络服务应用上下文中有两个属性：网络服务管理和服务命名空间(都是和服务相关的)

![image-20200919191354232](https://i-blog.csdnimg.cn/blog_migrate/88a0505cf2ec4aa38ec9db089044213c.png)

活性网络服务应用上下文重写了refresh方法，也是将父类用try-catch包围，如果出现异常，关闭网络服务。

![image-20200919192929560](https://i-blog.csdnimg.cn/blog_migrate/32632b864415093bc4075ac193eb951c.png)

活性网络服务应用上下文重写onRefresh方法，父类的自定义操作完成后，将会创建网络服务。

![image-20200919193021432](https://i-blog.csdnimg.cn/blog_migrate/c67cc50abd9fa1e8f40bd262d2893b13.png)

创建网络服务会判断，如果网络服务管理器是空的，就会重新创建网络管理器，否则直接初始化属性就完了。

创建网络服务管理器，首先会获取网络服务容器名称，然后从容器中获取网络服务的ring器，然后创建网络服务管理器，将网络服务传入。

创建网络服务管理器，同时会创建网络服务正常关闭生命周期和网络服务启动停止生命周期。

![image-20200919193736814](https://i-blog.csdnimg.cn/blog_migrate/6148dc887333d1cdf2c97a3e67dbe450.png)

获取网络服务容器名称，主要是从容器中获取网络服务容器类型的全部beanName。

![image-20200919193843076](https://i-blog.csdnimg.cn/blog_migrate/7761e228525c9b3aa7d76afe9502c9f2.png)

而且，网络服务容器只能注册一个。多余1个，少于1个都会异常。

当然http处理器也是相同

![image-20200919193945355](https://i-blog.csdnimg.cn/blog_migrate/5941a4ee9ef6ea32ae5a65b5394e0308.png)

活性网络服务应用上下文重写了关闭方法

![image-20200919194029496](https://i-blog.csdnimg.cn/blog_migrate/05f65913d6f9b6ccc32a460edca9e536.png)

主要是发布当前网络服务不在接受流量的事件。

##### WebServerManager

网络服务管理器。

网络服务管理器主要是处理网络服务相关的。

所以网络服务管理器的属性包括：网络服务，应用上下文和延时初始化的http处理器。

![image-20200919191549141](https://i-blog.csdnimg.cn/blog_migrate/47103edf35f11c66c3da494a214142b1.png)

WebServerManager主要也是实现了启动、停止等方法。

在启动方法中初始化http处理器，启动网络服务，以及发布网络服务初始化的事件。

![image-20200919191730160](https://i-blog.csdnimg.cn/blog_migrate/a9dd3c353b39214b990045c9430f7d3d.png)

在内部实现了延时初始化的http处理器

![image-20200919191801218](https://i-blog.csdnimg.cn/blog_migrate/18a114c0320457f2a66c34e6ef66107b.png)

延时处理器继承了HttpHandler接口，使得延时处理器可以处理请求。

延时处理器不进行初始化会抛出异常。

还有一个懒处理器。和延时处理器差不多。

![image-20200919192812868](https://i-blog.csdnimg.cn/blog_migrate/a470fa60b17f7374276734477f4f3d15.png)

##### HttpHandler

http处理器接口。

主要定义了处理ServletHttpRequest和ServletHttpResponse的处理。

![image-20200919191838574](https://i-blog.csdnimg.cn/blog_migrate/fde8e2b02bda58a41a08fdc64d095f51.png)

##### ServletHttpRequest

控制器http请求。

`ServletHttpRequest`继承了`HttpRequest`接口。

接口定义了请求相关的操作：

  1. 获取id
  2. 获取请求地址
  3. 获取请求参数
  4. 获取Cookies
  5. 获取本地地址
  6. 获取远程地址
  7. 获取ssl信息
  8. ServletHttpRequest的构造器

![image-20200919192546021](https://i-blog.csdnimg.cn/blog_migrate/97b66d018f627a94efa288760462abdb.png)

##### HttpRequest

http请求接口。

`httpRequest`继承了`HttpMessage`接口。

定义了获取请求的方法和获取请求的uri方法。

![image-20200919192242040](https://i-blog.csdnimg.cn/blog_migrate/c92f654abeb41c88e55e5883ac09bf0d.png)

##### HttpMessage

http消息接口。

定义了获取请求头的方法。

![image-20200919192134059](https://i-blog.csdnimg.cn/blog_migrate/092a850bc1299faa593580540d131433.png)

##### HttpMethod

Http方法枚举。

定义了网络请求的方式(get,post,put,delete…)

![image-20200919192509340](https://i-blog.csdnimg.cn/blog_migrate/3848393fb93a9e1d1649f3d52588750b.png)

##### AnnotationConfigReactiveWebServerApplicationContext

注解配置活性网络服务应用上下文。

`AnnotationConfigReactiveWebServerApplicationContext`继承了`ReactiveWebServerApplicationContext`实现了`AnnotationConfigRegistry`接口。

注解配置活性网络服务应用上下文继承了活性网络服务应用上下文，需要实现注解配置接口。

同样的，注解配置活性网络服务应用上下文主要是实现注解配置。

所以该上下文的属性主要都是注解配置相关的

![image-20200919194421989](https://i-blog.csdnimg.cn/blog_migrate/6f96321794070c3fce2b395ebc335815.png)

注解配置活性网络服务应用上下文的构造方法主要是注册或者扫描相关的注解，或者包，然后执行refresh方法。

![image-20200919194519979](https://i-blog.csdnimg.cn/blog_migrate/c0ccfccf3f0738b4503d21eef2cbd9ed.png)

其他核心的操作，比如上下文刷新前准备和beanFactory后置处理

![image-20200919194620977](https://i-blog.csdnimg.cn/blog_migrate/ca2ade2c0c101bc5d2b73a2462c9f2d3.png)

### AbstractRefreshableWebApplicationContext

抽象可刷新网络应用上下文。

`AbstractRefreshableWebApplicationContext`继承了`AbstractRefreshableConfigApplicationContext`实现了`ConfigurableWebApplicationContext`和`ThemeSource`接口。

抽象可刷新网络应用上下文继承了抽象可刷新可配置应用上下文，实现了可配置网络应用上下文接口和主题接口。

因为需要实现可配置网络应用上下文接口和主题接口，所以，抽象可刷新网络应用上下文中的属性主要是网络相关的属性：

比如控制器上下文，控制器配置，命名空间和主题服务。

![image-20200919195233394](https://i-blog.csdnimg.cn/blog_migrate/380989d75201d48004555d3661cb9750.png)

抽象可刷新网络应用上下文重写了beanFactory后置处理器。主要是忽略控制器配置相关的类。

![image-20200919195345598](https://i-blog.csdnimg.cn/blog_migrate/0400c4b0b6bd29b5e8dedfece86e2d85.png)

抽象可刷新网络应用上下文重写了onRefresh方法，主要是初始化主题服务。

![image-20200919195441123](https://i-blog.csdnimg.cn/blog_migrate/d4ceea3277fdbd441be9ac75b1463a4a.png)

#### GroovyWebApplicationContext

Groovy网络应用上下文。

`GroovyWebApplicationContext`继承了`AbstractRefreshableWebApplicationContext`实现了`GroovyObject`接口。

所以其属性也是兼容Groovy的一些对象

![image-20200919195656760](https://i-blog.csdnimg.cn/blog_migrate/81fe32ae7df3bb76f6c4b8e3469b8ad2.png)

包括bean包装器和类元数据。

Groovy网络应用上下文实现了在`AbstractRefreshableApplicationContext`中定义的抽象方法`loadBeanDefinitions`

![image-20200919195850951](https://i-blog.csdnimg.cn/blog_migrate/27155e09eb3e142ed392eb2e99f99acf.png)

主要包含了创建了信息读取器和初始化读取器，加载了信息。

初始化类信息读取器是一个空操作。

![image-20200919195952796](https://i-blog.csdnimg.cn/blog_migrate/58c88ff1a310df742048a426588cb59c.png)

加载类信息主要是使用Groovy的类信息读取器加载。

![image-20200919200022915](https://i-blog.csdnimg.cn/blog_migrate/282e34a7167f202b1af36e266932b473.png)

#### XmlWebApplicationContext

xml网络应用上下文。

`XmlWebApplicationContext`继承了`AbstractRefreshableWebApplicationContext`。

和Groovy网络应用上下文类似，实现了`AbstractRefreshableApplicationContext`中定义的抽象方法`loadBeanDefinitions`方法

![image-20200919200251713](https://i-blog.csdnimg.cn/blog_migrate/36bd68f31753899605eb97e8f01598c3.png)

步骤也都差不多，首先创建一个xml读取器，然后初始化读取器，最终调用读取器加载类信息。

初始化xml读取器也是空操作。

![image-20200919200347806](https://i-blog.csdnimg.cn/blog_migrate/a19a6575c024c5043bc0f8b71cd3eb4b.png)

调用xml读取器加载类信息

![image-20200919200405102](https://i-blog.csdnimg.cn/blog_migrate/1d1376ed7f78aabc32de7c6721fff9fe.png)

#### AnnotationConfigWebApplicationContext

注解配置网络应用上下文。

`AnnotationConfigWebApplicationContext`继承了`AbstractRefreshableWebApplicationContext`实现了`AnnotationConfigRegistry`接口。

注解配置网络应用上下文主要是实现`AbstractRefreshableApplicationContext`中的`loadBeanDefinitions`方法。

以及`AnnotationConfigRegistry`接口的方法。

所以注解配置网络应用上下文中的属性都是和注解配置相关的。

属性包含beanName生成器和作用域解析器。

![image-20200919200713339](https://i-blog.csdnimg.cn/blog_migrate/b5c47889682479588486919ba4657a20.png)

实现的`loadBeanDefinitions`方法，首先获取了扫描器和注册器，然后将beanName生成器初始化给扫描器和注册器，然后将作用域解析器也初始化给扫描器和注册器。

![image-20200919200817455](https://i-blog.csdnimg.cn/blog_migrate/89b316c0df4d6836cc96132a4b55ae2b.png)

如果是class表示注册，调用注册，如果是包路径表示扫描，调用扫描

![image-20200919201031133](https://i-blog.csdnimg.cn/blog_migrate/6c2b4fbfa2de34ba252460dfacfa49fe.png)

如果有其他本地配置(编码等)，那么就将本地配置也注册给读取器。

![image-20200919201402619](https://i-blog.csdnimg.cn/blog_migrate/ec58443f5a30e0b011b25ec3784f51f1.png)

### AbstractXmlApplicationContext

抽象xml应用上下文。

`AbstractXmlApplication`继承了`AbstratRefreshableConfigApplicationContext`。

所以抽象xml应用上下文需要实现`AbstractRefreshableConfigApplicationConext`中定义的抽象方法`loadBeanDefinitions`。

![image-20200919201927337](https://i-blog.csdnimg.cn/blog_migrate/abb796fddd4f44bcdb7e0af86305ff13.png)

抽象方法中，首先创建xml类信息读取器，然后初始化类信息读取器，最终调用读取器加载类信息。

初始化读取器中，根据是否需要验证xml进行设置读取器的属性

![image-20200919202105425](https://i-blog.csdnimg.cn/blog_migrate/324e3adf9b8f75bfc64dbe35fc46c608.png)

接着根据配置读取(其实到目前为止，都没有地方设置配置，这个配置需要用户手动设置)

![image-20200919202140118](https://i-blog.csdnimg.cn/blog_migrate/abdc4b4793bdbaf23a3c87a3f43670ec.png)

#### ClassPathXmlApplicationContext

类路径下xml应用上下文。

`ClassPathXmlApplciationContext`继承了`AbstractXmlApplicationContext`。

在`AbstractXmlApplicationContext`中就分析出，`AbstractXmlApplicationContext`的子类只需要设置配置即可。

所以类路径下xml应用上下文主要就是设置配置。

其唯一的属性也是本地配置的资源。

![image-20200919202532680](https://i-blog.csdnimg.cn/blog_migrate/ffcf674532c633e1ed7f42e469031326.png)

在构造方法中，用户将配置传入，然后就设置配置就好了

![image-20200919202608422](https://i-blog.csdnimg.cn/blog_migrate/98f6d676b0583cffda7b9debd4b85573.png)

如果有需要，那么就调用refresh方法了。

或者给出一个路径，需要解析完之后调用refresh方法。

![image-20200919202710596](https://i-blog.csdnimg.cn/blog_migrate/cac80976ac1f9991fe178263baa37d66.png)

#### FileSystemXmlApplicationContext

文件系统xml应用上下文。

`FileSystemXmlApplicationContext`继承了`AbstractXmlApplicationContext`。

所以，`FileSystemXmlApplicationContext`也需要设置配置就可以了。

直接在构造方法中设置就没问题了

![image-20200919202854400](https://i-blog.csdnimg.cn/blog_migrate/2194b44068c9d4e51c4e684cf71407e3.png)

如果需要调用refresh，那么就调用refresh方法。

一般情况下还是需要调用refresh方法的。除非是抽象类，抽象类不会调用refresh方法，而是由抽象类进行传递这个方法的调用，实现调用链的完整。

所以，这里可能是预留的一个扩展吧。目前找到的全部的调用还都是需要调用refresh的。

## 常用的注解

常用的注解有`@Service`,`@Controller`,`@Repository`等

这些注解都用`@Component`修饰，是`@Component`注解的别名，所以这些也属于`@Component`解析的范围之内。

![image-20200919204217612](https://i-blog.csdnimg.cn/blog_migrate/c5c9022e2d206c6c6d7ca56a579459b3.png)

![image-20200919204209510](https://i-blog.csdnimg.cn/blog_migrate/5de5e85ef2f60257e3125324944fd1f7.png)

![image-20200919204202869](https://i-blog.csdnimg.cn/blog_migrate/4110db07b41f3535acf4c7eaf21d6d67.png)

包括一些常用的，都是`@Component`的别名。

![image-20200919204328528](https://i-blog.csdnimg.cn/blog_migrate/141140aadeeeda887a144f269a00e5e4.png)

数不胜数，所以在ApplicationConext中处理`@Component`注解就够了。
