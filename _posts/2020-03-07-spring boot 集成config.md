---
layout: post
title: "spring boot 集成config"
date: 2020-03-07 20:30:43 +0800
categories: [微服务集成config, config与eureka, 微服务config集群化, 微服务config远程配置, 微服务config读取配置规则]
description: "本文详细介绍SpringBoot集成ConfigServer的步骤，包括本地配置、从Git读取配置、集群化及与Eureka集成实现高可用。通过具体示例，演示如何创建、配置并验证ConfigClient与ConfigServer的交互。"
keywords: 微服务集成config, config与eureka, 微服务config集群化, 微服务config远程配置, 微服务config读取配置规则
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104716562
> - 发布时间：2020-03-07 20:30:43
> - 阅读量：5
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#微服务集成config, #config与eureka, #微服务config集群化, #微服务config远程配置, #微服务config读取配置规则

## 摘要

文章浏览阅读5k次，点赞15次，收藏98次。本文详细介绍SpringBoot集成ConfigServer的步骤，包括本地配置、从Git读取配置、集群化及与Eureka集成实现高可用。通过具体示例，演示如何创建、配置并验证ConfigClient与ConfigServer的交互。

---

#### spring boot 集成config

  * [1\. config server(本地)](<#1_config_server_3>)
  *     * [1.1 创建](<#11__4>)
    * [1.2 配置(config server本身的)](<#12_config_server_6>)
    * [1.3 配置(本地对于config client的)](<#13_config_client_10>)
    * [1.4 注解](<#14__16>)
  * [2\. config client](<#2_config_client_18>)
  *     * [2.1 创建](<#21__19>)
    * [2.2 配置](<#22__21>)
    * [2.3 验证快速失败](<#23__29>)
    * [2.4 读取验证](<#24__33>)
    * [2.5 普通信息读取](<#25__48>)
  * [3\. config server从git读取](<#3_config_servergit_59>)
  *     * [3.1 创建远程配置文件](<#31__60>)
    * [3.2 修改config server 配置](<#32_config_server__65>)
  * [4\. config server 集群化](<#4_config_server__76>)
  *     * [4.1 eureka server](<#41_eureka_server_79>)
    * [4.2 config server & eureka](<#42_config_server__eureka_82>)
    *       * [4.2.1 创建springbootconfigeurekaserver](<#421_springbootconfigeurekaserver_83>)
      * [4.2.2 配置](<#422__85>)
      * [4.2.3 多实例启动](<#423__88>)
      * [4.2.4 验证](<#424__92>)
    * [4.3 config client& eureka](<#43_config_client_eureka_104>)
    *       * [4.3.1 创建springbootconfigeurekaclient](<#431_springbootconfigeurekaclient_105>)
      * [4.3.2 配置](<#432__107>)
      * [4.3.3 启动验证端口](<#433__110>)
      * [4.3.4 验证负载均衡](<#434__113>)
      * [4.3.5 验证message](<#435_message_121>)

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. config server(本地)

### 1.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4dd012d05162dd75278278fc2cb43032.png)

### 1.2 配置(config server本身的)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2ceb3579e64b4fd64d2c2339f1d26b2e.png)  
特别注意，需要在config server中配置profiles是native否则，config server默认从github仓库读取，启动时会异常：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c1bacda7296e5625701c85a80223b5ed.png)

### 1.3 配置(本地对于config client的)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/83714178a379b0d719f476909f85794a.png)  
注意文件夹的名字在1.2中指定的。  
同时这个配置文件的名字是config client的服务名字加标志。  
这个标志是标志是什么环境(开发，测试，生产等)  
可以随意填写。但是建议使用大家都明白的名字。

### 1.4 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c6a4404731d0b5ee9bee59593c20f77.png)

## 2\. config client

### 2.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/83d4400d17aacce3b3a72a7a6c7e7c79.png)

### 2.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/392e1e7fe8c08f317ec3032c252aa88c.png)  
注意配置文件名。  
bootstrap.yml优先于application配置读取。  
指定了config client的服务名字；  
制定了config server的地址；  
开启了读取失败时，快速失败终止；  
然后制定了读取的标志。

### 2.3 验证快速失败

只启动config client,不启动config server,验证快速失败。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4695e9d5655939806fa6815f0fea3bb.png)  
异常，读取配置异常，然后就终止了。

### 2.4 读取验证

先启动config server 然后启动config client  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/02bef7e3ece0467aeff430c8dcb6d090.png)  
它默认发布了好多的接口。  
我们尝试访问：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/74d5e05bceb94d5afb88f6f08370f3d9.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/22d1a0b2c62a3ae3f11f9a1b3c45ce9e.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/00385a679cb018de13d410ca1ee8a27d.png)  
你在指定的config clieng配置文件夹portconfig中的配置文件的信息都可以访问到。  
接下来启动config client  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4212d5852c668d76dce745ea9b76f9fd.png)  
实际上我们并没有在config client中为config client指定端口，而是在config server中指定了端口。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7aaebbd1ae6bc07e4d34fe4e115db4c0.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ccc318320f4818e90b308acd1784233.png)  
但是它依然使用我们指定的进行启动的。

### 2.5 普通信息读取

我们在config server中配置config client的端口的文件中还配置了一个message的配置，接下来将尝试读取这个message的值。  
首先需要改造config client,增加开放接口，用于读取message.  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4458eff9e6963cc3d3f0b5e0d4f67e9a.png)  
创建一个controller，在controller中有一个属性，这个属性就是对应的配置文件中的message,使用@Value读取。  
接着开放了一个接口，这个接口返回message的值。  
启动config client  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/492949a53232b2c0c973d30178d67f1a.png)  
访问：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5d91aa78a7bb1bd48011a66c821b4da7.png)  
成功读取。

## 3\. config server从git读取

### 3.1 创建远程配置文件

首先将之前的配置文件上传至guthub  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7406453e8c13b7ed5c90f8b741e037bf.png)  
然后将配置文件中的端口从8011修改为8012  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/36bb5b637ca6b9364397dc7067765cd0.png)

### 3.2 修改config server 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/63021eeeec2d0c5db0bc680a5ac7e793.png)  
搜索文件夹地址从这里得到  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a54b890ce6c2e66d01a3df0119e81000.png)  
然后重新启动config server 和config client  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b9dc23f23f00d6efb09d84bc2ce344dc.png)  
可以看到其端口已经变成远程配置的端口了。  
然后访问message  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8af04c066621211abe7302e5026a7db9.png)  
原来的端口已经无法访问了，我们换成新的端口试试。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff334e6d3e13dccca393db58c0ccc662.png)

## 4\. config server 集群化

config server集群化，是集成eureka进行高可用的。  
也就是在原来的基础上需要引入eureka server工程，且在config server 与config client中使用eureka。

### 4.1 eureka server

eureka server直接使用前面项目创建的eureka server,我们这里不会重新创建。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8bb5130465583330bae7556e334dbb9d.png)

### 4.2 config server & eureka

#### 4.2.1 创建springbootconfigeurekaserver

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de51e7ebf1d4baecf715c929141ca9a6.png)

#### 4.2.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4dfe35db38b33044a70300cd5a7fec49.png)  
直接将3.2的配置文件拷贝过来就行,记得修改服务名称。

#### 4.2.3 多实例启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b8ce82f48dbffe201ca3dfc64c35007.png)  
eureka  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e0f1db6debadc1774587301185a668dd.png)

#### 4.2.4 验证

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/32cdfa5098cbb0d63dcf0f6851d3a594.png)  
注意，这里从github读取可能存在超时情况。  
可以配置超时时间：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88bce95f486d31b04b77bacf267823e8.png)  
单位是秒  
ctrl+左键点击timeout  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a16211f848d80284c52c838487b4f8f8.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf94824b19061837114517a3893ee89e.png)  
没有timeout吗，应该是在父类中吧  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/625ee4db847fd4cf778c25180564eedc.png)  
找到了，看其默认是5秒钟，对于国内环境访问github，嗯嗯嗯，5秒钟还是有点勉强的，直接设置为60秒。

### 4.3 config client& eureka

#### 4.3.1 创建springbootconfigeurekaclient

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6cde1f0433ef193365a3013912d55f3f.png)

#### 4.3.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0198d861019f2ba648a32ee4b708031.png)  
这里一定遵循一个规则：${spring.application.name}-{spring.profiles.active}就是读取的配置文件的名字。

#### 4.3.3 启动验证端口

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3471cdf0efd632d97053cd4a88124e00.png)  
这个端口是我们在远程配置的端口。

#### 4.3.4 验证负载均衡

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ead39ece29bc9b992e6541f798419404.png)  
从日志看，其访问的是8011的config server实例  
而且日志中也打印了配置信息来源  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/38cc693112cfd119edab30bcaa3791d7.png)  
关闭configeurekaclient在重新启动，看会不会负载均衡。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d16b9ce447c3d674339b658dd7d88c1.png)  
成功负载均衡。

#### 4.3.5 验证message

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8e6c3a1161229356fde55fb82ac75d44.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2f60b03e556576aaf8bfed550a120b5c.png)  
成功访问，并且日志也打印了此次请求。  
最后看下所有的服务  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/39a0317b998e81ff13f268b4732cb762.png)