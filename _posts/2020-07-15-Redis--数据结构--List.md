---
layout: post
title: "Redis--数据结构--List"
date: 2020-07-15 20:48:51 +0800
categories: [Redis的List, Redis的链表操作, Redis用作消息队列, Redis的队列操作, Redis的队列常用操作]
description: "本文深入探讨Redis中List数据结构的特性与应用，包括高效的数据插入与删除操作、丰富的命令集，以及如何利用List实现消息队列等场景。通过具体示例，读者将了解List在不同模式下的工作原理。"
keywords: Redis的List, Redis的链表操作, Redis用作消息队列, Redis的队列操作, Redis的队列常用操作
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107370047
> - 发布时间：2020-07-15 20:48:51
> - 阅读量：425
> - 分类：Redis专栏收录该内容, 订阅专栏
> - 标签：#Redis的List, #Redis的链表操作, #Redis用作消息队列, #Redis的队列操作, #Redis的队列常用操作

## 摘要

文章浏览阅读425次。本文深入探讨Redis中List数据结构的特性与应用，包括高效的数据插入与删除操作、丰富的命令集，以及如何利用List实现消息队列等场景。通过具体示例，读者将了解List在不同模式下的工作原理。

---

#### Redis--数据结构--List

  * 1\. 介绍
  * 2\. 命令
  *     * 2.1 头部增加
    * 2.2 尾部增加
    * 2.3 头部存在增加
    * 2.4 尾部存在增加
    * 2.5 头部获取
    * 2.6 尾部获取
    * 2.7 统计元素个数
    * 2.8 获取指定范围的元素
    * 2.9 移除指定值
    * 2.10 设置指定索引的值
    * 2.11 获取指定索引的值
    * 2.12 保留指定范围的元素
    * 2.13 插入
    * 2.14 元素转移
  * 3\. 示例
  *     * 3.1 消息队列
    * 3.2 消息队列--确认模式
    * 3.3 消息队列--最大长度

## 1\. 介绍

在Redis中，List类型是按照插入顺序排序的字符串链表。和数据结构中的普通链表一样，我们可以在其头部(left)和尾部(right)添加新的元素。在插入时，如果该键并不存在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除，那么该键也将会被从数据库中删除。List中可以包含的最大元素数量是4294967295。

从元素插入和删除的效率视角来看，如果我们是在链表的两头插入或删除元素，这将会是非常高效的操作，即使链表中已经存储了百万条记录，该操作也可以在常量时间内完成。然而需要说明的是，如果元素插入或删除操作是作用于链表中间，那将会是非常低效的。

## 2\. 命令

### 2.1 头部增加

**命令** ：**LPUSH** key value [value1 …]

命令说明：指定Key的list的头部插入values.如果key不存在，那么会创建映射关系，然后在插入。如果key对应的value不是list，那么执行失败。

返回值：插入后，list的元素数量。

![image-20200715185948780](https://i-blog.csdnimg.cn/blog_migrate/2a0468869e62a919af56ef087a5d3a11.png)

### 2.2 尾部增加

**命令** ：**RPUSH** key value [value …]

命令说明：指定key的list的尾部插入values.如果key不存在，那么会创建映射关系，然后在插入。如果key对应的value不是list，那么执行失败。

返回值：插入后，list的元素数量。

![image-20200715190436688](https://i-blog.csdnimg.cn/blog_migrate/e2568224ee407d2beecbb1a370b30c30.png)

### 2.3 头部存在增加

**命令** ：**LPUSHX** key value

命令说明：当指定key存在时，会尝试在key对应的list的头部增加value，否则什么也不做。

返回值：插入后，listy的元素的数量。

![image-20200715190848652](https://i-blog.csdnimg.cn/blog_migrate/bf6f143017f49f4e598445db5bc61f80.png)

### 2.4 尾部存在增加

**命令** ：**RPUSHX** key value

命令说明：当指定key存在时，会尝试在key对应的list的尾部增加value，否则什么也不做。

返回值：插入后，list的元素的数量。

![image-20200715191119763](https://i-blog.csdnimg.cn/blog_migrate/ffc93600296df79d7a5ad75832008971.png)

### 2.5 头部获取

**命令** ：**LPOP** key

命令说明：获取指定key对应的list的头部元素，并将该元素从list中取出。如果key不存在，返回Nil，如果key对应的value不是list则执行异常。

返回值：key对应的list的头部元素。

![image-20200715192721698](https://i-blog.csdnimg.cn/blog_migrate/387ef8e6d15bb464411a0cd53ffe8055.png)

### 2.6 尾部获取

**命令** ：**RPOP** key

命令说明：获取指定key对应的list的尾部元素，并将该元素从list中取出。如果key不存在，返回Nil,如果key对应的value不是list则执行异常。

返回值：key对应的list的尾部元素。

![image-20200715192743699](https://i-blog.csdnimg.cn/blog_migrate/52dec9e7afb0f27d474c3f480b03d1cd.png)

### 2.7 统计元素个数

**命令** ：**LLEN** key

命令说明：统计指定key对应的list的数量。如果key不存在，那么返回0。如果key对应的value不是list，那么执行异常。

返回值：key对应的list的元素个数。

![image-20200715193628321](https://i-blog.csdnimg.cn/blog_migrate/bf8543c457d4038ea6b7b6b6d32afea2.png)

### 2.8 获取指定范围的元素

**命令** ：**LRANGE** key start end

命令说明：获取指定key对应的list的指定范围的元素。start为0表示list的头部，-1表示尾部，-2表示倒数第二个元素。end和start相同。如果start大于list的length，那么返回空列表；如果end大于listt的length，那么返回剩余列表。list元素不会移除

返回值：符合要求的元素组成的list.

![image-20200715194335132](https://i-blog.csdnimg.cn/blog_migrate/6129ac787dc1794afe5a98f0d22bdef1.png)

### 2.9 移除指定值

**命令** ：**LREM** key count value

命令说明：首先获取指定key的list，然后遍历list，如果list的元素值等于value，那么就移除，如果移除的元素个数等于count，那么就停止。count大于0，从头开始遍历，从前往后移除；如果count等于0，那么删除list内元素等于value的所有元素，移除所有；如果count小于0，那么从尾部开始遍历，从后往前移除。

返回值：返回移除的元素的数量。如果key不存在，返回0.

![image-20200715195106901](https://i-blog.csdnimg.cn/blog_migrate/98286c5044e5000be1671593f4d38d1f.png)

### 2.10 设置指定索引的值

**命令** ：**LSET** key index value

命令说明：设置指定key的list的index位置的元素为value.==index大于0表示从头开始，index小于0表示从尾开始，index等于0表示头。==如果index溢出，执行异常。

返回值：OK

![image-20200715195613850](https://i-blog.csdnimg.cn/blog_migrate/161be37289853796a168321ea3186e6d.png)![image-20200715195723722](https://i-blog.csdnimg.cn/blog_migrate/d89f433d64950ed84d4c524821bc997d.png)![image-20200715195839494](https://i-blog.csdnimg.cn/blog_migrate/fb40a4c898ad18c2c0a7684b826e007e.png)

### 2.11 获取指定索引的值

**命令** ：**LINDEX** key index

命令说明：获取指定key的list的指定index的元素。==index等于0表示头，index大于0表示从头开始，index小于0表示从尾开始。==如果key对应的value不是list，那么执行异常。

返回值：指定key的list的指定的元素，或者Nil(key不存在)

![image-20200715202038961](https://i-blog.csdnimg.cn/blog_migrate/9d9508476aee95c5bc8292544e1f9c4a.png)

### 2.12 保留指定范围的元素

**命令** ：**LTRIM** key start end

命令说明：移除指定key的list的start和end之外的元素。只保留指定范围的元素。==start和end都是数值，0表示头，大于0表示从头开始，小于0表示从尾开始。==如果start大于end，移除key的list的全部元素，返回空列表；如果start大于list的length，那么移除全部元素。如果end大于listt的length，那么表示移除start前面的元素。

返回值：OK

![image-20200715200743345](https://i-blog.csdnimg.cn/blog_migrate/8e909bbc5d4e06c7d0ddea5979e8c56f.png)![image-20200715200839556](https://i-blog.csdnimg.cn/blog_migrate/46483148c21db9e76717f99288b2c73c.png)

![image-20200715200951901](https://i-blog.csdnimg.cn/blog_migrate/e96fc72d8ad9bbc54d7a1bc4ad6fa32a.png)

### 2.13 插入

**命令** ：**LINSERT** key BEFORE|AFTER pivot value

命令说明：向指定key的list的指定元素pivot的前面或者后面插入value.如果key不存在，不做任何操作。如果key的value不是list,执行异常。

返回值：成功：插入后key对应的list的元素数量；没找到pivot：返回-1；key不存在：返回0.

![image-20200715202702709](https://i-blog.csdnimg.cn/blog_migrate/10ed58cec2307883f4e959032e9fc9ed.png)

### 2.14 元素转移

**命令** ：**RPOPLPUSH** source dest

命令说明：从source 的尾部取出一个元素，然后放到dest的头部。如果source不存在，则不做任何操作。如果source等于dest，则表示将元素从尾放到头。==先执行RPOP，在执行LPUSH，保证原子性。==如果source活着dest中存在不是list的映射，执行异常。

返回值：操作的元素。

![image-20200715203406466](https://i-blog.csdnimg.cn/blog_migrate/d69b6120604e31574e170774efbc621c.png)

## 3\. 示例

### 3.1 消息队列

创建list，生产者从头部增加，消费者从尾部取出。

[图片]

### 3.2 消息队列–确认模式

创建list，生产者从头部增加，消费者从尾部取出并放到临时队列中，消费者完全消费后，在将消息从临时队列中取出。(使用移除指定值，移除1次，从头开始)

![image-20200715204242446](https://i-blog.csdnimg.cn/blog_migrate/d0f0be0c3f1da8a172563d63dcda73a8.png)

### 3.3 消息队列–最大长度

创建list，使用保留指定范围的元素，可以保证list的长度恒定。

![image-20200715204632146](https://i-blog.csdnimg.cn/blog_migrate/1eb6d8ce62ee7aa088f87a2d25078528.png)
