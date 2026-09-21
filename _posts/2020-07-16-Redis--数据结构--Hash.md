---
layout: post
title: "Redis--数据结构--Hash"
date: 2020-07-16 19:03:24 +0800
categories: [Redis的hash, Redis的Hash操作, Redis的Hash实现限购, Redis的Hash的常用操作, Redis的哈希的用法]
description: "本文深入探讨了Redis中Hash数据结构的应用与操作，包括赋值、取值、键的检查与删除等基本命令，以及如何利用Hash进行优惠券抢购、激活码管理和用户名检查等实用场景。"
keywords: Redis的hash, Redis的Hash操作, Redis的Hash实现限购, Redis的Hash的常用操作, Redis的哈希的用法
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107390616
> - 发布时间：2020-07-16 19:03:24
> - 阅读量：419
> - 分类：Redis专栏收录该内容, 订阅专栏
> - 标签：#Redis的hash, #Redis的Hash操作, #Redis的Hash实现限购, #Redis的Hash的常用操作, #Redis的哈希的用法

## 摘要

文章浏览阅读419次。本文深入探讨了Redis中Hash数据结构的应用与操作，包括赋值、取值、键的检查与删除等基本命令，以及如何利用Hash进行优惠券抢购、激活码管理和用户名检查等实用场景。

---

#### Redis--数据结构--Hash

  * [1\. 介绍](<#1__1>)
  * [2\. 命令](<#2__5>)
  *     * [2.1 赋值](<#21__7>)
    * [2.2 取值](<#22__17>)
    * [2.3 键是否存在](<#23__27>)
    * [2.4 统计键的数量](<#24__37>)
    * [2.5 删除键](<#25__47>)
    * [2.6 不存在赋值](<#26__57>)
    * [2.7 值增加x](<#27_x_67>)
    * [2.8 获取所有键值对](<#28__77>)
    * [2.9 获取所有的键](<#29__87>)
    * [2.10 获取所有的值](<#210__97>)
    * [2.11 批量赋值](<#211__107>)
    * [2.12 批量获取](<#212__117>)
  * [3\. 示例](<#3__127>)
  *     * [3.1 抢优惠券](<#31__129>)
    * [3.2 激活码](<#32__137>)
    * [3.3 用户名是否被占用](<#33__143>)

## 1\. 介绍

可以把Redis中的Hash类型理解成java里面的Map<String,String>类型。key就是map的实例对象引用，field就是map的键，value就是map的键对应的值。==所以该类型非常适合于存储值对象的信息。==如果Hash中包含很少的字段，那么该类型的数据也将仅占用很少的磁盘空间。每一个Hash可以存储4294967295个键值对。

## 2\. 命令

### 2.1 赋值

**命令** ：**HSET** key field value

命令说明：对指定的key赋值field–value键值对，如果key里面已经存在field键值对，那么更新field对应的值，如果不存在则增加。

返回值：1表示新增键值对，0表示更新键值对。（不支持空值）

![image-20200716121151040](https://i-blog.csdnimg.cn/blog_migrate/5831fc7bec7e325c84c25bffe3f670de.png)

### 2.2 取值

**命令** ：**HGET** key field

命令说明：获取指定key中指定field的值。

返回值：指定key的指定field的值。如果key活着field不存在，返回空。

![image-20200716121512232](https://i-blog.csdnimg.cn/blog_migrate/f5a86cc6a8df3abc3a37c3529e4b9316.png)

### 2.3 键是否存在

**命令** ：**HEXISTS** key field

命令说明：指定key中是否存在指定field。

返回值：1：存在；0：field或者key不存在。

![image-20200716174111618](https://i-blog.csdnimg.cn/blog_migrate/b3c37b95cf3cc88563406872aa43131e.png)

### 2.4 统计键的数量

**命令** ：**HLEN** key

命令说明：统计key中field的数量。

返回值：key中field的数量，0：key不存在，或者key为空。

![image-20200716174418486](https://i-blog.csdnimg.cn/blog_migrate/7602526eaeeb06845f737ed3756ee046.png)

### 2.5 删除键

**命令** ：**HDEL** key field [field1 …]

命令说明：删除指定key的指定field。如果field或者key不存在，则不做任何操作。

返回值：实际删除的数量。0：没有删除任何field。

![image-20200716174742315](https://i-blog.csdnimg.cn/blog_migrate/e6c8e218012a0f932a8107680a333be0.png)

### 2.6 不存在赋值

**命令** ：**HSETNX** key field value

命令说明：如果指定key或者field不存在，那么创建并赋值。否则不做任何操作。

返回值：1：成功赋值；0：什么都没做。

![image](https://i-blog.csdnimg.cn/blog_migrate/f2b688e9a47863617b186348cb592a6f.png)

### 2.7 值增加x

**命令** ：**HIINCRBY** key field increment

命令说明：首先取得指定key的指定field的值value，然后将value+increment的结果赋值给field。如果key或者field不存在，那么会创建value为0的filed。==increment可以为负数。==如果value不可运算，执行异常。

返回值：value+increment

![image-20200716175659327](https://i-blog.csdnimg.cn/blog_migrate/3c98917c8ede82ec6fd230ba931114ec.png)

### 2.8 获取所有键值对

**命令** ：**HGETALL** key

命令说明：获取指定key的所有键值对。

返回值：一个field，一个value，交替返回。空hash返回空。

![image-20200716175857183](https://i-blog.csdnimg.cn/blog_migrate/317266ac79d1365a8732eb2c48f3781f.png)

### 2.9 获取所有的键

**命令** ：**HKEYS** key

命令说明：获取指定key的所有field。

返回值：field列表。当key不存在时，返回空。

![image-20200716183922958](https://i-blog.csdnimg.cn/blog_migrate/147806396f88d77b3a80b463e29e2897.png)

### 2.10 获取所有的值

**命令** ：**HVALS** key

命令说明：获取指定key的所有的value。

返回值：value列表。当key不存在时，返回空。

![image-20200716184157035](https://i-blog.csdnimg.cn/blog_migrate/2445caaaebb40d9f2ac50a610fcaee0b.png)

### 2.11 批量赋值

**命令** ：**HMSET** key field1 value1 [field2 value2 …]

命令说明：批量赋值。如果filed存在，则更新，否则创建。

返回值：OK

![image-20200716184426719](https://i-blog.csdnimg.cn/blog_migrate/cb211d8270bc77e54eeed5f3f8d44cc2.png)

### 2.12 批量获取

**命令** ：**HMGET** key field [field1 …]

命令说明：批量获取value。如果filed不存在，则返回列表中这个field的value是空。如果key不存在，返回空列表。

返回值：value列表。或者空列表，或者列表中含有空元素。

![image-20200716184855671](https://i-blog.csdnimg.cn/blog_migrate/f5da6c05d10c304afdfc9d63b6d5ef71.png)

## 3\. 示例

### 3.1 抢优惠券

创建商品Key,创建30,40,50元的优惠券，每种300,400,500张。

每次抢到，就需要用值增加的操作减少。

![image-20200716185331646](https://i-blog.csdnimg.cn/blog_migrate/62e0405d0bb58193ed092d5eaf696568.png)

### 3.2 激活码

创建激活码key,创建激活码field，value为1表示激活码未使用，为0表示已用。

![image-20200716185823255](https://i-blog.csdnimg.cn/blog_migrate/dbc6498869cd8375f25446dda4065823.png)

### 3.3 用户名是否被占用

首先创建用户名的key，用户输出用户名后使用键是否存在判断，增加的时候使用不存在赋值插入。

![image-20200716190050899](https://i-blog.csdnimg.cn/blog_migrate/8c38a892bac8234f843b3d2c768a0058.png)