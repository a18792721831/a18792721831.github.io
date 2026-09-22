---
layout: post
title: "redis 安全security"
date: 2020-07-23 15:52:49 +0800
categories: ["2020"]
description: "本文详细介绍了Redis的安全措施，包括常规安全模式、网络安全配置、轻量级认证及命令禁用策略，帮助用户理解如何保护Redis免受未授权访问。"
keywords: 2020
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107539190
> - 发布时间：2020-07-23 15:52:49
> - 阅读量：720
> - 分类：Redis专栏收录该内容, 订阅专栏

## 摘要

文章浏览阅读720次。本文详细介绍了Redis的安全措施，包括常规安全模式、网络安全配置、轻量级认证及命令禁用策略，帮助用户理解如何保护Redis免受未授权访问。

---

#### redis 安全security

  * 1\. redis 常规安全模式
  * 2\. 网络安全
  * 3\. 轻量认证
  * 4\. 命令的禁用

## 1\. redis 常规安全模式

redis被设计成仅有可信环境下的可信用户才可以访问。这意味着将redis实例直接暴露在网络上或者让不可信用户直接访问redis的是不安全的。

换句话说，redis不保证redis的安全，而是由使用者保证的。

## 2\. 网络安全

在redis的配置文件中，可以配置bind的ip,让redis只接受绑定的ip的客户端。

一般来说，如果redis需要禁止外部访问，只需bind本地IP即可。

`bind 127.0.0.1`

也可以绑定指定的网段：

`bind 192.168.1.100 10.0.0.1`

## 3\. 轻量认证

![image-20200723150834768](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/707e882d25688d26c3a810d18cb90cd8.png)

在redis的配置文件中可以配置

`requirepass password`来配置验证密码。

当配置了验证密码后，在开始执行其他命令之前需要执行`auth <password>`来验证。

![image-20200723151336634](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/17a3de1662c0b031e90fe102e76b9849.png)

使用`config get requ*`获取配置信息

发现有两个返回，分别是`requirepass`和空，意思是，requirepass现在的配置为空，表示不启用

使用`config set requirepass mypassword`设置密码是mypassword

然后我们退出客户端，然后重新登录

![image-20200723151646686](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ef5683a1c1a461448a855e2eb27d8310.png)

连接是没有问题的，但是却无法执行任何命令。

![image-20200723151728006](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/665977cde560605cd8db849ee091f764.png)

使用auth验证通过后，才能执行redis的命令。

但是这种方式也存在问题：

  * 密码在配置文件中是明文的。
  * redis的查询速度非常快，所以，使用暴力破解是可能的

![image-20200723151941808](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/16aab946de69d19000b215a9ed304b22.png)

redis每秒可以尝试15W的密码，所以如果密码长度比较短的话，很短时间就会被暴力破解。

> 假设密码是6位，那么全部的密码组合共有(26+26+10+33)^6, 每一位密码的可能性：大小写字符，数字，可打印符号。总共6位。所以6位密码总共有95^6种。

![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/489afc6502bc402454f8c2300efbba2e.png)

735 091 890 625 / 150000 = 4 900 612.6 秒 = 81 676.88 分 = 1361.28时 = 56.72天

看起来还不错吧，假设一个机器有4个核心，8线程。那么在一个机器上同时开8个客户端，每个客户端负责尝试指定范围的密码组合。

56.72 / 8 = 7.09 天

如果有多台机器，那么速度会更快。假设有100台机器，每台的机器配置都一样：

7.09 / 100 = 15.696 分钟

你设置的密码，在强大的算力下，只需15分钟的时间就会被破解。

当然，这只是理论上的，实际上还需要考虑网络，多线程切换等因素影响。

AUTH 命令在网络中是明文传输的。

## 4\. 命令的禁用

如果你登录了redis，意味着，redis中全部的命令你都能使用。对于普通用户来说，使用一些高危的操作很危险，所以需要对普通用户禁用一些命令。

redis是通过命令重命名来实现命令禁用的

`rename-command source-command dest-command`

通过配置`rename-command`将`source-command`重命名为`dest-command`

后面想要使用`source-command`只能输入`dest-command`来使用了。

对于普通用户来说，不知道`dest-command`，就无法使用`source-command`了。

如果`dest-command`设置为空，表示任何人都无法使用`source-command`了。

![image-20200723155001756](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4ca007dad0a3d943f2bd910301be17f5.png)
