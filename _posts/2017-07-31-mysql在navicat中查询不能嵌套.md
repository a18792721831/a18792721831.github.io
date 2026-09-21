---
layout: post
title: "mysql在navicat中查询不能嵌套"
date: 2017-07-31 12:41:56 +0800
categories: [mysql, 事务, select]
description: "本文通过一个具体的SQL示例探讨了事务嵌套的问题。实验证明，在Navicat中虽然可以进行事务的嵌套写法，但这种操作实际上会被自动提交，并不能实现预期中的回滚效果。文章最终得出了事务不可嵌套使用的结论。"
keywords: mysql, 事务, select
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/76423545
> - 发布时间：2017-07-31 12:41:56
> - 阅读量：1
> - 分类：java同时被 2 个专栏收录, 订阅专栏, mysql
> - 标签：#mysql, #事务, #select

## 摘要

文章浏览阅读1.4k次。本文通过一个具体的SQL示例探讨了事务嵌套的问题。实验证明，在Navicat中虽然可以进行事务的嵌套写法，但这种操作实际上会被自动提交，并不能实现预期中的回滚效果。文章最终得出了事务不可嵌套使用的结论。

---

事务可以嵌套吗？   
1.事务嵌套写，执行没有错误，但是在实际使用时候有问题。   
如下：   
SET autocommit = 0;   
START TRANSACTION;   
UPDATE emp SET ename = ‘jia’ WHERE empno = 7;   
SELECT * FROM emp;   
START TRANSACTION;   
UPDATE emp SET ename = ‘yong’ WHERE empno = 7;   
SELECT * FROM emp;   
START TRANSACTION;   
UPDATE emp SET ename = ‘qi’ WHERE empno = 7;   
SELECT * FROM emp;   
ROLLBACK;   
SELECT * FROM emp;   
ROLLBACK;   
SELECT * FROM emp;   
ROLLBACK;   
SELECT * FROM emp;   
COMMIT;   
SELECT * FROM emp;   
COMMIT;   
SELECT * FROM emp;   
COMMIT;   
SELECT * FROM emp;   
结果如下：   
![这里写图片描述](https://img-blog.csdn.net/20170731120153923?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
![这里写图片描述](https://img-blog.csdn.net/20170731123845139?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
![这里写图片描述](https://img-blog.csdn.net/20170731123918678?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
![这里写图片描述](https://img-blog.csdn.net/20170731124023082?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
![这里写图片描述](https://img-blog.csdn.net/20170731124049274?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
![这里写图片描述](https://img-blog.csdn.net/20170731124119685?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
所以，能不能得出以下结论：   
1.事务不可以嵌套；   
2.事务的嵌套写法在navicat中不会报错，但是会自动提交；   
3.如果1成立，那么，不存在多个rollback；