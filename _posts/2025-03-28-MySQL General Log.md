---
layout: post
title: "MySQL General Log"
date: 2025-03-28 17:58:28 +0800
categories: [mysql, 数据库, General Log, log_output]
description: "MySQL主从复制：https://blog.csdn.net/a18792721831/article/details/146117935Binlog 的特点是只记录数据修改语句，有时可能需要记录客户端执行的每条SQL语句，General Log 会记录所有的SQL。_mysql 开启generallog"
keywords: mysql, 数据库, General Log, log_output
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/146607343
> - 发布时间：2025-03-28 17:58:28
> - 阅读量：652
> - 分类：mysql专栏收录该内容, 订阅专栏
> - 标签：#mysql, #数据库, #General Log, #log_output

## 摘要

文章浏览阅读652次，点赞4次，收藏5次。MySQL主从复制：https://blog.csdn.net/a18792721831/article/details/146117935Binlog 的特点是只记录数据修改语句，有时可能需要记录客户端执行的每条SQL语句，General Log 会记录所有的SQL。_mysql 开启generallog

---

MySQL General Log  

#### MySQL General Log

  * General Log 的开启
  * General Log 的用法
  * log_output 参数

MySQL主从复制：https://blog.csdn.net/a18792721831/article/details/146117935  
MySQL Binlog：https://blog.csdn.net/a18792721831/article/details/146606305  
MySQL General Log：https://blog.csdn.net/a18792721831/article/details/146607343  
MySQL Slow Log：https://blog.csdn.net/a18792721831/article/details/147166971  
MySQL Error Log：https://blog.csdn.net/a18792721831/article/details/147167038  
MySQL Redo Log: https://blog.csdn.net/a18792721831/article/details/149862528  
MySQL Undo Log: https://blog.csdn.net/a18792721831/article/details/149880355

Binlog 的特点是只记录数据修改语句，有时可能需要记录客户端执行的每条SQL语句，General Log 会记录所有的SQL

## General Log 的开启

使用 `select @@general_log;`查看

![image-20250328172331381](https://i-blog.csdnimg.cn/img_convert/b6b7d3edf2ddc62b44fdf0a23e43fb1d.png)

使用 `set glogbal general_log_file = "/var/log/mysql/general_log.log"`设置`general_log`的保存位置

使用`set global general_log=on;`开启`general_log`

![image-20250328172748008](https://i-blog.csdnimg.cn/img_convert/0fb3ce8fc5d8e602c540ca2627b53c2c.png)

在服务端查看是否有`general_log`

![image-20250328172829488](https://i-blog.csdnimg.cn/img_convert/f9c0d0b71ebada08cf0473c1e8cc9298.png)

## General Log 的用法

执行一句SQL`select 'test_general_log';`查看是否被记录到了`general_log`中了

![image-20250328172914757](https://i-blog.csdnimg.cn/img_convert/e2dfb7e60abecf31b676d3d9b1469e4d.png)

![image-20250328172928636](https://i-blog.csdnimg.cn/img_convert/51610a8e7d6f03dcfff9024c43da3bfa.png)

一模一样。

如果需要永久生效，需要在配置文件的`[mysqld]`中配置
    
    
    general_log=on
    general_log_file=/var/log/mysql/general_log.log
    

## log_output 参数

另外，可以定义 `General_log`的输出方式，并由`log_output`参数控制(该参数对Slow Log 同样生效)。

`log_output`的几个值对应的效果：

  * TABLE:将记录保存在表中
  * FILE:讲记录保存在日志文件中
  * NONE:禁用日志记录

> TABLE和FILE可以同时开启

![image-20250328173444342](https://i-blog.csdnimg.cn/img_convert/60ee1ff20cc8043a93a7a2c8740baf83.png)

调整为 `TABLE`

![image-20250328173527261](https://i-blog.csdnimg.cn/img_convert/b66d8d5b318f8f87a6ff0fd696b41d4e.png)

执行一条sql

![image-20250328173711768](https://i-blog.csdnimg.cn/img_convert/c880669c3735fcad1ce4e8d3538c66fe.png)

查看file

![image-20250328173725801](https://i-blog.csdnimg.cn/img_convert/0e44275c673a3b01fd984e3d77427472.png)

之后的就不在记录了

设置同时记录

`set global log_output="TABLE,FILE";`

![image-20250328173858559](https://i-blog.csdnimg.cn/img_convert/41912eb344d4cc86a85a0494cbcd5f39.png)

查看file

![image-20250328173929317](https://i-blog.csdnimg.cn/img_convert/3c6d0d00ab11ccc97489322b71b30a48.png)
