---
layout: post
title: "Redis--数据结构--String"
date: 2020-07-14 20:23:49 +0800
categories: [Redis的String, Redis的字符串操作, Redis的字符串常用操作, Redis的字符串的批量操作, Redis的常用用法]
description: "本文深入探讨了Redis中String数据结构的功能与应用，包括基本命令、原子操作、批量处理及其实现秒杀、缓存等功能的示例。Redis的String类型能够存储二进制安全的数据，最大容量达到512MB。"
keywords: Redis的String, Redis的字符串操作, Redis的字符串常用操作, Redis的字符串的批量操作, Redis的常用用法
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107346717
> - 发布时间：2020-07-14 20:23:49
> - 阅读量：423
> - 分类：Redis专栏收录该内容, 订阅专栏
> - 标签：#Redis的String, #Redis的字符串操作, #Redis的字符串常用操作, #Redis的字符串的批量操作, #Redis的常用用法

## 摘要

文章浏览阅读423次。本文深入探讨了Redis中String数据结构的功能与应用，包括基本命令、原子操作、批量处理及其实现秒杀、缓存等功能的示例。Redis的String类型能够存储二进制安全的数据，最大容量达到512MB。

---

#### Redis--数据结构--String

  * [1\. 介绍](<#1__1>)
  * [2\. 命令](<#2__5>)
  *     * [2.1 赋值](<#21__7>)
    * [2.2 取值](<#22__17>)
    * [2.3 获取并更新](<#23__27>)
    * [2.4 递增](<#24__37>)
    * [2.5 递减](<#25__47>)
    * [2.6 增加](<#26__57>)
    * [2.7 减少](<#27__67>)
    * [2.8 超时赋值](<#28__77>)
    * [2.9 存在赋值](<#29__87>)
    * [2.10 追加](<#210__97>)
    * [2.11 获取长度](<#211__107>)
    * [2.12 批量赋值](<#212__117>)
    * [2.13 批量取值](<#213__127>)
    * [2.14 批量存在赋值](<#214__137>)
  * [3\. 示例](<#3__147>)
  *     * [3.1 秒杀](<#31__149>)
    * [3.2 缓存](<#32__167>)

## 1\. 介绍

字符串类型是Redis中最为基础的数据存储类型，它在Redis中是二进制安全的，这便意味着该类型可以接受任何格式的数据，如JPEG图像数据或Json对象描述信息等。在Redis中字符串类型的Value最多可以容纳的数据长度是512M。

## 2\. 命令

### 2.1 赋值

**命令** ：**SET** key value

命令说明：如果不存在指定Keyd的映射，那么增加这个映射，否则更新映射的值。

返回值：OK

![image-20200714190140584](https://i-blog.csdnimg.cn/blog_migrate/69ce30b123ae41dbcbbfc0ea53f54a03.png)

### 2.2 取值

**命令** ：**GET** key

命令说明：获取指定Key的值，如果值不是String类型，那么返回错误

返回值：如果key存在，返回key的值，否则返回null

![image-20200714190422278](https://i-blog.csdnimg.cn/blog_migrate/4ff6e2f6b5b5c93d18cc4d7f07a687ab.png)

### 2.3 获取并更新

**命令** ：**GETSET** key value

命令说明：首先获取key的值，然后用value覆盖原值。这是一个原子操作。

返回值：如果原来不存在key的映射的值，那么返回null,否则返回key原来的值。

![image-20200714190722375](https://i-blog.csdnimg.cn/blog_migrate/a69dc5922d7bd09cb26136a4f1c18d67.png)

### 2.4 递增

**命令** ：**INCR** key

命令说明：将指定key的值递增1后返回。key对应的值必须是数字，或者可以转为数字，否则执行失败。该操作具有原子性。

返回值：key对应的值递增后的值，如果key不存在，那么原值为0.

![image-20200714191120514](https://i-blog.csdnimg.cn/blog_migrate/5a74f102bc9c621f2fcf39eff68f8098.png)

### 2.5 递减

**命令** ：**DECR** key

命令说明：将指定key的值递减1后返回。key对应的值必须是数字，或者可以转为数字，否则执行失败。该操作具有原子性。

返回值：key对应的值递减后的值，如果key不存在，那么原值为0.

![image-20200714191611299](https://i-blog.csdnimg.cn/blog_migrate/a61fda05b2b776a52cd43abfd6698532.png)

### 2.6 增加

**命令** ：**INCRBY** key value

命令说明：将指定key的值增加指定的值value后返回。key对应的值必须是数字，或者可以转为数字，否则执行失败。该操作具有原子性。

返回值：key对应的值增加后的值，如果key不存在，那么原值为0.

![image-20200714191957001](https://i-blog.csdnimg.cn/blog_migrate/c6e3109d1ee7c6a3677111f710317bda.png)

### 2.7 减少

**命令** ：**DECRBY** key value

命令说明：将指定key的值减少指定的值value后返回。key对应的值必须是数字，或者可以转为数字，否则执行失败。该操作具有原子性。

返回值：key对应的值减少后的值，如果key不存在，那么原值为0.

![image](https://i-blog.csdnimg.cn/blog_migrate/5775a253a82b9180d4b8c8ef2c941cd4.png)

### 2.8 超时赋值

**命令** ：**SETEX** key seconds value

命令说明：设置key的值为value，同时设置该key存活时间为seconds.该操作具有原子性。

返回值：OK（如果key有原值，将会被覆盖）

![image-20200714193513124](https://i-blog.csdnimg.cn/blog_migrate/62b2045956b4d85331af500a9bdc4649.png)

### 2.9 存在赋值

**命令** ：**SETNX** key value

命令说明：如果key不存在，那么增加key的映射，以及key的值为value；如果key存在，那么什么都不做。

返回值：1：设置成功；0：设置失败。

![image-20200714193920938](https://i-blog.csdnimg.cn/blog_migrate/e682441acf9e10fabb1ae4a2b5c018f1.png)

### 2.10 追加

**命令** ：**APPEND** key value

命令说明：将value追加到key的值的后面，如果key不存在，那么初始值为空。

返回值：追加后value的长度。

![image-20200714194227149](https://i-blog.csdnimg.cn/blog_migrate/1b2946f59853054f4c1df309d2afe84c.png)

### 2.11 获取长度

**命令** ：**STRLEN** key value

命令说明：返回指定key的值的字符串的长度。如果key不存在，长度为0.

返回值：长度。

![image-20200714195346962](https://i-blog.csdnimg.cn/blog_migrate/9e749a409a24b6bcaedcca37cc0269df.png)

### 2.12 批量赋值

**命令** ：**MSET** key0 value0[key1 value1…]

命令说明：批量赋值key指定的value。该操作具有原子性。

返回值：OK

![image-20200714195753948](https://i-blog.csdnimg.cn/blog_migrate/a2595fb5c7cced36a293a703ca185da9.png)

### 2.13 批量取值

**命令** ：**MGET** key0 [key1 …]

命令说明：批量取值key。如果key不存在，或者不是字符串，返回空(Nil)

返回值：values 列表

![image-20200714200142731](https://i-blog.csdnimg.cn/blog_migrate/790f955d452fec9cef37c79e0a91da68.png)

### 2.14 批量存在赋值

**命令** ：**MSETNX** key1 value1 [key2 value2 …]

命令说明：批量设置值，不存在设置，如果任意一个key已经存在，那么该次操作失败，该次全部操作回滚。

返回值：1：成功；0：失败。

![image-20200714200518955](https://i-blog.csdnimg.cn/blog_migrate/1756407ef3f6c3407f8f0eb1b8566f81.png)

## 3\. 示例

### 3.1 秒杀

背景：618活动，在线1W人，秒杀10件商品。

  1. 商品初始化
  2. 秒杀
  3. 结束

设置商品数量：

![image-20200714200852118](https://i-blog.csdnimg.cn/blog_migrate/30c51f4ac80ba77bf06c1867d32ebe17.png)

然后每个线程都使用递减操作：

![image-20200714200926420](https://i-blog.csdnimg.cn/blog_migrate/803882c243c0eca9109cf1a7d08d5946.png)

因为递减是原子操作，所以只要最后线程得到的结果大于0，那么就是秒杀成功的线程

### 3.2 缓存

背景：将Redis作为数据库的缓存，将经常使用的数据放到redis中。

修改的时候使用获取并更新操作。

![image-20200714201402041](https://i-blog.csdnimg.cn/blog_migrate/daaa48d26a0f6e6a80ebe99ba683c50f.png)

当然可以设置缓存过期时间：

![image-20200714201500388](https://i-blog.csdnimg.cn/blog_migrate/671b3fcd075b8412204792a8ad4129b7.png)