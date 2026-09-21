---
layout: post
title: "spring boot 整合JPA"
date: 2020-02-19 18:59:44 +0800
categories: [spring boot jpa, springboot整合jpa, jpa常用, jpa测试类, DSC]
description: "本文详细介绍SpringBoot项目中整合JPA的过程，包括创建项目、配置数据源、实体类定义、DAO层操作、Service接口及实现、扩展JPA查询、Controller层设计与测试验证等关键步骤，提供了一个完整的JPA应用案例。"
keywords: spring boot jpa, springboot整合jpa, jpa常用, jpa测试类, DSC
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104380311
> - 发布时间：2020-02-19 18:59:44
> - 阅读量：898
> - 分类：微服务同时被 3 个专栏收录, 订阅专栏, spring boot, JPA
> - 标签：#spring boot jpa, #springboot整合jpa, #jpa常用, #jpa测试类, #DSC

## 摘要

文章浏览阅读898次。本文详细介绍SpringBoot项目中整合JPA的过程，包括创建项目、配置数据源、实体类定义、DAO层操作、Service接口及实现、扩展JPA查询、Controller层设计与测试验证等关键步骤，提供了一个完整的JPA应用案例。

---

#### spring boot 整合JPA

  * [1\. 创建gradle项目](<#1_gradle_4>)
  * [2\. 配置数据源](<#2__7>)
  * [3\. 实体类](<#3__15>)
  * [4\. DAO](<#4_DAO_21>)
  * [5\. service接口](<#5_service_25>)
  * [6\. 逻辑删除](<#6__29>)
  * [7\. serviceimpl](<#7_serviceimpl_37>)
  * [8\. 实现扩展jpa查询](<#8_jpa_39>)
  * [9\. controller](<#9_controller_44>)
  * [10\. 测试类](<#10__46>)
  *     * [10.1 dao](<#101_dao_47>)
    * [10.2 service](<#102_service_53>)
    * [10.3 controller](<#103_controller_60>)
    * [10.4 真实验证](<#104__67>)

  
git地址   
https://github.com/a18792721831/studySpringCloud.git   
JPA 全称为JAVA Persistence API，它是一个数据持久化的类和方法的集合。JPA的目标是制定一个由很多数据库供应商实现的API，开发人员可以通过编码实现该API。目前，在Java项目开发中提到JPA一般是指用Hibernate 的实现，因为在Java的ORM框架中，只有 Hibernate实现得最好。 

## 1\. 创建gradle项目

创建一个新的gradle项目，选择依赖即可  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/701bcf47388e9dde14c5f5ec20a3d2ef.png)

## 2\. 配置数据源

在工程的配置文件 application.yml文件中加上相应的配置，需要配置两个选项，DataSource数据源的配置和JPA的配置。其中，数据源的配置包括连接oracle的驱动类（例如com.oracle.jdbc.Driver)、oracle 数据库的地址 Url、oracle数据库的用户名username 和密码 password，JPA 的配置包括hibernate. ddl-auto 配置，配置为create时，程序启动时会在oracle数据库创建表：配置为 update时，在程序启动时不会在oracle数据库中建表：jpa.show.sql配置为在通过JPA 操作数据库时是否显示操作的SQL语句。配置代码如下：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6e2dd3965d56925607ac01a4aa325d2f.png)  
不要担心，当依赖下载完，在配置文件中写配置是有提示的(自动补全)

## 3\. 实体类

通过@Entity 注解表明该类是一个实体类，它和数据库的表名相对应；@Id注解表明该变  
量对应于数据库中的Id，@GeneratedValue 注解配置. Id 字段为自增长；@Column 表明该变量对应于数据库表中的字段，unique＝true 表明该变量对应于数据库表中的字段为唯一约束。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c7d5d087b83efe90d88ddd0dcebd53f1.png)

## 4\. DAO

数据访问层DAO，通过编写一个UserDao类，该类继承 JpaRepository的接口，继承之  
后就能对数据库进行读写操作，包含了基本的单表查询的方法，非常方便。在UserDao类写一个findByUsername的方法，传入参数username，JPA已经实现了根据某个字段去查找的方法所以该方法可以根据username字段从数据库中获取User的数据，不需要做额外的编码。代码如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f3ba469e39a5e63474e50b98beb47c86.png)

## 5\. service接口

service 接口定义提供哪些服务  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5fd2fc839dc3fa113871a5be2d2d822e.png)  
ps：service接口本身不会注入spring容器，而是将service的实现以service 接口的名字注入spring容器

## 6\. 逻辑删除

为了实现逻辑删除，需要有一个字段来标记记录是否有效。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c5b1f052ac049c49872927794ffcb942.png)  
创建枚举，有name和value属性，使用lombok注解生成get方法，然后实现from方法，根据vale获取枚举，外层使用optional进行处理空指针异常。  
然后实现转换类  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5be713637c1db961c57aa1d37cfe8cbf.png)  
接着在属性中指定使用哪个转换类  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0c5ca213b15330e7b5c091260463a913.png)

## 7\. serviceimpl

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0fde1d6f8bb5d1ec79aebf44432dcbea.png)

## 8\. 实现扩展jpa查询

实现根据名字和状态进行查询，jpa中没有已经实现的，只能自己实现  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/990090b292df84a02ef0e02598b0a418.png)  
然后就可以使用了  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1c2f93c384041dedfd67bb8069b2fdcd.png)

## 9\. controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/726b6ffbfe4c80741a188d40ca8d5874.png)

## 10\. 测试类

### 10.1 dao

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4049e66d16ac5ddf229589c17caa42e8.png)  
运行  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/750140d8a4c925eb9b4bd6f51964560c.png)  
验证  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/10e9edd1ba69cfdad632276f11fbd3a7.png)

### 10.2 service

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88be8b0019053b9c0155963780372509.png)  
测试结果  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/168b306078421ffafea65ba625856471.png)  
验证  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/35717b7c7f2d5f4bd9421167bcc1d6dc.png)  
这两条是新生成的，不是dao测试生成的，其值不同

### 10.3 controller

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e5e9ffe101e2745f4cf87aa38f73ccb9.png)  
其id是在service中新增的  
测试结果  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/98b7062ec4f598d9ecff0e390f3b8873.png)  
每次测试完成会将数据删除  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b0400833e8b22308f5c0f35b419eacf6.png)

### 10.4 真实验证

首先手动往数据库表中增加一条记录  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b4c64d3416c8cf7c9083df8aea746f1.png)  
启动  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88b5d523ce7cdfe574855b872e22b54a.png)  
然后在浏览器验证查询的controller  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bfd94425b8a54a4a229f054614cd19ff.png)