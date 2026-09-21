---
layout: post
title: "tomcat实现https双向认证配置"
date: 2018-11-22 18:39:51 +0800
categories: [tomcat设置, htpps和http, https双向认证, ca根证书签名, 浏览器服务器互相认证]
description: "本文详述了在Tomcat中实现HTTPS双向认证的全过程，包括证书生成、转换、签名及安装步骤，以及如何配置Tomcat进行HTTPS加密，确保客户端与服务器之间的通信安全。"
keywords: tomcat设置, htpps和http, https双向认证, ca根证书签名, 浏览器服务器互相认证
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/83625643
> - 发布时间：2018-11-22 18:39:51
> - 阅读量：7
> - 分类：java同时被 2 个专栏收录, 订阅专栏, 网站
> - 标签：#tomcat设置, #htpps和http, #https双向认证, #ca根证书签名, #浏览器服务器互相认证

## 摘要

文章浏览阅读7.1k次，点赞3次，收藏25次。本文详述了在Tomcat中实现HTTPS双向认证的全过程，包括证书生成、转换、签名及安装步骤，以及如何配置Tomcat进行HTTPS加密，确保客户端与服务器之间的通信安全。

---

#### Tomcat实现https双向认证配置

  * [1.生成证书库](<#1_5>)
  * [2.jks转p12](<#2jksp12_31>)
  * [3.证书库导出cer文件](<#3cer_45>)
  * [4.证书库生成证书请求](<#4_60>)
  * [5.对证书请求进行签名](<#5_66>)
  * [6.例子](<#6_75>)
  *     * [6.1创建证书库](<#61_76>)
    * [6.2导出根证书](<#62_85>)
    * [6.3证书库发起请求](<#63_90>)
    * [6.4跟证书库对请求签名](<#64_97>)
    * [6.5根证书添加信任](<#65_104>)
    * [6.6签名文件添加信任](<#66_111>)
    * [6.7证书库导出cer](<#67cer_118>)
    * [6.8 cer生成临时证书库](<#68_cer_125>)
    * [6.9从临时证书库导出p12](<#69p12_132>)
  * [7.证书安装](<#7_139>)
  * [8.客户端服务端增加相互信任](<#8_182>)
  * [9.tomcat设置](<#9tomcat_191>)
  * [填坑了](<#_236>)
  * [1. 目的](<#1__237>)
  * [2. 加密](<#2__242>)
  *     * [2.1 对称加密](<#21__244>)
    * [2.2 非对称加密](<#22__253>)
    * [2.3 非对称加密的优点](<#23__269>)
  * [3. tomcat https单项加密](<#3_tomcat_https_308>)
  *     * [3.1 数字证书](<#31__309>)
    * [3.2 数字证书的生成](<#32__318>)
    * [3.3 tomcat配置](<#33_tomcat_339>)
  * [4. 证书信任](<#4__375>)
  *     * [4.1 http/https请求过程](<#41_httphttps_402>)
    * [4.2 DNS劫持](<#42_DNS_424>)
    *       * [4.2.1 hosts](<#421_hosts_441>)
      * [4.2.2 本地DNS](<#422_DNS_464>)
      * [4.2.3 远程DNS](<#423_DNS_478>)
      * [4.2.4 跟域名服务器](<#424__486>)
    * [4.3 根证书](<#43__491>)
    *       * [4.3.1 创建签名证书](<#431__503>)
      * [4.3.2 签名请求](<#432__544>)
      * [4.3.3 签名回复](<#433__577>)
      * [4.3.4 签名回复导入](<#434__592>)
  * [5. tomcat https加密(证书安全)](<#5_tomcat_https_632>)
  * [6. tomcat https双向加密(证书安全)](<#6_tomcat_https_644>)
  * [7. tomcat https 双向加密-多证书(证书安全)](<#7_tomcat_https__707>)

  
注意：本文偏重于实际操作，理论部分请自行掌握。   
注意：所有的代码不可换行，在命令提示符中一行完成。   
环境：window 7  
注意：jdk版本–jdk1.8(1.7未测试，不过jdk6038这个版本一定不可以。因为使用根证书进行签名，这个版本的jdk没有对应的命令，可能是还不支持。不过，签名这一步可以在其他的环境中完成。生成证书的每一步分理论上都可以在独立的环境中完成。)

## 1.生成证书库

首先，根证书就是给别的证书签名的证书，根证书的职责就是与CA做同样的事情，确认其他的证书是否可信，如果可信，进行签名。
    
    
    keytool -genkey -alias basic -keyalg dsa -keysize 1024 -sigalg dsa -startdate 2018/11/01 -validity 365 -keystore H:\basic.keystore -storepass basic1 -keypass keybasic
    

结果(密码最少6位)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec0c3ebcf09b674056da551b7f123902.png)  
解释一下：  
各位把keystore当做数据库，basic当做表就比较容易理解了。上面的命令使用类比数据库的方式来解释：  
keytool表示使用java_home中的bin中的keytool.exe这个程序  
-genkey表示新建一个密钥对–新建一个数据库  
-alias表示数据库中新建一个表  
basic是数据库中的一张表的名字  
-keyalg数据库中数据加密的方式  
dsa一种加密方法  
-keysize加密长度  
1024字节  
-sigalg签名的加密方式  
dsa加密方式  
-startdate数据库开始（创建）时间（ps：时间格式yyyy/mm/dd，还可以加上具体的时间不过在oracle官网中没有给出具体的例子[keytool官网文档传送门](<https://docs.oracle.com/javase/8/docs/technotes/tools/windows/keytool.html#keytool_option_genkeypair>)）  
-validity有效时间  
365（天）  
-keystore创建数据库（证书库）  
H:\basic.keystore(路径和文件名字，其实文件的后缀隐士的给出了证书库的格式jks,其他的请在官网查找)  
-storepass数据库的密码  
-keypass数据库表的密码

## 2.jks转p12
    
    
    keytool -importkeystore -srckeystore H:\basic.keystore -destkeystore H:\basic.p12 -srcstorepass basic1 -deststorepass basicp
    

结果：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c1b94862a0eafc711dbad431e852ec9.png)  
说明：  
-importstore 使用导入证书库功能  
-srckeystore源证书库  
-destkeystore目标证书库（后缀表示格式-storetype具体查看文档，不过使用后缀更加简单，本来命令就很长了。）  
-srcstorepass源证书库的密码  
-deststorepass目标证书库的密码  
主密码：证书库条目的密码（数据库表的密码）  
此时在目标位置生成了一个basic.p12的证书库，p12可以在大多数的浏览器中导入。

## 3.证书库导出cer文件

首先使用查看命令查看证书库，看看有哪些条目（表）
    
    
    keytool -v -list -keystore H:\basic.keystore -storepass basic1
    

结果：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e47fc8b45728a97f97649ea8c35de81c.png)  
说明：  
-v详细输出  
-list查看  
接下来获得cer文件
    
    
    keytool -export -v -alias basic -keystore H:\basic.keystore -file H:\basic.cer -storepass basic1 -keypass keypass
    

此时生成了basic.cer文件，cer文件是给window用来安装证书的（ps:linux以后有时间补充）

## 4.证书库生成证书请求
    
    
    keytool -certreq -alias key_server -keystore H:\server.keystore -file H:\serverreq.cer -storepass server -keypass keyserver
    

此时会生成一个cer文件，安装提示无效的证书。  
证书请求就是需要发送给ca或者第三方请求签名的文件。

## 5.对证书请求进行签名
    
    
    keytool -gencert -v -alias basic -infile H:\serverreq.cer -outfile H:\resserver.cer -keystore H:\basic.keystore -storepass basic1 -keypass keybasic
    

此时会生成一个新的cer文件，这个文件就是签名后的文件。  
说明：服务器发起请求，生成请求文件，第三方对请求文件进行签名，服务器把第三方的证书添加到可信列表，然后对签名后的文件添加可信链表，此时，就会形成一个信任链。  
服务器信任第三方，签名后的文件来源于第三方，所以，信任签名后的文件。然后呢，我们的服务器也可以进行签名，这样，对于每一个证书，都可以追溯其源头，形成信任链。  
这也是为什么ca是根证书认证中心。  
以上命令执行无任何问题，但是，可能由于某些参数设置的问题，导致生成的证书不可用。所以，上面只是命令的解析，后面会有一个完整的例子。

## 6.例子

### 6.1创建证书库
    
    
    keytool -genkey -alias key_basic -validity 365 -keystore H:\basic.keystore -storepass basic1 -keypass keybasic -keyalg rsa
    //创建根证书库
    keytool -genkey -alias key_client -validity 365 -keystore H:\client.keystore -storepass client -keypass keyclient -keyalg rsa
    //创建客户端证书库
    keytool -genkey -alias key_server -validity 365 -keystore H:\server.keystore -storepass server -keypass keyserver keyalg rsa
    //创建服务器证书库
    

### 6.2导出根证书
    
    
    keytool -export -v -alias key_basic -keystore H:\basic.keystore -file H:\basic.cer -storepass basic1 -keypass keybasic
    //导出根证书
    

### 6.3证书库发起请求
    
    
    keytool -certreq -alias key_server -keystore H:\server.keystore -file H:\server_req.cer -storepass server -keypass keyserver
    //服务器发起请求
    keytool -certreq -alias key_client -keystore H:\client.keystore -file H:\client_req.cer -storepass client -keypass keyclient
    //客户端发起请求
    

### 6.4跟证书库对请求签名
    
    
    keytool -gencert -v -alias key_basic -infile H:\server_req.cer -outfile H:\res_server.cer -keystore H:\basic.keystore -storepass basic1 -keypass keybasic
    //根证书签名服务器请求
    keytool -gencert -v -alias key_basic -infile H:\client_req.cer -outfile H:\res_client.cer -keystore H:\basic.keystore -storepass basic1 -keypass keybasic
    //根证书签名客户端请求
    

### 6.5根证书添加信任
    
    
    keytool -importcert -v -alias key_basic -file H:\basic.cer -keystore H:\server.keystore -storepass server -keypass keybasic
    //服务器添加根证书
    keytool -importcert -v -alias key_basic -file H:\basic.cer -keystore H:\client.keystore -storepass client -keypass keybasic
    //客户端添加根证书
    

### 6.6签名文件添加信任
    
    
    keytool -importcert -v -alias key_server -file H:\res_server.cer -keystore H:\server.keystore -storepass server -keypass keyserver
    //服务器添加签名文件
    keytool -importcert -v -alias key_client -file H:\res_client.cer -keystore H:\client.keystore -storepass client -keypass keyclient
    //客户端添加签名文件
    

### 6.7证书库导出cer
    
    
    keytool -export -v -alias key_server -keystore H:\server.keystore -file H:\server.cer -storepass server -keypass keyserver
    //服务器导出cer
    keytool -export -v -alias key_client -keystore H:\client.keystore -file H:\client.cer -storepass client -keypass keyclient
    //客户端导出cer
    

### 6.8 cer生成临时证书库
    
    
    keytool -importcert -v -alias server_temp -keystore H:\server_temp.keystore -file H:\server.cer -storepass servertemp -keypass server_temp 
    //服务器cer创建临时证书库
    keytool -importcert -v -alias client_temp -keystore H:\client_temp.keystore -file H:\client.cer -storepass clienttemp -keypass client_temp 
    //客户端cer创建临时证书库
    

### 6.9从临时证书库导出p12
    
    
    keytool -importkeystore -v -srcalias server_temp -destalias server -srckeystore H:\server_temp.keystore -destkeystore H:\server.p12  -srcstoretype jks -deststoretype pkcs12 -srcstorepass servertemp -deststorepass server -srckeypass server_temp
    //服务器临时证书库导出p12
    keytool -importkeystore -v -srcalias client_temp -destalias client -srckeystore H:\client_temp.keystore -destkeystore H:\client.p12  -srcstoretype jks -deststoretype pkcs12 -srcstorepass clienttemp -deststorepass client -srckeypass client_temp
    //客户端临时证书库导出p12
    

## 7.证书安装

上述命令正确执行后生成的文件  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b4de25c40bc062a11883bbc8a130895c.png)  
安装根证书  
basic.cer  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44791a8eb8b2b2b5ccb1e28c34459f30.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1578253f7f8a419222ca67d6488c161.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76666b40a76c57d34263a6f344ef1cdf.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c7091305c8cc0ea4c813ad3581a693ea.png)  
安装client.cer  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3a690315d364efc12b09da1d2fddc65f.png)  
但是如果卸载掉之前装的basic.cer  
使用win+R运行certmgr.msc  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f8321ec75fcdca9eee801f764d57e474.png)  
第二个basic就是之前安装的，第一个是我自己测试的时候安装的。右键删除basic证书。  
此时安装client.cer  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bd7d2c537aef5a9365a2441be6f56e48.png)  
很明显，有警告提示了。如果你能把自己basic.cer直接在操作系统层面，从计算机出厂就默认安装，那么，你就有和ca同样的权利，使用basic进行签名的证书，都不会有警告。这也是正规证书和野证书的区别。正规证书是我们一开始安装client.cer的情况，野证书会有提示。  
这也是我们为什么要大费周折的创建一个basic证书库，因为把basic证书库自己添加到可信任机构，那么由basic签名的证书都不会提示。  
接下来安装p12文件（未安装basic）  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e9a51a4c0872f769c7acfcd6a4a0c95f.png)  
提示输入密码，这个密码是哪里设置的？
    
    
    keytool -importkeystore -v -srcalias client_temp -destalias client -srckeystore H:\client_temp.keystore -destkeystore H:\client.p12  -srcstoretype jks -deststoretype pkcs12 -srcstorepass clienttemp -deststorepass client -srckeypass client_temp
    //客户端临时证书库导出p12
    

这里的deststorepass的值就是上面图片需要的密码。  
现在查看一下服务器证书库，客户端证书库，跟证书库的情况：  
根证书库：
    
    
    keytool -v -list -keystore H:\basic.keystore -storepass basic1
    //查看
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/86944b37de4226f1d0d2a9c109114cda.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ea871374e192f4f43c30300b23683843.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a8cb94de77273df3a6892b5c75bebced.png)  
总结一下：  
basic:----------key_basic------所有者basic------发布者basic-----链长度1  
server:----key_basic-------所有者basic------发布者basic  
-------key_server---------链长度2-------所有者server-----发布者basic  
链接到key_basic  
client:----key_client------所有者basic------发布者basic  
------key_client-----链长度2-----所有者client-----发布者basic

## 8.客户端服务端增加相互信任

服务器证书库添加信任客户端证书（增加一张表）
    
    
    keytool -importcert -alias key_client -keystore H:\server.keystore -file H:\client.cer -storepass server -keypass keyclient 
    

客户端证书库添加信任服务端证书
    
    
    keytool -importcert -alias key_server -keystore H:\client.keystore -file H:\server.cer -storepass client -keypass keyserver
    

## 9.tomcat设置

Tomcat安装目录下的conf文件夹中的server.xml文件中加入
    
    
    <Connector port="8060" protocol="org.apache.coyote.http11.Http11Protocol" 
    	SSLEnabled="true"  maxThreads="150" scheme="https" secure="true"   
        clientAuth="true" sslProtocol="TLS"   
        keystoreFile="H:/server.keystore" keystorePass="server" 
    	truststoreFile="H:/server.keystore" truststorePass="server"
    	/>
    

注意反斜杠  
然后启动  
注意：这里有一个坑----  
先不要用bin目录下的startup,先使用configtest，测试一下，会发现报错：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5eacd299b0a7aa91d9fa57d7bd5f6725.png)  
原因：我们数据库有一个密码，数据库表还有一个密码。  
对应：我们证书库有一个密码，而每一个条目还有对应的密码。  
但是tomcat配置中只有一个密码参数，所以要求证书库的密码和条目的密码需要一致。  
  
修改对应条目的密码
    
    
    keytool -keypasswd -v -alias key_server -keystore H:\server.keystore -storepass server -keypass keyserver -new server
    //修改条目密码
    

第二个坑:使用keytool -help 没有-keypasswd这个参数。导致刚开始只要发现需要改条目密码就需要重新创建。=。。=很坑。  
再次运行测试：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/08e438a1dbf7af86e5204ca935571b54.png)  
然后使用startup启动  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e041f862dfc59a14b4ce9b196059fbc6.png)  
使用浏览器访问：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f77483d536c590656e2ff55d79e8f55d.png)  
浏览器导入p12文件：客户端的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ac40e376aa4e94f5006a0e0ee7d1a27c.png)

安装basic.cer  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/62321227ff8101cecb587a9fea244e90.png)

----------------------------------------------2021-01-08---------更新

之前留下的坑，当时做不下去了。  
最近在看http协议

里面也涉及到了https的配置。正好将这个坑给补上。

## 填坑了

## 1. 目的

首先，我们为什么需要https？  
当然是安全。  
http请求的报文是没有加密的，是明文的。所有的信息在互联网的世界中裸奔，谁都可以看到。  
为了解决这个问题，就需要给报文加密。加密了，其他即使拦截到，也因为无法解密，从而保护报文的安全。

## 2. 加密

目前从秘钥的角度来说，加密分为对称加密，和非对称加密。

### 2.1 对称加密

给你一个数字，12345，如何进行加密呢？  
比如将12345加密为34567  
发现规律了吗？  
我将每一位的数字都加2，这个2就是秘钥。  
解密就是加密的逆过程，将每一位减2即可。  
34567->12345.  
这种就是对称加密：加密和解密使用相同的秘钥。  
解密就是加密的逆过程。

### 2.2 非对称加密

还是相同的数字，12345，除了对称加密，你还能想到其他的加密方式吗？  
比如将12345加密为89012  
这个规律也很简单，将数字的每一位加7，然后对10进行取余。  
解密和对称加密不同。  
对称加密的解密是加密的逆过程。  
非对称加密的加密过程中用到秘钥称为公钥。  
也就是7.  
根据规则和公钥，可以计算出一个私钥，这个私钥就是解密用的。  
因为我们的规则是：(x+y)%10  
x作为明文的位，y是公钥。  
私钥的计算方式：10-y  
所以，私钥是3.  
非对称加密就是使用私钥，在加密一次，然后就得到明文了。  
89012->12345

### 2.3 非对称加密的优点

**使用对称加密**  
实际使用中，非对称加密一般更加安全。  
因为对于对称加密来说，解密的人知道秘钥，就知道了加密规则。  
这样就能进行伪装。  
比如客户端和服务端使用对称加密。  
A客户端先假装和服务器通信，获取到服务器的秘钥后，对服务器做了代理。  
此时B客户端也来访问服务器，但是B客户端和服务器不知道A对服务器做了代理，此时B客户端会先访问A(代理服务器)，然后A因为已经知道了对称加密的秘钥，所以A可以拦截到B的请求，然后进行解密，接着进行篡改请求，然后将篡改后的请求进行加密，接着请求服务器。  
服务器收到请求，使用对称加密秘钥解密，然后响应请求，返回A。  
A收到响应，将请求解密，接着篡改响应，然后将响应使用对称秘钥进行加密，返回给B。  
B收到响应，解密。  
举个实际的例子：  
我们使用网页版的京东进行购物。  
B请求给自己的手机号码充值20元。  
A拦截到请求后，将请求进行篡改：给B的手机号充值20,；给A的手机号充值0.01.  
京东服务器收到请求后，给A的手机号充值0.01，给B的手机充值20.  
然后将服务器响应：手机号A成功充值0.01，手机号B成功充值20.  
A拦截到响应，解密后，将响应修改为：手机号B成功充值20，然后加密返回。

听起来没毛病是吧。  
但是整个过程中，A平白无故获得了0.01元的话费收入。  
而且，现在一般充值话费，都是分为两部分：20=19.99+0.01  
现在充值20元话费，运营商实际上收取19.99元。

但是在B来看，没毛病，我充值20元，花了20元。

实际上A利用信息差，从B哪里非法收入了0.01.

1个或许没问题。但是，每天通过京东充值话费的有多少呢？

**使用非对称加密**  
在网站中，服务器一般会将公钥分发给客户端，告诉客户端，你请求我的时候，需要使用公钥加密后，在传给我。  
服务端收到请求后，使用私钥进行解密，就能知道客户端的请求内容了。  
非对称加密的优势在于：客户端即使获得了公钥，也无法破解私钥，无法对密文进行解密。  
还是以充值话费的例子：  
A假装和服务器通信，获取了公钥，然后A对服务器做了代理。  
B使用相同的公钥加密请求：给B的手机号充值20元话费。  
A拦截到了B的请求，因为A拿的也是公钥，无法解密，也就无法篡改了。(即使强行篡改，也会使服务器无法解密识别)  
服务器最终只给B的手机号充值了20元，B只花费了19.99元。

## 3. tomcat https单项加密

### 3.1 数字证书

在互联网中，非对称加密的公钥一般存储在数字证书中。  
在win系统中，使用win+r运行certmgr.msc打开证书管理界面  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f6e76e594ee02002f2cd1204746da099.png)  
比如这种  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/956321a5af3606c007a898af000f89d8.png)  
证书分为很多种，有些证书中不仅仅可以有公钥，还可以有私钥  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9cce5c70e0f76793047c9fcb6d099593.png)

### 3.2 数字证书的生成

首先需要安装jdk，我们使用keytool生成数字证书。  
常见来说，我们将公钥和私钥在一起存储使用keystore，也就是证书库的方式存储。  
只要公钥则使用cer存储，也就是证书。  
确保jdk安装成功，且版本最好和我的一致。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/97077eed63be3824ac53d8c8559ed64f.png)  
首先我们创建一个证书库
    
    
    keytool -genkey -alias userDataKey -keyalg rsa -validity 365 -keystore userDataKey.keystore -storepass userkey -keypass userkey
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0bd6c39e37a6628c887a60e41f75b747.png)  
要求输入一些信息。  
接着会生成指定的证书库  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/63189b14ad5aa7e2a2d829441e573142.png)

我们查看证书库
    
    
    keytool -v -list -keystore userDataKey.keystore -storepass userkey
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aee723165dd94c68e1b9d60ec443edcf.png)  
从这里看出我们真正加密算法是SHA256和RSA

### 3.3 tomcat配置

我们使用8.5.32版本的tomcat.  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/75ebd76ba60ffe1e05ddf9000368d298.png)

首先打开server.xml,在里面找到实例配置  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf7b43052ab3da51f21b077ad49e1bd8.png)  
将这个注释放开
    
    
        <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
                   maxThreads="150" SSLEnabled="true">
            <SSLHostConfig>
                <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                             type="RSA" />
            </SSLHostConfig>
        </Connector>
    

然后将证书库修改为我们刚才生成的证书库。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df102f82928f78efed95d9abf67f6b25.png)

然后启动  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0819d29d2dd1d29dbc5ea0142ac00bb.png)  
访问，会提示不是专用链接  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb332ba3e5eca172367e02b505e9661e.png)  
点击高级，选择继续访问  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c04ac4cbebc2165f5d226772d6da652c.png)  
最终我们可以访问到需要的界面  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4ed1766172af8997763b2dd9440dc939.png)  
点击不安全几个字

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/848b0009a3490bb4d26b9432b22f3fdd.png)  
会提示证书无效  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aee268583e0120ca70bb15cb3c6ca5fd.png)  
查看证书，正好是我们刚才生成的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/693f8470a1883c759ef4b60aae37a4da.png)  
证书状态是不受信任的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e52bf878d5754aef7bd717216e7c4430.png)

## 4. 证书信任

我们在3中实现了tomcat下的https单项加密。  
在2.3中的例子中，我们将，使用了非对称加密，可以避免信息被篡改。  
但是，这不是绝对的。  
我们还是以例子来说：

A假装和服务器通信，得到了服务器分发的数字证书，也就是公钥。  
然后A对服务器做代理拦截。

B请求服务器充值话费，但是B不知道A做了拦截，于是B首先请求A，A发现B第一次请求，于是A将自己生成的数字证书发送给B，而B不知道自己受到的证书是A的证书，还是服务器的证书，没有验证的手段。  
我们这里将服务器的证书成为真证书，而将A的证书称为野证书。

所以，B拿着A给的野证书，也就是公钥，对请求进行加密。然后将请求发送给了A。

A受到加密后的请求，用野证书的私钥进行解密，拿到了B请求的明文。然后A对请求明文进行篡改，篡改完成之后，使用服务器的真证书进行加密，加密完成后，发送隔服务器。

服务器收到加密后的密文请求，使用真证书的私钥进行解密，发现，请求符合服务器的规则，服务器能识别，于是进行了处理。

处理完成后，服务器将响应使用私钥进行加密，然后将加密后的密文响应返回给A服务器。

A服务器收到响应密文后，使用真证书的公钥进行解密(非对称加密，公钥和私钥都可以加密解密。在一次加密解密的完成过程中，必须使用两个秘钥。这两个秘钥是互补的)，得到了明文响应，我们称服务器的响应为真响应，代理服务器的响应为假响应。  
A服务器篡改明文真响应，然后使用自己的野证书的私钥进行加密，然后将加密后的密文假响应返回给B。

B收到A服务器的密文假响应后，使用野证书的公钥进行解密，获得了假响应的明文。

在整个通信的过程中，问题就在于B服务器无法确定自己受到的证书是真证书还是野证书，也就无法知道自己请求的是真正的服务器还是代理服务器。

### 4.1 http/https请求过程

看到这里可能有人会问，我们访问京东，为什么会访问到A的代理服务器呢？

这里就涉及到了http、https的请求过程了。

我们一般访问一个网站，通常有两种方式：  
1.直接在浏览器的地址栏输入网站的域名进行访问。  
比如常用的www.baidu.com.  
一般浏览器默认是http访问，域名默认就是万维网，也就是www.  
所有，有时候，在浏览器的地址栏输入baidu.com也能访问。  
2.如果我们不记得网站的域名，就需要搜索，然后通过搜索结果的超链接进行访问。

不管是方法1还是方法2，本质上是相同的，都是通过域名访问网站。

我们还应该明确一点，访问网站，其实就是访问网站对应的应用。

所以，http/https请求都需要一个请求的前置操作：访问DNS。  
我们在配置计算机网络的时候，要指定DNS服务器的ip地址。(有时候是DHCP则是通过IP地址的分配，计算机请求路由器，路由器返回DNS服务器地址的)  
浏览器拿着域名，去问DNS，www.baidu.com的网站的IP是多少？

DNS正确的返回x.x.x.x:aaaa。  
接着浏览器在去访问x.x.x.x:aaaa.

### 4.2 DNS劫持

如果DNS被劫持了呢？  
什么是DNS劫持？  
就是说我要访问百度，结果出来了百A，百B等等网站。  
这就是DNS被劫持了。

DNS劫持如何实现？  
如果你了解浏览器请求DNS的过程，你就很容易实现DNS劫持。

使用浏览器访问网站是一个非常频繁的操作，基本上我们使用浏览器，几乎无时无刻在访问网站，而每一次访问网站都需要请求DNS服务器。  
我们每一次访问DNS服务器，就需要和DNS服务器建立TCP连接，这样整个过程是非常耗费资源的。  
为了优化DNS请求的速度，我们使用多级缓存的方式，来解决这个问题。  
DNS访问的优先级

  1. 本地hosts
  2. 本地DNS
  3. 远程DNS(远程DNS会级联上报)
  4. 跟域名服务器(跟DNS)

#### 4.2.1 hosts

第一级：hosts文件。  
在hosts文件中会存储一些域名和IP的映射关系。  
hosts文件在`C:\Windows\System32\drivers\etc`目录下  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de29c8f64ec349986a49db4fd06a4f01.png)  
里面的内容也很简单，每一行就是一个映射关系，前面是ip地址，中间使用空格分隔，然后是域名。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3749db53acf1a0cc224cbc9893f6f1f0.png)  
比如我们增加一个DNS关系  
baia.com -> baidu.com  
首先我们通过cmd的ping命令得到baidu.com的ip地址  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ad038d50697cad7c4e4e5e79e7d56ad.png)  
接着增加baia.com的映射  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/23ca979daa7165dea4bc081cc6984dcf.png)  
接着访问baia.com  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4341384180bf4a09a15a3a87dba416b5.png)  
请注意，这里访问我们劫持的baia.com必须使用https的方式请求。  
因为baidu做了全站https支持。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c1cc355cd81107229062906003c7f8e5.png)  
点击高级，就会提示，我们收到的证书是baidu的证书，但是访问的域名却不是baidu的域名。  
从这里也就能证明，我们确实做了dns劫持，将baia劫持到了baidu上。

不过，因为hosts文件的优先级最高，所以一般情况下，不建议修改这里的映射关系。(之前有种跳过G F F FW的方式就是修改hosts)  
因为hosts的映射失效了，就会导致域名无法访问。(但是该改还是改，不用怕，改出问题了，将自己增加的删除就行)

#### 4.2.2 本地DNS

打开我们的服务管理  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/14d2faf3e1989e27757e957c39145c55.png)  
选择管理哦，不是属性。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d450804df467b6dc2cccd3161cd3a09e.png)  
找到DNS Clinet服务  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f077ead3bd640fd8a40887edf383966f.png)  
里面的说明

> DNS 客户端服务(dnscache)缓存域名系统(DNS)名称并注册该计算机的完整计算机名。如果该服务被停止，将继续解析 DNS 名称。然而，将不缓存 DNS 名称的查询结果，且不注册计算机名。如果该服务被禁用，则任何明确依赖于它的服务都将无法启动。

这个是本机的DNS，如果停止本机的DNS，那么每次就会从IP地址指定的DNS查询映射关系。

有了本机的DNS，就会将远程的DNS的映射关系，在本机进行缓存。在下一次访问的时候，就会快很多(因为本机访问，会受到网络等因素的波动影响)。

#### 4.2.3 远程DNS

我们打开cmd,使用`ipconfig/all`，查看远程DNS  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b9a2f0f6227f1ef37b0cee8450a0926d.png)  
这个就是我自己的远程dns，一般你在公司，使用的是公司的dns服务器，在家里，则是使用运营商的dns服务器。  
在dns服务器中会对一些特殊的域名，做过滤。比如涩情、暴力、血腥的网站的域名，做个过滤，那么使用这些dns服务器的用户，就不能访问这些被过滤的网站，起到一个保护的功能。

如果远程DNS服务器上没有查询域名的映射关系，远程DNS服务器就会查询他自己的DNS服务器。也就是上报。  
一直上报，直到跟域名服务器

#### 4.2.4 跟域名服务器

> 根服务器主要用来管理互联网的主目录，最早是IPV4，全球只有13台（这13台IPv4根域名服务器名字分别为“A”至“M”），1个为主根服务器在美国。其余12个均为辅根服务器，其中9个在美国，欧洲2个，位于英国和瑞典，亚洲1个位于日本。

如果根域名服务器也找不到，那么很可惜，这个域名你就无法访问，其他人也无法通过域名进行访问。

### 4.3 根证书

对于我们京东充值话费的例子，如何避免DNS劫持下的数字证书被替换的问题呢？

其实也很简单，找一个担保人。  
就像我们生活中一样，A和B借钱，但是A和B还不太熟悉，B不敢直接把钱借给A。于是A和B找了一个他们都信任，都熟悉的C，在C的担保下，B将前借给了A。  
C作为担保人，就需要督促B将出借金交给A，同时在还款日期时，督促A将本金和利息归还B。  
数字证书也是一样的，我们访问京东，就会收到京东的数字证书，浏览器会检查证书是否是可信的。(还记得我们3中，自己给tomcat配置单项加密的时候，证书状态是不可信的)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/93368a30c5dc31d657c9ddfec682534d.png)  
证书上记录了担保人，也就是颁发者。以及有效期。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cdc1be727306095dcbeaf58ce5a94585.png)  
同时证书的状态也是可信的。

#### 4.3.1 创建签名证书

我们之前用借贷的方式，解释了可信证书。  
在借贷场景中，我们需要找一个担保人。  
同时我们在查看京东的证书的时候，我们知道了颁发者是  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3431e586ba35596673dbc58a78217f71.png)  
接着，我们打开证书管理器，在其中找找这个证书  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ae87daef7039c2c3033c78307ee2e963.png)  
我们发现这个证书在中间证书颁发机构的目录下。  
请注意两个关键点：

  1. 中间机构
  2. 颁发机构  
所以，我们看到这个中间证书的颁发者是![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1953d4d88849ed49ef946a994225e805.png)  
再找  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c21567555e11cbaa9985f72092d6473c.png)  
找到了。  
所以，类比这个过程，我们首先创建一个担保证书。  
这个担保证书是服务器和客户端都信任的证书。  
也就是签名证书。

    
    
    keytool -genkey -alias basic -keystore basic.keystore -keypass basic1 -storepass basic1 -validity 365 -keyalg rsa
    

生成一个签名证书库，  
使用
    
    
    keytool -v -list -keystore basic.keystore -storepass basic1
    

查看证书库。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/43e3accb808ccdd2679c1e2bc06955f6.png)  
接着将证书库中的证书导出来(公钥)
    
    
    keytool -export -v -alias basic -keystore basic.keystore -file basic.cer -storepass basic1
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c6d089b26772bfa428a0e322e753e07.png)  
接着我们安装签名证书(也就是公钥)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a92746c76f94288f179bbcc6a9dbdb2f.png)  
我们安装到信任的根证书办法机构目录下  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cc295a63ed5ffb527ec4fbc5fb930dbb.png)  
我们选择是  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/86db0840446c8ee35462f927f6a354bd.png)  
此时在证书管理界面中就能看到我们安装的证书了  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/355afd2c64719a9224ef607befff5fcf.png)

#### 4.3.2 签名请求

现在，担保人找到了，我们需要和担保人说明白是什么事情。  
比如A向担保人提出借钱请求，担保人了解情况后，就可以向其他人证明，A借钱是有能力还款的。至于谁愿意给A借，就不是担保人能决定的了。

比如你去银行借钱，找担保人担保，借1W快，担保人给你担保。  
你去银行借钱，找担保人担保，借1个亿，我想，没人给你担保吧。(一个普通人的角色，大佬估计也不会看到我写的东西)

所以，首先，你要想担保人请求借钱。  
这个在某些情况下，是银行要担保人出具担保证明。  
然后你就带着礼物，请求担保人给你出具担保证明。

我们先生成clienta待签名的证书库
    
    
    keytool -genkey -alias clienta -keystore clienta.keystore -validity 365 -keyalg rsa -storepass client -keypass client
    

查看  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37eab4b22426887811a21867283b64dc.png)  
此时如果我们生成clienta证书的公钥cer文件，然后安装cer公钥文件，会不受信任的。
    
    
    keytool -export -v -alias clienta -keystore clienta.keystore -file clienta.cer -storepass client -keypass client
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1868cda60c1636a4bd8882fbcc1ba64b.png)

对于clienta生成签名请求
    
    
    keytool -certreq -alias clienta -keystore clienta.keystore -file clienta_req.cer -storepass client -keypass client
    

查看签名请求
    
    
    keytool -printcertreq -v -rfc -file clienta_req.cer
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/491e17a4f9be1e2e7ec7389fa085fc23.png)

#### 4.3.3 签名回复

借款人找到担保人说明请求后，担保人经过评估，同意开具担保证明。也就是同意在担保证明上签字。或者叫回复。  
所以，我们使用信任的basic证书库对clienta_req.cer的签名请求进行回复。(也可以认为是签名过程)
    
    
    keytool -gencert -v -alias basic -infile clienta_req.cer -outfile res_clienta.cer -keystore basic.keystore -storepass basic1 -keypass basic1
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1a6ac9975ad9371b2ade2687c7ddd0be.png)  
查看签名回复  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9d7c8252b85ef4ecf27181cab7e7e294.png)  
怎么说呢，这个结合我们的例子，主要就是这样的。  
服务器直接将自己的证书进行分发，容易出现被篡改。  
所以。服务器分发的是经过担保机构加密的证书。  
此时即使别人拦截了加密的证书，对证书进行替换，也无法通过担保机构的审核(因为审核需要备案，像一个域名，基本上只允许注册一个证书(同一个担保机构))，如果代理服务器想替换证书，就意味着一个域名有两个证书，这样就有问题了。  
就会出现我们在4.2.1中的，劫持了百度的dns，然后就无法访问，因为证书和域名不匹配。  
这样就在一定程度上避免了篡改和替换。

#### 4.3.4 签名回复导入

在上面我们讲到，签名过程实际上是将clienta的公钥使用信任证书basic的私钥进行加密，然后返回clienta加密后的密文。  
然后clienta的证书库就会将信任证书basic加密后的密文倒入自己的证书库。  
要让整个信任链能成功建立，首先cliena证书库要先信任basic才行，我只有先信任你，才能信任你的签名。  
先让clienta证书库导入信任证书库的公钥
    
    
    keytool -importcert -v -alias clenta -file basic.cer -keystore clienta.keystore -storepass client -keypass client
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5afe87a97a2eee54dbcc339a60db0045.png)  
此时查看clienta证书库  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/498967ef9bd58b7f3a33e3aff1a8e375.png)  
虽然别名相同，但是类型不同哦。  
然后将签名回复导入
    
    
    keytool -importcert -v -alias clienta -file res_clienta.cer -keystore clienta.keystore -storepass client -keypass client
    

因为我们已经信任了basic，所以，basic的回复，直接就会信任，并不会在给出提示，询问是否信任。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d122388bf26e2f4a0cad5b16acc9b960.png)  
我们查看clienta证书库

接着导出clienta证书库的公钥证书
    
    
    keytool -export -v -alias clienta -keystore clienta.keystore -file clienta.cer -storepass client -keypass client
    

此时查看  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d73e07d5cdd94984b4b0c888bc87ab4c.png)  
发现证书链的长度已经变成了2，证书1是clienta的证书，证书2就是信任证书basic的公钥  
请注意证书2的指纹尾数是88  
而clienta证书中信任basic的证书的指纹尾数也是88  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/099631d90dd9c6eed8eb27c5f6ca3a69.png)  
将接受了签名回复的证书库导出证书(公钥)
    
    
    keytool -export -v -alias clienta -keystore clienta.keystore -file clienta.cer -storepass client
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/044c3c0a1c4dfd8bb9714ee5e6fe0f86.png)  
然后安装证书  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fed4d1053a6d88c6d63abb2acfd258c4.png)  
此时，颁发者就是信任证书了。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/93a24bfc53368b2629fe8952944d1e69.png)  
证书的状态和证书链也都是完整的。

## 5. tomcat https加密(证书安全)

我们在4中生成了安全的，经过信任的证书。  
接着我们将信任的证书库，配置到tomcat中。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4ef062dd665e38b57edc99c034f8acb6.png)  
然后重启tomcat,然后访问  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3842b8f5a8d24eb36f4ff622efccc052.png)  
虽然还是报红(这是没办法的，我们的根证书basic是野证书，真正的ca证书签名要花钱的)  
但是，当我们查看证书的时候，证书是有效的。  
还有一点，请注意，我们给tomcat配置了证书库和密码，tomcat生成了只有90天的证书，然后将这个证书分发给了浏览器。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d60d7739a4838bdd3e8f7d6c852df790.png)  
keytool导出证书，有效期无法修改，只能是90天。  
如果有浏览器和tomcat中间有代理服务器，拦截了tomcat分发的证书，那么，只要我们没有用basic对代理服务器的证书进行签名，那么这里证书就是有问题的(类似之前看到的baia和百度的证书)

## 6. tomcat https双向加密(证书安全)

因为既有服务器端证书，也有客户端证书。  
所以，我们先卸载全部的自己安装的证书,留下信任证书basic.  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/31f12bd5998f56e245480483a3f3e8e4.png)  
然后删除目前我们生成的clienta的证书。(不删除也行，重新生成也行)  
这部分就和原来写的6完全相同了。(只操作6就行了，不需要8)  
这里不再重新赘述。  
你生成的证书应该有这些：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2813554303f374e814cdf85cc377a7f1.png)  
然后我们打开tomcat的server.xml，进行配置  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2e71509dcdeee209986de4fa991e2324.png)
    
    
      <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
                   maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
                   keystoreFile="C:/Users/star10008377001/Desktop/tomcat/server.keystore" 
           				 keystorePass="server"
                   clientAuth="true"  
           				 sslProtocol="TLS" 
                	 type="RSA"
                	 truststoreFile="C:/Users/star10008377001/Desktop/tomcat/clienta.keystore"
                	 truststorePassword="client"
                   >
           
        </Connector>
    

可以在tomcat的webapps目录下的doc目录下，查看tomcat的文档  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99a5b0c304a0ad488a8df9e8830bd8a0.png)  
找到ssl相关的html  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/91358514c3de9f6a7419bd81d14e15ed.png)  
浏览器中会展示这样的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f2c76628ec6c8fe2a2fa83914ab14a98.png)  
找到Connector  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/86c1278a23e767f09741c84f57ed009a.png)  
找到SSLHostConfig  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76ceb1fc758bed3056347fbbda8465e6.png)  
然后就能找到配置项和说明了  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/672eca179b13b876332ed3358a86bddc.png)  
这里需要注意protocol的版本哦  
接着我们启动tomcat，然后用浏览器访问  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/40119f488406cdb0151ddc922ed83316.png)  
会提示没有提供登录证书  
可以看到，服务器端的证书是有了，但是客户端的证书还没有  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0876240fe577698740ebd336f96590dc.png)  
我们使用keytool将keystore转为p12类型的证书。  
用于安装
    
    
    keytool -importkeystore -srckeystore clienta.keystore -srcstoretype jks -destkeystore clienta.p12 -deststoretype pkcs12 -storepass client -srcstorepass client
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b8b769cf112f39173da8ec401d293194.png)  
此时生成了clienta.p12的安装证书库。  
我们双击安装  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/811fa8260ac8f917a7fe93d9462c739e.png)  
自动选择证书存储位置就行  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ea3b4431dc8f0cf20331f048aeed6fb6.png)  
接着刷新浏览器，浏览器就会提示我们选择用于身份验证的证书  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/caff9f12b859eed04f740f2a991a947a.png)  
我们点击确定，就可以访问了  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8332f643938f210c66e06d64950719ad.png)

**注意点：细心。双向认证，我从头开始做了不下8遍。不细心，导致各种各样的问题。特别是协议不支持的问题。**

## 7. tomcat https 双向加密-多证书(证书安全)

前面将的都是配置一个客户端证书，一个服务端证书。  
如果我们有多组证书该怎么配置呢？  
先要明白，一个端口确实只能配置一对证书。  
但是我们可以做端口转发。  
比如我按照7中的例子，生成了clientb的证书。以及serverb的证书。  
接着我们在tomcat的server.xml中配置  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/439355ca32607731a7bdd047b1912ec7.png)

里面主要注意两个点：首先是端口不能存在冲突，其次是做端口转发。
    
    
          <Connector port="8442" protocol="org.apache.coyote.http11.Http11NioProtocol"
                   maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
                   keystoreFile="C:/Users/star10008377001/Desktop/tomcat/serverb.keystore" 
           				 keystorePass="server"
                   clientAuth="true"  
           				 sslProtocol="TLS" 
                	 type="RSA"
                	 truststoreFile="C:/Users/star10008377001/Desktop/tomcat/clientb.keystore"
                	 truststorePassword="client"
                	 redirectPort="8443"
                   >
           
        </Connector>
    

接着启动，并访问8442端口  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe958f84f3f702823567b21b49b28936.png)  
此时客户端的证书都是安装的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6f949b245e65bab2ec62b5cbcafbc132.png)  
客户端认证证书，需要带有私钥的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb2df3f3dcfa46f0cf48e51864ad7429.png)  
我们删除clienta证书，此时8443无法访问。  
右键删除就行  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/314554dc31e8e42cefcd0bfd82ce3c48.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/697d1d6af2660c367b3df1e1e4df337b.png)  
重启浏览器此时访问8443  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/15f1e0650416d2f966d42f7623aa0095.png)  
会要我们选择一个证书。  
也就是说，只要客户端有任意一个证书就行，不管是哪个端口。

如果我们将clienta证书也安装。重启浏览器访问  
此时选择证书就是两个![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d5b8d5ae78baa2f576e6fb3d1e177168.png)

注意点：浏览器每次重启，都需要选择客户端认证的证书。  
所以，如果修改了tomcat配置，发现证书还是生效。那么可能需要重启浏览器。  
如果是安装证书，则不需要重启浏览器。

至此，终于将这个大坑填上。