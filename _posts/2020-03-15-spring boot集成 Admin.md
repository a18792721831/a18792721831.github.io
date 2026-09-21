---
layout: post
title: "spring boot集成 Admin"
date: 2020-03-15 20:32:38 +0800
categories: [cloud admin, admin server, admin client, admin hystrix, admin user/pswd]
description: "本文详细介绍了SpringBoot Admin的集成过程，包括Admin Server与Client的配置，以及如何集成Turbine和Security。从创建项目到配置、日志模板、注解和启动验证，提供了全面的指导。特别关注了安全性的配置，如用户认证和授权。"
keywords: cloud admin, admin server, admin client, admin hystrix, admin user/pswd
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104862831
> - 发布时间：2020-03-15 20:32:38
> - 阅读量：632
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#cloud admin, #admin server, #admin client, #admin hystrix, #admin user/pswd

## 摘要

文章浏览阅读632次。本文详细介绍了SpringBoot Admin的集成过程，包括Admin Server与Client的配置，以及如何集成Turbine和Security。从创建项目到配置、日志模板、注解和启动验证，提供了全面的指导。特别关注了安全性的配置，如用户认证和授权。

---

#### spring boot集成 Admin

  * [1\. spring boot admin](<#1_spring_boot_admin_3>)
  *     * [1.1 spring boot admin server](<#11_spring_boot_admin_server_4>)
    *       * [1.1.1 创建](<#111__5>)
      * [1.1.2 配置](<#112__7>)
      * [1.1.3 日志模板](<#113__10>)
      * [1.1.4 注解](<#114__12>)
      * [1.1.5 启动](<#115__14>)
    * [1.2 spring boot admin client](<#12_spring_boot_admin_client_44>)
    *       * [1.2.1 创建](<#121__45>)
      * [1.2.2 配置](<#122__47>)
      * [1.2.3 日志模板](<#123__49>)
      * [1.2.4 注解](<#124__51>)
      * [1.2.5 启动](<#125__53>)
  * [2\. spring boot admin 集成 turbine(admin 2.x不支持,未实现)](<#2_spring_boot_admin__turbineadmin_2x_56>)
  *     * [2.1 spring boot admin client hystrix](<#21_spring_boot_admin_client_hystrix_57>)
    *       * [2.1.1 创建](<#211__58>)
      * [2.1.2 配置](<#212__60>)
      * [2.1.3 日志模板](<#213__62>)
      * [2.1.4 注解](<#214__64>)
      * [2.1.5 controller](<#215_controller_66>)
      * [2.1.6 hystrix 配置](<#216_hystrix__68>)
      * [2.1.7 启动验证](<#217__71>)
    * [2.2 spring boot admin client service](<#22_spring_boot_admin_client_service_74>)
    *       * [2.2.1 创建](<#221__75>)
      * [2.2.2 配置](<#222__77>)
      * [2.2.3 日志模板](<#223__79>)
      * [2.2.4 注解](<#224__81>)
      * [2.2.5 controller](<#225_controller_83>)
      * [2.2.6 启动验证](<#226__85>)
    * [2.3 spring boot admin client turbine](<#23_spring_boot_admin_client_turbine_89>)
    *       * [2.3.1 创建](<#231__90>)
      * [2.3.2 配置](<#232__92>)
      * [2.3.3 日志模板](<#233__95>)
      * [2.3.4 注解](<#234__96>)
      * [2.3.5 启动验证](<#235__98>)
    * [2.4 spring boot admin server turbine](<#24_spring_boot_admin_server_turbine_102>)
  * [3\. spring boot admin 集成 security](<#3_spring_boot_admin__security_105>)
  *     * [3.1 创建](<#31__106>)
    * [3.2 配置](<#32__108>)
    * [3.3 配置类](<#33__166>)
    * [3.4 日志模板](<#34__205>)
    * [3.5 注解](<#35__206>)
    * [3.6 启动验证](<#36__207>)

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. spring boot admin

### 1.1 spring boot admin server

#### 1.1.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18722056f6220a38ad8ff098d1f47b84.png)

#### 1.1.2 配置

详细配置见 https://codecentric.github.io/spring-boot-admin/2.2.1/  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cabc31f274476b537344fc013a38f323.png)

#### 1.1.3 日志模板

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c9a233cff53e9b917979ad4a5109139.png)

#### 1.1.4 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc8ae1070d16b1b22fc6220fa45368c8.png)

#### 1.1.5 启动

启动eureka server以及admin-server  
然后访问admin-server的端口  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7b445dd31ba236f89aa8fbac49e5411.png)  
admin-clinet是接下来需要创建的。  
选中进去还有更加详细的信息  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c1f9d977d058ee151e8edc3931750e7.png)  
同时在项目根路径下生成了日志文件  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c4c906ee2d4ecdafc3b2ef92dca79309.png)  
当然，在界面上也能看日志  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1828543ed94616bc84cb4167975cdc52.png)  
还可以给不同的日志设置不同的颜色  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cc14b9f5f1625a2e8d318bd03f0ee90d.png)  
因为admin组件集成了actuator，所以，在actuator里面的信息admin都能展示。  
很强大。  
展示线程活动  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f109978105cfbc7b2ee68e546647a013.png)  
黄色是wait  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7dc814784bbe76c35240127251d3893c.png)  
绿色是run  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d9f9b0a474593d9095aa330333b38d06.png)  
甚至子线程也能展示  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d90cb52489ed32646c44ecbe02e4f144.png)  
映射等  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a1202dbdd77fbd91dfa7aaf13ba653e.png)  
没有集成缓存组件，所以没有缓存。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0f9e6d28230e8537f6d006ead3570533.png)  
关键性日志等  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/24c430532629236c5f05e90dce39da1e.png)  
话说admin的展示很强大，也很美观，但是感觉有些重，太细了，对于生产一般好几年，这个就有些不太适用了。

### 1.2 spring boot admin client

#### 1.2.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e102dda0505af040a8b6ecf5d910b1a1.png)

#### 1.2.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9b6e912ab0442215a919e97a2eb6919f.png)

#### 1.2.3 日志模板

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc8ae1070d16b1b22fc6220fa45368c8.png)

#### 1.2.4 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f01a0e6f3ff5a53198106e277f67961.png)

#### 1.2.5 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1247e219c324eb3153b5c77e9a253a7d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cc0ad35941610824612044d0e93331a5.png)

## 2\. spring boot admin 集成 turbine(admin 2.x不支持,未实现)

### 2.1 spring boot admin client hystrix

#### 2.1.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d262a15d72d1f3690fb98a468682425a.png)

#### 2.1.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b7882eeca2ac1427823abf024f36440c.png)

#### 2.1.3 日志模板

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/86c5cfa02dd5e930da92ede0fa6c1435.png)

#### 2.1.4 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f6178996873025f816eaaa567fd5f1bc.png)

#### 2.1.5 controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/664f015533a767004074e8f0b795dcaf.png)

#### 2.1.6 hystrix 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e58e0610e41061eea0f40da8a6ee4d29.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3307b12d6df95cb997adf2699054081d.png)

#### 2.1.7 启动验证

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a1f3b50df766d429f74a33a8c421652b.png)  
成功熔断。

### 2.2 spring boot admin client service

#### 2.2.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e82103cd4434d0f5dd39f31d62f702e.png)

#### 2.2.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb7ff75dd61f82f7d1f03965ecbd6b34.png)

#### 2.2.3 日志模板

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a541fd0769298661e46e7ecacc7086fc.png)

#### 2.2.4 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0b8d2ac63fa462bf5487a02c8a80fe4.png)

#### 2.2.5 controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d337ca92dbab408feaad7640606a0fec.png)

#### 2.2.6 启动验证

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1acb53fcdde7dcc37fede027c8d92c9c.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3e63e609c8fe8c6f51c5018d6361bf05.png)  
熔断器正确访问。

### 2.3 spring boot admin client turbine

#### 2.3.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ecf2c3fea00fd6fde0307a6213b47bec.png)

#### 2.3.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8eb2b8df3c629fc9d9173023c097b799.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/feac007bbc9844d268dcbc05d27dadaa.png)

#### 2.3.3 日志模板

#### 2.3.4 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f02b9898cddf2043f2e28ba30372f28b.png)

#### 2.3.5 启动验证

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f4309e85d4944f9013f7633319aa3f00.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9242f6c1444af08327e9438d045ace5b.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ef8f44aa260f77fc158e670730e68323.png)

### 2.4 spring boot admin server turbine

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/014972c0c0b824da5c3ace55100b0a88.png)  
擦，看的资料太老旧了，spring cloud admin 2.x删除了这部分。。。。

## 3\. spring boot admin 集成 security

### 3.1 创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eae9abf397f8a128c3f39c5d06ed7880.png)

### 3.2 配置
    
    
    server:
      port: 8321
    
    eureka:
      client:
    #    register-with-eureka: false
    #    fetch-registry: false
        service-url:
          defaultZone: http://127.0.0.1:8761/eureka/
      instance:
        metadata-map:
          user.name: ${admin.login.user.name}
          user.password: ${admin.login.user.pswd}
    
    
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
      file: logs/spring-boot-admin-server.log
    
    spring:
      freemarker:
        template-loader-path: classpath:/templates/
        prefer-file-system-access: false
      application:
        name: admin-server
      boot:
        admin:
          client:
            username: ${admin.login.user.name}
            password: ${admin.login.user.pswd}
            instance:
              metadata:
                user.name: ${admin.login.user.name}
                user.password: ${admin.login.user.pswd}
    management:
      endpoints:
        web:
          exposure:
            include: '*'
    
    
    
    admin.login.user.name=admin
    admin.login.user.pswd=123456
    admin.login.user.role=ADMIN
    

这里有一个诡异的问题：密码必须是password，配置其他的密码，在最终登录时，都是无效用户名密码。  
基于此，尝试修改用户名，发现在其他条件不变的条件下，修改用户名是可以登录的。  
修改密码则导致无法登录。

### 3.3 配置类

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4cc1f442bc64548a824b475d0a0a9695.png)  
35行设置默认的url是`/`  
38设置所有的都需要凭证  
42行设置登录界面  
43行设置登出界面  
46行设置使用httpBasic(通过配置，应该可以实现其他的)  
45、46行设置禁止csrf(不知道这个scrf是个啥)  
47行设置保存用户名密码的选项

比较坑的是52行  
首先给大家看下官网给出的例子：  
https://codecentric.github.io/spring-boot-admin/2.2.1/#_securing_spring_boot_admin_server  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8a9774cfc8ec0bc7a0219d4ae1cde098.png)  
对52行完全没有任何说明，给的例子里面是这样的：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4dd16e5394654d0258df0cfe3ba270fb.png)  
发现没有，他的用户名和密码以及角色，都是在代码中硬编码了。  
所以，对于官网给出的例子，我个人感觉这里硬编码不太好。  
使用硬编码就必须使用user的用户名和password的密码。  
(这个之前没想到官网实例还是这么坑，且没啥说明)  
经过不断的测试，决定使用配置。
    
    
    admin.login.user.name=admin
    admin.login.user.pswd=123456
    admin.login.user.role=ADMIN
    

然后在config里面读取配置，这样至少不是硬编码了。  
还有一个疑点：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/086562a212c22f132da30fe20f69e91b.png)  
这个必须这样写，当然还有其他的几种可以选择。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a118baa612ad340e68f9920788bda0d1.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3450730eb0095c64551779bea7c53414.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dbf72eb2b6437a3ae3b92ba03c1d2871.png)  
有故事，不单纯。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54af1fdd83c4f19e94a80cf9889697b9.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5a892522c2a2631d02b75407fdf61cff.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a71a462e46692a48362729e8e236c724.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/30a52e56ab89e37cbbf3cd4e5432ce8a.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a2418de09282b03a1baf05d9573a6b1a.png)  
所以，这个noop还真不是乱写的。  
看这些类的名称以及操作，应该是与密码加密存储使用有关的。  
仔细看看里面的操作，noop差不多就是不加密(无语。。。。)  
当然，其他的我未亲自验证，这里只是一个猜测。

### 3.4 日志模板

### 3.5 注解

### 3.6 启动验证

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b65825f801f67152a739cf4e1635e2cd.png)  
这个记住用户就是在配置类里面最后一行加上去的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a6b30661baf7bc3eb3755cc3c2f02a51.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7dafcce2d210f189341fc6b07b43abf0.png)