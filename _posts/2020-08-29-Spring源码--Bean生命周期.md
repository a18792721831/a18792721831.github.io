---
layout: post
title: "Spring源码--Bean生命周期"
date: 2020-08-29 15:42:14 +0800
categories: [spring bean, bean生命周期, bean从创建到销毁, bean完整的生命历程, bean经过的每个阶段]
description: "Spring源码--Bean生命周期BeanNameAwareBeanFactoryAwareApplicationContextAwarepostProcessBeforeInitialization@PostConstructafterPropertiesSetpostProcessAfterInitialization@PreDestoryDisposableBean实例先来一张bean的生命周期图github地址：https://github.com/a18792721831/studySp_preconstruct predestory postprocessafterinitialization after propertiesset"
keywords: spring bean, bean生命周期, bean从创建到销毁, bean完整的生命历程, bean经过的每个阶段
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/108295349
> - 发布时间：2020-08-29 15:42:14
> - 阅读量：262
> - 分类：spring同时被 2 个专栏收录, 订阅专栏, springMVC
> - 标签：#spring bean, #bean生命周期, #bean从创建到销毁, #bean完整的生命历程, #bean经过的每个阶段

## 摘要

文章浏览阅读262次。Spring源码--Bean生命周期BeanNameAwareBeanFactoryAwareApplicationContextAwarepostProcessBeforeInitialization@PostConstructafterPropertiesSetpostProcessAfterInitialization@PreDestoryDisposableBean实例先来一张bean的生命周期图github地址：https://github.com/a18792721831/studySp_preconstruct predestory postprocessafterinitialization after propertiesset

---

#### Spring源码--Bean生命周期

  * BeanNameAware
  * BeanFactoryAware
  * ApplicationContextAware
  * postProcessBeforeInitialization
  * @PostConstruct
  * afterPropertiesSet
  * postProcessAfterInitialization
  * @PreDestory
  * DisposableBean
  * 实例

  
先来一张bean的生命周期图 

![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7173e533e627f1851026834f18f02b31.png)

github地址：

https://github.com/a18792721831/studySpringSource.git

## BeanNameAware

BeanNameAware的结构图非常简单

![image-20200829134754413](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b6db183b3fbdf7485753231b4b85fc2b.png)

其中Aware是一个标记性接口，内部没有定义任何方法：

![image-20200829134850253](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3d4beed7d3183d87fe82885f0f0aa609.png)

通过其注释就能看出来

![image-20200829134952050](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b13ed89f9944fbd8b95f3b422ac5adf6.png)

`BeanNameAware`接口只有一个方法：`setBeanName`设置bean的名字。

![image-20200829135326711](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cea7a5ba6b2915c34277e5aa79e67ff6.png)

说实话，我一开始对这个方法是一脸懵B的，这个方法有什么用呢？

从名字和注释上来看，貌似是设置这个bean的名字的。

但是仔细一想，不对啊，设置名字，不应该是参数为空，返回值为String，这样才能设置名字到BeanFactory中啊。

但是这方法刚好相反。

那么就用反向思路来思考：这个方法不是设置bean的名字的，而是获取bean的名字的。

结合spring的设计思想：普通的bean对于beanFactory是无感知的，也就是说，作为一个普通的bean，我不知道谁会依赖我，也不会知道是谁管理我，我就是一个普通的bean。

换句话说，最为一个普通的社畜，只要有活干，有砖搬就行了。管他给谁搬砖，搬砖做什么。

但是不能一辈子做一个社畜，人还是要有理想的。

作为一个bean，如果想知道自己在beanFactory中的名字，或者说代号，那么就需要实现`BeanNameAware`的接口，这样就表明了这个bean不是一个咸鱼，是一个有理想有抱负的bean，他想知道自己在beanFactory中的名字，不想做一个无名之士。

那么spring容器在启动的时候，发现这里有一个有志向的bean，就会仁慈的将这个bean在beanFactory中的名字通过`setBeanName`方法告诉这个bean.

总结来说：这个方法是bean获取自己在beanFactory中的名字。

## BeanFactoryAware

`BeanFactoryAware`和`BeanNameAware`类似，结构非常简单，都是继承了标记性接口：`Aware`接口，然后实现一个方法：

![image-20200829140559564](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ca9ff8c222ab61b4267ba91ce01e6207.png)

如果你理解了`BeanNameAware`接口的方法，那么这个方法也非常容易理解。

作为bean，如果想知道自己在哪一个beanFactory中，就需要实现这个接口，然后容器就会将这个bean所在的beanFactory的BeanFactory告诉bean。

举个例子哈：有些公司为了更加明确的区分业务，就会将原来一个完整公司，拆分成多个子公司，每个子公司负责一部分具体的业务。原公司里面的人还是这么多，甚至办公的工位都没有发生改变，但是已经是属于不同的公司了。如果一个员工在拆分后想知道自己是哪一个分公司的，就需要发邮件问人力资源。

`setBeanFactory`就是人力资源将这个bean所在的beanFactory返回给这个bean。

到了目前为止，这个bean已经和其他的bean有了明显的不同了，这个bean已经知道了自己是谁，自己在哪了。

它还是一个咸鱼吗？

## ApplicationContextAware

非常有名的三个问题：你是谁，你在哪，你从哪里来？

讲过前面两步，bean小兄弟已经知道了自己是谁，自己在哪，那么，还剩下一个问题，就是bean存在的意义，从哪里来。

`ApplicationContext`就是bean生活的范围，所以，清晰的认知自己生活的范围，就明白了自己存在的意义了。

![image-20200829145234557](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/af679d5a259e4fb7b428187d1b90cc53.png)

`ApplicationContextAware`的方法，就是告诉bean，运行时的上下文环境。

## postProcessBeforeInitialization

预初始化操作，针对全部的bean生效。

![image-20200829145910153](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/60b20e21f9c20666caf450a07e1d7b29.png)

可以实现偷梁换柱。将默认的一个bean替换为其他的bean，以最终return的结果为主。

## @PostConstruct

从Java EE5规范开始，Servlet中增加了两个影响Servlet生命周期的注解，@PostConstruct和@PreDestroy，这两个注解被用来修饰一个非静态的void（）方法。

![image-20200829150557581](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9ea09f40f135f34c733455abbacd78a1.png)

## afterPropertiesSet

InitializingBean接口为bean提供了初始化方法的方式，它只包括afterPropertiesSet方法，凡是继承该接口的类，在初始化bean的时候都会执行该方法。

这个接口的方法是怎么被调用的呢？

我们全盘搜索一下：`instanceof InitializingBean`

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5c07b1d86e6006119e70a18a8c51f08d.png)

如果init-method和afterPropertiesSet同时配置，先执行afterPropertiesSet方法，然后执行init-method.

## postProcessAfterInitialization

预初始化操作，针对全部的bean生效。

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6ba8f7cde200958af66dcbfccf9edd5d.png)

## @PreDestory

从Java EE5规范开始，Servlet中增加了两个影响Servlet生命周期的注解，@PostConstruct和@PreDestroy，这两个注解被用来修饰一个非静态的void（）方法。

![image-20200829152019234](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/045d716246db4d41c5693510d68ea9d0.png)

## DisposableBean

对于实现了 DisposableBean 的 bean ，在spring释放该bean后调用它的destroy() 方法。

![image-20200829152439912](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/be001aa6f5447170788d54b09c7dd545.png)

## 实例

创建一个bean
    
    
    import org.springframework.beans.BeansException;
    import org.springframework.beans.factory.*;
    import org.springframework.beans.factory.config.BeanPostProcessor;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.ApplicationContextAware;
    import org.springframework.stereotype.Component;
    
    import javax.annotation.PostConstruct;
    import javax.annotation.PreDestroy;
    
    /**
     * @author jiayq
     * @Date 2020-08-29
     */
    @Component
    public class People implements BeanNameAware, BeanFactoryAware, ApplicationContextAware, BeanPostProcessor, InitializingBean, DisposableBean {
    
        public People() {
            System.out.println("1.create");
        }
    
        @Override
        public void setBeanName(String name) {
            System.out.println("2.set bean name : " + name);
        }
    
        @Override
        public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
            System.out.println("3.set bean factory , name is " + beanFactory.getClass().getSimpleName());
        }
    
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            System.out.println("4.set application context , name is " + applicationContext.getClass().getSimpleName() + " , id is " + applicationContext.getId());
        }
    
        @Override
        public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("5.post process before initialization , bean name is " + beanName);
            return bean;
        }
    
        @PostConstruct
        public void init() {
            System.out.println("6.post construct , this name is " + this.getClass().getSimpleName());
        }
    
        @Override
        public void afterPropertiesSet() throws Exception {
            System.out.println("7.after properties set ");
        }
    
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("8.post process after initializattion , bean name is " + beanName);
            return bean;
        }
    
        public void say() {
            System.out.println("9.normal survival");
        }
    
        @PreDestroy
        public void destory() {
            System.out.println("10.pre destory");
        }
    
        @Override
        public void destroy() throws Exception {
            System.out.println("11.destroy by DisposableBean");
        }
    }
    

main类
    
    
    import com.study.source.beans.People;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.context.ConfigurableApplicationContext;
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    
    @SpringBootApplication
    public class SourceApplication {
    
        public static void main(String[] args) {
    //        ConfigurableApplicationContext context = SpringApplication.run(SourceApplication.class, args);
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext("com.study.source");
            People bean = context.getBean(People.class);
            bean.say();
            context.close();
        }
    }
    

输出结果：

![image-20200829152730962](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5fb1a8f385a08da83613ac9fe069f578.png)

![image-20200829152811633](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/59518299cef5169032a21bcdd63e22f4.png)

查看下`AnnotationConfigApplicationContext`的调用链

![image-20200829153038102](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/32e82ef67748f089ada7b06ca36ea76a.png)

其中的refresh就是spring中核心的内容了

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/007768df6d8fbe175d9089c39e2eb1e8.png)
