---
layout: post
title: "java Calendar类的add方法与oracle的add_months方法的使用"
date: 2019-11-07 18:54:36 +0800
categories: [java Calendar, java时间计算, oracle ADD_MONTHS, oracle时间计算, 同样的计算方式不同计算结果]
description: "本文探讨了Java中Calendar类的add方法与Oracle数据库中ADD_MONTHS函数在处理日期时的差异，特别是在处理月底日期时的不同表现。Java不考虑月底概念，而Oracle则会根据下一个月的实际天数来确定月底日期。"
keywords: java Calendar, java时间计算, oracle ADD_MONTHS, oracle时间计算, 同样的计算方式不同计算结果
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/102960826
> - 发布时间：2019-11-07 18:54:36
> - 阅读量：1
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#java Calendar, #java时间计算, #oracle ADD_MONTHS, #oracle时间计算, #同样的计算方式不同计算结果

## 摘要

文章浏览阅读1.4k次。本文探讨了Java中Calendar类的add方法与Oracle数据库中ADD_MONTHS函数在处理日期时的差异，特别是在处理月底日期时的不同表现。Java不考虑月底概念，而Oracle则会根据下一个月的实际天数来确定月底日期。

---

#### java Calendar类的add方法与oracle的add_months方法的使用

  * 1.java Calendar类
  * 2.oracle ADD_MONTHS
  * 3.为什么不同

  
最近程序中出现了一个非常怪异的问题：   
在java程序中时间范围是2019.9.30~2019.10.30   
但是在c++的程序用的是2019.9.30~2019.10.31 

java 程序负责往oracle数据库中写入数据，C++程序从oracle程序中读取数据。

一写一读怎么就出现问题了呢？

而且客户反映，这个问题比较特殊，只有开始时间是月底，且时间段是1个月的才会复现。

真是见了鬼了。

ok，查问题。  
发现c++从oracle数据库中查询数据时并不是直接查询截止时间，而是根据开始时间+月份数计算的，使用的就是oracle里面的一个函数ADD_MONTHS。  
这个时间就出现了偏差。

真是坑啊。

java中使用Calendar类的add方法计算的，所以oracle中就使用相同的业务逻辑进行处理。

测试正常的数据也没有什么问题。

但是，就是月底这个数据就出现问题了。。。。。。。。。。

oracle有月底概念，java没有月底概念。 

## 1.java Calendar类
    
    
    public class MyCl {
    
    	@Test
    	public void getNextMonthOneDate() throws ParseException {
    		SimpleDateFormat dateFormat = new SimpleDateFormat(
    				"yyyy-MM-dd");
    		Date date = dateFormat.parse("2019-1-31");
    		System.out.println(dateFormat.format(date));
    		Calendar calendar = Calendar.getInstance();
    		calendar.setTime(date);
    		calendar.add(Calendar.MONTH, 1);
    //		calendar.add(Calendar.SECOND, -1);
    		Date end = new Date(calendar.getTimeInMillis());
    		System.out.println(dateFormat.format(end));
    	}
    	
    }
    

就这样一个简单的程序，我们计算下时间：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7ffc3465c71caa82c4d07316b1be1bf9.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/deb9502a7c8328dcdc0791d3a0fe2590.png)  
是的，我测试用的jdk的版本为6u38版本（还有公司在用吗？）

## 2.oracle ADD_MONTHS

oracle的ADD_MONTHS是oracle实现的函数，接收2个参数，返回一个参数。  
第一个参数为时间，第二个参数为整数，返回时间。
    
    
    SELECT TO_CHAR(ADD_MONTHS(TO_DATE('2020-4-30', 'yyyy-mm-dd'), 1),
                   'yyyy-mm-dd')
      FROM DUAL;
    
    

我们用这个SQL语句对同样的时间进行查询：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ffa67cf571eeb37304815309280c1642.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7ae511e19c1d1e92a3547f9be7ba5d07.png)  
oracle的版本  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a38a677a9e23434fb67698c3467431e2.png)

## 3.为什么不同

经过查看jdk源码，oracle的实现以及自己测试数据，得出下面的结论。（如有问题，还请指正）  
在oracle中，存在两种概念：  
对于普通日期，对日期加1月，就是说下一个是什么日期。  
举个例子：  
3月5日过一个月是几月几号？  
4月5日。

对于月底日期，对日期加1月，就是说下一个月的月底是什么日期。  
举个例子：  
3月5日下一个月月底是几月几号？  
4月30日。

总结：  
x年y月z日加n月  
=> (x+(y+n)/13)年((y+n) > 12 ? ((y+n)%12 == 0 ? 12:(y+n)%12) : (y+n))月min(dayofmon(y),dayofmon((y+n) > 12 ? (y+n)%12 : (y+n))) || enddayofmon((y+n) > 12 ? (y+n)%12 : (y+n))日  
举个例子：  
2019年5月4日+4月  
=>2019年9月4日  
2019年5月31日+8月  
=> 2020年1月31日  
2019年2月27日+22月  
2019+24/13=2020  
24%12 = 0 => 12  
=>2020年12月27日  
2019年2月28日+22月  
=>2020年12月31日  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b56d14c9fd65245f3f20524c0039a5b6.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/accd6f0860819b4f30668678a2af8ca9.png)

在java中就只有普通日期，没有月底日期这个概念：  
举个例子：  
2019.1.31  
=>2019.2.28

2019.2.28  
=>2019.3.28

总结：  
x年y月z日加n月  
=> (x+(y+n)/13)年((y+n) > 12 ? ((y+n)%12 == 0 ? 12:(y+n)%12) : (y+n))月min(dayofmon(y),dayofmon((y+n) > 12 ? (y+n)%12 : (y+n))) 日  
举个例子：  
2019年5月4日+4月  
=>2019年9月4日  
2019年5月31日+8月  
=>2020年1月31日  
2019年2月27日+22月  
2019+24/13 = 2020  
24%12 = 0 =>12  
=>2020年12月27日  
2019年2月28日+22月  
=>2020年12月28日  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e642f4ddf108764dd4d80b926c8850f3.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fa18da579ac26c04d7627a3a545e25e1.png)

说了这么多，这两种计算日期的方式有什么区别？ oracle有月底概念，java没有月底概念。
