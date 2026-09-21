---
layout: post
title: "spring cloud OAuth2"
date: 2020-03-21 18:49:20 +0800
categories: [OAuth5.2.x学习, Spring Security, OAuth2.x与5.2.x, OAuth5.2.x那些改动, OAuth2]
description: "本文深入解析了OAuth2的必要性和原理，详细阐述了OAuth2在Spring Cloud中的实现，包括OAuth2Provider和OAuth2Client的角色划分，以及在Spring Security框架下的配置细节。"
keywords: OAuth5.2.x学习, Spring Security, OAuth2.x与5.2.x, OAuth5.2.x那些改动, OAuth2
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/105006679
> - 发布时间：2020-03-21 18:49:20
> - 阅读量：4
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#OAuth5.2.x学习, #Spring Security, #OAuth2.x与5.2.x, #OAuth5.2.x那些改动, #OAuth2

## 摘要

文章浏览阅读4.3k次，点赞2次，收藏11次。本文深入解析了OAuth2的必要性和原理，详细阐述了OAuth2在Spring Cloud中的实现，包括OAuth2Provider和OAuth2Client的角色划分，以及在Spring Security框架下的配置细节。

---

#### spring cloud OAuth2

  * 1\. 为什么需要oauth
  * 2\. oauth的原理
  * 3\. oauth的组成
  *     * 3.1 OAuth2 Provider
    *       * 3.1.1 授权服务Authorization Server
      *         * 3.1.1.1 ClientDetailServiceConfigurer
        * 3.1.1.2 AuthorizationServerEndpointsConfigurer
        * 3.1.1.3 AuthorizationServerSecurityConfigurer
      * 3.1.2 资源服务Resource Server
    * 3.2 OAuth2 Client
    *       * 3.2.1 Protected Resource Configuration
      * 3.2.2 Client Configuration
  * 4\. 实例

  
git地址   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. 为什么需要oauth

在互联网上，不是所有的资源都是开放的。比如公司内部的机密资料，代码，方案等，都不是向所有人开放的，而是有选择性的开放。你是公司的员工，那么你就能够访问资源，如果你不是公司的员工，那么你就无法访问。  
那么，程序或者网站如何区分你是否是公司内部的员工呢？  
常见的做法有：

  * 公司不开放注册，然后分配公司内部员工使用用户名密码进行验证访问。
  * 公司指定用户才能访问，比如ip限定。

但是上述的做法如果是公司自己管理或者自己开发的网站或者程序，那么还凑活使用。  
如果公司使用的是第三方托管资源，那么就不可避免的需要将授权认证信息提供给第三方，这样第三方才能进行验证并允许访问资源。  
如果第三方受到攻击，那么会导致大量的信息泄露(记得前一段时间报道，docker hub受到攻击，导致许多授权信息泄露，github好像也发生过类似的信息，以及企业常见的代码管理git lab等等)

为了避免因第三方受到攻击，而拖累客户，就需要一个类似互联网网站证书的机制，通过认证获取授权(类似从CA获取证书)，然后用户拿着授权信息去第三方访问(一般第三方拥有打带宽，或者更好的资源访问体验)。当第三方受到请求后，将授权信息拿到授权认证服务器认证，通过后再允许访问。

通俗来讲：  
用户向第三方说，我要访问XXX资源，  
第三方一看是用户YY访问，  
接着去问公司，用户YY是你公司的员工吗，他有访问XXX资源的权限吗？  
公司回复，用户YY是我公司的员工，有访问XXX资源的权限。  
接着第三方说，用户YY有权限访问XXX资源，你去访问吧  
然后用户YY就拿到了想要访问的资源。

## 2\. oauth的原理

在1中，我们通俗的理解了oauth的整个流程机制，以及oauth的必要性。不会因第三方拖累公司(一般第三方需要提供的服务多，受到攻击的可能性大，而公司只指定第三方可以访问，受到的攻击几乎没有)  
接下来看下oauth的一些专有名词。  
oauth的工业标准：  
https://tools.ietf.org/html/rfc6749  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1a5225e9aa647787ab8dc7c5b19922c6.png)  
oauth是基于http授权的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0bdd31309f8c4a610699efba6c3ca9e8.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48647d2e07e728edb2c1bb57c7a85c55.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d3eb2a339e19062bb8024435a419c5a0.png)

## 3\. oauth的组成

OAuth2协议在Spring Resources中的实现为Spring OAuth2.Spring Oauth2分为两部分：Oauth2 Provider和OAuth2 Client.

### 3.1 OAuth2 Provider

OAuth2 Provider 负责公开被 OAuth2保护起来的资源。OAuth2 Provider需要配置代表用户的 OAuth2客户端信息，被用户允许的客户端就可以访问被OAuth2保护的资源。OAuth2 Provider 通过管理和验证 OAuth2令牌来控制客户端是否有权限访问被其保护的资源。另外，Oauth 2 Provider 还必须为用户提供认证 API接口。根据认证 API 接口，用户提供账号和密码等信息，来确认客户端是否可以被 OAuth2 Provider 授权。这样做的好处就是第三方客户端不需要获取用户的账号和密码，通过授权的方式就可以访问被OAuth2保护起来的资源。  
OAuth2 Provider的角色被分为Authorization Service（授权服务）和Resource Service（资源服务），通常它们不在同一个服务中，可能一个Authorization Service对应多个 Resource Service。Spring OAuth2 需配合 Spring Security 一起使用，所有的请求由Spring MVC控制器处理，并经过一系列的Spring Security过滤器。  
在Spring Security过滤器链中有以下两个节点，这两个节点是向Authorization Service 获取验证和授权的。

  * 授权节点：默认为/oauth/auhorize。
  * 获取 Token节点：默认为/oauth/token。

#### 3.1.1 授权服务Authorization Server

在配置Authorization Server时，需要考虑客户端从用户获取访问令牌的授权类型。Authorization Server需要配置客户端的详细信息和令牌服务实现。  
在任何实现了AuthorizationServerConfigurer接口的类上加@EnableAuthorizationServer注解，开启Authorization Server的功能，以Bean的形式注入Spring IoC容器中，并需要实现以下3个配置：

其整体实现如图：  
https://naotu.baidu.com/file/1465ee54a758ba989953f8518f028172?token=14b06e4991ef8660  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0229cc35acd43d55b379f57e55efce9e.png)

  *     1. ClientDetailsServiceConfigurer:配置客户端信息
  *     2. AuthorizationServerEndpointsConfigurer:配置授权的Token节点和服务
  *     3. AuthorizationServerSecurityConfigurer:配置Token节点的安全策略

##### 3.1.1.1 ClientDetailServiceConfigurer

客户端的配置信息既可以放在内存中，也可以放在数据库中。  
需要配置以下信息：

  * clientId:客户端id,需要在Authorization Server中是唯一的
  * secret:客户端的密码
  * scope:客户端的域
  * authorizedGrantTypes:认证类型
  * authorities:权限信息

##### 3.1.1.2 AuthorizationServerEndpointsConfigurer

在默认情况下，AuthorizationServerEndpointsConfigurer配置开启了所有的验证类型，除了密码类型的验证，密码配置只有配置了authrizationManager的配置才会开启。  
AuthorizationServerEndpointsConfigurer的配置由以下组成：

  * authorizationManager:只有配置了这个，密码认证才会开启。
  * userDetailsService:配置获取用户认证信息的接口。
  * authorizationCodeServices:配置验证码服务。
  * implicaitGranter:配置管理implict验证的状态。
  * tokenGranter:配置Token Granter  
Token的策略现在有以下三种：

  1. InMemoryTokenStore:Token存储在内存中
  2. JdbcTokenStore:Token存储在数据库中
  3. JwtTonkenStore:采用JWT形式

##### 3.1.1.3 AuthorizationServerSecurityConfigurer

如果资源服务和授权服务是在同一个服务中，用默认的配置即可，不需要做其他任何的配置。但是如果资源服务和授权服务不在同一个服务中，则需要做一些额外配置。如果采用RemoteTokenService(远程Token校验)，资源服务器的每次请求所携带的Token都需要从授权服务器做校验。此时需要配置"/oauth/check_token"校验节点的校验策略。

#### 3.1.2 资源服务Resource Server

Resource Server提供了收OAuth2保护的资源，可以是API接口、HTML页面、js文件等。Spring OAuth2提供了实现此类保护功能的Spring Security认证过滤器。在加了@Configuration的注解的配置类上加@EnableResourceServer注解，开启Resource Server的功能。

  1. tokenServices:定义Token Service.比如可以配置ResourceServerTokenServices类，配置Token是如何编码解码。如果Resource Server和Authorization Server在同一个工程上，就不需要配置tokenServices。也可以使用RemoteTokenService类，采用远程授权方式，这个时候也不需要tokenService.
  2. resourceId:配置资源id

### 3.2 OAuth2 Client

OAuth2 Client（客户端）用于访问被OAuth2保护起来的资源。客户端需要提供用于存储用户的授权码和访问令牌的机制，需要配置如下两个选项。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b0850842715fcede2dc821e0aecc94d.png)  
http://naotu.baidu.com/file/93e43f7235046ba93909e83fb992d034?token=482eebbed3e161ba

#### 3.2.1 Protected Resource Configuration

使用OAuth2ProtectedResourceDeatils类型的Bean来定义受保护的资源，受保护的资源具有以下属性：

  * Id:资源的Id,在Spring OAuth2中没有用到，用于客户端寻找资源，不需要做配置，默认即可。
  * clientId:OAuth2 Client的Id，和之前OAuth2 Provider中配置的一一对应。
  * clientSecret:客户端密码，和之前OAuth2 Provider中配置的一一对应。
  * accessTokenUri:获取Token的API节点
  * scope:客户端的域
  * clientAuthorizationSchema:有两种客户端验证类型，分别是Http Basic,和Form,默认是Http Basic.
  * userAuthorizationUri:如果用户需要授权访问资源，则用户将被重定向到认证url.

#### 3.2.2 Client Configuration

对于 OAuth2 Client的配置，可以使用@EnableOAuth2Client注解来简化配置，当然还需要两个配置：

  * 创建一个过滤器Bean(Bean的id为oauth2ClientContextFilter),用来存储当前请求和上下文请求。在请求期间，如果用户需要进行身份认证，则用户会被重定向到OAuth2的认证url.
  * 在Reques域内创建AccessTokenRequest类型的Bean.

## 4\. 实例

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/767a720ce550187b99c69df41b3a2544.png)  
https://www.processon.com/view/link/5e75aa28e4b011fccea52457

原本打算按照教程，做一个实例，完完整整的练习一遍。  
但是吧，没想到。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c5c2d5115ab26d3f74ca54c4f18ea356.png)  
[传送门](<https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Migration-Guide>)  
教程是spring boot 1.x的，对应的OAuth2.x  
现在是spring boot 2.x的，对应的OAuth5.2.x  
作重要的是：  
OAuth5.2.x他们做的改动太大了。  
5.2.x做大的改动就是将OAuth2.x中的注解废弃了，且不兼容。  
那么在5.2.x中是如何实现相同的配置呢？  
他们使用了DSL。说白了就是装饰器配置。。。  
在5.2.x中，将所有的配置全部放在了WebSecurityConfigurerAdapter中  
也就是说不管你想怎么配置，全部都在一个类中。  
嗯，有些无法适应。  
比如：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dbfd2d3fcb2e66812b2303ecb2a65ab0.png)  
去掉了@EnableOAuth2Client的注解，替换为了oauth2Client配置  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a8018d4677c7fea16033470386c120fa.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/09437b68e056b48166f8b4d86d018e6a.png)  
废弃了@EnableOAuth2Sso注解，替换为了oauth2Login()配置  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0383c998aa12a9783b9a416d908c9162.png)  
废弃了@EnableResourceServer注解，而是使用oauth2ResourceServer配置  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f3977b7e8c4599ece226694baca05398.png)  
废弃了ResourceServerConfigurerAdapter适配器类，将其融入了WebSecurityConfigurerAdapter类。  
好多好多，而且开放的issues也很多。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d460f8facb8b47e73ed7fda057305d5.png)

怎么说呢，不管怎么改都可以，但是最好还是不要动一些常用的配置。  
比如他废弃了好多的注解，然后都放在了WebSecurityConfigurerAdapter中。  
这些方法的配置也没有很好的注释说明：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dd3ecbf5a0546924c8d693eec8e4907e.png)  
从这里以下一半的注释都很少。  
就导致，这东西现在不知道该怎么用，网上的资料也是少的可怜。至少你19年11月之前的资料都是无法用的(可能能用一点点)。  
我到现在大概看源码、找资料花费了大概6个小时，依然没有成功。  
这是我找到一些可能有用的信息：

  1. oauth2的文档地址：https://oauth.net/2/
  2. 迁移说明：https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Migration-Guide
  3. spring官方文档说明：https://spring.io/projects/spring-security-oauth
  4. git的readme文档：https://github.com/spring-projects/spring-security
  5. oauth2resourceserver-opaque例子:https://github.com/spring-projects/spring-security/tree/master/samples/boot/oauth2resourceserver-opaque
  6. oauth2webclient例子:https://github.com/spring-projects/spring-security/blob/5.3.0.RELEASE/samples/boot/oauth2webclient/src/main/java/sample/config/SecurityConfig.java

这部分先存疑吧，等过段时间，资料丰富了在回来补充。
