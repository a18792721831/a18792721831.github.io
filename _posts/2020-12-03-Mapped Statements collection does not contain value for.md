---
layout: post
title: "Mapped Statements collection does not contain value for"
date: 2020-12-03 11:32:09 +0800
categories: [Mapped Stat, Statements, collection, does notcontain, containvaluefor]
description: "IllegalArgumentException: Mapped Statements collection does not contain value for  XXX。说下我的异常背景：我在使用spring batch的MyBatisCursorItemReader的时候，传入queryId，提示这个异常。经过调试后发现，因为我是在@Bean中去启动job的。在启动Job的时候，MyBatis的Mapper映射还未解析完成，所以找不到我想要的queryId。简单来说：使用@Bean注解的be_mapped statements collection does not contain value for com.ht.core.entity.m"
keywords: Mapped Stat, Statements, collection, does notcontain, containvaluefor
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/110518974
> - 发布时间：2020-12-03 11:32:09
> - 阅读量：832
> - 分类：mybatis同时被 2 个专栏收录, 订阅专栏, spring batch
> - 标签：#Mapped Stat, #Statements, #collection, #does notcontain, #containvaluefor

## 摘要

文章浏览阅读832次。IllegalArgumentException: Mapped Statements collection does not contain value for  XXX。说下我的异常背景：我在使用spring batch的MyBatisCursorItemReader的时候，传入queryId，提示这个异常。经过调试后发现，因为我是在@Bean中去启动job的。在启动Job的时候，MyBatis的Mapper映射还未解析完成，所以找不到我想要的queryId。简单来说：使用@Bean注解的be_mapped statements collection does not contain value for com.ht.core.entity.m

---

IllegalArgumentException: Mapped Statements collection does not contain value for XXX。  
说下我的异常背景：  
我在使用spring batch的MyBatisCursorItemReader的时候，传入queryId，提示这个异常。  
经过调试后发现，因为我是在@Bean中去启动job的。在启动Job的时候，MyBatis的Mapper映射还未解析完成，所以找不到我想要的queryId。

简单来说：  
使用@Bean注解的bean先于Mapper解析加载，如果在@Bean中使用到了Mapper，那么就会出现这个异常。

解决方案：  
1.使用@AutoConfigureAfter注解不好使  
2.使用@Autowired解决

既然我们在@Bean中用到了Mapper，那么就需要spring容器先将Mapper装配好，我们在Bean中才能使用。所以使用@Autowired可以让Mapper先于Bean装配。

异常堆栈
    
    
    Caused by: java.lang.IllegalArgumentException: Mapped Statements collection does not contain value for com.study.study6itemreader.dao.CustomerQuery.queryCustomer
    	at org.apache.ibatis.session.Configuration$StrictMap.get(Configuration.java:964) ~[mybatis-3.5.4.jar:3.5.4]
    	at org.apache.ibatis.session.Configuration.getMappedStatement(Configuration.java:755) ~[mybatis-3.5.4.jar:3.5.4]
    	at org.apache.ibatis.session.Configuration.getMappedStatement(Configuration.java:748) ~[mybatis-3.5.4.jar:3.5.4]
    	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectCursor(DefaultSqlSession.java:122) ~[mybatis-3.5.4.jar:3.5.4]
    	... 66 common frames omitted
    

解决方式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2939e45365356b79359ebc7aaa5a14e7.png)  
虽然我们实际上没有用到CustomerQuery这个Mapper，但是这样写可以保证在装配runJob的时候，Mapper已经装配完成了。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/53620f77b8ae9a0ea3d67247ccbc14b5.png)  
这只是我的解决方式，希望对你有用。