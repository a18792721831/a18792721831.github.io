---
layout: post
title: "MyBatis映射多对多关系映射&oracle数据库表的操作"
date: 2019-03-08 20:31:37 +0800
categories: [MyBatis最全例子, MyBatis多对多关系, Oracle创建表、序列、索引、说明以及约束, MyBatis有这一个就够了, MyBatis所有知识的练习]
description: "本文详细介绍了在MyBatis中实现多对多关系的映射方法，包括数据库设计、实体类创建、Mapper文件编写及测试过程。通过具体实例展示了如何处理多对多关系的数据查询和插入。"
keywords: MyBatis最全例子, MyBatis多对多关系, Oracle创建表、序列、索引、说明以及约束, MyBatis有这一个就够了, MyBatis所有知识的练习
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88319971
> - 发布时间：2019-03-08 20:31:37
> - 阅读量：931
> - 分类：java同时被 3 个专栏收录, 订阅专栏, mybatis, MyBatis
> - 标签：#MyBatis最全例子, #MyBatis多对多关系, #Oracle创建表、序列、索引、说明以及约束, #MyBatis有这一个就够了, #MyBatis所有知识的练习

## 摘要

文章浏览阅读931次。本文详细介绍了在MyBatis中实现多对多关系的映射方法，包括数据库设计、实体类创建、Mapper文件编写及测试过程。通过具体实例展示了如何处理多对多关系的数据查询和插入。

---

#### MyBatis多对多映射

  * 1.多对多关系
  * 2.关系映射
  * 3.例子
  *     * 3.1数据库准备
    *       * 3.1.1删除表（原来存在表的话）
      * 3.1.2创建表
      * 3.1.3增加表的约束
      * 3.1.4创建序列
      * 3.1.5创建索引
      * 3.1.6创建列说明
      * 3.1.7查看说明
    * 3.2数据准备
    *       * 3.2.1创建一个MyBatis工程
      * 3.2.2导入jar包
      * 3.2.3创建实体
      * 3.2.4创建工具类
      * 3.2.5创建properties
      * 3.2.6创建resource
      * 3.2.7创建insertpeplemapper
      * 3.2.8增加service数据
      * 3.2.9增加中间表数据
    * 3.3按照id查询数据
    *       * 3.3.1按照id查询people
      * 3.3.2按照id查询service
      * 3.3.3按照people:id查询service
      * 3.3.4按照service:id查询people
    * 3.4 多对多关系
    *       * 3.4.1多对多关系people-service查询方式
      * 3.4.2多对多关系people-service结果方式
      * 3.4.3多对多关系service-people查询方式
      * 3.4.4 对多对关系service-people结果方式
    * 3.5 测试
    *       * 3.5.1新增一个用户
      * 3.5.2 新增一个服务
      * 3.5.3查询用户
      * 3.5.4查询服务
      * 3.5.5增加关系
      * 3.5.6查询用户
      * 3.5.7查询服务
  * 4.总结

## 1.多对多关系

生活总是公平的，你能得到的大部分东西，其他人也有方法得到。  
同样的，你可用拥有更多不同的东西。  
上述描述的就是多对多的关系。  
举个例子：  
生活中有各种各样的服务：停车服务、理发服务、出行服务、存款服务、取款服务。。。。  
一个人可以享受使用多个服务，同样的，一个服务可以被多个人享受。

## 2.关系映射

1.通过查询映射  
2.通过结果映射

## 3.例子

### 3.1数据库准备

#### 3.1.1删除表（原来存在表的话）
    
    
    DROP TABLE people;  --如果表存在就删除表
    
    
    
    DROP TABLE service;--删除表如果表存在
    
    
    
    DROP TABLE people_service; --如果表存在就删除
    

#### 3.1.2创建表
    
    
    CREATE TABLE service(        --创建表
    ID NUMBER NOT NULL,          --定义表字段
    NAME VARCHAR2(50) NOT NULL,  --定义表字段
    remark VARCHAR2(50)          --定义表字段
    );
    
    
    
    CREATE TABLE people(          --创建表
    ID NUMBER NOT NULL,           --定义表字段
    NAME VARCHAR2(50) NOT NULL,   --定义表字段
    age NUMBER NOT NULL,          --定义表字段
    sex NUMBER NOT NULL           --定义表字段
    );
    
    
    
    CREATE TABLE people_service(         --创建中间表
    people_id NUMBER NOT NULL,           --定义表字段
    service_id NUMBER NOT NULL           --定义表字段
    );
    

#### 3.1.3增加表的约束
    
    
    ALTER TABLE service         --修改表
    ADD CONSTRAINT service_pk   --增加约束
    PRIMARY KEY (ID);           --主键约束
    
    ALTER TABLE service         --修改表
    ADD CONSTRAINT service_only --增加约束
    UNIQUE (ID);                --唯一性约束
    
    
    
    ALTER TABLE people            --修改表
    ADD CONSTRAINT people_pk      --增加约束
    PRIMARY KEY (ID);             --主键约束
    
    ALTER TABLE people            --修改表
    ADD CONSTRAINT people_notnull --增加约束
    UNIQUE (ID);                  --唯一性约束
    
    ALTER TABLE people            --修改表
    ADD CONSTRAINT people_age     --增加约束
    CHECK(age < 150 AND age > 0); --检查约束0到150之间
    
    ALTER TABLE people            --修改表
    ADD CONSTRAINT people_sex     --增加约束
    CHECK(sex IN (0,1));          --检查约束0或1
    
    
    
    ALTER TABLE people_service           --修改表
    ADD CONSTRAINT people_fk             --增加约束
    FOREIGN KEY(people_id)               --外键约束
    REFERENCES people(ID)                --被参照表及字段
    ON DELETE CASCADE;                   --可级联删除ON DELETE SET NULL 级联置空
    
    ALTER TABLE people_service           --修改表
    ADD CONSTRAINT service_fk            --增加约束
    FOREIGN KEY(service_id)              --外键约束
    REFERENCES service(ID)               --被参照表及字段
    ON DELETE CASCADE;                   --可级联删除
    

#### 3.1.4创建序列
    
    
    CREATE SEQUENCE seq_service --创建序列
    MINVALUE 1                  --最小值1
    MAXVALUE 99999              --最大值99999
    START WITH 1                --从1开始
    INCREMENT BY 1              --每次增加1
    CACHE 20;                   --缓存20
    
    
    
    CREATE SEQUENCE seq_people    --创建序列
    MINVALUE 1                    --最小值1
    MAXVALUE 99999                --最大值99999
    START WITH 1                  --从1开始
    INCREMENT BY 1                --每次增加1
    CACHE 20;                     --缓存20
    

#### 3.1.5创建索引

（主键约束会自动增加索引）
    
    
    CREATE UNIQUE INDEX service_index--创建索引
    ON service(ID);                  --参照字段
    
    
    
    CREATE UNIQUE INDEX people_index --创建索引
    ON people(ID);                   --参照字段
    

#### 3.1.6创建列说明
    
    
    COMMENT ON TABLE service IS '服务表:存储所有的服务';                    --增加说明
    COMMENT ON COLUMN service.id IS '服务的id,不可以为空，主键，不可重复';  --增加说明 
    COMMENT ON COLUMN service.name IS '服务的名称,不可以为空';              --增加说明
    COMMENT ON COLUMN service.remark IS '服务的说明,可以为空';              --增加说明
    
    
    
    COMMENT ON COLUMN people.id IS '人的id,不可为空,不可重复,主键';
    COMMENT ON COLUMN people.name IS '人的姓名，不可为空,最大值50';
    COMMENT ON COLUMN people.age IS '人的年龄，不可为空';
    COMMENT ON COLUMN people.sex IS '人的性别，不可为空';
    

#### 3.1.7查看说明
    
    
    SELECT *
      FROM USER_COL_COMMENTS UCC
     WHERE UCC.TABLE_NAME = UPPER('service');--查看说明
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/dd496d24cf27dd7d845c078033c6a822.png)
    
    
    SELECT * FROM USER_COL_COMMENTS UCC WHERE UCC.TABLE_NAME = UPPER('people')
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ad944cf24a0fcd5f41f729628bf2913f.png)

### 3.2数据准备

#### 3.2.1创建一个MyBatis工程

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c3884fddef26abeaa70194ebfe1a1059.png)

#### 3.2.2导入jar包

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7a10c32701b05056df8a2333cd0e1790.png)

#### 3.2.3创建实体

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/defda3e5ed4b0dbc80a3ef9010ca22f9.png)
    
    
    package domain;
    
    import java.io.Serializable;
    import java.util.List;
    
    public class People implements Serializable {
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = -3270893239281340723L;
    
    	private Long id;
    
    	private String name;
    
    	private Integer age;
    
    	private Integer sex;
    
    	private List<Service> services;
    	
    	public List<Service> getServices() {
    		return services;
    	}
    
    	public void setServices(List<Service> services) {
    		this.services = services;
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
    				+ this.age + ",sex=" + this.sex + ",service=" + this.services
    				+ "]";
    	}
    
    }
    
    
    
    
    
    package domain;
    
    import java.io.Serializable;
    import java.util.List;
    
    public class Service implements Serializable {
    
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
    
    
    
    
    package domain;
    
    import java.io.Serializable;
    
    public class PeopleService implements Serializable {
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 7074684736563312938L;
    
    	private Long people_fk;
    
    	private Long service_fk;
    
    	public Long getPeople_fk() {
    		return people_fk;
    	}
    
    	public void setPeople_fk(Long people_fk) {
    		this.people_fk = people_fk;
    	}
    
    	public Long getService_fk() {
    		return service_fk;
    	}
    
    	public void setService_fk(Long service_fk) {
    		this.service_fk = service_fk;
    	}
    
    }
    
    

#### 3.2.4创建工具类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/25c73615da9502d865e16a81441928af.png)
    
    
    package util;
    
    import java.io.Reader;
    
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    
    public class MyBatisSessionUtils {
    
    	private static SqlSessionFactory sessionFactory = null;
    
    	private static final String PATH = "resource/mybatis.xml";
    
    	static {
    		try {
    			Reader reader = Resources.getResourceAsReader(PATH);
    			sessionFactory = new SqlSessionFactoryBuilder().build(reader);
    		} catch (Exception e) {
    			e.printStackTrace();
    		}
    	}
    
    	public static SqlSession getSession() {
    		return sessionFactory.openSession();
    	}
    }
    
    

#### 3.2.5创建properties

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8fcf086da520338502845727103042eb.png)  
log4j.properties
    
    
    # Global logging configuration
    log4j.rootLogger=ERROR, stdout
    # MyBatis logging configuration...
    log4j.logger.com=DEBUG
    # Console output...
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
    
    

ojdbc.properties
    
    
    driver=oracle.jdbc.driver.OracleDriver
    url=jdbc:oracle:thin:@127.0.0.1:1521:oracle
    username=study
    password=study
    

#### 3.2.6创建resource

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/419092d91b5f694d974ab9a44a872dc3.png)
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
      <!-- 配置外在化 -->
      <properties >
      	<property name="resource" value="property/ojdbc.properties"/>
      	<property name="resource" value="property/log4j.properties"/>
      </properties>
      <settings>
       	<!-- 延迟加载全局 -->
      	<setting name="lazyLoadingEnabled" value="true"/>
      	<!-- 关联对象属性的延迟加载 -->
      	<setting name="aggressiveLazyLoading" value="false"/>
      </settings>
      <!-- 改变运行时行为 -->
      <typeAliases>
    	  <!-- 配置别名 -->
      	<typeAlias type="domain.People" alias="people"/>
    	<typeAlias type="domain.Service" alias="service"/>
    	<typeAlias type="domain.PeopleService" alias="people_service"/>
      </typeAliases>
      <!-- 环境配置 -->
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
      </mappers>
    </configuration>
    

#### 3.2.7创建insertpeplemapper

baseMapper.xml  
baseMapper是所有表的全字段和表名
    
    
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
    	<sql id="str_select_service">
    		s.id,s.name,s.remark
    	</sql>
    	<sql id="str_table_service">
    		service s
    	</sql>
    	<sql id="str_select_people_service">
    		ps.people_id,ps.service_id
    	</sql>
    	<sql id="str_table_people_service">
    		people_service ps
    	</sql>
    </mapper>
    

insertPeopleMapper.xml  
用来插入数据
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="insertPeopleMapper">
    	<insert id="insertPeople" parameterType="people">
    		insert into 
    		<include refid="baseMapper.str_table_people"></include>
    		(<include refid="baseMapper.str_select_people"></include>) 
    		values(SEQ_PEOPLE.NEXTVAL,#{name},#{age},#{sex})
    	</insert>
    </mapper>
    

创建测试类测试
    
    
    @Test
    	public void insertOnePeople(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		People people = new People();
    		people.setName("people0");
    		people.setAge(22);
    		people.setSex(1);
    		System.out.println(session.insert("insertPeopleMapper.insertPeople", people));
    		session.commit();
    		session.close();
    	}
    

把mapper.xml加入到mybatis.xml文件中，测试运行：
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 1678854096.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@641147d0]
    DEBUG [main] - ==>  Preparing: insert into people p ( p.id,p.name,p.age,p.sex ) values(SEQ_PEOPLE.NEXTVAL,?,?,?) 
    DEBUG [main] - ==> Parameters: people0(String), 22(Integer), 1(Integer)
    DEBUG [main] - <==    Updates: 1
    1
    DEBUG [main] - Committing JDBC Connection [oracle.jdbc.driver.T4CConnection@641147d0]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@641147d0]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@641147d0]
    DEBUG [main] - Returned connection 1678854096 to pool.
    
    

查询mapper  
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
    

编写测试方法
    
    
    @Test
    	public void selectPeople(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		People people = new People();
    		session.selectList("selectPeopleMapper.selectPeople", people).forEach(
    				p -> System.out.println(p));
    		session.close();
    	}
    

mybatis.xml中加入新增加的mapper文件  
运行测试
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 364604394.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@15bb6bea]
    DEBUG [main] - ==>  Preparing: select p.id,p.name,p.age,p.sex from people p 
    DEBUG [main] - ==> Parameters: 
    DEBUG [main] - <==      Total: 3
    people [id=141,name=people0,age=22,sex=1,service=null]
    people [id=142,name=people0,age=22,sex=1,service=null]
    people [id=143,name=people0,age=22,sex=1,service=null]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@15bb6bea]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@15bb6bea]
    DEBUG [main] - Returned connection 364604394 to pool.
    
    

编写批量增加数据的方法：
    
    
    @Test
    	public void batchAddPeople(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		People people = new People();
    		for(int i = 1;i < 200;i++){
    			people.setName("people"+i);
    			people.setAge(i%120 + 1);
    			people.setSex(i%2);
    			session.insert("insertPeopleMapper.insertPeople", people);
    		}
    		session.commit();
    		session.close();
    	}
    

运行结果:  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4c58443d00442c0b908e96053bd12020.png)

#### 3.2.8增加service数据

创建insertServiceMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="insertServiceMapper">
    	<insert id="insertService" parameterType="service">
    		insert into 
    		<include refid="baseMapper.str_table_service"></include>
    		(<include refid="baseMapper.str_select_service"></include>) 
    		values(SEQ_PEOPLE.NEXTVAL,#{name},#{remark})
    	</insert>
    </mapper>
    

编写测试方法
    
    
    @Test
    	public void insertOneService(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		Service service = new Service();
    		service.setName("service0");
    		service.setRemark("remark0");
    		System.out.println(session.insert("insertServiceMapper.insertService",
    				service));
    		session.commit();
    		session.close();
    	}
    

增加到mybatis.xml中  
运行测试:
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 37380050.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@23a5fd2]
    DEBUG [main] - ==>  Preparing: insert into service s ( s.id,s.name,s.remark ) values(SEQ_PEOPLE.NEXTVAL,?,?) 
    DEBUG [main] - ==> Parameters: service0(String), remark0(String)
    DEBUG [main] - <==    Updates: 1
    1
    DEBUG [main] - Committing JDBC Connection [oracle.jdbc.driver.T4CConnection@23a5fd2]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@23a5fd2]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@23a5fd2]
    DEBUG [main] - Returned connection 37380050 to pool.
    
    

编写查询的mapper  
selectServiceMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectServiceMapper">
    	<select id="selectService" parameterType="service" resultType="service">
    		select 
    		<include refid="baseMapper.str_select_service"></include>
    		from 
    		<include refid="baseMapper.str_table_service"></include>
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
    

编写查询的测试方法
    
    
    @Test
    	public void selectService(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		Service service = new Service();
    		session.selectList("selectServiceMapper.selectService", service)
    				.forEach(p -> System.out.println(p));
    		session.close();
    	}
    

加入到mybatis.xml中  
运行测试
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 1473611564.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@57d5872c]
    DEBUG [main] - ==>  Preparing: select s.id,s.name,s.remark from service s 
    DEBUG [main] - ==> Parameters: 
    DEBUG [main] - <==      Total: 2
    [id=464,name=service0,remark=remark0,peoples=null]
    [id=665,name=service0,remark=remark0,peoples=null]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@57d5872c]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@57d5872c]
    DEBUG [main] - Returned connection 1473611564 to pool.
    
    

批量增加服务  
编写批量方法：
    
    
    @Test
    	public void batchAddService(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		Service service = new Service();
    		for(int i = 1;i < 200;i++){
    			service.setName("service"+i);
    			service.setRemark("remark"+i);
    			session.insert("insertServiceMapper.insertService", service);
    		}
    		session.commit();
    		session.close();
    	}
    

运行测试结果  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c9962afe2d57cfb566d25d2f16699c17.png)

#### 3.2.9增加中间表数据

insertPeopleServiceMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="insertPeopleServiceMapper">
    	<insert id="insertPeopleService" parameterType="people_service">
    		insert into 
    		<include refid="baseMapper.str_table_people_service"></include>
    		(<include refid="baseMapper.str_select_people_service"></include>) 
    		values(#{people_fk},#{service_fk})
    	</insert>
    </mapper>
    

因为存在外键的关系，所以不能随便的加  
需要根据查询结果进行设置  
编写中间表添加方法
    
    
    @Test
    	public void doPeopleService(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		List<People> peoples = session.selectList(
    				"selectPeopleMapper.selectPeople", new People());
    		List<Service> services = session.selectList(
    				"selectServiceMapper.selectService", new Service());
    		peoples.forEach(people -> {
    			services.forEach(service -> {
    				PeopleService peopleService = new PeopleService();
    				peopleService.setPeople_fk(people.getId());
    				peopleService.setService_fk(service.getId());
    				System.out.println(session.insert(
    						"insertPeopleServiceMapper.insertPeopleService",
    						peopleService));
    			});
    		});
    		session.commit();
    		session.close();
    	}
    

添加到mybatis.xml中  
执行测试方法：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/06eb0730dc77d5e5650e6f1123442d94.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/01fdefbaeb37286524de1c5dd3121a69.png)  
402*201=80802  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/884509f91bf6329dd6aec3d1b3d22737.png)  
所以，多对多的关系非常的复杂，想要完全描述多对多的关系也是一个数据量庞大的操作。  
编写查询中间表  
selectPeopleServiceMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectPeopleServiceMapper">
    	<select id="selectPeopleService" parameterType="people_service" resultType="people_service">
    		select 
    		<include refid="baseMapper.str_select_people_service"></include>
    		from 
    		<include refid="baseMapper.str_table_people_service"></include>
    		<where>
    			<if test="people_fk != null and people_fk != ''">
    		  	 	and people_id=#{people_fk}
    		  	 </if>
    		  	 <if test="service_fk != null and service_fk != ''">
    		  	 	and service_id=#{service_fk}
    		  	 </if>
    		</where>
    	</select>
    </mapper>
    

加入mybatis.xml中  
编写测试方法
    
    
    @Test
    	public void selectPeopleService(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		PeopleService peopleService = new PeopleService();
    		peopleService.setService_fk(105L);
    		session.selectList("selectPeopleServiceMapper.selectPeopleService",
    				peopleService).forEach(p -> System.out.println(p));
    		session.close();
    	}
    

运行结果  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e5a87e9fde5132cac305905e098706be.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d658223776136d3c2b7954c5cbdff630.png)  
到此数据准备完成。

### 3.3按照id查询数据

#### 3.3.1按照id查询people

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
    

加入mybatis.xml  
编写测试方法
    
    
    @Test
    	public void selectPeopleById(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println(session.selectOne("peopleMapper.selectById", 269));
    		session.close();
    	}
    

运行结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 660017404.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@275710fc]
    DEBUG [main] - ==>  Preparing: select p.id,p.name,p.age,p.sex from people p WHERE id=? 
    DEBUG [main] - ==> Parameters: 269(Integer)
    DEBUG [main] - <==      Total: 1
    people [id=269,name=people5,age=6,sex=1,service=null]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@275710fc]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@275710fc]
    DEBUG [main] - Returned connection 660017404 to pool.
    
    

#### 3.3.2按照id查询service

selectServiceByIdMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="serviceMapper">
    	<select id="selectById" parameterType="Long" resultType="service">
    		select <include refid="baseMapper.str_select_service"></include>
    		from <include refid="baseMapper.str_table_service"></include>
    		<where>
    			<if test="_parameter != null and _parameter != ''">
    				and id=#{_parameter}
    			</if>
    		</where>
    	</select>
    </mapper>
    

加入mybatis.xml  
编写测试方法
    
    
    @Test
    	public void selectServiceById(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println(session.selectOne("serviceMapper.selectById", 169L));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 1386883398.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@52aa2946]
    DEBUG [main] - ==>  Preparing: select s.id,s.name,s.remark from service s WHERE id=? 
    DEBUG [main] - ==> Parameters: 169(Long)
    DEBUG [main] - <==      Total: 1
    [id=169,name=service169,remark=remark169,peoples=null]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@52aa2946]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@52aa2946]
    DEBUG [main] - Returned connection 1386883398 to pool.
    
    

#### 3.3.3按照people:id查询service

selectServiceByPeopleIdMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectServiceByPeopleIdMapper">
    	<select id="selectById" parameterType="Long" resultType="service">
    		select <include refid="baseMapper.str_select_service"></include>
    		from <include refid="baseMapper.str_table_service"></include>
    		,<include refid="baseMapper.str_table_people_service"></include>
    		<where>
    			s.id = ps.service_id
    			<if test="_parameter != null and _parameter != ''">
    				and ps.people_id=#{_parameter}
    			</if>
    		</where>
    	</select>
    </mapper>
    

加入mybatis.xml中  
编写测试方法：
    
    
    @Test
    	public void selectServiceByPeopleId(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		session.selectList("selectServiceByPeopleIdMapper.selectById", 266L)
    				.forEach(p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 1564984895.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@5d47c63f]
    DEBUG [main] - ==>  Preparing: select s.id,s.name,s.remark from service s , people_service ps WHERE s.id = ps.service_id and ps.people_id=? 
    DEBUG [main] - ==> Parameters: 266(Long)
    DEBUG [main] - <==      Total: 201
    [id=464,name=service0,remark=remark0,peoples=null]
    [id=665,name=service0,remark=remark0,peoples=null]
    [id=1,name=service1,remark=remark1,peoples=null]
    [id=2,name=service2,remark=remark2,peoples=null]
    [id=3,name=service3,remark=remark3,peoples=null]
    [id=4,name=service4,remark=remark4,peoples=null]
    [id=5,name=service5,remark=remark5,peoples=null]
    [id=6,name=service6,remark=remark6,peoples=null]
    [id=7,name=service7,remark=remark7,peoples=null]
    [id=8,name=service8,remark=remark8,peoples=null]
    [id=9,name=service9,remark=remark9,peoples=null]
    [id=10,name=service10,remark=remark10,peoples=null]
    [id=11,name=service11,remark=remark11,peoples=null]
    [id=12,name=service12,remark=remark12,peoples=null]
    [id=13,name=service13,remark=remark13,peoples=null]
    [id=14,name=service14,remark=remark14,peoples=null]
    [id=15,name=service15,remark=remark15,peoples=null]
    [id=16,name=service16,remark=remark16,peoples=null]
    [id=17,name=service17,remark=remark17,peoples=null]
    [id=18,name=service18,remark=remark18,peoples=null]
    [id=19,name=service19,remark=remark19,peoples=null]
    [id=20,name=service20,remark=remark20,peoples=null]
    [id=21,name=service21,remark=remark21,peoples=null]
    [id=22,name=service22,remark=remark22,peoples=null]
    [id=23,name=service23,remark=remark23,peoples=null]
    [id=24,name=service24,remark=remark24,peoples=null]
    [id=25,name=service25,remark=remark25,peoples=null]
    [id=26,name=service26,remark=remark26,peoples=null]
    [id=27,name=service27,remark=remark27,peoples=null]
    [id=28,name=service28,remark=remark28,peoples=null]
    [id=29,name=service29,remark=remark29,peoples=null]
    [id=30,name=service30,remark=remark30,peoples=null]
    [id=31,name=service31,remark=remark31,peoples=null]
    [id=32,name=service32,remark=remark32,peoples=null]
    [id=33,name=service33,remark=remark33,peoples=null]
    [id=34,name=service34,remark=remark34,peoples=null]
    [id=35,name=service35,remark=remark35,peoples=null]
    [id=36,name=service36,remark=remark36,peoples=null]
    [id=37,name=service37,remark=remark37,peoples=null]
    [id=38,name=service38,remark=remark38,peoples=null]
    [id=39,name=service39,remark=remark39,peoples=null]
    [id=40,name=service40,remark=remark40,peoples=null]
    [id=41,name=service41,remark=remark41,peoples=null]
    [id=42,name=service42,remark=remark42,peoples=null]
    [id=43,name=service43,remark=remark43,peoples=null]
    [id=44,name=service44,remark=remark44,peoples=null]
    [id=45,name=service45,remark=remark45,peoples=null]
    [id=46,name=service46,remark=remark46,peoples=null]
    [id=47,name=service47,remark=remark47,peoples=null]
    [id=48,name=service48,remark=remark48,peoples=null]
    [id=49,name=service49,remark=remark49,peoples=null]
    [id=50,name=service50,remark=remark50,peoples=null]
    [id=51,name=service51,remark=remark51,peoples=null]
    [id=52,name=service52,remark=remark52,peoples=null]
    [id=53,name=service53,remark=remark53,peoples=null]
    [id=54,name=service54,remark=remark54,peoples=null]
    [id=55,name=service55,remark=remark55,peoples=null]
    [id=56,name=service56,remark=remark56,peoples=null]
    [id=57,name=service57,remark=remark57,peoples=null]
    [id=58,name=service58,remark=remark58,peoples=null]
    [id=59,name=service59,remark=remark59,peoples=null]
    [id=60,name=service60,remark=remark60,peoples=null]
    [id=61,name=service61,remark=remark61,peoples=null]
    [id=62,name=service62,remark=remark62,peoples=null]
    [id=63,name=service63,remark=remark63,peoples=null]
    [id=64,name=service64,remark=remark64,peoples=null]
    [id=65,name=service65,remark=remark65,peoples=null]
    [id=66,name=service66,remark=remark66,peoples=null]
    [id=67,name=service67,remark=remark67,peoples=null]
    [id=68,name=service68,remark=remark68,peoples=null]
    [id=69,name=service69,remark=remark69,peoples=null]
    [id=70,name=service70,remark=remark70,peoples=null]
    [id=71,name=service71,remark=remark71,peoples=null]
    [id=72,name=service72,remark=remark72,peoples=null]
    [id=73,name=service73,remark=remark73,peoples=null]
    [id=74,name=service74,remark=remark74,peoples=null]
    [id=75,name=service75,remark=remark75,peoples=null]
    [id=76,name=service76,remark=remark76,peoples=null]
    [id=77,name=service77,remark=remark77,peoples=null]
    [id=78,name=service78,remark=remark78,peoples=null]
    [id=79,name=service79,remark=remark79,peoples=null]
    [id=80,name=service80,remark=remark80,peoples=null]
    [id=81,name=service81,remark=remark81,peoples=null]
    [id=82,name=service82,remark=remark82,peoples=null]
    [id=83,name=service83,remark=remark83,peoples=null]
    [id=84,name=service84,remark=remark84,peoples=null]
    [id=85,name=service85,remark=remark85,peoples=null]
    [id=86,name=service86,remark=remark86,peoples=null]
    [id=87,name=service87,remark=remark87,peoples=null]
    [id=88,name=service88,remark=remark88,peoples=null]
    [id=89,name=service89,remark=remark89,peoples=null]
    [id=90,name=service90,remark=remark90,peoples=null]
    [id=91,name=service91,remark=remark91,peoples=null]
    [id=92,name=service92,remark=remark92,peoples=null]
    [id=93,name=service93,remark=remark93,peoples=null]
    [id=94,name=service94,remark=remark94,peoples=null]
    [id=95,name=service95,remark=remark95,peoples=null]
    [id=96,name=service96,remark=remark96,peoples=null]
    [id=97,name=service97,remark=remark97,peoples=null]
    [id=98,name=service98,remark=remark98,peoples=null]
    [id=99,name=service99,remark=remark99,peoples=null]
    [id=100,name=service100,remark=remark100,peoples=null]
    [id=101,name=service101,remark=remark101,peoples=null]
    [id=102,name=service102,remark=remark102,peoples=null]
    [id=103,name=service103,remark=remark103,peoples=null]
    [id=104,name=service104,remark=remark104,peoples=null]
    [id=105,name=service105,remark=remark105,peoples=null]
    [id=106,name=service106,remark=remark106,peoples=null]
    [id=107,name=service107,remark=remark107,peoples=null]
    [id=108,name=service108,remark=remark108,peoples=null]
    [id=109,name=service109,remark=remark109,peoples=null]
    [id=110,name=service110,remark=remark110,peoples=null]
    [id=111,name=service111,remark=remark111,peoples=null]
    [id=112,name=service112,remark=remark112,peoples=null]
    [id=113,name=service113,remark=remark113,peoples=null]
    [id=114,name=service114,remark=remark114,peoples=null]
    [id=115,name=service115,remark=remark115,peoples=null]
    [id=116,name=service116,remark=remark116,peoples=null]
    [id=117,name=service117,remark=remark117,peoples=null]
    [id=118,name=service118,remark=remark118,peoples=null]
    [id=119,name=service119,remark=remark119,peoples=null]
    [id=120,name=service120,remark=remark120,peoples=null]
    [id=121,name=service121,remark=remark121,peoples=null]
    [id=122,name=service122,remark=remark122,peoples=null]
    [id=123,name=service123,remark=remark123,peoples=null]
    [id=124,name=service124,remark=remark124,peoples=null]
    [id=125,name=service125,remark=remark125,peoples=null]
    [id=126,name=service126,remark=remark126,peoples=null]
    [id=127,name=service127,remark=remark127,peoples=null]
    [id=128,name=service128,remark=remark128,peoples=null]
    [id=129,name=service129,remark=remark129,peoples=null]
    [id=130,name=service130,remark=remark130,peoples=null]
    [id=131,name=service131,remark=remark131,peoples=null]
    [id=132,name=service132,remark=remark132,peoples=null]
    [id=133,name=service133,remark=remark133,peoples=null]
    [id=134,name=service134,remark=remark134,peoples=null]
    [id=135,name=service135,remark=remark135,peoples=null]
    [id=136,name=service136,remark=remark136,peoples=null]
    [id=137,name=service137,remark=remark137,peoples=null]
    [id=138,name=service138,remark=remark138,peoples=null]
    [id=139,name=service139,remark=remark139,peoples=null]
    [id=140,name=service140,remark=remark140,peoples=null]
    [id=141,name=service141,remark=remark141,peoples=null]
    [id=142,name=service142,remark=remark142,peoples=null]
    [id=143,name=service143,remark=remark143,peoples=null]
    [id=144,name=service144,remark=remark144,peoples=null]
    [id=145,name=service145,remark=remark145,peoples=null]
    [id=146,name=service146,remark=remark146,peoples=null]
    [id=147,name=service147,remark=remark147,peoples=null]
    [id=148,name=service148,remark=remark148,peoples=null]
    [id=149,name=service149,remark=remark149,peoples=null]
    [id=150,name=service150,remark=remark150,peoples=null]
    [id=151,name=service151,remark=remark151,peoples=null]
    [id=152,name=service152,remark=remark152,peoples=null]
    [id=153,name=service153,remark=remark153,peoples=null]
    [id=154,name=service154,remark=remark154,peoples=null]
    [id=155,name=service155,remark=remark155,peoples=null]
    [id=156,name=service156,remark=remark156,peoples=null]
    [id=157,name=service157,remark=remark157,peoples=null]
    [id=158,name=service158,remark=remark158,peoples=null]
    [id=159,name=service159,remark=remark159,peoples=null]
    [id=160,name=service160,remark=remark160,peoples=null]
    [id=161,name=service161,remark=remark161,peoples=null]
    [id=162,name=service162,remark=remark162,peoples=null]
    [id=163,name=service163,remark=remark163,peoples=null]
    [id=164,name=service164,remark=remark164,peoples=null]
    [id=165,name=service165,remark=remark165,peoples=null]
    [id=166,name=service166,remark=remark166,peoples=null]
    [id=167,name=service167,remark=remark167,peoples=null]
    [id=168,name=service168,remark=remark168,peoples=null]
    [id=169,name=service169,remark=remark169,peoples=null]
    [id=170,name=service170,remark=remark170,peoples=null]
    [id=171,name=service171,remark=remark171,peoples=null]
    [id=172,name=service172,remark=remark172,peoples=null]
    [id=173,name=service173,remark=remark173,peoples=null]
    [id=174,name=service174,remark=remark174,peoples=null]
    [id=175,name=service175,remark=remark175,peoples=null]
    [id=176,name=service176,remark=remark176,peoples=null]
    [id=177,name=service177,remark=remark177,peoples=null]
    [id=178,name=service178,remark=remark178,peoples=null]
    [id=179,name=service179,remark=remark179,peoples=null]
    [id=180,name=service180,remark=remark180,peoples=null]
    [id=181,name=service181,remark=remark181,peoples=null]
    [id=182,name=service182,remark=remark182,peoples=null]
    [id=183,name=service183,remark=remark183,peoples=null]
    [id=184,name=service184,remark=remark184,peoples=null]
    [id=185,name=service185,remark=remark185,peoples=null]
    [id=186,name=service186,remark=remark186,peoples=null]
    [id=187,name=service187,remark=remark187,peoples=null]
    [id=188,name=service188,remark=remark188,peoples=null]
    [id=189,name=service189,remark=remark189,peoples=null]
    [id=190,name=service190,remark=remark190,peoples=null]
    [id=191,name=service191,remark=remark191,peoples=null]
    [id=192,name=service192,remark=remark192,peoples=null]
    [id=193,name=service193,remark=remark193,peoples=null]
    [id=194,name=service194,remark=remark194,peoples=null]
    [id=195,name=service195,remark=remark195,peoples=null]
    [id=196,name=service196,remark=remark196,peoples=null]
    [id=197,name=service197,remark=remark197,peoples=null]
    [id=198,name=service198,remark=remark198,peoples=null]
    [id=199,name=service199,remark=remark199,peoples=null]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@5d47c63f]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@5d47c63f]
    DEBUG [main] - Returned connection 1564984895 to pool.
    
    

#### 3.3.4按照service:id查询people

selectPeopleByServiceIdMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="selectPeopleByServiceIdMapper">
    	<select id="selectById" parameterType="Long" resultType="people">
    		select <include refid="baseMapper.str_select_people"></include>
    		from <include refid="baseMapper.str_table_people"></include>
    		,<include refid="baseMapper.str_table_people_service"></include>
    		<where>
    			p.id = ps.people_id
    			<if test="_parameter != null and _parameter != ''">
    				and ps.service_id=#{_parameter}
    			</if>
    		</where>
    	</select>
    </mapper>
    

加入到mybatis.xml中  
编写测试方法
    
    
    @Test
    	public void selectPeopleByServiceId(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		session.selectList("selectPeopleByServiceIdMapper.selectById", 198L)
    				.forEach(p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 1256440269.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@4ae3c1cd]
    DEBUG [main] - ==>  Preparing: select p.id,p.name,p.age,p.sex from people p , people_service ps WHERE p.id = ps.people_id and ps.service_id=? 
    DEBUG [main] - ==> Parameters: 198(Long)
    DEBUG [main] - <==      Total: 402
    people [id=267,name=people3,age=4,sex=1,service=null]
    people [id=268,name=people4,age=5,sex=0,service=null]
    people [id=269,name=people5,age=6,sex=1,service=null]
    people [id=270,name=people6,age=7,sex=0,service=null]
    people [id=271,name=people7,age=8,sex=1,service=null]
    people [id=272,name=people8,age=9,sex=0,service=null]
    people [id=273,name=people9,age=10,sex=1,service=null]
    people [id=274,name=people10,age=11,sex=0,service=null]
    people [id=275,name=people11,age=12,sex=1,service=null]
    people [id=141,name=people0,age=22,sex=1,service=null]
    people [id=142,name=people0,age=22,sex=1,service=null]
    people [id=143,name=people0,age=22,sex=1,service=null]
    people [id=265,name=people1,age=2,sex=1,service=null]
    people [id=266,name=people2,age=3,sex=0,service=null]
    people [id=287,name=people23,age=24,sex=1,service=null]
    people [id=288,name=people24,age=25,sex=0,service=null]
    people [id=289,name=people25,age=26,sex=1,service=null]
    people [id=290,name=people26,age=27,sex=0,service=null]
    people [id=291,name=people27,age=28,sex=1,service=null]
    people [id=292,name=people28,age=29,sex=0,service=null]
    people [id=293,name=people29,age=30,sex=1,service=null]
    people [id=294,name=people30,age=31,sex=0,service=null]
    people [id=295,name=people31,age=32,sex=1,service=null]
    people [id=296,name=people32,age=33,sex=0,service=null]
    people [id=297,name=people33,age=34,sex=1,service=null]
    people [id=298,name=people34,age=35,sex=0,service=null]
    people [id=276,name=people12,age=13,sex=0,service=null]
    people [id=277,name=people13,age=14,sex=1,service=null]
    people [id=278,name=people14,age=15,sex=0,service=null]
    people [id=279,name=people15,age=16,sex=1,service=null]
    people [id=280,name=people16,age=17,sex=0,service=null]
    people [id=281,name=people17,age=18,sex=1,service=null]
    people [id=282,name=people18,age=19,sex=0,service=null]
    people [id=283,name=people19,age=20,sex=1,service=null]
    people [id=284,name=people20,age=21,sex=0,service=null]
    people [id=285,name=people21,age=22,sex=1,service=null]
    people [id=286,name=people22,age=23,sex=0,service=null]
    people [id=299,name=people35,age=36,sex=1,service=null]
    people [id=300,name=people36,age=37,sex=0,service=null]
    people [id=301,name=people37,age=38,sex=1,service=null]
    people [id=308,name=people44,age=45,sex=0,service=null]
    people [id=309,name=people45,age=46,sex=1,service=null]
    people [id=310,name=people46,age=47,sex=0,service=null]
    people [id=311,name=people47,age=48,sex=1,service=null]
    people [id=312,name=people48,age=49,sex=0,service=null]
    people [id=313,name=people49,age=50,sex=1,service=null]
    people [id=314,name=people50,age=51,sex=0,service=null]
    people [id=315,name=people51,age=52,sex=1,service=null]
    people [id=316,name=people52,age=53,sex=0,service=null]
    people [id=317,name=people53,age=54,sex=1,service=null]
    people [id=318,name=people54,age=55,sex=0,service=null]
    people [id=302,name=people38,age=39,sex=0,service=null]
    people [id=303,name=people39,age=40,sex=1,service=null]
    people [id=304,name=people40,age=41,sex=0,service=null]
    people [id=305,name=people41,age=42,sex=1,service=null]
    people [id=306,name=people42,age=43,sex=0,service=null]
    people [id=307,name=people43,age=44,sex=1,service=null]
    people [id=331,name=people67,age=68,sex=1,service=null]
    people [id=332,name=people68,age=69,sex=0,service=null]
    people [id=333,name=people69,age=70,sex=1,service=null]
    people [id=334,name=people70,age=71,sex=0,service=null]
    people [id=335,name=people71,age=72,sex=1,service=null]
    people [id=336,name=people72,age=73,sex=0,service=null]
    people [id=337,name=people73,age=74,sex=1,service=null]
    people [id=338,name=people74,age=75,sex=0,service=null]
    people [id=339,name=people75,age=76,sex=1,service=null]
    people [id=340,name=people76,age=77,sex=0,service=null]
    people [id=341,name=people77,age=78,sex=1,service=null]
    people [id=319,name=people55,age=56,sex=1,service=null]
    people [id=320,name=people56,age=57,sex=0,service=null]
    people [id=321,name=people57,age=58,sex=1,service=null]
    people [id=322,name=people58,age=59,sex=0,service=null]
    people [id=323,name=people59,age=60,sex=1,service=null]
    people [id=324,name=people60,age=61,sex=0,service=null]
    people [id=325,name=people61,age=62,sex=1,service=null]
    people [id=326,name=people62,age=63,sex=0,service=null]
    people [id=327,name=people63,age=64,sex=1,service=null]
    people [id=328,name=people64,age=65,sex=0,service=null]
    people [id=329,name=people65,age=66,sex=1,service=null]
    people [id=330,name=people66,age=67,sex=0,service=null]
    people [id=342,name=people78,age=79,sex=0,service=null]
    people [id=343,name=people79,age=80,sex=1,service=null]
    people [id=344,name=people80,age=81,sex=0,service=null]
    people [id=351,name=people87,age=88,sex=1,service=null]
    people [id=352,name=people88,age=89,sex=0,service=null]
    people [id=353,name=people89,age=90,sex=1,service=null]
    people [id=354,name=people90,age=91,sex=0,service=null]
    people [id=355,name=people91,age=92,sex=1,service=null]
    people [id=356,name=people92,age=93,sex=0,service=null]
    people [id=357,name=people93,age=94,sex=1,service=null]
    people [id=358,name=people94,age=95,sex=0,service=null]
    people [id=359,name=people95,age=96,sex=1,service=null]
    people [id=360,name=people96,age=97,sex=0,service=null]
    people [id=361,name=people97,age=98,sex=1,service=null]
    people [id=362,name=people98,age=99,sex=0,service=null]
    people [id=345,name=people81,age=82,sex=1,service=null]
    people [id=346,name=people82,age=83,sex=0,service=null]
    people [id=347,name=people83,age=84,sex=1,service=null]
    people [id=348,name=people84,age=85,sex=0,service=null]
    people [id=349,name=people85,age=86,sex=1,service=null]
    people [id=350,name=people86,age=87,sex=0,service=null]
    people [id=374,name=people110,age=111,sex=0,service=null]
    people [id=375,name=people111,age=112,sex=1,service=null]
    people [id=376,name=people112,age=113,sex=0,service=null]
    people [id=377,name=people113,age=114,sex=1,service=null]
    people [id=378,name=people114,age=115,sex=0,service=null]
    people [id=379,name=people115,age=116,sex=1,service=null]
    people [id=380,name=people116,age=117,sex=0,service=null]
    people [id=381,name=people117,age=118,sex=1,service=null]
    people [id=382,name=people118,age=119,sex=0,service=null]
    people [id=383,name=people119,age=120,sex=1,service=null]
    people [id=384,name=people120,age=1,sex=0,service=null]
    people [id=385,name=people121,age=2,sex=1,service=null]
    people [id=363,name=people99,age=100,sex=1,service=null]
    people [id=364,name=people100,age=101,sex=0,service=null]
    people [id=365,name=people101,age=102,sex=1,service=null]
    people [id=366,name=people102,age=103,sex=0,service=null]
    people [id=367,name=people103,age=104,sex=1,service=null]
    people [id=368,name=people104,age=105,sex=0,service=null]
    people [id=369,name=people105,age=106,sex=1,service=null]
    people [id=370,name=people106,age=107,sex=0,service=null]
    people [id=371,name=people107,age=108,sex=1,service=null]
    people [id=372,name=people108,age=109,sex=0,service=null]
    people [id=373,name=people109,age=110,sex=1,service=null]
    people [id=386,name=people122,age=3,sex=0,service=null]
    people [id=387,name=people123,age=4,sex=1,service=null]
    people [id=388,name=people124,age=5,sex=0,service=null]
    people [id=394,name=people130,age=11,sex=0,service=null]
    people [id=395,name=people131,age=12,sex=1,service=null]
    people [id=396,name=people132,age=13,sex=0,service=null]
    people [id=397,name=people133,age=14,sex=1,service=null]
    people [id=398,name=people134,age=15,sex=0,service=null]
    people [id=399,name=people135,age=16,sex=1,service=null]
    people [id=400,name=people136,age=17,sex=0,service=null]
    people [id=401,name=people137,age=18,sex=1,service=null]
    people [id=402,name=people138,age=19,sex=0,service=null]
    people [id=403,name=people139,age=20,sex=1,service=null]
    people [id=404,name=people140,age=21,sex=0,service=null]
    people [id=405,name=people141,age=22,sex=1,service=null]
    people [id=389,name=people125,age=6,sex=1,service=null]
    people [id=390,name=people126,age=7,sex=0,service=null]
    people [id=391,name=people127,age=8,sex=1,service=null]
    people [id=392,name=people128,age=9,sex=0,service=null]
    people [id=393,name=people129,age=10,sex=1,service=null]
    people [id=418,name=people154,age=35,sex=0,service=null]
    people [id=419,name=people155,age=36,sex=1,service=null]
    people [id=420,name=people156,age=37,sex=0,service=null]
    people [id=421,name=people157,age=38,sex=1,service=null]
    people [id=422,name=people158,age=39,sex=0,service=null]
    people [id=423,name=people159,age=40,sex=1,service=null]
    people [id=424,name=people160,age=41,sex=0,service=null]
    people [id=425,name=people161,age=42,sex=1,service=null]
    people [id=426,name=people162,age=43,sex=0,service=null]
    people [id=427,name=people163,age=44,sex=1,service=null]
    people [id=428,name=people164,age=45,sex=0,service=null]
    people [id=406,name=people142,age=23,sex=0,service=null]
    people [id=407,name=people143,age=24,sex=1,service=null]
    people [id=408,name=people144,age=25,sex=0,service=null]
    people [id=409,name=people145,age=26,sex=1,service=null]
    people [id=410,name=people146,age=27,sex=0,service=null]
    people [id=411,name=people147,age=28,sex=1,service=null]
    people [id=412,name=people148,age=29,sex=0,service=null]
    people [id=413,name=people149,age=30,sex=1,service=null]
    people [id=414,name=people150,age=31,sex=0,service=null]
    people [id=415,name=people151,age=32,sex=1,service=null]
    people [id=416,name=people152,age=33,sex=0,service=null]
    people [id=417,name=people153,age=34,sex=1,service=null]
    people [id=429,name=people165,age=46,sex=1,service=null]
    people [id=430,name=people166,age=47,sex=0,service=null]
    people [id=431,name=people167,age=48,sex=1,service=null]
    people [id=438,name=people174,age=55,sex=0,service=null]
    people [id=439,name=people175,age=56,sex=1,service=null]
    people [id=440,name=people176,age=57,sex=0,service=null]
    people [id=441,name=people177,age=58,sex=1,service=null]
    people [id=442,name=people178,age=59,sex=0,service=null]
    people [id=443,name=people179,age=60,sex=1,service=null]
    people [id=444,name=people180,age=61,sex=0,service=null]
    people [id=445,name=people181,age=62,sex=1,service=null]
    people [id=446,name=people182,age=63,sex=0,service=null]
    people [id=447,name=people183,age=64,sex=1,service=null]
    people [id=448,name=people184,age=65,sex=0,service=null]
    people [id=432,name=people168,age=49,sex=0,service=null]
    people [id=433,name=people169,age=50,sex=1,service=null]
    people [id=434,name=people170,age=51,sex=0,service=null]
    people [id=435,name=people171,age=52,sex=1,service=null]
    people [id=436,name=people172,age=53,sex=0,service=null]
    people [id=437,name=people173,age=54,sex=1,service=null]
    people [id=461,name=people197,age=78,sex=1,service=null]
    people [id=462,name=people198,age=79,sex=0,service=null]
    people [id=463,name=people199,age=80,sex=1,service=null]
    people [id=465,name=people1,age=2,sex=1,service=null]
    people [id=466,name=people2,age=3,sex=0,service=null]
    people [id=467,name=people3,age=4,sex=1,service=null]
    people [id=468,name=people4,age=5,sex=0,service=null]
    people [id=469,name=people5,age=6,sex=1,service=null]
    people [id=470,name=people6,age=7,sex=0,service=null]
    people [id=471,name=people7,age=8,sex=1,service=null]
    people [id=472,name=people8,age=9,sex=0,service=null]
    people [id=449,name=people185,age=66,sex=1,service=null]
    people [id=450,name=people186,age=67,sex=0,service=null]
    people [id=451,name=people187,age=68,sex=1,service=null]
    people [id=452,name=people188,age=69,sex=0,service=null]
    people [id=453,name=people189,age=70,sex=1,service=null]
    people [id=454,name=people190,age=71,sex=0,service=null]
    people [id=455,name=people191,age=72,sex=1,service=null]
    people [id=456,name=people192,age=73,sex=0,service=null]
    people [id=457,name=people193,age=74,sex=1,service=null]
    people [id=458,name=people194,age=75,sex=0,service=null]
    people [id=459,name=people195,age=76,sex=1,service=null]
    people [id=460,name=people196,age=77,sex=0,service=null]
    people [id=473,name=people9,age=10,sex=1,service=null]
    people [id=474,name=people10,age=11,sex=0,service=null]
    people [id=475,name=people11,age=12,sex=1,service=null]
    people [id=482,name=people18,age=19,sex=0,service=null]
    people [id=483,name=people19,age=20,sex=1,service=null]
    people [id=484,name=people20,age=21,sex=0,service=null]
    people [id=485,name=people21,age=22,sex=1,service=null]
    people [id=486,name=people22,age=23,sex=0,service=null]
    people [id=487,name=people23,age=24,sex=1,service=null]
    people [id=488,name=people24,age=25,sex=0,service=null]
    people [id=489,name=people25,age=26,sex=1,service=null]
    people [id=490,name=people26,age=27,sex=0,service=null]
    people [id=491,name=people27,age=28,sex=1,service=null]
    people [id=492,name=people28,age=29,sex=0,service=null]
    people [id=493,name=people29,age=30,sex=1,service=null]
    people [id=476,name=people12,age=13,sex=0,service=null]
    people [id=477,name=people13,age=14,sex=1,service=null]
    people [id=478,name=people14,age=15,sex=0,service=null]
    people [id=479,name=people15,age=16,sex=1,service=null]
    people [id=480,name=people16,age=17,sex=0,service=null]
    people [id=481,name=people17,age=18,sex=1,service=null]
    people [id=505,name=people41,age=42,sex=1,service=null]
    people [id=506,name=people42,age=43,sex=0,service=null]
    people [id=507,name=people43,age=44,sex=1,service=null]
    people [id=508,name=people44,age=45,sex=0,service=null]
    people [id=509,name=people45,age=46,sex=1,service=null]
    people [id=510,name=people46,age=47,sex=0,service=null]
    people [id=511,name=people47,age=48,sex=1,service=null]
    people [id=512,name=people48,age=49,sex=0,service=null]
    people [id=513,name=people49,age=50,sex=1,service=null]
    people [id=514,name=people50,age=51,sex=0,service=null]
    people [id=515,name=people51,age=52,sex=1,service=null]
    people [id=516,name=people52,age=53,sex=0,service=null]
    people [id=494,name=people30,age=31,sex=0,service=null]
    people [id=495,name=people31,age=32,sex=1,service=null]
    people [id=496,name=people32,age=33,sex=0,service=null]
    people [id=497,name=people33,age=34,sex=1,service=null]
    people [id=498,name=people34,age=35,sex=0,service=null]
    people [id=499,name=people35,age=36,sex=1,service=null]
    people [id=500,name=people36,age=37,sex=0,service=null]
    people [id=501,name=people37,age=38,sex=1,service=null]
    people [id=502,name=people38,age=39,sex=0,service=null]
    people [id=503,name=people39,age=40,sex=1,service=null]
    people [id=504,name=people40,age=41,sex=0,service=null]
    people [id=517,name=people53,age=54,sex=1,service=null]
    people [id=518,name=people54,age=55,sex=0,service=null]
    people [id=519,name=people55,age=56,sex=1,service=null]
    people [id=526,name=people62,age=63,sex=0,service=null]
    people [id=527,name=people63,age=64,sex=1,service=null]
    people [id=528,name=people64,age=65,sex=0,service=null]
    people [id=529,name=people65,age=66,sex=1,service=null]
    people [id=530,name=people66,age=67,sex=0,service=null]
    people [id=531,name=people67,age=68,sex=1,service=null]
    people [id=532,name=people68,age=69,sex=0,service=null]
    people [id=533,name=people69,age=70,sex=1,service=null]
    people [id=534,name=people70,age=71,sex=0,service=null]
    people [id=535,name=people71,age=72,sex=1,service=null]
    people [id=536,name=people72,age=73,sex=0,service=null]
    people [id=520,name=people56,age=57,sex=0,service=null]
    people [id=521,name=people57,age=58,sex=1,service=null]
    people [id=522,name=people58,age=59,sex=0,service=null]
    people [id=523,name=people59,age=60,sex=1,service=null]
    people [id=524,name=people60,age=61,sex=0,service=null]
    people [id=525,name=people61,age=62,sex=1,service=null]
    people [id=549,name=people85,age=86,sex=1,service=null]
    people [id=550,name=people86,age=87,sex=0,service=null]
    people [id=551,name=people87,age=88,sex=1,service=null]
    people [id=552,name=people88,age=89,sex=0,service=null]
    people [id=553,name=people89,age=90,sex=1,service=null]
    people [id=554,name=people90,age=91,sex=0,service=null]
    people [id=555,name=people91,age=92,sex=1,service=null]
    people [id=556,name=people92,age=93,sex=0,service=null]
    people [id=557,name=people93,age=94,sex=1,service=null]
    people [id=558,name=people94,age=95,sex=0,service=null]
    people [id=559,name=people95,age=96,sex=1,service=null]
    people [id=537,name=people73,age=74,sex=1,service=null]
    people [id=538,name=people74,age=75,sex=0,service=null]
    people [id=539,name=people75,age=76,sex=1,service=null]
    people [id=540,name=people76,age=77,sex=0,service=null]
    people [id=541,name=people77,age=78,sex=1,service=null]
    people [id=542,name=people78,age=79,sex=0,service=null]
    people [id=543,name=people79,age=80,sex=1,service=null]
    people [id=544,name=people80,age=81,sex=0,service=null]
    people [id=545,name=people81,age=82,sex=1,service=null]
    people [id=546,name=people82,age=83,sex=0,service=null]
    people [id=547,name=people83,age=84,sex=1,service=null]
    people [id=548,name=people84,age=85,sex=0,service=null]
    people [id=560,name=people96,age=97,sex=0,service=null]
    people [id=561,name=people97,age=98,sex=1,service=null]
    people [id=562,name=people98,age=99,sex=0,service=null]
    people [id=569,name=people105,age=106,sex=1,service=null]
    people [id=570,name=people106,age=107,sex=0,service=null]
    people [id=571,name=people107,age=108,sex=1,service=null]
    people [id=572,name=people108,age=109,sex=0,service=null]
    people [id=573,name=people109,age=110,sex=1,service=null]
    people [id=574,name=people110,age=111,sex=0,service=null]
    people [id=575,name=people111,age=112,sex=1,service=null]
    people [id=576,name=people112,age=113,sex=0,service=null]
    people [id=577,name=people113,age=114,sex=1,service=null]
    people [id=578,name=people114,age=115,sex=0,service=null]
    people [id=579,name=people115,age=116,sex=1,service=null]
    people [id=563,name=people99,age=100,sex=1,service=null]
    people [id=564,name=people100,age=101,sex=0,service=null]
    people [id=565,name=people101,age=102,sex=1,service=null]
    people [id=566,name=people102,age=103,sex=0,service=null]
    people [id=567,name=people103,age=104,sex=1,service=null]
    people [id=568,name=people104,age=105,sex=0,service=null]
    people [id=592,name=people128,age=9,sex=0,service=null]
    people [id=593,name=people129,age=10,sex=1,service=null]
    people [id=594,name=people130,age=11,sex=0,service=null]
    people [id=595,name=people131,age=12,sex=1,service=null]
    people [id=596,name=people132,age=13,sex=0,service=null]
    people [id=597,name=people133,age=14,sex=1,service=null]
    people [id=598,name=people134,age=15,sex=0,service=null]
    people [id=599,name=people135,age=16,sex=1,service=null]
    people [id=600,name=people136,age=17,sex=0,service=null]
    people [id=601,name=people137,age=18,sex=1,service=null]
    people [id=602,name=people138,age=19,sex=0,service=null]
    people [id=603,name=people139,age=20,sex=1,service=null]
    people [id=580,name=people116,age=117,sex=0,service=null]
    people [id=581,name=people117,age=118,sex=1,service=null]
    people [id=582,name=people118,age=119,sex=0,service=null]
    people [id=583,name=people119,age=120,sex=1,service=null]
    people [id=584,name=people120,age=1,sex=0,service=null]
    people [id=585,name=people121,age=2,sex=1,service=null]
    people [id=586,name=people122,age=3,sex=0,service=null]
    people [id=587,name=people123,age=4,sex=1,service=null]
    people [id=588,name=people124,age=5,sex=0,service=null]
    people [id=589,name=people125,age=6,sex=1,service=null]
    people [id=590,name=people126,age=7,sex=0,service=null]
    people [id=591,name=people127,age=8,sex=1,service=null]
    people [id=630,name=people166,age=47,sex=0,service=null]
    people [id=631,name=people167,age=48,sex=1,service=null]
    people [id=632,name=people168,age=49,sex=0,service=null]
    people [id=633,name=people169,age=50,sex=1,service=null]
    people [id=634,name=people170,age=51,sex=0,service=null]
    people [id=635,name=people171,age=52,sex=1,service=null]
    people [id=636,name=people172,age=53,sex=0,service=null]
    people [id=637,name=people173,age=54,sex=1,service=null]
    people [id=638,name=people174,age=55,sex=0,service=null]
    people [id=639,name=people175,age=56,sex=1,service=null]
    people [id=640,name=people176,age=57,sex=0,service=null]
    people [id=641,name=people177,age=58,sex=1,service=null]
    people [id=642,name=people178,age=59,sex=0,service=null]
    people [id=643,name=people179,age=60,sex=1,service=null]
    people [id=644,name=people180,age=61,sex=0,service=null]
    people [id=645,name=people181,age=62,sex=1,service=null]
    people [id=646,name=people182,age=63,sex=0,service=null]
    people [id=647,name=people183,age=64,sex=1,service=null]
    people [id=648,name=people184,age=65,sex=0,service=null]
    people [id=649,name=people185,age=66,sex=1,service=null]
    people [id=604,name=people140,age=21,sex=0,service=null]
    people [id=605,name=people141,age=22,sex=1,service=null]
    people [id=606,name=people142,age=23,sex=0,service=null]
    people [id=650,name=people186,age=67,sex=0,service=null]
    people [id=651,name=people187,age=68,sex=1,service=null]
    people [id=652,name=people188,age=69,sex=0,service=null]
    people [id=607,name=people143,age=24,sex=1,service=null]
    people [id=608,name=people144,age=25,sex=0,service=null]
    people [id=653,name=people189,age=70,sex=1,service=null]
    people [id=654,name=people190,age=71,sex=0,service=null]
    people [id=655,name=people191,age=72,sex=1,service=null]
    people [id=609,name=people145,age=26,sex=1,service=null]
    people [id=610,name=people146,age=27,sex=0,service=null]
    people [id=611,name=people147,age=28,sex=1,service=null]
    people [id=656,name=people192,age=73,sex=0,service=null]
    people [id=657,name=people193,age=74,sex=1,service=null]
    people [id=612,name=people148,age=29,sex=0,service=null]
    people [id=613,name=people149,age=30,sex=1,service=null]
    people [id=614,name=people150,age=31,sex=0,service=null]
    people [id=658,name=people194,age=75,sex=0,service=null]
    people [id=659,name=people195,age=76,sex=1,service=null]
    people [id=660,name=people196,age=77,sex=0,service=null]
    people [id=615,name=people151,age=32,sex=1,service=null]
    people [id=616,name=people152,age=33,sex=0,service=null]
    people [id=617,name=people153,age=34,sex=1,service=null]
    people [id=661,name=people197,age=78,sex=1,service=null]
    people [id=662,name=people198,age=79,sex=0,service=null]
    people [id=663,name=people199,age=80,sex=1,service=null]
    people [id=618,name=people154,age=35,sex=0,service=null]
    people [id=619,name=people155,age=36,sex=1,service=null]
    people [id=620,name=people156,age=37,sex=0,service=null]
    people [id=664,name=people0,age=22,sex=1,service=null]
    people [id=621,name=people157,age=38,sex=1,service=null]
    people [id=622,name=people158,age=39,sex=0,service=null]
    people [id=623,name=people159,age=40,sex=1,service=null]
    people [id=624,name=people160,age=41,sex=0,service=null]
    people [id=625,name=people161,age=42,sex=1,service=null]
    people [id=626,name=people162,age=43,sex=0,service=null]
    people [id=627,name=people163,age=44,sex=1,service=null]
    people [id=628,name=people164,age=45,sex=0,service=null]
    people [id=629,name=people165,age=46,sex=1,service=null]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@4ae3c1cd]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@4ae3c1cd]
    DEBUG [main] - Returned connection 1256440269 to pool.
    
    

### 3.4 多对多关系

#### 3.4.1多对多关系people-service查询方式

moreToMoreForPeopleMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="moreToMoreForPeopleMapper">
    	<resultMap type="people" id="resultWithService">
    		<id property="id" column="id"/>
    		<result property="name" column="name"/>
    		<result property="age" column="age"/>
    		<result property="sex" column="sex"/>
    		<collection property="services" ofType="service" column="id"
    	select="selectServiceByPeopleIdMapper.selectById"></collection>
    	</resultMap>
    	<select id="selectPeopleWithService" parameterType="people" resultMap="resultWithService">
    		select <include refid="baseMapper.str_select_people"></include>
    		from <include refid="baseMapper.str_table_people"></include>
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
    

加入mybatis.xml  
编写测试方法
    
    
    @Test
    	public void selectPeopleWithService(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		People people = new People();
    		people.setId(268L);
    		session.selectList("moreToMoreForPeopleMapper.selectPeopleWithService",
    				people).forEach(p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 558187323.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@2145433b]
    DEBUG [main] - ==>  Preparing: select p.id,p.name,p.age,p.sex from people p WHERE id=? 
    DEBUG [main] - ==> Parameters: 268(Long)
    DEBUG [main] - <==      Total: 1
    DEBUG [main] - ==>  Preparing: select s.id,s.name,s.remark from service s , people_service ps WHERE s.id = ps.service_id and ps.people_id=? 
    DEBUG [main] - ==> Parameters: 268(Long)
    DEBUG [main] - <==      Total: 201
    people [id=268,name=people4,age=5,sex=0,service=[[id=464,name=service0,remark=remark0,peoples=null], [id=665,name=service0,remark=remark0,peoples=null], [id=1,name=service1,remark=remark1,peoples=null], [id=2,name=service2,remark=remark2,peoples=null], [id=3,name=service3,remark=remark3,peoples=null], [id=4,name=service4,remark=remark4,peoples=null], [id=5,name=service5,remark=remark5,peoples=null], [id=6,name=service6,remark=remark6,peoples=null], [id=7,name=service7,remark=remark7,peoples=null], [id=8,name=service8,remark=remark8,peoples=null], [id=9,name=service9,remark=remark9,peoples=null], [id=10,name=service10,remark=remark10,peoples=null], [id=11,name=service11,remark=remark11,peoples=null], [id=12,name=service12,remark=remark12,peoples=null], [id=13,name=service13,remark=remark13,peoples=null], [id=14,name=service14,remark=remark14,peoples=null], [id=15,name=service15,remark=remark15,peoples=null], [id=16,name=service16,remark=remark16,peoples=null], [id=17,name=service17,remark=remark17,peoples=null], [id=18,name=service18,remark=remark18,peoples=null], [id=19,name=service19,remark=remark19,peoples=null], [id=20,name=service20,remark=remark20,peoples=null], [id=21,name=service21,remark=remark21,peoples=null], [id=22,name=service22,remark=remark22,peoples=null], [id=23,name=service23,remark=remark23,peoples=null], [id=24,name=service24,remark=remark24,peoples=null], [id=25,name=service25,remark=remark25,peoples=null], [id=26,name=service26,remark=remark26,peoples=null], [id=27,name=service27,remark=remark27,peoples=null], [id=28,name=service28,remark=remark28,peoples=null], [id=29,name=service29,remark=remark29,peoples=null], [id=30,name=service30,remark=remark30,peoples=null], [id=31,name=service31,remark=remark31,peoples=null], [id=32,name=service32,remark=remark32,peoples=null], [id=33,name=service33,remark=remark33,peoples=null], [id=34,name=service34,remark=remark34,peoples=null], [id=35,name=service35,remark=remark35,peoples=null], [id=36,name=service36,remark=remark36,peoples=null], [id=37,name=service37,remark=remark37,peoples=null], [id=38,name=service38,remark=remark38,peoples=null], [id=39,name=service39,remark=remark39,peoples=null], [id=40,name=service40,remark=remark40,peoples=null], [id=41,name=service41,remark=remark41,peoples=null], [id=42,name=service42,remark=remark42,peoples=null], [id=43,name=service43,remark=remark43,peoples=null], [id=44,name=service44,remark=remark44,peoples=null], [id=45,name=service45,remark=remark45,peoples=null], [id=46,name=service46,remark=remark46,peoples=null], [id=47,name=service47,remark=remark47,peoples=null], [id=48,name=service48,remark=remark48,peoples=null], [id=49,name=service49,remark=remark49,peoples=null], [id=50,name=service50,remark=remark50,peoples=null], [id=51,name=service51,remark=remark51,peoples=null], [id=52,name=service52,remark=remark52,peoples=null], [id=53,name=service53,remark=remark53,peoples=null], [id=54,name=service54,remark=remark54,peoples=null], [id=55,name=service55,remark=remark55,peoples=null], [id=56,name=service56,remark=remark56,peoples=null], [id=57,name=service57,remark=remark57,peoples=null], [id=58,name=service58,remark=remark58,peoples=null], [id=59,name=service59,remark=remark59,peoples=null], [id=60,name=service60,remark=remark60,peoples=null], [id=61,name=service61,remark=remark61,peoples=null], [id=62,name=service62,remark=remark62,peoples=null], [id=63,name=service63,remark=remark63,peoples=null], [id=64,name=service64,remark=remark64,peoples=null], [id=65,name=service65,remark=remark65,peoples=null], [id=66,name=service66,remark=remark66,peoples=null], [id=67,name=service67,remark=remark67,peoples=null], [id=68,name=service68,remark=remark68,peoples=null], [id=69,name=service69,remark=remark69,peoples=null], [id=70,name=service70,remark=remark70,peoples=null], [id=71,name=service71,remark=remark71,peoples=null], [id=72,name=service72,remark=remark72,peoples=null], [id=73,name=service73,remark=remark73,peoples=null], [id=74,name=service74,remark=remark74,peoples=null], [id=75,name=service75,remark=remark75,peoples=null], [id=76,name=service76,remark=remark76,peoples=null], [id=77,name=service77,remark=remark77,peoples=null], [id=78,name=service78,remark=remark78,peoples=null], [id=79,name=service79,remark=remark79,peoples=null], [id=80,name=service80,remark=remark80,peoples=null], [id=81,name=service81,remark=remark81,peoples=null], [id=82,name=service82,remark=remark82,peoples=null], [id=83,name=service83,remark=remark83,peoples=null], [id=84,name=service84,remark=remark84,peoples=null], [id=85,name=service85,remark=remark85,peoples=null], [id=86,name=service86,remark=remark86,peoples=null], [id=87,name=service87,remark=remark87,peoples=null], [id=88,name=service88,remark=remark88,peoples=null], [id=89,name=service89,remark=remark89,peoples=null], [id=90,name=service90,remark=remark90,peoples=null], [id=91,name=service91,remark=remark91,peoples=null], [id=92,name=service92,remark=remark92,peoples=null], [id=93,name=service93,remark=remark93,peoples=null], [id=94,name=service94,remark=remark94,peoples=null], [id=95,name=service95,remark=remark95,peoples=null], [id=96,name=service96,remark=remark96,peoples=null], [id=97,name=service97,remark=remark97,peoples=null], [id=98,name=service98,remark=remark98,peoples=null], [id=99,name=service99,remark=remark99,peoples=null], [id=100,name=service100,remark=remark100,peoples=null], [id=101,name=service101,remark=remark101,peoples=null], [id=102,name=service102,remark=remark102,peoples=null], [id=103,name=service103,remark=remark103,peoples=null], [id=104,name=service104,remark=remark104,peoples=null], [id=105,name=service105,remark=remark105,peoples=null], [id=106,name=service106,remark=remark106,peoples=null], [id=107,name=service107,remark=remark107,peoples=null], [id=108,name=service108,remark=remark108,peoples=null], [id=109,name=service109,remark=remark109,peoples=null], [id=110,name=service110,remark=remark110,peoples=null], [id=111,name=service111,remark=remark111,peoples=null], [id=112,name=service112,remark=remark112,peoples=null], [id=113,name=service113,remark=remark113,peoples=null], [id=114,name=service114,remark=remark114,peoples=null], [id=115,name=service115,remark=remark115,peoples=null], [id=116,name=service116,remark=remark116,peoples=null], [id=117,name=service117,remark=remark117,peoples=null], [id=118,name=service118,remark=remark118,peoples=null], [id=119,name=service119,remark=remark119,peoples=null], [id=120,name=service120,remark=remark120,peoples=null], [id=121,name=service121,remark=remark121,peoples=null], [id=122,name=service122,remark=remark122,peoples=null], [id=123,name=service123,remark=remark123,peoples=null], [id=124,name=service124,remark=remark124,peoples=null], [id=125,name=service125,remark=remark125,peoples=null], [id=126,name=service126,remark=remark126,peoples=null], [id=127,name=service127,remark=remark127,peoples=null], [id=128,name=service128,remark=remark128,peoples=null], [id=129,name=service129,remark=remark129,peoples=null], [id=130,name=service130,remark=remark130,peoples=null], [id=131,name=service131,remark=remark131,peoples=null], [id=132,name=service132,remark=remark132,peoples=null], [id=133,name=service133,remark=remark133,peoples=null], [id=134,name=service134,remark=remark134,peoples=null], [id=135,name=service135,remark=remark135,peoples=null], [id=136,name=service136,remark=remark136,peoples=null], [id=137,name=service137,remark=remark137,peoples=null], [id=138,name=service138,remark=remark138,peoples=null], [id=139,name=service139,remark=remark139,peoples=null], [id=140,name=service140,remark=remark140,peoples=null], [id=141,name=service141,remark=remark141,peoples=null], [id=142,name=service142,remark=remark142,peoples=null], [id=143,name=service143,remark=remark143,peoples=null], [id=144,name=service144,remark=remark144,peoples=null], [id=145,name=service145,remark=remark145,peoples=null], [id=146,name=service146,remark=remark146,peoples=null], [id=147,name=service147,remark=remark147,peoples=null], [id=148,name=service148,remark=remark148,peoples=null], [id=149,name=service149,remark=remark149,peoples=null], [id=150,name=service150,remark=remark150,peoples=null], [id=151,name=service151,remark=remark151,peoples=null], [id=152,name=service152,remark=remark152,peoples=null], [id=153,name=service153,remark=remark153,peoples=null], [id=154,name=service154,remark=remark154,peoples=null], [id=155,name=service155,remark=remark155,peoples=null], [id=156,name=service156,remark=remark156,peoples=null], [id=157,name=service157,remark=remark157,peoples=null], [id=158,name=service158,remark=remark158,peoples=null], [id=159,name=service159,remark=remark159,peoples=null], [id=160,name=service160,remark=remark160,peoples=null], [id=161,name=service161,remark=remark161,peoples=null], [id=162,name=service162,remark=remark162,peoples=null], [id=163,name=service163,remark=remark163,peoples=null], [id=164,name=service164,remark=remark164,peoples=null], [id=165,name=service165,remark=remark165,peoples=null], [id=166,name=service166,remark=remark166,peoples=null], [id=167,name=service167,remark=remark167,peoples=null], [id=168,name=service168,remark=remark168,peoples=null], [id=169,name=service169,remark=remark169,peoples=null], [id=170,name=service170,remark=remark170,peoples=null], [id=171,name=service171,remark=remark171,peoples=null], [id=172,name=service172,remark=remark172,peoples=null], [id=173,name=service173,remark=remark173,peoples=null], [id=174,name=service174,remark=remark174,peoples=null], [id=175,name=service175,remark=remark175,peoples=null], [id=176,name=service176,remark=remark176,peoples=null], [id=177,name=service177,remark=remark177,peoples=null], [id=178,name=service178,remark=remark178,peoples=null], [id=179,name=service179,remark=remark179,peoples=null], [id=180,name=service180,remark=remark180,peoples=null], [id=181,name=service181,remark=remark181,peoples=null], [id=182,name=service182,remark=remark182,peoples=null], [id=183,name=service183,remark=remark183,peoples=null], [id=184,name=service184,remark=remark184,peoples=null], [id=185,name=service185,remark=remark185,peoples=null], [id=186,name=service186,remark=remark186,peoples=null], [id=187,name=service187,remark=remark187,peoples=null], [id=188,name=service188,remark=remark188,peoples=null], [id=189,name=service189,remark=remark189,peoples=null], [id=190,name=service190,remark=remark190,peoples=null], [id=191,name=service191,remark=remark191,peoples=null], [id=192,name=service192,remark=remark192,peoples=null], [id=193,name=service193,remark=remark193,peoples=null], [id=194,name=service194,remark=remark194,peoples=null], [id=195,name=service195,remark=remark195,peoples=null], [id=196,name=service196,remark=remark196,peoples=null], [id=197,name=service197,remark=remark197,peoples=null], [id=198,name=service198,remark=remark198,peoples=null], [id=199,name=service199,remark=remark199,peoples=null]]]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@2145433b]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@2145433b]
    DEBUG [main] - Returned connection 558187323 to pool.
    
    

#### 3.4.2多对多关系people-service结果方式
    
    
    <!-- 结果方式 -->
    	<resultMap type="people" id="resultPeopleWithService">
    		<id property="id" column="pid"/>
    		<result property="name" column="pname"/>
    		<result property="age" column="page"/>
    		<result property="sex" column="psex"/>
    		<collection property="services" ofType="service">
    			<id property="id" column="sid"/>
    			<result property="name" column="sname"/>
    			<result property="remark" column="sremark"/>
    		</collection>
    	</resultMap>
    	<select id="selectPeopleWithServiceResult" parameterType="people" resultMap="resultPeopleWithService">
    		select p.id as pid,p.name as pname,p.age as page,p.sex as psex,
    		s.id as sid,s.name as sname,s.remark as sremark
    		from <include refid="baseMapper.str_table_people"></include>,
    		<include refid="baseMapper.str_table_service"></include>,
    		<include refid="baseMapper.str_table_people_service"></include>
    		<where>
    			and p.id=ps.people_id
    			and s.id=ps.service_id
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
    

编写测试方法
    
    
    @Test
    	public void selectPeopleWithServiceResult(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		People people = new People();
    		people.setId(268L);
    		session.selectList(
    				"moreToMoreForPeopleMapper.selectPeopleWithServiceResult",
    				people).forEach(p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 558187323.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@2145433b]
    DEBUG [main] - ==>  Preparing: select p.id as pid,p.name as pname,p.age as page,p.sex as psex, s.id as sid,s.name as sname,s.remark as sremark from people p , service s , people_service ps WHERE p.id=ps.people_id and s.id=ps.service_id and p.id=? 
    DEBUG [main] - ==> Parameters: 268(Long)
    DEBUG [main] - <==      Total: 201
    people [id=268,name=people4,age=5,sex=0,service=[[id=464,name=service0,remark=remark0,peoples=null], [id=665,name=service0,remark=remark0,peoples=null], [id=1,name=service1,remark=remark1,peoples=null], [id=2,name=service2,remark=remark2,peoples=null], [id=3,name=service3,remark=remark3,peoples=null], [id=4,name=service4,remark=remark4,peoples=null], [id=5,name=service5,remark=remark5,peoples=null], [id=6,name=service6,remark=remark6,peoples=null], [id=7,name=service7,remark=remark7,peoples=null], [id=8,name=service8,remark=remark8,peoples=null], [id=9,name=service9,remark=remark9,peoples=null], [id=10,name=service10,remark=remark10,peoples=null], [id=11,name=service11,remark=remark11,peoples=null], [id=12,name=service12,remark=remark12,peoples=null], [id=13,name=service13,remark=remark13,peoples=null], [id=14,name=service14,remark=remark14,peoples=null], [id=15,name=service15,remark=remark15,peoples=null], [id=16,name=service16,remark=remark16,peoples=null], [id=17,name=service17,remark=remark17,peoples=null], [id=18,name=service18,remark=remark18,peoples=null], [id=19,name=service19,remark=remark19,peoples=null], [id=20,name=service20,remark=remark20,peoples=null], [id=21,name=service21,remark=remark21,peoples=null], [id=22,name=service22,remark=remark22,peoples=null], [id=23,name=service23,remark=remark23,peoples=null], [id=24,name=service24,remark=remark24,peoples=null], [id=25,name=service25,remark=remark25,peoples=null], [id=26,name=service26,remark=remark26,peoples=null], [id=27,name=service27,remark=remark27,peoples=null], [id=28,name=service28,remark=remark28,peoples=null], [id=29,name=service29,remark=remark29,peoples=null], [id=30,name=service30,remark=remark30,peoples=null], [id=31,name=service31,remark=remark31,peoples=null], [id=32,name=service32,remark=remark32,peoples=null], [id=33,name=service33,remark=remark33,peoples=null], [id=34,name=service34,remark=remark34,peoples=null], [id=35,name=service35,remark=remark35,peoples=null], [id=36,name=service36,remark=remark36,peoples=null], [id=37,name=service37,remark=remark37,peoples=null], [id=38,name=service38,remark=remark38,peoples=null], [id=39,name=service39,remark=remark39,peoples=null], [id=40,name=service40,remark=remark40,peoples=null], [id=41,name=service41,remark=remark41,peoples=null], [id=42,name=service42,remark=remark42,peoples=null], [id=43,name=service43,remark=remark43,peoples=null], [id=44,name=service44,remark=remark44,peoples=null], [id=45,name=service45,remark=remark45,peoples=null], [id=46,name=service46,remark=remark46,peoples=null], [id=47,name=service47,remark=remark47,peoples=null], [id=48,name=service48,remark=remark48,peoples=null], [id=49,name=service49,remark=remark49,peoples=null], [id=50,name=service50,remark=remark50,peoples=null], [id=51,name=service51,remark=remark51,peoples=null], [id=52,name=service52,remark=remark52,peoples=null], [id=53,name=service53,remark=remark53,peoples=null], [id=54,name=service54,remark=remark54,peoples=null], [id=55,name=service55,remark=remark55,peoples=null], [id=56,name=service56,remark=remark56,peoples=null], [id=57,name=service57,remark=remark57,peoples=null], [id=58,name=service58,remark=remark58,peoples=null], [id=59,name=service59,remark=remark59,peoples=null], [id=60,name=service60,remark=remark60,peoples=null], [id=61,name=service61,remark=remark61,peoples=null], [id=62,name=service62,remark=remark62,peoples=null], [id=63,name=service63,remark=remark63,peoples=null], [id=64,name=service64,remark=remark64,peoples=null], [id=65,name=service65,remark=remark65,peoples=null], [id=66,name=service66,remark=remark66,peoples=null], [id=67,name=service67,remark=remark67,peoples=null], [id=68,name=service68,remark=remark68,peoples=null], [id=69,name=service69,remark=remark69,peoples=null], [id=70,name=service70,remark=remark70,peoples=null], [id=71,name=service71,remark=remark71,peoples=null], [id=72,name=service72,remark=remark72,peoples=null], [id=73,name=service73,remark=remark73,peoples=null], [id=74,name=service74,remark=remark74,peoples=null], [id=75,name=service75,remark=remark75,peoples=null], [id=76,name=service76,remark=remark76,peoples=null], [id=77,name=service77,remark=remark77,peoples=null], [id=78,name=service78,remark=remark78,peoples=null], [id=79,name=service79,remark=remark79,peoples=null], [id=80,name=service80,remark=remark80,peoples=null], [id=81,name=service81,remark=remark81,peoples=null], [id=82,name=service82,remark=remark82,peoples=null], [id=83,name=service83,remark=remark83,peoples=null], [id=84,name=service84,remark=remark84,peoples=null], [id=85,name=service85,remark=remark85,peoples=null], [id=86,name=service86,remark=remark86,peoples=null], [id=87,name=service87,remark=remark87,peoples=null], [id=88,name=service88,remark=remark88,peoples=null], [id=89,name=service89,remark=remark89,peoples=null], [id=90,name=service90,remark=remark90,peoples=null], [id=91,name=service91,remark=remark91,peoples=null], [id=92,name=service92,remark=remark92,peoples=null], [id=93,name=service93,remark=remark93,peoples=null], [id=94,name=service94,remark=remark94,peoples=null], [id=95,name=service95,remark=remark95,peoples=null], [id=96,name=service96,remark=remark96,peoples=null], [id=97,name=service97,remark=remark97,peoples=null], [id=98,name=service98,remark=remark98,peoples=null], [id=99,name=service99,remark=remark99,peoples=null], [id=100,name=service100,remark=remark100,peoples=null], [id=101,name=service101,remark=remark101,peoples=null], [id=102,name=service102,remark=remark102,peoples=null], [id=103,name=service103,remark=remark103,peoples=null], [id=104,name=service104,remark=remark104,peoples=null], [id=105,name=service105,remark=remark105,peoples=null], [id=106,name=service106,remark=remark106,peoples=null], [id=107,name=service107,remark=remark107,peoples=null], [id=108,name=service108,remark=remark108,peoples=null], [id=109,name=service109,remark=remark109,peoples=null], [id=110,name=service110,remark=remark110,peoples=null], [id=111,name=service111,remark=remark111,peoples=null], [id=112,name=service112,remark=remark112,peoples=null], [id=113,name=service113,remark=remark113,peoples=null], [id=114,name=service114,remark=remark114,peoples=null], [id=115,name=service115,remark=remark115,peoples=null], [id=116,name=service116,remark=remark116,peoples=null], [id=117,name=service117,remark=remark117,peoples=null], [id=118,name=service118,remark=remark118,peoples=null], [id=119,name=service119,remark=remark119,peoples=null], [id=120,name=service120,remark=remark120,peoples=null], [id=121,name=service121,remark=remark121,peoples=null], [id=122,name=service122,remark=remark122,peoples=null], [id=123,name=service123,remark=remark123,peoples=null], [id=124,name=service124,remark=remark124,peoples=null], [id=125,name=service125,remark=remark125,peoples=null], [id=126,name=service126,remark=remark126,peoples=null], [id=127,name=service127,remark=remark127,peoples=null], [id=128,name=service128,remark=remark128,peoples=null], [id=129,name=service129,remark=remark129,peoples=null], [id=130,name=service130,remark=remark130,peoples=null], [id=131,name=service131,remark=remark131,peoples=null], [id=132,name=service132,remark=remark132,peoples=null], [id=133,name=service133,remark=remark133,peoples=null], [id=134,name=service134,remark=remark134,peoples=null], [id=135,name=service135,remark=remark135,peoples=null], [id=136,name=service136,remark=remark136,peoples=null], [id=137,name=service137,remark=remark137,peoples=null], [id=138,name=service138,remark=remark138,peoples=null], [id=139,name=service139,remark=remark139,peoples=null], [id=140,name=service140,remark=remark140,peoples=null], [id=141,name=service141,remark=remark141,peoples=null], [id=142,name=service142,remark=remark142,peoples=null], [id=143,name=service143,remark=remark143,peoples=null], [id=144,name=service144,remark=remark144,peoples=null], [id=145,name=service145,remark=remark145,peoples=null], [id=146,name=service146,remark=remark146,peoples=null], [id=147,name=service147,remark=remark147,peoples=null], [id=148,name=service148,remark=remark148,peoples=null], [id=149,name=service149,remark=remark149,peoples=null], [id=150,name=service150,remark=remark150,peoples=null], [id=151,name=service151,remark=remark151,peoples=null], [id=152,name=service152,remark=remark152,peoples=null], [id=153,name=service153,remark=remark153,peoples=null], [id=154,name=service154,remark=remark154,peoples=null], [id=155,name=service155,remark=remark155,peoples=null], [id=156,name=service156,remark=remark156,peoples=null], [id=157,name=service157,remark=remark157,peoples=null], [id=158,name=service158,remark=remark158,peoples=null], [id=159,name=service159,remark=remark159,peoples=null], [id=160,name=service160,remark=remark160,peoples=null], [id=161,name=service161,remark=remark161,peoples=null], [id=162,name=service162,remark=remark162,peoples=null], [id=163,name=service163,remark=remark163,peoples=null], [id=164,name=service164,remark=remark164,peoples=null], [id=165,name=service165,remark=remark165,peoples=null], [id=166,name=service166,remark=remark166,peoples=null], [id=167,name=service167,remark=remark167,peoples=null], [id=168,name=service168,remark=remark168,peoples=null], [id=169,name=service169,remark=remark169,peoples=null], [id=170,name=service170,remark=remark170,peoples=null], [id=171,name=service171,remark=remark171,peoples=null], [id=172,name=service172,remark=remark172,peoples=null], [id=173,name=service173,remark=remark173,peoples=null], [id=174,name=service174,remark=remark174,peoples=null], [id=175,name=service175,remark=remark175,peoples=null], [id=176,name=service176,remark=remark176,peoples=null], [id=177,name=service177,remark=remark177,peoples=null], [id=178,name=service178,remark=remark178,peoples=null], [id=179,name=service179,remark=remark179,peoples=null], [id=180,name=service180,remark=remark180,peoples=null], [id=181,name=service181,remark=remark181,peoples=null], [id=182,name=service182,remark=remark182,peoples=null], [id=183,name=service183,remark=remark183,peoples=null], [id=184,name=service184,remark=remark184,peoples=null], [id=185,name=service185,remark=remark185,peoples=null], [id=186,name=service186,remark=remark186,peoples=null], [id=187,name=service187,remark=remark187,peoples=null], [id=188,name=service188,remark=remark188,peoples=null], [id=189,name=service189,remark=remark189,peoples=null], [id=190,name=service190,remark=remark190,peoples=null], [id=191,name=service191,remark=remark191,peoples=null], [id=192,name=service192,remark=remark192,peoples=null], [id=193,name=service193,remark=remark193,peoples=null], [id=194,name=service194,remark=remark194,peoples=null], [id=195,name=service195,remark=remark195,peoples=null], [id=196,name=service196,remark=remark196,peoples=null], [id=197,name=service197,remark=remark197,peoples=null], [id=198,name=service198,remark=remark198,peoples=null], [id=199,name=service199,remark=remark199,peoples=null]]]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@2145433b]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@2145433b]
    DEBUG [main] - Returned connection 558187323 to pool.
    
    

#### 3.4.3多对多关系service-people查询方式

moreToMoreForServiceMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="moreToMoreForServiceMapper">
    	<resultMap type="service" id="resultWithPeople">
    		<id property="id" column="id"/>
    		<result property="name" column="name"/>
    		<result property="remark" column="remark"/>
    		<collection property="peoples" ofType="people" column="id"
    	select="selectPeopleByServiceIdMapper.selectById"></collection>
    	</resultMap>
    	<select id="selectServiceWithPeople" parameterType="service" resultMap="resultWithPeople">
    		select <include refid="baseMapper.str_select_service"></include>
    		from <include refid="baseMapper.str_table_service"></include>
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
    </mapper>
    

加入到mybatis.xml  
编写测试方法：
    
    
    @Test
    	public void selectServiceWithPeople(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		Service service = new Service();
    		service.setId(128L);
    		session.selectList(
    				"moreToMoreForServiceMapper.selectServiceWithPeople", service)
    				.forEach(p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 453523494.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@1b083826]
    DEBUG [main] - ==>  Preparing: select s.id,s.name,s.remark from service s WHERE s.id=? 
    DEBUG [main] - ==> Parameters: 128(Long)
    DEBUG [main] - <==      Total: 1
    DEBUG [main] - ==>  Preparing: select p.id,p.name,p.age,p.sex from people p , people_service ps WHERE p.id = ps.people_id and ps.service_id=? 
    DEBUG [main] - ==> Parameters: 128(Long)
    DEBUG [main] - <==      Total: 402
    [id=128,name=service128,remark=remark128,peoples=[people [id=268,name=people4,age=5,sex=0,service=null], people [id=269,name=people5,age=6,sex=1,service=null], people [id=270,name=people6,age=7,sex=0,service=null], people [id=271,name=people7,age=8,sex=1,service=null], people [id=272,name=people8,age=9,sex=0,service=null], people [id=273,name=people9,age=10,sex=1,service=null], people [id=274,name=people10,age=11,sex=0,service=null], people [id=275,name=people11,age=12,sex=1,service=null], people [id=141,name=people0,age=22,sex=1,service=null], people [id=142,name=people0,age=22,sex=1,service=null], people [id=143,name=people0,age=22,sex=1,service=null], people [id=265,name=people1,age=2,sex=1,service=null], people [id=266,name=people2,age=3,sex=0,service=null], people [id=267,name=people3,age=4,sex=1,service=null], people [id=288,name=people24,age=25,sex=0,service=null], people [id=289,name=people25,age=26,sex=1,service=null], people [id=290,name=people26,age=27,sex=0,service=null], people [id=291,name=people27,age=28,sex=1,service=null], people [id=292,name=people28,age=29,sex=0,service=null], people [id=293,name=people29,age=30,sex=1,service=null], people [id=294,name=people30,age=31,sex=0,service=null], people [id=295,name=people31,age=32,sex=1,service=null], people [id=296,name=people32,age=33,sex=0,service=null], people [id=297,name=people33,age=34,sex=1,service=null], people [id=298,name=people34,age=35,sex=0,service=null], people [id=276,name=people12,age=13,sex=0,service=null], people [id=277,name=people13,age=14,sex=1,service=null], people [id=278,name=people14,age=15,sex=0,service=null], people [id=279,name=people15,age=16,sex=1,service=null], people [id=280,name=people16,age=17,sex=0,service=null], people [id=281,name=people17,age=18,sex=1,service=null], people [id=282,name=people18,age=19,sex=0,service=null], people [id=283,name=people19,age=20,sex=1,service=null], people [id=284,name=people20,age=21,sex=0,service=null], people [id=285,name=people21,age=22,sex=1,service=null], people [id=286,name=people22,age=23,sex=0,service=null], people [id=287,name=people23,age=24,sex=1,service=null], people [id=299,name=people35,age=36,sex=1,service=null], people [id=300,name=people36,age=37,sex=0,service=null], people [id=301,name=people37,age=38,sex=1,service=null], people [id=308,name=people44,age=45,sex=0,service=null], people [id=309,name=people45,age=46,sex=1,service=null], people [id=310,name=people46,age=47,sex=0,service=null], people [id=311,name=people47,age=48,sex=1,service=null], people [id=312,name=people48,age=49,sex=0,service=null], people [id=313,name=people49,age=50,sex=1,service=null], people [id=314,name=people50,age=51,sex=0,service=null], people [id=315,name=people51,age=52,sex=1,service=null], people [id=316,name=people52,age=53,sex=0,service=null], people [id=317,name=people53,age=54,sex=1,service=null], people [id=318,name=people54,age=55,sex=0,service=null], people [id=319,name=people55,age=56,sex=1,service=null], people [id=302,name=people38,age=39,sex=0,service=null], people [id=303,name=people39,age=40,sex=1,service=null], people [id=304,name=people40,age=41,sex=0,service=null], people [id=305,name=people41,age=42,sex=1,service=null], people [id=306,name=people42,age=43,sex=0,service=null], people [id=307,name=people43,age=44,sex=1,service=null], people [id=331,name=people67,age=68,sex=1,service=null], people [id=332,name=people68,age=69,sex=0,service=null], people [id=333,name=people69,age=70,sex=1,service=null], people [id=334,name=people70,age=71,sex=0,service=null], people [id=335,name=people71,age=72,sex=1,service=null], people [id=336,name=people72,age=73,sex=0,service=null], people [id=337,name=people73,age=74,sex=1,service=null], people [id=338,name=people74,age=75,sex=0,service=null], people [id=339,name=people75,age=76,sex=1,service=null], people [id=340,name=people76,age=77,sex=0,service=null], people [id=341,name=people77,age=78,sex=1,service=null], people [id=342,name=people78,age=79,sex=0,service=null], people [id=320,name=people56,age=57,sex=0,service=null], people [id=321,name=people57,age=58,sex=1,service=null], people [id=322,name=people58,age=59,sex=0,service=null], people [id=323,name=people59,age=60,sex=1,service=null], people [id=324,name=people60,age=61,sex=0,service=null], people [id=325,name=people61,age=62,sex=1,service=null], people [id=326,name=people62,age=63,sex=0,service=null], people [id=327,name=people63,age=64,sex=1,service=null], people [id=328,name=people64,age=65,sex=0,service=null], people [id=329,name=people65,age=66,sex=1,service=null], people [id=330,name=people66,age=67,sex=0,service=null], people [id=343,name=people79,age=80,sex=1,service=null], people [id=344,name=people80,age=81,sex=0,service=null], people [id=345,name=people81,age=82,sex=1,service=null], people [id=351,name=people87,age=88,sex=1,service=null], people [id=352,name=people88,age=89,sex=0,service=null], people [id=353,name=people89,age=90,sex=1,service=null], people [id=354,name=people90,age=91,sex=0,service=null], people [id=355,name=people91,age=92,sex=1,service=null], people [id=356,name=people92,age=93,sex=0,service=null], people [id=357,name=people93,age=94,sex=1,service=null], people [id=358,name=people94,age=95,sex=0,service=null], people [id=359,name=people95,age=96,sex=1,service=null], people [id=360,name=people96,age=97,sex=0,service=null], people [id=361,name=people97,age=98,sex=1,service=null], people [id=362,name=people98,age=99,sex=0,service=null], people [id=346,name=people82,age=83,sex=0,service=null], people [id=347,name=people83,age=84,sex=1,service=null], people [id=348,name=people84,age=85,sex=0,service=null], people [id=349,name=people85,age=86,sex=1,service=null], people [id=350,name=people86,age=87,sex=0,service=null], people [id=375,name=people111,age=112,sex=1,service=null], people [id=376,name=people112,age=113,sex=0,service=null], people [id=377,name=people113,age=114,sex=1,service=null], people [id=378,name=people114,age=115,sex=0,service=null], people [id=379,name=people115,age=116,sex=1,service=null], people [id=380,name=people116,age=117,sex=0,service=null], people [id=381,name=people117,age=118,sex=1,service=null], people [id=382,name=people118,age=119,sex=0,service=null], people [id=383,name=people119,age=120,sex=1,service=null], people [id=384,name=people120,age=1,sex=0,service=null], people [id=385,name=people121,age=2,sex=1,service=null], people [id=363,name=people99,age=100,sex=1,service=null], people [id=364,name=people100,age=101,sex=0,service=null], people [id=365,name=people101,age=102,sex=1,service=null], people [id=366,name=people102,age=103,sex=0,service=null], people [id=367,name=people103,age=104,sex=1,service=null], people [id=368,name=people104,age=105,sex=0,service=null], people [id=369,name=people105,age=106,sex=1,service=null], people [id=370,name=people106,age=107,sex=0,service=null], people [id=371,name=people107,age=108,sex=1,service=null], people [id=372,name=people108,age=109,sex=0,service=null], people [id=373,name=people109,age=110,sex=1,service=null], people [id=374,name=people110,age=111,sex=0,service=null], people [id=386,name=people122,age=3,sex=0,service=null], people [id=387,name=people123,age=4,sex=1,service=null], people [id=388,name=people124,age=5,sex=0,service=null], people [id=395,name=people131,age=12,sex=1,service=null], people [id=396,name=people132,age=13,sex=0,service=null], people [id=397,name=people133,age=14,sex=1,service=null], people [id=398,name=people134,age=15,sex=0,service=null], people [id=399,name=people135,age=16,sex=1,service=null], people [id=400,name=people136,age=17,sex=0,service=null], people [id=401,name=people137,age=18,sex=1,service=null], people [id=402,name=people138,age=19,sex=0,service=null], people [id=403,name=people139,age=20,sex=1,service=null], people [id=404,name=people140,age=21,sex=0,service=null], people [id=405,name=people141,age=22,sex=1,service=null], people [id=389,name=people125,age=6,sex=1,service=null], people [id=390,name=people126,age=7,sex=0,service=null], people [id=391,name=people127,age=8,sex=1,service=null], people [id=392,name=people128,age=9,sex=0,service=null], people [id=393,name=people129,age=10,sex=1,service=null], people [id=394,name=people130,age=11,sex=0,service=null], people [id=418,name=people154,age=35,sex=0,service=null], people [id=419,name=people155,age=36,sex=1,service=null], people [id=420,name=people156,age=37,sex=0,service=null], people [id=421,name=people157,age=38,sex=1,service=null], people [id=422,name=people158,age=39,sex=0,service=null], people [id=423,name=people159,age=40,sex=1,service=null], people [id=424,name=people160,age=41,sex=0,service=null], people [id=425,name=people161,age=42,sex=1,service=null], people [id=426,name=people162,age=43,sex=0,service=null], people [id=427,name=people163,age=44,sex=1,service=null], people [id=428,name=people164,age=45,sex=0,service=null], people [id=429,name=people165,age=46,sex=1,service=null], people [id=406,name=people142,age=23,sex=0,service=null], people [id=407,name=people143,age=24,sex=1,service=null], people [id=408,name=people144,age=25,sex=0,service=null], people [id=409,name=people145,age=26,sex=1,service=null], people [id=410,name=people146,age=27,sex=0,service=null], people [id=411,name=people147,age=28,sex=1,service=null], people [id=412,name=people148,age=29,sex=0,service=null], people [id=413,name=people149,age=30,sex=1,service=null], people [id=414,name=people150,age=31,sex=0,service=null], people [id=415,name=people151,age=32,sex=1,service=null], people [id=416,name=people152,age=33,sex=0,service=null], people [id=417,name=people153,age=34,sex=1,service=null], people [id=430,name=people166,age=47,sex=0,service=null], people [id=431,name=people167,age=48,sex=1,service=null], people [id=438,name=people174,age=55,sex=0,service=null], people [id=439,name=people175,age=56,sex=1,service=null], people [id=440,name=people176,age=57,sex=0,service=null], people [id=441,name=people177,age=58,sex=1,service=null], people [id=442,name=people178,age=59,sex=0,service=null], people [id=443,name=people179,age=60,sex=1,service=null], people [id=444,name=people180,age=61,sex=0,service=null], people [id=445,name=people181,age=62,sex=1,service=null], people [id=446,name=people182,age=63,sex=0,service=null], people [id=447,name=people183,age=64,sex=1,service=null], people [id=448,name=people184,age=65,sex=0,service=null], people [id=449,name=people185,age=66,sex=1,service=null], people [id=432,name=people168,age=49,sex=0,service=null], people [id=433,name=people169,age=50,sex=1,service=null], people [id=434,name=people170,age=51,sex=0,service=null], people [id=435,name=people171,age=52,sex=1,service=null], people [id=436,name=people172,age=53,sex=0,service=null], people [id=437,name=people173,age=54,sex=1,service=null], people [id=461,name=people197,age=78,sex=1,service=null], people [id=462,name=people198,age=79,sex=0,service=null], people [id=463,name=people199,age=80,sex=1,service=null], people [id=465,name=people1,age=2,sex=1,service=null], people [id=466,name=people2,age=3,sex=0,service=null], people [id=467,name=people3,age=4,sex=1,service=null], people [id=468,name=people4,age=5,sex=0,service=null], people [id=469,name=people5,age=6,sex=1,service=null], people [id=470,name=people6,age=7,sex=0,service=null], people [id=471,name=people7,age=8,sex=1,service=null], people [id=472,name=people8,age=9,sex=0,service=null], people [id=473,name=people9,age=10,sex=1,service=null], people [id=450,name=people186,age=67,sex=0,service=null], people [id=451,name=people187,age=68,sex=1,service=null], people [id=452,name=people188,age=69,sex=0,service=null], people [id=453,name=people189,age=70,sex=1,service=null], people [id=454,name=people190,age=71,sex=0,service=null], people [id=455,name=people191,age=72,sex=1,service=null], people [id=456,name=people192,age=73,sex=0,service=null], people [id=457,name=people193,age=74,sex=1,service=null], people [id=458,name=people194,age=75,sex=0,service=null], people [id=459,name=people195,age=76,sex=1,service=null], people [id=460,name=people196,age=77,sex=0,service=null], people [id=474,name=people10,age=11,sex=0,service=null], people [id=475,name=people11,age=12,sex=1,service=null], people [id=476,name=people12,age=13,sex=0,service=null], people [id=483,name=people19,age=20,sex=1,service=null], people [id=484,name=people20,age=21,sex=0,service=null], people [id=485,name=people21,age=22,sex=1,service=null], people [id=486,name=people22,age=23,sex=0,service=null], people [id=487,name=people23,age=24,sex=1,service=null], people [id=488,name=people24,age=25,sex=0,service=null], people [id=489,name=people25,age=26,sex=1,service=null], people [id=490,name=people26,age=27,sex=0,service=null], people [id=491,name=people27,age=28,sex=1,service=null], people [id=492,name=people28,age=29,sex=0,service=null], people [id=493,name=people29,age=30,sex=1,service=null], people [id=477,name=people13,age=14,sex=1,service=null], people [id=478,name=people14,age=15,sex=0,service=null], people [id=479,name=people15,age=16,sex=1,service=null], people [id=480,name=people16,age=17,sex=0,service=null], people [id=481,name=people17,age=18,sex=1,service=null], people [id=482,name=people18,age=19,sex=0,service=null], people [id=506,name=people42,age=43,sex=0,service=null], people [id=507,name=people43,age=44,sex=1,service=null], people [id=508,name=people44,age=45,sex=0,service=null], people [id=509,name=people45,age=46,sex=1,service=null], people [id=510,name=people46,age=47,sex=0,service=null], people [id=511,name=people47,age=48,sex=1,service=null], people [id=512,name=people48,age=49,sex=0,service=null], people [id=513,name=people49,age=50,sex=1,service=null], people [id=514,name=people50,age=51,sex=0,service=null], people [id=515,name=people51,age=52,sex=1,service=null], people [id=516,name=people52,age=53,sex=0,service=null], people [id=494,name=people30,age=31,sex=0,service=null], people [id=495,name=people31,age=32,sex=1,service=null], people [id=496,name=people32,age=33,sex=0,service=null], people [id=497,name=people33,age=34,sex=1,service=null], people [id=498,name=people34,age=35,sex=0,service=null], people [id=499,name=people35,age=36,sex=1,service=null], people [id=500,name=people36,age=37,sex=0,service=null], people [id=501,name=people37,age=38,sex=1,service=null], people [id=502,name=people38,age=39,sex=0,service=null], people [id=503,name=people39,age=40,sex=1,service=null], people [id=504,name=people40,age=41,sex=0,service=null], people [id=505,name=people41,age=42,sex=1,service=null], people [id=517,name=people53,age=54,sex=1,service=null], people [id=518,name=people54,age=55,sex=0,service=null], people [id=519,name=people55,age=56,sex=1,service=null], people [id=526,name=people62,age=63,sex=0,service=null], people [id=527,name=people63,age=64,sex=1,service=null], people [id=528,name=people64,age=65,sex=0,service=null], people [id=529,name=people65,age=66,sex=1,service=null], people [id=530,name=people66,age=67,sex=0,service=null], people [id=531,name=people67,age=68,sex=1,service=null], people [id=532,name=people68,age=69,sex=0,service=null], people [id=533,name=people69,age=70,sex=1,service=null], people [id=534,name=people70,age=71,sex=0,service=null], people [id=535,name=people71,age=72,sex=1,service=null], people [id=536,name=people72,age=73,sex=0,service=null], people [id=520,name=people56,age=57,sex=0,service=null], people [id=521,name=people57,age=58,sex=1,service=null], people [id=522,name=people58,age=59,sex=0,service=null], people [id=523,name=people59,age=60,sex=1,service=null], people [id=524,name=people60,age=61,sex=0,service=null], people [id=525,name=people61,age=62,sex=1,service=null], people [id=549,name=people85,age=86,sex=1,service=null], people [id=550,name=people86,age=87,sex=0,service=null], people [id=551,name=people87,age=88,sex=1,service=null], people [id=552,name=people88,age=89,sex=0,service=null], people [id=553,name=people89,age=90,sex=1,service=null], people [id=554,name=people90,age=91,sex=0,service=null], people [id=555,name=people91,age=92,sex=1,service=null], people [id=556,name=people92,age=93,sex=0,service=null], people [id=557,name=people93,age=94,sex=1,service=null], people [id=558,name=people94,age=95,sex=0,service=null], people [id=559,name=people95,age=96,sex=1,service=null], people [id=560,name=people96,age=97,sex=0,service=null], people [id=537,name=people73,age=74,sex=1,service=null], people [id=538,name=people74,age=75,sex=0,service=null], people [id=539,name=people75,age=76,sex=1,service=null], people [id=540,name=people76,age=77,sex=0,service=null], people [id=541,name=people77,age=78,sex=1,service=null], people [id=542,name=people78,age=79,sex=0,service=null], people [id=543,name=people79,age=80,sex=1,service=null], people [id=544,name=people80,age=81,sex=0,service=null], people [id=545,name=people81,age=82,sex=1,service=null], people [id=546,name=people82,age=83,sex=0,service=null], people [id=547,name=people83,age=84,sex=1,service=null], people [id=548,name=people84,age=85,sex=0,service=null], people [id=561,name=people97,age=98,sex=1,service=null], people [id=562,name=people98,age=99,sex=0,service=null], people [id=569,name=people105,age=106,sex=1,service=null], people [id=570,name=people106,age=107,sex=0,service=null], people [id=571,name=people107,age=108,sex=1,service=null], people [id=572,name=people108,age=109,sex=0,service=null], people [id=573,name=people109,age=110,sex=1,service=null], people [id=574,name=people110,age=111,sex=0,service=null], people [id=575,name=people111,age=112,sex=1,service=null], people [id=576,name=people112,age=113,sex=0,service=null], people [id=577,name=people113,age=114,sex=1,service=null], people [id=578,name=people114,age=115,sex=0,service=null], people [id=579,name=people115,age=116,sex=1,service=null], people [id=580,name=people116,age=117,sex=0,service=null], people [id=563,name=people99,age=100,sex=1,service=null], people [id=564,name=people100,age=101,sex=0,service=null], people [id=565,name=people101,age=102,sex=1,service=null], people [id=566,name=people102,age=103,sex=0,service=null], people [id=567,name=people103,age=104,sex=1,service=null], people [id=568,name=people104,age=105,sex=0,service=null], people [id=592,name=people128,age=9,sex=0,service=null], people [id=593,name=people129,age=10,sex=1,service=null], people [id=594,name=people130,age=11,sex=0,service=null], people [id=595,name=people131,age=12,sex=1,service=null], people [id=596,name=people132,age=13,sex=0,service=null], people [id=597,name=people133,age=14,sex=1,service=null], people [id=598,name=people134,age=15,sex=0,service=null], people [id=599,name=people135,age=16,sex=1,service=null], people [id=600,name=people136,age=17,sex=0,service=null], people [id=601,name=people137,age=18,sex=1,service=null], people [id=602,name=people138,age=19,sex=0,service=null], people [id=603,name=people139,age=20,sex=1,service=null], people [id=581,name=people117,age=118,sex=1,service=null], people [id=582,name=people118,age=119,sex=0,service=null], people [id=583,name=people119,age=120,sex=1,service=null], people [id=584,name=people120,age=1,sex=0,service=null], people [id=585,name=people121,age=2,sex=1,service=null], people [id=586,name=people122,age=3,sex=0,service=null], people [id=587,name=people123,age=4,sex=1,service=null], people [id=588,name=people124,age=5,sex=0,service=null], people [id=589,name=people125,age=6,sex=1,service=null], people [id=590,name=people126,age=7,sex=0,service=null], people [id=591,name=people127,age=8,sex=1,service=null], people [id=630,name=people166,age=47,sex=0,service=null], people [id=631,name=people167,age=48,sex=1,service=null], people [id=632,name=people168,age=49,sex=0,service=null], people [id=633,name=people169,age=50,sex=1,service=null], people [id=634,name=people170,age=51,sex=0,service=null], people [id=635,name=people171,age=52,sex=1,service=null], people [id=636,name=people172,age=53,sex=0,service=null], people [id=637,name=people173,age=54,sex=1,service=null], people [id=638,name=people174,age=55,sex=0,service=null], people [id=639,name=people175,age=56,sex=1,service=null], people [id=640,name=people176,age=57,sex=0,service=null], people [id=641,name=people177,age=58,sex=1,service=null], people [id=642,name=people178,age=59,sex=0,service=null], people [id=643,name=people179,age=60,sex=1,service=null], people [id=644,name=people180,age=61,sex=0,service=null], people [id=645,name=people181,age=62,sex=1,service=null], people [id=646,name=people182,age=63,sex=0,service=null], people [id=647,name=people183,age=64,sex=1,service=null], people [id=648,name=people184,age=65,sex=0,service=null], people [id=649,name=people185,age=66,sex=1,service=null], people [id=604,name=people140,age=21,sex=0,service=null], people [id=605,name=people141,age=22,sex=1,service=null], people [id=606,name=people142,age=23,sex=0,service=null], people [id=650,name=people186,age=67,sex=0,service=null], people [id=651,name=people187,age=68,sex=1,service=null], people [id=652,name=people188,age=69,sex=0,service=null], people [id=607,name=people143,age=24,sex=1,service=null], people [id=608,name=people144,age=25,sex=0,service=null], people [id=609,name=people145,age=26,sex=1,service=null], people [id=653,name=people189,age=70,sex=1,service=null], people [id=654,name=people190,age=71,sex=0,service=null], people [id=655,name=people191,age=72,sex=1,service=null], people [id=610,name=people146,age=27,sex=0,service=null], people [id=611,name=people147,age=28,sex=1,service=null], people [id=612,name=people148,age=29,sex=0,service=null], people [id=656,name=people192,age=73,sex=0,service=null], people [id=657,name=people193,age=74,sex=1,service=null], people [id=658,name=people194,age=75,sex=0,service=null], people [id=613,name=people149,age=30,sex=1,service=null], people [id=614,name=people150,age=31,sex=0,service=null], people [id=615,name=people151,age=32,sex=1,service=null], people [id=659,name=people195,age=76,sex=1,service=null], people [id=660,name=people196,age=77,sex=0,service=null], people [id=661,name=people197,age=78,sex=1,service=null], people [id=616,name=people152,age=33,sex=0,service=null], people [id=617,name=people153,age=34,sex=1,service=null], people [id=662,name=people198,age=79,sex=0,service=null], people [id=663,name=people199,age=80,sex=1,service=null], people [id=664,name=people0,age=22,sex=1,service=null], people [id=618,name=people154,age=35,sex=0,service=null], people [id=619,name=people155,age=36,sex=1,service=null], people [id=620,name=people156,age=37,sex=0,service=null], people [id=621,name=people157,age=38,sex=1,service=null], people [id=622,name=people158,age=39,sex=0,service=null], people [id=623,name=people159,age=40,sex=1,service=null], people [id=624,name=people160,age=41,sex=0,service=null], people [id=625,name=people161,age=42,sex=1,service=null], people [id=626,name=people162,age=43,sex=0,service=null], people [id=627,name=people163,age=44,sex=1,service=null], people [id=628,name=people164,age=45,sex=0,service=null], people [id=629,name=people165,age=46,sex=1,service=null]]]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@1b083826]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@1b083826]
    DEBUG [main] - Returned connection 453523494 to pool.
    
    

#### 3.4.4 对多对关系service-people结果方式
    
    
    <!-- 结果方式 -->
    	<resultMap type="service" id="resultServiceWithPeople">
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
    	<select id="selectServiceWithPeopleResult" parameterType="service" resultMap="resultServiceWithPeople">
    		select p.id as pid,p.name as pname,p.age as page,p.sex as psex,
    		s.id as sid,s.name as sname,s.remark as sremark
    		from <include refid="baseMapper.str_table_people"></include>,
    		<include refid="baseMapper.str_table_service"></include>,
    		<include refid="baseMapper.str_table_people_service"></include>
    		<where>
    			and p.id=ps.people_id
    			and s.id=ps.service_id
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
    

编写测试方法
    
    
    @Test
    	public void selectServiceWithPeopleResult(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		Service service = new Service();
    		service.setId(128L);
    		session.selectList(
    				"moreToMoreForServiceMapper.selectServiceWithPeopleResult",
    				service).forEach(p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 274722023.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - ==>  Preparing: select p.id as pid,p.name as pname,p.age as page,p.sex as psex, s.id as sid,s.name as sname,s.remark as sremark from people p , service s , people_service ps WHERE p.id=ps.people_id and s.id=ps.service_id and s.id=? 
    DEBUG [main] - ==> Parameters: 128(Long)
    DEBUG [main] - <==      Total: 402
    [id=128,name=service128,remark=remark128,peoples=[people [id=268,name=people4,age=5,sex=0,service=null], people [id=269,name=people5,age=6,sex=1,service=null], people [id=270,name=people6,age=7,sex=0,service=null], people [id=271,name=people7,age=8,sex=1,service=null], people [id=272,name=people8,age=9,sex=0,service=null], people [id=273,name=people9,age=10,sex=1,service=null], people [id=274,name=people10,age=11,sex=0,service=null], people [id=275,name=people11,age=12,sex=1,service=null], people [id=141,name=people0,age=22,sex=1,service=null], people [id=142,name=people0,age=22,sex=1,service=null], people [id=143,name=people0,age=22,sex=1,service=null], people [id=265,name=people1,age=2,sex=1,service=null], people [id=266,name=people2,age=3,sex=0,service=null], people [id=267,name=people3,age=4,sex=1,service=null], people [id=288,name=people24,age=25,sex=0,service=null], people [id=289,name=people25,age=26,sex=1,service=null], people [id=290,name=people26,age=27,sex=0,service=null], people [id=291,name=people27,age=28,sex=1,service=null], people [id=292,name=people28,age=29,sex=0,service=null], people [id=293,name=people29,age=30,sex=1,service=null], people [id=294,name=people30,age=31,sex=0,service=null], people [id=295,name=people31,age=32,sex=1,service=null], people [id=296,name=people32,age=33,sex=0,service=null], people [id=297,name=people33,age=34,sex=1,service=null], people [id=298,name=people34,age=35,sex=0,service=null], people [id=276,name=people12,age=13,sex=0,service=null], people [id=277,name=people13,age=14,sex=1,service=null], people [id=278,name=people14,age=15,sex=0,service=null], people [id=279,name=people15,age=16,sex=1,service=null], people [id=280,name=people16,age=17,sex=0,service=null], people [id=281,name=people17,age=18,sex=1,service=null], people [id=282,name=people18,age=19,sex=0,service=null], people [id=283,name=people19,age=20,sex=1,service=null], people [id=284,name=people20,age=21,sex=0,service=null], people [id=285,name=people21,age=22,sex=1,service=null], people [id=286,name=people22,age=23,sex=0,service=null], people [id=287,name=people23,age=24,sex=1,service=null], people [id=299,name=people35,age=36,sex=1,service=null], people [id=300,name=people36,age=37,sex=0,service=null], people [id=301,name=people37,age=38,sex=1,service=null], people [id=308,name=people44,age=45,sex=0,service=null], people [id=309,name=people45,age=46,sex=1,service=null], people [id=310,name=people46,age=47,sex=0,service=null], people [id=311,name=people47,age=48,sex=1,service=null], people [id=312,name=people48,age=49,sex=0,service=null], people [id=313,name=people49,age=50,sex=1,service=null], people [id=314,name=people50,age=51,sex=0,service=null], people [id=315,name=people51,age=52,sex=1,service=null], people [id=316,name=people52,age=53,sex=0,service=null], people [id=317,name=people53,age=54,sex=1,service=null], people [id=318,name=people54,age=55,sex=0,service=null], people [id=319,name=people55,age=56,sex=1,service=null], people [id=302,name=people38,age=39,sex=0,service=null], people [id=303,name=people39,age=40,sex=1,service=null], people [id=304,name=people40,age=41,sex=0,service=null], people [id=305,name=people41,age=42,sex=1,service=null], people [id=306,name=people42,age=43,sex=0,service=null], people [id=307,name=people43,age=44,sex=1,service=null], people [id=331,name=people67,age=68,sex=1,service=null], people [id=332,name=people68,age=69,sex=0,service=null], people [id=333,name=people69,age=70,sex=1,service=null], people [id=334,name=people70,age=71,sex=0,service=null], people [id=335,name=people71,age=72,sex=1,service=null], people [id=336,name=people72,age=73,sex=0,service=null], people [id=337,name=people73,age=74,sex=1,service=null], people [id=338,name=people74,age=75,sex=0,service=null], people [id=339,name=people75,age=76,sex=1,service=null], people [id=340,name=people76,age=77,sex=0,service=null], people [id=341,name=people77,age=78,sex=1,service=null], people [id=342,name=people78,age=79,sex=0,service=null], people [id=320,name=people56,age=57,sex=0,service=null], people [id=321,name=people57,age=58,sex=1,service=null], people [id=322,name=people58,age=59,sex=0,service=null], people [id=323,name=people59,age=60,sex=1,service=null], people [id=324,name=people60,age=61,sex=0,service=null], people [id=325,name=people61,age=62,sex=1,service=null], people [id=326,name=people62,age=63,sex=0,service=null], people [id=327,name=people63,age=64,sex=1,service=null], people [id=328,name=people64,age=65,sex=0,service=null], people [id=329,name=people65,age=66,sex=1,service=null], people [id=330,name=people66,age=67,sex=0,service=null], people [id=343,name=people79,age=80,sex=1,service=null], people [id=344,name=people80,age=81,sex=0,service=null], people [id=345,name=people81,age=82,sex=1,service=null], people [id=351,name=people87,age=88,sex=1,service=null], people [id=352,name=people88,age=89,sex=0,service=null], people [id=353,name=people89,age=90,sex=1,service=null], people [id=354,name=people90,age=91,sex=0,service=null], people [id=355,name=people91,age=92,sex=1,service=null], people [id=356,name=people92,age=93,sex=0,service=null], people [id=357,name=people93,age=94,sex=1,service=null], people [id=358,name=people94,age=95,sex=0,service=null], people [id=359,name=people95,age=96,sex=1,service=null], people [id=360,name=people96,age=97,sex=0,service=null], people [id=361,name=people97,age=98,sex=1,service=null], people [id=362,name=people98,age=99,sex=0,service=null], people [id=346,name=people82,age=83,sex=0,service=null], people [id=347,name=people83,age=84,sex=1,service=null], people [id=348,name=people84,age=85,sex=0,service=null], people [id=349,name=people85,age=86,sex=1,service=null], people [id=350,name=people86,age=87,sex=0,service=null], people [id=375,name=people111,age=112,sex=1,service=null], people [id=376,name=people112,age=113,sex=0,service=null], people [id=377,name=people113,age=114,sex=1,service=null], people [id=378,name=people114,age=115,sex=0,service=null], people [id=379,name=people115,age=116,sex=1,service=null], people [id=380,name=people116,age=117,sex=0,service=null], people [id=381,name=people117,age=118,sex=1,service=null], people [id=382,name=people118,age=119,sex=0,service=null], people [id=383,name=people119,age=120,sex=1,service=null], people [id=384,name=people120,age=1,sex=0,service=null], people [id=385,name=people121,age=2,sex=1,service=null], people [id=363,name=people99,age=100,sex=1,service=null], people [id=364,name=people100,age=101,sex=0,service=null], people [id=365,name=people101,age=102,sex=1,service=null], people [id=366,name=people102,age=103,sex=0,service=null], people [id=367,name=people103,age=104,sex=1,service=null], people [id=368,name=people104,age=105,sex=0,service=null], people [id=369,name=people105,age=106,sex=1,service=null], people [id=370,name=people106,age=107,sex=0,service=null], people [id=371,name=people107,age=108,sex=1,service=null], people [id=372,name=people108,age=109,sex=0,service=null], people [id=373,name=people109,age=110,sex=1,service=null], people [id=374,name=people110,age=111,sex=0,service=null], people [id=386,name=people122,age=3,sex=0,service=null], people [id=387,name=people123,age=4,sex=1,service=null], people [id=388,name=people124,age=5,sex=0,service=null], people [id=395,name=people131,age=12,sex=1,service=null], people [id=396,name=people132,age=13,sex=0,service=null], people [id=397,name=people133,age=14,sex=1,service=null], people [id=398,name=people134,age=15,sex=0,service=null], people [id=399,name=people135,age=16,sex=1,service=null], people [id=400,name=people136,age=17,sex=0,service=null], people [id=401,name=people137,age=18,sex=1,service=null], people [id=402,name=people138,age=19,sex=0,service=null], people [id=403,name=people139,age=20,sex=1,service=null], people [id=404,name=people140,age=21,sex=0,service=null], people [id=405,name=people141,age=22,sex=1,service=null], people [id=389,name=people125,age=6,sex=1,service=null], people [id=390,name=people126,age=7,sex=0,service=null], people [id=391,name=people127,age=8,sex=1,service=null], people [id=392,name=people128,age=9,sex=0,service=null], people [id=393,name=people129,age=10,sex=1,service=null], people [id=394,name=people130,age=11,sex=0,service=null], people [id=418,name=people154,age=35,sex=0,service=null], people [id=419,name=people155,age=36,sex=1,service=null], people [id=420,name=people156,age=37,sex=0,service=null], people [id=421,name=people157,age=38,sex=1,service=null], people [id=422,name=people158,age=39,sex=0,service=null], people [id=423,name=people159,age=40,sex=1,service=null], people [id=424,name=people160,age=41,sex=0,service=null], people [id=425,name=people161,age=42,sex=1,service=null], people [id=426,name=people162,age=43,sex=0,service=null], people [id=427,name=people163,age=44,sex=1,service=null], people [id=428,name=people164,age=45,sex=0,service=null], people [id=429,name=people165,age=46,sex=1,service=null], people [id=406,name=people142,age=23,sex=0,service=null], people [id=407,name=people143,age=24,sex=1,service=null], people [id=408,name=people144,age=25,sex=0,service=null], people [id=409,name=people145,age=26,sex=1,service=null], people [id=410,name=people146,age=27,sex=0,service=null], people [id=411,name=people147,age=28,sex=1,service=null], people [id=412,name=people148,age=29,sex=0,service=null], people [id=413,name=people149,age=30,sex=1,service=null], people [id=414,name=people150,age=31,sex=0,service=null], people [id=415,name=people151,age=32,sex=1,service=null], people [id=416,name=people152,age=33,sex=0,service=null], people [id=417,name=people153,age=34,sex=1,service=null], people [id=430,name=people166,age=47,sex=0,service=null], people [id=431,name=people167,age=48,sex=1,service=null], people [id=438,name=people174,age=55,sex=0,service=null], people [id=439,name=people175,age=56,sex=1,service=null], people [id=440,name=people176,age=57,sex=0,service=null], people [id=441,name=people177,age=58,sex=1,service=null], people [id=442,name=people178,age=59,sex=0,service=null], people [id=443,name=people179,age=60,sex=1,service=null], people [id=444,name=people180,age=61,sex=0,service=null], people [id=445,name=people181,age=62,sex=1,service=null], people [id=446,name=people182,age=63,sex=0,service=null], people [id=447,name=people183,age=64,sex=1,service=null], people [id=448,name=people184,age=65,sex=0,service=null], people [id=449,name=people185,age=66,sex=1,service=null], people [id=432,name=people168,age=49,sex=0,service=null], people [id=433,name=people169,age=50,sex=1,service=null], people [id=434,name=people170,age=51,sex=0,service=null], people [id=435,name=people171,age=52,sex=1,service=null], people [id=436,name=people172,age=53,sex=0,service=null], people [id=437,name=people173,age=54,sex=1,service=null], people [id=461,name=people197,age=78,sex=1,service=null], people [id=462,name=people198,age=79,sex=0,service=null], people [id=463,name=people199,age=80,sex=1,service=null], people [id=465,name=people1,age=2,sex=1,service=null], people [id=466,name=people2,age=3,sex=0,service=null], people [id=467,name=people3,age=4,sex=1,service=null], people [id=468,name=people4,age=5,sex=0,service=null], people [id=469,name=people5,age=6,sex=1,service=null], people [id=470,name=people6,age=7,sex=0,service=null], people [id=471,name=people7,age=8,sex=1,service=null], people [id=472,name=people8,age=9,sex=0,service=null], people [id=473,name=people9,age=10,sex=1,service=null], people [id=450,name=people186,age=67,sex=0,service=null], people [id=451,name=people187,age=68,sex=1,service=null], people [id=452,name=people188,age=69,sex=0,service=null], people [id=453,name=people189,age=70,sex=1,service=null], people [id=454,name=people190,age=71,sex=0,service=null], people [id=455,name=people191,age=72,sex=1,service=null], people [id=456,name=people192,age=73,sex=0,service=null], people [id=457,name=people193,age=74,sex=1,service=null], people [id=458,name=people194,age=75,sex=0,service=null], people [id=459,name=people195,age=76,sex=1,service=null], people [id=460,name=people196,age=77,sex=0,service=null], people [id=474,name=people10,age=11,sex=0,service=null], people [id=475,name=people11,age=12,sex=1,service=null], people [id=476,name=people12,age=13,sex=0,service=null], people [id=483,name=people19,age=20,sex=1,service=null], people [id=484,name=people20,age=21,sex=0,service=null], people [id=485,name=people21,age=22,sex=1,service=null], people [id=486,name=people22,age=23,sex=0,service=null], people [id=487,name=people23,age=24,sex=1,service=null], people [id=488,name=people24,age=25,sex=0,service=null], people [id=489,name=people25,age=26,sex=1,service=null], people [id=490,name=people26,age=27,sex=0,service=null], people [id=491,name=people27,age=28,sex=1,service=null], people [id=492,name=people28,age=29,sex=0,service=null], people [id=493,name=people29,age=30,sex=1,service=null], people [id=477,name=people13,age=14,sex=1,service=null], people [id=478,name=people14,age=15,sex=0,service=null], people [id=479,name=people15,age=16,sex=1,service=null], people [id=480,name=people16,age=17,sex=0,service=null], people [id=481,name=people17,age=18,sex=1,service=null], people [id=482,name=people18,age=19,sex=0,service=null], people [id=506,name=people42,age=43,sex=0,service=null], people [id=507,name=people43,age=44,sex=1,service=null], people [id=508,name=people44,age=45,sex=0,service=null], people [id=509,name=people45,age=46,sex=1,service=null], people [id=510,name=people46,age=47,sex=0,service=null], people [id=511,name=people47,age=48,sex=1,service=null], people [id=512,name=people48,age=49,sex=0,service=null], people [id=513,name=people49,age=50,sex=1,service=null], people [id=514,name=people50,age=51,sex=0,service=null], people [id=515,name=people51,age=52,sex=1,service=null], people [id=516,name=people52,age=53,sex=0,service=null], people [id=494,name=people30,age=31,sex=0,service=null], people [id=495,name=people31,age=32,sex=1,service=null], people [id=496,name=people32,age=33,sex=0,service=null], people [id=497,name=people33,age=34,sex=1,service=null], people [id=498,name=people34,age=35,sex=0,service=null], people [id=499,name=people35,age=36,sex=1,service=null], people [id=500,name=people36,age=37,sex=0,service=null], people [id=501,name=people37,age=38,sex=1,service=null], people [id=502,name=people38,age=39,sex=0,service=null], people [id=503,name=people39,age=40,sex=1,service=null], people [id=504,name=people40,age=41,sex=0,service=null], people [id=505,name=people41,age=42,sex=1,service=null], people [id=517,name=people53,age=54,sex=1,service=null], people [id=518,name=people54,age=55,sex=0,service=null], people [id=519,name=people55,age=56,sex=1,service=null], people [id=526,name=people62,age=63,sex=0,service=null], people [id=527,name=people63,age=64,sex=1,service=null], people [id=528,name=people64,age=65,sex=0,service=null], people [id=529,name=people65,age=66,sex=1,service=null], people [id=530,name=people66,age=67,sex=0,service=null], people [id=531,name=people67,age=68,sex=1,service=null], people [id=532,name=people68,age=69,sex=0,service=null], people [id=533,name=people69,age=70,sex=1,service=null], people [id=534,name=people70,age=71,sex=0,service=null], people [id=535,name=people71,age=72,sex=1,service=null], people [id=536,name=people72,age=73,sex=0,service=null], people [id=520,name=people56,age=57,sex=0,service=null], people [id=521,name=people57,age=58,sex=1,service=null], people [id=522,name=people58,age=59,sex=0,service=null], people [id=523,name=people59,age=60,sex=1,service=null], people [id=524,name=people60,age=61,sex=0,service=null], people [id=525,name=people61,age=62,sex=1,service=null], people [id=549,name=people85,age=86,sex=1,service=null], people [id=550,name=people86,age=87,sex=0,service=null], people [id=551,name=people87,age=88,sex=1,service=null], people [id=552,name=people88,age=89,sex=0,service=null], people [id=553,name=people89,age=90,sex=1,service=null], people [id=554,name=people90,age=91,sex=0,service=null], people [id=555,name=people91,age=92,sex=1,service=null], people [id=556,name=people92,age=93,sex=0,service=null], people [id=557,name=people93,age=94,sex=1,service=null], people [id=558,name=people94,age=95,sex=0,service=null], people [id=559,name=people95,age=96,sex=1,service=null], people [id=560,name=people96,age=97,sex=0,service=null], people [id=537,name=people73,age=74,sex=1,service=null], people [id=538,name=people74,age=75,sex=0,service=null], people [id=539,name=people75,age=76,sex=1,service=null], people [id=540,name=people76,age=77,sex=0,service=null], people [id=541,name=people77,age=78,sex=1,service=null], people [id=542,name=people78,age=79,sex=0,service=null], people [id=543,name=people79,age=80,sex=1,service=null], people [id=544,name=people80,age=81,sex=0,service=null], people [id=545,name=people81,age=82,sex=1,service=null], people [id=546,name=people82,age=83,sex=0,service=null], people [id=547,name=people83,age=84,sex=1,service=null], people [id=548,name=people84,age=85,sex=0,service=null], people [id=561,name=people97,age=98,sex=1,service=null], people [id=562,name=people98,age=99,sex=0,service=null], people [id=569,name=people105,age=106,sex=1,service=null], people [id=570,name=people106,age=107,sex=0,service=null], people [id=571,name=people107,age=108,sex=1,service=null], people [id=572,name=people108,age=109,sex=0,service=null], people [id=573,name=people109,age=110,sex=1,service=null], people [id=574,name=people110,age=111,sex=0,service=null], people [id=575,name=people111,age=112,sex=1,service=null], people [id=576,name=people112,age=113,sex=0,service=null], people [id=577,name=people113,age=114,sex=1,service=null], people [id=578,name=people114,age=115,sex=0,service=null], people [id=579,name=people115,age=116,sex=1,service=null], people [id=580,name=people116,age=117,sex=0,service=null], people [id=563,name=people99,age=100,sex=1,service=null], people [id=564,name=people100,age=101,sex=0,service=null], people [id=565,name=people101,age=102,sex=1,service=null], people [id=566,name=people102,age=103,sex=0,service=null], people [id=567,name=people103,age=104,sex=1,service=null], people [id=568,name=people104,age=105,sex=0,service=null], people [id=592,name=people128,age=9,sex=0,service=null], people [id=593,name=people129,age=10,sex=1,service=null], people [id=594,name=people130,age=11,sex=0,service=null], people [id=595,name=people131,age=12,sex=1,service=null], people [id=596,name=people132,age=13,sex=0,service=null], people [id=597,name=people133,age=14,sex=1,service=null], people [id=598,name=people134,age=15,sex=0,service=null], people [id=599,name=people135,age=16,sex=1,service=null], people [id=600,name=people136,age=17,sex=0,service=null], people [id=601,name=people137,age=18,sex=1,service=null], people [id=602,name=people138,age=19,sex=0,service=null], people [id=603,name=people139,age=20,sex=1,service=null], people [id=581,name=people117,age=118,sex=1,service=null], people [id=582,name=people118,age=119,sex=0,service=null], people [id=583,name=people119,age=120,sex=1,service=null], people [id=584,name=people120,age=1,sex=0,service=null], people [id=585,name=people121,age=2,sex=1,service=null], people [id=586,name=people122,age=3,sex=0,service=null], people [id=587,name=people123,age=4,sex=1,service=null], people [id=588,name=people124,age=5,sex=0,service=null], people [id=589,name=people125,age=6,sex=1,service=null], people [id=590,name=people126,age=7,sex=0,service=null], people [id=591,name=people127,age=8,sex=1,service=null], people [id=630,name=people166,age=47,sex=0,service=null], people [id=631,name=people167,age=48,sex=1,service=null], people [id=632,name=people168,age=49,sex=0,service=null], people [id=633,name=people169,age=50,sex=1,service=null], people [id=634,name=people170,age=51,sex=0,service=null], people [id=635,name=people171,age=52,sex=1,service=null], people [id=636,name=people172,age=53,sex=0,service=null], people [id=637,name=people173,age=54,sex=1,service=null], people [id=638,name=people174,age=55,sex=0,service=null], people [id=639,name=people175,age=56,sex=1,service=null], people [id=640,name=people176,age=57,sex=0,service=null], people [id=641,name=people177,age=58,sex=1,service=null], people [id=642,name=people178,age=59,sex=0,service=null], people [id=643,name=people179,age=60,sex=1,service=null], people [id=644,name=people180,age=61,sex=0,service=null], people [id=645,name=people181,age=62,sex=1,service=null], people [id=646,name=people182,age=63,sex=0,service=null], people [id=647,name=people183,age=64,sex=1,service=null], people [id=648,name=people184,age=65,sex=0,service=null], people [id=649,name=people185,age=66,sex=1,service=null], people [id=604,name=people140,age=21,sex=0,service=null], people [id=605,name=people141,age=22,sex=1,service=null], people [id=606,name=people142,age=23,sex=0,service=null], people [id=650,name=people186,age=67,sex=0,service=null], people [id=651,name=people187,age=68,sex=1,service=null], people [id=652,name=people188,age=69,sex=0,service=null], people [id=607,name=people143,age=24,sex=1,service=null], people [id=608,name=people144,age=25,sex=0,service=null], people [id=609,name=people145,age=26,sex=1,service=null], people [id=653,name=people189,age=70,sex=1,service=null], people [id=654,name=people190,age=71,sex=0,service=null], people [id=655,name=people191,age=72,sex=1,service=null], people [id=610,name=people146,age=27,sex=0,service=null], people [id=611,name=people147,age=28,sex=1,service=null], people [id=612,name=people148,age=29,sex=0,service=null], people [id=656,name=people192,age=73,sex=0,service=null], people [id=657,name=people193,age=74,sex=1,service=null], people [id=658,name=people194,age=75,sex=0,service=null], people [id=613,name=people149,age=30,sex=1,service=null], people [id=614,name=people150,age=31,sex=0,service=null], people [id=615,name=people151,age=32,sex=1,service=null], people [id=659,name=people195,age=76,sex=1,service=null], people [id=660,name=people196,age=77,sex=0,service=null], people [id=661,name=people197,age=78,sex=1,service=null], people [id=616,name=people152,age=33,sex=0,service=null], people [id=617,name=people153,age=34,sex=1,service=null], people [id=662,name=people198,age=79,sex=0,service=null], people [id=663,name=people199,age=80,sex=1,service=null], people [id=664,name=people0,age=22,sex=1,service=null], people [id=618,name=people154,age=35,sex=0,service=null], people [id=619,name=people155,age=36,sex=1,service=null], people [id=620,name=people156,age=37,sex=0,service=null], people [id=621,name=people157,age=38,sex=1,service=null], people [id=622,name=people158,age=39,sex=0,service=null], people [id=623,name=people159,age=40,sex=1,service=null], people [id=624,name=people160,age=41,sex=0,service=null], people [id=625,name=people161,age=42,sex=1,service=null], people [id=626,name=people162,age=43,sex=0,service=null], people [id=627,name=people163,age=44,sex=1,service=null], people [id=628,name=people164,age=45,sex=0,service=null], people [id=629,name=people165,age=46,sex=1,service=null]]]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Returned connection 274722023 to pool.
    
    

### 3.5 测试

#### 3.5.1新增一个用户
    
    
    @Test
    	public void insertOnePeopleTest(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		People people = new People();
    		people.setName("testpeople");
    		people.setAge(102);
    		people.setSex(1);
    		System.out.println(session.insert("insertPeopleMapper.insertPeople", people));
    		session.commit();
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 1931444790.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - ==>  Preparing: insert into people p ( p.id,p.name,p.age,p.sex ) values(SEQ_PEOPLE.NEXTVAL,?,?,?) 
    DEBUG [main] - ==> Parameters: testpeople(String), 102(Integer), 1(Integer)
    DEBUG [main] - <==    Updates: 1
    1
    DEBUG [main] - Committing JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - Returned connection 1931444790 to pool.
    
    

#### 3.5.2 新增一个服务

测试方法
    
    
    @Test
    	public void insertOneServiceTest(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		Service service = new Service();
    		service.setName("testservice");
    		service.setRemark("testMoreToMoreservice");
    		System.out.println(session.insert("insertServiceMapper.insertService",service));
    		session.commit();
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 1931444790.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - ==>  Preparing: insert into service s ( s.id,s.name,s.remark ) values(SEQ_SERVICE.NEXTVAL,?,?) 
    DEBUG [main] - ==> Parameters: testservice(String), testMoreToMoreservice(String)
    DEBUG [main] - <==    Updates: 1
    1
    DEBUG [main] - Committing JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - Returned connection 1931444790 to pool.
    
    

#### 3.5.3查询用户

测试方法
    
    
    @Test
    	public void selectPeopleTest(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		People people = new People();
    		people.setName("testpeople");
    		session.selectList("selectPeopleMapper.selectPeople", people).forEach(
    				p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 274722023.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - ==>  Preparing: select p.id,p.name,p.age,p.sex from people p WHERE name=? 
    DEBUG [main] - ==> Parameters: testpeople(String)
    DEBUG [main] - <==      Total: 1
    people [id=666,name=testpeople,age=102,sex=1,service=null]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Returned connection 274722023 to pool.
    
    

#### 3.5.4查询服务

测试方法
    
    
    @Test
    	public void selectServiceTest(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		Service service = new Service();
    		service.setName("testservice");
    		session.selectList("selectServiceMapper.selectService", service)
    				.forEach(p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 274722023.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - ==>  Preparing: select s.id,s.name,s.remark from service s WHERE name=? 
    DEBUG [main] - ==> Parameters: testservice(String)
    DEBUG [main] - <==      Total: 1
    [id=200,name=testservice,remark=testMoreToMoreservice,peoples=null]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Returned connection 274722023 to pool.
    
    

#### 3.5.5增加关系

mapper
    
    
    <insert id="insertPeopleServiceOne" parameterType="people_service">
    		insert into 
    		<include refid="baseMapper.str_table_people_service"></include>
    		(PS.PEOPLE_ID, PS.SERVICE_ID) 
    		values(#{people_fk},#{service_fk})
    	</insert>
    

测试方法
    
    
    @Test
    	public void insertPeopleService(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		PeopleService peopleService = new PeopleService();
    		peopleService.setPeople_fk(666L);
    		peopleService.setService_fk(200L);
    		System.out
    				.println(session.insert(
    						"insertPeopleServiceMapper.insertPeopleServiceOne",
    						peopleService));
    		session.commit();
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 1931444790.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - ==>  Preparing: insert into people_service ps (PS.PEOPLE_ID, PS.SERVICE_ID) values(?,?) 
    DEBUG [main] - ==> Parameters: 666(Long), 200(Long)
    DEBUG [main] - <==    Updates: 1
    1
    DEBUG [main] - Committing JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@731f8236]
    DEBUG [main] - Returned connection 1931444790 to pool.
    
    

#### 3.5.6查询用户

测试方法
    
    
    @Test
    	public void selectPeopleAndService(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		People people = new People();
    		people.setName("testpeople");
    		session.selectList("moreToMoreForPeopleMapper.selectPeopleWithService",
    				people).forEach(p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 274722023.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - ==>  Preparing: select p.id,p.name,p.age,p.sex from people p WHERE p.name=? 
    DEBUG [main] - ==> Parameters: testpeople(String)
    DEBUG [main] - <==      Total: 1
    DEBUG [main] - ==>  Preparing: select s.id,s.name,s.remark from service s , people_service ps WHERE s.id = ps.service_id and ps.people_id=? 
    DEBUG [main] - ==> Parameters: 666(Long)
    DEBUG [main] - <==      Total: 1
    people [id=666,name=testpeople,age=102,sex=1,service=[[id=200,name=testservice,remark=testMoreToMoreservice,peoples=null]]]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Returned connection 274722023 to pool.
    
    

#### 3.5.7查询服务

测试方法
    
    
    @Test
    	public void selectServiceAndPeople(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		Service service = new Service();
    		service.setName("testservice");
    		session.selectList(
    				"moreToMoreForServiceMapper.selectServiceWithPeople", service)
    				.forEach(p -> System.out.println(p));
    		session.close();
    	}
    

测试结果
    
    
    DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
    DEBUG [main] - Opening JDBC Connection
    DEBUG [main] - Created connection 274722023.
    DEBUG [main] - Setting autocommit to false on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - ==>  Preparing: select s.id,s.name,s.remark from service s WHERE s.name=? 
    DEBUG [main] - ==> Parameters: testservice(String)
    DEBUG [main] - <==      Total: 1
    DEBUG [main] - ==>  Preparing: select p.id,p.name,p.age,p.sex from people p , people_service ps WHERE p.id = ps.people_id and ps.service_id=? 
    DEBUG [main] - ==> Parameters: 200(Long)
    DEBUG [main] - <==      Total: 1
    [id=200,name=testservice,remark=testMoreToMoreservice,peoples=[people [id=666,name=testpeople,age=102,sex=1,service=null]]]
    DEBUG [main] - Resetting autocommit to true on JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Closing JDBC Connection [oracle.jdbc.driver.T4CConnection@105fece7]
    DEBUG [main] - Returned connection 274722023 to pool.
    
    

## 4.总结

MyBatis的三种映射基本上可以满足所有的关系，而且resultMap这个方法可以让返回的结果封装更加灵活。  
所以，编写MyBatis的映射应该尽可能的考虑到复用性。
