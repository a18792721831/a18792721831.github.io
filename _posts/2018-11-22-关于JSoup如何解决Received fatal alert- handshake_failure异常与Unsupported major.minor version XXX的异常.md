---
layout: post
title: "关于JSoup如何解决Received fatal alert: handshake_failure异常与Unsupported major.minor version XXX的异常"
date: 2018-11-22 17:20:43 +0800
categories: ["JSoup异常", "Received fatal alert: handshake_fai", "Unsupported major.minor version 52.", "jdk版本", "切换库文件"]
description: "本文详细解析了HTTPS握手异常（Received fatal alert: handshake_failure）与Unsupported major.minor version错误的原因，主要由JDK版本过低或编译与运行JDK版本不一致引起。提供了升级JDK版本及调整Eclipse项目配置的解决方案。"
keywords: ["JSoup异常", "Received fatal alert: handshake_fai", "Unsupported major.minor version 52.", "jdk版本", "切换库文件"]
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/84345022
> - 发布时间：2018-11-22 17:20:43
> - 阅读量：2
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#JSoup异常, #Received fatal alert: handshake_fai, #Unsupported major.minor version 52., #jdk版本, #切换库文件

## 摘要

文章浏览阅读2.4k次。本文详细解析了HTTPS握手异常（Received fatal alert: handshake_failure）与Unsupported major.minor version错误的原因，主要由JDK版本过低或编译与运行JDK版本不一致引起。提供了升级JDK版本及调整Eclipse项目配置的解决方案。

---

#### https握手异常与jdk版本异常

  * [1.Received fatal alert: handshake_failure](<#1Received_fatal_alert_handshake_failure_1>)
  * [2.Unsupported major.minor version XXX](<#2Unsupported_majorminor_version_XXX_44>)

## 1.Received fatal alert: handshake_failure

描述：
    
    
    javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
    	at com.sun.net.ssl.internal.ssl.Alerts.getSSLException(Alerts.java:174)
    	at com.sun.net.ssl.internal.ssl.Alerts.getSSLException(Alerts.java:136)
    	at com.sun.net.ssl.internal.ssl.SSLSocketImpl.recvAlert(SSLSocketImpl.java:1837)
    	at com.sun.net.ssl.internal.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:1019)
    	at com.sun.net.ssl.internal.ssl.SSLSocketImpl.performInitialHandshake(SSLSocketImpl.java:1203)
    	at com.sun.net.ssl.internal.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1230)
    	at com.sun.net.ssl.internal.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1214)
    	at sun.net.www.protocol.https.HttpsClient.afterConnect(HttpsClient.java:434)
    	at sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.connect(AbstractDelegateHttpsURLConnection.java:166)
    	at sun.net.www.protocol.https.HttpsURLConnectionImpl.connect(HttpsURLConnectionImpl.java:133)
    	at org.jsoup.helper.HttpConnection$Response.execute(HttpConnection.java:449)
    	at org.jsoup.helper.HttpConnection$Response.execute(HttpConnection.java:434)
    	at org.jsoup.helper.HttpConnection.execute(HttpConnection.java:181)
    	at org.jsoup.helper.HttpConnection.get(HttpConnection.java:170)
    

原因：  
jdk的版本太低，或者jvm的版本太低，只能发送ssl1的请求，但是响应的是ssl2或者更高版本的响应，导致ssl握手失败。  
解决方案：  
增加jdk的版本（最简单），如果非要用低版本，网上也有解决的方案。  
步骤：（eclipse）  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f1be7298f6d4aef46a5421395e220e70.png)  
右键->build path->configure build path  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2fdcf9f3a9bb809c135c4ed912b18916.png)  
Add Library  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/59136e3afa69634e356c211bf25911e2.png)  
next  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8abb639c9c930d73e4904a0bca6768bb.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/140e73d37d7d4f77a124ed32a39945fd.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4391b17962693574140773543f15ced9.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d052f724c3034d6c14b9e62ef654d267.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2af968fa036461f93a8afdddd06bd4e5.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c1e57e374c77c66d0ea9106194f18bba.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/329ee0ea28492c0084d173ea00e2f03f.png)  
或者不用选择：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/05bcf1acb074bf7a267848ba31069f20.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/90fdf530302f994cb2bf5d3c0fd5a292.png)  
不删除也可以  
但有可能引起混乱  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/551ba2ffa24772a0c87e18a0bf928691.png)

## 2.Unsupported major.minor version XXX

原因：编译jdk版本与运行的jdk版本不一致  
右键->build path->configure build path  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/12188c3525eb1b6b433787972b2876c7.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c9aabbb7ddaa6f725a850fdae317789.png)  
然后一直确定就可以，最后会重新编译项目。