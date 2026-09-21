---
layout: post
title: "Python学习----sqlite"
date: 2019-05-26 16:09:44 +0800
categories: [python, sqlite, 无需安装的数据库, 最简单的数据库, 本地存储数据]
description: "本文介绍了SQLite，它是轻型关系型数据库，设计目标为嵌入式，占用资源低，常见于移动设备等，无需安装，在硬盘以db文件存储数据。还提到Python标准库支持SQLite，导入模块即可使用。指出它适合安全性要求不高的辅助功能。"
keywords: python, sqlite, 无需安装的数据库, 最简单的数据库, 本地存储数据
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/90577159
> - 发布时间：2019-05-26 16:09:44
> - 阅读量：371
> - 分类：Python专栏收录该内容, 订阅专栏
> - 标签：#python, #sqlite, #无需安装的数据库, #最简单的数据库, #本地存储数据

## 摘要

文章浏览阅读371次。本文介绍了SQLite，它是轻型关系型数据库，设计目标为嵌入式，占用资源低，常见于移动设备等，无需安装，在硬盘以db文件存储数据。还提到Python标准库支持SQLite，导入模块即可使用。指出它适合安全性要求不高的辅助功能。

---

#### Python学习----sqlite

  * [1.SQLite](<#1SQLite_1>)
  * [2.python集成sqlite](<#2pythonsqlite_9>)
  * [3.总结](<#3_106>)

## 1.SQLite

SQLite，是一款轻型的数据库，是遵守ACID的关系型数据库管理系统，它包含在一个相对小的C库中。它是D.RichardHipp建立的公有领域项目。它的设计目标是嵌入式的，而且目前已经在很多嵌入式产品中使用了它，它占用资源非常的低，在嵌入式设备中，可能只需要几百K的内存就够了。  
不像常见的客户-服务器范例，SQLite引擎不是个程序与之通信的独立进程，而是连接到程序中成为它的一个主要部分。所以主要的通信协议是在编程语言内的直接API调用。这在消耗总量、延迟时间和整体简单性上有积极的作用。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/581b9d7c9de6a3569ebf48e83946c0d2.png)  
以上数据来源于百度百科。  
sqlite常见于移动设备，小型工具，切入式等。比如很多的安卓程序，某某监控系统基于单片机的那种，pc的工具，比如流量监控等等。  
sqlite是与客户端在一起的数据存储机制，无需安装，直接集成在软件内。  
数据库存储数据最后肯定存储于硬盘内，sqlite最终在硬盘上创建一个db文件存储数据。

## 2.python集成sqlite

sqlite已经在python的标准库中支持，所以使用直接导入模块即可。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4f6ec8b96172d8fab67dca2b6c709391.png)
    
    
    import sqlite3
    
    # 数据库存储实例
    # 也就是数据库文件名
    linkUrl = 'hello.db'
    
    # 获取连接
    conn = sqlite3.connect(linkUrl)
    # print(dir(conn))
    
    # 获取操作游标
    cur = conn.cursor()
    # print(dir(cur))
    
    # 创建表的SQL
    # sqlCreate = 'create table user(name text,id number,sex text)'
    
    # 执行
    # cur.execute(sqlCreate)
    
    # 创建插入SQL
    # sqlInsert = "insert into user values('test1', 001, '男')"
    
    # 执行SQL
    # cur.execute(sqlInsert)
    
    # 提交事务
    # conn.commit()
    
    # 创建查询SQL
    sqlSelect = 'select * from user'
    
    # 执行查询SQL
    # data = cur.execute(sqlSelect)
    
    # 获取所有结果
    # print(data.fetchall())
    
    # 创建批量插入SQL
    # batchInsertSql = 'insert into user values (?,?,?)'
    
    # 创建批量数据
    # batchInsertData = [('batch1', 2, '男'), ('batch2', 3, '女'), ('batch3', 4, '人妖')]
    
    # 执行批量的SQL
    # data = cur.executemany(batchInsertSql,batchInsertData)
    
    # 提交批量的事物
    # conn.commit()
    
    # 执行查询批量结果SQL
    # batch_data = cur.execute(sqlSelect)
    
    # 获取所有的结果
    # print(batch_data.fetchall())
    
    # 创建更新SQL
    # sqlUpdate = "update user set name = 'tst' where id = 001"
    
    # 执行更新SQL
    # data = cur.execute(sqlUpdate)
    
    # 提交更新
    # conn.commit()
    
    # 执行查询更新结果SQL
    # update_data = cur.execute(sqlSelect)
    
    # 获取所有的结果
    # print(update_data.fetchall())
    
    # 创建删除SQL
    # deleteSql = 'delete from user where id = 1'
    
    # 执行删除SQL
    # data = cur.execute(deleteSql)
    
    # 提交删除事物
    # conn.commit()
    
    # 执行查询删除结果SQL
    delete_data = cur.execute(sqlSelect)
    
    # 获取所有的结果
    print(delete_data.fetchall())
    
    # 关闭操作游标
    cur.close()
    
    # 关闭连接
    conn.close()
    
    

## 3.总结

sqlite是一个非常好用的本地数据存储解决方案，但是安全性方面肯定不如其余数据库那样安全。  
如果是一些安全性不高，作为辅助功能的，还是可以考虑使用sqlite的。