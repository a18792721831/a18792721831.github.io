---
layout: post
title: "bean、容器、Ioc和DI"
date: 2020-09-12 17:21:55 +0800
categories: [springBean, spring容器, springDI, springIoc, spring源码]
description: "深入剖析Spring框架的核心组件，包括Bean、容器、IoC和DI的概念，以及它们在Spring中的实现细节，如BeanFactory、ApplicationContext、AutowireCapableBeanFactory等关键接口与类的职责与工作流程。"
keywords: springBean, spring容器, springDI, springIoc, spring源码
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/108551762
> - 发布时间：2020-09-12 17:21:55
> - 阅读量：418
> - 分类：spring专栏收录该内容, 订阅专栏
> - 标签：#springBean, #spring容器, #springDI, #springIoc, #spring源码

## 摘要

文章浏览阅读418次。深入剖析Spring框架的核心组件，包括Bean、容器、IoC和DI的概念，以及它们在Spring中的实现细节，如BeanFactory、ApplicationContext、AutowireCapableBeanFactory等关键接口与类的职责与工作流程。

---

#### bean、容器、Ioc和DI

  * bean、容器、Ioc和DI
  * bean
  *     * bean简介
    * bean 创建
    *       * FactoryBean
      * BeanDefinition
      * BeanMetadataElement
      * AttributeAccessor
  * 容器
  *     * Java中的容器
    * Spring中的容器
    * 接口
    *       * BeanFactory
      * HierarchicalBeanFactory
      * ListableBeanFactory
      * SingletonBeanRegistry
      * ConfigurableBeanFactory
      * AutowireCapableBeanFactory
      * ConfigurableListableBeanFactory
      * AliasRegistry
      * BeanDefinitionRegistry
    * 实现
    *       * SimpleAliasRegistory
      * DefaultSingletonBeanRegistory
      *         * 注册--SingletonBeanRegistry
        * 获取--SingletonBeanRegistry
        * 是否包含--SingletonBeanRegistry
        * 获取全部的beanName--SingletonBeanRegistry
        * 获取单例容器中已经注册的bean的数量--SingletonBeanRegistry
        * 获取单例互斥对象--SingletonBeanRegistry
        * 使用给定的beanFactory创建beanInstance
        * 移除
        * 设置正在创建的bean
        * 注册有依赖的bean
        * 判断两个bean之间是否存在依赖
        * 获取依赖的beanName
        * 单例容器销毁
        * DefaultSingletonBeanRegistory多线程处理
      * FactoryBeanRegistorySupport
      *         * 关系
        * 获取beanInstance的类型
        * 获取正在创建的beanInstance的映射
        * 处理指定的FactoryBean
        * doGetObjectFromFactoryBean
        * beforeSingletonCreation
        * postProcessObjectFromFactoryBean
        * afterSingletonCreation
        * 留给子类实现的扩展
      * AbstractBeanFactory
      *         * 关系
        * 多层容器
        * 类加载
        * 依赖处理
        * 属性配置
        * 类型转换
        * 属性解析
        * bean处理器
        * 作用域
        * 类信息
        * 留给子类扩展
      * AbstractAutowireCapableBeanFactory
      *         * 关系
        * 创建Bean实例
        * bean前置处理
        * bean处理器factory后置处理器
        * bean后置处理器
        * 自动装配
        * 配置bean
        * 销毁bean
        * bean创建时依赖忽略
        * 子类扩展
      * DefaultListableBeanFactory
      *         * 关系
        * 抽象方法实现
        * 依赖处理
        * bean信息处理
        * 序列化和反序列化
        * 查漏补缺
        * 使用
      * XmlBeanFactory
      *         * 关系
        * XmlBeanFactory的操作
        * XmlBeanDefinitionReader#loadBeanDefinitions
        * registryBeanDefinition
  * 总结
  *     * bean
    * 容器
    * Ioc
    * DI

  
github地址: 

https://github.com/a18792721831/studySpringSource.git

## bean、容器、Ioc和DI

刚开始学习spring源码，总是对Ioc，容器，bean这些名词，都知道是啥玩意，其概念，特点等都知道一些。别人问起来，也大概能说个一二三 ，但是，Ioc，容器，bean到底是什么，就有些模糊了。

目前处于知其然而不知其所以然，所以，什么是Ioc，什么是容器，什么是bean？

今天就结合这几天对这些知识的了解，做个简单的总结。(能力有限，多多包涵)

最后分析一个经典的spring容器`XmlBeanFactory`

## bean

[JavaBean的创建和使用](<https://blog.csdn.net/a18792721831/article/details/76541292>)

### bean简介

![这里写图片描述](https://i-blog.csdnimg.cn/blog_migrate/91f2caf89bacbabbc6000b9f51ff34c1.jpeg)

### bean 创建

![这里写图片描述](https://i-blog.csdnimg.cn/blog_migrate/a256d96a2559b30a3754beb5ebc76fcf.jpeg)

我认为这个对于javaBean的定义还是比较准确的，理解上也是比较简单的。

bean就是Java实体类。

#### FactoryBean

在spring中又有一个FactoryBean的接口，这个接口定义了springBean应该是什么样的：

![image-20200903184606285](https://i-blog.csdnimg.cn/blog_migrate/97a7d633af982ba8fd0bb68376be4986.png)

可以看到，springBean的定义非常简单，只需要实现两个方法就行，分别是getObject和getObjectType即可。

定义了spring容器中存储的对象。

| 接口                  | 说明                |
|---------------------|-------------------|
| FactoryBean         | springBean的最基本的定义 |
| BeanDefinition      | springBean的详细的定义  |
| BeanMetadataElement | springBean的元数据的定义 |
| AtrributeAccessor   | springBean的元数据的操作 |

#### BeanDefinition

![image-20200903185703182](https://i-blog.csdnimg.cn/blog_migrate/164a06b91ce9d6d1383bc7e5165df0fd.png)

BeanDefinition才是springBean中常用的描述对象，因为FactoryBean中能够存储的信息实在是太少了，所以BeanDefinition是对FactoryBean的一个扩展。

不过FactoryBean和BeanDefinition之间没有什么关系。

BeanDefinition中记录了springBean的作用域

![image-20200903190026298](https://i-blog.csdnimg.cn/blog_migrate/c60c1489c3ca70f39b3123f6df57264a.png)

而且还对springBean进行角色区分：

![image-20200903190158811](https://i-blog.csdnimg.cn/blog_migrate/b0e312665d2941266e061e08ac891fc5.png)

0是用户自定义的springBean

1是外部的springBean

2是内部的springBean

这个接口主要定义了这些操作：

  1. 设置beanName
  2. 设置bean的作用域
  3. 设置是否懒加载
  4. 设置依赖
  5. 设置当前bean是否可以作为其他bean自动装配的候选类型
  6. 设置当前bean是否是其他bean的自动装配的主要候选类型
  7. 设置bean的初始化的方法
  8. 设置bean的销毁的方法
  9. 设置bean的角色

描述spring容器中bean的对象。

#### BeanMetadataElement

bean的元数据。

这个接口就非常简单了，获取元数据，默认还返回空。

![image-20200903191334687](https://i-blog.csdnimg.cn/blog_migrate/85a8b133eeceae01c3e1330657bb6274.png)

#### AttributeAccessor

这个接口定义了访问和设置元数据的操作。

![image-20200903191542587](https://i-blog.csdnimg.cn/blog_migrate/f7076aae03c4504b17259954c8530f1a.png)

所以，这个接口主要定义了这些操作：

  1. 设置属性
  2. 移除属性
  3. 是否有属性
  4. 获取全部的属性

这个接口主要定义了springBean元数据的交互。

## 容器

容器是存放bean的一个对象，bean是Java实体类。所以，容器就是存储java实体的对象。

### Java中的容器

暂且不谈spring中高大上的容器。

即使是在java的jdk中，也有一部分容器：

`java.util.List,java.util.Set,java.util.Map`

这三个最常用的容器。当然还有一些扩展的队列、栈等等。这些也是容器，也能存储java实体对象。

### Spring中的容器

spring中的容器实现，有BeanFactory和ApplicationContext这两类。

其中，Applicattion是基于BeanFactory的。

### 接口

BeanFactory是Spring中最底层的容器接口，它定义了作为一个spring容器应该有的方法。

你可以将BeanFactory和java集合里面的Collection想成同样的定义。

在Java集合容器中，Collection定义了Java集合容器的一些基本方法。

| 容器         | Collection             | BeanFactory              |
|------------|------------------------|--------------------------|
| 将bean放入容器  | add,addAll             |                          |
| 容器是否为空     | isEmpty                |                          |
| 从容器清除bean  | clear,remove,removeAll |                          |
| 是否包含       | contains               | containsBean,isTypeMatch |
| 遍历         | iterator               |                          |
| 容器中bean的数量 | size                   |                          |
| 从容器中获取bean | 配合iterator实现           | getBean                  |
| 转为数组       | toArray                |                          |

虽然java的容器和Spring的容器提提供的操作吗，或者说定义容器的方法有所不同，但是其行为相似。

都是存储、管理bean.

或者说，spring中，划分的更加专一，详细了。

![img](https://i-blog.csdnimg.cn/blog_migrate/b7040a0aa6dc958cf90c93f82afe4a8c.png)

这里面的哪一个接口，定义了什么？

我会尝试用一句话概括这里面的每一个接口或者类。

| 接口                              | 说明                                 |
|---------------------------------|------------------------------------|
| BeanFactory                     | spring容器最基础的定义                     |
| HierarchicalBeanFactory         | spring容器本身信息管理                     |
| ListableBeanFactory             | spring容器的遍历                        |
| SingletonBeanRegistory          | 单例springBean的注册                    |
| ConfigurableBeanFactory         | springBean的作用域的定义与管理               |
| AutowireCapableBeanFactory      | springBean自动装配及其模式的定义              |
| ConfigurableListableBeanFactory | springBean自动装配中依赖的处理(主要是冲突依赖和循环依赖) |
| AliasRegistory                  | springBean的别名的注册和管理                |
| BeanDefinitionRegistory         | springBean的详细信息的注册和管理              |

#### BeanFactory

![image-20200901193121803](https://i-blog.csdnimg.cn/blog_migrate/4471b7fa7fa0363c97d496d4b8c8d4c2.png)

  1. 从容器中获取bean
  2. 容器中是否包含指定bean

定义了spring容器最基本的操作。

#### HierarchicalBeanFactory

![image-20200901193355567](https://i-blog.csdnimg.cn/blog_migrate/a646528c00da53582905df947a5679f2.png)

  1. 获取父容器(上层容器)
  2. 是否包含bean

定义了spring容器可以嵌套使用。

#### ListableBeanFactory

在前面中，BeanFactory最核心的方法就是两个，一个是从容器中获取bean，另一个是是否包含bean。

但是这两个方法都有一个相同点，都需要有参数。因为Spring容器中管理和持有的都是单例模式，所以相比于Java容器中的bean，有这样的区别。

Java的容器里面的bean更注重于数据，Spring的容器里面的bean更注重于功能。

我们通常的用法：

有一大堆的数据需要保存，那么，我们创建一个标准的javaBean，然后将数据设置到javaBean里面。然后将标准的javaBean放到Java的容器里面。

当需要这些容器里面的javaBean进行一系列的操作的时候，就会从Spring的容器中得到操作的对象，然后将javaBean传入SpringBean的方法中进行处理。

所以，javaBean和SpringBean的注重点不同。

也就是说，java的容器里面一般是一个类的不同实例，spring的容器里面一般是不同的类的唯一的实例。

因为一般是唯一的，所以，得到了名字，一般就确定了是哪一个类。

这个接口主要定义了这些操作：

  1. 是否包含指定的bean
  2. 根据类的相关信息获取类的名字(bean的名字)

遍历spring容器。

#### SingletonBeanRegistry

看到这里，也许你和我一样有这样的疑惑：

在java的容器里面有两个方法:add,addAll，将javaBean放入java的容器内。

那么，spring容器里面的单例的springBean是如何放入的呢？

java的容器存储的是javaBean，注重的是数据，所以，需要将一个类的一系列对象，交给java容器存储和管理。

spring的容器存储的是springBean，注重的是功能，所以，需要将某个类的唯一对象，交给spring容器管理和存储。

经过上面的分析，我们理解了为什么java容器需要具体的和javaBean对象，然后才能将javaBean对象放入容器。

对于spring容器来说，spring容器只需要知道需要把哪个类放入spring容器即可。

所以，spring容器的springBean的增加，参数是类的相关的信息。

而`SingletonBeanRegistry`就是定义spring容器如何将一个springBean放入的。

![image-20200901195618699](https://i-blog.csdnimg.cn/blog_migrate/58dad7245c01ddbceec818a51176ec22.png)

所以，这个接口定义了这些操作：

  1. bean存入spring容器
  2. 获取bean
  3. 获取全部的beanName
  4. 获取spring容器中bean的数量
  5. 是否包含指定bean

单例bean的容器的操作。

#### ConfigurableBeanFactory

我们前面说到springBean在spring容器中一般是单例的，但是，并不是全部的spring都符合这个规则。

springBean的作用域有单例和原型：

[spring Bean的作用域–单例&原型](<https://blog.csdn.net/a18792721831/article/details/87640850>)

这个类就是定义的，springBean的作用域：

![image-20200902193353838](https://i-blog.csdnimg.cn/blog_migrate/78330194534f7374f5334cb3f6a2c37e.png)

这个接口主要做这几件事：

  1. 设置父容器
  2. 设置类加载器
  3. 设置类型转换器
  4. 设置自定义
  5. 注册bean前后置处理
  6. 注册bean的作用域
  7. 获取依赖的beans
  8. 销毁bean,销毁全部单例bean

定义了springBean的作用域。

#### AutowireCapableBeanFactory

在此之前的spring容器定义的操作，其实更注重的是bean的管理上。

如何存储，反而不是非常的注重。

在此之前，我们交给spring容器一个bean，那么spring容器就使用jdk容器的技术，将springBean管理起来即可。

但是，创建一个bean，也是一件繁琐的事情。

举个例子：

类A依赖类B，类B依赖类C。

现在需要把三个类都放入spring容器中，那么就需要创建三次C类的对象，创建两次B类的对象，创建一次A类的对象。

这样才能把类A,类B，类C都放入容器。

看到问题了吗，如果springBean存在非常多的依赖Bean，那么，创建Bean将是一个非常繁琐且容器出现问题的地方。

基于此，这个接口定义了可以自动装配的spring容器。

![image-20200902195256491](https://i-blog.csdnimg.cn/blog_migrate/458a7e8f0fc654c975bd2b9cc7fa9e25.png)

自动装配有5个模式：

  1. 不自动装配
  2. 根据名字自动装配
  3. 根据类型自动装配
  4. 根据构造器自动装配
  5. 自动检测，多种模式混合

这个类主要的操作：

  1. 创建bean
  2. 自动装配bean
  3. 配置bean
  4. 指定bean执行条件
  5. 初始化bean
  6. 执行前后置bean处理
  7. 销毁bean
  8. 解析bean
  9. 解析bean依赖

自动装配bean.

#### ConfigurableListableBeanFactory

我们在装配bean的时候，会存在循环依赖。

比如A依赖B，B依赖C，C依赖A。

此时在装配A的时候，处理依赖时，应该忽略自己本身。

换句话说，在装配A的时候，发现A依赖B，此时就会优先装配B，在装配B的时候，又发现B依赖C，于是继续优先装配C，在装配C的时候，需要忽略A这个依赖。

否则就进入死循环了。

`ConfigurableListableBeanFactory`中的方法就是定义spring容器在自动装配的时候，忽略哪些类型：

![image-20200903183147413](https://i-blog.csdnimg.cn/blog_migrate/4c72dd5a08798271f08c7b09b2a59dcf.png)

所以，这个类主要的操作：

  1. spring容器自动装配忽略指定类型
  2. spring容器对指定的bean装配指定的依赖
  3. 判断指定的依赖是否是自动装配的候选
  4. 获取springBean的详细定义
  5. 获取spring容器内bean的名字的遍历器
  6. 取消、冻结springBean的元数据
  7. 实例化单例对象

这个接口主要是处理springBean自动装配过程中的依赖问题。

#### AliasRegistry

一般我们可以随意指定springBean一个别名。在获取springBean的时候，使用别名也可以获取。

这个接口就是定义springBean注册别名的。

![image-20200903192611645](https://i-blog.csdnimg.cn/blog_migrate/2177c353e31771842430a57bd2d1b016.png)

这个接口定义的操作：

  1. springBean注册别名
  2. 移除别名
  3. 是否是别名
  4. 获取全部的别名

这个接口定义了springBean的别名的操作。

#### BeanDefinitionRegistry

springBean的详细定义的注册接口。spring容器有多个注册接口，每个接口的侧重点不同。

比如：

SigletonBeanRegistory侧重于单例类的注册

AliasRegistory侧重于springBean别名的注册

BeanDefinitioonRegistory注重于springBean详细信息的注册

![image-20200903193441217](https://i-blog.csdnimg.cn/blog_migrate/699e672c1d764f547a5a243435cff0a9.png)

这个接口主要定义：

  1. springBean详细信息的注册
  2. springBean详细信息的移除
  3. 是否包含指定springBean的详细信息
  4. 获取全部的springBean的详细信息
  5. 获取spring容器中注册的bean的详细信息的数量
  6. 获取spring容器中定义的bean的名字是否被使用

这个接口主要定义了springBean的详细信息的注册和管理。

到了这里，和BeanFactory相关的接口，都已经全部过了一遍，每个接口的侧重都不同，有些侧重springBean，有些侧重spring容器本身，有些侧重自动装配依赖，有些侧重解决依赖，有些侧重springBean的作用域，还有些是不同的注册方法。

### 实现

![img](https://i-blog.csdnimg.cn/blog_migrate/b7040a0aa6dc958cf90c93f82afe4a8c.png)

在前面我们基本上将这个关系中的接口都过了又一遍(Deprecated和SuppressWarning是注解)

接下来我们过一遍实现吧。

#### SimpleAliasRegistory

我们给这个类，起一个中文的名字吧。

别名简单注册器

`SimpleAliasRegistory`实现了`AliasRegistory`接口，所以，这个实现主要是完成接口中定义的内容。

![image-20200903195429340](https://i-blog.csdnimg.cn/blog_migrate/6bdaec16c9cd5cf8b5e7836d96d930ed.png)

这是核心的注册方法

![image-20200903195623975](https://i-blog.csdnimg.cn/blog_migrate/d6c47358e063040c39733ab71cedcc7f.png)

注册的方法也很简单，首先使用`synchronized`给`ConcurrentHashMap`加锁，然后注册别名。

简单别名注册器中的别名和bean的映射关系采用ConcurrentHashMap存储，格式是bean的别名对应bean的名字。

![image-20200903195848233](https://i-blog.csdnimg.cn/blog_migrate/301ffc72e7c7505458e1e8ede6196aed.png)

当然，注册之前还需要判断是否是循环别名。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb1f52a2355c61064f70997575443c73.png#pic_center)

为了保持并发安全，在简单别名注册器中，对于保存别名映射关系的`ConCurrentHashMap`的写入操作都是需要使用`synchronized`加锁，而读取则不需要。

这里面还有一个比较巧妙的实现：

获取指定beanName的全部别名。

首先这个映射关系是一个map，键是别名，value是beanName。

当然，value也可能是一个别名。

也就是说，这个map中实际上保存的可能是一个树(为什么不是网(注册的时候进行的循环注册检测))

所以，获取指定beanName的全部别名，就是获取这个beanName下的全部子树(不包含beanName)

获取一个树形结构，使用迭代是最好的选择。当然，写个死循环一次一次的遍历也能实现。

![image-20200903200745587](https://i-blog.csdnimg.cn/blog_migrate/9b23aabb3be09bf51469439bec1d542e.png)

为什么说比较巧妙呢？

我们知道，一个迭代必须满足两个条件：

  1. 出口明确
  2. 不会造成死循环
  3. 迭代深度数不会特别大(否则会造成方法栈溢出)

首先127行调用迭代，也就是迭代的入口，然后139行，隐含了迭代的出口，141行则是迭代的循环。

SimpleAliasRegistory使用`ConCurrentHashMap`保存<别名,beanName>的映射关系，实际保存的是一个树形结构。

#### DefaultSingletonBeanRegistory

默认的单例容器注册器继承了简单别名注册器，需要实现单例bean容器接口定义的全部操作：

![image-20200901195618699](https://i-blog.csdnimg.cn/blog_migrate/58dad7245c01ddbceec818a51176ec22.png)

![image-20200907154143806](https://i-blog.csdnimg.cn/blog_migrate/dac1ba0b7ff526e6a880ad107a4dd206.png)

在默认单例容器注册器中，有很多属性，这些属性用于记录单例容器的一些信息。

  * `SUPPRESSED_EXCEPTIONS_LIMIT`：默认100行异常堆栈
  * `singletonObjects`：存储beanName和beanInstance映射
  * `singletonFactories`：存储beanName和beanFactory映射
  * `earlySingletonObjects`：存储预加载的beanName和beanInstance映射
  * `registeredSingletons`：存储已经注册的beanName的set
  * `singletonsCurrentlyInCreation`：正在创建的beanName的set
  * `inCreationCheckExclusions`：当前正在创建忽略依赖的beanName
  * `suppressedExceptions`：异常
  * `singletonsCurrentlyInDestruction`：是否开始销毁单例
  * `disposableBeans`：存储销毁的beanName和beanInstance映射
  * `containedBeanMap`：存储beanName和beanName的别名的映射关系
  * `dependentBeanMap`：存储beanName和依赖的beanNames
  * `dependenciesForBeanMap`：存储beanName和依赖的beanName的映射关系

这个容器是由这些属性构成的。

接下来我们仔细分析，单例容器的操作，是如何基于上述属性完成的。

##### 注册–SingletonBeanRegistry

![image-20200907161048691](https://i-blog.csdnimg.cn/blog_migrate/2535e3b77945a08ccf7961b123e0cbe8.png)

将一个beanInstance注册到单例容器中，需要调用这个方法。参数一个是beanName，一个是beanInstance。

逻辑也是非常的简单，首先使用`synchronized`将`singletonObjects`上锁，然后首先尝试从Map中取，看看是不是已经注册了。

如果没有注册，则加入到`singletonObjects`中。

![image-20200907164915366](https://i-blog.csdnimg.cn/blog_migrate/d8d8dae48f0d0eb20153c56e442792f7.png)

当然，我们注册了一个beanInstance，那么未注册的beanInstance就少一个。所以需要移除beanName对应的beanFactory因为bean已经被注册，就表示，bean已经被创建完成了，而且当前容器是单例容器。换句话说，在单例容器中，一个bean被注册，那么这个beanName对应的beanFactory就失去了作用。

同时，因为beanInstance已经被注册，那么预加载的映射中也应该去除。

还需要在已经注册的beanName的set集合中加入。

##### 获取–SingletonBeanRegistry

![image-20200907165613718](https://i-blog.csdnimg.cn/blog_migrate/205e29493b1074f3d74c685edd1cf18a.png)

![image-20200907165706339](https://i-blog.csdnimg.cn/blog_migrate/0781112fbc3ca7079824020613dfe757.png)

首先尝试从`singletonObjects`中获取，如果获取失败，而且获取的bean是正在创建的bean，那么尝试对`singletonObjects`加锁(如果加锁成功，那么表示创建bean已经完成了，因为只有bean创建完成，而且放入`singletonObjects`中后，才会释放锁。而`syynchronized`是排他锁，不可重入锁，所以，获取锁，就表示创建完成。)

当然，也可能存在创建bean的线程创建失败，那么此时，`singletonObjects`还是空的，此时就需要获取的线程创建并刷新映射关系了。

否则就返回空。

##### 是否包含–SingletonBeanRegistry

这个倒是很简单了，直接看看已经注册的beanName的映射关系中是否包含指定的key(beanName)。

![image-20200907170431814](https://i-blog.csdnimg.cn/blog_migrate/a6442a5c733d770ed5432835240ce666.png)

##### 获取全部的beanName–SingletonBeanRegistry

这个和上面的非常类似，唯一的区别在于，如果存储已经注册的beanName的set非常大，那么整个过程比较耗时，就需要上锁了。

对于整个默认的单例容器注册器来说，最重要的就是`singletonObjects`映射，所以，加锁和释放锁，操作的一般都是这个核心的映射关系。

![image-20200907171057620](https://i-blog.csdnimg.cn/blog_migrate/a7ce397c089bf3ffe3e6362315efe112.png)

##### 获取单例容器中已经注册的bean的数量–SingletonBeanRegistry

![image-20200907171136740](https://i-blog.csdnimg.cn/blog_migrate/6ee3e4f1b4ecf20e5354a7e7e2bf1168.png)

##### 获取单例互斥对象–SingletonBeanRegistry

![image-20200907172232423](https://i-blog.csdnimg.cn/blog_migrate/734998c972fb6736b81bddcbcd2b48d9.png)

说实话，这个方法到底是干什么的，暂时没看明白，从注释上看，是将默认单例容器注册器中最核心的`singletonObjects`暴露出去。提供子类或者其他类使用。

##### 使用给定的beanFactory创建beanInstance

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e9d4f7ff254626c7a93dc6c8b64b6d7e.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb811fda447d53809dbfeab9b6da0774.png#pic_center)

这个方法就有意思了。

  * 给`singletonObjects`加锁
  * 当前单例容器是否正在销毁(如果正在销毁，就无法在创建beanInstance了，否则会造成映射关系残留)
  * 检查指定的beanName是否是排除的bean(创建前检查)(检查是不是排除的bean以及是不是正在创建的beanInstance)
  * 初始化异常集合对象
  * 使用传入的beanFactory创建bean.
  * 是否发生异常，如果发生异常，那么进行异常处理
  * 创建后检测(检查是不是排除的bean，以及是不是正在创建的beanInstance)
  * 将创建的beanInstance加入到`singletonObjects`中
  * 返回创建的bean

##### 移除

有创建肯定有移除。

![image-20200907182115539](https://i-blog.csdnimg.cn/blog_migrate/9aea651b7fffc0eddc97f2e22c6bea23.png)

首先对`singletonObjects`上锁，然后从`singletonObjects、singleFactory、earlySingletonObjects和registeredSingletons`移除。

##### 设置正在创建的bean

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b62fa1a872ded8d19fe63fd6a4582d1d.png#pic_center)

这两个方法很奇怪，是覆盖的子类的方法。这两个方法的定义在`ConfigurableBeanFactory`中的

![image-20200907182426682](https://i-blog.csdnimg.cn/blog_migrate/a263d82149f594de98b58c6c8e655918.png)

但是`DefaultSingletonBeanRegitry`并没有继承或者实现`ConfigurableBeanFactory`。所以是覆盖。

##### 注册有依赖的bean

在`DefaultSingletonBeanRegistry`中有一对属性:`dependentBeanMap和dependenciesForBeanMap`。这一对属性分别是依赖的bean和依赖的bean的映射关系。

![image-20200907183228865](https://i-blog.csdnimg.cn/blog_migrate/5c3c98223ecae8f1ddb6a14fb88af09f.png)

如果这个依赖的beanInstance的别名可能不存在，但是beanName一定存在。所以先处理别名，后处理映射关系。

##### 判断两个bean之间是否存在依赖

![image-20200907183752386](https://i-blog.csdnimg.cn/blog_migrate/000aaaf593ea8e5a74a6022ef29906c3.png)

这个关系就是从依赖相关的两个映射中读取的。

而且这判断还是一个迭代

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3cf7d68b727ea806b3e9ac0b19c3708c.png#pic_center)

第一次迭代，初始化了alreadySeen集合，然后将依赖关系中的每一个value都进行遍历。(依赖关系的数据格式：Map<String,Set< String>>)

第二次及之后的迭代，对每一个Set进行判断，是否包含(dependentBeans.contains)

出口：全部遍历一遍。

感觉这里用java8的流处理是不是更好些。

##### 获取依赖的beanName

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6868b7cd35af2f49d383644bc789f518.png#pic_center)

问一个正经问题，这个依赖关系是怎么设置进来的？

其唯一的入口就是调用注册有依赖的bean

##### 单例容器销毁

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1eab35f2aa5ee989807564ce2824fea0.png#pic_center)

有注册，肯定就有注销，这个注销的方法，也是`ConfigurableBeanFactory`接口定义的方法。

首先将销毁标志设置为销毁，然后从销毁的bean映射关系中获取需要销毁的beanNames，然后一个一个的销毁，最后将别名、依赖、依赖关系映射清空。

最后清空单例容器中全部的映射。

单个bean的销毁

![image-20200907185114465](https://i-blog.csdnimg.cn/blog_migrate/73941e0188a7b4e1d9a10aa3431906eb.png)

首先将该beanName从已注册集合中移除。

然后将该beanInstance从销毁映射中移除。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d693cc7240458b2f3e81d5de866c400c.png#pic_center)

最后先移除依赖关系，移除依赖。最后才是调用beanInstance的destroy方法移除beanInstance.

beanInstance移除后，需要同步将别名的beanInstance一样移除。

换句话说，bean销毁了，那么和这个bean有关的一切，都需要进行销毁，包含注册信息，依赖信息，依赖的bean，别名等。

##### DefaultSingletonBeanRegistory多线程处理

`DefaultSingletonBeanRegistroy`支持多个线程并行创建：

![image-20200907190529907](https://i-blog.csdnimg.cn/blog_migrate/e4d439674dcbbbf0120b2dc312eae568.png)

#### FactoryBeanRegistorySupport

##### 关系

`FactoryBeanRegistorySupport`继承于`DefaultSingletonBeanRegistory`，是一个抽象类。

![image-20200907190801300](https://i-blog.csdnimg.cn/blog_migrate/d6498d0ad3427e0518b5fb2228f884c2.png)

内部有一个factoryBean和beanName的映射。

##### 获取beanInstance的类型

![image-20200907192518014](https://i-blog.csdnimg.cn/blog_migrate/c4bb07d9e12a8076c4aa6a358a07ab48.png)

传入一个factoryBean也就是beanInstance，然后返回Class<?>也就是这个beanInstance的类型。

这个方法就非常的简单了，直接调用FactoryBean的getObjectType方法即可。

请注意，FactoryBean和BeanFactory的区别：FactoryBean的重点是Bean，也就是beanInstance；BeanFactory的重点是Factory，也就是beanCreater。

##### 获取正在创建的beanInstance的映射

同样的，这个抽象类，也将自己创建的beanInstance的集合容器暴露给子类。

![image-20200907192926754](https://i-blog.csdnimg.cn/blog_migrate/2c4221e0806753426581f0c32a34bb19.png)

##### 处理指定的FactoryBean

这个方法是这个抽象类的核心，功能就是处理指定的FactoryBean。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/50855c67210a60a99957d5412381f189.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aaf04be4e9037156b81c44034b48168f.png#pic_center)

  * 判断指定的FactoryBean是否是单例的
  * 如果是单例的，那么给`singletonObjects`上锁
  * 根据beanName获取beanInstance(FactoryBean)
  * 如果beanInstance是空的，那么执行`doGetObjectFromFactoryBean`
  * 二次检查，beanInstance是否在映射关系中(第一次获取的时候，可能其他线程在创建，但是还未写回)
  * 如果二次获取不为空，表示获取到了，那么将实际的beanInstance设置为二次获取的结果
  * 如果需要对beanInstance进行处理，那么首先会判断当前beanInstance不在其他线程中处理
  * 然后进行`beforeSingletonCreation`处理
  * 然后调用`postProcessObjectFromFactoryBean`处理
  * 最后是`afterSingletonCreation`处理
  * 全部处理完成，才会认为beanInstance创建完成了
  * 如果不是单例的，那么直接创建beanInstance即可，然后判断如果需要处理，就调用postProcess处理。(没有before和after处理)

##### doGetObjectFromFactoryBean

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1a42a38ef575a485d2d4bf4c40284ff4.png#pic_center)

这个就简单了。直接获取FactoryBean的getObject方法即可，如果获取的结果是个空，那么判断当前bean是否正在创建beanInstance，如果没有，那么返回一个空的Bean

##### beforeSingletonCreation

![image-20200907194926496](https://i-blog.csdnimg.cn/blog_migrate/ae51e3c2fe41b18c18a4fb3a5bed2f1a.png)

这里是一个提供给子类的创建前回调。在父类中会判断当前beanInstance是否正在创建。或者创建过程中出现异常。可以认为是创建前检查。

##### postProcessObjectFromFactoryBean

![image-20200907195118598](https://i-blog.csdnimg.cn/blog_migrate/50769e16b0bf9e5639a844641078dc27.png)

这也是一个提供子类扩展的方法，创建完beanInstance后进行的处理。

##### afterSingletonCreation

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16f560007dd8f844384e04e19333f478.png#pic_center)

这个方法是创建后处理，如果在创建的时候出现了异常，并不会立即抛出异常，而是记录下来，然后在这个方法中处理。

`beforeSingleCreation`是前置处理

##### 留给子类实现的扩展

`postProcessObjectFromFactoryBean`

#### AbstractBeanFactory

##### 关系

`AbstractBeanFactory`实现了`ConfigurableBeanFactory`接口，并且继承了`FactoryBeanRegistrySupport`抽象类。

也就是说，`AbstractBeanFactory`要实现父容器、类加载、类转换、前后置处理以及作用域等，这些是接口定义的操作。

而`ConfigureableBeanFactory`则继承了`HierarchicalBeanFactory`和`SingletonBeanRegistory`接口。

所以`AbstractBeanFactory`还需要实现上层容器的处理，以及单例注册的功能。

同时，`AbstractBeanFactory`也可以实现`FactoryBeanRegistrySupport`的bean前后置以及BeanFactory后的处理。

综上所述，`AbstractBeanFactory`需要实现的操作：

  * 多层容器
  * 类加载
  * 类转换
  * 前后置处理
  * 作用域
  * 单例注册
  * beanFactory后置处理
  * FactoryBean的创建

##### 多层容器

![image-20200909190043228](https://i-blog.csdnimg.cn/blog_migrate/5acf047cb718e6095b0339d4a2b3746b.png)

在`AbstractBbeanFactory`中，有一个父容器，而这个父容器也是一个spring容器，可以是任意类型的spring容器，只要实现`BeanFactory`即可。

所以，spring容器的嵌套是不做任何限制的，可以有多层容器。

而每一层容器实现了什么功能，作为子容器，是完全不知道的。

父容器有两种方式设置：1.构造器传入；2.set方法设置。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/14f2e01ee429da7bab54316167832021.png#pic_center)

![image-20200909190533290](https://i-blog.csdnimg.cn/blog_migrate/41e726cb75f83a029a2000dfb3afd427.png)

而且，父容器不允许重复设置。

父容器可以通过get方法获取

![image-20200909190406240](https://i-blog.csdnimg.cn/blog_migrate/37f94365ed310ba6972addc985bc2c9f.png)

##### 类加载

在`AbstractBeanFactory`中，持有两个类加载器。一个是默认的当前线程的类加载器，另一个是临时的类加载器。

![image-20200909190749258](https://i-blog.csdnimg.cn/blog_migrate/14a915911e01bae9d8930571efb19be8.png)

默认的类加载器加载顺序：当前线程的类加载器>ClassUtils的类加载>系统默认的类加载器

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d7f7bf3b1f73ffe59692242b21376281.png#pic_center)

类加载器也是可以指定的。

![image-20200909191009888](https://i-blog.csdnimg.cn/blog_migrate/e38e6b61d2d0970f1b86fe1edc652198.png)

##### 依赖处理

![image-20200909191124794](https://i-blog.csdnimg.cn/blog_migrate/3e38db60be466695114eac4f38680949.png)

依赖解析器，也是可以指定的。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6a0973c6d56b42203dc01ac6694ab8c5.png#pic_center)

在创建bean的时候，需要处理依赖，此时就会用到依赖解析器

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/871b0de63dd45ed89ed6c587aaf0df85.png#pic_center)

判断一个beanName是否可以进行类型匹配的时候，不仅仅需要beanInstance的类型匹配，其依赖也应该是匹配的。

![image-20200909191530643](https://i-blog.csdnimg.cn/blog_migrate/a720c460cb1c2aec93c48fd1a3b1c7d5.png)

这个方法就是如何从beanInstance的类型到全部依赖的类型做的比较，非常复杂。

##### 属性配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4096cf07ea91fef6c5c4e651aa85cc26.png#pic_center)

属性配置分为两个种类：

一个是客户化的属性注册，另一个是简单的属性编辑器。

属性注册器：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25af6f6375313e43659789f6e6eb4c8d.png#pic_center)

属性编辑器：

![image-20200909193451008](https://i-blog.csdnimg.cn/blog_migrate/86edb1bedbe7b69248c6b8e6af81ed07.png)

##### 类型转换

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d7a3cfb3e2381b1e35c6d2543eed5f3.png#pic_center)

可以指定自定义的类型转换器，如果不设置，就使用默认的类型转换器。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4cd5e153796f79024f65ac863a176552.png#pic_center)

然后将类型转换器注册到自定义的编辑器中即可。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c9ebc14891289e82a3f309751be4227.png#pic_center)

如果属性编辑器不为空，那么将每一个属性编辑器都进行注册。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/285aeb92f6546f0a870deee4f58e8848.png#pic_center)

##### 属性解析

![image-20200909194107103](https://i-blog.csdnimg.cn/blog_migrate/54709e34ab8d5120f92221682edc486c.png)

字符串类型的属性解析器

![image-20200909194126829](https://i-blog.csdnimg.cn/blog_migrate/ea7d32e27adad56ffd8f8c0b629ac7e3.png)

可以传入一个字符串，然后进行属性解析，返回解析后的属性

![image-20200909194218357](https://i-blog.csdnimg.cn/blog_migrate/1e799910a630cd8edcda64a98b18a0ac.png)

其实就是说，自定义配置的配置可能形形色色，没有一定的标准。而这个方法就是将传入的不标准的属性，尝试用全部的解析器进行解析，只要有一个能够成功解析，就立刻返回。

##### bean处理器

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b848b0f3b9c512a3b4b7dba6b4fa0861.png#pic_center)

bean处理器分为前置处理器和后置处理器

![image-20200909195005475](https://i-blog.csdnimg.cn/blog_migrate/1557a5e26172472b6dda8d59f924def6.png)

加入的时候会先移除旧的处理器，然后判断是类bean处理器，最后才是真正的加入。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8764c042d7c553573e3b6956c918e9f9.png#pic_center)

bean处理器有很多的种类

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3da57a432937495a4c4c458841d26474.png#pic_center)

`BeanPostProcessor`提供的是全体bean的前后置处理，也就是容器级别的bean前后置处理。而有些则是类信息合并的，有些设计冲突解决的，有些是专门为注解开发的。功能各不相同。

不过这些信息在`AbstractBeanFactory`中都没有用到

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/60e2000a944382bae5dffb5cac46f8ab.png#pic_center)

而是提供给子类使用了。

看来`AbstractBeanFactory`留给子类扩展的操作，应该和bean处理器有关。

##### 作用域

![image-20200909195748945](https://i-blog.csdnimg.cn/blog_migrate/a2159fa9677f922b7d24b10824119810.png)

`AbstractBeanFactory`实现作用域注册的方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/033bf7449bc8c4220a26214a8c069140.png#pic_center)

作用域注册方法，在spring的容器中只实现了一次。一方面这个比较简单，没有什么可以特殊化的，另一方面，`AbstractBeanFactory`是所有spring容器的父类。

##### 类信息

![image-20200909200139421](https://i-blog.csdnimg.cn/blog_migrate/d1e1f3bc883ee76a2ca81ae801328398.png)

`AbstractBeanFactory`使用`ConcurrentHashMap`存储类信息。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0c826b50e0f90df1e2c6db683a546fbd.png#pic_center)

bean信息在有写入操作的时候，需要加锁，使用`synchronized`加锁。

子类的bean需要和父类的bean进行类信息合并。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fad857202ca6adce19dcdedf3139bab4.png#pic_center)

比如A继承于B，A有beanInstance 小a，B有beanInstance 小b。

小a,b都在同一个容器中，在加载类信息的时候，小a,b的类信息都需要加载，在加载a的类信息的时候，就已经将b的类信息加载了。如果在加载b的时候，不进行类信息的合并，那么，就会造成b有两个类信息。

也许你会这样认为，类信息使用map存储，如果重复就直接覆盖不就完了吗？》‘

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/742671ddee55959b092297a8c6b86fc8.png#pic_center)

在写入的时候，键是beanName。

如果不合并，此时就是这样的

| key | 类信息 |
|-----|-----|
| a   | A   |
| a   | B   |
| b   | B   |

这样在后续处理的时候，会造成歧义，所以，这里会进行合并。

`AbstractBeanFactory`默认是单例bean

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6b56d5edccf24e0899673fe90f6466f0.png#pic_center)

##### 留给子类扩展

是否存在类信息

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d46f16cb2d28043233dc982d08319316.png#pic_center)

获取类信息

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf0b727c3c0993a1d619ac543fdac4aa.png#pic_center)

根据类信息和参数创建beanInstance

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f098551e8e1fb9171a6f34de7c41055.png#pic_center)

可以看到，`AbstractBeanFactory`是没有实现`FactoryBeanRegistrySupport`中前后置factory后置处理器的，全部交由子类实现。

#### AbstractAutowireCapableBeanFactory

##### 关系

`AbstractAutowireCapoableBeanFactory`继承了`AbstractBeanFactory`,同时实现了`AutowireCapableBeanFactory`接口。

既然`AbstractAutowireCapableBeanFactory`继承了`AbstractBeanFactory`那么，就必须实现抽象接口或者继续传递。很明显`AbsttractAutowireCapableBeanFactory`是继续传递这些未实现的方法。

`AbstractAutowireCapableBeanFactory`实现了`AutowireCapableBeanFactory`接口，就需要实现这个接口定义的功能。

也就是必须实现：创建Bean，自动装配Bean，配置Bean，初始化Bean，执行beanInstance的前后置处理，初始化Bean，销毁Bean，解析Bean的依赖等操作。

##### 创建Bean实例

创建Bean实例的策略，默认是Cglib

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2f6ec229e65b76c62bd89e328a65afa2.png#pic_center)

这是`AbstractAutowireCapableBeanFactory`的创建Bean实例的方法：
    
    
    	// 传入beanName和bean的信息，还有参数
    	@Override
    	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    			throws BeanCreationException {
    
    		if (logger.isTraceEnabled()) {
    			logger.trace("Creating instance of bean '" + beanName + "'");
    		}
    		RootBeanDefinition mbdToUse = mbd;
            // 确保无论如何得到一个Class对象。而且这个Class对象还必须和传入的值有关系。
    		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
            // 如果bean定义中没有得到的bean信息，那么需要加入。
    		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
    			mbdToUse = new RootBeanDefinition(mbd);
    			mbdToUse.setBeanClass(resolvedClass);
    		}
            // 进行方法重写准备
    		try {
                // 判断重写方法队列是否为空
                // 判断需要重写的方法在类整个关系中出现了几次
                // 同样的方法出现超过1次，才需要重写
                // 否则就把重写标志设置为不重写
    			mbdToUse.prepareMethodOverrides();
    		}
    		catch (BeanDefinitionValidationException ex) {
    			throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
    					beanName, "Validation of method overrides failed", ex);
    		}
    
    		try {
                // bean的前后置处理
                // 在resolveBeforeInstantiation方法中，首先会获取BeanPostProcessor
                // 然后执行BeanPostProcessor的前后置处理
                // 如果没有前后置处理，那么就返回null
    			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
                // 如果有前后置处理，那么就直接返回处理后的bean
    			if (bean != null) {
    				return bean;
    			}
    		}
    		catch (Throwable ex) {
    			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
    					"BeanPostProcessor before instantiation of bean failed", ex);
    		}
    
    		try {
                // 否则执行创建bean
                // 创建完bean后会执行合并处理的bean处理器
                // 如果bean是单例bean实例，那么需要将bean实例加入到单例容器中
                
                // 接着初始化bean
                // 自动装配(根据名称、类型、混合等方式自动装配)
                // 执行bean的前置处理
                // 执行factory后置处理(afterPropertiesSet方法)
                // 执行bean的后置处理
                
                // 如果是单例bean，那么将bean注册到单例容器内
                // 依赖处理，设置作用域，设置自定义销毁方法
                //
    			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    			if (logger.isTraceEnabled()) {
    				logger.trace("Finished creating instance of bean '" + beanName + "'");
    			}
                // 结束
    			return beanInstance;
    		}
    		catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
    			// A previously detected exception with proper bean creation context already,
    			// or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
    			throw ex;
    		}
    		catch (Throwable ex) {
    			throw new BeanCreationException(
    					mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
    		}
    	}
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/36b693811c5fe0fc63a617f4f20393ec.png#pic_center)

这是整个的时序图：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff0cfcb42f6ce10a77e89f9148c8479d.png#pic_center)

巨复杂，不过，看起来挺容易，挺舒服的。这里面大量使用了模板方法的设计模式，哪些操作，哪些属性可以暴露给子类，哪些是子类不需要关心的等等，分的非常清楚。

当然，创建bean不止这一个方法，不过这是最全的，其他几个方法营业都是调用这个方法实现的。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/374d1bb766e55d08eede919e1161d1fc.png#pic_center)

##### bean前置处理

在AutowireCapableBeanFactory中定义了前后置处理器(全体bean生效)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d454c18b7a7028e13c35832dfcb418d0.png#pic_center)

##### bean处理器factory后置处理器

在`FactoryBeanRegistorySupport`中，要求子类实现或者说覆盖factory后置处理器：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/661e5b027e4fd10c9be9bd5ca9fd1914.png#pic_center)

直接执行后置处理器。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/861aa2d475d3daffefc2efb6200523d3.png#pic_center)

##### bean后置处理器

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c60ce54d1a904af4402a9c8ee71dd985.png#pic_center)

##### 自动装配

毕竟这个类最核心的就两点：1.创建bean；2.自动装配bean；

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6dd61df3d34cc7330f12a198ab6fbd4e.png#pic_center)

自动装配的方法有三个。

首先看第一个，这是时序图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76ad99de5b8efaabcfdd1f3e0dcf6100.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6d08879b6551972cddc7fce59d804e44.png#pic_center)

大概的逻辑就是创建类信息，然后设置作用域，获取创建策略，使用创建策略创建实例。

然后设置属性依赖等等。

其余两个方法大同小异

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/62213fe76bb306b37368266cf0dc7ea2.png#pic_center)

虽然这也是一种获取bean的方式，但是这种方式获取的bean都是一次性的，不是单例的，可复用的。

##### 配置bean

虽然说是配置bean，实际上，这个方法也是一个获取bean实例的方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cc485406c6e11895a3446e0965e6b1cd.png#pic_center)

##### 销毁bean

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e7eeb087ce1c20805e3a45437556fba.png#pic_center)

创建了销毁bean的适配器。

这里使用了适配器模式。

[23种设计模式----适配器模式----结构模式](<https://blog.csdn.net/a18792721831/article/details/82702551>)

适配器里面的销毁方法

![image](https://i-blog.csdnimg.cn/blog_migrate/81e30c2e683526b8176ff1b6aff0351b.png#pic_center)

会在销毁方法中调用销毁前处理。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/22767989d63054ce2b695bc1736b5d57.png#pic_center)

看到这里，bean的生命周期中的方法，除了注解的两个方法暂时没见过外，其他的方法都看到了，也知道了具体是在什么时机调用的。

##### bean创建时依赖忽略

`AbstractAutowireCapableBeanFactory`在创建bean时，肯定需要忽略一些依赖，所以也提供了两个方法用于忽略接口和忽略类型。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/22da62c9ed0902cc10d4310551580a45.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/61c5b1ddee181de7f5e5aa46dc3d9ec8.png#pic_center)

##### 子类扩展

`AbstractAutowireCapableBeanFactory`实现了`AbstractBeanFactory`的createBean方法，但是其他两个getBeanDefinition和containsBeanDefinition都没有实现。  
`AbstractAutowireCapableBeanFactory`实现了`FactoryBeanRegistorySupport`的factory后置处理器。

同时也实现了bean的前后置处理器的调用(实际实现是自定义的)

这个beanFactory基本上实现了核心的功能。

创建bean,DI都貌似实现了。

而它本身就是一个抽象的BeanFactory。

换句话说，如果`AbstractAutowireCapableBeanFactory`将剩下的两个方法实现，那么，它就可以当做一个最简的BeanFactory使用了。

#### DefaultListableBeanFactory

最简spring容器，这个spring容器是其他容器的基础，其他容器都是扩展这个最简的基本容器实现的。

既然如此，最简容器就需要实现全部的抽象方法。

包括获取bean的信息，是否存在bean的信息。

##### 关系

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9456ad269bd1ec2c84a63a36c8b27fe4.png#pic_center)

`DefaultListableBeanFactory`继承了`AbstractAutowireCapableBeanFactory`类，需要实现`containsBeanDefinition`和`getBeanDefinition`方法。

`DefaultListableBeanFactory`实现了`ConfigurableListableBeanFactory`接口，实现依赖处理。

`DefaultListableBeanFactory`实现了`BeanDefinitionRegistory`接口，实现bean信息的处理。

`DefaultListableBeanFactory`实现了`Serializable`接口，使得beanFactory可以序列化和反序列化。

##### 抽象方法实现

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/61d9f9f81d1a28c1eaa8767f46b049a8.png#pic_center)

可以看到这个方法，是从一个Map中查询：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3a5cf1fe709a14d52c910a0023edd066.png#pic_center)

那么，是什么时候方进去的呢？

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/72facdf2521596eb5186df5124897810.png#pic_center)

通过跟踪代码，发现是在注册bean信息的时候方进去的。

在方进去的时候，首先会验证bean信息，验证bean信息主要是看看这个bean类型有没有需要被覆盖的方法，如果有就进行方法的覆盖。

接着会尝试获取，看看这个bean信息是不是已经注册了，如果已经注册了，那么允许不允许进行覆盖，如果允许覆盖则进行覆盖。覆盖的时候未加锁。

接着判断这个bean信息是否已经创建过bean，如果已经创建过bean,那么不仅仅需要更新bean信息的映射，还需要更新beanInstance。(更新的时候会加锁，因为会更新beanInstance的映射)

如果bean信息没有创建过bean，那么就直接更新。

如果bean信息对应的bean是单例的bean，而且beanInstance已经创建成功了，那么移除bean信息(因为beanInstance是单例，而且beanInstance已经创建成功了，就不在需要bean信息了)

这是注册bean信息的时序图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d411ab71d2877bb5b1c41a4e7d0308fa.png#pic_center)

整个`rehistoryBeanDefinition`还是挺复杂的。

还有一个抽象方法是`getBeanDefinition`，这个方法倒是很简单

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0914017ba50575151266601d52624189.png#pic_center)

直接从`beanDefinitionMap`中获取即可。

##### 依赖处理

依赖处理是`ConfigurableListableBeanFactory`中定义的操作，主要是忽略依赖，冻结bean信息，以及创建单例beanInstance。

忽略依赖很简单，直接将依赖放入忽略依赖的对应的Map中即可：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/557ef443639c9a8741ed1246d76cd728.png#pic_center)

用的时候也很简单，如果对应的类型在忽略的依赖中，那么就跳过这个类型，这样就是忽略类型了。

`ConfigurableListableBeanFactory`还有比如手动指定依赖的方法：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9363cf576f0d5f597342375eb59c997b.png#pic_center)

当然还有单例实例的处理：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e6d14f4fd2d78ee864d394b60dad096.png#pic_center)

首先会遍历全部的bean信息，然后从合并后的bean信息映射中，找到这个bean信息。

接着判断bean信息是不是抽象的，是不是单例的，是不是懒加载的。

如果都不是，那么就是单例的，可实例化的，需要初始化加载的。就会调用`AbstractBeanFactory`中的`getBean`方法获取beanInstance。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44b7269da5ca77172dfcaf5d9edc9639.png#pic_center)

如果是单例的bean，那么就从单例beanName的Map中找，否则就从全部的beanName的Map中找。

##### bean信息处理

`BeanDefinitionRegistry`定义的就是bean信息的处理操作。

但是怎么说呢，`AbstractBeanFactory`也实现了`BeanDefinitionRegistry`接口，但是`AbstractBeanFactory`实际上没有实现其中的两个方法，这两个方法就是抽象方法`containsBeanDefinition`和`getBeanDefinition`方法，而这两个方法，在`DefaultListableBeanFactory`中再次因实现`BeanDefinitionRegistry`接口，而要求实现，所以这两部分属于重复了。接口关系虽然不同，但是定义的方法相同

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/04fe1c1a2a3e5a722a8aaaa1b82529d6.png#pic_center)

bean信息的处理，主要还是依赖于这些属性：1.beanDefiniitionMap；2.frozenBeanDefdinitionNames；3.beanDefinitionNames;

##### 序列化和反序列化

我们知道序列化和反序列化主要调用的是readObject和writeObject方法。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7d3af1b6ff30aaf99f3f4e81dce44678.png#pic_center)

没想到吧，`DefaultListableBeanFactory`是不允许进行实例化的。

##### 查漏补缺

怎么说`DefaultListableBeanFactory`是一个完全的默认的spring容器，代码量还是有的2065行呢，看看除此之外，还有没有什么比较重要的逻辑。

获取有指定注解的beanNames

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b437756159daf38e6ddd73095b9d4459.png#pic_center)

这个方法很重要，特别是spring新版本推荐使用注解配置。

##### 使用

它有两个构造器

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37615c0b58e424bded4948564ebba5ac.png#pic_center)

但是貌似这个spring容器还是不够智能，需要手动创建，手动注册。

不管怎么说，这至少是目前我们遇到的第一个可以实例化的spring容器。尝试使用下。

首先我们创建三个bean类型

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/786d1ad5995e385715b63bb272401d0b.png#pic_center)

很简单的类型

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d3cd373d7b82ed4c63b18be5307a7ad4.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c2b6d77955bb65314bbf4c6e0f5a981.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/29b5579106aede57fe7f027c1c37d9d8.png#pic_center)

很简单的模型：

客户下有用户，客户有客户级别的产品。用户下有产品。产品只有名字。

接着，我们创建一个main.

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c03467c24b57400d6d68e8e6a17368c8.png#pic_center)

尝试运行，发现虽然报错了，因为我们创建beanDefinition不正确还是依赖的问题。

不过，至少我们想要的目的都达到了

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b7adc5e7d290b0e74709f4c81785564.png#pic_center)

我们告诉了spring容器三个bean的bean信息，然后spring容器给我们创建了三个beanInstance.

如果创建两个bean处理器呢？

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/973d63c9d17151563a9b4266bfac107a.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b277df6dfebca9c8f30350e6c5fb9572.png#pic_center)

但是因为我们没有主动处理依赖，所以，里面的依赖还是空的。

如果手动注册依赖呢？

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54870ae5633616c37176d0a145a1a3c3.png#pic_center)

很遗憾还是空的

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb6dc2206ae99d88bda31f6fcec0ec3e.png#pic_center)

我们虽然注册了依赖，但是没有处理依赖，所以：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/17f3696c2743e1dc28a177d25dd07acf.png#pic_center)

手动处理依赖。

此时执行，发现和预期的一致了：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d0964e5598679f3616177a9987d9cd96.png#pic_center)

至少有一定的顺序性了，至于对不对，就暂时不考虑了。

我们从beanFactory获取的bean，没有自动装配，所以，还需要手动调用自动装配。所以，使用autowire从spring容器中获取bean:

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/86198f046bb3cf5e01efa3b64d06f514.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/da74fee88ab3ff3e78e40e415985d5b2.png#pic_center)

哈哈，完美，虽然名字都是空的，但是至少，属性不为空了。

我们查看日志发现，注册的三个bean都是单例的bean，所以，如果我们将注册完product的bean后，将product的beanInstance的名字设置，那么，其余两个beanInstance中的product的名字应该和我们设置的是一样的。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76805b076a1c8ba0b754136151a0a91b.png#pic_center)

看看，这三个beanInstance里面的名字都是一样的。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/924d738e85f2f6028bdfec9e5deba7a5.png#pic_center)

#### XmlBeanFactory

在上面，我们用`DefaultListableBeanFactory`基本上能够实现spring容器中全部的功能，包括，创建bean，管理bean，自动处理依赖，自动装配等等。

但是在使用的过程中，发现还是不够智能，需要人工创建许多中间信息，然后`DefaultListableBeanFactory`才能帮我们简单的实现这些功能。

我想应该没有人会直接使用`DefaultListableBeanFactory`吧。毕竟`DefaultListableBeanFactory`是给框架使用的。

接下来我们就看一个启蒙级别的spring容器`XmlBeanFactory`

虽然这个spring容器已经被废弃了，但是我想，有一部分人接触spring都是从`XmlBeanFactory`开始的吧。

##### 关系

`XmlBeanFactory`的关系就很简单了，直接继承`DefaultListableBeanFactory`就完了。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/902e8daca0be6437cffcad053cc0c059.png#pic_center)

比起其他的，我们可能更关心，`XmlBeanFactory`主要做了什么吧？

##### XmlBeanFactory的操作

首先，`XmlBeanFactory`中有一个属性，这个属性非常重要，而且这个属性不是一般的属性：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e5f1534b0e73400cf12829c89d67ebd.png#pic_center)

请注意，这类创建这个属性的时候，将`XmlBeanFactory`给传入了。

也就是说，你可以认为，这个reader基本上拥有spring容器的全部信息。

`XmlBeanFactory`只有两个构造方法，核心是调用reader的loadBeanDefinitions方法。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/224903e9f1b6bf965b9c6e42eaabac78.png#pic_center)

##### XmlBeanDefinitionReader#loadBeanDefinitions

先对xml文件进行解码

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5c9cae7194d1bff9ae123d3987f5340b.png#pic_center)

然后获得指定编码后的输入流

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/082276d64c2cff800b5edfe6e313b50a.png#pic_center)

目前，一般xml验证的两种方式：DTD和XSD：

spring这两种方式都支持

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b5743a7ac3b2b223ac80341de29db982.png#pic_center)

而且是spring自己实现的，厉害哦。

OK，不跑题，我们继续看，通过输入流，获取了Document对象，然后根据Document对象注册beanDefinitions

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9d3719e1536dc3229db80516c70781ed.png#pic_center)

因为需要返回传入的Document解析了多少个beanDefinition（可能有多个xml配置bean，因为spring支持正则匹配配置文件）

所以首先先要获取注册前spring容器中已经注册了多少个beanDefinition

然后就注册本次需要注册的beanDefinition

最后做减法即可。

##### registryBeanDefinition

在这个方法中就实际开始解析bean信息了：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf1af079a2b69d94c7a5eaf9ba91be20.png#pic_center)

看看这里

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1ac0e5eb67fa06af6dd2d6be6721f13.png#pic_center)

是不是非常熟悉，这些都是在xml中配置bean的时候，可以配置的属性。

但是总感觉少了点，毕竟bean可以配置的属性可比这多的多了。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/575f374e3fc53f101b31dfebebca36c8.png#pic_center)

重点就在这个方法中了

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d7c1d2fd49f56b5819b6186872ed384.png#pic_center)

这个方法倒是也是非常的简单，那么，到底在哪呢？

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/af6bb387e0af2fb95aedc5403e577c1f.png#pic_center)

没错，是这个属性。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/baa5d83e6f5924bbe4cdec9b91087dfc.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/78f29ac5a13be776d75f98dac7b039cd.png#pic_center)

怎么说呢，`XmlBeanFactory`将自己传给了`XmlBeanDefinitionReader`，因为`XmlBeanDefinitionReader`拥有全部的spring容器的信息，需要访问的映射关系，也都能全部访问。所以在解析xml的时候，解析到一个，就设置到对应的映射关系中，需要执行的操作，在解析的时候，就会执行。

## 总结

### bean

bean就是一个Object实例。就是一个类的一个实例。

在spring中，bean一般只在容器中管理与存储的Object实例。

在spring容器中的bean都实现了FactoryBean接口。

### 容器

容器就是能够存储、管理Object对象的实例。

在java中一般是集合

在spring中是BeanFactory的实现类。

spring容器能够创建beanInstance，解决依赖，执行操作等等。(实际存储是依赖于java集合的，管理是spring容器的核心)

### Ioc

在非Ioc中，我依赖你，我就会主动创建你，我主动创建了你之后，就失去了扩展性。

在Ioc中，我依赖你，我不会主动创建你，而是声明，我有一个依赖，是你。这样，如果被依赖的对象发生了扩展，那么在Ioc中，我不用管做任何修改，就能实现被依赖的扩展的实现。

Ioc就是控制反转，这是没错的。控制反转的目的是为了更好的扩展性。

### DI

为什么DI在Ioc后面呢，DI就是依赖注入。

如果没有Ioc，那么DI就没有存在的必要了，依赖我自己都解决了，还需要什么注入呢？

在Ioc中，一个类依赖了另一个类，在依赖的类中只是声明，我要依赖另一个类。

但是实际上，谁将这个依赖实现呢？

就是DI，在DI中，需要处理循环依赖，处理依赖的依赖等等。
