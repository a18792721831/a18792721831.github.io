---
layout: post
title: "Redis--数据结构--Sorted Set"
date: 2020-07-18 14:11:18 +0800
categories: [Redis的SortedSet, Redis的排序Set的操作, Redis有序集合, Redis实现排行榜, Redis有序集合的应用]
description: "本文深入讲解Redis中的SortedSet数据结构，包括其基本概念、常用命令及其应用实例，如排行榜和数据超时清理等场景，展示了SortedSet在处理有序集合方面的强大功能。"
keywords: Redis的SortedSet, Redis的排序Set的操作, Redis有序集合, Redis实现排行榜, Redis有序集合的应用
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107428187
> - 发布时间：2020-07-18 14:11:18
> - 阅读量：646
> - 分类：Redis专栏收录该内容, 订阅专栏
> - 标签：#Redis的SortedSet, #Redis的排序Set的操作, #Redis有序集合, #Redis实现排行榜, #Redis有序集合的应用

## 摘要

文章浏览阅读646次。本文深入讲解Redis中的SortedSet数据结构，包括其基本概念、常用命令及其应用实例，如排行榜和数据超时清理等场景，展示了SortedSet在处理有序集合方面的强大功能。

---

#### Redis--数据结构--Sorted Set

  * [1\. 介绍](<#1__1>)
  * [2\. 命令](<#2__7>)
  *     * [2.1 增加](<#21__9>)
    * [2.2 统计个数](<#22__19>)
    * [2.3 统计指定分数范围的元素个数](<#23__29>)
    * [2.4 获取分数](<#24__39>)
    * [2.5 增加分数](<#25__49>)
    * [2.6 获取指定范围的元素(score从小到大)](<#26_score_59>)
    * [2.7 获取指定范围的元素(score从大到小)](<#27_score_69>)
    * [2.8 获取指定分数范围的元素(score从小到大)](<#28_score_79>)
    * [2.9 获取指定分数范围的元素(score从大到小)](<#29_score_89>)
    * [2.10 获取元素的索引(score从小到大)](<#210_score_99>)
    * [2.11 获取元素的索引(score从大到小)](<#211_score_109>)
    * [2.12 删除](<#212__119>)
    * [2.13 根据索引范围删除](<#213__129>)
    * [2.14 根据分数范围删除](<#214__139>)
  * [3\. 示例](<#3__149>)
  *     * [3.1 排行榜](<#31__151>)
    * [3.2 排行榜--分页](<#32__165>)
    * [3.3 排行榜查询](<#33__181>)
    * [3.4 定期清理超时数据](<#34__187>)

## 1\. 介绍

Sorted Set是字符串的集合，不允许重复的成员出现在一个Set中。Sorted Set是有序集合，在Sorted Sett内部的每一个元素，都有一个score与value关联。Sort Set的有序性就是通过score保证的。Sorted Set属于Set，具有Set的全部特性，同时Sorted Set拥有Set的额外的一个特性Sort。尽管Set的value不允许重复，但是score却是允许重复的。

在Set中增加、删除、更新元素是一个资源消耗非常小的操作。因为Sorted Set是有序的，所以即使访问Set的中部的元素，依然具有很高的效率。这是Redis的优势，其他数据库中想要实现这一点，比较困难。

## 2\. 命令

### 2.1 增加

**命令** ：**ZADD** key score element [score element …]

命令说明：增加Sorted Set元素，如果key不存在，那么会创建key，如果element不存在，那么会将element加入key的Sorted Set.如果element已经在key对应的Sorted Set中了，那么就更新element的score，更新了score，就会进行重新排序。如果key对应的不是Sorted Set，那么执行异常。

返回值：实际增加的element数量。0：element本来就存在，本次只是进行了更新score。

![image-20200717150604983](https://i-blog.csdnimg.cn/blog_migrate/545f211242ef9a817fac2477c7eecb39.png)

### 2.2 统计个数

**命令** ：**ZCARD** key

命令说明：获取key对应的Sorted Set的element数量。

返回值：返回key对应的Sorted Set的元素数量。0：key不存在或者Sorted Set为空。

![image-20200717152059938](https://i-blog.csdnimg.cn/blog_migrate/5fe9ffdfeac6b37882a976bfb62c565c.png)

### 2.3 统计指定分数范围的元素个数

**命令** ：**ZCOUNTT** key [(]min [(]max

命令说明：统计key对应的Sorted Set中元素的score在min和max范围内的元素数量。==min表示闭区间，(min表示开区间；max表示闭区间，(max表示开区间。开闭区间可以自由搭配。==如果key对应的不是Sorted Set，执行异常。

返回值：指定key对应的Sorted Set的元素的score在指定区间内的元素数量。0：key不存在或者score区间内的元素为空。

![image-20200717152756007](https://i-blog.csdnimg.cn/blog_migrate/634c1fc639175df4788d05d43d06d82a.png)

### 2.4 获取分数

**命令** ：**ZSCORE** key element

命令说明：获取指定key的Sorted Set的指定element的score。如果key对应的不是Sorted Set，执行异常。

返回值：字符串形式的分数。空：key不存在，或者element不存在。

![image-20200717154152583](https://i-blog.csdnimg.cn/blog_migrate/790ec0ba78665f00425e1961ced349fe.png)

### 2.5 增加分数

**命令** ：**ZINCRBY** key increment element

命令说明：指定的key的Sorted Set的指定的element的score更新为score+increment。如果key不存在，则创建key对应的Sorted Set。如果指定key对应的Sorted Set中不存在element则，新增element，并且设置原score为0，然后将0+increment设置为新的score。如果key对应的不是Sorted Set，执行异常。

返回值：更新后的分数，以字符串形式。

![image-20200718114418653](https://i-blog.csdnimg.cn/blog_migrate/0548cf819b955c4138a5b1124098bd08.png)

### 2.6 获取指定范围的元素(score从小到大)

**命令** ：**ZRANGE** key start end [WITHSCORES]

命令说明：获取指定key对应的Sorted Set在start到end(index)范围内的元素(score从小到大)。==如果有[WITHSCORES]那么返回元素的时候会将score一起返回。==start：0，第一个元素；-1，最后一个元素。如果start > end返回空列表；如果end > length返回start之后的全部元素。如果key对应的不是Sorted Set，执行异常。

返回值：元素列表。一个元素，一个分数，交替返回。

![image-20200718115021256](https://i-blog.csdnimg.cn/blog_migrate/20a5b18146c94aac9e878250bf23f73c.png)

### 2.7 获取指定范围的元素(score从大到小)

**命令** ：**ZREVRANGE** key start end [WITHSCORES]

命令说明：获取指定key对应的Sorted Set在start到end(index)范围内的元素(score从大到小)。==如果有[WITHSCORES]那么返回元素的时候会将score一起返回。==start：0，第一个元素；-1，最后一个元素。如果start > end返回空列表；如果end > length返回start之后的全部元素。如果key对应的不是Sorted Set，执行异常。

返回值：元素列表。一个元素，一个分数，交替返回。

![image-20200718115402428](https://i-blog.csdnimg.cn/blog_migrate/1b0792130e0a66e08d0f6c2708376f5e.png)

### 2.8 获取指定分数范围的元素(score从小到大)

**命令** ：**ZRANGEBYSCORE** key min max [WITHSCORES] [LIMIT offset count]

命令说明：获取指定key的Sorted Set中score在min和max的闭区间内的元素(score从小到大)。WITHSCORES一起返回元素和分数，交替返回。LIMIT offset count表示从满足条件列表的第offset开始返回，返回count个。min和max可以使用==(==来使用开区间。如果分数相同，则按照字典顺序返回。

返回值：元素列表。一个元素，一个分数，交替返回。

![image-20200718120105294](https://i-blog.csdnimg.cn/blog_migrate/341bffc22cac77412e0c7b2787b841c1.png)

### 2.9 获取指定分数范围的元素(score从大到小)

**命令** ：**ZREVRANGEBYSCORE** key max min [WITHSCORES] [LIMIT offset count]

命令说明：获取指定key的Sorted Set中score在min和max的闭区间内的元素(score从大到小)。WITHSCORES一起返回元素和分数，交替返回。LIMIT offset count表示从满足条件列表的第offset开始返回，返回count个。min和max可以使用==(==来使用开区间。如果分数相同，则按照字典顺序返回。

返回值：元素列表。一个元素，一个分数，交替返回。

![image-20200718130203688](https://i-blog.csdnimg.cn/blog_migrate/24ce504feb4ce2d9b14cf80a2ca6e863.png)

### 2.10 获取元素的索引(score从小到大)

**命令** ：**ZRANK** key element

命令说明：在Sorted Set中的元素都是按照score从小到大的顺序排序，如果score相同则按照字典顺序排序。获取到指定key对应的Sorted Set的element的index值。

返回值：0：第一个元素的索引；非0：元素的索引值；空：元素或者key不存在。

![image-20200718130724359](https://i-blog.csdnimg.cn/blog_migrate/91b32caabd09069d9b07bba2749cde06.png)

### 2.11 获取元素的索引(score从大到小)

**命令** ：**ZREVRANK** key element

命令说明：在Sorted Set中的元素都是按照score从大到小的顺序排序，如果score相同则按照字典顺序排序。获取到指定key对应的Sorted Set的element的index值。

返回值：0：第一个元素的索引；非0：元素的索引值；空：元素或者key不存在。

![image-20200718131026439](https://i-blog.csdnimg.cn/blog_migrate/1c056625b5c4c0fc663f601dd90ea607.png)

### 2.12 删除

**命令** ：**ZREM** key element [element …]

命令说明：删除指定key对应的Sorted Set中指定的lelement的元素。如果元素或者key不存在，则什么都不做。如果key对应的不是Sorted Set，执行异常。

返回值：实际被删除的element的数量。

![image](https://i-blog.csdnimg.cn/blog_migrate/fd12f70381b8f1ff066af2745de086ae.png)

### 2.13 根据索引范围删除

**命令** ：**ZREMRANGEBYRANK** key start end

命令说明：删除指定key对应的Sorted Set中索引在start和end区间内的元素。start为0表示第一个索引，为-1表示最后一个索引。如果key对应的不是Sorted Set，执行异常。

返回值：实际被删除的element的数量。

![image-20200718131905805](https://i-blog.csdnimg.cn/blog_migrate/2e5498e8eb7489ae410463faece47092.png)

### 2.14 根据分数范围删除

**命令** ：ZREMRANGEBYSCORE** key min max

命令说明：删除指定key对应的Sorted Set中分数在min和max闭区间内的元素。可以使用==(==使用开区间。如果key对应的不是Sorted Set，执行异常。

返回值：实际被删除的element的数量。

![image-20200718132550050](https://i-blog.csdnimg.cn/blog_migrate/d8461e1911ae2582cde3299de41acca9.png)

## 3\. 示例

### 3.1 排行榜

我们经常可以看到在一些比赛中，会展示排行榜的前x名。

首先创建一个Sorted Set，然后我们将name和score存入。最后获取的时候按照index或者score范围查询即可。

获取前5名

![image-20200718133843096](https://i-blog.csdnimg.cn/blog_migrate/a143ef35750f5f3138c8bad71291b767.png)

获取分数排名前5

![image-20200718134223579](https://i-blog.csdnimg.cn/blog_migrate/176f04f5980ff29fade15278c978fa99.png)

### 3.2 排行榜–分页

假设我们的排行榜上面不仅仅有分数，还有参赛者的介绍等等。

所以：

冠军：一页展示

亚军，季军：一页展示

第4~10名：一页展示

第20~50名：一页展示

![image-20200718134913095](https://i-blog.csdnimg.cn/blog_migrate/68ee6e2721e0a3c51ae6b1b3a2afcdf1.png)

### 3.3 排行榜查询

如果排行榜比较多的时候，玩家需要查询自己在排行榜中的位置。

![image-20200718135903986](https://i-blog.csdnimg.cn/blog_migrate/9689e6c5924e90e332b3405cd6d4d6f4.png)

### 3.4 定期清理超时数据

将数据的超时时间设置为分数，定期清理到时间的数据。

![image-20200718140417390](https://i-blog.csdnimg.cn/blog_migrate/5fb4350858bf4ccc4366470e08cb9f3c.png)