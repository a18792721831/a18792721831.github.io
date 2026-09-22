---
layout: post
title: "MySQL Redo Log"
date: 2025-08-02 17:58:44 +0800
categories: [mysql, 数据库, Redo Log, 重做日志, 日志]
description: "MySQL的Redo Log是InnoDB存储引擎实现事务持久化(Durability)的关键机制，采用WAL(预写式日志)技术，先记录日志再写磁盘。Redo Log通过记录页的修改而非整个页的落盘，将随机IO转为顺序IO，显著提升写入性能（性能测试显示参数设置为0或2比1快1倍）。它由内存缓冲区和磁盘文件组成，通过innodb_flush_log_at_trx_commit参数控制落盘频率（建议设为1以保证崩溃恢复）。MySQL 8.0.30后改用目录式存储（#innodb_redo），支持动态调整red"
keywords: mysql, 数据库, Redo Log, 重做日志, 日志
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/149862528
> - 发布时间：2025-08-02 17:58:44
> - 阅读量：1
> - 分类：mysql专栏收录该内容, 订阅专栏
> - 标签：#mysql, #数据库, #Redo Log, #重做日志, #日志

## 摘要

文章浏览阅读1.3k次，点赞22次，收藏29次。MySQL的Redo Log是InnoDB存储引擎实现事务持久化(Durability)的关键机制，采用WAL(预写式日志)技术，先记录日志再写磁盘。Redo Log通过记录页的修改而非整个页的落盘，将随机IO转为顺序IO，显著提升写入性能（性能测试显示参数设置为0或2比1快1倍）。它由内存缓冲区和磁盘文件组成，通过innodb_flush_log_at_trx_commit参数控制落盘频率（建议设为1以保证崩溃恢复）。MySQL 8.0.30后改用目录式存储（#innodb_redo），支持动态调整red

---

MySQL Redo Log

MySQL主从复制：https://blog.csdn.net/a18792721831/article/details/146117935  
MySQL Binlog：https://blog.csdn.net/a18792721831/article/details/146606305  
MySQL General Log：https://blog.csdn.net/a18792721831/article/details/146607343  
MySQL Slow Log：https://blog.csdn.net/a18792721831/article/details/147166971  
MySQL Error Log：https://blog.csdn.net/a18792721831/article/details/147167038  
MySQL Redo Log: https://blog.csdn.net/a18792721831/article/details/149862528  
MySQL Undo Log: https://blog.csdn.net/a18792721831/article/details/149880355

#### MySQL Redo Log

  * Redo Log 介绍
  * Redo Log 的落盘
  * Redo Log 的数量及大小修改
  * CheckPoint
  * LSN
  * Mysql 8.0 的 Redo Log 归档
  * Mysql 8.0 中的 Redo Log 禁用
  * 官网文档

  
InnoDB使用 Redo Log 来保证数据的一致性和可持久化。 

## Redo Log 介绍

MySQL采用的是WAL(Write-Ahead Logging)技术，也就是先写日志在写磁盘。如果有修改操作，则先将操作记录在 Redo Log 中，然后将 Redo Log Buffer 中的数据刷新到磁盘的日志文件中，最后写入数据文件中。

Redo Log 记录了这个页做了什么改动，对Redo Log 进行落盘，不仅可以保证数据持久化，还可以提高写入效率。

  * MySQL如果每次都按页落盘，即使有少量数据的修改，每次落盘也是整个数据页，那么IO成本将会非常高。而Redo Log 记录的是缓冲池中页的修改记录，因此每次只需要对 Redo Log 进行落盘，而不需要对整个页落盘。大大降低了落盘的数据量，节约成本，提高写入效率。
  * 如果每次都是将页落盘，那么页会在不同的位置，因此落盘时基本上就是随机IO，而有了Redo Log，只将Redo Log 落盘，就可以将这些随机 IO 转换成顺序IO ,这样也可以提高写入效率。

Redo Log 另一个很重要的作用就是崩溃恢复能力。事务在提交时会把Redo Log 刷新到磁盘中，如果此时系统宕机了，那么重启之后，只需要根据 Redo Log 的记录把数据页未更新的部分更新一下，就可以恢复事务所做的数据变更。

## Redo Log 的落盘

Redo Log 由两部分组成: Redo Log 缓冲区和 Redo Log 文件。

Redo Log 缓冲区的大小通过 `innodb_log_buffer_size` 参数来设置。通过 `show global variables like 'innodb_log_buffer_size';`查看

![image-20250412170026934](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412170026934.png)

16777216B=16384K=16M

当事务更新时，一般先写入Redo Log 缓冲区，在写入 Redo Log 文件。而具体的写入频率由`innodb_flush_log_at_trx_commit`参数控制。查看 `show global variables like 'innodb_flush_log_at_trx_commit';`

![image-20250412170324262](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412170324262.png)

`innodb_flush_log_at_trx_commit`参数有3个有效值：

  * 0，每秒将日志缓冲区写入日志文件一次，并在日志文件上执行磁盘刷新操作，未刷新日志的事务可能会在崩溃中丢失，此时 InnoDB不在符合事务持久性的要求。
  * 1，在每次提交事务时，日志缓冲区都会写入日志文件中，并在日志文件上执行磁盘刷新操作。
  * 2，在每次提交事务后写入日志，并且日志每秒刷新一次到磁盘。未刷新日志的事务可能会在崩溃中丢失。当MySQL服务发生宕机，但操作系统没有发生宕机时，不会出现数据丢失。但是当操作系统宕机时，重启后可能会丢失Redo Log 缓冲中还没有刷新到 Redo Log 文件中的数据。

选择不同的落盘频率，对于MySQL的性能存在一定的影响

创建一张表，用于验证性能
    
    
    use test;
    drop table if exists test_redo;
    create table `test_redo`(
    `id` int not null auto_increment,
    `a` varchar(20) default null,
    `b` int default null,
    `c` datetime not null default current_timestamp,
    primary key (`id`)
    ) engine=innodb charset=utf8mb4;
    drop procedure if exists test_insert;
    delimiter ;;
    create procedure test_insert()
    begin
    declare i int;
    set i = 1;
    while(i <= 100000) do
    insert into test_redo(a,b) values(i,i);
    set i=i+1;
    end while;
    end;;
    delimiter ;
    

执行

![image-20250412172229332](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412172229332.png)

执行**`set global innodb_flush_log_at_trx_commit=0`**后，调用 `test_insert`插入10万的数据

![image-20250412172936711](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412172936711.png)

花费了3分27秒5，也就是207.5秒。

执行**`set global innodb_flush_log_at_trx_commit=1`**,调用`test_insert`插入10万数据

![image-20250412173943934](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412173943934.png)

花费了6分55秒4，也就是415.4秒，是等于0的2倍

执行**`set global innodb_flush_log_at_trx_commit=2`**，调用`test_insert`插入10万数据

![image-20250412174656848](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250412174656848.png)

和等于0差不多。

`innodb_flush_log_at_trx_commit`参数设置为0或2的写入速度比设置为1的写入速度快很多。但是如果某个MySQL实例将 `innodb_flush_log_at_trx_commit`参数设置为0或2，那么它其实已经不具备ACID中的D(Durability,持久性)。在一般情况下，建议将`innodb_flush_log_at_trx_commit`参数设置为1，这样可以保证MySQL在崩溃恢复时数据不丢失。

## Redo Log 的数量及大小修改

MySQL启动后，Redo Log 的大小和个数就是固定的，比如可以配置3个大小为2GB的Redo Log，文件名类似于 `ib_logfile0`。Redo Log 是循环写的。

从 MySQL 8.0.30 开始，支持“多文件”redo log，文件名格式为：`#ib_redo<N>`

> https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html#innodb-redo-log-multiple-files

从MySQL 从 **8.0.30 版本** （2022年7月发布）开始，对 InnoDB 的 redo log 存储方式进行了重大调整，将传统的 `ib_logfile0`、`ib_logfile1` 文件改为存储在 `#innodb_redo` 目录中

> https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html
> 
> ![Clipboard_Screenshot_1745236922](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1745236922.png)

![image-20250421200242503](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250421200242503.png)

> 带有 `_tmp`是备用redo_log。因为在 mysql 8.0 中增加了动态调整 redo_log大小的能力，在调整过程中，会使用备用redo_log，直到调整完成。
> 
> 使用文件 rename 防止在动态调整过程中因为宕机导致的文件丢失。

相关参数

| 参数                          | 默认值                 | 说明                         |
|-----------------------------|---------------------|----------------------------|
| `innodb_redo_log_directory` | `./#innodb_redo`    | 指定 redo log 目录路径（可独立于数据目录） |
| `innodb_redo_log_capacity`  | `104857600` (100MB) | 总 redo log 容量（动态调整，单位：字节）  |
| `innodb_redo_log_files`     | 32                  | 最大 redo log 文件数（自动管理实际使用数） |

![image-20250421200404372](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250421200404372.png)

![image-20250528173259715](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250528173259715.png)

从第一个文件开始写，当redo_log0写满了，在写redo_log1,直到最后一个redo_log也写满了，继续从第一个文件开始写。其中 `write pos`是当前记录的位置，一边写一边移动；CheckPoint是当前要擦除的位置，也是向后推移的。擦除记录前会确保记录已经更新到数据文件中。

RodoLog 几个重要的参数：

  * Innodb_log_group_home_dir:控制 Redo Log 的存放路径，如果没有配置，默认在数据目录下

![image-20250802144856927](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802144856927.png)

![image-20250802144948081](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802144948081.png)

如果要修改，需要修改 mysql.cnf文件

先在mysql目录下创建redolog目录,记得修改权限

![image-20250802160718676](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802160718676.png)

接着在redolog目录下创建`'#innodb_redo'`目录，记得修改目录权限为 750 ，文件权限为 640 ，并且修改权限组为 ‘999:999’

> docker 中mysql 以999 的uid执行

接着配置Redo Log 的目录为新创建的目录
        
        [mysqld]
        server_id=100
        log_bin=/var/lib/mysql-bin/mysql-bin
        binlog_format=row
        early-plugin-load=keyring_file.so
        keyring_file_data=/var/lib/mysql/keyring
        binlog_encryption=on
        log-error=/var/log/mysql/mysql-error.log
        innodb_log_group_home_dir=/var/lib/mysql-redolog
        

然后重启
        
        docker run \
        --name master \
        -e MYSQL_ROOT_PASSWORD=master \
        -v /data/mysql/master/log:/var/log/mysql \
        -v /data/mysql/master/data:/var/lib/mysql \
        -v /data/mysql/master/conf:/etc/mysql/conf.d \
        -v /data/mysql/master/binlog:/var/lib/mysql-bin \
        -v /data/mysql/master/redolog1:/var/lib/mysql-redolog \
        -p 3106:3306 \
        -d \
        mysql:8.0 
        

启动后mysql会在新的redolog目录下创建redofile

![image-20250802165310731](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802165310731.png)

  * innodb_log_file_size:控制Redo Log的大小，默认值为 48Mb,但是不能超过 512GB除以 innodb_log_files_in_group 参数配置的值，如果 innodb_log_files_in_group 参数的值为2，那么 innodb_log_file_size 参数的最大值为256G.

如果innodb_redo_file_size 参数设置的太小，那么有可能导致Mysql 的redolog 频繁切换，频繁的触发数据的 CheckPoint，刷新脏页的次数随之增加，从而影响IO性能。

如果有一个大的事务把所有的Redo Log 写满了还没有写完，就会导致日志不能切换，Mysql 有可能就不能正常提供服务了。

如果参数设置的太大，虽然可以提升IO性能，但是当Mysql宕机时，恢复的时间也很长。

  * Innodb_log_files_in_group:控制Redo Log的文件数量

## CheckPoint

CheckPoint 是Redo Log 中当前要擦除的位置，Redo Log 在擦除记录前会确保记录已经更新到数据文件中。因此，也可以认为CheckPoint就是控制数据页刷新到磁盘的操作。

如果发生宕机重启，那么将会从CheckPoint之后开始恢复，之前已经刷新的数据页不需要恢复。

## LSN

Innodb的LSN(Log Sequence Number, 日志序列号)。

LSN用来精确记录Redo Log的位置信息，大小为 8 字节(mysql 5.6.3 之前是4字节)。

LSN的值是根据Redo Log 写入量计算的，写入多少字节的Log, LSN就增长多少。

可以通过 `show engine InnoDB status\G`查看
    
    
    =====================================
    2025-08-02 09:10:54 140514356577856 INNODB MONITOR OUTPUT
    =====================================
    Per second averages calculated from the last 2 seconds
    -----------------
    BACKGROUND THREAD
    -----------------
    srv_master_thread loops: 1 srv_active, 0 srv_shutdown, 676 srv_idle
    srv_master_thread log flush and writes: 0
    ----------
    SEMAPHORES
    ----------
    OS WAIT ARRAY INFO: reservation count 10
    OS WAIT ARRAY INFO: signal count 12
    RW-shared spins 0, rounds 0, OS waits 0
    RW-excl spins 0, rounds 0, OS waits 0
    RW-sx spins 0, rounds 0, OS waits 0
    Spin rounds per wait: 0.00 RW-shared, 0.00 RW-excl, 0.00 RW-sx
    ------------
    TRANSACTIONS
    ------------
    Trx id counter 5902
    Purge done for trx's n:o < 5899 undo n:o < 0 state: running but idle
    History list length 0
    LIST OF TRANSACTIONS FOR EACH SESSION:
    ---TRANSACTION 421989504019672, not started
    0 lock struct(s), heap size 1128, 0 row lock(s)
    ---TRANSACTION 421989504018864, not started
    0 lock struct(s), heap size 1128, 0 row lock(s)
    ---TRANSACTION 421989504018056, not started
    0 lock struct(s), heap size 1128, 0 row lock(s)
    --------
    FILE I/O
    --------
    I/O thread 0 state: waiting for completed aio requests (insert buffer thread)
    I/O thread 1 state: waiting for completed aio requests (read thread)
    I/O thread 2 state: waiting for completed aio requests (read thread)
    I/O thread 3 state: waiting for completed aio requests (read thread)
    I/O thread 4 state: waiting for completed aio requests (read thread)
    I/O thread 5 state: waiting for completed aio requests (write thread)
    I/O thread 6 state: waiting for completed aio requests (write thread)
    I/O thread 7 state: waiting for completed aio requests (write thread)
    I/O thread 8 state: waiting for completed aio requests (write thread)
    Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
     ibuf aio reads:
    Pending flushes (fsync) log: 0; buffer pool: 0
    874 OS file reads, 275 OS file writes, 108 OS fsyncs
    0.00 reads/s, 0 avg bytes/read, 0.00 writes/s, 0.00 fsyncs/s
    -------------------------------------
    INSERT BUFFER AND ADAPTIVE HASH INDEX
    -------------------------------------
    Ibuf: size 1, free list len 0, seg size 2, 0 merges
    merged operations:
     insert 0, delete mark 0, delete 0
    discarded operations:
     insert 0, delete mark 0, delete 0
    Hash table size 34679, node heap has 3 buffer(s)
    Hash table size 34679, node heap has 0 buffer(s)
    Hash table size 34679, node heap has 0 buffer(s)
    Hash table size 34679, node heap has 1 buffer(s)
    Hash table size 34679, node heap has 0 buffer(s)
    Hash table size 34679, node heap has 0 buffer(s)
    Hash table size 34679, node heap has 0 buffer(s)
    Hash table size 34679, node heap has 0 buffer(s)
    0.00 hash searches/s, 0.00 non-hash searches/s
    ---
    LOG
    ---
    Log sequence number          29869673
    Log buffer assigned up to    29869673
    Log buffer completed up to   29869673
    Log written up to            29869673
    Log flushed up to            29869673
    Added dirty pages up to      29869673
    Pages flushed up to          29869673
    Last checkpoint at           29869673
    Log minimum file id is       0
    Log maximum file id is       0
    32 log i/o's done, 0.00 log i/o's/second
    ----------------------
    BUFFER POOL AND MEMORY
    ----------------------
    Total large memory allocated 0
    Dictionary memory allocated 489041
    Buffer pool size   8192
    Free buffers       7192
    Database pages     996
    Old database pages 387
    Modified db pages  0
    Pending reads      0
    Pending writes: LRU 0, flush list 0, single page 0
    Pages made young 0, not young 0
    0.00 youngs/s, 0.00 non-youngs/s
    Pages read 853, created 143, written 201
    0.00 reads/s, 0.00 creates/s, 0.00 writes/s
    No buffer pool page gets since the last printout
    Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
    LRU len: 996, unzip_LRU len: 0
    I/O sum[0]:cur[0], unzip sum[0]:cur[0]
    --------------
    ROW OPERATIONS
    --------------
    0 queries inside InnoDB, 0 queries in queue
    0 read views open inside InnoDB
    Process ID=1, Main thread ID=140514029545024 , state=sleeping
    Number of rows inserted 0, updated 0, deleted 0, read 0
    0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
    Number of system rows inserted 8, updated 331, deleted 8, read 4791
    0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
    ----------------------------
    END OF INNODB MONITOR OUTPUT
    ============================
    
    

  * Log sequence number: 表示整个数据库最新的LSN值
  * Log flushed up to: 表示最新写入日志文件的事务产生的LSN值
  * Last checkpoint at:表示最新的数据页刷新到磁盘时的LSN值

## Mysql 8.0 的 Redo Log 归档

在数据备份过程中可能会复制 Redo Log。如果Mysql 的写入比较多，那么复制 Redo Log 的速度就会跟不上 Redo Log的生成速度。加上Redo Log是以覆盖方式记录的，就有可能会丢失部分Redo Log。

Mysql 8.0.17 引入了 Redo Log 归档，按照 Redo Log 记录顺序写入归档文件中，以解决备份时Redo Log丢失的问题。

Redo Log 的归档由`innodb_redo_log_archive_dirs`参数控制。

首先创建一个Redo Log 的归档文件夹：

![image-20250802171922474](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802171922474.png)

设置参数

![image-20250802172022591](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802172022591.png)

因为是docker 启动的MySQL，因此需要先关闭MySQL，在将 Redo Log 挂载进去
    
    
    docker run \
    --name master \
    -e MYSQL_ROOT_PASSWORD=master \
    -v /data/mysql/master/log:/var/log/mysql \
    -v /data/mysql/master/data:/var/lib/mysql \
    -v /data/mysql/master/conf:/etc/mysql/conf.d \
    -v /data/mysql/master/binlog:/var/lib/mysql-bin \
    -v /data/mysql/master/redolog:/var/lib/mysql-redolog \
    -v /data/mysql/master/redolog_archive:/var/lib/mysql-redolog-archive \
    -p 3106:3306 \
    -d \
    mysql:8.0 
    

![image-20250802172233975](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802172233975.png)

设置Redo Log 的归档目录
    
    
    set global innodb_redo_log_archive_dirs='redolog-archiving:/var/lib/mysql-redolog-archive';
    

> redolog-archiving 表示归档目录的标识符，冒号后面的才是归档目录

![image-20250802172432785](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802172432785.png)

注意此时仅仅是设置了Redo Log 的归档目录，mysql不会自动开始归档

![image-20250802172549315](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802172549315.png)

还需要启动归档
    
    
    do innodb_redo_log_archive_start("redolog-archiving", "redo-0001");
    

> 需要注意，如果mysql实例被禁用了Redo Log ，那么启动归档会失败
> 
> Error executing DO statement. You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ‘"’ at line 1 - Connection: master: 350ms

`redolog-archiving`是定义的归档目录的标识符，`redo-0001`是保存归档文件的子目录。

使用`show global status like 'innodb_redo_log_enabled'`查看是否禁用

![image-20250802172940570](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802172940570.png)

使用`alter instance enable innodb redo_log;`启用Redo Log

![image-20250802173022662](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802173022662.png)

需要注意的是如果需要使用子目录，比如 `redo-0001`，此时需要自己创建

> Error executing DO statement. Redo log archive directory ‘/var/lib/mysql-redolog-archive/redo-0001’ does not exist or is not a directory - Connection: master: 311ms

![image-20250802173219230](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802173219230.png)

创建了目录，并且正确的设置了权限后，就会成功启动

> DO statement executed successfully. - Connection: master: 210ms

接着查看归档目录

![image-20250802173307100](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802173307100.png)

此时会在Redo Log的归档目录下创建对应的Sql文件

执行一个sql，看看会不会刷新到归档目录的sql文件中
    
    
    create database test;
    use test;
    create table t(id int,name varchar(256));
    insert into t values(1, 'test');
    

![image-20250802173622901](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802173622901.png)

![image-20250802174138833](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802174138833.png)

停止归档
    
    
    do innodb_redo_log_archive_stop();
    

![image-20250802174350757](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20250802174350757.png)

## Mysql 8.0 中的 Redo Log 禁用

从 Mysql 8.0.21开始可以禁用 Redo Log。

可以在数据导入过程中加速。

> Mysql 即使禁用 Redo Log，在Mysql 启动时，依然会检查 Redo Log 目录，并且如果Redo Log目录中存在CheckPoint之后的数据，依然会检查完整性，并尝试恢复数据。
> 
> 只有当Mysql启动后，才能禁用 Redo Log。禁用后会永久生效。

禁用
    
    
    alter instance disable innodb redo_log;
    

查看
    
    
    SHOW GLOBAL STATUS LIKE 'Innodb_redo_log_enabled';
    

启用
    
    
    alter instance enable innodb redo_log;
    

> 从实际测试情况来看，禁用与启用 redo log 有 10%~30% 的执行时间差异
> 
> https://segmentfault.com/a/1190000023415055

## 官网文档

https://dev.mysql.com/doc/refman/8.4/en/innodb-redo-log.html
