---
layout: post
title: "spring事务xml配置方式"
date: 2019-02-25 22:19:23 +0800
categories: [spring事务, spring事务xml配置, 纯xml配置spring事务, spring综合项目, spring实现小项目]
description: "本文详细介绍了Spring事务的XML配置方式，包括理论知识、具体实例、运行结果及总结。通过创建Spring项目，实现数据增删改查操作，展示事务管理与AOP切面的结合应用。"
keywords: spring事务, spring事务xml配置, 纯xml配置spring事务, spring综合项目, spring实现小项目
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/87926263
> - 发布时间：2019-02-25 22:19:23
> - 阅读量：1
> - 分类：java同时被 3 个专栏收录, 订阅专栏, spring
> - 标签：#spring事务, #spring事务xml配置, #纯xml配置spring事务, #spring综合项目, #spring实现小项目

## 摘要

文章浏览阅读1.3k次。本文详细介绍了Spring事务的XML配置方式，包括理论知识、具体实例、运行结果及总结。通过创建Spring项目，实现数据增删改查操作，展示事务管理与AOP切面的结合应用。

---

#### spring事务xml配置方式

  * 1.理论知识
  * 2.实例
  *     * 2.1 创建一个spring项目
    * 2.2Java文件
    * 2.3xml文件
    * 2.4运行结果
  * 3.总结

## 1.理论知识

<https://blog.csdn.net/a18792721831/article/details/87924108>

## 2.实例

### 2.1 创建一个spring项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a5f402463f594655cc82fc343f9e556d.png)

### 2.2Java文件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0b6b07139b4deeb2ee76481d9b2c28e.png)
    
    
    package aspect;
    
    import java.text.SimpleDateFormat;
    import java.util.Date;
    
    import org.aspectj.lang.JoinPoint;
    
    public class MyAspect {
    
       
       public void beforeDao(JoinPoint joinPoint){
       	System.out.println("Dao     Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    before");
       }
       
       public void beforeService(JoinPoint joinPoint){
       	System.out.println("Service Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    before");
       }
       
       public void beforeTransaction(JoinPoint joinPoint){
       	System.out.println("Transaction Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    before");
       }
       
       public void afterReturnDao(JoinPoint joinPoint){
       	System.out.println("Dao     Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    afterReturn");
       }
       
       public void afterReturnService(JoinPoint joinPoint){
       	System.out.println("Service Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    afterReturn");
       }
       
       public void afterReturnTransaction(JoinPoint joinPoint){
       	System.out.println("Transaction Info  "+showTime() + "[" + joinPoint.getSignature().getName()+ "]    afterReturn");
       }
       
       private String showTime(){
       	return "  "+new SimpleDateFormat("yyyy-mm-dd HH:mm:ss,s").format(new Date())+"  ";
       }
    }
    
    
    
    
    package client;
    
    import java.util.ArrayList;
    import java.util.Iterator;
    import java.util.List;
    import java.util.Map;
    import java.util.Set;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import service.PeopleService;
    import domain.People;
    
    
    public class Main {
    
       public static void main(String[] args) {
       	@SuppressWarnings("resource")
       	ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
       			"resource/*.xml");
       	PeopleService peopleService = (PeopleService) applicationContext.getBean("peopleService");
       	
       	//add 2 people,people scope is prototype
       	People aPeople = (People) applicationContext.getBean("people");
       	People bPeople = (People) applicationContext.getBean("people");
       	//set people
       	aPeople.setName("aPeople");
       	aPeople.setAge(23);
       	aPeople.setSex(0);
       	bPeople.setName("bPeople");
       	bPeople.setAge(32);
       	bPeople.setSex(1);
       	
       	//add..
       	List<People> peoples = new ArrayList<>();
       	peoples.add(aPeople);
       	peoples.add(bPeople);
       	
       	//print peoples
       	for(People people : peoples){
       		System.out.println(people);
       	}
       	
       	peoples = peopleService.batchAddPeople(peoples);
       	
       	//print peoples(with id)
       	for(People people : peoples){
       		System.out.println(people);
       	}
       	
       	//update peoples   name : srcname+update
       	for(People people : peoples){
       		people.setName(people.getName()+"update");
       	}
       	
       	//print updated peoples
       	for(People people : peoples){
       		System.out.println(people);
       	}
       	
       	//update..
       	peopleService.batchUpdPeople(peoples);
       	
       	//query all add peoples
       	Map<People, List<People>> queryPeoMap = peopleService.batchQuyPeople(peoples);
       	
       	//print query peoples
       	Set<People> set = queryPeoMap.keySet();
       	Iterator<People> iterator = set.iterator();
       	while(iterator.hasNext()){
       		People queryOnePeople = iterator.next();
       		System.out.println("query:"+queryOnePeople);
       		for(People queryResult : queryPeoMap.get(queryOnePeople)){
       			System.out.println("result:"+queryResult);
       		}
       	}
       	
       	//delete id = 92(database has been)
       	People delPeople = (People) applicationContext.getBean("people");
       	delPeople.setId(Long.valueOf(92));
       	
       	//get peoples
       	List<People> delPeoples = new ArrayList<>();
       	delPeoples.add(delPeople);
       	
       	
       	//print delpeople
       	for(People people : delPeoples){
       		System.out.println(people);
       	}
       	
       	//delete..
       	delPeoples = peopleService.batchDelPeople(delPeoples);
       	
       	//print deleted peoples should equals srcdelpeoples
       	for(People people : delPeoples){
       		System.out.println(people);
       	}
       	
       	//test Transaction all peoples age = srcage + 5 , sex = !sex
       	peopleService.testTransaction();
       	
       }
    
    }
    
    
    
    
    package dao;
    
    import java.util.List;
    
    import domain.People;
    
    public interface PeopleDao {
    
       People addPeople(People people);
       
       People delPeople(People people);
       
       List<People> queryPeople(People people);
       
       People updatePeople(People people);
    
       void testTransaction();
    }
    
    
    
    
    package daoimpl;
    
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import java.sql.Types;
    import java.util.List;
    
    import org.springframework.jdbc.core.JdbcTemplate;
    import org.springframework.jdbc.core.RowMapper;
    
    import dao.PeopleDao;
    import domain.People;
    
    public class PeopleDaoImpl implements PeopleDao{
    
       private JdbcTemplate jdbcTemplate;
       
       private final String SELECT = "P.ID, P.NAME, P.AGE, P.SEX ";
       
       private final String TABLE = "PEOPLE P ";
       
       private final String SEQ_SQL = "SELECT SEQ_PEOPLE.NEXTVAL FROM DUAL ";
       
       private Long getSeq(){
       	return (Long)this.jdbcTemplate.queryForObject(SEQ_SQL, new RowMapper<Long>() {
       		@Override
       		public Long mapRow(ResultSet rs, int arg1) throws SQLException {
       			return Long.valueOf(rs.getLong("NEXTVAL"));
       		}
       	});
       }
       
       @Override
       public People addPeople(People people) {
       	String insert_sql = "INSERT INTO " + TABLE + " ( " + SELECT
       			+ ") VALUES(?,?,?,?)";
       	people.setId(getSeq());
       	this.jdbcTemplate.update(insert_sql, new Object[] { people.getId(),
       			people.getName(), people.getAge(), people.getSex() },
       			new int[] { Types.NUMERIC, Types.VARCHAR, Types.NUMERIC,
       					Types.NUMERIC });
       	return people;
       }
    
       @Override
       public People delPeople(People people) {
       	String delete_sql = "DELETE FROM " + TABLE + " WHERE 1 = 1 "
       			+ getCondition(people);
       	this.jdbcTemplate.update(delete_sql);
       	return people;
       }
       
       private String getCondition(People people){
       	String condition = "";
       	condition += people.getId() == null ? "" : " AND P.ID = " + people.getId();
       	condition += people.getName() == null ? "" : " AND P.NAME = '" + people.getName()+"'";
       	condition += people.getAge() == null ? "" : " AND P.AGE =   " + people.getAge();
       	condition += people.getSex() == null ? "" : " AND P.SEX =   " + people.getSex();
       	return condition;
       }
       
       @Override
       public List<People> queryPeople(People people) {
       	String select_sql = "SELECT " + SELECT + " FROM " + TABLE
       			+ " WHERE 1 = 1 " + getCondition(people);
       	return this.jdbcTemplate.query(select_sql,new PeopleRowMap());
       }
    
       @Override
       public People updatePeople(People people) {
       	String update_sql = "UPDATE PEOPLE P SET ";
       	update_sql += people.getName() == null ? "" : (" P.NAME = '"
       			+ people.getName() + "',");
       	update_sql += people.getAge() == null ? "" : (" P.AGE = "
       			+ people.getAge() + ",");
       	update_sql += people.getSex() == null ? "" : (" P.SEX = "
       			+ people.getSex());
       	update_sql += " WHERE P.ID = " + people.getId();
       	this.jdbcTemplate.update(update_sql);
       	return people;
       }
    
       public JdbcTemplate getJdbcTemplate() {
       	return jdbcTemplate;
       }
    
       public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
       	this.jdbcTemplate = jdbcTemplate;
       }
       
       @Override
       public void testTransaction(){
       	this.jdbcTemplate.update("UPDATE PEOPLE P SET P.AGE = P.AGE + 5");
       	@SuppressWarnings("unused")
       	int m = 1/0;
       	this.jdbcTemplate.update("UPDATE PEOPLE P SET P.SEX = 1 - P.SEX");
       }
       
    }
    class PeopleRowMap implements RowMapper<People>{
    
       @Override
       public People mapRow(ResultSet arg0, int arg1) throws SQLException {
       	People people = new People();
       	people.setId(arg0.getLong("ID"));
       	if(arg0.getString("NAME") != null){
       		people.setName(arg0.getString("NAME"));
       	}
       	people.setAge(arg0.getInt("AGE"));
       	people.setSex(arg0.getInt("SEX"));
       	return people;
       }
       
    } 
    
    
    
    
    package domain;
    
    import java.io.Serializable;
    
    public class People implements Serializable{
    
       /**
        * 
        */
       private static final long serialVersionUID = -3270893239281340723L;
    
       private String name;
       
       private Long id;
       
       private Integer age;
       
       private Integer sex;
    
       public void Clear(){
       	this.name = null;
       	this.id = null;
       	this.age = null;
       	this.sex = null;
       }
       
       @Override
       public String toString() {
       	return this.getClass().getSimpleName() + "name:"
       	+ this.name + "age:" + this.age + "sex:" + this.sex;
       }
    
       public String getName() {
       	return name;
       }
    
       public void setName(String name) {
       	this.name = name;
       }
    
       public Long getId() {
       	return id;
       }
    
       public void setId(Long id) {
       	this.id = id;
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
       
    }
    
    
    
    
    package service;
    
    import java.util.List;
    import java.util.Map;
    
    import domain.People;
    
    public interface PeopleService {
    
       List<People> batchAddPeople(List<People> peoples);
       
       List<People> batchDelPeople(List<People> peoples);
       
       List<People> batchUpdPeople(List<People> peoples);
       
       Map<People, List<People>> batchQuyPeople(List<People> peoples);
       
       void testTransaction();
    }
    
    
    
    
    package serviceimpl;
    
    import java.util.HashMap;
    import java.util.List;
    import java.util.Map;
    
    import service.PeopleService;
    import dao.PeopleDao;
    import domain.People;
    
    public class PeopleServiceImpl implements PeopleService{
    
       private PeopleDao peopleDao;
       
       public PeopleDao getPeopleDao() {
       	return peopleDao;
       }
    
       public void setPeopleDao(PeopleDao peopleDao) {
       	this.peopleDao = peopleDao;
       }
    
       @Override
       public List<People> batchAddPeople(List<People> peoples) {
       	for(People people : peoples){
       		people = peopleDao.addPeople(people);
       	}
       	return peoples;
       }
    
       @Override
       public List<People> batchDelPeople(List<People> peoples) {
       	for(People people : peoples){
       		people = peopleDao.delPeople(people);
       	}
       	return peoples;
       }
    
       @Override
       public List<People> batchUpdPeople(List<People> peoples) {
       	for(People people : peoples){
       		people = peopleDao.updatePeople(people);
       	}
       	return peoples;
       }
    
       @Override
       public Map<People, List<People>> batchQuyPeople(
       		List<People> peoples) {
       	Map<People, List<People>> result = new HashMap<>(peoples.size());
       	for(People people : peoples){
       		List<People> oneresult = peopleDao.queryPeople(people);
       		result.put(people, oneresult);
       	}
       	return result;
       }
       
       @Override
       public void testTransaction(){
       	peopleDao.testTransaction();
       }
       
    }
    
    

### 2.3xml文件

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
       <!-- 增强类 -->
       <bean id="aspect" class="aspect.MyAspect">
       </bean>
       <!-- aop编程 -->
       <aop:config>
       	<!-- 配置切面 -->
       	<aop:aspect ref="aspect">
       		<!-- 配置切入点 -->
       		<aop:pointcut expression="execution(* dao.*+.*(..))" id="pointCutDao"/>
       		<aop:pointcut expression="execution(* service.*+.*(..))" id="pointCutService"/>
       		<aop:pointcut expression="execution(* service.*+.testTransaction(..))" id="pointCutTransaction"/>
       		<!-- 关联切入点和增强类 -->
       		<!-- 前置通知 -->
       		<aop:before method="beforeDao" pointcut-ref="pointCutDao"/>
       		<aop:before method="beforeService" pointcut-ref="pointCutService"/>
       		<aop:before method="beforeTransaction" pointcut-ref="pointCutTransaction"/>
       		<!-- 后置通知 -->
       		<aop:after-returning method="afterReturnDao" pointcut-ref="pointCutDao"/>
       		<aop:after-returning method="afterReturnService" pointcut-ref="pointCutService"/>
       		<aop:after-returning method="afterReturnTransaction" pointcut-ref="pointCutTransaction"/>
       	</aop:aspect>
       </aop:config>
    </beans>
    

bean.xml
    
    
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
       <bean id="people" class="domain.People" scope="prototype"></bean>
       <bean id="peopleDao" class="daoimpl.PeopleDaoImpl">
       	<property name="jdbcTemplate" ref="jdbcTemplate"></property>
       </bean>
       <bean id="peopleService" class="serviceimpl.PeopleServiceImpl">
       	<property name="peopleDao" ref="peopleDao"></property>
       </bean>
    </beans>
    

dataBase.xml
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd">
       <!-- 配置数据源 -->
       <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
       	<!-- 数据库驱动 -->
       	<property name="driverClassName">
       		<value>oracle.jdbc.driver.OracleDriver</value>
       	</property>
       	<!-- 连接数据库的url -->
       	<property name="url">
       		<value>jdbc:oracle:thin:@127.0.0.1:1521:oracle</value>
       	</property>
       	<property name="username">
       		<value>study</value>
       	</property>
       	<property name="password">
       		<value>study</value>
       	</property>
       </bean>
       <!-- 配置JDBC的模板 -->
       <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
       	<!-- 默认必须使用数据源（也就是还可以不使用） -->
       	<property name="dataSource" ref="dataSource">
       	</property>
       </bean>
       <!-- 配置注入类 -->
       <!--注入就是使用模板的方法（按照普通属性看待） 
       <bean id="xxx" class="xxx">
       </bean>
        -->
    </beans>
    

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
       <!-- 编写通知 -->
       <tx:advice id="txAdvice" transaction-manager="transactionManager">
       	<tx:attributes >
       		<tx:method name="*" propagation="REQUIRED" isolation="DEFAULT" read-only="false"/>			
       	</tx:attributes>
       </tx:advice>
    </beans>
    

### 2.4运行结果
    
    
    Peoplename:aPeopleage:23sex:0
    Peoplename:bPeopleage:32sex:1
    Service Info    2019-53-25 20:53:19,19  [batchAddPeople]    before
    Dao     Info    2019-53-25 20:53:19,19  [addPeople]    before
    Dao     Info    2019-53-25 20:53:19,19  [addPeople]    afterReturn
    Dao     Info    2019-53-25 20:53:19,19  [addPeople]    before
    Dao     Info    2019-53-25 20:53:19,19  [addPeople]    afterReturn
    Service Info    2019-53-25 20:53:19,19  [batchAddPeople]    afterReturn
    Peoplename:aPeopleage:23sex:0
    Peoplename:bPeopleage:32sex:1
    Peoplename:aPeopleupdateage:23sex:0
    Peoplename:bPeopleupdateage:32sex:1
    Service Info    2019-53-25 20:53:19,19  [batchUpdPeople]    before
    Dao     Info    2019-53-25 20:53:19,19  [updatePeople]    before
    Dao     Info    2019-53-25 20:53:19,19  [updatePeople]    afterReturn
    Dao     Info    2019-53-25 20:53:19,19  [updatePeople]    before
    Dao     Info    2019-53-25 20:53:20,20  [updatePeople]    afterReturn
    Service Info    2019-53-25 20:53:20,20  [batchUpdPeople]    afterReturn
    Service Info    2019-53-25 20:53:20,20  [batchQuyPeople]    before
    Dao     Info    2019-53-25 20:53:20,20  [queryPeople]    before
    Dao     Info    2019-53-25 20:53:20,20  [queryPeople]    afterReturn
    Dao     Info    2019-53-25 20:53:20,20  [queryPeople]    before
    Dao     Info    2019-53-25 20:53:20,20  [queryPeople]    afterReturn
    Service Info    2019-53-25 20:53:20,20  [batchQuyPeople]    afterReturn
    query:Peoplename:aPeopleupdateage:23sex:0
    result:Peoplename:aPeopleupdateage:23sex:0
    query:Peoplename:bPeopleupdateage:32sex:1
    result:Peoplename:bPeopleupdateage:32sex:1
    Peoplename:nullage:nullsex:null
    Service Info    2019-53-25 20:53:20,20  [batchDelPeople]    before
    Dao     Info    2019-53-25 20:53:20,20  [delPeople]    before
    Dao     Info    2019-53-25 20:53:20,20  [delPeople]    afterReturn
    Service Info    2019-53-25 20:53:20,20  [batchDelPeople]    afterReturn
    Peoplename:nullage:nullsex:null
    Service Info    2019-53-25 20:53:20,20  [testTransaction]    before
    Transaction Info    2019-53-25 20:53:20,20  [testTransaction]    before
    Dao     Info    2019-53-25 20:53:20,20  [testTransaction]    before
    Exception in thread "main" java.lang.ArithmeticException: / by zero
       at daoimpl.PeopleDaoImpl.testTransaction(PeopleDaoImpl.java:94)
       at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
       at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
       at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
       at java.lang.reflect.Method.invoke(Method.java:498)
       at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:343)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:198)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
       at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:56)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
       at org.springframework.aop.framework.adapter.AfterReturningAdviceInterceptor.invoke(AfterReturningAdviceInterceptor.java:55)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
       at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:93)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
       at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:212)
       at com.sun.proxy.$Proxy4.testTransaction(Unknown Source)
       at serviceimpl.PeopleServiceImpl.testTransaction(PeopleServiceImpl.java:62)
       at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
       at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
       at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
       at java.lang.reflect.Method.invoke(Method.java:498)
       at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:343)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:198)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
       at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:56)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
       at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:56)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
       at org.springframework.aop.framework.adapter.AfterReturningAdviceInterceptor.invoke(AfterReturningAdviceInterceptor.java:55)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
       at org.springframework.aop.framework.adapter.AfterReturningAdviceInterceptor.invoke(AfterReturningAdviceInterceptor.java:55)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
       at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:93)
       at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
       at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:212)
       at com.sun.proxy.$Proxy5.testTransaction(Unknown Source)
       at client.Main.main(Main.java:104)
    
    

## 3.总结

xml配置比较臃肿，但是与框架的耦合较低；  
注解使用方便，与框架耦合较高。
