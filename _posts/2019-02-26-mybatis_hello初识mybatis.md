---
layout: post
title: "mybatis_hello初识mybatis"
date: 2019-02-26 20:04:03 +0800
categories: [MyBatis, 如何创建MyBatis项目, MyBatis工作原理, MyBatis的执行步骤, MyBatis与oracle数据库集成]
description: "本文详细介绍MyBatis框架的基本概念、工作原理及实战应用，包括环境搭建、配置文件解析、映射文件创建、Java类设计及日志配置等关键步骤。"
keywords: MyBatis, 如何创建MyBatis项目, MyBatis工作原理, MyBatis的执行步骤, MyBatis与oracle数据库集成
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87946151
> - 发布时间：2019-02-26 20:04:03
> - 阅读量：392
> - 分类：java同时被 3 个专栏收录, 订阅专栏, mybatis, MyBatis
> - 标签：#MyBatis, #如何创建MyBatis项目, #MyBatis工作原理, #MyBatis的执行步骤, #MyBatis与oracle数据库集成

## 摘要

文章浏览阅读392次。本文详细介绍MyBatis框架的基本概念、工作原理及实战应用，包括环境搭建、配置文件解析、映射文件创建、Java类设计及日志配置等关键步骤。

---

#### MyBatis初体验

  * 1.简介
  * 2.入门
  * 3.MyBatis工作原理
  * 4.例子
  *     * 4.1准备
    * 4.2创建一个Java工程
    * 4.3创建Java类
    * 4.4 xml文件
    * 4.5 配置文件
    * 4.6运行
  * 5.总结

## 1.简介

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

## 2.入门

<https://codeload.github.com/mybatis/mybatis-3/zip/mybatis-3.5.0>  
下载jar包  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e5c93c7fe6ea85195066910f0ad798f1.png)  
解压  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71e60a44a9bd66bfbe3eed7620d3a2b9.png)

## 3.MyBatis工作原理

1.读取MyBatis配置文件；  
2.加载映射文件；  
3.构建会话工厂；  
4.创建SQLSession对象；  
5.MyBatis底层定义了一个Executor接口操作数据库，根据SqlSession传递的参数动态生成需要执行的SQL语句，同时负责查询缓存的维护；  
6.在Executor接口的执行方法中，包含一个MappedStatement类型的参数，此参数是对映射信息的封装；  
7.输入参数映射；  
8.输出结果映射。

## 4.例子

### 4.1准备

MyBatis的jar包：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a57f817914d71384122a8ffac469c48b.png)  
ojdbc驱动的jar包  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fab626c2ffc18872368182058ce21dc2.png)  
注意ojdbc和jdk的对应关系。

### 4.2创建一个Java工程

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7bace67cab445e0b16be2e78ccecc4cb.png)  
jdk1.8  
导入jar包  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df16e951521d87f9b06863868d8f7ec7.png)  
其中除去ojdbc与mybatis的jar包，其余都是下载的mybatis的jar包中的lib中的文件。

### 4.3创建Java类

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/56a91af7d3130b9f67a31c1e06840fe0.png)
    
    
    package domain;
    
    import java.io.Serializable;
    
    public class People implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = -3270893239281340723L;
    
    	private Long id;
    	
    	private String name;
    	
    	private Integer age;
    	
    	private Integer sex;
    
    	public Long getId() {
    		return id;
    	}
    
    	public void setId(Long id) {
    		this.id = id;
    	}
    
    	public String getName() {
    		return name;
    	}
    
    	public void setName(String name) {
    		this.name = name;
    	}
    
    	public Integer getAge() {
    		return age;
    	}
    
    	public void setAge(Integer age) {
    		this.age = age;
    	}
    
    	public Integer getSex() {
    		return sex;
    	}
    
    	public void setSex(Integer sex) {
    		this.sex = sex;
    	}
    
    	@Override
    	public String toString() {
    		return "people [id=" + this.id + ",name=" + this.name + ",age="
    				+ this.age + ",sex=" + this.sex;
    	}
    	
    	
    }
    
    
    
    
    package client;
    
    import java.io.IOException;
    import java.io.InputStream;
    
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    
    import domain.People;
    
    public class Main {
    
    	public static void main(String[] args) throws IOException {
    		String resource = "resource/mybatis.xml";
    		// 1.读取配置文件
    		InputStream inputStream = Resources.getResourceAsStream(resource);
    		// 2.根据配置文件构建SqlSessionFactory
    		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder()
    				.build(inputStream);
    		// 3.通过SqlSessionFactory创建SqlSession
    		SqlSession sqlSession = sqlSessionFactory.openSession();
    		// 4.执行mapper中的sql，返回结果
    		People people = sqlSession.selectOne(
    				"mapper.PeopleMapper.selectPeopleById", Long.valueOf(101));
    		System.out.println(people);
    		sqlSession.close();
    	}
    
    }
    
    

### 4.4 xml文件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/73921480cf207ff24f2fcc21d7411aa6.png)  
PeopleMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="mapper.PeopleMapper">
      <select id="selectPeopleById" resultType="domain.People"
      	parameterType="Long">
        select * from people where id = #{id}
      </select>
    </mapper>
    
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
      <environments default="oracle">
        <environment id="oracle">
          <transactionManager type="JDBC"/>
          <dataSource type="POOLED">
            <property name="driver" value="oracle.jdbc.driver.OracleDriver"/>
            <property name="url" value="jdbc:oracle:thin:@127.0.0.1:1521:oracle"/>
            <property name="username" value="study"/>
            <property name="password" value="study"/>
          </dataSource>
        </environment>
      </environments>
      <mappers>
        <mapper resource="mapper/PeopleMapper.xml"/>
      </mappers>
    </configuration>
    

### 4.5 配置文件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25c0db0bfcd173163b6e9c550983f3ea.png)  
log4j.properties
    
    
    # Global logging configuration
    log4j.rootLogger=ERROR, stdout
    # MyBatis logging configuration...
    log4j.logger.org.mybatis.example.BlogMapper=TRACE
    # Console output...
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
    log4j.logger.domain=DEBUG
    

### 4.6运行
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    people [id=101,name=aPeople,age=53,sex=0]
    

## 5.总结

主要有几个点：  
1.jdk与ojdbc对应  
2.jdk与mybatis对应  
3.log4j配置  
4.jdbc的url  
5.出现bug如何调试  
log4j配置  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2382643abac95c420f1af3a74ca6fbf1.png)  
红框中是自己需要打印日志的包的路径。  
优化的点：  
jdbc的配置应该使用propertis配置，而不是写死:

新增ojdbc.properties文件：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7311849133222329a850709a01de671d.png)  
ojdbc.properties
    
    
    driver=oracle.jdbc.driver.OracleDriver
    url=jdbc:oracle:thin:@127.0.0.1:1521:oracle
    username=study
    password=study
    

原mybatis.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
      <properties resource="property/ojdbc.properties"></properties>
      <environments default="oracle">
        <environment id="oracle">
          <transactionManager type="JDBC"/>
          <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
          </dataSource>
        </environment>
      </environments>
      <mappers>
        <mapper resource="mapper/PeopleMapper.xml"/>
      </mappers>
    </configuration>
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2dc27540cf0b87f1c30ad1522fb69679.png)  
第一个框内是读取配置文件；  
第二个框是引用读取的值，以字符串的方式拼接。

运行结果与原来相同。
