---
layout: post
title: "MySQL Binlog"
date: 2025-03-28 17:20:44 +0800
categories: [mysql, 数据库, Binlog, Binlog解析, Binlog加密, Binlog参数]
description: "Binlog 包含描述数据库修改的语句，如 create , update 等数据变更语句，不会记录类似 select , show 等不修改数据的语句。如果想记录所有的 SQL 语句，可以使用 General Log。_mysql binlog"
keywords: mysql, 数据库, Binlog, Binlog解析, Binlog加密, Binlog参数
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/146606305
> - 发布时间：2025-03-28 17:20:44
> - 阅读量：1
> - 分类：mysql专栏收录该内容, 订阅专栏
> - 标签：#mysql, #数据库, #Binlog, #Binlog解析, #Binlog加密, #Binlog参数

## 摘要

文章浏览阅读1.1k次，点赞18次，收藏8次。Binlog 包含描述数据库修改的语句，如 create , update 等数据变更语句，不会记录类似 select , show 等不修改数据的语句。如果想记录所有的 SQL 语句，可以使用 General Log。_mysql binlog

---

MySQL Binlog  

#### MySQL Binlog

  * 介绍
  * 查看 Binlog 位点
  * 开启和关闭 Binlog
  * Binlog 的作用
  * Binlog 记录的格式
  * Binlog 的解析
  * Binlog 加密
  * Binlog 的清理
  *     * 根据Binlog文件名删除
    * 根据时间删除
  * Binlog 保留参数
  * Binlog 的落盘
  * Binlog 相关参数

MySQL主从复制：https://blog.csdn.net/a18792721831/article/details/146117935  
MySQL Binlog：https://blog.csdn.net/a18792721831/article/details/146606305  
MySQL General Log：https://blog.csdn.net/a18792721831/article/details/146607343  
MySQL Slow Log：https://blog.csdn.net/a18792721831/article/details/147166971  
MySQL Error Log：https://blog.csdn.net/a18792721831/article/details/147167038  
MySQL Redo Log: https://blog.csdn.net/a18792721831/article/details/149862528  
MySQL Undo Log: https://blog.csdn.net/a18792721831/article/details/149880355

## 介绍

Binlog 包含描述数据库修改的语句，如 create , update 等数据变更语句，不会记录类似 select , show 等不修改数据的语句。

如果想记录所有的 SQL 语句，可以使用 General Log

## 查看 Binlog 位点

使用 `show master status` 查看当前Binlog的位点

![image-20250310183401493](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250310183401493.png)

创建databases,并创建一张表
    
    
    create database test;
    use test;
    create table t(id int, name varchar(256));
    

然后在查看Binlog的位点信息

![image-20250310183551459](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250310183551459.png)

其 Position 从2427增加到 2818。

使用 `show binlog events in 'mysql-bin.000004' from 2427 \G` 查看从 2427 开始发生了什么操作

![image-20250310183847224](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250310183847224.png)

创建database和table 的sql都被记录到 Binlog 中了。

试着查询一下呢

![image-20250310184029054](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250310184029054.png)

发现执行了 select 的语句，但是 Position 并没有增加。

## 开启和关闭 Binlog

开启Binlog ，需要在配置文件中的 `[mysqld]`中加上如下语句：
    
    
    log-bin = /data/mysql/binlog/mysql-bin
    

表示Binlog的存放路径为`/data/mysql/binlog` 文件名为`mysql-bin`后接Binlog的序列号

比如 `my.cfg`文件如下：
    
    
    [mysqld]
    server_id=100
    log_bin=/var/lib/mysql-bin/mysql-bin
    binlog_format=row
    plugin-load-add=mysql_native_password.so
    authentication_policy=mysql_native_password
    

我是使用docker 启动的，挂载如下：
    
    
    docker run \
    --name master \
    -e MYSQL_ROOT_PASSWORD=master \
    -v /data/mysql/master/log:/var/log/mysql \
    -v /data/mysql/master/data:/var/lib/mysql \
    -v /data/mysql/master/conf:/etc/mysql/conf.d \
    -v /data/mysql/master/binlog:/var/lib/mysql-bin \
    -p 3106:3306 \
    -d \
    mysql:8.0
    

表示将 容器内的 `/var/lib/mysql-bin` 目录挂载到`/data/mysql/master/binlog`目录，在配置文件中指定Binlog日志存储目录是 `/var/lib/mysql-bin`目录，文件名是 `mysql-bin`后接序列号

![Clipboard_Screenshot_1741603687](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1741603687.png)

为了跟踪使用了那些Binlog文件，mysqld还创建了一个Binlog索引文件，其中包含Binlog文件的名称。默认情况下，该名称与Binlog文件具有相同的基本名称，扩展名为`.index`。比如上面的 `mysql-bin.index`。

![image-20250311142515832](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250311142515832.png)

如果没有指定Binlog的文件名和路径，那么在mysqld中的配置为空

`log-bin`

默认存放在datadir下，Binlog的文件名为主机名后接 Binlog 的序列号。

一般建议指定一个基本名称，防止更改主机名时出现 Binlog 的文件名与之前不一致的现象。

关闭 Binlog， 需要在配置文件中加上
    
    
    skip_log_bin
    

或者
    
    
    disable_log_bin
    

如果要关闭当前会话的 Binlog，可以执行如下语句
    
    
    set sql_log_bin=0;
    

## Binlog 的作用

复制：主库的变更先写入Binlog，然后传到从库进行回放。

灾备：当误操作后，可以先把全备导入某个新的实例中，然后通过全备时间点到误操作中间的 Binlog 解析出所有事务（排除误操作的事务），并在新实例中执行这些事务，达到恢复到误操作前一刻的状态。

## Binlog 记录的格式

Binlog 可以设置为以下几种日志格式

  * Statement(基于SQL语句的格式)：每条会修改数据的SQL语句都会记录在 Binlog 中，不需要记录每行的变化。
  * Row(基于行)：会非常清楚地记录每行数据被修改的细节。
  * Mixed(混合模式)：以上两种格式的混合采用，默认采用的 statement 格式保存Binlog，statement格式无法准确复制的操作可以使用row格式保存Binlog。

| 日志格式      | 优点                                     | 缺点                                       |
|-----------|----------------------------------------|------------------------------------------|
| statement | 日志量少，节约IO，性能高                          | 在主从复制中可能会导致主从数据不一致，比如使用了不确定函数，UUID()函数等等 |
| row       | 主从数据基本一致，支持闪回                          | 日志量多                                     |
| mixed     | 日志量少，节约IO,性能高，解决了statement格式部分数据不一致的情况 | 不支持闪回，不分高可用架构不支持该格式，不方便将数据同步到其他类型的数据库，   |

Binlog 记录的格式由参数 binlog_format 控制，如果要设置为 row格式，则在 `[mysqld]` 中加入如下语句

`binlog_format=row`

当然也支持动态修改，修改参数`binlog_format`的全局值

`set global binlog_format='row';`

修改参数`binlog_format`的会话级别的方法：

`set session binlog_format='row';`

![image-20250327191714860](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327191714860.png)

![image-20250327191823940](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327191823940.png)

![image-20250327191906986](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327191906986.png)

![image-20250327191937754](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327191937754.png)

![image-20250327192021973](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327192021973.png)

## Binlog 的解析

Binlog 文件不能直接查看，需要通过 mysqlbinlog 工具解析。

比如在 `row` 格式下，解析 Binlog 的方法

首先查看`binlog_format`是否为 `row`

![image-20250327193658060](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327193658060.png)

首先查看位点

![image-20250327193729589](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327193729589.png)

文件index=6,pos=157

创建一个表

![image-20250327193807721](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327193807721.png)

创建一个表，并且插入一行数据后，binlog的index=6，但是pos=362

接着解析binlog文件

`mysqlbinlog --start-position=157 mysql-bin.000006 -vv > ./log`

![image-20250327194322704](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327194322704.png)
    
    
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
    /*!40019 SET @@session.max_insert_delayed_threads=0*/;
    /*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
    DELIMITER /*!*/;
    # at 4
    #250327 19:36:22 server id 100  end_log_pos 126 CRC32 0xf96205bf        Start: binlog v 4, server v 8.0.41 created 250327 19:36:22 at startup
    # Warning: this binlog is either in use or was not closed properly.
    ROLLBACK/*!*/;
    BINLOG '
    NjjlZw9kAAAAegAAAH4AAAABAAQAOC4wLjQxAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    AAAAAAAAAAAAAAAAAAA2OOVnEwANAAgAAAAABAAEAAAAYgAEGggAAAAICAgCAAAACgoKKioAEjQA
    CigAAb8FYvk=
    '/*!*/;
    # at 157
    #250327 19:37:52 server id 100  end_log_pos 234 CRC32 0xb65c3b6b        Ignorable
    # Ignorable event type 34 (MySQL Anonymous_Gtid)
    # at 234
    #250327 19:37:52 server id 100  end_log_pos 362 CRC32 0xddef90fd        Query   thread_id=8     exec_time=0     error_code=0
    use `mysql`/*!*/;
    SET TIMESTAMP=1743075472/*!*/;
    SET @@session.pseudo_thread_id=8/*!*/;
    SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1, @@session.check_constraint_checks=1/*!*/;
    SET @@session.sql_mode=1168113696/*!*/;
    SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
    SET @@session.character_set_client=255,@@session.collation_connection=255,@@session.collation_server=255/*!*/;
    SET @@session.lc_time_names=0/*!*/;
    SET @@session.collation_database=DEFAULT/*!*/;
    create table test_t(name varchar(256))
    /*!*/;
    DELIMITER ;
    # End of log file
    ROLLBACK /* added by mysqlbinlog */;
    /*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
    

也可以直接 `mysqlbinlog mysql-bin.000006`

![image-20250327194456064](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327194456064.png)

## Binlog 加密

从Mysql 8.0.14 开始，可以对 Binlog 文件和中继日志文件进行加密，从而保护敏感数据。

可以在配置文件的`[mysqld]`中加上如下语句开启 Binlog 加密
    
    
    early-plugin-load=keyring_file.so
    keyring_file_data=/var/lib/mysql/keyring
    binlog_encryption=on
    

也可以通过`select @@binlog_encryption`查看

![image-20250327194808777](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327194808777.png)

修改`[mysqld]`

![image-20250327200129300](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327200129300.png)

再次查看

![image-20250327200205898](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327200205898.png)

使用 `show binary logs;` 查看

![image-20250327200229356](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327200229356.png)

可以发现最新的 binlog encryption 已经变成yes了

第一次配置过加密的插件后，后面可以通过 `set global binlog_encryption='off';`关闭或者`set global binlog_encryption='on';`开启

![image-20250327200502553](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327200502553.png)

![image-20250327200922428](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327200922428.png)

从上图可以看出，`mysql-bin.000009` 文件是加密,使用`mysqlbinlog`解析加密的binlog文件

![image-20250327201111292](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250327201111292.png)

已经解析不出来结果了。

应该使用MySQL的用户密码进行解析才行
    
    
    mysqlbinlog --read-from-remote-server \
    -hx.x.x.x \
    -Px \
    -uu \
    -pp \
    --start-position=157 \
    mysql-bin.000009 -vv >./log3
    

查看log3的内容
    
    
    # The proper term is pseudo_replica_mode, but we use this compatibility alias
    # to make the statement usable on server versions 8.0.24 and older.
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
    /*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
    DELIMITER /*!*/;
    # at 157
    #250327 20:08:16 server id 100  end_log_pos 0 CRC32 0x07906a17 	Start: binlog v 4, server v 8.0.41 created 250327 20:08:16
    BINLOG '
    sD/lZw9kAAAAegAAAAAAAAAAAAQAOC4wLjQxAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    AAAAAAAAAAAAAAAAAAAAAAAAEwANAAgAAAAABAAEAAAAYgAEGggAAAAICAgCAAAACgoKKioAEjQA
    CigAARdqkAc=
    '/*!*/;
    # at 157
    #250327 20:08:31 server id 100  end_log_pos 234 CRC32 0x17b798d5 	Anonymous_GTID	last_committed=0	sequence_number=1	rbr_only=no	original_committed_timestamp=1743077311135115	immediate_commit_timestamp=1743077311135115	transaction_length=206
    # original_commit_timestamp=1743077311135115 (2025-03-27 20:08:31.135115 CST)
    # immediate_commit_timestamp=1743077311135115 (2025-03-27 20:08:31.135115 CST)
    /*!80001 SET @@session.original_commit_timestamp=1743077311135115*//*!*/;
    /*!80014 SET @@session.original_server_version=80041*//*!*/;
    /*!80014 SET @@session.immediate_server_version=80041*//*!*/;
    SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
    # at 234
    #250327 20:08:31 server id 100  end_log_pos 363 CRC32 0x7d250931 	Query	thread_id=10	exec_time=0	error_code=0	Xid = 100
    use `mysql`/*!*/;
    SET TIMESTAMP=1743077311/*!*/;
    SET @@session.pseudo_thread_id=10/*!*/;
    SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
    SET @@session.sql_mode=1168113696/*!*/;
    SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
    /*!\C utf8mb4 *//*!*/;
    SET @@session.character_set_client=255,@@session.collation_connection=255,@@session.collation_server=255/*!*/;
    SET @@session.lc_time_names=0/*!*/;
    SET @@session.collation_database=DEFAULT/*!*/;
    /*!80011 SET @@session.default_collation_for_utf8mb4=255*//*!*/;
    /*!80013 SET @@session.sql_require_primary_key=0*//*!*/;
    create table test_t1(name varchar(256))
    /*!*/;
    SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
    DELIMITER ;
    # End of log file
    /*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
    

创建表 test1 的sql成功被解析。

## Binlog 的清理

对于一个繁忙的MySQL 实例，其 Binlog 增长也是比较快的，因此，需要设置其保留天数，如果磁盘即将满，那么可能还要单独删除历史Binlog。

### 根据Binlog文件名删除

可以使用`purge binary logs` 语句来删除指定 Binlog 的文件名或指定时间之前的 Binlog 文件

首先使用`show binary logs`查看

![image-20250328164722647](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328164722647.png)

然后使用 `purge binary logs to 'mysql-bin.000004'`删除 1~3 的Binlog文件

![image-20250328164842418](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328164842418.png)

直接在mysql服务的binlog目录下查看

![image-20250328164910321](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328164910321.png)

也是没有了

### 根据时间删除

直接在mysql的服务端查看Binlog文件的详细信息

![image-20250328165017505](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328165017505.png)

假设我们删除 20点之前的Binlog文件，执行`purge binary logs before '2025-03-27 20:00:00';`

![image-20250328165309545](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328165309545.png)

好像和预期有点不符，不过还是可以看到是可以删除的。

## Binlog 保留参数

一般不会直接删除Binlog文件，而是通过设置`expire_logs_days`参数或`binlog_expire_logs_seconds`参数。

`expire_logs_days`参数定义了日志保留天数，`binlog_expire_logs_seconds`参数定义了日志保留秒数。

MySQL 8.0 建议设置`binlog_expire_logs_seconds`参数，在未来版本中可能会废除`expire_logs_days`参数。

![image-20250328165803855](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328165803855.png)

![image-20250328165851969](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328165851969.png)

`expire_logs_days`为0表示永远不删除，`expire_logs_days`默认值为0

`binlog_expire_logs_seconds`默认为 2592000秒，等于30天，也就是默认是保存30天

假设将`binlog_expire_logs_seconds`设置为1小时，也就是3600s，那么之前保留的Binlog文件会被删除

![image-20250328170130371](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328170130371.png)

![image-20250328170416098](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328170416098.png)

不管设置为多少，不会自动删除

需要使用`flush logs`触发生效

![image-20250328170521962](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328170521962.png)

## Binlog 的落盘

Binlog 同步到磁盘的频率由 `sync_binlog` 参数控制。`sync_binlog`参数大致有这几种配置

  * `sync_binlog=0`:禁用MySQL服务将Binlog 同步到磁盘的功能，是由操作系统控制Binlog的刷盘。在这种情况下，性能比较好，但是当操作系统崩溃时可能会丢失部分事务。
  * `sync_binlog=1`：每个事务都会同步到磁盘。这是最安全的设置，但是磁盘写入次数的增加可能会导致性能下降
  * `sync_binlog=N`：表示每N个事物Binlog同步一次到磁盘。当操作系统崩溃时，服务器提交的事务可能没有被刷新到Binlog中，此时可能会丢失部分事务，虽然设置比较大的值可以提高性能，但是数据丢失的风险也会增加。

![image-20250328171003002](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328171003002.png)

可以通过 `set global sync_binlog=3;`调整

![image-20250328171107662](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328171107662.png)

## Binlog 相关参数

  1. `max_binlog_size`：单个Binlog文件大小的最大值

![image-20250328171815372](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250328171815372.png)

  2. `log-slave-update`：从库从主库接收的更新是否记录到从库自身的Binlog中，如果从库后面又接了从库，或者在从库上做备份，或者MySQL 5.6 主从复制使用了GTID模式，那么建议开启这个参数

  3. `binlog-do-db`:后面接库名，表示当前数据库只记录该参数设置的库的 Binlog ，其他库都不记录

  4. `binlog-ignore-db`：后面接库名，表示当前数据库不记录该参数设置的库的Binlog，其他库都记录
