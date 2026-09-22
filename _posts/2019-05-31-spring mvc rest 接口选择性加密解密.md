---
layout: post
title: "spring mvc rest 接口选择性加密解密"
date: 2019-05-31 19:08:21 +0800
categories: [springMVC, 加密解密, 注解与拦截器实现加密解密, 加密解密算法, 加密解密算法比较]
description: "博客围绕Spring MVC部分接口加密、返回值解密需求展开。分析了注解和拦截器两种实现方式，推荐注解方式。介绍了对称和非对称加密应用场景，详细阐述MD5、SHA1等多种加密算法，并对散列、对称、非对称加密算法进行比较。"
keywords: springMVC, 加密解密, 注解与拦截器实现加密解密, 加密解密算法, 加密解密算法比较
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/90721634
> - 发布时间：2019-05-31 19:08:21
> - 阅读量：1
> - 分类：java同时被 2 个专栏收录, 订阅专栏, springMVC
> - 标签：#springMVC, #加密解密, #注解与拦截器实现加密解密, #加密解密算法, #加密解密算法比较

## 摘要

文章浏览阅读1.6k次。博客围绕Spring MVC部分接口加密、返回值解密需求展开。分析了注解和拦截器两种实现方式，推荐注解方式。介绍了对称和非对称加密应用场景，详细阐述MD5、SHA1等多种加密算法，并对散列、对称、非对称加密算法进行比较。

---

#### spring mvc rest 接口选择性加密解密

  * 1.需求
  * 2.分析
  * 3.实现
  *     * 3.1注解方式
    *       * 3.1.1定义注解
      * 3.1.2定义注解Aspect切面
      * 3.1.3使用
    * 3.2拦截器
    *       * 3.2.1定义拦截器
      * 3.2.2配置拦截器
      * 3.2.3使用
  * 4.加密
  *     * 4.1对称加密
    * 4.2非对称加密
  * 5.加密算法
  *     * 5.1MD5算法
    * 5.2SHA1算法
    * 5.3HMAC算法
    * 5.4AES/DES/3DES算法
    * 5.5DES算法
    * 5.63DES算法
    * 5.7AES算法
    * 5.8RSA算法
    * 5.9ECC算法
  * 6.加密算法比较
  *     * 6.1散列算法
    * 6.2对称加密算法
    * 6.3非对称加密算法比较

## 1.需求

spring mvc rest接口以前是采用https加密的，但是现在需要更加安全的加密。  
而且不是对所有的接口进行加密，是对部分接口进行加密，接口返回值进行解密。

## 2.分析

实现方式有两种：  
1.Aspect + Annotation  
2.interceptor + requestParameter  
第一种方式是最灵活的：  
自定义注解，然后在Aspect中对注解的方法进行处理。  
第二种方法也能实现：  
自定义拦截器加请求参数与返回参数，即参数中有一个参数控制是否加密解密。

第二种方式很明显参数冗余，管理不变，使用麻烦。  
第一种参数就很好了，扩展容易，使用容易，提倡使用。

## 3.实现

### 3.1注解方式

#### 3.1.1定义注解
    
    
    package com.annotation;
    
    import java.lang.annotation.*;
    
    @Target(ElementType.METHOD)
    //使用在方法级别上
    @Retention(RetentionPolicy.RUNTIME)
    //运行时有效
    @Documented
    //生成文档
    public @interface Encryption {
    }
    
    

#### 3.1.2定义注解Aspect切面
    
    
    package com.aop;
    
    import org.aspectj.lang.JoinPoint;
    import org.aspectj.lang.annotation.After;
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Before;
    import org.aspectj.lang.annotation.Pointcut;
    import org.springframework.stereotype.Component;
    
    @Aspect
    //Aspect注解
    @Component
    //spring bean的自动注解
    //aop切面
    public class EncryptionAspect {
    
        /**
         * 加密切点
         */
        @Pointcut("@annotation(com.startimes.selfserviceApp.annotation.Encryption)")
        public void encryptionPointcut(){
        }
    
        /**
         * 前置通知--解密
         * @param joinPoint
         */
        @Before("encryptionPointcut()")
        public void doBefore(JoinPoint joinPoint){
            System.out.println("encryptionPointcut");
        }
    
        /**
         * 后置通知--加密
         * @param joinPoint
         */
        @After("encryptionPointcut()")
        public void doAfter(JoinPoint joinPoint){
            System.out.println("encryptionPointcutAfter");
        }
    }
    
    

#### 3.1.3使用

在方法前面加入注解：  
@Encryption

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/196ee9bdfd0164baf6fd88ad7780098c.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ebb8534eee60767f00d60b884916760b.png)

### 3.2拦截器

#### 3.2.1定义拦截器
    
    
    package com.interceptor;
    
    import org.springframework.ui.ModelMap;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.context.request.WebRequestInterceptor;
    
    public class EncryptionInterceptor implements WebRequestInterceptor  {
    
        @Override
        public void preHandle(WebRequest request) throws Exception {
            //TODO:自定义代码
            System.out.println("preHandle");
        }
    
        @Override
        public void postHandle(WebRequest request, ModelMap model) throws Exception {
            //TODO:自定义代码
            System.out.println("postHandle");
        }
    
        @Override
        public void afterCompletion(WebRequest request, Exception ex) throws Exception {
            //TODO:自定义代码
            System.out.println("afterCompletion");
        }
    }
    
    

#### 3.2.2配置拦截器

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/30ef865c0bcbf475c95c434f0291f907.png)

#### 3.2.3使用

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/db67770356d985c3ead62c0bfe08ea6c.png)  
不使用参数，如果使用参数就在拦截器里判断参数，然后进行相应的处理。

## 4.加密

### 4.1对称加密

对于无需加密的接口采用数字签名加密，即https  
对于需要加密的接口采用将MD5(session)作为密钥的对称加密  
缺点：需要服务器与客户端有相同的session

### 4.2非对称加密

对于无需加密的接口采用非对称加密，即https+双向认证+自定义证书  
对于需要加密的接口采用非对称加密。

## 5.加密算法

### 5.1MD5算法

MD5 用的是 哈希函数，它的典型应用是对一段信息产生 信息摘要，以 防止被篡改。严格来说，MD5 不是一种 加密算法 而是 摘要算法。无论是多长的输入，MD5 都会输出长度为 128bits 的一个串 (通常用 16 进制 表示为 32 个字符)。

### 5.2SHA1算法

SHA1 是和 MD5 一样流行的 消息摘要算法，然而 SHA1 比 MD5 的 安全性更强。对于长度小于 2 ^ 64 位的消息，SHA1 会产生一个 160 位的 消息摘要。基于 MD5、SHA1 的信息摘要特性以及 不可逆 (一般而言)，可以被应用在检查 文件完整性 以及 数字签名 等场景。

### 5.3HMAC算法

HMAC 是密钥相关的 哈希运算消息认证码（Hash-based Message Authentication Code），HMAC 运算利用 哈希算法 (MD5、SHA1 等)，以 一个密钥 和 一个消息 为输入，生成一个 消息摘要 作为 输出。  
HMAC 发送方 和 接收方 都有的 key 进行计算，而没有这把 key 的第三方，则是 无法计算 出正确的 散列值的，这样就可以 防止数据被篡改。

### 5.4AES/DES/3DES算法

AES、DES、3DES 都是 对称 的 块加密算法，加解密 的过程是 可逆的。常用的有 AES128、AES192、AES256 (默认安装的 JDK 尚不支持 AES256，需要安装对应的 jce 补丁进行升级 jce1.7，jce1.8)。

### 5.5DES算法

DES 加密算法是一种 分组密码，以 64 位为 分组对数据 加密，它的 密钥长度 是 56 位，加密解密 用 同一算法。  
DES 加密算法是对 密钥 进行保密，而 公开算法，包括加密和解密算法。这样，只有掌握了和发送方 相同密钥 的人才能解读由 DES加密算法加密的密文数据。因此，破译 DES 加密算法实际上就是 搜索密钥的编码。对于 56 位长度的 密钥 来说，如果用 穷举法 来进行搜索的话，其运算次数为 2 ^ 56 次。

### 5.63DES算法

是基于 DES 的 对称算法，对 一块数据 用 三个不同的密钥 进行 三次加密，强度更高。

### 5.7AES算法

AES 加密算法是密码学中的 高级加密标准，该加密算法采用 对称分组密码体制，密钥长度的最少支持为 128 位、 192 位、256 位，分组长度 128 位，算法应易于各种硬件和软件实现。这种加密算法是美国联邦政府采用的 区块加密标准。  
AES 本身就是为了取代 DES 的，AES 具有更好的 安全性、效率 和 灵活性。

### 5.8RSA算法

RSA 加密算法是目前最有影响力的 公钥加密算法，并且被普遍认为是目前 最优秀的公钥方案 之一。RSA 是第一个能同时用于 加密 和 数字签名 的算法，它能够 抵抗 到目前为止已知的 所有密码攻击，已被 ISO 推荐为公钥数据加密标准。  
RSA 加密算法 基于一个十分简单的数论事实：将两个大 素数 相乘十分容易，但想要对其乘积进行 因式分解 却极其困难，因此可以将 乘积 公开作为 加密密钥。

### 5.9ECC算法

ECC 也是一种 非对称加密算法，主要优势是在某些情况下，它比其他的方法使用 更小的密钥，比如 RSA 加密算法，提供 相当的或更高等级 的安全级别。不过一个缺点是 加密和解密操作 的实现比其他机制 时间长 (相比 RSA 算法，该算法对 CPU 消耗严重)。

## 6.加密算法比较

### 6.1散列算法

| 名称    | 安全性 | 速度 |
|-------|-----|----|
| SHA-1 | 高   | 慢  |
| MD5   | 中   | 快  |

### 6.2对称加密算法

| 名称   | 密钥长度        | 运行速度 | 安全性 | 资源消耗 |
|------|-------------|------|-----|------|
| DES  | 56          | 较快   | 低   | 中    |
| 3DES | 112、168     | 慢    | 中   | 高    |
| AES  | 128、192、256 | 快    | 高   | 低    |

### 6.3非对称加密算法比较

| 名称  | 成熟度 | 安全性 | 运算速度 | 资源消耗 |
|-----|-----|-----|------|------|
| RSA | 高   | 高   | 中    | 中    |
| ECC | 高   | 高   | 慢    | 高    |
