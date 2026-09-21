---
layout: post
title: "MyBatis动态SQL"
date: 2019-03-03 21:16:47 +0800
categories: [MyBatis动态SQL, MyBatis如何降低工作量, MyBatis的动态SQL的应用, MyBatis动态SQL的实例, MyBatis各种动态SQL解析]
description: "MyBatis动态SQL1.动态SQL的必要性2.动态SQL的标签2.1共用的配置：2.2if2.3choose2.4where2.5trim2.6set2.7foreach2.8bind3.例子3.1创建一个MyBatis的工程：3.2Java文件3.3mybatis.xml3.4properties3.5mapper4.运行结果5总结1.动态SQL的必要性学习了MyBatis的各种标签后，..."
keywords: MyBatis动态SQL, MyBatis如何降低工作量, MyBatis的动态SQL的应用, MyBatis动态SQL的实例, MyBatis各种动态SQL解析
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88093445
> - 发布时间：2019-03-03 21:16:47
> - 阅读量：1
> - 分类：java同时被 3 个专栏收录, 订阅专栏, mybatis, MyBatis
> - 标签：#MyBatis动态SQL, #MyBatis如何降低工作量, #MyBatis的动态SQL的应用, #MyBatis动态SQL的实例, #MyBatis各种动态SQL解析

## 摘要

文章浏览阅读1k次，点赞3次，收藏22次。MyBatis动态SQL1.动态SQL的必要性2.动态SQL的标签2.1共用的配置：2.2if2.3choose2.4where2.5trim2.6set2.7foreach2.8bind3.例子3.1创建一个MyBatis的工程：3.2Java文件3.3mybatis.xml3.4properties3.5mapper4.运行结果5总结1.动态SQL的必要性学习了MyBatis的各种标签后，...

---

#### MyBatis动态SQL

  * [1.动态SQL的必要性](<#1SQL_1>)
  * [2.动态SQL的标签](<#2SQL_19>)
  *     * [2.1共用的配置：](<#21_27>)
    * [2.2if](<#22if_43>)
    * [2.3choose](<#23choose_74>)
    * [2.4where](<#24where_108>)
    * [2.5trim](<#25trim_140>)
    * [2.6set](<#26set_172>)
    * [2.7foreach](<#27foreach_227>)
    * [2.8bind](<#28bind_251>)
  * [3.例子](<#3_287>)
  *     * [3.1创建一个MyBatis的工程：](<#31MyBatis_288>)
    * [3.2Java文件](<#32Java_291>)
    * [3.3mybatis.xml](<#33mybatisxml_632>)
    * [3.4properties](<#34properties_670>)
    * [3.5mapper](<#35mapper_690>)
  * [4.运行结果](<#4_699>)
  * [5总结](<#5_1083>)

## 1.动态SQL的必要性

学习了MyBatis的各种标签后，已经觉得Mybatis已经很方便了。但是，MyBatis的标签还是有许多的不好的地方：  
如果一个实体有4个属性，那么使用标签的方式写查询语句需要写多少条呢？  
4个属性4条；  
全查1条；  
全不查1条；  
2个属性查：3+2+1=6条；  
3个属性查：4条；  
总共：  
4+1+1+6+4 = 16条。

如果有N个属性呢：  
1+1+(n->1–N)CnM;  
解释一下：  
全查，全不查，然后从1到n的排列组合：每次取n个，CnM(M是总数)求和。  
这个化解下来就是N的平方次。  
对于二次函数，根据走向知道，其的增量是递增的，所以，属性个数多的时候，需要写的语句非常恐怖的。  
而动态SQL可能只需要写一条。

## 2.动态SQL的标签

1.if  
2.choose  
3.where  
4.trim  
5.set  
6.foreach  
7.bind

### 2.1共用的配置：

base.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="peopleBaseMapper">
      <sql id="str_select">
      	p.id,p.name,p.age,p.sex
      </sql>
      <sql id="str_table">
      	people p
      </sql>
    </mapper>
    

### 2.2if

if标签表示如果，判断条件为真就会拼接if中的SQL语句。  
ifMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="peopleIfMapper">
      <select id="selectIf" parameterType="people" resultType="people">
      	select <include refid="peopleBaseMapper.str_select">
      	</include>
      	 from <include refid="peopleBaseMapper.str_table">
      	 </include>
      	 where 1=1
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
      </select>
    </mapper>
    

其中include元素表示引用id为XXX的SQL片段。

### 2.3choose

choose标签与switch类似  
chooseMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="peopleChooseMapper">
      <select id="selectChoose" parameterType="people" resultType="people">
      	select <include refid="peopleBaseMapper.str_select">
      	</include>
      	 from <include refid="peopleBaseMapper.str_table">
      	 </include>
      	 where 1=1
      	 <choose>
      	 	<when test="id != null and id != ''">
      	 		and id=#{id}
      	 	</when>
      	 	<when test="name != null and name != ''">
      	 		and name=#{name}
      	 	</when>
      	 	<when test="age != null and age != ''">
      	 		and age=#{age}
      	 	</when>
      	 	<when test="sex != null and sex != ''">
      	 		and sex=#{sex}
      	 	</when>
      	 	<otherwise>
      	 	</otherwise>
      	 </choose>
      </select>
    </mapper>
    

### 2.4where

上面两种都需要写where 1=1这是为了让后面可以直接用and+条件，不用考虑第一个条件不用and的情况，所以，如果条件都为空就是where 1=1  
where标签根据条件去除第一个条件的and以及都为空时，不拼接where关键字  
whereMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="peopleWhereMapper">
      <select id="selectWhere" parameterType="people" resultType="people">
      	select <include refid="peopleBaseMapper.str_select">
      	</include>
      	 from <include refid="peopleBaseMapper.str_table">
      	 </include>
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
    

### 2.5trim

trim比where标签更加强大，where只能处理SQL中的条件，而trim标签可以处理包括where的类似情况。  
trim的两个属性prefix表示前缀（如果trim中的为空则前缀不拼接），prefixOverrides表示如果前缀后面紧跟的字符为指定的字符则去掉前缀后面的这个字符，其余的不去掉。  
trimMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="peopleTrimMapper">
      <select id="selectTrim" parameterType="people" resultType="people">
      	select <include refid="peopleBaseMapper.str_select">
      	</include>
      	 from <include refid="peopleBaseMapper.str_table">
      	 </include>
      	 <trim prefix="where" prefixOverrides="and">
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
      	 </trim>
      </select>
    </mapper>
    

### 2.6set

类似SQL的条件，如果在更新或者插入操作时，设置或者更新的值可能只有其中一个字段，使用if也需要进行处理后才能使用，比如设置一个无用字段放在最前面，只是为了让后面的字段可以直接拼接，+属性。  
set可以实现根据需要增加set关键字以及去掉最后一个属性后面的英文逗号：  
setMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="peopleSetMapper">
    	<update id="updateSet" parameterType="people">
    		update <include refid="peopleBaseMapper.str_table"></include>
    		<set>
    		  	<if test="name != null and name != ''">
    		  	 	name=#{name},
    		  	</if>
    		  	<if test="age != null and age != ''">
    		  	 	age=#{age},
    		  	</if>
    		  	<if test="sex != null and sex != ''">
    		  	 	sex=#{sex},
    		  	</if>
    		</set>
    		<where>
    			<choose>
    				<when test="id != null and id != ''">
    					id=#{id}
    				</when>
    				<otherwise>
    					1=2
    				</otherwise>
    			</choose>
    		</where>
    	</update>
    	<update id="updateTrim" parameterType="people">
    		update <include refid="peopleBaseMapper.str_table"></include>
    		<trim prefix="set" prefixOverrides=",">
    			<if test="name != null and name != ''">
    		  	 	,name=#{name}
    		  	</if>
    		  	<if test="age != null and age != ''">
    		  	 	,age=#{age}
    		  	</if>
    		  	<if test="sex != null and sex != ''">
    		  	 	,sex=#{sex}
    		  	</if>
    		</trim>
    		<trim prefix="where" prefixOverrides="and">
    			<if test="id != null and id != ''">
    				and id=#{id}
    			</if>
    		</trim>
    	</update>
    </mapper>
    

### 2.7foreach

上述要么查一个，要么查很多，如果我需要查询指定的一批id，进行批量查询，那么就只能多次调用查询一个的方法，这样对数据库的压力比较大。  
foreach就是在数据库以及JDBC底层级别上进行批量处理，处理的速度以及降低资源的消耗上非常出色。  
foreachMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="peopleForeachMapper">
      <select id="selectForeach" parameterType="List" resultType="people">
      	select <include refid="peopleBaseMapper.str_select">
      	</include>
      	 from <include refid="peopleBaseMapper.str_table">
      	 </include>
      	 <where>
      	 	and id in
      	 	<foreach collection="list" item="id" index="index" open="(" separator="," close=")">
      	 		#{id}
      	 	</foreach>
      	 </where>
      </select>
    </mapper>
    

### 2.8bind

上述所有的查询都是精确查询，如果使用模糊查询则面临着SQL注入的风险。  
如果直接使用${}直接进行字符串的拼接，SQL的风险无法避免。  
bind则使用指定的规则进行拼接安全的SQL：  
name给这个字符串拼接指定一个标识，  
value是字符串的拼接规则。  
bindMapper.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="peopleBindMapper">
      <select id="selectBind" parameterType="people" resultType="people">
      	<bind name="people_name" value="'%'+name+'%'"/>
      	select <include refid="peopleBaseMapper.str_select">
      	</include>
      	 from <include refid="peopleBaseMapper.str_table">
      	 </include>
      	 <where>
    	  	 <if test="id != null and id != ''">
    	  	 	and id=#{id}
    	  	 </if>
    	  	 <if test="name != null and name != ''">
    	  	 	and name like #{people_name}
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
    

## 3.例子

### 3.1创建一个MyBatis的工程：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/79071da486787b18441e42ca90c194ad.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3dd9db72a956c831d1ff17c3d75f23f4.png)

### 3.2Java文件
    
    
    package client;
    
    import java.util.ArrayList;
    import java.util.List;
    
    import org.apache.ibatis.session.SqlSession;
    import org.junit.Test;
    
    import util.MyBatisSessionUtils;
    import domain.People;
    
    
    public class Main {
    
    	@Test
    	public void testIfAll(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("all");
    		People people = new People();
    		session.selectList("peopleIfMapper.selectIf",
    				people).forEach(p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testIfId(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("id=103");
    		People people = new People();
    		people.setId(103L);
    		System.out
    				.println(session.selectOne("peopleIfMapper.selectIf", people));
    		session.close();
    	}
    	
    	@Test
    	public void testIfName(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("name=aPeopleupdate");
    		People people = new People();
    		people.setName("aPeopleupdate");
    		session.selectList("peopleIfMapper.selectIf", people).forEach(
    				p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testIfIdAndName(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("id=119,name=aPeopleupdate");
    		People people = new People();
    		people.setId(119L);
    		people.setName("aPeopleupdate");
    		System.out.println(session.selectOne("peopleIfMapper.selectIf",people));
    		session.close();
    	}
    	
    	@Test
    	public void testIfAllAttr(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("id=127,name=aPeopleupdate,age=28,sex=0");
    		People people = new People();
    		people.setId(127L);
    		people.setName("aPeopleupdate");
    		people.setAge(28);
    		people.setSex(0);
    		System.out.println(session.selectOne("peopleIfMapper.selectIf", people));
    		session.close();
    	}
    	
    	@Test
    	public void testChooseId(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("choose:id=103");
    		People people = new People();
    		people.setId(103L);
    		System.out.println(session.selectOne("peopleChooseMapper.selectChoose",people));
    		session.close();
    	}
    	
    	@Test
    	public void testChooseName(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("choose:name=aPeopleupdate");
    		People people = new People();
    		people.setName("aPeopleupdate");
    		session.selectList("peopleChooseMapper.selectChoose", people).forEach(
    				p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testChooseAge(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("choose:age=28");
    		People people = new People();
    		people.setAge(28);
    		session.selectList("peopleChooseMapper.selectChoose", people).forEach(
    				p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testChooseIdAndName(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("choose:id=129,name=aPeopleupdate");
    		People people = new People();
    		people.setId(129L);
    		people.setName("bPeopleupdate");
    		System.out.println(session.selectOne("peopleChooseMapper.selectChoose",
    				people));
    		session.close();
    	}
    	
    	@Test
    	public void testChooseAllNull(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("choose:all null");
    		session.selectList("peopleChooseMapper.selectChoose", new People())
    				.forEach(p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testWhereAll(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("where:all");
    		session.selectList("peopleWhereMapper.selectWhere", new People())
    				.forEach(p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testWhereId(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("where:id=103");
    		People people = new People();
    		people.setId(103L);
    		System.out.println(session.selectOne("peopleWhereMapper.selectWhere",
    				people));
    		session.close();
    	}
    	
    	@Test
    	public void testWhereAgeAndSex(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("where:age=28,sex=0");
    		People people = new People();
    		people.setAge(28);
    		people.setSex(0);
    		session.selectList("peopleWhereMapper.selectWhere", people).forEach(
    				p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testTrimAll(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("trim:all");
    		session.selectList("peopleTrimMapper.selectTrim", new People())
    				.forEach(p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testTrimId(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("trim:id=103");
    		People people = new People();
    		people.setId(103L);
    		System.out.println(session.selectOne("peopleTrimMapper.selectTrim",people));
    		session.close();
    	}
    	
    	@Test
    	public void testTrimNameAndSex(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("trim:name=aPeopleupdate,sex=0");
    		People people = new People();
    		people.setName("aPeopleupdate");
    		people.setSex(0);
    		session.selectList("peopleTrimMapper.selectTrim", people).forEach(
    				p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testUpdateSetName(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("update:name=20190303");
    		People people = new People();
    		people.setId(119L);
    		people.setName("20190303");
    		System.out.println(session.update("peopleSetMapper.updateSet", people));
    		System.out.println(session.selectOne("peopleTrimMapper.selectTrim",people));
    		session.close();
    	}
    	
    	@Test
    	public void testUpdateSetNull(){
    		//必须保证set中至少有一个值
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("update:null");
    		System.out.println(session.update("peopleSetMapper.updateSet", new People()));
    		session.close();
    	}
    	
    	@Test
    	public void testUpdateTrim(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		System.out.println("update:trim:name=20190303");
    		People people = new People();
    		people.setId(119L);
    		people.setName("20190303");
    		System.out.println(session.update("peopleSetMapper.updateTrim", people));
    		System.out.println(session.selectOne("peopleTrimMapper.selectTrim",people));
    		session.close();
    	}
    	
    	@Test
    	public void testSelectForeach(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		List<Long> ids = new ArrayList<>();
    		ids.add(117L);
    		ids.add(118L);
    		ids.add(119L);
    		session.selectList("peopleForeachMapper.selectForeach", ids).forEach(
    				p -> System.out.println(p));
    		session.close();
    	}
    	
    	@Test
    	public void testSelectBindLike(){
    		SqlSession session = MyBatisSessionUtils.getSession();
    		People people = new People();
    		people.setName("update");
    		session.selectList("peopleBindMapper.selectBind", people).forEach(
    				p -> System.out.println(p));
    		session.close();
    	}
    }
    
    
    
    
    package domain;
    
    import java.io.Serializable;
    
    import org.apache.ibatis.type.Alias;
    
    @Alias(value = "people")
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
    				+ this.age + ",sex=" + this.sex + "]";
    	}
    	
    	
    }
    
    
    
    
    package util;
    
    import java.io.Reader;
    
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    
    public class MyBatisSessionUtils {
    
    	private static SqlSessionFactory sessionFactory = null;
    	
    	private static final String PATH="resource/mybatis.xml";
    	
    	static{
    		try{
    			Reader reader = Resources.getResourceAsReader(PATH);
    			sessionFactory = new SqlSessionFactoryBuilder().build(reader);
    		} catch (Exception e){
    			e.printStackTrace();
    		}
    	}
    	
    	public static SqlSession getSession(){
    		return sessionFactory.openSession();
    	}
    }
    
    

### 3.3mybatis.xml
    
    
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
      <!-- 配置外在化 -->
      <properties resource="property/ojdbc.properties"></properties>
      <!-- 改变运行时行为 -->
      <typeAliases>
    	  <!-- 配置别名 -->
      	<typeAlias type="domain.People" alias="people"/>
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
      	<mapper resource="mapper/base.xml"/>
      	<mapper resource="mapper/chooseMapper.xml"/>
      	<mapper resource="mapper/foreachMapper.xml"/>
      	<mapper resource="mapper/ifMapper.xml"/>
      	<mapper resource="mapper/setMapper.xml"/>
      	<mapper resource="mapper/trimMapper.xml"/>
      	<mapper resource="mapper/whereMapper.xml"/>
      	<mapper resource="mapper/bindMapper.xml"/>
      </mappers>
    </configuration>
    

### 3.4properties

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
    

ojdbc.properties
    
    
    driver=oracle.jdbc.driver.OracleDriver
    url=jdbc:oracle:thin:@127.0.0.1:1521:oracle
    username=study
    password=study
    

### 3.5mapper

base.xml  
bindMapper.xml  
chooseMapper.xml  
foreachMapper.xml  
ifMapper.xml  
setMapper.xml  
trimMapper.xml  
whereMapper.xml

## 4.运行结果

testIfAll
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    all
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    people [id=104,name=bPeopleupdate,age=62,sex=1]
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=122,name=bPeopleupdate,age=37,sex=1]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=124,name=bPeopleupdate,age=37,sex=1]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=126,name=bPeopleupdate,age=37,sex=1]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=128,name=bPeopleupdate,age=37,sex=1]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=130,name=bPeopleupdate,age=37,sex=1]
    people [id=135,name=2019-34-27 07:34:34,age=0,sex=0]
    people [id=136,name=2019-38-27 07:38:09,age=0,sex=0]
    people [id=102,name=bPeople,age=62,sex=1]
    people [id=105,name=aPeopleupdate,age=53,sex=0]
    people [id=101,name=aPeopleupdateMyBatisupdateMyBatis,age=53,sex=0]
    people [id=106,name=bPeople,age=62,sex=1]
    people [id=107,name=aPeopleupdate,age=53,sex=0]
    people [id=108,name=bPeople,age=62,sex=1]
    people [id=109,name=aPeopleupdate,age=53,sex=0]
    people [id=110,name=bPeopleupdate,age=62,sex=1]
    people [id=111,name=aPeopleupdate,age=48,sex=0]
    people [id=112,name=bPeopleupdate,age=57,sex=1]
    people [id=113,name=aPeopleupdate,age=43,sex=0]
    people [id=114,name=bPeopleupdate,age=52,sex=1]
    people [id=115,name=aPeopleupdate,age=38,sex=0]
    people [id=116,name=bPeopleupdate,age=47,sex=1]
    people [id=117,name=aPeopleupdate,age=33,sex=0]
    people [id=118,name=bPeopleupdate,age=42,sex=1]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    people [id=120,name=bPeopleupdate,age=37,sex=1]
    
    

testIfId
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    id=103
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    
    

testIfName
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    name=aPeopleupdate
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=105,name=aPeopleupdate,age=53,sex=0]
    people [id=107,name=aPeopleupdate,age=53,sex=0]
    people [id=109,name=aPeopleupdate,age=53,sex=0]
    people [id=111,name=aPeopleupdate,age=48,sex=0]
    people [id=113,name=aPeopleupdate,age=43,sex=0]
    people [id=115,name=aPeopleupdate,age=38,sex=0]
    people [id=117,name=aPeopleupdate,age=33,sex=0]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    
    

testIfIdAndName
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    id=119,name=aPeopleupdate
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    
    

testIfAllAttr
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    id=127,name=aPeopleupdate,age=28,sex=0
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    
    

testChooseId
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    choose:id=103
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    
    

testChooseName
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    choose:name=aPeopleupdate
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=105,name=aPeopleupdate,age=53,sex=0]
    people [id=107,name=aPeopleupdate,age=53,sex=0]
    people [id=109,name=aPeopleupdate,age=53,sex=0]
    people [id=111,name=aPeopleupdate,age=48,sex=0]
    people [id=113,name=aPeopleupdate,age=43,sex=0]
    people [id=115,name=aPeopleupdate,age=38,sex=0]
    people [id=117,name=aPeopleupdate,age=33,sex=0]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    
    

testChooseAge
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    choose:age=28
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    
    

testChooseIdAndName
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    choose:id=129,name=aPeopleupdate
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    
    

testChooseAllNull
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    choose:all null
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    people [id=104,name=bPeopleupdate,age=62,sex=1]
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=122,name=bPeopleupdate,age=37,sex=1]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=124,name=bPeopleupdate,age=37,sex=1]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=126,name=bPeopleupdate,age=37,sex=1]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=128,name=bPeopleupdate,age=37,sex=1]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=130,name=bPeopleupdate,age=37,sex=1]
    people [id=135,name=2019-34-27 07:34:34,age=0,sex=0]
    people [id=136,name=2019-38-27 07:38:09,age=0,sex=0]
    people [id=102,name=bPeople,age=62,sex=1]
    people [id=105,name=aPeopleupdate,age=53,sex=0]
    people [id=101,name=aPeopleupdateMyBatisupdateMyBatis,age=53,sex=0]
    people [id=106,name=bPeople,age=62,sex=1]
    people [id=107,name=aPeopleupdate,age=53,sex=0]
    people [id=108,name=bPeople,age=62,sex=1]
    people [id=109,name=aPeopleupdate,age=53,sex=0]
    people [id=110,name=bPeopleupdate,age=62,sex=1]
    people [id=111,name=aPeopleupdate,age=48,sex=0]
    people [id=112,name=bPeopleupdate,age=57,sex=1]
    people [id=113,name=aPeopleupdate,age=43,sex=0]
    people [id=114,name=bPeopleupdate,age=52,sex=1]
    people [id=115,name=aPeopleupdate,age=38,sex=0]
    people [id=116,name=bPeopleupdate,age=47,sex=1]
    people [id=117,name=aPeopleupdate,age=33,sex=0]
    people [id=118,name=bPeopleupdate,age=42,sex=1]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    people [id=120,name=bPeopleupdate,age=37,sex=1]
    
    

testWhereAll
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    where:all
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    people [id=104,name=bPeopleupdate,age=62,sex=1]
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=122,name=bPeopleupdate,age=37,sex=1]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=124,name=bPeopleupdate,age=37,sex=1]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=126,name=bPeopleupdate,age=37,sex=1]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=128,name=bPeopleupdate,age=37,sex=1]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=130,name=bPeopleupdate,age=37,sex=1]
    people [id=135,name=2019-34-27 07:34:34,age=0,sex=0]
    people [id=136,name=2019-38-27 07:38:09,age=0,sex=0]
    people [id=102,name=bPeople,age=62,sex=1]
    people [id=105,name=aPeopleupdate,age=53,sex=0]
    people [id=101,name=aPeopleupdateMyBatisupdateMyBatis,age=53,sex=0]
    people [id=106,name=bPeople,age=62,sex=1]
    people [id=107,name=aPeopleupdate,age=53,sex=0]
    people [id=108,name=bPeople,age=62,sex=1]
    people [id=109,name=aPeopleupdate,age=53,sex=0]
    people [id=110,name=bPeopleupdate,age=62,sex=1]
    people [id=111,name=aPeopleupdate,age=48,sex=0]
    people [id=112,name=bPeopleupdate,age=57,sex=1]
    people [id=113,name=aPeopleupdate,age=43,sex=0]
    people [id=114,name=bPeopleupdate,age=52,sex=1]
    people [id=115,name=aPeopleupdate,age=38,sex=0]
    people [id=116,name=bPeopleupdate,age=47,sex=1]
    people [id=117,name=aPeopleupdate,age=33,sex=0]
    people [id=118,name=bPeopleupdate,age=42,sex=1]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    people [id=120,name=bPeopleupdate,age=37,sex=1]
    
    

testWhereId
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    where:id=103
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    
    

testWhereAgeAndSex
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    where:age=28,sex=0
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    
    

testTrimAll
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    trim:all
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    people [id=104,name=bPeopleupdate,age=62,sex=1]
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=122,name=bPeopleupdate,age=37,sex=1]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=124,name=bPeopleupdate,age=37,sex=1]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=126,name=bPeopleupdate,age=37,sex=1]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=128,name=bPeopleupdate,age=37,sex=1]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=130,name=bPeopleupdate,age=37,sex=1]
    people [id=135,name=2019-34-27 07:34:34,age=0,sex=0]
    people [id=136,name=2019-38-27 07:38:09,age=0,sex=0]
    people [id=102,name=bPeople,age=62,sex=1]
    people [id=105,name=aPeopleupdate,age=53,sex=0]
    people [id=101,name=aPeopleupdateMyBatisupdateMyBatis,age=53,sex=0]
    people [id=106,name=bPeople,age=62,sex=1]
    people [id=107,name=aPeopleupdate,age=53,sex=0]
    people [id=108,name=bPeople,age=62,sex=1]
    people [id=109,name=aPeopleupdate,age=53,sex=0]
    people [id=110,name=bPeopleupdate,age=62,sex=1]
    people [id=111,name=aPeopleupdate,age=48,sex=0]
    people [id=112,name=bPeopleupdate,age=57,sex=1]
    people [id=113,name=aPeopleupdate,age=43,sex=0]
    people [id=114,name=bPeopleupdate,age=52,sex=1]
    people [id=115,name=aPeopleupdate,age=38,sex=0]
    people [id=116,name=bPeopleupdate,age=47,sex=1]
    people [id=117,name=aPeopleupdate,age=33,sex=0]
    people [id=118,name=bPeopleupdate,age=42,sex=1]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    people [id=120,name=bPeopleupdate,age=37,sex=1]
    
    

testTrimId
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    trim:id=103
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    
    

testTrimNameAndSex
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    trim:name=aPeopleupdate,sex=0
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=105,name=aPeopleupdate,age=53,sex=0]
    people [id=107,name=aPeopleupdate,age=53,sex=0]
    people [id=109,name=aPeopleupdate,age=53,sex=0]
    people [id=111,name=aPeopleupdate,age=48,sex=0]
    people [id=113,name=aPeopleupdate,age=43,sex=0]
    people [id=115,name=aPeopleupdate,age=38,sex=0]
    people [id=117,name=aPeopleupdate,age=33,sex=0]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    
    

testUpdateSetName
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    update:name=20190303
    1
    people [id=119,name=20190303,age=28,sex=0]
    
    

testUpdateSetNull  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/232a40ff4195009d2ddbf82a45e4ef03.png)  
testUpdateTrim
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    update:trim:name=20190303
    1
    people [id=119,name=20190303,age=28,sex=0]
    
    

testSelectForeach
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    people [id=117,name=aPeopleupdate,age=33,sex=0]
    people [id=118,name=bPeopleupdate,age=42,sex=1]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    
    

testSelectBindLike
    
    
    log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
    log4j:WARN Please initialize the log4j system properly.
    log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
    people [id=103,name=aPeopleupdate,age=53,sex=0]
    people [id=104,name=bPeopleupdate,age=62,sex=1]
    people [id=121,name=aPeopleupdate,age=28,sex=0]
    people [id=122,name=bPeopleupdate,age=37,sex=1]
    people [id=123,name=aPeopleupdate,age=28,sex=0]
    people [id=124,name=bPeopleupdate,age=37,sex=1]
    people [id=125,name=aPeopleupdate,age=28,sex=0]
    people [id=126,name=bPeopleupdate,age=37,sex=1]
    people [id=127,name=aPeopleupdate,age=28,sex=0]
    people [id=128,name=bPeopleupdate,age=37,sex=1]
    people [id=129,name=aPeopleupdate,age=28,sex=0]
    people [id=130,name=bPeopleupdate,age=37,sex=1]
    people [id=105,name=aPeopleupdate,age=53,sex=0]
    people [id=101,name=aPeopleupdateMyBatisupdateMyBatis,age=53,sex=0]
    people [id=107,name=aPeopleupdate,age=53,sex=0]
    people [id=109,name=aPeopleupdate,age=53,sex=0]
    people [id=110,name=bPeopleupdate,age=62,sex=1]
    people [id=111,name=aPeopleupdate,age=48,sex=0]
    people [id=112,name=bPeopleupdate,age=57,sex=1]
    people [id=113,name=aPeopleupdate,age=43,sex=0]
    people [id=114,name=bPeopleupdate,age=52,sex=1]
    people [id=115,name=aPeopleupdate,age=38,sex=0]
    people [id=116,name=bPeopleupdate,age=47,sex=1]
    people [id=117,name=aPeopleupdate,age=33,sex=0]
    people [id=118,name=bPeopleupdate,age=42,sex=1]
    people [id=119,name=aPeopleupdate,age=28,sex=0]
    people [id=120,name=bPeopleupdate,age=37,sex=1]
    
    

## 5总结

可以看到使用动态SQL可以大大的减轻开发的工作量，以及开发过程中的出错率。  
所以应该尽可能的使用动态SQL。