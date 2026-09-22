---
layout: post
title: "spring boot配置https加密以及http升级2.0"
date: 2021-01-13 18:58:26 +0800
categories: [微服务安全加密, https加密配置, https双向认证加密, 微服务http1.1升级2.0, 加密证书生成与使用]
description: "spring boot配置https加密以及http升级2.0https加密http升级2.0https加密在spring boot项目中配置微服务加密非常简单。首先需要生成证书，如果是单项加密就只需要一个证书，如果是双向加密就需要至少两个证书。数字证书的生成请看tomcat实现https双向认证配置生成证书后，将证书放到resources目录下然后在application.properties或者application.yaml中配置证书如果是单项认证，那么将client-auth设置_springboot密文传输升级"
keywords: 微服务安全加密, https加密配置, https双向认证加密, 微服务http1.1升级2.0, 加密证书生成与使用
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/112582259
> - 发布时间：2021-01-13 18:58:26
> - 阅读量：604
> - 分类：微服务专栏收录该内容, 订阅专栏
> - 标签：#微服务安全加密, #https加密配置, #https双向认证加密, #微服务http1.1升级2.0, #加密证书生成与使用

## 摘要

文章浏览阅读604次。spring boot配置https加密以及http升级2.0https加密http升级2.0https加密在spring boot项目中配置微服务加密非常简单。首先需要生成证书，如果是单项加密就只需要一个证书，如果是双向加密就需要至少两个证书。数字证书的生成请看tomcat实现https双向认证配置生成证书后，将证书放到resources目录下然后在application.properties或者application.yaml中配置证书如果是单项认证，那么将client-auth设置_springboot密文传输升级

---

#### spring boot配置https加密以及http升级2.0

  * https加密
  * http升级2.0

## https加密

在spring boot项目中配置微服务加密非常简单。  
首先需要生成证书，如果是单项加密就只需要一个证书，如果是双向加密就需要至少两个证书。  
数字证书的生成请看  
[tomcat实现https双向认证配置](<https://blog.csdn.net/a18792721831/article/details/83625643>)  
生成证书后，将证书放到resources目录下  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fb46a391107c499582c1686bbe2cfc61.png)  
然后在application.properties或者application.yaml中配置证书  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9cbbf63ced964dfba9c92d6cf22cdaab.png)  
如果是单项认证，那么将`client-auth`设置成其他的，而`trust`开头的就不需要了。  
如果单项认证，客户端什么都不需要做。  
如果是双向认证，如果没有证书，就会提示  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0d3de08393bbea7e68363185f1c91a6e.png)  
我们将客户端的JKS证书转为PKCS12证书，就可以安装客户端证书(服务端证书安装只需要安装cer文件即可，客户端证书安装需要安装p12文件)。其他的证书格式都可以，只要保证客户端证书安装后，带有私钥就行。  
按下win+R，输入certmgr.msc，打开证书管理界面(在浏览器的设置->安全->证书管理也能打开)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/823b8d796cc923eb9f3670c067c78b4b.png)  
安装完证书，刷新浏览器，此时浏览器就会弹框提示选择证书用于身份验证  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/623c3bf089c78e77406b2f03e6105485.png)  
我们点击确定，就能使用双向认证的方式访问了  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5f8d00d44a38da8394b752f10cc846f0.png)

## http升级2.0

如何查看自己现在使用的http版本呢？  
我们首先使用浏览器的开发者工具(谷歌浏览器、edge是f12)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bfa792a3a5900f50590db71cbfbfe2bd.png)  
接着刷新界面，此时在开发者工具中会展示有请求发出  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/26d81a1e1d09cd95590bf8bd389de3b7.png)  
接着我们随机选择一个请求，右键  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/04afb09b592d89d537ab0207a47a534a.png)  
然后打开文本编辑器，粘贴，找到我们的请求  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/52821bdfa42b8453b460a12d0eb745b9.png)  
里面的httpVersion很清楚的标明了是HTTP1.1版本。  
升级HTTP2.0也很容易  
在微服务的application.properties和application.yaml中增加配置  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0e4d1350cf827f28fed0bac1b2594b98.png)  
然后重启微服务并访问，然后拷贝数据到文本编辑器中找到请求  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/17b40922d5c90f36d12e7f8703d54860.png)  
这样就升级为了http2.0了
