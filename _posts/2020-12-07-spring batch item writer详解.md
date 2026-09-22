---
layout: post
title: "spring batch item writer详解"
date: 2020-12-07 20:34:09 +0800
categories: [batch writer, ItemWriter, batch写操作, batch根据数据路由写, batch组合写和路由写]
description: "spring batch item writer详解ItemWriteItemWriterItemStream系统写组件写数据库JdbcBatchItemWriterJpaItemWriterMyBatisItemWriter组合写Item路由Writer服务复用ItemWriterAdapterPropertyExtractingDelegatingItemWriter自定义ItemWriter不可重启ItemWriter可重启ItemWriter拦截器接口Annotation执行顺序拦截器异常Merge_assertupdates"
keywords: batch writer, ItemWriter, batch写操作, batch根据数据路由写, batch组合写和路由写
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/110832092
> - 发布时间：2020-12-07 20:34:09
> - 阅读量：4
> - 分类：spring batch专栏收录该内容, 订阅专栏
> - 标签：#batch writer, #ItemWriter, #batch写操作, #batch根据数据路由写, #batch组合写和路由写

## 摘要

文章浏览阅读4.5k次，点赞2次，收藏14次。spring batch item writer详解ItemWriteItemWriterItemStream系统写组件写数据库JdbcBatchItemWriterJpaItemWriterMyBatisItemWriter组合写Item路由Writer服务复用ItemWriterAdapterPropertyExtractingDelegatingItemWriter自定义ItemWriter不可重启ItemWriter可重启ItemWriter拦截器接口Annotation执行顺序拦截器异常Merge_assertupdates

---

#### spring batch item writer详解

  * ItemWrite
  *     * ItemWriter
    * ItemStream
    * 系统写组件
  * 写数据库
  *     * JdbcBatchItemWriter
    * JpaItemWriter
    * MyBatisItemWriter
  * 组合写
  * Item路由Writer
  * 服务复用
  *     * ItemWriterAdapter
    * PropertyExtractingDelegatingItemWriter
  * 自定义ItemWriter
  *     * 不可重启ItemWriter
    * 可重启ItemWriter
  * 拦截器
  *     * 接口
    * Annotation
    * 执行顺序
    * 拦截器异常
    * Merge

  
github地址： 

https://github.com/a18792721831/studybatch.git

文章列表：

[spring batch 入门](<https://blog.csdn.net/a18792721831/article/details/109514091>)

[spring batch连接数据库](<https://blog.csdn.net/a18792721831/article/details/109546988>)

[spring batch元数据](<https://blog.csdn.net/a18792721831/article/details/109550018>)

[spring batch Job详解](<https://blog.csdn.net/a18792721831/article/details/109691602>)

[spring batch step详解](<https://blog.csdn.net/a18792721831/article/details/110200113>)

[spring batch ItemReader详解](<https://blog.csdn.net/a18792721831/article/details/110637684>)

[spring batch itemProcess详解](<https://blog.csdn.net/a18792721831/article/details/110681958>)

[spring batch itemWriter详解](<https://blog.csdn.net/a18792721831/article/details/110832092>)

[spring batch 作业流](<https://blog.csdn.net/a18792721831/article/details/111083504>)

[spring batch 健壮性](<https://blog.csdn.net/a18792721831/article/details/111404284>)

[spring batch 扩展性](<https://blog.csdn.net/a18792721831/article/details/111408460>)

## ItemWrite

spring batch通过Tasklet完成具体的任务，chunk类型的tasklet定义了标准的读、处理、写的执行步骤。ItemWriter是实现写的重要组件，spring batch框架提供了丰富的写基础设施来完成各种数据源的写入功能。

spring batch框架默认提供了丰富的Writer实现；如果不能满足需求可以快速方便地实现自定义的数据写入；对于已经存在的持久化服务，框架提供了复用现有服务的能力，避免重复开发。

spring batch框架通常针对大数据量进行处理，同时框架需要讲作业处理的状态实时地持久化到数据库中，如果读取一条记录就进行写操作或者状态数据的提交，会大量消耗系统资源，导致批处理框架性能较差。在面向批处理chunk的操作中，可以通过属性commit-interval设置read多少条记录后进行一次提交。通过设置commit-interval的间隔值，减少提交频次，降低资源使用率。

### ItemWriter

ItemWriter是Step中对资源的写处理阶段，spring batch框架已经提供了各种类型的写实现。

所有的写操作需要实现ItemWriter接口

![image-20201205161807119](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8754c3f473d484e24a421e1c1d3a0a32.png)

写操作的参数是一个List,所以通常情况下是批量写入。

### ItemStream

spring batch框架童年时提供了另外一个重要的接口ItemStream。ItemStream接口定义了写操作与执行上下文ExecutionContext交互的能力。可以将已经写的条数通过该接口存放在执行上下文ExecutionContext中(ExecutionContext中的数据在批处理commit的时候会通过JobRepository持久化到数据库中)，这样到Job发生异常重新启动Job的时候，写操作可以跳过已经成功写过的数据，继续从上次出错的地方(可以从执行上下文中获取上次成功写的位置)继续写。

ItemStream接口

![image-20201205162205841](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/89f1a6e72ec30d43961e93c61e617146.png)

open操作根据参数executionContext打开需要读取资源的stream；可以根据持久化在执行上下文executionContext中的数据重新定位需要写入记录的位置。

update操作将需要持久化的数据存放在执行上下文executionContext中

close操作关闭读取的资源

### 系统写组件

spring batch框架提供的写组件

| ItemWriter                             | 说明                                                     |
|----------------------------------------|--------------------------------------------------------|
| FlatFileItemWriter                     | 写Flat类型文件                                              |
| MultiResourceItemWriter                | 多文件写组件                                                 |
| StaxEventItemWriter                    | 写XML类型文件                                               |
| AmqpItemWriter                         | 写AMQP类型消息                                              |
| ClassifierCompositeItemWriter          | 根据Classifier路由不同的Item到特定的ItemWriter处理                  |
| HibernateItemWriter                    | 基于Hibernate方式写数据库                                      |
| IbatisBatchItemWriter                  | 基于Ibatis方式写数据库                                         |
| ItemWriterAdapter                      | ItemWriter适配器，可以复用现有的写服务                               |
| JdbcBatchItemWriter                    | 基于JDBC方式写数据库                                           |
| JmsItemWriter                          | 写JMS队列                                                 |
| JpaItemWriter                          | 基于Jpa方式写数据库                                            |
| GemfireItemWriter                      | 基于分布式数据库Gemfire的写组件                                    |
| SpELMappingGemfireItemWriter           | 基于spring表达式语言写分布式数据库Gemfire的组件                         |
| MimeMessageItemWriter                  | 发送邮件的写组件                                               |
| MongoItemWriter                        | 基于分布式文件存储的数据库MongoDB写组件                                |
| Neo4jItemWriter                        | 面向网络的数据库Neo4j的写组件                                      |
| PropertyExtractingDelegatingItemWriter | 属性抽取代理写组件：通过调用给定的spring bean方法执行写入，参数有Item中指定的属性字段作为参数 |
| RepositoryItemWriter                   | 基于spring Data的写组件                                      |
| SimpleMailMessageItemWriter            | 发送邮件的写组件                                               |
| CompositeItemWriter                    | 条目写的组合模式，支持组装多个ItemWriter                              |

## 写数据库

spring batch框架对于写数据库提供了较好的支持，包括基于JDBC和ORM的写入方式。

### JdbcBatchItemWriter

spring batch框架提供了对JDBC谢支持的组件JdbcBatchItemWriter。JdbcBatchItemWriter实现了ItemWriter接口，将Item对象转换为数据库中的记录。

JdbcBatchItemWriter对用户屏蔽了数据库访问的操作细节，且提供了批处理的特性，JdbcBatchItemWriter会批量执行一组SQL语句来提高性能，而不是逐条执行SQL语句，每次批量提交的语句数和chunk中定义的提交间隔是一致的。

JdbcBatchItemWriter关键接口

| 关键类                                        | 说明                                           |
|--------------------------------------------|----------------------------------------------|
| DataSource                                 | 提供写入数据库的数据源信息                                |
| ItemPreparedStatementSetter                | 为SQL语句中有"?"的参数提供赋值接口                         |
| ColuniMapItemPreparedStatementSetter       | 接口ItemPreparedStatementSetter的实现类，提供基于列的参数设置 |
| ItemSqlParameterSourceProvider             | 为SQL语句中有命名的参数提供赋值接口                          |
| BeanPropertyItemSqlParameterSourceProvider | 从给定的Item中根据参数名称获取Item对应的属性值作为参数              |
| NamedParameterJdbcOperations               | JdbcTemplate操作，提供执行SQL的能力                    |

JdbcBatchItemWriter关键属性

| JdbcBatchItemWriter属性          | 类型                             | 说明                        |
|--------------------------------|--------------------------------|---------------------------|
| dataSourec                     | DataSource                     | 数据源，通过该属性指定使用的数据库信息       |
| sql                            | String                         | 执行的SQL语句                  |
| itemSqlParameterSourceProvider | ItemSqlParameterSourceProvider | 为SQL语句中有命名的参数提供赋值         |
| itemPreparedStatementSetter    | ItemPreparedStatementSetter    | 为SQL语句中有"?"的参数提供赋值        |
| assertUpdates                  | Boolean                        | 当没有修改、删除一条记录时，是否抛出异常。默认抛出 |

使用JdbcBatchItemWriter至少需要配置dataSource,sql两个属性。dataSourec指定访问的数据源，sql用于指定处查询的SQL语句。

首先我们在数据库中创建表

![image-20201205171611159](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bfc748c899fee4ec557529fe25ed2f36.png)

接着创建JdbcBatchItemWriter的写入组件

![image-20201205175642396](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b4f6f9c36d199ed1818178f2f9de0ac3.png)

然后使用这个写入器，完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class JdbcBatchItemWriterJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("jdbc-batch-item-writer-step")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory, JdbcBatchItemWriter<People> writer) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("jdbc-batch-item-writer-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 20 ? null : new People(null, "name : " + atomicLong.getAndIncrement()))
                    .processor((Function<People, People>) item -> {
                        System.out.println("process : " + item);
                        return item;
                    })
                    .writer(writer)
                    .build();
        }
    
        @Bean
        public JdbcBatchItemWriter<People> writer(DataSource dataSource) {
            return new JdbcBatchItemWriterBuilder<People>()
                    .dataSource(dataSource)
                    .sql("insert into people(id,name) values(null,?)")
                    .itemPreparedStatementSetter(((item, ps) -> ps.setString(1, item.getName())))
                    .assertUpdates(false)
                    .build();
        }
    
    }
    

执行结果

![image-20201205175728974](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/574efd4ec186b91e54df913b616c2490.png)

数据库中查看结果

![image-20201205175753272](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1a9ef548eb911f305de7ece85db72e24.png)

我们这里使用的是问号，也可以使用变量名

![image-20201205175908220](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/945f5254735eab5acd3412f558c39816.png)

这次少写入点，写入10个

![image-20201205175955925](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b3ecddcdecd78276ecd9f0f1824aadf6.png)

数据库中也有了

![image-20201205180012514](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2c3d7da27383a805d212869948b697a3.png)

### JpaItemWriter

对象关系映射(Object Relational Mapping,ORM)是一种为解决面向对象与关系数据库存在的互不匹配现象的技术。简单地说，ORM是通过使用描述对象和数据库之间映射的元数据，将Java程序中的对象自动持久化到关系数据库中。

JPA(Java Persistence API)是Sun官方提出的Java持久化规范。JPA通过注解或者XML描述对象-关系表的映射关系，并将运行期的实体对象持久化到数据库中；它为Java开发人员提供了一种对象/关系映射工具来管理Java应用中的关系数据。spring batch框架对ORM类型的JPA提供了基于写的ItemWriter。

JpaItemWriter实现了ItemWriter接口，核心作用是将Item对象转换为数据库中的记录。

JpaItemWriter关键属性

| JpaItemWriter        | 类型                   | 说明                                      |
|----------------------|----------------------|-----------------------------------------|
| entityManagerFactory | EntityManagerFactory | JPA提供的实体管理器的工厂类，用于生成实体管理EntityManager对象 |

使用JpaItemWriter需要配置属性entityManagerFactory：entityManagerFactory负责创建EntityManager，EntityManager负责完成对实体的增删改查。

首先引入JPA的依赖

![image-20201205181431233](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/360dbae49d88538de74719e4270874c4.png)

接着配置Jpa的EntityManagerFactory

![image-20201207085031412](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/473772718b06cc73fd68b66f0d190026.png)

除此之外，还需要配置Jpa事务管理器(Jpa有默认的，事务级别和spring相同，做了适配)

![image-20201207085115566](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9c05ac4cdb95d6cb1e8f52b2cf662346.png)

整体代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class JpaItemWriterJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("jpa-item-writer-step")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory, JpaItemWriter<People> writer,JpaTransactionManager jpaTransactionManager) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("jpa-item-writer-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 10 ? null : new People(null, "jpa : " + atomicLong.getAndIncrement()))
                    .processor((Function<People, People>) item -> {
                        System.out.println("jpa : " + item);
                        return item;
                    })
                    .writer(writer)
                    .transactionManager(jpaTransactionManager)
                    .build();
        }
    
        @Bean
        public JpaItemWriter<People> writer(LocalContainerEntityManagerFactoryBean entityManagerFactoryBean) {
            return new JpaItemWriterBuilder<People>()
                    .entityManagerFactory(entityManagerFactoryBean.getObject())
                    .build();
        }
    
        @Bean
        public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean(DataSource dataSource, EntityManagerFactoryBuilder builder) {
            return builder
                    .dataSource(dataSource)
                    .packages(Study8ItemwriterApplication.class)
                    .build();
        }
    
        @Bean
        public JpaTransactionManager transactionManager(DataSource dataSource) {
            final JpaTransactionManager transactionManager = new JpaTransactionManager();
            transactionManager.setDataSource(dataSource);
            return transactionManager;
        }
    }
    

执行结果

![image-20201207085204404](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c90b0d05796e6c35462655ba19dccfab.png)

数据库查询

![image-20201207085219456](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1f7488ab236d4014bb43004c5b14dcf9.png)

### MyBatisItemWriter

我们常用的ORM还有MyBatis，相比JPA，MyBatis的SQL可能更加直观(xml方式)。

MyBatisItemWriter需要指定数据源和SQL。在MyBatis中配置的是SqlSessionFactory.

首先引入MyBatis的依赖

![image-20201207085246245](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/134a78413477b59ef4dc04dad8a8f458.png)

接着配置MyBatis的SqlSessionFactory

![image-20201207091441242](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/71cdcf57e934a3b451518a54283e3a18.png)

增加MyBatis的接口

![image-20201207091507759](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/37c50b1abc546cefd1e648424c598c50.png)

以及XML文件

![image-20201207091524061](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7939752a799a1e3580a82d29d346299f.png)

增加扫描的Mapper注解

![image-20201207091541289](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/eeb41429974a7d1a592a0ce613443a47.png)

设置实体的作用域

![image-20201207091602935](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/596d64c818eda97210121d31035ab902.png)

创建MyBatis的写入器

![image-20201207091628017](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c345cc9ce62e61a7a37121626a6a0d82.png)

assertUpdates默认为true，表示当1条记录都没有被插入时，会抛出异常。设置为false，则忽略这个问题 ，即使没有数据被插入，也不会抛出异常。

同样的，对于MyBatis，一定要保证Mapper先读取

![image-20201207091753651](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8c6d9f449337f044c7b1d7a8cda039c2.png)

Job的完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class MyBatisBatchItemWriterJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job, PeopleDao peopleDao) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("mybatis-batch-item-writer-step")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory, MyBatisBatchItemWriter<People> writer) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("mybatis-batch-item-writer-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 10 ? null : new People(null, "mybatis : " + atomicLong.getAndIncrement()))
                    .processor((Function<People, People>) item -> {
                        System.out.println("mybatis : " + item);
                        return item;
                    })
                    .writer(writer)
                    .build();
        }
    
        @Bean
        public MyBatisBatchItemWriter<People> writer(SqlSessionFactoryBean sqlSessionFactoryBean) throws Exception {
            return new MyBatisBatchItemWriterBuilder<People>()
                    .sqlSessionFactory(sqlSessionFactoryBean.getObject())
                    .assertUpdates(false)
                    .statementId("addPeople")
                    .build();
        }
    
        @Bean
        public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource bossDataSource) {
            SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
            factoryBean.setDataSource(bossDataSource);
            factoryBean.setTypeAliasesPackage("com.study.study8itemwriter.domain");
            return factoryBean;
        }
    
    }
    

执行结果

![image-20201207091828518](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b284290aac4d2e4f45c798be18168786.png)

数据库记录

![image-20201207091839177](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/947c759a57eecb0a66a1354f0aef91a9.png)

## 组合写

在spring batch框架中对于chunk只能配置一个ItemWriter，但在有些业务场景中需要将一个Item同时写到多个不同的资源文件中，即需要写入到多个ItemWriter中。spring batch框架提供了组合ItemWriter(CompositeItemWriter)的模式满足需求。

但是吧，经过我自己验证，对于同一个数据库，同时使用MyBtais和Jpa，似乎只有Jpa起作用。

我们在上面的例子中进行：

首先是两个Jpa的writer

![image-20201207163521980](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/473d3def63ef5c1c200a48e877ee1de3.png)

以及一个MyBatis的Writer

![image-20201207163543642](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b5c7d8f0cbe8b0c7a4de4aecb87f2497.png)

最后全部扔到组合写处理器中

![image-20201207163628356](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b119466034cbcfb6f78075576fc3c828.png)

启动

![image-20201207163647478](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/87459ad8842f22ea34cd1b5ca610696e.png)

发现数据库中只存入了5条记录，而不是预期的15条

![image-20201207163718146](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/47ed286cd2a2a6b1bf876e0062189069.png)

而多数据源，或者不同种类的数据源，是能预期写入的。

如果我们去除Jpa，只有Mybatis

![image-20201207170612018](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/79d43cf8b339f72e2abe83bc0d4e9185.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class CompositeItemWriterJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job, PeopleDao peopleDao) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("compo-batch-item-writer-step")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory, MyBatisBatchItemWriter<People> myBatisWriter1, MyBatisBatchItemWriter<People> myBatisWriter2) {
            AtomicLong atomicLong = new AtomicLong();
            CompositeItemWriter<People> writer = new CompositeItemWriterBuilder<People>().delegates(Arrays.asList(myBatisWriter1, myBatisWriter2))
                    .ignoreItemStream(true).build();
            return stepBuilderFactory.get("compo-batch-item-writer-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 5 ? null : new People(null, "compo : " + atomicLong.getAndIncrement()))
                    .processor((Function<People, People>) item -> {
                        System.out.println("compo : " + item);
                        return item;
                    })
                    .writer(writer)
                    .build();
        }
    
        @Bean
        public MyBatisBatchItemWriter<People> myBatisWriter1(SqlSessionFactoryBean sqlSessionFactoryBean) throws Exception {
            return new MyBatisBatchItemWriterBuilder<People>()
                    .sqlSessionFactory(sqlSessionFactoryBean.getObject())
                    .assertUpdates(false)
                    .statementId("addPeople")
                    .build();
        }
    
        @Bean
        public MyBatisBatchItemWriter<People> myBatisWriter2(SqlSessionFactoryBean sqlSessionFactoryBean) throws Exception {
            return new MyBatisBatchItemWriterBuilder<People>()
                    .sqlSessionFactory(sqlSessionFactoryBean.getObject())
                    .assertUpdates(false)
                    .statementId("addPeople")
                    .build();
        }
    
        @Bean
        public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource bossDataSource) {
            SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
            factoryBean.setDataSource(bossDataSource);
            factoryBean.setTypeAliasesPackage("com.study.study8itemwriter.domain");
            return factoryBean;
        }
    }
    

数据还是5条

![image-20201207170631566](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/807cade282fc42dedd624c60071cb7bb.png)

但是数据库中存入了10条

![image-20201207170653236](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f469c6c218dfaeb093b0e8281d5e18f0.png)

因为我们配置的是chunk3，所以每3条数据为一组。

## Item路由Writer

在组合写中，我们是将1条记录，写到多处。

对于每一条记录都是多处写。相当于1条记录，拷贝了好多份。

还有一些场景：对于全部的数据，我们只希望存储1份，但是根据数据的重要程度，存储在不同的设备上。

比如：奇数记录在A数据库，偶数记录在B数据库。

总体还是1份数据。

spring batch框架提供了支持Item路由写的组件ClassifierCompositeItemWriter。

![image-20201207171108204](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9d565f6ba09bac1f4f72d4d7c2d56512.png)

我们在原有代码的基础上，增加两个方法：

![image-20201207180641769](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/51cfedf0ccade55cfd044dcb03791062.png)

在真正持久化的时候，会标出是哪个方法写入的

![image-20201207180753074](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f05a9a886c590c25765e01c8bb68be30.png)

接着创建这两个写入器

![image-20201207180823601](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6921235c1eedf08b1890b353465485cf.png)

然后配置

![image-20201207180947762](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3b71db53f7720c717496d53912e7bac5.png)

当然还有其他的路由器

![image-20201207181010896](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e23bbc14fa3cb073051cbe9b96644661.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class ClassifierCompositeItemWriterJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job, PeopleDao peopleDao) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("compo-batch-item-writer-step")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory, MyBatisBatchItemWriter<People> myBatisWriter1, MyBatisBatchItemWriter<People> myBatisWriter2) {
            AtomicLong atomicLong = new AtomicLong();
            ClassifierCompositeItemWriter<People> itemWriter = new ClassifierCompositeItemWriterBuilder<People>()
                    .classifier(new BackToBackPatternClassifier<People, ItemWriter<? super People>>(people -> {
                String string = people.getName();
                return string.substring(string.lastIndexOf(":") + 1, string.length());
            }, str -> {
                Integer integer = Integer.parseInt(str.trim());
                if (integer % 2 == 0) {
                    return myBatisWriter2;
                }
                return myBatisWriter1;
            })).build();
            return stepBuilderFactory.get("compo-batch-item-writer-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 5 ? null : new People(null, "compo : " + atomicLong.getAndIncrement()))
                    .processor((Function<People, People>) item -> {
                        System.out.println("compo : " + item);
                        return item;
                    })
                    .writer(itemWriter)
                    .build();
        }
    
        @Bean
        public MyBatisBatchItemWriter<People> myBatisWriter1(SqlSessionFactoryBean sqlSessionFactoryBean) throws Exception {
            return new MyBatisBatchItemWriterBuilder<People>()
                    .sqlSessionFactory(sqlSessionFactoryBean.getObject())
                    .assertUpdates(false)
                    .statementId("addPeople1")
                    .build();
        }
    
        @Bean
        public MyBatisBatchItemWriter<People> myBatisWriter2(SqlSessionFactoryBean sqlSessionFactoryBean) throws Exception {
            return new MyBatisBatchItemWriterBuilder<People>()
                    .sqlSessionFactory(sqlSessionFactoryBean.getObject())
                    .assertUpdates(false)
                    .statementId("addPeople2")
                    .build();
        }
    
        @Bean
        public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource bossDataSource) {
            SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
            factoryBean.setDataSource(bossDataSource);
            factoryBean.setTypeAliasesPackage("com.study.study8itemwriter.domain");
            return factoryBean;
        }
    
    }
    

执行结果

![image-20201207181050915](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d3dc045df050a742f6e12e5b35c2f92c.png)

数据库中的记录

![image-20201207181104861](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2babc603c47d497cf5e711058f36e49f.png)

可以很明显的看出，奇数使用的是1写入器，偶数使用2写入器。

## 服务复用

复用现有的企业资产和服务是提高企业应用开发的快捷手段，spring batch框架的写组件提供了复用现有服务的能力，利用spring batch框架提供的ItemWriterAdapter、PropertyExtractingDelegatingItemWriter可以方便地复用业务服务、spring bean、EJB或者其他远程服务。ItemWriterAdapter代理的现有服务需要能够处理Item对象；PropertyExtractingDelegatingItemWriter代理的服务支持更复杂的参数，参数可以根据指定的属性从Item中抽取。

### ItemWriterAdapter

ItemWriterAdapter关键属性

| ItemWriterAdapter属性 | 类型       | 说明                                          |
|---------------------|----------|---------------------------------------------|
| targetObject        | Object   | 需要调用的目标服务对象                                 |
| targetMethod        | String   | 需要调用的目标操作名称                                 |
| arguments           | Object[] | 需要调用的操作参数。默认不需要传入参数，默认情况下，会将每次处理的item作为参数传入 |

首先创建一个简单服务

![image-20201207182847104](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0b6bfc807b2b369a4dff14b44ea10900.png)

接着创建相关的job使用这个Service

创建写入器，并使用

![image-20201207182931454](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/aa1c0147de8d5f0c140dc8a5c1e3de6b.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class ItemWriterAdapterJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("item-writer-adapter-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory, ItemWriterAdapter<People> writer) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("item-writer-adapter-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 5 ? null : new People(null, "name : " + atomicLong.getAndIncrement()))
                    .processor((Function<People, People>) item -> {
                        System.out.println("process : " + item);
                        return item;
                    })
                    .writer(writer)
                    .build();
        }
    
        @Bean
        public ItemWriterAdapter<People> writer(PeopleService peopleService) {
            ItemWriterAdapter writerAdapter = new ItemWriterAdapter();
            writerAdapter.setTargetObject(peopleService);
            writerAdapter.setTargetMethod("print");
            return writerAdapter;
        }
    }
    

执行结果

![image-20201207183000079](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/001b29e2c0314de150326dec5129ab5e.png)

### PropertyExtractingDelegatingItemWriter

PropertyExtractingDelegatingItemWriter代理的服务支持更加复杂的参数 ，参数可以根据指定的属性值从item中抽取(ItemWriterAdapter仅支持参数类型为具体的Item对象)。

比如增加如下服务

![image-20201207184313388](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2510923284cb4b2b3a9c52a56b61ffbf.png)

创建写入器

![image-20201207184348357](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4bdfc1932b7d6cc622d11fb13d8bb716.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class PropertyExtractingDelegatingJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("pro-item-writer-adapter-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory, PropertyExtractingDelegatingItemWriter<People> writer) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("pro-item-writer-adapter-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 5 ? null : new People(null, " " + atomicLong.getAndIncrement()))
                    .processor((Function<People, People>) item -> {
                        System.out.println("process : " + item);
                        return item;
                    })
                    .writer(writer)
                    .build();
        }
    
        @Bean
        public PropertyExtractingDelegatingItemWriter<People> writer(PeopleService peopleService) {
            PropertyExtractingDelegatingItemWriter<People> writer = new PropertyExtractingDelegatingItemWriter<>();
            writer.setTargetObject(peopleService);
            writer.setTargetMethod("printName");
            writer.setFieldsUsedAsTargetMethodArguments(new String[]{"name"});
            return writer;
        }
    
    }
    

执行结果

![image-20201207184416996](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/86d9b9d406915fd6b9949291ea592599.png)

## 自定义ItemWriter

spring batch框架提供了丰富的ItemWriter组件，当这些默认的系统组件不能满足需求时，我们可以自己实现ItemWriter接口来完成需要的业务操作。自定义实现ItemWriter非常简单，只需要实现接口ItemWriter接口。只实现ItemWriter接口的写入器不支持重启，为了支持可重启的自定义ItemWriter需要新增实现接口ItemStream。

### 不可重启ItemWriter

接口定义

![image-20201207184701772](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/74f2fe4c2ec74075cdfbe0481b8f3fd6.png)

我们实现一个自己的写入器

![image-20201207185759822](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3c11e5981f4a9a0a864a9b6b0603c11a.png)

然后使用这个写入器
    
    
    @EnableBatchProcessing
    @Configuration
    public class MyItemWriterJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("my-item-writer-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("my-item-writer-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 5 ? null : new People(null, "name : " + atomicLong.getAndIncrement()))
                    .processor((Function<People, People>) item -> {
                        System.out.println("process : " + item);
                        return item;
                    })
                    .writer(new MyItemWriter<People>())
                    .build();
        }
    
    }
    

我们写的写入器，在写包含4这个数据的时候，一定会抛出异常。

![image-20201207190044965](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ce63e34b2083cd6456fd9e88f1e5fe89.png)

在不修改参数的情况下重启

![image-20201207190217501](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b8cd4776612159c49187673f03b57b6e.png)

它还是从0开始，从头开始。不能接着上次写入失败的记录或者chunk继续写入。

初次之外，如果写入器比较简单，也可以直接使用lambda表达式写

![image-20201207191012080](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/37e206783c5f47bcf6b857ef96ab509e.png)

其结果是相同的

![image-20201207191226254](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0ef91f95dc289932a595a640d95bf190.png)

### 可重启ItemWriter

spring batch框架对job提供了可重启的能力，spring batch框架提供的写组件中，文件写入等没有事务管理的都实现了ItemStream接口。

和ItemReader不一样的是，ItemReader的系统组件基本上都实现了ItemStream接口，而ItemWriter仅有部分的系统组件实现了ItemStream接口。因为通常情况下如果写的资源本身是事务性的，那么单个写入失败，会导致真个事务失败，从而导致本批次写入失败。所以下次写入时，需要从开始重头写入。因此本身具有事务性的写操作不需要事先ItemStream就支持可重启的特性。

如果写操作本身是有状态的，为了支持可重启的特性必须实现ItemStream。

实现我们自己的可重启的写入器。

![image-20201207193706090](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/99ae7acbe212ba861ceb25bfc1c6d8c5.png)

核心写入逻辑相同，遇到4的时候异常,但是需要用到我们保存的number

![image-20201207200147357](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/538ff742cf98af85c4b7519271916751.png)  
然后使用自定义的可重启的写入器

![image-20201207193804849](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4d741318318fc53d1e5e6b1cbcc85b18.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class RestartItemWriterJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("restartv-item-writer-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("restart-item-writer-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 5 ? null : new People(null, "name : " + atomicLong.getAndIncrement()))
                    .processor((Function<People, People>) item -> {
                        System.out.println("process : " + item);
                        return item;
                    })
                    .writer(new RestartItemWriter<People>())
                    .build();
        }
    
    }
    
    

第一次执行异常

![image-20201207193830028](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4d951f36d21791f2df1ccc9f511ca81b.png)

第二次执行

![image-20201207200019839](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/28d6419aac184ffd78058fc42fff328f.png)

直接从上次失败的地方开始写入。核心实现还是我们的read方法如何处理从exectionContext中拿到的记录号。

## 拦截器

spring batch框架在ItemWriter执行阶段提供了拦截器，使得在ItemWriter执行前后能够加入自定义的业务逻辑。

### 接口

接口定义

![image-20201207200342737](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3d0d7a1d72a9fa694a64b804ee454430.png)

实现自己的写入拦截器

![image-20201207200717094](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f63479566c41a48bcedc03916ba0c21e.png)

然后使用自己的写入拦截器

![image-20201207201616598](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/af80d653edc65fb0f9d514c50f44e31d.png)

运行结果

![image-20201207201713814](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9a4ba2e3ca3568cb0da5cc9c86f1901b.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class MyItemWriterLisJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addLong("id", 2L).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("my-item-writer-lis-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("my-item-writer-lis-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 5 ? null : new People(atomicLong.getAndIncrement(), "lis"))
                    .processor((Function<People, People>) item -> {
                        System.out.println(" process : " + item);
                        return item;
                    })
                    .writer(items -> {
                        for (People people : items) {
                            if (people.getId() == 4) {
                                throw new RuntimeException("people id is 4");
                            }
                            System.out.println(" writer : " + people);
                        }
                    })
                    .listener(new MyItemWriterLis())
                    .build();
        }
    
    }
    

### Annotation

除了实现接口，还可以使用注解定义拦截器：

  * @BeanforeWriter
  * @AfterWriter
  * @OnWriterError

比如

![image-20201207201913282](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b8e9d633219aa4597271e94299bb32df.png)

使用

![image-20201207201945952](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/316f11bb062831b9c59e19e226d6d829.png)

结果

![image-20201207202332235](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/23b6ca6e0b40116a88b9d88ff23754d8.png)

### 执行顺序

配置的多个ItemWriterListener，拦截器之间的执行顺序按照配置的顺序执行。beforeWriter方法和配置顺序完全相同，afterWriter方法和配置顺序完全相反。

onWriteError方法和配置顺序相同

![image-20201207202332235](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/23b6ca6e0b40116a88b9d88ff23754d8.png)

### 拦截器异常

拦截器异常会导致整个Job异常，所以在执行自定义的拦截器的时候，需要考虑对拦截器发生的异常做处理，避免影响业务。

比如

![image-20201207202730315](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/426a1c8cbb4ed48f8e6880c162d844c4.png)

执行结果

![image-20201207202859319](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7e848389c75cdd09899399c7ff0d6989.png)

确实是因为我们抛出的异常导致失败

![image-20201207202933349](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4aa1662a712bf215408872676e2d8b37.png)

Job失败

![image-20201207202843291](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/28b7cbaa6b6deaf87407903704cc6715.png)

### Merge

spring batch框架提供了多处配置拦截器执行，可以在chunk配置，也可以在tasklet配置。而且基于step的抽象和继承，可以在子step中控制是否执行父step。

通过在子step中使用super调用父step的监听，就可以实现将父、子step的拦截器全部注册。

如果在子step中没有调用父step中注册拦截器的方法，那么父step中的拦截器就不会注册，也就不会执行。
