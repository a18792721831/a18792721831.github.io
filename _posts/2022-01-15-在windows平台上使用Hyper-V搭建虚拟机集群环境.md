---
layout: post
title: "在windows平台上使用Hyper-V搭建虚拟机集群环境"
date: 2022-01-15 01:04:39 +0800
categories: [windows, centos, linux]
description: "本文详细介绍如何在Windows平台上使用Hyper-V创建并配置Linux虚拟机集群，包括安装操作系统、网络配置及使用Xshell进行连接。"
keywords: windows, centos, linux
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/122505308
> - 发布时间：2022-01-15 01:04:39
> - 阅读量：4
> - 分类：大数据同时被 3 个专栏收录, 订阅专栏, hadoop, 技术分享
> - 标签：#windows, #centos, #linux

## 摘要

文章浏览阅读4.2k次，点赞2次，收藏12次。本文详细介绍如何在Windows平台上使用Hyper-V创建并配置Linux虚拟机集群，包括安装操作系统、网络配置及使用Xshell进行连接。

---

#### 在windows平台上使用Hyper-V搭建虚拟机集群环境

  * [开启windows服务](<#windows_1>)
  * [启动服务](<#_35>)
  * [下载镜像](<#_53>)
  * [创建虚拟机](<#_59>)
  * [安装linux系统](<#linux_153>)
  * [工具连接](<#_205>)
  * [多个虚拟机](<#_289>)
  * [总结](<#_317>)
  * [直接使用物理机的网卡--解决虚拟机网络慢的问题](<#_329>)

## 开启windows服务

首先需要启动hyper-v的windows服务

![image-20220114173507925](https://i-blog.csdnimg.cn/blog_migrate/3948c3e0d0ac7c87a8ee25caaca9a092.png)

打开windows更新

![image-20220114173610883](https://i-blog.csdnimg.cn/blog_migrate/efea52e7b2bf08636d67536a9c9bd86c.png)

选择开发人员模式

![image-20220114173646380](https://i-blog.csdnimg.cn/blog_migrate/713e98c06b77022075003ca439662051.png)

然后选择应用

![image-20220114173715973](https://i-blog.csdnimg.cn/blog_migrate/64eb924e1265d0379b6eb76e871d34a2.png)

然后选择【程序和功能】

![image-20220114173748839](https://i-blog.csdnimg.cn/blog_migrate/14642cb7fe2e6d9238618e54d2b9f304.png)

然后选择【启用或关闭windows功能】

![image-20220114173832545](https://i-blog.csdnimg.cn/blog_migrate/a8ae450b46eea6960d5983e79d7b9d7e.png)

把Hyper-V的√选中

![image-20220114173903636](https://i-blog.csdnimg.cn/blog_migrate/d5c18cf998dde2157993735c0d810f48.png)

重启电脑生效。

如果上述步骤中哪一步没有，请更新windows至最新。

## 启动服务

打开【服务】管理

![image-20220114174034646](https://i-blog.csdnimg.cn/blog_migrate/e60297740c3a290bbf997573677de89d.png)

手动启动Hyper-V的服务

![image-20220114174117700](https://i-blog.csdnimg.cn/blog_migrate/0c1bfb0bb387f0979d82207f5fa2edac.png)

然后打开Hyper-V的虚拟机创建向导

![image-20220114174152916](https://i-blog.csdnimg.cn/blog_migrate/be947ad224d72d66b1e05afa567a0850.png)

当出现如下窗口，表示Hyper-V启动成功

![image-20220114174322406](https://i-blog.csdnimg.cn/blog_migrate/d7649adfa78bc354257a971204f9de1a.png)

## 下载镜像

下载自己想安装的操作系统，我选择的是centos，[The CentOS Project](<https://www.centos.org/>)

![image-20220114174448535](https://i-blog.csdnimg.cn/blog_migrate/1b0f432712bfb3ebe5dc5b456e303acf.png)

## 创建虚拟机

在创建虚拟机向导中，选择本地安装源

![image-20220114174521620](https://i-blog.csdnimg.cn/blog_migrate/f4d1b601eaca39752cf25366100e2fd8.png)

然后更改安装源

![image-20220114174551754](https://i-blog.csdnimg.cn/blog_migrate/0a5c47209ce4719ec78ba116005084c4.png)

选择想要安装的操作系统的镜像文件，如果你安装的操作系统不是windows，请去掉windows的√

![image-20220114174744997](https://i-blog.csdnimg.cn/blog_migrate/fdb940e33983fa0db98e07fba2b1de36.png)

接着配置虚拟机的物理资源

![image-20220114174820531](https://i-blog.csdnimg.cn/blog_migrate/741e5fb029bfd1d7b0ecf0c7415090fe.png)

**内存**

![image-20220114175107670](https://i-blog.csdnimg.cn/blog_migrate/87769e6f8923ecaa2d840f0114ffe475.png)

**处理器**

![image-20220114175140290](https://i-blog.csdnimg.cn/blog_migrate/c088315d749dd8a97efe4a3ed6ec2c5f.png)

**硬盘**

默认是虚拟机使用C盘的磁盘空间

![image-20220114175241903](https://i-blog.csdnimg.cn/blog_migrate/46564f938536f0861eba6937c8fff5e7.png)

使用虚拟硬盘新建向导，自定义位置

![image-20220114175311941](https://i-blog.csdnimg.cn/blog_migrate/571cda82b3a732149d0110302566c5b8.png)

选择动态扩展

![image-20220114175327660](https://i-blog.csdnimg.cn/blog_migrate/0aaa4509c98c5a8b564a72ee4f8ffd21.png)

自己指定存储名字和位置

![image-20220114175411002](https://i-blog.csdnimg.cn/blog_migrate/9e58ba2c5ededcd9551ed8fa9f4b0d4e.png)

指定磁盘大小

![image-20220114175432626](https://i-blog.csdnimg.cn/blog_migrate/e48daaf6f2db5abacd8255cc4c22f0e6.png)

最后完成

![image-20220114175444361](https://i-blog.csdnimg.cn/blog_migrate/6d70a2a1cd6ae14841372ce741304968.png)

**虚拟机主机名**

![image-20220114175519609](https://i-blog.csdnimg.cn/blog_migrate/323aa88f2918a6fbcf8b4f4331c95643.png)

**集成服务**

![image-20220114175541704](https://i-blog.csdnimg.cn/blog_migrate/452abceab2608f99046d968e8a3a8652.png)

**检查点**

关闭检查点可以节省资源

![image-20220114175619501](https://i-blog.csdnimg.cn/blog_migrate/77594910564aa19bf389c978f5eb7ffd.png)

**智能分页**

![image-20220114175725094](https://i-blog.csdnimg.cn/blog_migrate/20a880b808ef926195b308c3e4e09e5a.png)

**自动启动**

自动启动是指是否物理主机启动时，自动启动虚拟机

![image-20220114175808426](https://i-blog.csdnimg.cn/blog_migrate/061bf2f6507874fb0f76951c783ed2f2.png)

**自动停止**

自动停止是指物理主机关闭时，保存虚拟机

![image-20220114175855518](https://i-blog.csdnimg.cn/blog_migrate/777f846cadf22078d5550a470ad5a526.png)

到了这里就配置完了物理资源了，点击确定

然后接着点击连接，进入操作系统安装

![image-20220114175942148](https://i-blog.csdnimg.cn/blog_migrate/f3cebc088ad57c0f4cf6054c219d4c41.png)

需要点击启动，来启动虚拟机

![image-20220114180002325](https://i-blog.csdnimg.cn/blog_migrate/0b8153db2ea9bc0b591313ff1b481dba.png)

你可以在Hyper-V管理中修改物理资源配置。

## 安装linux系统

选择安装操作系统

![image-20220114180030598](https://i-blog.csdnimg.cn/blog_migrate/a5df3d8a6ada9e20ea3939ea7197514f.png)

然后会进行一系列的检查等操作，检查通过后，就会展示可视化界面选择语言

![image-20220114180126714](https://i-blog.csdnimg.cn/blog_migrate/bb14b6a44450639ef2df8742bc29e6dc.png)

然后选择安装位置

![image-20220114180216182](https://i-blog.csdnimg.cn/blog_migrate/13b4b20ed8b7f8034f9b21d35bf2076b.png)

点进去什么都不需要操作，点击完成即可

![image-20220114180353230](https://i-blog.csdnimg.cn/blog_migrate/410017800832484d1d8b460cb1c52989.png)

然后选择开始安装，这里因为下载的就是最小的包，安装也是默认最小安装即可

![image-20220114180445704](https://i-blog.csdnimg.cn/blog_migrate/b94a1fc4bc5127b126c66c50a287c083.png)

在安装的过程中，设置root密码

![image-20220114180524331](https://i-blog.csdnimg.cn/blog_migrate/d05601ac993b526eb4346bc8ac19d0cc.png)

为了简单，设置为123456，简单密码需要点击两次完成

![image-20220114180559237](https://i-blog.csdnimg.cn/blog_migrate/e26db38ec9f6e046e330f056f7b5d556.png)

![image-20220114180626698](https://i-blog.csdnimg.cn/blog_migrate/59c2102bdbbc7641c57137ba93cd74ae.png)

然后等待安装完成即可

![image-20220114180655759](https://i-blog.csdnimg.cn/blog_migrate/486abd4fe738e6703f882f2494f8de1a.png)

安装完成重启就好了

![image-20220114182415914](https://i-blog.csdnimg.cn/blog_migrate/1a10bc5d2916660d28dfc04272b5447c.png)

重启后就进入系统了

![image-20220114182509219](https://i-blog.csdnimg.cn/blog_migrate/49bd97872ca42411a0139d831dbc058d.png)

登录root，即可进入

![image-20220114182544644](https://i-blog.csdnimg.cn/blog_migrate/fb3aea073b620efdc6b11be17e179d8b.png)

别忘记安装完操作系统后，把挂载的操作系统镜像弹出，要不然每次启动虚拟机，都会挂载，也会影响启动速度。

取消挂载是在设置中，移除硬件资源即可(需要在虚拟机关机的状态下修改配置)

## 工具连接

首先新增虚拟交换机

![image-20220114235650681](https://i-blog.csdnimg.cn/blog_migrate/572dcdbacf083f4ac409ae80f0541a80.png)

记得是创建内部类型的交换机

![image-20220114204405361](https://i-blog.csdnimg.cn/blog_migrate/0a270a7fd0df2f6a64a57f8b6edaadd7.png)

设置名字后确定

![image-20220114204502379](https://i-blog.csdnimg.cn/blog_migrate/7c96a0bf603d7e5ab935cb83cbd11632.png)

此时打开物理机的网络适配器

![image-20220114204607041](https://i-blog.csdnimg.cn/blog_migrate/52cbd37c78ad72b1a2c5f43ba20e77ec.png)

![image-20220114204619443](https://i-blog.csdnimg.cn/blog_migrate/422195c20ed8ba1c694925afe7bb3134.png)

找到物理机使用的网卡

![image-20220114235757778](https://i-blog.csdnimg.cn/blog_migrate/6aabb19f91865df83d797bd7eee6cdfe.png)

打开【属性】-【共享】，选择刚才创建的网卡共享网络

![image-20220114235840889](https://i-blog.csdnimg.cn/blog_migrate/74f2551a5240bf7455038f5128bd8229.png)

中间会提示一些信息，不管他确定。

确定之后，被共享的网卡的ip为192.168.137.1，这里千万不要修改。网上很多说这里可以随便写，你随便写了，就会导致虚拟机无法链接外网。这是windows定的一个nat转换地址，当然修改注册表可以修改这个ip，不过我没有尝试。

![image-20220115000058241](https://i-blog.csdnimg.cn/blog_migrate/4bc7f51938eb33b235fff26687b50ae3.png)

除此之外，还需要关闭硬件网卡的一些校验和检测，用于增加网卡性能【属性】-【配置】-【高级】下的IPv4校验和关闭，TCP硬件校验和关闭，UDP硬件校验和关闭。

![image-20220115000301035](https://i-blog.csdnimg.cn/blog_migrate/579eaa91d3cb39c4a2854f567a86a166.png)

物理机设置完了，还需要设置虚拟机

使用root登录虚拟机，然后切换到`/etc/sysconfig/network-scrpts/`目录下，然后使用`vi ifcfg-eth0`编辑网络配置。

主要修改这些内容
    
    
    BOOTPROTO=static
    ONBOOT=yes
    IPADDR=192.168.137.101 # 网址可以随便写，但是需要是在192.168.137.1网段下的
    NETMASK=255.255.255.0
    GATEWAY=192.168.137.1
    DNS1=114.114.114.114
    DNS2=8.8.8.8
    

修改完保存退出

![image-20220115000720035](https://i-blog.csdnimg.cn/blog_migrate/6001c4d1abee80309e48dd5f15af0e95.png)

然后使用`systemctl restart network`重启网络服务

然后使用`ip addr`查看网络信息

![image-20220115000805177](https://i-blog.csdnimg.cn/blog_migrate/18e2bbec008974d46ce50e650ce0d15c.png)

接着使用`ping www.baidu.com`验证外网是否可用

![image-20220115000856760](https://i-blog.csdnimg.cn/blog_migrate/dc22e0f35276c5b661134c278d027919.png)

然后在物理机上使用cmd的命令`ping 192.168.137.101`ping虚拟机

![image-20220115000938581](https://i-blog.csdnimg.cn/blog_migrate/e1deb56c4271dc72fb05b2179ff7c38f.png)

需要注意的是，这里一个单向的交换机，物理机可以ping虚拟机，但是虚拟机不能ping物理机。

最后我们使用xshell工具连接虚拟机(虚拟机连接是可以关闭的，关闭虚拟机连接后，虚拟机还是启动的)

![image-20220115001111089](https://i-blog.csdnimg.cn/blog_migrate/bd38a75e4dc3f2a34f9cf2c7101d1b8b.png)

xshell可以在[XSHELL - NetSarang Website](<https://www.xshell.com/zh/xshell/>)下载，我们选择家庭和学校的免费版。填写你的邮箱后，会给你的邮箱发送下载链接，下载即可。(下载比较慢，科学上网能快点)

下载安装后打开，连接虚拟机

![image-20220115001242114](https://i-blog.csdnimg.cn/blog_migrate/2ccab2c94ec7c3fb866db9050b7f8eea.png)

## 多个虚拟机

我自己的笔记本是16G内存的，而且初始分配的硬盘比较小(因为随时可以调整)

所以我打算创建3个虚拟机，一起启动，做到虚拟机互联，访问外网，固定ip，物理机使用xshell访问三个虚拟机。

首先创建3个虚拟机，配置都相同分别为`hadoop01,hadoop02,hadoop03`

这些虚拟机使用同一个网络交换机hadoop

安装操作系统的时候，设置root密码也相同123456

(我之前创建的时候ip错位了，101给了hadoop02，可以重新设置ip,给hadoop02设置102，hadoop01设置101，hadoop03设置103)

在设置网络的使用，同时设置主机名`vi /etc/hostname`，设置完成后将主机名进行映射，不仅仅映射自己，还需要映射其他的虚拟机。

`vi /etc/hosts`

![image-20220115005830124](https://i-blog.csdnimg.cn/blog_migrate/714cbd038a9a2ac7daed0be6616d0380.png)

全部虚拟机这样配置后，就可以相互访问了

![image-20220115010053710](https://i-blog.csdnimg.cn/blog_migrate/29236fddf10bc0d5d46b3ed09a717070.png)

![image-20220115010116396](https://i-blog.csdnimg.cn/blog_migrate/1e4c47ab880cc9a07094038e42bb3dff.png)

![image-20220115010142714](https://i-blog.csdnimg.cn/blog_migrate/d884ee2a830bf4095e3259567d5c15ea.png)

## 总结

总的来说还是很不错的，除了网络设置有点难度(其实也不算难，主要是网上的资料不正确)，其他的都比较容易。

而且占用的资源也算还行，并不是很大。

![image-20220115010318970](https://i-blog.csdnimg.cn/blog_migrate/8ebe2266ff4399efd146d24cd82299a3.png)

自己学习啥的，也够折腾了。

== 2022-01-22 更新==

## 直接使用物理机的网卡–解决虚拟机网络慢的问题

之前使用内网的网络交换设备，然后将物理机外网网卡共享给内网的网络交换设备，虽然也能使用，但是存在一个非常大的问题：**网络慢** 。  
这个问题真的好恶心，物理机使用xshell连接虚拟机，都需要将近2分钟才能连上。  
这也太慢了吧。  
想到物理网卡可以共享网络给内网网络交换机，那么为什么不让虚拟机直接使用物理网卡呢？  
说干就干。  
首先在hyper-v的网络交换设置中增加物理网卡的虚拟网卡，其实也是虚拟网卡，只是对外的  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/875194d270360b654d3c1c5be9f897d4.png)  
然后给虚拟机更换为这个网络交换机  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/edc75d3c3d33d36394493c8f6bad6ac7.png)  
然后重新启动虚拟机，使用hyper-v自己的连接窗口连接虚拟机，此时之前可用的xshell的连接可能已经无效了，无法连接了，所以需要使用hyper-v自己的连接工具。  
连接上后需要修改ip地址，修改为和物理机相同网段的地址  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/53597dedf180393fed565e62e35d6b54.png)  
然后修改`/etc/hosts`文件，以及物理机的hosts文件。  
理论上物理机的hosts不用修改，虚拟机的也不用修改，因为都是同一个网段，使用同一个网卡，连接的同一个路由器。  
如果无法互通，那么配置`/etc/hsots`可能是一个解决方案。  
刚配置上，网速还是挺快的  
不知道会不会和虚拟内网一样，用一段时间就变慢了。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/49785e9cda8e5898e66aa4b490ee3616.png)