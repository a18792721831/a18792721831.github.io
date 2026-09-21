---
layout: post
title: "MySQL Undo Log"
date: 2025-08-03 14:33:46 +0800
categories: [mysql, 数据库, Undo Log, 回滚日志, 日志]
description: "MySQL Undo Log是InnoDB实现事务回滚和MVCC机制的关键组件。它记录事务修改前的数据，包含四种格式：INSERT、UPDATE现有记录、UPDATE已删除记录和DELETE标记记录。Undo Log由Purge线程负责清理，配置参数包括回滚段数量、日志文件大小限制等。与Redo Log不同，Undo Log用于事务回滚和非锁定读取，而Redo Log确保事务持久性。在MySQL启动时，系统会先恢复Redo Log，再处理Undo Log进行回滚操作。通过合理配置Undo Log参数，可以优"
keywords: mysql, 数据库, Undo Log, 回滚日志, 日志
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/149880355
> - 发布时间：2025-08-03 14:33:46
> - 阅读量：1
> - 分类：mysql专栏收录该内容, 订阅专栏
> - 标签：#mysql, #数据库, #Undo Log, #回滚日志, #日志

## 摘要

文章浏览阅读1k次，点赞27次，收藏24次。MySQL Undo Log是InnoDB实现事务回滚和MVCC机制的关键组件。它记录事务修改前的数据，包含四种格式：INSERT、UPDATE现有记录、UPDATE已删除记录和DELETE标记记录。Undo Log由Purge线程负责清理，配置参数包括回滚段数量、日志文件大小限制等。与Redo Log不同，Undo Log用于事务回滚和非锁定读取，而Redo Log确保事务持久性。在MySQL启动时，系统会先恢复Redo Log，再处理Undo Log进行回滚操作。通过合理配置Undo Log参数，可以优

---

MySQL Undo Log

MySQL主从复制：https://blog.csdn.net/a18792721831/article/details/146117935  
MySQL Binlog：https://blog.csdn.net/a18792721831/article/details/146606305  
MySQL General Log：https://blog.csdn.net/a18792721831/article/details/146607343  
MySQL Slow Log：https://blog.csdn.net/a18792721831/article/details/147166971  
MySQL Error Log：https://blog.csdn.net/a18792721831/article/details/147167038  
MySQL Redo Log: https://mp.csdn.net/mp_blog/creation/success/149862528  
MySQL Undo Log: https://blog.csdn.net/a18792721831/article/details/149880355

#### MySQL Undo Log

  * 介绍
  * Undo Log 的 Purge
  * 两种 Undo Log
  * Undo Log 的格式
  * 回滚
  * 配置
  * Binlog 和 Redo Log 的区别
  * Undo Log 和 Redo Log 的区别

## 介绍

Redo Log 记录了事务操作的变化。当事务发生回滚时，会记录到Undo Log 中。

Undo Log 是单个事务所匹配的撤销日志记录的集合。Undo Log 包含事务版本号和修改前的记录，应用于 MVCC.

当用于读取一行记录时，如果该记录已经被其他事务占用，那么当前事务可以通过Undo Log 读取之前的行版本信息，因为没有事务需要对旧数据进行修改操作，所以也不需要加锁，以此实现非锁定读取。

## Undo Log 的 Purge

当事务提交时，InnoDB不会立即删除 Undo Log ，其他事务读取的是开启事务时最新提交的行版本信息，只要该事物不结束，就不能删除该行版本。

但是，事务提交时会放入待清理的链表，由 Purge 线程判断是否有其他事务在使用 undo 段中的上一个事务之前的版本，并决定是否可以清理 Undo Log 的日志文件。

## 两种 Undo Log

在事务执行过程中会产生两种 Undo Log

  * insert 的 Undo Log: insert 是不需要 Purge 线程的，只要事务提交了，就可以丢掉回滚记录。

  * update 的 Undo Log记录(delete也包含)：delete实际上不会执行删除操作，而是将delete 对象打上 delete flag 标记，最终的delete操作是由Purge线程完成的。

update Undo Log 分为两种情况，判断update 的列是否为主键：

如果不是主键的列，则在 Undo Log 中直接反向记录是如何 update 的，update 是直接进行的。

如果是主键列，则 update 分两步执行，先删除旧的行，在插入新的数据。

## Undo Log 的格式

Undo Log 的格式有以下4种：

  * TRX_UNDO_INSERT_REC: 记录插入时的Undo Log 类型，只记录了表ID及主键信息。在回滚时，只需要通过主键在原 B+ 树中找到对应的记录进行删除。
  * TRX_UNDO_UPD_EXIST_REC:更新一条存在记录的 Undo Log，会记录表ID、主键、每个被更新的列的原始值和新值。在恢复时，根据主键信息修改成原来的值。
  * TRX_UNDO_UPD_DEL_REC:更新一条已经打了删除标志记录的 Undo Log类型，与 TRX_UNDO_UPD_EXIST_REC 相似。
  * TRX_UNDO_DEL_MARK_REC:在删除记录时，对记录打删除标志的Undo Log 类型，只记录了表ID及主键信息，用来在回滚或Purge时找到对应的记录。

## 回滚

在 Mysql 启动过程中会进行如下步骤：

  * 恢复Redo Log
  * 在Undo Log 中，将回滚段的 undo 扫描出来并缓存
  * 依次处理每个 undo 段，根据 undo 段的状态决定后面的措施
  * 根据TRX_UNDO_TRX_NO按照从大到小的顺序排序
  * 执行与回滚相关的操作

## 配置

`innodb_rollback_segments` 参数配置回滚段的数量，范围[1,128]

`innodb_undo_directory`参数指定 Undo 表空间所在的目录，默认为数据目录。

`innodb_max_undo_log_size`参数表示每个 Undo Log对应的日志文件的最大值。

`innodb_undo_tablespaces`参数表示 InnoDB使用的 Undo 表空间的数量，从 mysql 8.0.14 开始，该参数不能配置，未来移除该参数。

当启用`innodb_undo_log_truncate`参数时，如果超过 `innodb_max_undo_log_size`参数配置的值，那么Undo 表空间将被标为截断，只有独立的 Undo 表空间可以被截断，在system表空间中的 Undo Log 不可以被截断。如果配置为 `ON`，并且配置了两个或者两个以上的Undo 表空间数据文件，当某个日志文件的大小超过设置的最大值之后，就会自动收缩表空间数据文件。

`innodb_undo_tablespaces`参数的值必须大于或等于2才能设置`innodb_undo_log_truncate`为 `ON`。

`innodb_purge_rseg_truncate_frequency`参数指定 Purge 操作被唤醒多少次之后才释放回滚段。当Undo 表空间中的回滚段被释放时，Undo 表空间才会被截断。因此如果`innodb_purge_rseg_truncate_frequency`参数设置的越小，Undo 表空间被尝试截断的频率越高。

![image-20250803120237413](https://i-blog.csdnimg.cn/img_convert/c8589efcbdc4d46e14014daa257e9c55.png)

在目录下新建 undolog目录，并设置权限

![image-20250803120507377](https://i-blog.csdnimg.cn/img_convert/92bc74196afebc04455337bb6d11b0fc.png)

在配置文件中加入`innodb_undo_directory=/var/lib/mysql-undolog`配置

![image-20250803120452044](https://i-blog.csdnimg.cn/img_convert/72ba49c2fb098444e204b0f0b3b15263.png)

使用如下命令挂载并启动
    
    
    docker run \
    --name master \
    -e MYSQL_ROOT_PASSWORD=master \
    -v /data/mysql/master/log:/var/log/mysql \
    -v /data/mysql/master/data:/var/lib/mysql \
    -v /data/mysql/master/conf:/etc/mysql/conf.d \
    -v /data/mysql/master/binlog:/var/lib/mysql-bin \
    -v /data/mysql/master/redolog:/var/lib/mysql-redolog \
    -v /data/mysql/master/undolog:/var/lib/mysql-undolog \
    -v /data/mysql/master/redolog_archive:/var/lib/mysql-redolog-archive \
    -p 3106:3306 \
    -d \
    mysql:8.0 
    

> 记得删除原来 data 目录下的 undo_00* 文件，要不然会启动失败。
> 
> 因为使用 docker stop master 停止的，所以是安全关闭的mysql，此时可以认为undo为空(因为所有的链接将被关闭，未提交事务认为回滚)

![image-20250803121934492](https://i-blog.csdnimg.cn/img_convert/4f4b746276ac33c17db8d3bdf856a475.png)

在指定的目录下会产生新的Undo Log

![image-20250803143226791](https://i-blog.csdnimg.cn/img_convert/505afa4cb1612c37ec64e72af57f2dd4.png)

## Binlog 和 Redo Log 的区别

Redo Log 是在 Inno DB 层产生的。Binlog 不单单记录 Inno DB 的修改，也记录其他任何存储引擎的修改。

Redo Log 是物理逻辑格式的日志，记录的是每页的修改。Binlog 是一种逻辑日志，记录的是对应变更的SQL。

## Undo Log 和 Redo Log 的区别

Redo Log 用来保证事务的持久性，Undo Log 用来帮助事务回滚和MVCC实现。

从写入顺序来说，Redo Log 是顺序写的， Undo Log 是随机读写的。
