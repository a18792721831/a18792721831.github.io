---
layout: post
title: "spring&mybatis整合--DAO方式"
date: 2019-03-11 20:20:51 +0800
categories: [spring&amp;amp;mybatis整合, spring&amp;amp;mybatis传统开发方式整合, spring&amp;amp;mybatis整合自定向下开发, 完整的spring&amp;amp;mybatis项目, 一步步教你学会spring&amp;amp;mybatis]
description: "本文详细介绍了一种自顶向下的Spring与MyBatis整合方式，包括从搭建环境到完成多对多关系映射的全过程。通过具体步骤指导读者如何创建Java工程、配置数据源、实现DAO模式、编写Mapper文件及测试类，最终实现人员与服务的增删查改功能。"
keywords: spring&amp;amp;mybatis整合, spring&amp;amp;mybatis传统开发方式整合, spring&amp;amp;mybatis整合自定向下开发, 完整的spring&amp;amp;mybatis项目, 一步步教你学会spring&amp;amp;mybatis
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88407233
> - 发布时间：2019-03-11 20:20:51
> - 阅读量：626
> - 分类：java同时被 3 个专栏收录, 订阅专栏, spring
> - 标签：#spring&amp;amp;mybatis整合, #spring&amp;amp;mybatis传统开发方式整合, #spring&amp;amp;mybatis整合自定向下开发, #完整的spring&amp;amp;mybatis项目, #一步步教你学会spring&amp;amp;mybatis

## 摘要

文章浏览阅读626次。本文详细介绍了一种自顶向下的Spring与MyBatis整合方式，包括从搭建环境到完成多对多关系映射的全过程。通过具体步骤指导读者如何创建Java工程、配置数据源、实现DAO模式、编写Mapper文件及测试类，最终实现人员与服务的增删查改功能。

---

#### spring&mybatis整合--DAO方式--自顶向下的方式

  * 1.新建一个Java工程
  * 2.导入jar包
  * 3.数据库准备
  * 4.创建实体
  * 5.创建增强类
  * 6.创建日志配置
  * 7.创建工具类
  * 8.配置数据源
  * 9.创建工程配置
  * 10.增加服务
  * 11.增加Dao
  * 12.服务的实现
  * 13.Dao的实现
  * 14.mapper编写
  * 15.编写测试类
  * 16.运行结果

## 1.新建一个Java工程

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f3f8043d891f34ca921d93961212f7d4.png)

## 2.导入jar包

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/461f360de7910833f95f3f299dbd52b2.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5de19479fcee5b0d185b7da2ebec3f54.png)

## 3.数据库准备
    
    
    -- Create table
    create table PEOPLE
    (
      id   NUMBER not null,
      name VARCHAR2(50) not null,
      age  NUMBER not null,
      sex  NUMBER not null
    )
    tablespace USERS
      pctfree 10
      initrans 1
      maxtrans 255
      storage
      (
        initial 64K
        next 1M
        minextents 1
        maxextents unlimited
      );
    -- Add comments to the columns 
    comment on column PEOPLE.id
      is '人的id,不可为空,不可重复,主键';
    comment on column PEOPLE.name
      is '人的姓名，不可为空,最大值50';
    comment on column PEOPLE.age
      is '人的年龄，不可为空';
    comment on column PEOPLE.sex
      is '人的性别，不可为空';
    -- Create/Recreate primary, unique and foreign key constraints 
    alter table PEOPLE
      add constraint PEOPLE_PK primary key (ID)
      using index 
      tablespace USERS
      pctfree 10
      initrans 2
      maxtrans 255
      storage
      (
        initial 64K
        next 1M
        minextents 1
        maxextents unlimited
      );
    -- Create/Recreate check constraints 
    alter table PEOPLE
      add constraint PEOPLE_AGE
      check (age < 150 AND age > 0);
    alter table PEOPLE
      add constraint PEOPLE_SEX
      check (sex IN (0,1));
    
    
    
    
    -- Create table
    create table SERVER
    (
      id     NUMBER not null,
      name   VARCHAR2(50) not null,
      remark VARCHAR2(50)
    )
    tablespace USERS
      pctfree 10
      initrans 1
      maxtrans 255
      storage
      (
        initial 64K
        next 1M
        minextents 1
        maxextents unlimited
      );
    -- Add comments to the table 
    comment on table SERVER
      is '服务表:存储所有的服务';
    -- Add comments to the columns 
    comment on column SERVER.id
      is '服务的id,不可以为空，主键，不可重复';
    comment on column SERVER.name
      is '服务的名称,不可以为空';
    comment on column SERVER.remark
      is '服务的说明,可以为空';
    -- Create/Recreate primary, unique and foreign key constraints 
    alter table SERVER
      add constraint SERVICE_PK primary key (ID)
      using index 
      tablespace USERS
      pctfree 10
      initrans 2
      maxtrans 255
      storage
      (
        initial 64K
        next 1M
        minextents 1
        maxextents unlimited
      );
    
    
    
    
    -- Create table
    create table PEOPLE_SERVER
    (
      people_id NUMBER not null,
      server_id NUMBER not null
    )
    tablespace USERS
      pctfree 10
      initrans 1
      maxtrans 255
      storage
      (
        initial 64K
        next 1M
        minextents 1
        maxextents unlimited
      );
    -- Create/Recreate primary, unique and foreign key constraints 
    alter table PEOPLE_SERVER
      add constraint PEOPLE_FK foreign key (PEOPLE_ID)
      references PEOPLE (ID) on delete cascade;
    alter table PEOPLE_SERVER
      add constraint SERVICE_FK foreign key (SERVER_ID)
      references SERVER (ID) on delete cascade;
    
    

## 4.创建实体

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6ffcb85849df971908648abe41d5eae0.png)
    
    
    package domain;
    
    import java.io.Serializable;
    import java.util.List;
    
    import org.springframework.context.annotation.Scope;
    import org.springframework.stereotype.Component;
    
    @Component(value = "people")
    @Scope("prototype")
    public class People implements Serializable {
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = -3270893239281340723L;
    
    	private Long id;
    
    	private String name;
    
    	private Integer age;
    
    	private Integer sex;
    
    	private List<Server> servers;
    	
    	public List<Server> getServers() {
    		return servers;
    	}
    
    	public void setServers(List<Server> servers) {
    		this.servers = servers;
    	}
    
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
    				+ this.age + ",sex=" + this.sex + ",server=" + this.servers
    				+ "]";
    	}
    
    }
    
    
    
    
    package domain;
    
    import java.io.Serializable;
    
    import org.springframework.context.annotation.Scope;
    import org.springframework.stereotype.Component;
    
    @Component(value = "peopleServer")
    @Scope("prototype")
    public class PeopleServer implements Serializable {
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 7074684736563312938L;
    
    	private Long people_fk;
    
    	private Long server_fk;
    
    	public Long getPeople_fk() {
    		return people_fk;
    	}
    
    	public void setPeople_fk(Long people_fk) {
    		this.people_fk = people_fk;
    	}
    
    	public Long getServer_fk() {
    		return server_fk;
    	}
    
    	public void setServer_fk(Long server_fk) {
    		this.server_fk = server_fk;
    	}
    
    	@Override
    	public String toString() {
    		return "[people_id="+this.people_fk+",server_id="+this.server_fk+"]";
    	}
    
    }
    
    
    
    
    package domain;
    
    import java.io.Serializable;
    import java.util.List;
    
    import org.springframework.context.annotation.Scope;
    import org.springframework.stereotype.Component;
    
    @Component(value = "server")
    @Scope("prototype")
    public class Server implements Serializable {
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 6140189429513974999L;
    
    	private Long id;
    
    	private String name;
    
    	private String remark;
    
    	private List<People> peoples;
    
    	public List<People> getPeoples() {
    		return peoples;
    	}
    
    	public void setPeoples(List<People> peoples) {
    		this.peoples = peoples;
    	}
    
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
    
    	public String getRemark() {
    		return remark;
    	}
    
    	public void setRemark(String remark) {
    		this.remark = remark;
    	}
    
    	@Override
    	public String toString() {
    		return "[id=" + this.id + ",name=" + this.name + ",remark="
    				+ this.remark + ",peoples=" + this.peoples + "]";
    	}
    
    }
    
    

## 5.创建增强类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3e78934991dcaf285bc9f6eb133881b6.png)
    
    
    package aspect;
    
    import java.text.SimpleDateFormat;
    import java.util.Date;
    
    import org.aspectj.lang.JoinPoint;
    import org.aspectj.lang.annotation.AfterReturning;
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Before;
    import org.aspectj.lang.annotation.Pointcut;
    import org.springframework.stereotype.Component;
    
    @Aspect
    @Component("myAspect")
    public class MyAspect {
    
    	@Pointcut("execution(* dao.*+.*(..))")
    	public void pointCutDao(){}
    	
    	@Pointcut("execution(* service.*+.*(..))")
    	public void pointCutService(){}
    	
    	@Pointcut("execution(* service.*+.*(..))")
    	public void pointCutTransaction(){}
    	
    	@Before("pointCutDao()")
    	public void beforeDao(JoinPoint joinPoint){
    		System.out.println("Dao     Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    before");
    	}
    	
    	@Before("pointCutService()")
    	public void beforeService(JoinPoint joinPoint){
    		System.out.println("Service Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    before");
    	}
    	
    	@Before("pointCutTransaction()")
    	public void beforeTransaction(JoinPoint joinPoint){
    		System.out.println("Transaction Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    before");
    	}
    	
    	@AfterReturning("pointCutDao()")
    	public void afterReturnDao(JoinPoint joinPoint){
    		System.out.println("Dao     Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    afterReturn");
    	}
    	
    	@AfterReturning("pointCutService()")
    	public void afterReturnService(JoinPoint joinPoint){
    		System.out.println("Service Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    afterReturn");
    	}
    	
    	@AfterReturning("pointCutTransaction()")
    	public void afterReturnTransaction(JoinPoint joinPoint){
    		System.out.println("Transaction Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    afterReturn");
    	}
    	
    	private String showTime(){
    		return "  "+new SimpleDateFormat("yyyy-mm-dd HH:mm:ss,s").format(new Date())+"  ";
    	}
    }
    
    

## 6.创建日志配置

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/71cd00a93236f03a6ca84f8414d82bfe.png)
    
    
    # Global logging configuration
    log4j.rootLogger=DEBUG, stdout
    # MyBatis logging configuration...
    log4j.logger=DEBUG
    # Console output...
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
    
    

## 7.创建工具类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d0d300b55c56317a46d9af46cc382d3c.png)
    
    
    package util;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    public class MyApplicationFactory {
    
    	private static final ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    			"resource/*.xml");
    	
    	//使用泛型
    	public static <T> T getBean(Class<T> clazz){
    		return applicationContext.getBean(clazz);
    	}
    	
    	public static ApplicationContext getApplicationContext(){
    		return applicationContext;
    	}
    }
    
    

## 8.配置数据源

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/29b403f2665cfa46ab3fdd2b26038e37.png)
    
    
    ojdbc.driver=oracle.jdbc.driver.OracleDriver
    ojdbc.url=jdbc:oracle:thin:@127.0.0.1:1521:oracle
    ojdbc.username=study
    ojdbc.password=study
    #最大连接数
    ojdbc.maxTotal=30
    #最大空闲连接数
    ojdbc.maxIdle=10
    #初始化连接数
    ojdbc.initialSize=5
    

## 9.创建工程配置

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3feff7eedcd267f472a0179a8ab4f51f.png)  
aop.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:aop="http://www.springframework.org/schema/aop"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/aop
    	http://www.springframework.org/schema/aop/spring-aop.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd">
    	<!-- 指定需要扫描的包 -->
    	<context:component-scan base-package="domain,aspect,daoImpl,serviceImpl,util,test">
    	</context:component-scan>
    	<!-- 启动基于注解的声明式AspectJ支持 -->
    	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
    </beans>
    

beanx.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:aop="http://www.springframework.org/schema/aop"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/aop
    	http://www.springframework.org/schema/aop/spring-aop.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd">
    	<!-- 开启注解注入的装配方式 -->
    	<context:annotation-config></context:annotation-config>
    </beans>
    

dataBase.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd">
    	<!--  加载配置文件 -->
    	<context:property-placeholder location="classpath:properties/ojdbc.properties"/>
    	<!-- 配置数据源 -->
    	<bean id="dataSource" 
    		class="org.apache.commons.dbcp2.BasicDataSource">
    		<!-- 数据库驱动 -->
    		<property name="driverClassName">
    			<value>${ojdbc.driver}</value>
    		</property>
    		<!-- 连接数据库的url -->
    		<property name="url">
    			<value>${ojdbc.url}</value>
    		</property>
    		<property name="username">
    			<value>${ojdbc.username}</value>
    		</property>
    		<property name="password">
    			<value>${ojdbc.password}</value>
    		</property>
    		<property name="maxTotal">
    			<value>${ojdbc.maxTotal}</value>
    		</property>
    		<property name="maxIdle">
    			<value>${ojdbc.maxIdle}</value>
    		</property>
    		<property name="initialSize">
    			<value>${ojdbc.initialSize}</value>
    		</property>
    	</bean>
    	<!-- 配置MyBatis工厂 -->
    	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    		<property name="dataSource" ref="dataSource"></property>
    		<property name="configLocation">
    			<value>classpath:resource/mybatis.xml</value>
    		</property>
    	</bean>
    </beans>
    

mybatis.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
      <settings>
       	<!-- 延迟加载全局 -->
      	<setting name="lazyLoadingEnabled" value="true"/>
      	<!-- 关联对象属性的延迟加载 -->
      	<setting name="aggressiveLazyLoading" value="false"/>
      </settings>
      <!-- 改变运行时行为 -->
      <typeAliases>
    	  <!-- 配置别名 -->
    	  <package name="domain"/>
      </typeAliases>
      <mappers>
      	<mapper resource="mapper/baseMapper.xml"/>
      	<mapper resource="mapper/insertPeopleMapper.xml"/>
      	<mapper resource="mapper/selectPeopleMapper.xml"/>
      	<mapper resource="mapper/insertServerMapper.xml"/>
      	<mapper resource="mapper/selectServerMapper.xml"/> 
      	<mapper resource="mapper/insertPeopleServerMapper.xml"/>
      	<mapper resource="mapper/selectPeopleServerMapper.xml"/>
      	<mapper resource="mapper/selectPeopleByIdMapper.xml"/>
      	<mapper resource="mapper/selectServerByIdMapper.xml"/>
      	<mapper resource="mapper/selectServerByPeopleIdMapper.xml"/>
      	<mapper resource="mapper/selectPeopleByServerIdMapper.xml"/>
      	<mapper resource="mapper/moreToMoreForPeopleMapper.xml"/>
      	<mapper resource="mapper/moreToMoreForServerMapper.xml"/>
      	<mapper resource="mapper/selectSeqMapper.xml"/>
      </mappers>
    </configuration>
    

transaction.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:aop="http://www.springframework.org/schema/aop"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xmlns:tx="http://www.springframework.org/schema/tx"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/aop
    	http://www.springframework.org/schema/aop/spring-aop.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd
    	http://www.springframework.org/schema/tx
    	http://www.springframework.org/schema/tx/spring-tx.xsd
    	">
    	<!-- 注册事物管理器，依赖于数据源的一个bean -->
    	<bean id="transactionManager"
    		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    		<property name="dataSource" ref="dataSource"></property>
    	</bean>
    	<!-- 开启注解扫描 -->
    	<tx:annotation-driven transaction-manager="transactionManager" />
    	<!-- 编写通知 -->
    	<tx:advice id="txAdvice" transaction-manager="transactionManager">
    		<tx:attributes >
    			<tx:method name="*" propagation="REQUIRED" isolation="DEFAULT" read-only="false"/>			
    		</tx:attributes>
    	</tx:advice>
    </beans>
    

## 10.增加服务

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ea26b1170223f246136e838885b28d55.png)
    
    
    package service;
    
    import java.util.List;
    
    import domain.People;
    import domain.Server;
    
    public interface PeopleServerService {
    
    	void insert(People people,Server server);
    	
    	List<People> selectPeople(People people);
    	
    	List<Server> selectServer(Server server);
    	
    }
    
    

## 11.增加Dao

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/eec3d208074446cdc6886c9f44975926.png)
    
    
    package dao;
    
    import java.util.List;
    
    import domain.People;
    
    
    public interface PeopleDao {
    
    	int insertPeople(People people);
    	
    	List<People> selectPeople(People people);
    	
    	People selectPeopleById(Long peopleId);
    	
    	List<People> selectPeopleByServerId(Long serverId);
    	
    	List<People> selectPeopleWithServer(People people);
    	
    }
    
    
    
    
    package dao;
    
    import java.util.List;
    
    import domain.PeopleServer;
    
    public interface PeopleServerDao {
    
    	int insertPeopleServer(PeopleServer peopleServer);
    	
    	List<PeopleServer> selelctPeopleServer(PeopleServer peopleServer);
    	
    }
    
    
    
    
    package dao;
    
    public interface SeqForPeopleServerDao {
    
    	Long getPeopleSeq();
    	
    	Long getServerSeq();
    	
    }
    
    
    
    
    package dao;
    
    import java.util.List;
    
    import domain.Server;
    
    public interface ServerDao {
    
    	int insertServer(Server server);
    	
    	List<Server> selectServer(Server server);
    	
    	Server selectServerById(Long serverId);
    	
    	List<Server> selectServerByPeopleId(Long peopleId);
    	
    	List<Server> selectServerWithPeople(Server server);
    	
    }
    
    

## 12.服务的实现

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/07dc733dc02b7f3086d0921018d5e954.png)
    
    
    package serviceImpl;
    
    import java.util.List;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.springframework.util.Assert;
    
    import service.PeopleServerService;
    import util.MyApplicationFactory;
    import dao.PeopleDao;
    import dao.PeopleServerDao;
    import dao.SeqForPeopleServerDao;
    import dao.ServerDao;
    import domain.People;
    import domain.PeopleServer;
    import domain.Server;
    
    @Service("peopleServerServer")
    public class PeopleServerServiceImpl implements PeopleServerService{
    
    	@Autowired
    	private PeopleDao peopleDao;
    	
    	@Autowired
    	private ServerDao serverDao;
    	
    	@Autowired
    	private PeopleServerDao peopleServerDao;
    	
    	@Autowired
    	private SeqForPeopleServerDao seqForPeopleServerDao;
    	
    	@Transactional
    	@Override
    	public void insert(People people, Server server) {
    		Assert.notNull(people, "people.is.null");
    		Assert.notNull(server, "server.is.null");
    		if(people.getId() == null){
    			people.setId(seqForPeopleServerDao.getPeopleSeq());
    			if(peopleDao.insertPeople(people) != 1){
    				//抛出异常希望事务回滚
    				throw new RuntimeException("insert.people.exception");
    			}
    		}
    		if(server.getId() == null){
    			server.setId(seqForPeopleServerDao.getServerSeq());
    			if(serverDao.insertServer(server) != 1){
    				throw new RuntimeException("insert.server.exception");
    			}
    		}
    		PeopleServer peopleServer = MyApplicationFactory
    				.getBean(PeopleServer.class);
    		peopleServer.setPeople_fk(people.getId());
    		peopleServer.setServer_fk(server.getId());
    		if(peopleServerDao.insertPeopleServer(peopleServer) != 1){
    			throw new RuntimeException("insert.people.server.exception");
    		}
    	}
    
    	@Override
    	public List<People> selectPeople(People people) {
    		return peopleDao.selectPeopleWithServer(people);
    	}
    
    	@Override
    	public List<Server> selectServer(Server server) {
    		return serverDao.selectServerWithPeople(server);
    	}
    
    	public PeopleDao getPeopleDao() {
    		return peopleDao;
    	}
    
    	public void setPeopleDao(PeopleDao peopleDao) {
    		this.peopleDao = peopleDao;
    	}
    
    	public ServerDao getServerDao() {
    		return serverDao;
    	}
    
    	public void setServerDao(ServerDao serverDao) {
    		this.serverDao = serverDao;
    	}
    
    	public PeopleServerDao getPeopleServerDao() {
    		return peopleServerDao;
    	}
    
    	public void setPeopleServerDao(PeopleServerDao peopleServerDao) {
    		this.peopleServerDao = peopleServerDao;
    	}
    
    	public SeqForPeopleServerDao getSeqForPeopleServerDao() {
    		return seqForPeopleServerDao;
    	}
    
    	public void setSeqForPeopleServerDao(SeqForPeopleServerDao seqForPeopleServerDao) {
    		this.seqForPeopleServerDao = seqForPeopleServerDao;
    	}
    
    }
    
    

## 13.Dao的实现

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4f0acf5dc92ae2c48f5c600ad32ddadb.png)
    
    
    package daoImpl;
    
    import java.util.List;
    
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Repository;
    
    import dao.PeopleDao;
    import domain.People;
    
    @Repository("peopleDao")
    public class PeopleDaoImpl implements PeopleDao{
    
    	@Autowired
    	private SqlSessionFactory sqlSessionFactory;
    	
    	@Override
    	public int insertPeople(People people) {
    		return this.sqlSessionFactory.openSession().insert(
    				"insertPeopleMapper.insertPeople", people);
    	}
    
    	@Override
    	public List<People> selectPeople(People people) {
    		return this.sqlSessionFactory.openSession().selectList(
    				"selectPeopleMapper.selectPeople", people);
    	}
    
    	@Override
    	public People selectPeopleById(Long peopleId) {
    		return this.sqlSessionFactory.openSession().selectOne(
    				"peopleMapper.selectById", peopleId);
    	}
    
    	@Override
    	public List<People> selectPeopleByServerId(Long serverId) {
    		return this.sqlSessionFactory.openSession().selectList(
    				"selectPeopleByServerIdMapper.selectById", serverId);
    	}
    
    	@Override
    	public List<People> selectPeopleWithServer(People people) {
    		return this.sqlSessionFactory.openSession().selectList(
    				"moreToMoreForPeopleMapper.selectPeopleWithServer", people);
    	}
    
    	public SqlSessionFactory getSqlSessionFactory() {
    		return sqlSessionFactory;
    	}
    
    	public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
    		this.sqlSessionFactory = sqlSessionFactory;
    	}
    
    }
    
    
    
    
    package daoImpl;
    
    import java.util.List;
    
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Repository;
    
    import dao.PeopleServerDao;
    import domain.PeopleServer;
    
    @Repository("peopleServerDao")
    public class PeopleServerDaoImpl implements PeopleServerDao{
    
    	@Autowired
    	private SqlSessionFactory sqlSessionFactory;
    	
    	@Override
    	public int insertPeopleServer(PeopleServer peopleServer) {
    		return this.sqlSessionFactory.openSession().insert(
    				"insertPeopleServerMapper.insertPeopleServerOne",
    				peopleServer);
    	}
    
    	@Override
    	public List<PeopleServer> selelctPeopleServer(PeopleServer peopleServer) {
    		return this.sqlSessionFactory.openSession().selectList(
    				"selectPeopleServerMapper.selectPeopleServer", peopleServer);
    	}
    
    	public SqlSessionFactory getSqlSessionFactory() {
    		return sqlSessionFactory;
    	}
    
    	public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
    		this.sqlSessionFactory = sqlSessionFactory;
    	}
    
    }
    
    
    
    
    package daoImpl;
    
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Repository;
    
    import dao.SeqForPeopleServerDao;
    
    @Repository("peopleServerSeq")
    public class SeqForPeopleServerDaoImpl implements SeqForPeopleServerDao{
    
    	@Autowired
    	private SqlSessionFactory sqlSessionFactory;
    	
    	@Override
    	public Long getPeopleSeq() {
    		return this.sqlSessionFactory.openSession().selectOne(
    				"selectSeqMapper.selectPeopleSeq");
    	}
    
    	@Override
    	public Long getServerSeq() {
    		return this.sqlSessionFactory.openSession().selectOne(
    				"selectSeqMapper.selectServerSeq");
    	}
    
    	public SqlSessionFactory getSqlSessionFactory() {
    		return sqlSessionFactory;
    	}
    
    	public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
    		this.sqlSessionFactory = sqlSessionFactory;
    	}
    
    }
    
    
    
    
    package daoImpl;
    
    import java.util.List;
    
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Repository;
    
    import dao.ServerDao;
    import domain.Server;
    
    @Repository("serverDao")
    public class ServerDaoImpl implements ServerDao{
    
    	@Autowired
    	private SqlSessionFactory sqlSessionFactory;
    	
    	@Override
    	public int insertServer(Server server) {
    		return this.sqlSessionFactory.openSession().insert(
    				"insertServerMapper.insertServer", server);
    	}
    
    	@Override
    	public List<Server> selectServer(Server server) {
    		return this.sqlSessionFactory.openSession().selectList(
    				"selectServerMapper.selectServer", server);
    	}
    
    	@Override
    	public Server selectServerById(Long serverId) {
    		return this.sqlSessionFactory.openSession().selectOne(
    				"serverMapper.selectById", serverId);
    	}
    
    	@Override
    	public List<Server> selectServerByPeopleId(Long peopleId) {
    		return this.sqlSessionFactory.openSession().selectList(
    				"selectServerByPeopleIdMapper.selectById", peopleId);
    	}
    
    	@Override
    	public List<Server> selectServerWithPeople(Server server) {
    		return this.sqlSessionFactory.openSession().selectList(
    				"moreToMoreForServerMapper.selectServerWithPeople", server);
    	}
    
    	public SqlSessionFactory getSqlSessionFactory() {
    		return sqlSessionFactory;
    	}
    
    	public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
    		this.sqlSessionFactory = sqlSessionFactory;
    	}
    
    }
    
    

## 14.mapper编写

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/637cdfa41ab92c9cc0ee4df5e8d0b7e6.png)  
baseMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="baseMapper">
    	<sql id="str_select_people">
    		p.id,p.name,p.age,p.sex
    	</sql>
    	<sql id="str_table_people">
    		people p
    	</sql>
    	<sql id="str_select_server">
    		s.id,s.name,s.remark
    	</sql>
    	<sql id="str_table_server">
    		server s
    	</sql>
    	<sql id="str_select_people_server">
    		ps.people_id as people_fk,ps.server_id as server_fk
    	</sql>
    	<sql id="str_table_people_server">
    		people_server ps
    	</sql>
    </mapper>
    

insertPeopleMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="insertPeopleMapper">
    	<insert id="insertPeople" parameterType="people">
    		insert into 
    		<include refid="baseMapper.str_table_people"></include>
    		(<include refid="baseMapper.str_select_people"></include>) 
    		values(#{id},#{name},#{age},#{sex})
    	</insert>
    </mapper>
    

insertPeopleServerMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="insertPeopleServerMapper">
    	<insert id="insertPeopleServer" parameterType="peopleServer">
    		insert into 
    		<include refid="baseMapper.str_table_people_server"></include>
    		(<include refid="baseMapper.str_select_people_server"></include>) 
    		values(#{people_fk},#{server_fk})
    	</insert>
    	<insert id="insertPeopleServerOne" parameterType="peopleServer">
    		insert into 
    		<include refid="baseMapper.str_table_people_server"></include>
    		(PS.PEOPLE_ID, PS.SERVER_ID) 
    		values(#{people_fk},#{server_fk})
    	</insert>
    </mapper>
    

insertServerMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="insertServerMapper">
    	<insert id="insertServer" parameterType="server">
    		insert into 
    		<include refid="baseMapper.str_table_server"></include>
    		(<include refid="baseMapper.str_select_server"></include>) 
    		values(#{id},#{name},#{remark})
    	</insert>
    </mapper>
    

moreToMoreForPeopleMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="moreToMoreForPeopleMapper">
    	<resultMap type="people" id="resultWithServer">
    		<id property="id" column="id"/>
    		<result property="name" column="name"/>
    		<result property="age" column="age"/>
    		<result property="sex" column="sex"/>
    		<collection property="servers" ofType="server" column="id"
    	select="selectServerByPeopleIdMapper.selectById"></collection>
    	</resultMap>
    	<select id="selectPeopleWithServer" parameterType="people" resultMap="resultWithServer">
    		select <include refid="baseMapper.str_select_people"></include>
    		from <include refid="baseMapper.str_table_people"></include>
    		<where>
    			<if test="id != null and id != ''">
    		  	 	and p.id=#{id}
    		  	 </if>
    		  	 <if test="name != null and name != ''">
    		  	 	and p.name=#{name}
    		  	 </if>
    		  	 <if test="age != null and age != ''">
    		  	 	and p.age=#{age}
    		  	 </if>
    		  	 <if test="sex != null and sex != ''">
    		  	 	and p.sex=#{sex}
    		  	 </if>
    		</where>
    	</select>
    	
    	<!-- 结果方式 -->
    	<resultMap type="people" id="resultPeopleWithServer">
    		<id property="id" column="pid"/>
    		<result property="name" column="pname"/>
    		<result property="age" column="page"/>
    		<result property="sex" column="psex"/>
    		<collection property="servers" ofType="server">
    			<id property="id" column="sid"/>
    			<result property="name" column="sname"/>
    			<result property="remark" column="sremark"/>
    		</collection>
    	</resultMap>
    	<select id="selectPeopleWithServerResult" parameterType="people" resultMap="resultPeopleWithServer">
    		select p.id as pid,p.name as pname,p.age as page,p.sex as psex,
    		s.id as sid,s.name as sname,s.remark as sremark
    		from <include refid="baseMapper.str_table_people"></include>,
    		<include refid="baseMapper.str_table_server"></include>,
    		<include refid="baseMapper.str_table_people_server"></include>
    		<where>
    			and p.id=ps.people_id
    			and s.id=ps.server_id
    			<if test="id != null and id != ''">
    		  	 	and p.id=#{id}
    		  	 </if>
    		  	 <if test="name != null and name != ''">
    		  	 	and p.name=#{name}
    		  	 </if>
    		  	 <if test="age != null and age != ''">
    		  	 	and p.age=#{age}
    		  	 </if>
    		  	 <if test="sex != null and sex != ''">
    		  	 	and p.sex=#{sex}
    		  	 </if>
    		</where>
    	</select>
    </mapper>
    

moreToMoreForServerMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="moreToMoreForServerMapper">
    	<resultMap type="server" id="resultWithPeople">
    		<id property="id" column="id"/>
    		<result property="name" column="name"/>
    		<result property="remark" column="remark"/>
    		<collection property="peoples" ofType="people" column="id"
    	select="selectPeopleByServerIdMapper.selectById"></collection>
    	</resultMap>
    	<select id="selectServerWithPeople" parameterType="server" resultMap="resultWithPeople">
    		select <include refid="baseMapper.str_select_server"></include>
    		from <include refid="baseMapper.str_table_server"></include>
    		<where>
    			<if test="id != null and id != ''">
    		  	 	and s.id=#{id}
    		  	 </if>
    		  	 <if test="name != null and name != ''">
    		  	 	and s.name=#{name}
    		  	 </if>
    		  	 <if test="remark != null and remark != ''">
    		  	 	and s.remark=#{remark}
    		  	 </if>
    		</where>
    	</select>
    	
    	<!-- 结果方式 -->
    	<resultMap type="server" id="resultServerWithPeople">
    		<id property="id" column="sid"/>
    		<result property="name" column="sname"/>
    		<result property="remark" column="sremark"/>
    		<collection property="peoples" ofType="people">
    			<id property="id" column="pid"/>
    			<result property="name" column="pname"/>
    			<result property="age" column="page"/>
    			<result property="sex" column="psex"/>
    		</collection>
    	</resultMap>
    	<select id="selectServerWithPeopleResult" parameterType="server" resultMap="resultServerWithPeople">
    		select p.id as pid,p.name as pname,p.age as page,p.sex as psex,
    		s.id as sid,s.name as sname,s.remark as sremark
    		from <include refid="baseMapper.str_table_people"></include>,
    		<include refid="baseMapper.str_table_server"></include>,
    		<include refid="baseMapper.str_table_people_server"></include>
    		<where>
    			and p.id=ps.people_id
    			and s.id=ps.server_id
    			<if test="id != null and id != ''">
    		  	 	and s.id=#{id}
    		  	 </if>
    		  	 <if test="name != null and name != ''">
    		  	 	and s.name=#{name}
    		  	 </if>
    		  	 <if test="remark != null and remark != ''">
    		  	 	and s.remark=#{remark}
    		  	 </if>
    		</where>
    	</select>
    </mapper>
    

selectPeopleByIdMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="peopleMapper">
    	<select id="selectById" parameterType="Long" resultType="people">
    		select <include refid="baseMapper.str_select_people"></include>
    		from <include refid="baseMapper.str_table_people"></include>
    		<where>
    			<if test="_parameter != null and _parameter != ''">
    				and id=#{_parameter}
    			</if>
    		</where>
    	</select>
    </mapper>
    

selectPeopleByServerIdMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectPeopleByServerIdMapper">
    	<select id="selectById" parameterType="Long" resultType="people">
    		select <include refid="baseMapper.str_select_people"></include>
    		from <include refid="baseMapper.str_table_people"></include>
    		,<include refid="baseMapper.str_table_people_server"></include>
    		<where>
    			p.id = ps.people_id
    			<if test="_parameter != null and _parameter != ''">
    				and ps.server_id=#{_parameter}
    			</if>
    		</where>
    	</select>
    </mapper>
    

selectPeopleMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectPeopleMapper">
    	<select id="selectPeople" parameterType="people" resultType="people">
    		select 
    		<include refid="baseMapper.str_select_people"></include>
    		from 
    		<include refid="baseMapper.str_table_people"></include>
    		<where>
    			<if test="id != null and id != ''">
    		  	 	and id=#{id}
    		  	 </if>
    		  	 <if test="name != null and name != ''">
    		  	 	and name=#{name}
    		  	 </if>
    		  	 <if test="age != null and age != ''">
    		  	 	and age=#{age}
    		  	 </if>
    		  	 <if test="sex != null and sex != ''">
    		  	 	and sex=#{sex}
    		  	 </if>
    		</where>
    	</select>
    </mapper>
    

selectPeopleServerMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectPeopleServerMapper">
    	<select id="selectPeopleServer" parameterType="peopleServer" resultType="peopleServer">
    		select 
    		<include refid="baseMapper.str_select_people_server"></include>
    		from 
    		<include refid="baseMapper.str_table_people_server"></include>
    		<where>
    			<if test="people_fk != null and people_fk != ''">
    		  	 	and people_id=#{people_fk}
    		  	 </if>
    		  	 <if test="server_fk != null and server_fk != ''">
    		  	 	and server_id=#{server_fk}
    		  	 </if>
    		</where>
    	</select>
    </mapper>
    

selectSeqMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectSeqMapper">
    	<select id="selectPeopleSeq" resultType="Long">
    		SELECT SEQ_PEOPLE.NEXTVAL FROM DUAL
    	</select>
    	<select id="selectServerSeq" resultType="Long">
    		SELECT SEQ_SERVER.NEXTVAL FROM DUAL
    	</select>
    </mapper>
    

selectServerByIdMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="serverMapper">
    	<select id="selectById" parameterType="Long" resultType="server">
    		select <include refid="baseMapper.str_select_server"></include>
    		from <include refid="baseMapper.str_table_server"></include>
    		<where>
    			<if test="_parameter != null and _parameter != ''">
    				and id=#{_parameter}
    			</if>
    		</where>
    	</select>
    </mapper>
    

selectServerByPeopleIdMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectServerByPeopleIdMapper">
    	<select id="selectById" parameterType="Long" resultType="server">
    		select <include refid="baseMapper.str_select_server"></include>
    		from <include refid="baseMapper.str_table_server"></include>
    		,<include refid="baseMapper.str_table_people_server"></include>
    		<where>
    			s.id = ps.server_id
    			<if test="_parameter != null and _parameter != ''">
    				and ps.people_id=#{_parameter}
    			</if>
    		</where>
    	</select>
    </mapper>
    

selectServerMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectServerMapper">
    	<select id="selectServer" parameterType="server" resultType="server">
    		select 
    		<include refid="baseMapper.str_select_server"></include>
    		from 
    		<include refid="baseMapper.str_table_server"></include>
    		<where>
    			<if test="id != null and id != ''">
    		  	 	and id=#{id}
    		  	 </if>
    		  	 <if test="name != null and name != ''">
    		  	 	and name=#{name}
    		  	 </if>
    		  	 <if test="remark != null and remark != ''">
    		  	 	and remark=#{remark}
    		  	 </if>
    		</where>
    	</select>
    </mapper>
    

## 15.编写测试类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/screenshot/image-20260104155254.png)
    
    
    package test;
    
    import org.apache.commons.dbcp2.BasicDataSource;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.junit.Test;
    
    import service.PeopleServerService;
    import util.MyApplicationFactory;
    import domain.People;
    import domain.PeopleServer;
    import domain.Server;
    
    public class Main {
    
    	@Test
    	public void testInsert(){
    		//增加一个20190309的people和20190309的server以及之间的关系
    		People people = MyApplicationFactory.getBean(People.class);
    		people.setName("20190309");
    		people.setAge(22);
    		people.setSex(1);
    		Server server = MyApplicationFactory.getBean(Server.class);
    		server.setName("20190309");
    		server.setRemark("testInsertSpringMybatis");
    		PeopleServerService peopleServerService = MyApplicationFactory
    				.getBean(PeopleServerService.class);
    		peopleServerService.insert(people, server);
    	}
    	
    	@Test
    	public void selectPeople(){
    		People people = MyApplicationFactory.getBean(People.class);
    		people.setName("20190309");
    		PeopleServerService peopleServerService = MyApplicationFactory
    				.getBean(PeopleServerService.class);
    		peopleServerService.selectPeople(people).forEach(
    				p -> System.out.println(p));
    	}
    	
    	@Test
    	public void selectServer(){
    		Server server = MyApplicationFactory.getBean(Server.class);
    		server.setName("20190309");
    		PeopleServerService peopleServerService = MyApplicationFactory
    				.getBean(PeopleServerService.class);
    		peopleServerService.selectServer(server).forEach(
    				p -> System.out.println(p));
    	}
    	
    	@Test
    	public void testService(){
    		MyApplicationFactory.getBean(PeopleServerService.class);
    	}
    	
    	@Test
        public void setDateSource() {
    		BasicDataSource dataSource = (BasicDataSource) MyApplicationFactory.getApplicationContext().getBean("dataSource");
    		System.out.println(dataSource.getUsername());
        }
    	
    	@Test
    	public void insertPeopleServer(){
    		SqlSession sqlSession = MyApplicationFactory.getBean(SqlSessionFactory.class).openSession();
    		PeopleServer peopleServer = MyApplicationFactory.getBean(PeopleServer.class);
    		peopleServer.setPeople_fk(666L);
    		peopleServer.setServer_fk(199L);
    		System.out.println(sqlSession.insert("insertPeopleServerMapper.insertPeopleServerOne",peopleServer));
    		sqlSession.commit();
    		sqlSession.close();
    		
    		
    	}
    }
    
    

## 16.运行结果

testInsert
    
    
    Service Info    2019-56-11 20:56:19,19  [insert]    before
    Transaction Info    2019-56-11 20:56:19,19  [insert]    before
    Dao     Info    2019-56-11 20:56:19,19  [getPeopleSeq]    before
    Dao     Info    2019-56-11 20:56:19,19  [getPeopleSeq]    afterReturn
    Dao     Info    2019-56-11 20:56:19,19  [insertPeople]    before
    Dao     Info    2019-56-11 20:56:19,19  [insertPeople]    afterReturn
    Dao     Info    2019-56-11 20:56:19,19  [getServerSeq]    before
    Dao     Info    2019-56-11 20:56:19,19  [getServerSeq]    afterReturn
    Dao     Info    2019-56-11 20:56:19,19  [insertServer]    before
    Dao     Info    2019-56-11 20:56:19,19  [insertServer]    afterReturn
    Dao     Info    2019-56-11 20:56:19,19  [insertPeopleServer]    before
    Dao     Info    2019-56-11 20:56:19,19  [insertPeopleServer]    afterReturn
    Service Info    2019-56-11 20:56:19,19  [insert]    afterReturn
    Transaction Info    2019-56-11 20:56:19,19  [insert]    afterReturn
    
    

selectPeople
    
    
    Service Info    2019-56-11 20:56:36,36  [selectPeople]    before
    Transaction Info    2019-56-11 20:56:36,36  [selectPeople]    before
    Dao     Info    2019-56-11 20:56:36,36  [selectPeopleWithServer]    before
    Dao     Info    2019-56-11 20:56:37,37  [selectPeopleWithServer]    afterReturn
    Service Info    2019-56-11 20:56:37,37  [selectPeople]    afterReturn
    Transaction Info    2019-56-11 20:56:37,37  [selectPeople]    afterReturn
    people [id=674,name=20190309,age=22,sex=1,server=[[id=206,name=20190309,remark=testInsertSpringMybatis,peoples=null]]]
    people [id=677,name=20190309,age=22,sex=1,server=[[id=207,name=20190309,remark=testInsertSpringMybatis,peoples=null]]]
    people [id=682,name=20190309,age=22,sex=1,server=[[id=208,name=20190309,remark=testInsertSpringMybatis,peoples=null]]]
    
    

selectServer
    
    
    Service Info    2019-56-11 20:56:52,52  [selectServer]    before
    Transaction Info    2019-56-11 20:56:52,52  [selectServer]    before
    Dao     Info    2019-56-11 20:56:52,52  [selectServerWithPeople]    before
    Dao     Info    2019-56-11 20:56:53,53  [selectServerWithPeople]    afterReturn
    Service Info    2019-56-11 20:56:53,53  [selectServer]    afterReturn
    Transaction Info    2019-56-11 20:56:53,53  [selectServer]    afterReturn
    [id=206,name=20190309,remark=testInsertSpringMybatis,peoples=[people [id=674,name=20190309,age=22,sex=1,server=null]]]
    [id=207,name=20190309,remark=testInsertSpringMybatis,peoples=[people [id=677,name=20190309,age=22,sex=1,server=null]]]
    [id=208,name=20190309,remark=testInsertSpringMybatis,peoples=[people [id=682,name=20190309,age=22,sex=1,server=null]]]
    
    

insertPeopleServer
    
    
    1
