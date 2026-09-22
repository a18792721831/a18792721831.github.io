---
layout: post
title: "Spring源码--BeanFactory"
date: 2020-08-22 19:03:24 +0800
categories: [Spring源码解析, BeanFactory源码解析, SAX解析XML文件, profile机制实现原理, XML验证模式DTD, XSD]
description: "本文深入探讨了Spring框架的核心组件BeanFactory的源码，解析了其接口定义与实现细节，包括getBean方法的多种重载形式，以及更高级的ApplicationContext接口。通过分析XmlBeanFactory的UML图和源码，介绍了BeanDefinition的读取与注册过程，揭示了配置文件的封装机制和XML验证模式（DTD与XSD）的应用。"
keywords: Spring源码解析, BeanFactory源码解析, SAX解析XML文件, profile机制实现原理, XML验证模式DTD, XSD
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/108172805
> - 发布时间：2020-08-22 19:03:24
> - 阅读量：441
> - 分类：spring专栏收录该内容, 订阅专栏
> - 标签：#Spring源码解析, #BeanFactory源码解析, #SAX解析XML文件, #profile机制实现原理, #XML验证模式DTD, #XSD

## 摘要

文章浏览阅读441次。本文深入探讨了Spring框架的核心组件BeanFactory的源码，解析了其接口定义与实现细节，包括getBean方法的多种重载形式，以及更高级的ApplicationContext接口。通过分析XmlBeanFactory的UML图和源码，介绍了BeanDefinition的读取与注册过程，揭示了配置文件的封装机制和XML验证模式（DTD与XSD）的应用。

---

#### Spring源码--BeanFactory

  * BeanFactory
  * 容器的基本使用
  * XmlBeanFactory源码
  * XmlBeanDefinitionReader
  * XmlBeanFactory
  *     * 配置文件的封装
    * 加载Bean
    * XML的验证模式
    *       * DTD
      * XSD
    * 获取Document
    * BeanDefinitions

  
github地址：   
https://github.com/a18792721831/studySpringSource.git 

## BeanFactory

`Spring Ioc` 是一个管理Bean的容器，在Spring的定义中，他要求所有的Ioc容器都需要实现接口`BeanFactory`。`BeanFactory`是一个顶级容器接口。
    
    
    public interface BeanFactory {
    
        // 前缀
    	String FACTORY_BEAN_PREFIX = "&";
    
        // 根据名称获取bean
    	Object getBean(String name) throws BeansException;
    
        // 根据名称获取bean，返回指定类型
    	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
    
        // 根据名称获取bean，使用指定的参数初始化
    	Object getBean(String name, Object... args) throws BeansException;
    
        // 根据类型获取bean
    	<T> T getBean(Class<T> requiredType) throws BeansException;
    
        // 根据类型获取bean，使用指定的参数初始化
    	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
    
        // 根据类型获取bean提供者
    	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
    
    	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
    
        // 是否存在指定名称的bean
    	boolean containsBean(String name);
    
        // 指定的bean是否是单例
    	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
    
        // 指定的bean是否是原型
    	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
    
        // 是否类型匹配
    	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
    
    	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
    
        // 获取指定bean的类型
    	@Nullable
    	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
    
    	@Nullable
    	Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;
    
        // 获取bean的别名
    	String[] getAliases(String name);
    
    }
    

在`BeanFactory`中含有多个getBean方法，这是Ioc容器最重要的方法之一，主要是从Ioc容器中获取Bean。这些getBean的方法中有按类型、按名称获取bean，这也是说，在Spring Ioc容器中，允许按照名称或者类型获取bean.

由于`BeanFacttory`的功能还不够强大，因此Spring在`BeanFactory`的基础上，还设计了更为高级的接口：`ApplicationContext`，这是`BeanFactory`的子接口之一,在Spring的体系中`BeanFactory`和`ApplicationContext`是最为重要的接口设计，实际使用的大部分Spring Ioc容器是`ApplicationContext`接口的实现类。

![ApplicationContext](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e9192a3e0c79422c2d14da6034a6f06f.png)

`ApplicationContext`接口通过集成上级接口，进而集成`BeanFactory`接口。在`BeanFactory`的基础上，扩展了消息国际化接口(MessageSource)和资源模式解析接口(ResourcePatternResolver)。

## 容器的基本使用

我们在idea中创建一个springboot的工程，只需要lombok和web的starter即可。(gradle)

![image-20200820193932622](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1dc534ea00f556eabb7f1dcd84029340.png)

我们这里创建了spring boot的工程，但是只是使用gradle自动下载spring的依赖即可。

还记得一个spring项目是如何创建的吗？

复习一下：

[spring–hello~!(如何搭建一个spring项目)](<https://blog.csdn.net/a18792721831/article/details/87110126>)

[spring核心容器创建的两种方式](<https://blog.csdn.net/a18792721831/article/details/87111551>)

省去依赖包，我们创建一个spring的项目

![image-20200820194405030](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ae1e66ad780ddd7ed5ed6500d51b7039.png)

新增bean

![image-20200820194429874](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3a64b64f62756f485240efbb4734bb29.png)

新增beans.xml

![image-20200820194453702](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/91de51cf41b36b3d67a8f7c06740973a.png)

然后注释掉spring boot中原来的代码，添加我们的代码

![image-20200820194528160](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f80c241db46816ceddf75a9e1bcb112c.png)

运行

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4e5a02ad5685afd631a60bfb5b9d5461.gif)

可以看到，最终，我们打印出了people的name.也就是说，通过`XmlBeanFactory`加载了我们制定的配置文件后，使用指定的配置文件，创建了我们需要的Ioc容器。最后我们可以通过名字获取bean。

## XmlBeanFactory源码

虽然`XmlBeanFactory`基本上没有人在使用，而且也被标记为废弃了，但是，作为我spring入门的一个方法，我还是想看看其具体的实现。

这是`XmlBeanFactory`的UML图：

![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b7040a0aa6dc958cf90c93f82afe4a8c.png)

我们可以看到，`XmlBeanFactory`直接继承`DefaultListableBeanFactory`,而`DefaultListableBeanFactory`上面又有非常多的继承关系。

  * `AliasRegistry`:定义对alias的简单增删改等操作。
  * `SimpleAliasRegistry`:主要使用Map作为alias的缓存，并对接口`AliasRegistry`进行实现。
  * `SingletonBeanRegistry`:定义对单例的注册及获取。
  * `BeanFactry`:定义获取bean及bean的各种特性。
  * `DefaultSingletonBeanRegistry`:对接口`SingletonBeanRegistry`的实现。
  * `HierarchicalBeanFactory`:继承`BeanFactory`，也就是在`BeanFactory`定义的功能的基础上增加了对`parentFactory`的支持。
  * `BeanDefinitionRegistry`:定义对`BeanDefinition`的各种增删改操作。
  * `FactoryBeanRegistrySupport`:在`DefaultSingletonBeanRegistry`基础上增加了对`BeanFactory`的特殊处理。
  * `ConfigurableBeanFactory`:提供配置Factory的各种方法。
  * `ListableBeanFactory`:根据各种条件获取bean的配置清单。
  * `AbstractBeanFactory`:综合`FactoryBeanRegistrySupport`和`ConfigurableBeanFactory`的功能。
  * `AutowireCapableBeanFactory`:提供创建bean、自动注入、初始化以及应用bean的后处理器。
  * `AbstractAutowireCapableBeanFactory`A:综合`AbstractBeanFactory`并对接口`AutowireCapableBeanFactory`进行实现。
  * `ConfigurableListableBeanFactory`:`BeanFactory`配置清单，指定忽略类型及接口等。
  * `DefaultListableBeanFactory`:综合上述全部功能，主要是对Bean进行注册后的处理。

`XmlBeanFactory`对`DefaultListableBeanFactory`进行了扩展，从Xml文档中读取`BeanDefinition`，对于注册和获取都是使用的父类继承的方法。

![image-20200820202243293](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/60a2baf00db79ed8d2c406be5032353c.png)

## XmlBeanDefinitionReader

这是`XmlBeanDefinitionReader`的UML图

![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9dfeb9a9a336352cbb3c34e3a0f5bb8f.png)

这些类主要的操作：

  * `ResourceLoader`:定义资源加载器，主要应用于根据给定的资源文件地址返回对应的Resource。
  * `BeanDefinitionReader`:主要定义自语言文件读取并转换为`BeanDefinition`的各个功能。
  * `EnvironmentCapable`:定义获取Environment方法。
  * `DockumentLoader`:定义从资源文件加载到转换为Document的功能。
  * `AbstractBeanDefinitionReader`:对`EnvironmentCapable,BeanDefinitionReader`类定义的功能进行实现。
  * `BeanDefinitionDocumentReader`定义读取Document并注册BeanDefinition功能。
  * `BeanDefinitionParserDelegate`:定义解析Element的各种方法。

## XmlBeanFactory

我们看下`XmlBeanFactory`的时序图：

![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/df0aea21c34600e8c73dcc0063d48645.png)

在main方法中首先调用`ClassPathResource`的构造方法来构造Resource资源文件的实例对象，这样后续的资源处理就可以用Resource提供的服务进行操作，有了Resource后，就可以进行`XmlBeanFactory`的初始化了。

### 配置文件的封装

Spring的配置文件读取是通过`ClassPathResource`进行封装的，比如`new ClassPathResource("beans.xml")`。

在java中，将不同来源的资源抽象成URL，通过注册不同的`handler(URLStreamHandler)`来处理不同来源的资源的读取逻辑，一般handler的类型使用不同的前缀来识别，比如:`file:,http:,jar:`等。但是URL没有默认定义相对ClassPath或者ServletContext等资源的handler，虽然可以注册自己的URLStreamHandler来解析特定的URL前缀(比如c`classpath:`),这样需要了解URL的实现机制，而且URL也没有提供基本方法检查资源是否存在等。所以Spring对内部使用到的资源资源实现了自己的抽象结构：`Resource`

这是Resource的方法

![image-20200822144138299](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4890fa5a55fd9cf9795fd1a6083effcc.png)

Resource接口抽象了所有Spring内部使用到的底层资源:File,URL,Classpath等。

定义了3个判断当前资源状态的方法：存在性(`exists`)、可读性(`isReadable`)、是否处于打开状态(`isOpen`）。另外，Resource接口还提供了不同资源到URL、URI、File类型的转换，以及获取`lastModified`属性、文件名(不带路径信息的文件名，`getFilename()`)的方法。为了便于操作，Resource 还提供了基于当前资源创建一个相对资源的方法：`createRelativeO`。在错误处理中需要详细地打印出错的资源文件，因而 `Resource`还提供了 `getDescription`()方法用于在错误处理中的打印信息。

![image-20200822143834013](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c71b7a8b29f0debc91b8e76c516e9d2d.png)

不同来源的资源文件都有相应的Resource实现：文件(`FileSystemResource`),ClassPath资源(`ClassPathResource`)，URL资源（`UrlResource`），InputStream资源(`InputStreamResource`),Byte数组(`ByteArrayResource`)等。

具体实现也很简单，ClassPathResource是通过ClassLoader进行读取文件的：

![image-20200822145454635](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/de2c8329fc75ab59f9578985ccdaa07c.png)

得到了配置流后，就交给了`XmlBeanDefinitionReader`进行解析。

![image-20200822145716503](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f94bfca2538cd99f12d99c377294d99b.png)

![image-20200822145729279](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1606b3b93e0a48a8ccf45365cf0b0c65.png)

在初始化父类的时候，忽略装配一些类

![image-20200822145953961](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/21c9e1d87fd56974ac359ad8947634c1.png)

### 加载Bean

![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bc2ef0b85999b0b0ec020d43ae3d7577.png)

  1. 封装资源文件。当进入`XmlBeanDefinitionReader`后首先对参数Resource使用`EncodeResource`类进行封装。
  2. 获取输入流。从Resource中获取对应的`InputSttream`并构造`InputSource`。
  3. 通过构造的`InputSource`实例和Resource实例继续调用方法`doLoadBeanDefinitions`。

![image-20200822152733034](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/87d9657835eb3c99225170adefd67ed9.png)

`EncodeedResource`主要用于对资源文件的编码进行相应的编码处理。

![image-20200822154027396](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e433597edf24dd33f0d7ae415b7c1b3c.png)

如果设置了编码属性，会使用相应的编码作为输入流的编码。

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a2dc43cf55365ebc37d62bae46b28624.png)

这个方法主要是将重新编码的输入流转换为SAX的InputSource对象，同时如果指定了编码，需要设置相关的属性。

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/45b955f2539cfbf253b89a080b0b99c7.png)

这个方法处理了两件事情：

  * 加载XML文件，得到对应的Document.
  * 根据返回的Document注册bean

### XML的验证模式

XML文件的验证模式保证了XML文件的正确性，而比较常用的验证模式有两种：DTD和XSD。

![image-20200822181516391](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1186d27f1fd7988026835b6093516de9.png)

![image-20200822181600969](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/351f1e7bd94e7ba802f457583e964112.png)

#### DTD

DTD(`Document Type Dedfinition`)即文档类型定义，是一种XML约束模式语言，是XML文件的验证机制，属于XML文件组成的一部分。DTD是一种保证XML文档格式正确的有效方法，可以通过比较XML文档和DTD文件来看文档是否符合规范，元素和标签使用是否正确。一个DTD文档包含:元素的定义规则，元素间关系的定义规则，元素可以使用的属性，可以使用的实体或符号规则。

要使用DTD验证模式的时候需要在XML文件的头部声明：

![image-20200822172452919](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b9f3433e10c1896892c26447b435235e.png)

DTD文件里面是一些ENTITY节点

![image-20200822172528742](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/18dae4275a96f3e98d9c5b33566d050e.png)

#### XSD

XML Schema语言就是XSD(`XML Schema Definition`)。`Xml Schema`描述了XML文档的结构。可以用一个指定的`XML Schema`来验证某个XML文档，已检查改XML文档是否符合其要求。文档设计者可以通过过`XML Schema`指定一个XML文档所允许的结构和内容，并可以据此检查一个XML文档是否是以有效的。`XML Schema`本身是一个XML文档，他符合XML语法结构，可以使用通用的XML解析器解析它。

![image-20200822181110255](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/03471343fb0c98bd010fd76f3771b410.png)

在使用`XML Schema`文档对XML实例文档进行检验，除了要声明空间外(`xmlns=....`)，还必须指定该名称空间锁对应的`XML Schema`文档的存储位置。通过`schemaLocation`属性来指定名称空间所对一样的`XML Schema`文档的存储位置。它包含两个部分，一部分是名称空间的URL，另一部分就是该名称空间所表示的`XML Schema`文件位置或者URL地址。

### 获取Document

![image-20200822181621954](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c4ed5df0b6782a391efa109cc2ec7821.png)

调用了`DocumentLoader`接口的方法：

![image-20200822181704320](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a6d66d6eaa7c647cf7258352a4222321.png)

这个接口只有一个实现类`DefaultDocumentLoader`

![image-20200822182350576](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/da6615ac08e9c73f7d2321779a1ecbdf.png)

### BeanDefinitions

将配置文件转换为`Document`后，就会根据`Document`对象进行注册Bean.

![image-20200822182535309](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/afeebca0708880428d1f18d636826439.png)

注册bean：

![image-20200822183351796](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/46742fcdd26d62db1ff2bf4efde75f6b.png)

首先通过反射获取`BeanDefinitionDocumentReader`的对象。

![image-20200822183516072](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/16e7d3c71dd96978c2ec8a2e0ec1aef4.png)

然后记录下本次注册前，已经有多少个bean被注册了。

然后调用`documentReader`对象进行注册。

![image-20200822183726851](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ab095445ba4a1e91a13bc57519e01298.png)

使用的还是默认实现

![image-20200822183745063](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/524edf0cc2ba4cc1f25daf518e100960.png)

在使用documentReader进行读取时，首先读取的是root节点。

![image-20200822183936937](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ec4b16ac2f5d74f5050b36e3d84d9055.png)

接下来就是解析的核心逻辑了：

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b77ebe29c696c3e829bf3e479bcf1000.png)

首先处理profile

![image-20200822184440549](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a44e019790cf8bfd1d947aaa13b870dd.png)

解析bean就是这里了

![image-20200822184809167](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b259a7d8d80f3ec02fd2ddeea2729e94.png)

![image-20200822185320840](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/64423447940b3f6ba7999a74705673e1.png)
