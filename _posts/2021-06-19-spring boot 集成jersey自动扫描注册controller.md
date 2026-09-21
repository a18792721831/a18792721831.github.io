---
layout: post
title: "spring boot 集成jersey自动扫描注册controller"
date: 2021-06-19 15:35:07 +0800
categories: [boot集成jersey, jersey配置, jersey自动扫描, 优雅的注册到jersey, jersey的使用]
description: "本文详细介绍了如何在Spring Boot项目中集成Jersey，并通过自动扫描注册控制器，提高代码组织效率。涵盖了项目配置、接口设计、实现类编写及JerseyServiceAutoScanner工具的使用。"
keywords: boot集成jersey, jersey配置, jersey自动扫描, 优雅的注册到jersey, jersey的使用
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/118054635
> - 发布时间：2021-06-19 15:35:07
> - 阅读量：882
> - 分类：微服务同时被 3 个专栏收录, 订阅专栏, spring boot, spring
> - 标签：#boot集成jersey, #jersey配置, #jersey自动扫描, #优雅的注册到jersey, #jersey的使用

## 摘要

文章浏览阅读882次。本文详细介绍了如何在Spring Boot项目中集成Jersey，并通过自动扫描注册控制器，提高代码组织效率。涵盖了项目配置、接口设计、实现类编写及JerseyServiceAutoScanner工具的使用。

---

#### spring boot 集成jersey自动扫描注册controller

  * [1\. 项目准备](<#1__1>)
  * [2\. 项目配置](<#2__22>)
  * [3\. jersey使用注意](<#3_jersey_43>)
  * [4\. jersey 扫描注册](<#4_jersey__136>)

## 1\. 项目准备

使用idea创建一个jersey的项目，核心依赖如下：
    
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jersey</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    

引入了jersey，actuator和web

## 2\. 项目配置

jersey引入后，还需要配置jersey

新建一个类，用于配置jersey
    
    
    @Configuration
    public class JerseyConfig extends ResourceConfig {
        @PostConstruct
        public void init() {
            register(IDemoController.class);
            setApplicationName("test");
        }
    }
    

这样就把`IDemoController`注册到了jersey中了。

同时设置应用的名字为test(如果访问，需要加应用名)

## 3\. jersey使用注意

配置好了后，还需要注意，在jersey中注册的controller中需要对应的注解。

我比较喜欢用接口定义对外暴露的服务，然后用其他类实现。
    
    
    @RestController
    @Path("demo")
    @Consumes({MediaType.APPLICATION_JSON})
    @Produces({MediaType.APPLICATION_JSON})
    public interface IDemoController {
    
        @POST
        @Path("helloworld-no-param")
        String helloworld();
    
        @Path("helloworld")
        @GET
        @NoRequestHeader
        String helloworld(@QueryParam("name") String name);
    
        @Path("helloworld-integer")
        @POST
        Integer helloworldInteger(String name);
    
        @Path("helloworld-int")
        @POST
        int helloworldInt(String name);
    
        @POST
        @Path("helloworld-obj")
        HelloWorldRes helloWorld(HelloWorldReq helloWorldReq);
    
        @POST
        @Path("helloworld-obj-exp")
        HelloWorldRes helloWorldException(HelloWorldReq helloWorldReq);
    }
    

这样做的好处是接口专门处理对外暴露的服务相关的配置。比如暴露的服务是什么访问路径，什么访问格式，传参格式，请求方式等等，都在接口中设置。

实现类不必处理这些，实现类专心处理实现。这样服务的业务实现和访问控制就分离了。修改业务逻辑，不会修改到接口的请求；修改接口的请求等，不会修改到业务逻辑。
    
    
    @Controller
    public class DemoController implements IDemoController{
    
        @Override
        public String helloworld() {
            return "hello world no param";
        }
    
        @Override
        public String helloworld(String name) {
            return "hello world : " + name;
        }
    
        @Override
        public Integer helloworldInteger(String name) {
            return name.length();
        }
    
        @Override
        public int helloworldInt(String name) {
            return name.length();
        }
    
        @Override
        public HelloWorldRes helloWorld(HelloWorldReq helloWorldReq) {
            HelloWorldRes res = new HelloWorldRes();
            res.setMessage(helloWorldReq.toString());
            People people = new People();
            people.setName(helloWorldReq.getName());
            res.setPeople(people);
            return res;
        }
    
        @Override
        public HelloWorldRes helloWorldException(HelloWorldReq helloWorldReq) {
            HelloWorldRes res = new HelloWorldRes();
            res.setMessage(helloWorldReq.toString());
            HelloWorldRes worldRes = new HelloWorldRes();
            worldRes.setMessage(helloWorldReq.toString());
            throw new QuanBackRuntimeException("exception.test");
        }
    }
    

这样做，还有另一个好处。如果以后需要对业务实现做重构，那么对接口新增一个实现，然后把原实现类上的注解注释掉即可。如果重构出现问题，可以及时的恢复。

这样做，虽然写起来可能比较繁琐，但是优点多多。

## 4\. jersey 扫描注册

注册一个类，需要写一次注册，如果我们有多个controller，每一个controller都注册一遍，显然比较费劲，而且在开发的时候，也容易遗漏，所以，需要一个可以自动扫描的工具类，帮我们扫描。
    
    
    public class JerseyServiceAutoScanner {
        private JerseyServiceAutoScanner() {}
    
        public static Class[] getPublishJerseyServiceClasses(ApplicationContext applicationContext, String... scanPackages) {
            // 传入applicationContext对象，在整个spring容器中捞我们需要的controller
            // 传入的第二个参数是可变参数，字符串，用于传入需要扫描的包路径
            List<Class> jerseyServiceClasses = new ArrayList<>();
            if (scanPackages == null || scanPackages.length == 0) {
                return jerseyServiceClasses.toArray(new Class[jerseyServiceClasses.size()]);
            }
            ClassPathScanningCandidateComponentProvider scanner = new JerseyScanningComponentProvider(false);
            // 我只需要扫描使用了@RestController注解的controller，如果还有其他的组合条件，可以在这里增加
            scanner.addIncludeFilter(new AnnotationTypeFilter(RestController.class));
            for (var scanPackage : scanPackages) {
                jerseyServiceClasses.addAll(scanner.findCandidateComponents(scanPackage).stream()
                        .map(beanDefinition -> ClassUtils
                                .resolveClassName(beanDefinition.getBeanClassName(), applicationContext.getClassLoader()))
                        .collect(Collectors.toSet()));
            }
            // 返回符合条件的spring容器中的全部的类对象
            return jerseyServiceClasses.toArray(new Class[jerseyServiceClasses.size()]);
        }
    
        private static class JerseyScanningComponentProvider extends ClassPathScanningCandidateComponentProvider {
            public JerseyScanningComponentProvider(boolean useDefaultFilters) {
                super(useDefaultFilters);
            }
            @Override
            protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
                AnnotationMetadata metadata = beanDefinition.getMetadata();
                return (metadata.isIndependent() && metadata.isAbstract() && !beanDefinition.getMetadata().isAnnotation());
            }
        }
    }
    

如何使用这个工具类？

非常简单。
    
    
    @Configuration
    public class JerseyConfig extends ResourceConfig implements ApplicationContextAware {
    
        private ApplicationContext applicationContext;
    
        @PostConstruct
        public void init() {
            registerClasses(JerseyServiceAutoScanner.getPublishJerseyServiceClasses(applicationContext, "com.test"));
            setApplicationName("test");
        }
    
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            this.applicationContext = applicationContext;
        }
    }
    

配置类实现`ApplicationContextAware`接口 ，用于获取`applicationContext`对象，从spring容器中捞需要的类。

然后通过工具类获取全部的符合条件的controller类，最后使用`registerClasses`一次性全部注册。

以后如果增加了其他的controller，也不需要在这里配置了，只要保证增加的controller在扫描的包下即可。