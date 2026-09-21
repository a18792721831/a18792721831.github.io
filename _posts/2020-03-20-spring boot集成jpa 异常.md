---
layout: post
title: "spring boot集成jpa 异常"
date: 2020-03-20 20:09:42 +0800
categories: [spring jpa, BeanCreation, 不支持的字符集, orai18n.jar, jpa异常]
description: "本文解决了一个关于在Spring Boot项目中使用Oracle数据库时遇到的字符集错误问题，具体错误为“不支持的字符集:ZHS16GBK”。通过在项目中添加orai18n.jar依赖，成功解决了这一问题。"
keywords: spring jpa, BeanCreation, 不支持的字符集, orai18n.jar, jpa异常
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104997564
> - 发布时间：2020-03-20 20:09:42
> - 阅读量：2
> - 分类：微服务同时被 3 个专栏收录, 订阅专栏, spring boot, JPA
> - 标签：#spring jpa, #BeanCreation, #不支持的字符集, #orai18n.jar, #jpa异常

## 摘要

文章浏览阅读2.5k次。本文解决了一个关于在Spring Boot项目中使用Oracle数据库时遇到的字符集错误问题，具体错误为“不支持的字符集:ZHS16GBK”。通过在项目中添加orai18n.jar依赖，成功解决了这一问题。

---

现象：  
java.sql.SQLException: 不支持的字符集 (在类路径中添加 orai18n.jar): ZHS16GBK  
解决方式：  
增加依赖  
runtimeOnly ‘cn.easyproject:orai18n:12.1.0.2.0’

完整的异常:
    
    
    org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaConfiguration.class]: Invocation of init method failed; nested exception is javax.persistence.PersistenceException: [PersistenceUnit: default] Unable to build Hibernate SessionFactory; nested exception is org.hibernate.exception.GenericJDBCException: Unable to build DatabaseInformation
    	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1796) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:595) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:517) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:323) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:321) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1108) ~[spring-context-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:868) ~[spring-context-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:550) ~[spring-context-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:141) ~[spring-boot-2.2.5.RELEASE.jar:2.2.5.RELEASE]
    	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:747) [spring-boot-2.2.5.RELEASE.jar:2.2.5.RELEASE]
    	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:397) [spring-boot-2.2.5.RELEASE.jar:2.2.5.RELEASE]
    	at org.springframework.boot.SpringApplication.run(SpringApplication.java:315) [spring-boot-2.2.5.RELEASE.jar:2.2.5.RELEASE]
    	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1226) [spring-boot-2.2.5.RELEASE.jar:2.2.5.RELEASE]
    	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1215) [spring-boot-2.2.5.RELEASE.jar:2.2.5.RELEASE]
    	at com.study.springbootsecurityjpa.SpringbootsecurityjpaApplication.main(SpringbootsecurityjpaApplication.java:10) [main/:na]
    Caused by: javax.persistence.PersistenceException: [PersistenceUnit: default] Unable to build Hibernate SessionFactory; nested exception is org.hibernate.exception.GenericJDBCException: Unable to build DatabaseInformation
    	at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.buildNativeEntityManagerFactory(AbstractEntityManagerFactoryBean.java:403) ~[spring-orm-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.afterPropertiesSet(AbstractEntityManagerFactoryBean.java:378) ~[spring-orm-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean.afterPropertiesSet(LocalContainerEntityManagerFactoryBean.java:341) ~[spring-orm-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1855) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1792) ~[spring-beans-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	... 16 common frames omitted
    Caused by: org.hibernate.exception.GenericJDBCException: Unable to build DatabaseInformation
    	at org.hibernate.exception.internal.StandardSQLExceptionConverter.convert(StandardSQLExceptionConverter.java:47) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.engine.jdbc.spi.SqlExceptionHelper.convert(SqlExceptionHelper.java:113) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.engine.jdbc.spi.SqlExceptionHelper.convert(SqlExceptionHelper.java:99) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.tool.schema.internal.Helper.buildDatabaseInformation(Helper.java:163) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.tool.schema.internal.AbstractSchemaMigrator.doMigration(AbstractSchemaMigrator.java:96) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.tool.schema.spi.SchemaManagementToolCoordinator.performDatabaseAction(SchemaManagementToolCoordinator.java:184) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.tool.schema.spi.SchemaManagementToolCoordinator.process(SchemaManagementToolCoordinator.java:73) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.internal.SessionFactoryImpl.<init>(SessionFactoryImpl.java:314) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.boot.internal.SessionFactoryBuilderImpl.build(SessionFactoryBuilderImpl.java:468) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.jpa.boot.internal.EntityManagerFactoryBuilderImpl.build(EntityManagerFactoryBuilderImpl.java:1237) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.springframework.orm.jpa.vendor.SpringHibernateJpaPersistenceProvider.createContainerEntityManagerFactory(SpringHibernateJpaPersistenceProvider.java:58) ~[spring-orm-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean.createNativeEntityManagerFactory(LocalContainerEntityManagerFactoryBean.java:365) ~[spring-orm-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.buildNativeEntityManagerFactory(AbstractEntityManagerFactoryBean.java:391) ~[spring-orm-5.2.4.RELEASE.jar:5.2.4.RELEASE]
    	... 20 common frames omitted
    Caused by: java.sql.SQLException: 不支持的字符集 (在类路径中添加 orai18n.jar): ZHS16GBK
    	at oracle.sql.CharacterSetUnknown.failCharsetUnknown(CharacterSetFactoryThin.java:233) ~[ojdbc8-19.3.0.0.jar:19.3.0.0.0]
    	at oracle.sql.CharacterSetUnknown.convert(CharacterSetFactoryThin.java:194) ~[ojdbc8-19.3.0.0.jar:19.3.0.0.0]
    	at oracle.jdbc.driver.PhysicalConnection.throughDbCharset(PhysicalConnection.java:10365) ~[ojdbc8-19.3.0.0.jar:19.3.0.0.0]
    	at oracle.jdbc.driver.PhysicalConnection.enquoteIdentifier(PhysicalConnection.java:10442) ~[ojdbc8-19.3.0.0.jar:19.3.0.0.0]
    	at oracle.jdbc.driver.OracleStatement.enquoteIdentifier(OracleStatement.java:6452) ~[ojdbc8-19.3.0.0.jar:19.3.0.0.0]
    	at oracle.jdbc.driver.OracleStatement.getColumnIndex(OracleStatement.java:3853) ~[ojdbc8-19.3.0.0.jar:19.3.0.0.0]
    	at oracle.jdbc.driver.InsensitiveScrollableResultSet.findColumn(InsensitiveScrollableResultSet.java:270) ~[ojdbc8-19.3.0.0.jar:19.3.0.0.0]
    	at oracle.jdbc.driver.GeneratedResultSet.getString(GeneratedResultSet.java:596) ~[ojdbc8-19.3.0.0.jar:19.3.0.0.0]
    	at com.zaxxer.hikari.pool.HikariProxyResultSet.getString(HikariProxyResultSet.java) ~[HikariCP-3.4.2.jar:na]
    	at org.hibernate.tool.schema.extract.internal.SequenceInformationExtractorLegacyImpl.resultSetSequenceName(SequenceInformationExtractorLegacyImpl.java:114) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.tool.schema.extract.internal.SequenceInformationExtractorLegacyImpl.extractMetadata(SequenceInformationExtractorLegacyImpl.java:56) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.tool.schema.extract.internal.DatabaseInformationImpl.initializeSequences(DatabaseInformationImpl.java:65) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.tool.schema.extract.internal.DatabaseInformationImpl.<init>(DatabaseInformationImpl.java:59) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	at org.hibernate.tool.schema.internal.Helper.buildDatabaseInformation(Helper.java:155) ~[hibernate-core-5.4.12.Final.jar:5.4.12.Final]
    	... 29 common frames omitted