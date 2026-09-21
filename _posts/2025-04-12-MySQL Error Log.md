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

![image-20250412154555655](https://i-blog.csdnimg.cn/img_convert/cd5d001ff5fd23353381dbf94741a0a9.png)

使用 `set global log_error="/var/log/mysql/mysql-error.log";` 调整为文件记录Error Log

![image-20250412154813607](https://i-blog.csdnimg.cn/img_convert/9a0945ac00d88ffa9be62bb2c5c2f262.png)

报错了，说明 `log_error`是一个只读参数，在mysql启动的时候就会固定下来

因此需要再 my.cnf文件中进行设置

![image-20250412155002990](https://i-blog.csdnimg.cn/img_convert/61246d9f4dd3239f6c96f800c2403241.png)

然后重启

![image-20250412155041012](https://i-blog.csdnimg.cn/img_convert/d2a779472133f31d9a567c5cbf1c449f.png)

然后再次查看

![image-20250412155059006](https://i-blog.csdnimg.cn/img_convert/a58866b975b81df349f993fde2f2ad73.png)

启动成功

## Error Log 查看

查看Error Log 日志文件

![image-20250412155214061](https://i-blog.csdnimg.cn/img_convert/27d134a5124199a3a5ce12b18b03bf2f.png)

在MySQL启动过程中的一些错误信息，就会写入到这里，这些日志在启动MySQL的时候，会展示在标准输出中

## Error Log 滚动

使用`mv mysql-error.log mysql-error.log.1` 将现在错误日志文件进行归档

![image-20250412161241266](https://i-blog.csdnimg.cn/img_convert/f0ad8ba884357320dba33a85c94f1d18.png)

然后使用 `flush logs` 进行重新生成mysql-error.log 文件，如果不提前将mysql-error.log进行归档，那么就是清理 Error Log

![image-20250412161412731](https://i-blog.csdnimg.cn/img_convert/9f55c539545b8b0cde601b610c551b53.png)

查看是否重新生成mysql-error.log

![image-20250412161446221](https://i-blog.csdnimg.cn/img_convert/dabb2a6e362f0d8bd4366956e4c67027.png)
