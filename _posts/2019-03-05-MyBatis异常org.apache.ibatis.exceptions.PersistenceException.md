---
layout: post
title: "MyBatis异常org.apache.ibatis.exceptions.PersistenceException"
date: 2019-03-05 19:14:51 +0800
categories: [MyBatis异常, MyBatis映射异常, PersistenceException异常, MyBatis映射传入参数异常, MyBatis传入基本类型异常]
description: "本文详细解析了在MyBatis测试映射文件时遇到的PersistenceException异常，特别是关于在if条件和#{}"
keywords: MyBatis异常, MyBatis映射异常, PersistenceException异常, MyBatis映射传入参数异常, MyBatis传入基本类型异常
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88197523
> - 发布时间：2019-03-05 19:14:51
> - 阅读量：1
> - 分类：java同时被 3 个专栏收录, 订阅专栏, mybatis, MyBatis
> - 标签：#MyBatis异常, #MyBatis映射异常, #PersistenceException异常, #MyBatis映射传入参数异常, #MyBatis传入基本类型异常

## 摘要

文章浏览阅读1.8w次，点赞5次，收藏13次。 本文详细解析了在MyBatis测试映射文件时遇到的PersistenceException异常，特别是关于在if条件和#{}

---

MyBatis测试映射文件时，非常的容易出现org.apache.ibatis.exceptions.PersistenceException异常。

解决方法：使用_parameter获取值。

完整的异常信息如下:
    
    
    org.apache.ibatis.exceptions.PersistenceException: 
    ### Error querying database.  Cause: org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'id' in 'class java.lang.Integer'
    ### Cause: org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'id' in 'class java.lang.Integer'
    	at org.apache.ibatis.exceptions.ExceptionFactory.wrapException(ExceptionFactory.java:30)
    	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:150)
    	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:141)
    	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectOne(DefaultSqlSession.java:77)
    	at com.client.Main.queryCard(Main.java:16)
    	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    	at java.lang.reflect.Method.invoke(Method.java:498)
    	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:47)
    	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
    	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:44)
    	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
    	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:271)
    	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:70)
    	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:50)
    	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:238)
    	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:63)
    	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:236)
    	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:53)
    	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:229)
    	at org.junit.runners.ParentRunner.run(ParentRunner.java:309)
    	at org.eclipse.jdt.internal.junit4.runner.JUnit4TestReference.run(JUnit4TestReference.java:50)
    	at org.eclipse.jdt.internal.junit.runner.TestExecution.run(TestExecution.java:38)
    	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:459)
    	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:675)
    	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:382)
    	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.main(RemoteTestRunner.java:192)
    Caused by: org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'id' in 'class java.lang.Integer'
    	at org.apache.ibatis.reflection.Reflector.getGetInvoker(Reflector.java:395)
    	at org.apache.ibatis.reflection.MetaClass.getGetInvoker(MetaClass.java:163)
    	at org.apache.ibatis.reflection.wrapper.BeanWrapper.getBeanProperty(BeanWrapper.java:162)
    	at org.apache.ibatis.reflection.wrapper.BeanWrapper.get(BeanWrapper.java:49)
    	at org.apache.ibatis.reflection.MetaObject.getValue(MetaObject.java:122)
    	at org.apache.ibatis.scripting.xmltags.DynamicContext$ContextMap.get(DynamicContext.java:93)
    	at org.apache.ibatis.scripting.xmltags.DynamicContext$ContextAccessor.getProperty(DynamicContext.java:106)
    	at org.apache.ibatis.ognl.OgnlRuntime.getProperty(OgnlRuntime.java:2719)
    	at org.apache.ibatis.ognl.ASTProperty.getValueBody(ASTProperty.java:114)
    	at org.apache.ibatis.ognl.SimpleNode.evaluateGetValueBody(SimpleNode.java:212)
    	at org.apache.ibatis.ognl.SimpleNode.getValue(SimpleNode.java:258)
    	at org.apache.ibatis.ognl.ASTNotEq.getValueBody(ASTNotEq.java:50)
    	at org.apache.ibatis.ognl.SimpleNode.evaluateGetValueBody(SimpleNode.java:212)
    	at org.apache.ibatis.ognl.SimpleNode.getValue(SimpleNode.java:258)
    	at org.apache.ibatis.ognl.ASTAnd.getValueBody(ASTAnd.java:61)
    	at org.apache.ibatis.ognl.SimpleNode.evaluateGetValueBody(SimpleNode.java:212)
    	at org.apache.ibatis.ognl.SimpleNode.getValue(SimpleNode.java:258)
    	at org.apache.ibatis.ognl.Ognl.getValue(Ognl.java:493)
    	at org.apache.ibatis.ognl.Ognl.getValue(Ognl.java:457)
    	at org.apache.ibatis.scripting.xmltags.OgnlCache.getValue(OgnlCache.java:46)
    	at org.apache.ibatis.scripting.xmltags.ExpressionEvaluator.evaluateBoolean(ExpressionEvaluator.java:32)
    	at org.apache.ibatis.scripting.xmltags.IfSqlNode.apply(IfSqlNode.java:34)
    	at org.apache.ibatis.scripting.xmltags.MixedSqlNode.apply(MixedSqlNode.java:33)
    	at org.apache.ibatis.scripting.xmltags.TrimSqlNode.apply(TrimSqlNode.java:55)
    	at org.apache.ibatis.scripting.xmltags.MixedSqlNode.apply(MixedSqlNode.java:33)
    	at org.apache.ibatis.scripting.xmltags.DynamicSqlSource.getBoundSql(DynamicSqlSource.java:41)
    	at org.apache.ibatis.mapping.MappedStatement.getBoundSql(MappedStatement.java:293)
    	at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:81)
    	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:148)
    	... 26 more
    
    
    

对应的Mapper.xml
    
    
    <select id="selectById" parameterType="Long" resultType="card">
    		select <include refid="baseMapper.str_select_card"></include>
    		from <include refid="baseMapper.str_table_card"></include>
    		<where>
    			<if test="id != null and id != ''">
    				and id=#{id}
    			</if>
    		</where>
    	</select>
    

说一下解决方法：  
因为parameterType的值是Long，在参考的书中说MyBatis做了很多默认的别名，以及很多的类型处理器等等。我不知道是我使用的MyBatis的版本太高还是什么原因，只要是基本类型或者包装类型，在if的test以及#{}中，都不能使用书中案例id以及别名等等，只要使用一定是上面的异常。  
经过艰难的排查，最终认为是if的test以及#{}中不支持书中所写的别名等等，只能使用_parameter来获取值，所以修改后：  
Mapper.xml
    
    
    <select id="selectById" parameterType="Long" resultType="card">
    		select <include refid="baseMapper.str_select_card"></include>
    		from <include refid="baseMapper.str_table_card"></include>
    		<where>
    			<if test="_parameter != null and _parameter != ''">
    				and id=#{_parameter}
    			</if>
    		</where>
    	</select>