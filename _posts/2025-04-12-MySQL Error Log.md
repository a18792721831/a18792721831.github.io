---
layout: post
title: "MySQL Error Log"
date: 2025-04-12 16:24:18 +0800
categories: [mysql, 数据库, Error Log, 错误日志, 启动日志]
description: "MySQL 的 Error Log 不仅包含错误信息，还包含启动和关闭的一些记录。Error Log 也可以协助定位和解决问题。_mysql error log设置"
keywords: mysql, 数据库, Error Log, 错误日志, 启动日志
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/147167038
> - 发布时间：2025-04-12 16:24:18
> - 阅读量：466
> - 标签：#mysql, #数据库, #Error Log, #错误日志, #启动日志

## 摘要

文章浏览阅读466次，点赞4次，收藏5次。MySQL 的 Error Log 不仅包含错误信息，还包含启动和关闭的一些记录。Error Log 也可以协助定位和解决问题。_mysql error log设置

---

#### MySQL Error Log

  * Error Log 的开启
  * Error Log 查看
  * Error Log 滚动

  
MySQL Error Log 

MySQL主从复制：https://blog.csdn.net/a18792721831/article/details/146117935  
MySQL Binlog：https://blog.csdn.net/a18792721831/article/details/146606305  
MySQL General Log：https://blog.csdn.net/a18792721831/article/details/146607343  
MySQL Slow Log：https://blog.csdn.net/a18792721831/article/details/147166971  
MySQL Error Log：https://blog.csdn.net/a18792721831/article/details/147167038  
MySQL Redo Log: https://blog.csdn.net/a18792721831/article/details/149862528  
MySQL Undo Log: https://blog.csdn.net/a18792721831/article/details/149880355

MySQL 的 Error Log 不仅包含错误信息，还包含启动和关闭的一些记录。Error Log 也可以协助定位和解决问题

## Error Log 的开启

使用`show global variables like 'log_error';`查看

![image-20250412154555655](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412154555655.png)

使用 `set global log_error="/var/log/mysql/mysql-error.log";` 调整为文件记录Error Log

![image-20250412154813607](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412154813607.png)

报错了，说明 `log_error`是一个只读参数，在mysql启动的时候就会固定下来

因此需要再 my.cnf文件中进行设置

![image-20250412155002990](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412155002990.png)

然后重启

![image-20250412155041012](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412155041012.png)

然后再次查看

![image-20250412155059006](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412155059006.png)

启动成功

## Error Log 查看

查看Error Log 日志文件

![image-20250412155214061](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412155214061.png)

在MySQL启动过程中的一些错误信息，就会写入到这里，这些日志在启动MySQL的时候，会展示在标准输出中

## Error Log 滚动

使用`mv mysql-error.log mysql-error.log.1` 将现在错误日志文件进行归档

![image-20250412161241266](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412161241266.png)

然后使用 `flush logs` 进行重新生成mysql-error.log 文件，如果不提前将mysql-error.log进行归档，那么就是清理 Error Log

![image-20250412161412731](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412161412731.png)

查看是否重新生成mysql-error.log

![image-20250412161446221](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412161446221.png)
