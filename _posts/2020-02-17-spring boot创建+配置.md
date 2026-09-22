---
layout: post
title: "spring boot创建+配置"
date: 2020-02-17 17:33:23 +0800
categories: [spring boot 创建, springboot默认配置, springboot配置yml, springboot自定义配置, springboot配置解析器]
description: "本文详细介绍SpringBoot的特点，包括自动配置、起步依赖和Actuator监控。涵盖SpringBoot项目的创建过程、项目结构、web项目搭建、启动类配置及测试方法。同时，深入探讨自定义配置、实体配置、properties和yml配置文件的使用，以及如何通过自定义工厂解析yml配置。"
keywords: spring boot 创建, springboot默认配置, springboot配置yml, springboot自定义配置, springboot配置解析器
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104290947
> - 发布时间：2020-02-17 17:33:23
> - 阅读量：864
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#spring boot 创建, #springboot默认配置, #springboot配置yml, #springboot自定义配置, #springboot配置解析器

## 摘要

文章浏览阅读864次。本文详细介绍SpringBoot的特点，包括自动配置、起步依赖和Actuator监控。涵盖SpringBoot项目的创建过程、项目结构、web项目搭建、启动类配置及测试方法。同时，深入探讨自定义配置、实体配置、properties和yml配置文件的使用，以及如何通过自定义工厂解析yml配置。

---

#### spring boot创建+配置

  * 1.spring boot 简介
  *     * 1.1 特点
  * 2.创建spring boot
  *     * 2.1 创建
    * 2.2 项目结构
    * 2.3 web项目
    * 2.4 启动类
    * 2.5 测试
  * 3.配置
  *     * 3.1 自定义配置
    * 3.2 如何访问
    * 3.3 配置赋值到实体
    * 3.4 自定义配置文件-properties
    * 3.5 自定义配置文件-yml

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1.spring boot 简介

### 1.1 特点

  *     1. 自动配置  
自动配置就是程序需要什么，spring boot就会装配什么。
  *     2. 起步依赖  
向原来的项目中增加依赖是比较有挑战的，版本冲突等必须考虑。在springboot中，只需要增加一个依赖，这个依赖就会将其相关联的依赖都加入，并自动解决版本冲突。
  *     3. Actuator监控  
Actuator可以监控程序的运行。

## 2.创建spring boot

### 2.1 创建

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9231ecdbe8e5893a31a900adc599b128.png)

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b60c3a7809a3075f70f71f349c68662e.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cc8e825c893a97391eab58c2c995662c.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7764f733f457b4ae73eacd2d29f484d8.png)

### 2.2 项目结构

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b629b5525d7daca3a4059f9a2b8a6554.png)  
build.gradle是依赖管理  
resources为资源文件夹  
statics为静态资源  
templates为模板资源  
application.properties  
application.yml是配置文件  
SpringbootApplication为程序启动类

### 2.3 web项目

web项目会自动增加依赖  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/dd3387a3c5a75670542e8177d4a35bae.png)  
有时候maven的默认仓库下载比较慢，可以自动多个maven仓库镜像
    
    
        maven{
            url 'https://maven.aliyun.com/'
        }
        maven{
            url 'http://maven.aliyun.com/nexus/content/groups/public/'
        }
        maven{
            url 'https://repo1.maven.org/maven2/'
        }
    

### 2.4 启动类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/34a3f840d31d703a998f120088ce4784.png)  
@SpringBootApplication注解包含了@SpringBootConfiguration和@EnableAutoConfiguration和@ComponentScan，开启了包扫描、配置和自动配置的功能很。

我们新建一个Controller  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a15ecec0510aa7bf9885746222cae132.png)  
绑定controller的请求地址，然后写一个方法，方法也绑定请求地址，然后启动。  
接下来访问方法  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/72d6d8b5aae7169b089834fcf2402204.png)  
请注意，不写方法的请求地址，也会报异常的  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9300199539407f735733fa5bfc63b199.png)

其中，@RestController注解表明这个类是一个RestController。  
@RestController是spring 4.0版本的一个注解，功能相当于@Controller注解和@ResponseBody注解之和。  
@RequestMapping注解是绑定请求地址映射的。

### 2.5 测试

在gradle项目中，我们看到，自动生成了测试类。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/dbd5fdee790fa40477d196569288ad57.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/98f09475133dc34de758183a41ad8929.png)  
我们直接运行，可以看到启动了容器  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0cb87a48a863344ee62620ef774132fa.png)  
不过这个contextLoads测试类只是为了测试自动生成或依赖没有问题，整个架构能够正确的运行。  
更多的是我们自己写的，测试自己的controller,service,dao.  
接下来，我们创建controller包（与main里面的同目录名，类名+Test）  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c0e035a3efa8794468d20189ff57aed2.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/84bdad304a85ab2e6e9afc0f8c71f232.png)

  *     1. 为了防止端口占用导致的服务启动失败，先使用@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)随机使用端口。
  *     2. JUnit4可以使用@RunWith(SpringRunner.class),但是在JUnit5中是@ExtendWith(SpringExtension.class)
  *     3. 使用@LocalServerPort将随机端口读取进去
  *     4. 使用assertThat进行判断是否通过测试(否则即使失败，也是通过)
  *     5. 可以将初始化的操作放在init中  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ca25540bb412ea32ab87ddfff429dfb3.png)

## 3.配置

在spring boot 中有两种配置文件，properties和yml  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8c4dc4d5763c71ad17ed39f4d7aae527.png)

### 3.1 自定义配置

在yml中配置属性  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/15b0faf3f08b2e1081f9cfcba68a9c53.png)

### 3.2 如何访问

首先创建一个controller  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8e9fde586d36bd4cd76d61e3fbf0180b.png)  
接下来启动  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d03ad9afdaba9c562a6b52827c7c990f.png)  
然后访问  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1500bdc18bb746a37d25ff68c9fb3f22.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6b02ab17e18ef521589f4ccd2c2b30d6.png)  
接下来创建测试类  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9a043b72bd4cbe937d8e92f2e1043722.png)

### 3.3 配置赋值到实体

在配置文件中增加  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/491b40e3f994f6afe5cdaac777529ecd.png)  
接下来创建对应的bean  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ae6f2e7400696d995e5f5e1adb34b953.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/718f372c3008bcabb8eda38042485d07.png)  
创建访问controller  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/831e3235edb156e106ee05be731a2055.png)  
启动请求  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/db07093fbbb9f16a1eec4439c312c367.png)  
写测试类  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/69886bf2e1bf05e9d29d26407c4a0e74.png)  
通过比较自动注入的和通过controller获取得到的是否一致。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d30295a06d0e2b5469e0c39540f99571.png)

### 3.4 自定义配置文件-properties

我们在resources下创建我们自己的配置文件  
myTest.yml  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/93027bc090b13e49940159e62a41673b.png)  
数据结构如下：  
有一个抽象类，定义name,age,value  
然后扩展了三个子类：  
man、women、child  
最后组成了people  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c6179be1cdb2cefe860b341558af4cd7.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/de5a5826c121dca20e179c0c9ac7d19d.png)

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/33cf0133a1df9a9b27251f233f024eeb.png)  
需要增加依赖  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ac01c0ed1cd790eecaff96649da3798b.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/073a8fba552092935ae8a25a84e1cf19.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6c1f4b253f08aa27df3ebb27845e2537.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/39988f465707f7345ac1aa6e22984bca.png)  
注意前缀。  
创建controller  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/031b36ce251842ffdc25b109ab82fc5a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6bce1e69fb3603f91af9842f25f5244a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/406d60faa89ed55d380fcdd87525ad90.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f853910783c652412802a7459694b2eb.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6fe9602f0e5e5b4bf86054185cf80cef.png)  
创建测试类  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/78c3c02d6e2c5738197fad6185732b15.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d83d3a13fc36ece0449796dc479b31c7.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c0f624e2c8895a6be58247565bfe3544.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/839f32ed62aebaa259b2bb9aa89b389d.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3fff9959ff99d223db1cb30ce2d5c863.png)  
运行全部测试  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3523aefdc628cb667beabce47f6a339b.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4ed9b50c4b5563326e7e8df06e8c22c4.png)  
全部测试通过。

### 3.5 自定义配置文件-yml

> 因为默认情况下，@PropertySource注释不适用于YAML文件。
> 
> 但是从Spring 4.3开始，有可能使其工作。Spring  
>  4.3引入了PropertySourceFactory接口。这PropertySourceFactory是一家工厂PropertySource。使用的默认实现是DefaultPropertySourceFactory创建ResourcePropertySource实例的。
> 
> 编写自定义实现需要实现单个方法createPropertySource。定制实现需要做两件事：
> 
> 将给定资源加载到java.util.Properties对象中 创建一个PropertySource来包装已加载的属性
> 
> 要加载YAML文件，Spring提供了YamlPropertiesFactoryBean。此类将加载1个或多个文件并将其转换为java.util.Properties对象。Spring提供了一个PropertiesPropertySource包装java.util.Properties对象的[  
>  ] 。最后，PropertySource的名称是给定的或派生的。如合同中所述，派生名称是资源描述。

上述内容引用自 https://mdeinum.github.io/2018-07-04-PropertySource-with-yaml-files/

首先，创建yml解析的工厂
    
    
    public class YmlPropertySourceFactory implements PropertySourceFactory {
    
        @Override
        public PropertySource<?> createPropertySource(@Nullable String name, EncodedResource resource) throws IOException {
            Properties propertiesFromYaml = loadYamlIntoProperties(resource);
            String sourceName = name != null ? name : resource.getResource().getFilename();
            return new PropertiesPropertySource(sourceName, propertiesFromYaml);
        }
    
        private Properties loadYamlIntoProperties(EncodedResource resource) throws FileNotFoundException {
            try {
                YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
                factory.setResources(resource.getResource());
                factory.afterPropertiesSet();
                return factory.getObject();
            } catch (IllegalStateException e) {
                Throwable cause = e.getCause();
                if (cause instanceof FileNotFoundException) {
                    throw (FileNotFoundException) e.getCause();
                }
                throw e;
            }
        }
    
    }
    

然后创建自定义的yml文件  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ea0545ead00da30de8b9300a0df5d2e0.png)  
注意，前缀不能重复，在所有的配置文件中，不能重复。(感觉有些不科学)  
然后创建对应的实体  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4eb97a77383b25a6bcf86807a0229646.png)  
注意点  
实体上的  
@PropertySource(factory = YmlPropertySourceFactory.class,value = “classpath:myTest.yml”)  
注解需要指定factory  
然后写controller  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fa1aec77e6bbebd9811ac9548aa21095.png)  
最后写Test  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c3a28fd117aea32236d2972069de2244.png)  
测试结果  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2c516fc025b307e5c10873c1d23dd86b.png)  
其余类似[3.4](<https://editor.csdn.net/md?articleId=104290947#34_properties_106>)

完整项目，见git.
