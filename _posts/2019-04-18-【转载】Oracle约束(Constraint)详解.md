---
layout: post
title: "【转载】Oracle约束(Constraint)详解"
date: 2019-04-18 08:31:36 +0800
categories: [oracle, 约束, 各种约束]
description: "本文深入解析Oracle数据库中的五种约束：主键、唯一性、非空、外键和检查约束，探讨其作用、特点及应用场景，同时介绍如何调整约束状态以应对特定业务需求。"
keywords: oracle, 约束, 各种约束
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/89370536
> - 发布时间：2019-04-18 08:31:36
> - 阅读量：461
> - 分类：oracle专栏收录该内容, 订阅专栏
> - 标签：#oracle, #约束, #各种约束

## 摘要

文章浏览阅读461次。本文深入解析Oracle数据库中的五种约束：主键、唯一性、非空、外键和检查约束，探讨其作用、特点及应用场景，同时介绍如何调整约束状态以应对特定业务需求。

---

转自：<https://www.cnblogs.com/chengxiao/p/6032183.html>

##  [Oracle约束(Constraint)详解](<https://www.cnblogs.com/chengxiao/p/6032183.html>)

## 概述

约束是数据库用来确保数据满足业务规则的手段，不过在真正的企业开发中，除了主键约束这类具有强需求的约束，像外键约束，检查约束更多时候仅仅出现在数据库设计阶段，真实环境却很少应用，更多是放到程序逻辑中去进行处理。这也比较容易理解，约束会一定程度上较低数据库性能，有些规则直接在程序逻辑中处理就可以了，同时，也有可能在面对业务变更或是系统扩展时，数据库约束会使得处理不够方便。不过在我看来，数据库约束是保证数据准确性的最后一道防线，对于设计合理的系统，处于性能考虑数据库约束自然可有可无；不过若是面对关联关系较为复杂的系统，且对系统而言，数据的准确性完整性要高于性能要求，那么这些约束还是有必要的（否则，就会出现各种相对业务规则来说莫名其妙的脏数据，本人可是深有体会的。。）。总之，对于约束的选择无所谓合不合理，需要根据业务系统对于准确性和性能要求的侧重度来决定。

数据库约束有五种： 

  * **主键约束（PRIMARY KEY）**
  * **唯一性约束（UNIQUE)**
  * **非空约束（NOT NULL)**
  * **外键约束（FOREIGN KEY)**
  * **检查约束（CHECK)**

下面我们就分别来看下这五类约束： 

## 数据库约束

**主键约束（PRIMARY KEY)**

主键是定位表中单个行的方式，可唯一确定表中的某一行，关系型数据库要求所有表都应该有主键，不过Oracle没有遵循此范例要求，Oracle中的表可以没有主键（这种情况不多见）。关于主键有几个需要注意的点： 

  1. 键列必须必须具有唯一性，且不能为空，其实主键约束 相当于 UNIQUE+NOT NULL
  2. 一个表只允许有一个主键
  3. 主键所在列必须具有索引（主键的唯一约束通过索引来实现），如果不存在，将会在索引添加的时候自动创建

添加主键（约束的添加可在建表时创建，也可如下所示在建表后添加，一般推荐建表后添加，灵活度更高一些，建表时添加某些约束会有限制） 
    
    
    SQL> alter table emp add constraint emp_id_pk primary key(id);

**唯一性约束（UNIQUE)**

唯一性约束可作用在单列或多列上，对于这些列或列组合，唯一性约束保证每一行的唯一性。 

UNIQUE需要注意： 

  1. 对于UNIQUE约束来讲，索引是必须的。如果不存在，就自动创建一个（UNIQUE的唯一性本质上是通过索引来保证的）
  2. UNIQUE允许null值，UNIQUE约束的列可存在多个null。这是因为，Unique唯一性通过btree索引来实现，而btree索引中不包含null。当然，这也造成了在where语句中用null值进行过滤会造成全表扫描。

添加唯一约束 
    
    
    SQL> alter table emp add constraint emp_code_uq unique(code);

**非空约束（NOT NULL)**

非空约束作用的列也叫强制列。顾名思义，强制键列中必须有值，当然建表时候若使用default关键字指定了默认值，则可不输入。

添加非空约束，语法较特别
    
    
    SQL> alter table emp modify ename not null;

**外键约束（FOREIGN KEY）**

外键约束定义在具有父子关系的子表中，外键约束使得子表中的列对应父表的主键列，用以维护数据库的完整性。不过出于性能和后期的业务系统的扩展的考虑，很多时候，外键约束仅出现在数据库的设计中，实际会放在业务程序中进行处理。外键约束注意以下几点：

  1. 外键约束的子表中的列和对应父表中的列数据类型必须相同，列名可以不同
  2. 对应的父表列必须存在主键约束（PRIMARY KEY）或唯一约束（UNIQUE）
  3. 外键约束列允许NULL值，对应的行就成了孤行了

其实很多时候不使用外键，很多人认为会让删除操作比较麻烦，比如要删除父表中的某条数据，但某个子表中又有对该条数据的引用，这时就会导致删除失败。我们有两种方式来优化这种场景：

第一种方式简单粗暴，删除的时候，级联删除掉子表中的所有匹配行，在创建外键时，通过 **on delete cascade** 子句指定该外键列可级联删除：
    
    
    SQL> alter table emp add constraint emp_deptno_fk foreign key(deptno) references dept (deptno) on delete cascade;

第二种方式，删除父表中的对应行，会将对应子表中的所有匹配行的外键约束列置为NULL，通过 **on delete set null** 子句实施：
    
    
    SQL> alter table emp add constraint emp_deptno_fk foreign key(deptno) references dept(deptno) on delete set null;

实际上，外键约束列和对应的父表列可以在同一张表中，常见的就是表的业务逻辑含义是一棵树，最简单的例子如下（id为主键id，fid为父id，fid存储对id的引用），这种结构的表根据业务要求可通过Oracle的递归查询来获取这种层级关系

![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ec73b5aedba182769b0a6abfa2c2abba.png)

**检查约束（CHECK)**

检查约束可用来实施一些简单的规则，比如列值必须在某个范围内。检查的规则必须是一个结果为true或false 的表达式，比如：
    
    
    SQL> alter table emp add constraint emp_sex_ck check(sex in('男','女'));

## 约束状态

很多时候由于业务需要，比如我们有大量的历史数据，需要和现有数据合并，当前表存在数据库约束（如非空约束），而这些历史数据又包含违背非空约束的数据行，为了避免导入时由于违反约束而导入失败，我们通过调整约束状态来达到目的。

数据库约束有两类状态

**启用/禁用（enable/disable）** ：是否对新变更的数据启用约束验证

**验证/非验证 (validate/novalidate)** ：是否对表中已客观存在的数据进行约束验证

这两类四种状态从语法角度讲可以随意组合，默认是 enable validate

下面我们来看着四类组合会分别出现什么样的效果：

**enable validate** : 默认的约束组合状态，无法添加违反约束的数据行，数据表中也不能存在违反约束的数据行；

**enable novalidate** : 无法添加违反约束的数据行，但对已存在的违反约束的数据行不做验证；

**disable validate** : 可以添加违反约束的数据行，但对已存在的违反约束的数据行会做约束验证（从描述中可以看出来，这本来就是一种相互矛盾的约束组合，只不过是语法上支持这种组合罢了，造成的结果就是会导致DML失败）

**disable novalidate** : 可以添加违法约束的数据行，对已存在的违反约束的数据行也不做验证。

拿上面的例子来说，我们需要上传大量违反非空约束的历史数据（从业务角度讲这些数据不会造成系统功能异常），可以临时将约束状态转为 disable novalidate，以保证这些不合要求的数据导入表中
    
    
    SQL> alter table emp modify constraint emp_ename_nn disable novalidate;

在数据导入完成之后，我们再将约束状态转为enable novalidate 以确保之后添加的数据不会再违反约束
    
    
    SQL> alter table emp modify constraint emp_ename_nn enable novalidate;

## 总结

本文介绍了数据库中的五类约束，也提到了数据库约束的四种状态组合，当你由于业务需求或是系统扩展，在一个约束严苛的系统中由于约束限制频繁操作失败的时候，不同组合的约束状态或许能给你另一种处理方案。谢谢支持。

作者： [dreamcatcher-cx](<http://www.cnblogs.com/chengxiao/>)

出处： [<http://www.cnblogs.com/chengxiao/>](<http://www.cnblogs.com/chengxiao/>)

本文版权归作者和博客园共有，欢迎转载，但未经作者同意必须保留此段声明，且在页面明显位置给出原文链接。 

分类: [Oracle](<https://www.cnblogs.com/chengxiao/category/880741.html>)

好文要顶 关注我 收藏该文 ![image](//common.cnblogs.com/images/icon_weibo_24.png) ![image](//common.cnblogs.com/images/wechat.png)

[![image](//pic.cnblogs.com/face/1024555/20170217101538.png)](<http://home.cnblogs.com/u/chengxiao/>)

[dreamcatcher-cx](<http://home.cnblogs.com/u/chengxiao/>)   
[关注 - 30](<http://home.cnblogs.com/u/chengxiao/followees>)   
[粉丝 - 659](<http://home.cnblogs.com/u/chengxiao/followers>)

+加关注 

1

0

[« ](<https://www.cnblogs.com/chengxiao/p/5979059.html>) 上一篇： [数据结构(一)之线性表](<https://www.cnblogs.com/chengxiao/p/5979059.html> "发布于2016-10-22 20:20")   
[» ](<https://www.cnblogs.com/chengxiao/p/6059914.html>) 下一篇： [HashMap实现原理及源码分析](<https://www.cnblogs.com/chengxiao/p/6059914.html> "发布于2016-11-16 00:27")   

    
    
    	</div>
    	<div class="postDesc">posted @ <span id="post-date">2016-11-06 22:34</span> <a href="https://www.cnblogs.com/chengxiao/">dreamcatcher-cx</a> 阅读(<span id="post_view_count">21348</span>) 评论(<span id="post_comment_count">0</span>)  <a href="https://i.cnblogs.com/EditPosts.aspx?postid=6032183" rel="nofollow">编辑</a> <a href="#" "AddToWz(6032183);return false;">收藏</a></div>
    </div>