---
layout: post
title: "spring batch step详解"
date: 2020-11-26 18:45:58 +0800
categories: [batch step配置, batch step高级特性, batch step创建与定义, batch step拦截器, batch step事务]
description: "本文详细介绍了 Spring Batch 中 Step 的配置与使用方法，包括 Step 的组成、抽象与继承、执行拦截器等内容。探讨了 Tasklet 和 Chunk 模式的配置细节，并深入讲解了事务管理、异常处理、重试策略等高级特性。"
keywords: batch step配置, batch step高级特性, batch step创建与定义, batch step拦截器, batch step事务
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/110200113
> - 发布时间：2020-11-26 18:45:58
> - 阅读量：3
> - 分类：spring同时被 2 个专栏收录, 订阅专栏, spring batch
> - 标签：#batch step配置, #batch step高级特性, #batch step创建与定义, #batch step拦截器, #batch step事务

## 摘要

文章浏览阅读3.4k次，点赞3次，收藏16次。本文详细介绍了 Spring Batch 中 Step 的配置与使用方法，包括 Step 的组成、抽象与继承、执行拦截器等内容。探讨了 Tasklet 和 Chunk 模式的配置细节，并深入讲解了事务管理、异常处理、重试策略等高级特性。

---

#### spring batch step详解

  * Step 配置
  *     * step 抽象与继承
    * step 执行拦截器
    *       * step 拦截器定义
      * step执行拦截器
      * step组合拦截器
      * step拦截器注解
  * Tasklet 配置
  *     * 重启Step
    * 事务
    * 事务回滚
    * 多线程Step
    * 自定义Tasklet
  * Chunk配置
  *     * 提交间隔
    * 异常跳过
    * Step重试
    * Chunk完成策略
    * 读、处理事务
  * 拦截器
  *     * ChunkListener
    * ItemReadListener
    * ItemProcessListener
    * ItemWriteListener
    * SkipListener
    * RetryListener

  
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

## Step 配置

Step表示作业中的一个完整的步骤，一个Job可以由一个或者多个Step组成。Step包含了一个实际运行的批处理任务中所有必需的信息。

step,tasklet,chunk,read,process,write的关系如图

![image-20201114171723872](https://i-blog.csdnimg.cn/blog_migrate/dcd492986ba515940ce02f7315bfdff2.png)

Step属性

![image-20201114171930898](https://i-blog.csdnimg.cn/blog_migrate/1446beb107ed22b33dacd13d2a53eb33.png)

Step的组成

![image-20201114171954893](https://i-blog.csdnimg.cn/blog_migrate/eb43468f814a8498676dc4635143b526.png)

### step 抽象与继承

step和job一样，在最新的spring boot中都是bean的方式注入spring 容器中即可，所以和类的抽象，继承相同。
    
    
    public abstract class AbsStep extends TaskletStep{
    
        protected Logger logger = LoggerFactory.getLogger(this.getClass());
    
        public AbsStep() {
            setTasklet(new Tasklet() {
                @Override
                public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                    int sum = Integer.valueOf(chunkContext.getStepContext().getJobParameters().get("time").toString());
                    while (sum > 0) {
    					System.out.println(chunkContext.getStepContext().getStepName() + " exec : " + LocalDateTime.now());
                        sum--;
                    }
                    return RepeatStatus.FINISHED;
                }
            });
        }
    
        @Override
        protected void doExecute(StepExecution stepExecution) throws Exception {
            beforeAbsExec(stepExecution);
            logger.warn("abstract before real exec ");
            super.doExecute(stepExecution);
            logger.warn("abstract after real exec ");
            afterAbsExec(stepExecution);
        }
    
        abstract void beforeAbsExec(StepExecution stepExecution);
    
        abstract void afterAbsExec(StepExecution stepExecution);
    }
    

接着创建实现类
    
    
    @EnableBatchProcessing
    public class PlayAStep extends AbsStep {
    
        public PlayAStep() {
            setName("study5-abs-play-a-step");
        }
    
        @Override
        void beforeAbsExec(StepExecution stepExecution) {
            logger.warn(" PlayAStep before exec ");
        }
    
        @Override
        void afterAbsExec(StepExecution stepExecution) {
            logger.warn(" PlayAStep after exec ");
        }
    }
    

创建另一个实现类
    
    
    @EnableBatchProcessing
    public class PlayBStep extends AbsStep {
    
        public PlayBStep() {
            setName("study5-abs-play-b-step");
        }
        @Override
        void beforeAbsExec(StepExecution stepExecution) {
            logger.warn(" PlayBStep before exec ");
        }
    
        @Override
        void afterAbsExec(StepExecution stepExecution) {
            logger.warn(" PlayBStep after exec ");
        }
    }
    

有一点需要注意，step和job不同的地方，step不能由spring容器负责创建和管理。

在官网的sample中 ，都是建议使用builder创建。在builder中，也是手动创建实例的：

![image-20201116193730378](https://i-blog.csdnimg.cn/blog_migrate/3cde7cf42cce531cff3e6ebedfae2702.png)

创建完成后，并没有交给spring容器管理。

如果我们创建了一个step，非要交给spring容器，那么，在spring容器进行初始化处理的时候，会调用AbstractStep的afterPropertiesSet.在afterPropertiesSet中，要求名字和jobRepository不为空。

我们交给spring容器创建step实例的时候，无法保证在调用afterPropertiesSet方法之前注入jobRepository。而且，交给spring容器管理的bean一般都是单例的bean。虽然step可以复用，但是因参数等原因，还是没有支持自动放入spring容器管理。而是写了stepBuilder来创建。

基于这个原理，我们实现的抽象和继承的step，也需要创建builder类用于手动创建。(这里也可以使用xml配置，或者直接手动new实例，然后将name和jobRepository等属性注入。)
    
    
    @EnableBatchProcessing
    @Component
    public class StepBuilder {
    
        @Autowired
        private JobRepository jobRepository;
    
        @Autowired
        private PlatformTransactionManager transactionManager;
    
        public AbsStep get(Class clazz) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
            Object instance = clazz.getDeclaredConstructor().newInstance();
            if (instance instanceof AbsStep) {
                AbsStep step = (AbsStep) instance;
                step.setJobRepository(jobRepository);
                step.setTransactionManager(transactionManager);
                return step;
            } else{
                throw new NoSuchMethodException("");
            }
        }
    
    }
    

接着配置jobRepository
    
    
    @Configuration
    public class JobConfig {
    
        private static final Logger logger = LoggerFactory.getLogger(JobConfig.class);
    
        @Bean
        @Autowired
        public JobRepositoryFactoryBean jobRepositoryFactoryBean(DataSource dataSource, PlatformTransactionManager transactionManager) {
            JobRepositoryFactoryBean jobRepositoryFactoryBean = new JobRepositoryFactoryBean();
            jobRepositoryFactoryBean.setTransactionManager(transactionManager);
            jobRepositoryFactoryBean.setDataSource(dataSource);
            try {
                jobRepositoryFactoryBean.afterPropertiesSet();
            } catch (Exception e) {
                logger.error("create job repository factory bean error : {}", e.getMessage());
            }
            return jobRepositoryFactoryBean;
        }
    
        @Bean
        @Primary
        @Autowired
        public JobRepository jobRepository(JobRepositoryFactoryBean jobRepositoryFactoryBean) {
            JobRepository jobRepository = null;
            try {
                jobRepository = jobRepositoryFactoryBean.getObject();
            } catch (Exception e) {
                logger.error("create job repository error : {}", e.getMessage());
            }
            return jobRepository;
        }
    
    }
    

配置job并启动
    
    
    @EnableBatchProcessing
    @Configuration
    public class JobConf {
    
        @Bean
        public String runAbsJob(JobLauncher jobLauncher, Job absJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(absJob, new JobParametersBuilder()
                    .addLong("time", 10L)
                    .addDate("date", new Date())
                    .toJobParameters());
            return "";
        }
    
        @Bean
        public Job absJob(JobBuilderFactory jobBuilderFactory, StepBuilder stepBuilder) throws InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
            return jobBuilderFactory.get("study5-abs-job")
                    .start(stepBuilder.get(PlayAStep.class))
                    .next(stepBuilder.get(PlayBStep.class))
                    .validator(new DefaultJobParametersValidator(new String[]{"time"}, new String[]{}))
                    .build();
        }
    
    }
    

启动

![image-20201116194839799](https://i-blog.csdnimg.cn/blog_migrate/ad7036ea01e4622370f2caf90de3ac52.png)

![image-20201116194855151](https://i-blog.csdnimg.cn/blog_migrate/139832cb46c0994f7bab1164c1028b83.png)

### step 执行拦截器

#### step 拦截器定义

![image-20201116195102327](https://i-blog.csdnimg.cn/blog_migrate/891a93508430b1dbb31d9d6ac25b77fb.png)

是一个标记性接口。

![image-20201116195215211](https://i-blog.csdnimg.cn/blog_migrate/38e123c90e7deadd32ec1bf26164573f.png)

#### step执行拦截器

只有两个方法

![image-20201116195254691](https://i-blog.csdnimg.cn/blog_migrate/92166f035e9daf765c1820476d4ebd4e.png)

实现自己的step执行拦截器
    
    
    @Component
    public class StepAListener implements StepExecutionListener {
        @Override
        public void beforeStep(StepExecution stepExecution) {
            System.out.println( stepExecution.getStepName() + " Step a Listener before ");
        }
    
        @Override
        public ExitStatus afterStep(StepExecution stepExecution) {
            System.out.println(stepExecution.getStepName() + " Step a Listener after ");
            return stepExecution.getExitStatus();
        }
    }
    

使用
    
    
    @EnableBatchProcessing
    @Configuration
    public class StepLisJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job lisJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(lisJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job lisJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, StepAListener stepAListener) {
            return jobBuilderFactory.get("study5--job-step-listener")
                    .start(stepBuilderFactory.get("study5-step-step-listener")
                            .tasklet(new Tasklet() {
                                @Override
                                public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                                    System.out.println(chunkContext.getStepContext().getStepName() + "exec " + LocalDateTime.now());
                                    return RepeatStatus.FINISHED;
                                }
                            }).listener(stepAListener)
                            .build()).build();
        }
    
    }
    

启动

![image-20201116200052372](https://i-blog.csdnimg.cn/blog_migrate/2f377521a5d2090f0b9fbd0130c63f74.png)

#### step组合拦截器

我们在实现一个拦截器
    
    
    @Component
    public class StepBListener implements StepExecutionListener {
        @Override
        public void beforeStep(StepExecution stepExecution) {
            System.out.println(stepExecution.getStepName() + " Step b Listener before ");
        }
    
        @Override
        public ExitStatus afterStep(StepExecution stepExecution) {
            System.out.println(stepExecution.getStepName() + " Step b Listener after ");
            return stepExecution.getExitStatus();
        }
    }
    

接着在配置step拦截器的时候，创建一个组合拦截器，并注册到step上。
    
    
    @EnableBatchProcessing
    @Configuration
    public class ComStepLisJobCOnf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher,Job comLisJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(comLisJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job comLisJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory,
                             StepAListener stepAListener, StepBListener stepBListener) {
            CompositeStepExecutionListener listener = new CompositeStepExecutionListener();
            listener.register(stepAListener);
            listener.register(stepBListener);
            return jobBuilderFactory.get("study5-step-job-com-job")
                    .start(stepBuilderFactory.get("study5-step-step-com-lis")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println(chunkContext.getStepContext().getStepName() + " exec " + LocalDateTime.now());
                            return RepeatStatus.FINISHED;
                        }
                    }).listener(listener).build()).build();
        }
    
    }
    

启动

![image-20201116201146982](https://i-blog.csdnimg.cn/blog_migrate/c32519fe47eb0ff44a79a1ffbf87717e.png)

#### step拦截器注解

![image-20201116201212744](https://i-blog.csdnimg.cn/blog_migrate/20b2a517865217dc452a9508cf5cbd78.png)

得益于注解，在step的builder中重载了listener(Object)方法，所以，我们可以在一个普通bean上使用@BeforeStep和@AfterStep注解即可：
    
    
    @Component
    public class AnnoCListener {
    
        @BeforeStep
        public void before(StepExecution stepExecution){
            System.out.println(stepExecution.getStepName() + " exec Anno Listener " + LocalDateTime.now());
        }
    
        @AfterStep
        public ExitStatus after(StepExecution stepExecution) {
            System.out.println(stepExecution.getStepName() + " exec Anno Listener " + LocalDateTime.now());
            return stepExecution.getExitStatus();
        }
    
    }
    

然后使用
    
    
    @EnableBatchProcessing
    @Configuration
    public class AnnoStepLisJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job annoStepJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(annoStepJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job annoStepJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory,AnnoStepLisJobConf annoStepLisJobConf){
            return jobBuilderFactory.get("study5-anno-listener-job")
                    .start(stepBuilderFactory.get("study5-anno-listener-step")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println(chunkContext.getStepContext().getStepName() + " exec " + LocalDateTime.now());
                            return RepeatStatus.FINISHED;
                        }
                    }).listener(annoStepLisJobConf).build()).build();
        }
    
    }
    

启动

![image-20201117090436382](https://i-blog.csdnimg.cn/blog_migrate/2acbac6bda4a81186db1b99be47635dd.png)

需要注意的是，组合拦截器可能不支持注解方式配置的拦截器。

## Tasklet 配置

tasklet元素定义任务的具体执行逻辑，执行逻辑可以自定义实现，也可以使用spring batch的chunk操作，提供了标准的读、处理、写三步操作。通过tasklet元素同样可以定义事务、处理线程、启动控制、回滚控制和拦截器等。

tasklet属性说明

![image-20201120190039152](https://i-blog.csdnimg.cn/blog_migrate/06eceabb4d5444d52115f4b16750de46.png)

![image-20201120190058170](https://i-blog.csdnimg.cn/blog_migrate/dbec2eae7a319a9221961504045b95ce.png)

tasklet下级配置

![image-20201120190307317](https://i-blog.csdnimg.cn/blog_migrate/85554807ec6e447949b3ba544d1ce2ee.png)

### 重启Step

批处理作业框架需要支持任务重新启动，批处理作业处理数据发生错误的时候，在数据修复后需要能够将当前的任务实例执行完毕。spring batch框架支持状态为非"COMPELETED"的job实例重新启动，Job实例重启时，会从当前失败的step重新开始执行.

**重启次数**

默认Step是可以重启的，重启次数由**start-limit** 控制，默认是最大值

![image-20201120200934838](https://i-blog.csdnimg.cn/blog_migrate/48010888819b2acd40376e46e998cf87.png)

![image-20201120201246256](https://i-blog.csdnimg.cn/blog_migrate/4ea377050bab8adf80aa445206f965db.png)

**重启已完成**

默认情况下，已经执行完成的step不需要重新启动，但是，在一些特殊场景下，为了保证业务操作的完整 性，需要重新启动已经完成的step。此时就需要配置**allow-start-if-complete** 属性。

![image-20201120201207607](https://i-blog.csdnimg.cn/blog_migrate/2d015bb38ee43481cc45a83012a1e9d0.png)

![image-20201120201254906](https://i-blog.csdnimg.cn/blog_migrate/5f96d211225c9c37071c4c681a46271e.png)

### 事务

spring batch框架提供了事务能力保障Job可靠的执行，能够将Job的read,process和write三者有效的控制在一起，保证操作的完整性；Job执行期间的元数据状态的持久化同样依赖事务的保证。在Job执行期间，可以配置事务管理器、事务的基本属性(包括隔离级别、传播方式、事务超时等信息)

**事务管理器**

spring batch如果需要使用事务，需要在spring容器中声明管理器。

在spring boot 的starter中，默认有一个事务管理器。

我们在配置JobRepository时，就需要用到事务管理器，使用的是默认的事务管理器。

![image-20201121123053369](https://i-blog.csdnimg.cn/blog_migrate/11f7a8f216ba3e39652ec38f7014e56c.png)

![image-20201121123147969](https://i-blog.csdnimg.cn/blog_migrate/6efe9412218e6d86139bfe5ef4fb87cc.png)

当然，我们也可以定义自己的事务管理器：

![image-20201121124159927](https://i-blog.csdnimg.cn/blog_migrate/f5655566cfe9dbecbc97be08c02050f2.png)

然后在配置Step的时候使用

![image-20201121124224054](https://i-blog.csdnimg.cn/blog_migrate/a098aad10d9ceedd461ddb7263ccbbf2.png)

也就是说，我们可以为不同的step配置不同的事务。

很明显的，可以为不同的step配置不同的事务，那么，顺带就可以配置不同的数据库。

**事务属性**

事务属性一般有两个：隔离级别和传播方式。

事务的隔离级别

![image-20201121124540362](https://i-blog.csdnimg.cn/blog_migrate/ad1660180ac242666fcc000dc96a926c.png)

事务的传播方式

| 传播方式          | 说明                           |
|---------------|------------------------------|
| REQUIRED      | 支持当前事务，如果当前没有事务，就新建一个事务      |
| SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行    |
| MANDATORY     | 支持当前事务，如果当前没有事务，就抛出异常        |
| REQUIRES_NEW  | 新建事务，如果当前存在事务，则把当前事务挂起       |
| NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起 |
| NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常      |

比如我们可以创建事务的属性的bean，然后在配置step的时候使用

![image-20201121141317686](https://i-blog.csdnimg.cn/blog_migrate/a24dc0ea6c47a0cc79f0fa4f96519ce3.png)

需要注意一点，这里不能直接配spring的名字，在spring batch中做了一层封装

![image-20201121141355394](https://i-blog.csdnimg.cn/blog_migrate/d95ed4f97285b5802bcb64d5cd90b485.png)

必须用spring batch封装的变量

![image-20201121141424976](https://i-blog.csdnimg.cn/blog_migrate/2de87309fdda8fb1ac7b62fc38f4ddc4.png)

![image-20201121125512518](https://i-blog.csdnimg.cn/blog_migrate/63a49dadda576c78fc06578ba5d4ba5d.png)

### 事务回滚

通过事务控制可以较好的保证事务任务的执行。在业务处理过程中，包括读、写、处理数据，如果发生了异常会导致事务回滚，spring batch框架提供了发生特定异常不触发事务回滚的能力。

![image-20201121134411294](https://i-blog.csdnimg.cn/blog_migrate/62681c72a7c6c3542375ca9d5cfb643c.png)

需要注意，这是chunk的特性，使用tasklet直接注入是无法使用的，而且需要调用构造器的faultTolerant来选择高级特性的构造器，然后使用skip方法指定特定的异常，不进行回滚操作，而是跳过这些异常。

### 多线程Step

Job执行默认情况使用单个线程完成任务的执行。spring batch框架支持为step配置多个线程，即可以使用多个线程并行执行一个step，可以提高step的处理速度。

![image-20201121135346608](https://i-blog.csdnimg.cn/blog_migrate/7c36160028cbab1a08a62d7eaa46f29b.png)

### 自定义Tasklet

自定义的tasklet非常简单，直接实现tasklet接口即可。在前面的例子中我们基本上都是使用自定义的tasklet的，以内部类的方式使用。

tasklet接口只有一个execute方法需要实现，返回状态表示是否执行完毕，如果返回的状态不是执行完毕，那么会重复执行，直到执行完毕，或者返回null。

![image-20201121135838191](https://i-blog.csdnimg.cn/blog_migrate/a89b9db1e475f508ff59cd26759e4862.png)

空值也表示执行完毕

![image-20201121135915339](https://i-blog.csdnimg.cn/blog_migrate/b0362d3aebd7ce35f2d534ba2f2356dd.png)

当然spring batch也提供了一些tasklet接口的实现类，供我们直接或者间接使用

![image-20201121135957426](https://i-blog.csdnimg.cn/blog_migrate/778494c328f177d574927d7347055b07.png)

我们以`CallableTaskletAdapter`为例，体验一把
    
    
    @EnableBatchProcessing
    @Configuration
    public class CallTaskJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher,Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory,Step step) {
            return jobBuilderFactory.get("call-tasklet-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("callable-tasklet-step")
                    .tasklet(callTasklet())
                    .build();
        }
    
        private Tasklet callTasklet() {
            CallableTaskletAdapter adapter = new CallableTaskletAdapter();
            Callable<RepeatStatus> callable = () -> {
                System.out.println("call");
                return RepeatStatus.FINISHED;
            };
            adapter.setCallable(callable);
            return adapter;
        }
    
    }
    

创建的callable里面只打印了一句话`call`

然后我们执行，看看会不会输出这句话

![image-20201121141610950](https://i-blog.csdnimg.cn/blog_migrate/8a221da1728c2686cd63a95feec0da34.png)

## Chunk配置

chunk才是spring batch提倡使用的。前面的tasklet里面是没有任何规则，将全部的业务代码塞进去，执行就行。而spring batch框架提倡的批处理操作被抽象为读、处理、写这样三个基本逻辑。同时，为了支持各种场景，spring batch提供了各种各样的读写组件，包括格式化文件的读写，xml文件，数据库，jms消息读写等。

chunk除了读写处理这三个基本操作外，还有一些高级特性，比如异常处理，批处理的可靠性、稳定性、异常重入的能力。还有事务提交间隔、跳过策略、重试策略、读事务队列、处理完成策略等。

![image-20201121142042510](https://i-blog.csdnimg.cn/blog_migrate/59dbc71638d27bc68d021cf8ed4b2581.png)

常见的操作包含这些

![image-20201121142131314](https://i-blog.csdnimg.cn/blog_migrate/3319b6f611e4146ce2a89ca4c9425589.png)

![image-20201121142141622](https://i-blog.csdnimg.cn/blog_migrate/f43adba719eb5a858f3a4677bdb6c4ff.png)

### 提交间隔

在spring batch的hello world程序中，使用chunk需要设置一个提交数量，也就是提交间隔。频繁的提交会降低数据库的性能，所以，批量提交是一个好的选择。

在面向批处理chunk的操作中，可以通过属性commit-interval设置read多少条记录后进行一次提交。通过设置commit-interval的间隔值，减少提交频次，提升资源利用率。

![image-20201121142535239](https://i-blog.csdnimg.cn/blog_migrate/5cc7ed610c9779361d213e30a097871a.png)

比如：
    
    
    @EnableBatchProcessing
    @Configuration
    public class ChunkTimeJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher,Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory,Step step) {
            return jobBuilderFactory.get("chunk-time-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("chunk-time-step")
                    .<Long,Long>chunk(100)
                    .reader(reader())
                    .processor(processor())
                    .writer(writer())
                    .build();
        }
    
        private ItemProcessor<Long,Long> processor(){
            return new ItemProcessor<Long, Long>() {
                @Override
                public Long process(Long item) throws Exception {
                    return item;
                }
            };
        }
    
        private ItemWriter<Long> writer() {
            return new ItemWriter<Long>() {
                @Override
                public void write(List<? extends Long> items) throws Exception {
                    System.out.println(items.size());
                }
            };
        }
    
        private ItemReader<Long> reader(){
            List<Long> longs = new ArrayList<>(1000);
            for (long i = 0; i < 1000; i++) {
                longs.add(i);
            }
            return new ListItemReader<Long>(longs);
        }
    
    }
    

我们创建了1000个数字，然后什么都不做，接着调用写，在写数据的方法中打印传输的数量，在定义step的时候，我们定义使用chunk，而且每100个数据写一次，这样的话，预期共写10次。

执行结果如下

![image-20201121143719885](https://i-blog.csdnimg.cn/blog_migrate/53c0aae6348c278be4d454e2e0a077f7.png)

### 异常跳过

在进行批处理的时候，我们无法100%保证数据全部正确，总是会有异常数据的。如果处理1000W的数据，结果因为1个数据异常，造成整个job失败，从而导致重新执行整个job,这个代价就太大了。不能因为一个数据，就否定全部的工作，不能一颗老鼠屎，坏了整个米仓。

所以，对于异常的数据，支持跳过。
    
    
    @EnableBatchProcessing
    @Configuration
    public class ChunkSkipJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("chunk-skip-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("chunk-skip-step")
                    .<Long,Long>chunk(100)
                    .reader(reader())
                    .processor(processor())
                    .writer(writer())
                    .faultTolerant()
                    .skip(LessZoreException.class)
    	            .skipLimit(1000)
                    .build();
        }
    
        private ItemProcessor<Long,Long> processor(){
            return new ItemProcessor<Long, Long>() {
                @Override
                public Long process(Long item) throws Exception {
                    // 如果数字是负数，抛出异常
                    if (item < 0) {
                        throw new LessZoreException(item + " < 0 ");
                    }
                    return item;
                }
            };
        }
    
        private ItemWriter<Long> writer() {
            return new ItemWriter<Long>() {
                @Override
                public void write(List<? extends Long> items) throws Exception {
                    System.out.println(items.size());
                }
            };
        }
    
        private ItemReader<Long> reader(){
            List<Long> longs = new ArrayList<>(1000);
            for (long i = 0; i < 1000; i++) {
                // 如果是9的倍数，那么设置为负数
                if (i%9 == 0) {
                    longs.add(i*-1);
                } else {
                    longs.add(i);
                }
            }
            return new ListItemReader<Long>(longs);
        }
    
    }
    
    

我们还是1000个数字，在读取数字的时候，如果是9的倍数，那么将数字变为负数。在处理的时候，如果数字小于0，那么抛出我们自定义的小于0异常：

![image-20201121145005699](https://i-blog.csdnimg.cn/blog_migrate/f943cfc0dcb06d6aa0eb1f2d2479752a.png)

写入还是只打印传输的数量。

需要注意一点，如果我们设置了跳过的异常，但是没有设置最多允许跳过的数量，还是会抛出异常。

现在有一个问题：1000内有多少个9的倍数？我也懒得算，直接百度

![image-20201121145126118](https://i-blog.csdnimg.cn/blog_migrate/c4b75d6bbd6b9fa93f77ca61f05b44d8.png)

111个。

那么1000 - 111 = 889个写入数据，每100个写入一次，总共应该写入9次，最后一次写入89个。

和我们预期的一样吗？

试试

![image-20201121145454529](https://i-blog.csdnimg.cn/blog_migrate/9a25a2fefc7dd8c9cc2547823ea27061.png)

哦哦，和我们的预期不一样。其实是chunk的理解不对，chunk是以read的数量进行计数，而不是write数量进行计数的。

![image-20201121142535239](https://i-blog.csdnimg.cn/blog_migrate/5cc7ed610c9779361d213e30a097871a.png)

为什么最后一个是88个？

别忘记了0

0%9还是0

如果异常数量超过允许的最大值，也会抛出异常的。

将上面的例子中允许异常的数量调整为100.现在已知会抛出112个异常，但是我们允许的是100.别忘记修改名字或者参数，已经执行完的job_instance不会重复执行。

![image-20201121145824023](https://i-blog.csdnimg.cn/blog_migrate/884605678be9ee4e33f93a43812d943f.png)

![image-20201121145922608](https://i-blog.csdnimg.cn/blog_migrate/32521a43510c0c535a02f0637e0f3446.png)

抛出了异常，异常信息提示，跳过的数量超出允许的值100.

### Step重试

step执行期间read,process,write发生的任何异常都会导致step执行失败，进而导致作业的失败。批处理作业的自动化、定时触发，有特定的执行时间窗口特性，决定了尽可能地减少Job的失败。处理任务阶段发生的异常可以让业务失败，也可以通过skip的设置，跳过部分异常；但是也有一些异常，并不是必现的，比如网络异常，现在失败了，下一次就可能成功了。所以这类异常的出现可能在下次重新操作的时候消失，数据库锁的异常在下次提交的时候，可能就释放了。对于这些场景，我们并不希望作业失败，也不希望直接跳过，而是重试几次。尽可能让job成功。

spring batch框架提供了任务重试功能，重试次数限制功能、自定义重试策略以及重试拦截器能力。分别通过`retryable-execption-classes,retry-limit,retry-policy,cache-capacity,retry-listeners`实现。

  * retryable-exception-classes:定义可以重试的异常，可以定义一组重试的异常，如果发生了定义的异常或者子类异常都会导致重试。
  * retry-limit:任务执行重试的最大次数。
  * retry-polocy:定义自定义重试策略，需要实现接口RetryPolicy
  * cache-capacity:retry-policy缓存的大小，缓存用于存放重试上下文RetryContext，如果超过配置最大值，会发生异常。
  * retry-listeners:配置重试监听器，监听器需要实现接口RetryListener.

我们创建 一个step，验证重试的功能：
    
    
    @EnableBatchProcessing
    @Configuration
    public class RetryJobConf {
    
        private AtomicInteger atomicInteger = new AtomicInteger();
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder().addLong("date", 10L).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("chunk-skip-jobx")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory, RetryLis retryLis) {
            return stepBuilderFactory.get("chunk-skip-step")
                    .<Long, Long>chunk(3)
                    .reader(reader())
                    .processor(processor())
                    .writer(writer())
                    .faultTolerant()
                    .retry(LessZoreException.class)
                    .retryLimit(3)
                    .retryPolicy(new SimpleRetryPolicy(3, Map.of(LessZoreException.class, true)))
                    .allowStartIfComplete(true)
                    .build();
        }
    
        private ItemProcessor<Long, Long> processor() {
            return new ItemProcessor<Long, Long>() {
                @Override
                public Long process(Long item) throws Exception {
                    System.out.println(" process <" + item + "> " + atomicInteger.incrementAndGet());
                    throw new LessZoreException(item + " = 0 ");
                }
            };
        }
    
        private ItemWriter<Long> writer() {
            return new ItemWriter<Long>() {
                @Override
                public void write(List<? extends Long> items) throws Exception {
                    System.out.println("writer : " + items);
                }
            };
        }
    
        private ItemReader<Long> reader() {
            int num = 10;
            List<Long> longs = new ArrayList<>(num);
            while (--num > 0) {
                longs.add((long) num);
            }
            ListItemReader<Long> longListItemReader = new ListItemReader<>(longs);
            System.out.println("reader : " + longs);
            return longListItemReader;
        }
    
    }
    
    

我们首先创建了一个读取器，用于读取[1,9]数字，读取的数字将会打印出来。

接着创建一个处理器，处理器中每一个数据都将处理失败，抛出异常，同时，创建一个线程安全的计数器，用于标记处理器进入的次数。

写入器我们打印写入器中接收到的数据。其实写入器是拿不到任何数据的，因为全部的数据都在处理器中失败了。

在step配置的时候，指定chunk模式，然后设置好读取、处理、写入器后，开启高级功能，设置重试的异常以及重试的次数。

重试的策略和重试的异常和重试的次数，2选一即可。其实我们从重试策略中就能知道，从策略包含了上述的方法。

最后开启step成功重复执行的开关即可。

执行：

![image-20201123195214255](https://i-blog.csdnimg.cn/blog_migrate/30c6adbf3e5b3e034029eba43c0b0ffc.png)

如果重试也失败了，但是我还是不想让作业失败，这个该怎么处理呢？

跳过，前面我们就讲过异常跳过。

我们无法保证重试一定成功，如果重试，最后都失败了 ，那么，还是相同的问题，不能因为1个数据，就让给整个作业失败。

对于一个step，我们既可以配置重试策略，也可以配置跳过策略。

并且重试优先于跳过。

举个例子，我们配置stepA重试3次，跳过5个。

当第一个数据来了之后，进行处理和写入，如果期间发生了异常，那么就会先进行3次重试，如果3次重试都结束了，在执行一次跳过。

当第二个数据来了之后，进行处理和写入，也在期间发生了异常，那么就会先进行3次重试，如果3次重试都结束了，在执行一次跳过。

就这样，直到数据全部处理完成，或者达到最大跳过数量。

换句话说，如果我们数据足够多，那么配置3次重试，5次跳过。总共会执行18次process，对应6个数据，前5个不会终止，后面的3次重试失败，无法跳过，作业终止。

比如：

![image-20201123200105261](https://i-blog.csdnimg.cn/blog_migrate/f26f18dda78cbdb24242fee0d771b32d.png)

执行结果

![image-20201123200235075](https://i-blog.csdnimg.cn/blog_migrate/bd66b540e82af42a7478a8ad0d9404ca.png)

### Chunk完成策略

面向chunk的操作执行期间，根据设置的提交间隔commit-interval值，当读数据达到提交间隔后，执行一次提交操作，然后重复执行chunk的读操作，知道再次达到间隔值。spring batch框架除了提供commit-interval能力外，该框架还提供了chunk完成策略能力，通过完成策略可以配置任务的提交时机，chunk完成策略的定义接口为CompletionPolicy。

![image-20201123200832158](https://i-blog.csdnimg.cn/blog_migrate/e1f3e5a703be319a92d2636ef78dd01f.png)

说明：chunk-completion-policy定义批处理完成策略，不是表示任务的完成策略，chunk执行期间是按照chunk完成策略执行批量提交的，批量提交会执行一次写操作，同时将批处理的状态数据通过JobRepository持久化。

说明：属性chunk-completion-policy和属性commit-interval不能同时存在；在chunk中至少定义这两个其中的一个。

完成策略的接口定义

![image-20201123202430597](https://i-blog.csdnimg.cn/blog_migrate/955f4cac13aab0a9bdce3f2558382bea.png)

系统提供的接口的实现

![image-20201123202401471](https://i-blog.csdnimg.cn/blog_migrate/2f98445733cbdb1bc0e6c3c586d5a3cf.png)

我们创建自己的完成策略,内部类的方式
    
    
    class MyPolicy extends CompletionPolicySupport {
        @Override
        public boolean isComplete(RepeatContext context) {
            if (context.getStartedCount() % 3 == 0) {
                return true;
            } else {
                return false;
            }
        }
    }
    

我们定义，如果有3个数据处理完成，那么就写入。

![image-20201124195838670](https://i-blog.csdnimg.cn/blog_migrate/d1960f0f08c82c22e9d7662604990b4d.png)

当然这种配置方式还有更加简单的，如果是根据完成数量进行写入，可以直接设置commit-interval进行设置。

![image-20201124195945695](https://i-blog.csdnimg.cn/blog_migrate/3138705d67b1a05ca4262d6ccca30a0a.png)

执行结果和设置chunk-completion-policy相同

![image-20201124200103528](https://i-blog.csdnimg.cn/blog_migrate/1a6179cfde0b87bf8caa3528f56bc37b.png)

如果我们同时配置了commit-interval和chunk-completion-policy，spring batch就会抛出异常。  
![image-20201124195630272](https://i-blog.csdnimg.cn/blog_migrate/4ab7c8544311d883ff6b8daa8d56d7b0.png)

会抛出这样的异常

![image-20201124195654782](https://i-blog.csdnimg.cn/blog_migrate/599954379d9ca1146a00b96b7670bf23.png)

有了commit-interval为什么还要有chunk-completion-policy呢？

因为在chunk-completion-poliocy中我们可以查询数据库，可以定义自己的的提交时机。chunk-completion-policy比commit-interval更加的灵活。

### 读、处理事务

**读事务队列**

reader-transactional-queue:是否从一个事务性的队列读取数据，当reader从JMS的消息队列获取数据的时候，这个属性才生效。说白了，这个属性主要处理的是当出现异常的时候，是否需要将消费的消息重新投递，以及消息是否可以重复消费的问题。

true表示从一个事务性的队列中读取数据，一旦发生异常会导致事务回滚，从队列中读取的数据同样会被 重新放回到队列中；false表示从一个没有事务的队列获取数据，一旦发生异常导致事务回滚，消费掉的数据不会重新放回队列。

比如

![image-20201124201747437](https://i-blog.csdnimg.cn/blog_migrate/fa663fadebb5c2672ce559d33e4c9af7.png)

默认是有事务的，就是说，如果失败，会重新放回 读取队列，重新处理。

比如：
    
    
    @EnableBatchProcessing
    @Configuration
    public class TransJobConf {
    
        private Integer integer = 0;
    
        @Bean
        public String runJob(JobLauncher jobLauncher,Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder().addLong("id", 25L).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory,Step step) {
            return jobBuilderFactory.get("trans-job-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("tran-step-step")
                    .<Integer,Integer>chunk(3)
                    .reader(reader())
                    .processor(processor())
                    .writer(writer())
                    .faultTolerant()
                    .retry(LessZoreException.class)
                    .retryLimit(10)
                    .build();
        }
    
        private ItemWriter<Integer> writer() {
            return new ItemWriter<Integer>() {
                @Override
                public void write(List<? extends Integer> items) throws Exception {
                }
            };
        }
    
        private ItemProcessor processor() {
            return new ItemProcessor<Integer, Integer>() {
                @Override
                public Integer process(Integer item) throws Exception {
                    System.out.println("item = <" + item + "> ### ");
                    if (item == 3 ) {
                        throw new LessZoreException(" it's time !");
                    }
                    return item;
                }
            };
        }
    
        private ItemReader<Integer> reader() {
            return new ItemReader<Integer>() {
                @Override
                public Integer read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                    if (integer > 10) {
                        return null;
                    }
                    return integer++;
                }
            };
        }
    }
    

这个是读取器读取0~10，处理中，当读取到3的时候，异常。

如果是重新放回队列，那么会耗尽重试次数，重试次数总共是10次，并且最终失败。因为3永远都过不去。

![image-20201125192610516](https://i-blog.csdnimg.cn/blog_migrate/cc0d5578d2fe04cd052fe9960cd82600.png)

如果使用了`readerIsTransactionalQueue`方法，则表明读取器没有事务，当发生了异常，不会重新处理，也就是不会回滚。

这样就能执行成功，因为读取到3的时候，出现异常，进行重试，但是重试的时候，处理的是4，而不是3，这个有问题的就过去了，整个作业成功，而不是失败。

![image-20201125192816812](https://i-blog.csdnimg.cn/blog_migrate/c273774f5367187f1eea180ac74a1d38.png)

执行结果如下：

![image-20201125192852803](https://i-blog.csdnimg.cn/blog_migrate/70f861b8a9b3693030d7483c8db380bb.png)

为什么是这样的？按照我们的猜测，不是4和5都应该打印出来吗？

> 整个执行过程是这样的：首先，我们定义了chunk的处理数量是3，所以，每次读取器会一口气读取3个数据(第一轮是0,1,2;第二轮是3,4,5;6,7,8;9,10;)
> 
> 接着处理器一口气处理3个数据，第一轮是OK的，第二轮第一个数字是3，失败，因为有重试次数，而且读取器不会回滚，所以就读取第三轮了6开始继续处理。
> 
> 默认读取器是有事务的，失败会回滚，处理流程和上面一样，处理3的时候异常，读取器回滚，下一次重试继续处理3.

**处理事务**

process-transactional：处理数据是否在事务中，true表示再一次chunk处理期间将process处理的结果放在缓存中，当执行重试或者跳过策略时，可以看到缓存中处理的数据，在写操作完成前可以重新执行processor；false表示在一次chunk处理期间不会将processor处理的数据放在缓存中，即processor在chunk执行期间每一条记录仅会执行一次。

事务性processor处理过程

![image-20201124202334537](https://i-blog.csdnimg.cn/blog_migrate/a246a8914060584189a284d702747313.png)

非事务性processor处理过程

![image-20201124202404145](https://i-blog.csdnimg.cn/blog_migrate/e90a613cc501597990f2d102b752bb72.png)

但是经过仔细尝试，发现这个处理事务貌似不生效。

## 拦截器

chunk操作中提供了丰富的拦截器机制，通过拦截器可以实现额外的控制能力，比如日志记录，任务跟踪，状态报告和数据传递等。

![image-20201125193842859](https://i-blog.csdnimg.cn/blog_migrate/9208e1a2630a7229882ac13b1cb7773d.png)

这些拦截器的作用域

![image-20201125193923430](https://i-blog.csdnimg.cn/blog_migrate/b947aad11d9b2942c113db03e7e212a6.png)

拦截器的执行顺序

  1. JobExecutionListener.beforeJob()
  2. StepExecutionListener.beforeStep()
  3. ChunkListener.beforeChunk()
  4. ItemReaderListener.beforeRead()
  5. ItemReaderListener.afterRead()
  6. ItemProcessListener.beforeProcess()
  7. ItemProcessListener.afterProcess()
  8. ItemWriteListener.beforeWrite()
  9. ItemWriteListener.afterWwrite()
  10. ChunkListener.afterChunk()
  11. StepExecutionListener.afterStep()
  12. JobExecutionListener.afterJob()

### ChunkListener

接口定义：

![image-20201125194524701](https://i-blog.csdnimg.cn/blog_migrate/a886178899891243b29c122f8524307a.png)

注解

![image-20201125200504024](https://i-blog.csdnimg.cn/blog_migrate/57e4641c1f7d006ecc960879689fc9e4.png)

spring batch实现

![image-20201125200526519](https://i-blog.csdnimg.cn/blog_migrate/6058b5fef0ec44239e0b13e97cb798fd.png)

比如
    
    
    public class ChunkLis implements ChunkListener {
        @Override
        public void beforeChunk(ChunkContext context) {
            System.out.println("1. chunk lis before");
        }
    
        @Override
        public void afterChunk(ChunkContext context) {
            System.out.println("9. chunk lis after");
        }
    
        @Override
        public void afterChunkError(ChunkContext context) {
            System.out.println("0. chunk lis error");
        }
    }
    

配置

![image-20201125195346892](https://i-blog.csdnimg.cn/blog_migrate/c2331d7df85de5a78aebdc38541d6496.png)

### ItemReadListener

接口定义

![image-20201125195618631](https://i-blog.csdnimg.cn/blog_migrate/576f3ab8ce09e4981af0299cb3b51c3b.png)

注解

![image-20201125200542799](https://i-blog.csdnimg.cn/blog_migrate/80fab6de59e472806fa201c3c1bc9be0.png)

定义
    
    
    public class ItemReadLis implements ItemReadListener {
        @Override
        public void beforeRead() {
            System.out.println("4. item reader before");
        }
    
        @Override
        public void afterRead(Object item) {
            System.out.println("5. item reader after");
        }
    
        @Override
        public void onReadError(Exception ex) {
            System.out.println("00. item error");
        }
    }
    

配置

![image-20201125195913016](https://i-blog.csdnimg.cn/blog_migrate/8cd7a633c4f3a2dceda4770bfa0c0f5e.png)

### ItemProcessListener

接口定义

![image-20201125195857174](https://i-blog.csdnimg.cn/blog_migrate/2bf504b7239b85c311d64259a5ab7a25.png)

注解

![image-20201125200600164](https://i-blog.csdnimg.cn/blog_migrate/044eeb1849dc8310a7a8c068b2b45170.png)

定义
    
    
    public class ItemProcessLis implements ItemProcessListener {
        @Override
        public void beforeProcess(Object item) {
            System.out.println("6. item process before");
        }
    
        @Override
        public void afterProcess(Object item, Object result) {
            System.out.println("7. item process after");
        }
    
        @Override
        public void onProcessError(Object item, Exception e) {
            System.out.println("000. item process error");
        }
    }
    

配置

![image-20201125200118588](https://i-blog.csdnimg.cn/blog_migrate/75bec8a461c9fac0aead578a02d41d72.png)

### ItemWriteListener

接口定义

![image-20201125200138072](https://i-blog.csdnimg.cn/blog_migrate/d2b69f9dd00f4f22fcc8dbce45be09f4.png)

注解

![image-20201125200619054](https://i-blog.csdnimg.cn/blog_migrate/e4877e6db7811ab8d4a1ae36afc8898e.png)

定义
    
    
    public class ItemWriteLis implements ItemWriteListener {
        @Override
        public void beforeWrite(List items) {
            System.out.println("8. item write before");
        }
    
        @Override
        public void afterWrite(List items) {
            System.out.println("9. item write after");
        }
    
        @Override
        public void onWriteError(Exception exception, List items) {
            System.out.println("00000. item write error");
        }
    }
    

配置

![image-20201125200209681](https://i-blog.csdnimg.cn/blog_migrate/31eec2b64f7d048ddbc34995ad850f94.png)

### SkipListener

接口定义

![image-20201125200323738](https://i-blog.csdnimg.cn/blog_migrate/42bf0311cd12b2cca6ea722bb7e054b3.png)

注解

![image-20201125200635067](https://i-blog.csdnimg.cn/blog_migrate/356f4f8e0063595c11a1607b376cbfbe.png)

spring batch默认实现

![image-20201125200652212](https://i-blog.csdnimg.cn/blog_migrate/d803d2efb854d35f07a31dbe35ddbe42.png)

定义
    
    
    public class SkipLis implements SkipListener {
        @Override
        public void onSkipInRead(Throwable t) {
            System.out.println(" skip read ");
        }
    
        @Override
        public void onSkipInWrite(Object item, Throwable t) {
            System.out.println(" skip write ");
        }
    
        @Override
        public void onSkipInProcess(Object item, Throwable t) {
            System.out.println(" skip process ");
        }
    }
    

配置

![image-20201125200401828](https://i-blog.csdnimg.cn/blog_migrate/a2268f2201d35c090b26414762b81cc3.png)

### RetryListener

接口定义

![image-20201125200819224](https://i-blog.csdnimg.cn/blog_migrate/5575b8303d56a814fabb8b0034b07b40.png)

方法说明

![image-20201125200835830](https://i-blog.csdnimg.cn/blog_migrate/e9c56da42ca0c6cec8191587a1074344.png)

spring batch默认实现

![image-20201125200853635](https://i-blog.csdnimg.cn/blog_migrate/65a2ab5cff6642960066bc220ec5da09.png)

定义
    
    
    public class RetryLis implements RetryListener {
    
    
        @Override
        public <T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback) {
            System.out.println(" retry open ");
            return false;
        }
    
        @Override
        public <T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
            System.out.println(" retry close ");
        }
    
        @Override
        public <T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
            System.out.println(" retry error ");
        }
    }
    

配置

![image-20201125201334486](https://i-blog.csdnimg.cn/blog_migrate/06fe507eadb0867f43b2db87e0c75075.png)

配置任务

![image-20201125201708507](https://i-blog.csdnimg.cn/blog_migrate/473e64b1da93deeb5c50393d679f22b1.png)

执行结果

![image-20201125201956793](https://i-blog.csdnimg.cn/blog_migrate/180a200e2376af8a5b42c8a91d067894.png)

为什么chunk的拦截器没有执行？

调试发现，只有处理器和写入器设置成功了

![image-20201125203910132](https://i-blog.csdnimg.cn/blog_migrate/108ffb5fa53715238b0d3b7bd1461081.png)

这里存疑，不知道为什么。

经过多次尝试，发现与设置拦截器的顺序有关：

![image-20201126183449154](https://i-blog.csdnimg.cn/blog_migrate/9e2908323b35408e8de2ceee4420555d.png)

执行结果

![image-20201126183403077](https://i-blog.csdnimg.cn/blog_migrate/c552529a7f75eb3372e1a31ad864a3f1.png)

这里还有一个坑：

![image-20201126183522116](https://i-blog.csdnimg.cn/blog_migrate/7692353c791d128d93a0cb38ccd575ba.png)

拦截器必须返回true才会真的重试

![image-20201126183556559](https://i-blog.csdnimg.cn/blog_migrate/6547b210e094170ba8beb08c0336ac53.png)

而且有多个重试拦截器，必须每一个都返回true，只要有任意一个返回false就不会重试。

![image-20201126183700439](https://i-blog.csdnimg.cn/blog_migrate/95cae9daedba501dfad4dbfd518b5682.png)
