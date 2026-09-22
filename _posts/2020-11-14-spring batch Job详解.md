---
layout: post
title: "spring batch Job详解"
date: 2020-11-14 15:42:44 +0800
categories: [batch job配置, batch job高级特性, batch job运行与停止, batch job拦截与参数, batch job参数绑定]
description: "本文详细介绍了Spring Batch的Job调度原理，包括Job的基本配置、重启、不可重启、可重启、拦截器、JobParameters校验以及Job的抽象与继承。此外，还讨论了Job的生命周期，如作用域绑定、晚期绑定、同步与异步执行、定时任务、Web接口启动、代码和JMX终止，以及业务终止等情况。内容深入浅出，辅以代码示例，有助于读者全面理解Spring Batch的作业调度和执行机制。"
keywords: batch job配置, batch job高级特性, batch job运行与停止, batch job拦截与参数, batch job参数绑定
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/109691602
> - 发布时间：2020-11-14 15:42:44
> - 阅读量：5
> - 分类：spring同时被 2 个专栏收录, 订阅专栏, spring batch
> - 标签：#batch job配置, #batch job高级特性, #batch job运行与停止, #batch job拦截与参数, #batch job参数绑定

## 摘要

文章浏览阅读5.2k次。本文详细介绍了Spring Batch的Job调度原理，包括Job的基本配置、重启、不可重启、可重启、拦截器、JobParameters校验以及Job的抽象与继承。此外，还讨论了Job的生命周期，如作用域绑定、晚期绑定、同步与异步执行、定时任务、Web接口启动、代码和JMX终止，以及业务终止等情况。内容深入浅出，辅以代码示例，有助于读者全面理解Spring Batch的作业调度和执行机制。

---

#### spring batch Job详解

  * Job调度原理
  * Job的基本配置
  * Job重启
  *     * 不可重启Job
    * 可重启Job
  * Job拦截器
  *     * Job单个拦截器
    * Job组合拦截器
  * Job Parameters校验
  *     * 自定义的Job Parameters校验
    * 默认的Job Parameters校验
    * 组合的Job Parameters校验
  * Job抽象与继承
  *     * 抽象Job
    * 继承Job
  * 作用域绑定
  *     * 作用域
    * 参数绑定--LateBinding
    * 举例
  * Job运行
  *     * 调度作业
    * 同步异步
    * 定时任务执行
    * Web接口启动任务
  * Job 终止
  *     * 代码中终止
    * JMX终止
    * 业务终止

  
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

## Job调度原理

一个Job由1个或者多个Step组成，Step有读写处理三部分组成；Job运行期间，所有的数据通过Job Repository进行持久化，同时通过Job Launcher负责调度Job作业。

![image-20201109165536490](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c99ece23d8fb2125c77ebb23ea8b9886.png)

## Job的基本配置

Job的核心属性：

![image-20201109165640147](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6db2da93eb6cfce5726181b3833576e2.png)

Job的组成：

![image-20201109165705304](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e1c2339187c61745c89feecac5243f14.png)

## Job重启

通过设置restartable可以定义job是否可以重启。默认情况下Job是可以重启的，但是需要注意，即使配置了Job可以重启，仍然需要保证Job Instance的状态一定不为"COMPLETED".

### 不可重启Job

不可重启job的定义非常简单，只需要调用一个方法即可：

![image-20201109185217758](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f6810daf6e749ce240d4545833b0b75c.png)
    
    
        @Bean
        public Job noResJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory) {
            return jobBuilderFactory.get("job-4-no-restart")
                    .incrementer(new RunIdIncrementer())
                    .preventRestart()
                    .flow(stepBuilderFactory.get("step-4-no-restart").tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("exec!" + new SimpleDateFormat("YYYY-MM-DD HH:MM:SS").format(new Date()));
                            return RepeatStatus.CONTINUABLE;
                        }
                    }).build())
                    .end()
                    .build();
        }
    

这里创建了一个job，名字是`job-4-no-restart`，然后设置不可重启(默认是可以重启的)。这个Job非常的简单，只是不停的在打印exec!+时间。

是一个死循环。所以，需要手动停止。手动停止，在数据库中就不是"COMPLETED".

![image-20201109184850812](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4f5815a3690dd726883e3801748004c8.png)

![image-20201109184830405](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cafbe6aa2e4527e0a0f5975d5e4c097c.png)

此时状态不是"COMPLETED"，但是因为我们设置了不可重启，所以，在不修改任何代码的情况下，重新启动服务：

![image-20201109185023074](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/26fa002ac412bc72af7c43d074ca680d.png)

就会抛出JobRestartException。

### 可重启Job

Job默认是可以重启的。

我们拷贝不可重启的bean，然后去掉设置不可重启的操作。

![image-20201109185303763](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/28d7d5dd674ee2bd6a64a89158aa0581.png)

记得在调度中启动

![image-20201109185339279](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/86fed2abd255324d593848124fac57db.png)

第一次运行 还是死循环

![image-20201109185408462](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bb0a49d3fc6c84136ecdb7622ff3faee.png)

手动停止

![image-20201109185503426](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/056098a35b27d2ff3c6c62a1e33c56c0.png)

![image-20201109185441387](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7dd08f3e2112ad8bfd72c15337b3a853.png)

然后重新启动

![image-20201109185733935](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b25bea7b09c89fad4bc3d3233f62da37.png)

虽然也异常了，但是，异常非常明显不一样，这是说有一个JobExecution已经在执行了。

我们现在修改job,不要让job是一个死循环，而是当循环次数大于10次的时候，抛出异常：

![image-20201109192634158](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/98bba4044dadbb342c90a989b149efd3.png)

然后，在启动的时候修改job名字，或者传入一个参数。

![image-20201109192709983](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a19356980a6da233bb2f271a281bca5a.png)

启动在循环到第10次的时候，出现异常

![image-20201109192738791](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/69a40a5b5bd120c6ef9dec01ea43800d.png)

此时数据库中，记录的状态是FAILED.接着在不修改参数的前提下，注释掉抛出异常的代码。

![image-20201109192949280](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d92a0e2738df033c0b931bd863f57f7e.png)

因为我们没有修改Job的名字，也没有修改Job的参数，所以，在spring batch看来，这就是同一个Job Instance。

![image-20201109193306234](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1918b439d0d41ff45354f4abc90bdfcb.png)

接着重新启动，重新运行这个Job Instance

![image-20201109193253875](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fb83681b428a13ed1415676ded327ae9.png)

## Job拦截器

spring batch框架在Job执行阶段提供了拦截器，使得在Job执行前后能够加入自定义的业务逻辑处理。

### Job单个拦截器

Job 执行阶段拦截器需要实现接口：`JobExecutionListener`
    
    
    @Configuration
    public class JobListener implements JobExecutionListener {
    
        @Override
        public void beforeJob(JobExecution jobExecution) {
            System.out.println("JobListener before " + jobExecution.getJobInstance().getJobName());
        }
    
        @Override
        public void afterJob(JobExecution jobExecution) {
            System.out.println("JobListener after " + jobExecution.getExitStatus().getExitDescription());
        }
    }
    

使用
    
    
    @Configuration
    @EnableBatchProcessing
    public class LisJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job lisJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(lisJob, new JobParametersBuilder().addLong("id", 1L).toJobParameters());
            return "";
        }
    
        @Bean
        public Job lisJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, JobListener jobListener) {
            return jobBuilderFactory.get("study4-lis")
                    .start(stepBuilderFactory.get("study4-step")
                            .tasklet(new Tasklet() {
                                @Override
                                public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                                    System.out.println("exec");
                                    return RepeatStatus.FINISHED;
                                }
                            }).build())
                    .listener(jobListener)
                    .build();
        }
    
    }
    

执行结果

![image-20201109195042406](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/611b902114286bd5b022e5d34f8a32b0.png)

这里有个坑，Job的监听，只能实现接口，不能使用注解。

因为其他的监听，有Object参数的重载，而Job的监听Builder，没有Object的重载。

比如
    
    
    @Component
    public class AnnJobListener {
    
        @BeforeJob
        public void beforeJob(JobExecution jobExecution) {
            System.out.println("before " + jobExecution.getJobInstance().getJobName());
        }
    
        @AfterJob
        public void afterJob(JobExecution jobExecution) {
            System.out.println("after " + jobExecution.getJobInstance().getJobName());
        }
    
    }
    
    
    
    @EnableBatchProcessing
    @Configuration
    public class LisAnnJobConf{
    
        @Bean
        public String runJob(JobLauncher jobLauncher,Job annJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(annJob, new JobParameters());
            return "";
        }
    
        @Bean
        public Job annJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, AnnJobListener annJobListener) {
            return jobBuilderFactory.get("study4-anno")
                    .start(stepBuilderFactory.get("study-anno")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("exec");
                            return RepeatStatus.FINISHED;
                        }
                    })
                            .listener(annJobListener) // 有listerner(Object)的方法重载
                            .build())
                    // .listener(annJobListener) 没有listener(Object)的方法重载
                    .build();
        }
    
    }
    

启动

![image-20201109203534237](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7025fd23e99c2d787e0f2d8f605d2735.png)

前后操作都没有执行。

### Job组合拦截器

在Job中不尽可以配置单个的拦截器，还可以使用`CompositeJobExecutionListener`实现组合拦截器。

拦截器的顺序根据注册的先后顺序，进行拦截。

比如现在有3个拦截器
    
    
    @Component
    public class SecJobListener implements JobExecutionListener {
        @Override
        public void beforeJob(JobExecution jobExecution) {
            System.out.println("SecJobListener before : " + jobExecution.getJobInstance().getJobName());
        }
    
        @Override
        public void afterJob(JobExecution jobExecution) {
            System.out.println("SecJobListener after : " + jobExecution.getJobInstance().getJobName());
        }
    }
    
    
    
    @Component
    public class JobListener implements JobExecutionListener {
    
        @Override
        public void beforeJob(JobExecution jobExecution) {
            System.out.println("JobListener before " + jobExecution.getJobInstance().getJobName());
        }
    
        @Override
        public void afterJob(JobExecution jobExecution) {
            System.out.println("JobListener after " + jobExecution.getExitStatus().getExitDescription());
        }
    }
    
    
    
    @Component
    public class AnnJobListener implements JobExecutionListener {
    
        @Override
        public void beforeJob(JobExecution jobExecution) {
            System.out.println("AnnJobListener before " + jobExecution.getJobInstance().getJobName());
        }
    
        @Override
        public void afterJob(JobExecution jobExecution) {
            System.out.println("AnnJobListener after " + jobExecution.getJobInstance().getJobName());
        }
    
    }
    

接着，使用组合拦截器，将这三个拦截器配置给一个Job
    
    
    @EnableBatchProcessing
    @Configuration
    public class MoreLisJobConf {
    
        @Bean
        public String jobRunner(JobLauncher jobLauncher,Job moreLisJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(moreLisJob, new JobParametersBuilder().addLong("id", 1L).toJobParameters());
            return "";
        }
    
        @Bean
        public Job moreLisJob(AnnJobListener annJobListener, JobListener jobListener, SecJobListener secJobListener, JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory) {
            CompositeJobExecutionListener listener = new CompositeJobExecutionListener();
            listener.register(annJobListener);
            listener.register(jobListener);
            listener.register(secJobListener);
            return jobBuilderFactory.get("study4-more-listener")
                    .start(stepBuilderFactory.get("study4-more-listener")
                            .tasklet(new Tasklet() {
                                @Override
                                public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                                    System.out.println("exec!" + LocalTime.now().toString());
                                    return RepeatStatus.FINISHED;
                                }
                            }).build())
                    .listener(listener)
                    .build();
        }
    
    }
    

组合拦截器，是基于拦截器做了封装。组合拦截器内有一个拦截器列表。同时组合拦截器也实现了JobExecutionListener接口，当spring batch调用组合拦截器的before方法时，内部实际是调用全部的注册的拦截器的before方法；当spring batch调用组合拦截器的after方法时，内部实际倒序调用全部注册的拦截器的after方法。

执行结果

![image-20201110192447726](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1c8fe2b657ed136a100c2a4e68635b85.png)

## Job Parameters校验

spring batch框架提供了Job作业参数的校验功能，Job Parameters支持4种类型的参数：字符串、时间、长整型和双精度。但是传入的参数不一定符合这个规则，那么就需要对参数进行校验。除了类型校验，还可以有其他的校验，可以自己制定自定义的校验。需要实现接口`JobParametersValidator`。

当然spring batch也提供了一些简单的校验类，可供我们使用

![image-20201110193005900](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ccbd945f43a4fe9cf41d525a7ed7eb7e.png)

### 自定义的Job Parameters校验

首先需要实现接口校验的接口
    
    
    @Component
    public class ParameValidatory implements
            JobParametersValidator {
        @Override
        public void validate(JobParameters parameters) throws JobParametersInvalidException {
            System.out.println(parameters.getParameters());
            System.out.println("ParameValidatory + " + parameters.getClass().getSimpleName());
        }
    }
    

接着配置到Job上
    
    
    @EnableBatchProcessing
    @Configuration
    public class ParameJobConf {
    
    
        @Bean
        public String runJob(JobLauncher jobLauncher,Job parJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(parJob,new JobParametersBuilder().addLong("id", 2L).toJobParameters());
            return "";
        }
    
        @Bean
        public Job parJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, ParameValidatory parameValidatory) {
            return jobBuilderFactory.get("study4-parame")
                    .validator(parameValidatory)
                    .start(stepBuilderFactory.get("study4-parame")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("exec!"+ LocalTime.now());
                            return RepeatStatus.FINISHED;
                        }
                    }).build()).build();
        }
    
    }
    

启动

![image-20201110194643254](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6c012dd30b8413c12bbbddb4bc93c163.png)

为什么会被调用两次呢？

我们打个断点调试下

第一次调用在SimpleJobLauncher中的调用：

![image-20201110195027760](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/39180e2356c0c8536f896a7435ebf60e.png)

第二次是在AbstractJob中调用的

![image-20201110195108218](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7a884b7a8ed81da2e638fc0d61403516.png)

在JobLauncher真正运行的时候，会在执行线程地方启动后，调用Job的execut方法中调用AbstractJob中的validate方法。

### 默认的Job Parameters校验

spring batch框架默认提供了Job ParametersValidator的实现。

定义使用默认Job Parameters校验的Job
    
    
    @Configuration
    @EnableBatchProcessing
    public class DefParJobConnf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher,Job defParJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(defParJob, new JobParametersBuilder().addLong("id", 2L).toJobParameters());
            return "";
        }
    
        @Bean
        public Job defParJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory) {
            return jobBuilderFactory.get("study4-def-par")
                    .start(stepBuilderFactory.get("study4-def-par")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("exec ! " + LocalTime.now().toString());
                            return RepeatStatus.FINISHED;
                        }
                    }).build())
                    .validator(new DefaultJobParametersValidator(new String[]{"id"}, new String[]{"id"}))
                    .build();
        }
    
    }
    

正确运行了

![image-20201110201025978](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/69a8a383d43c38cfa941d47a21432e87.png)

我们要求参数校验，id必填，name可选。

![image-20201110201518394](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2fd720e97d598d70df93c81350e829cd.png)

接着将id传入空，name 传入非空

![image-20201110201559779](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a49417523fb4855823c02eca147ddbdd.png)

验证通过

![image-20201110201709782](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3ffb8fbbc1aab7528aa58dfb60b4e034.png)

如果一个参数都没有呢？

![image-20201110202000259](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1ec3994b605ea2d5d48d14ba4b2443c7.png)

抛出了Job Parameters验证异常，缺失必须的参数：id.

### 组合的Job Parameters校验

spring 框架还提供了组合校验器`CompositeJobparametersValidator`
    
    
    @Configuration
    @EnableBatchProcessing
    public class CompParJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job compParJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(compParJob, new JobParametersBuilder().toJobParameters());
            return "";
        }
    
        @Bean
        public Job compParJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory, ParameValidatory parameValidatory) {
            CompositeJobParametersValidator validator = new CompositeJobParametersValidator();
            validator.setValidators(Arrays.asList(parameValidatory, new DefaultJobParametersValidator(new String[]{"id"}, new String[]{"name"})));
            return jobBuilderFactory.get("study4-comp-par")
                    .start(stepBuilderFactory.get("study4-comp-par")
                            .tasklet(new Tasklet() {
                                @Override
                                public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                                    System.out.println("exec! " + LocalTime.now().toString());
                                    return RepeatStatus.FINISHED;
                                }
                            }).build())
                    .validator(validator)
                    .build();
        }
    
    }
    

我们定义了两个Job Parameters校验器，其中一个是必须有id，可选的name。自定义的Job Parameters校验器则是打印参数列表。

第一次，我们什么参数都不传：

![image-20201110203508467](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/10ada31597d4167821616cdeba3dbcf5.png)

提示少id参数

接着我们传入id参数

运行成功

![image-20201110203608269](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b4e4ae7a9cf37ba26422722e23f08a23.png)

增加name参数，也是可以运行成功的

![image-20201110203705625](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5beaae62c6cd33ffaf4475a2fd793b04.png)

需要注意，如果传入了必选参数和可选参数之外的参数，则验证失败(只针对默认的参数校验器，自定义的根据自己的规则校验)

![image-20201110203831262](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/af09106599a0afcc4453b23d77e62a48.png)

运行

![image-20201110203900886](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4a0ed1258104c62078430167ea90a07f.png)

参数校验异常，提示date参数既不在必选参数，也不在可选参数。

## Job抽象与继承

### 抽象Job

在spring batch中，job等同于bean。只要将job这个bean注入到容器中即可。

所以i，创建抽象Job就是创建抽象类。
    
    
    public abstract class AbsSimpleJob extends SimpleJob {
    
        protected final Logger log = LoggerFactory.getLogger(this.getClass());
    
        abstract void beforeExec();
    
        @Override
        protected void doExecute(JobExecution execution) throws JobInterruptedException, JobRestartException, StartLimitExceededException {
            beforeExec();
            log.info(" do execute before");
            super.doExecute(execution);
            log.info(" do execute after");
            afterExec();
        }
    
        abstract void afterExec();
    }
    

我们创建了一个抽象的Job，这个抽象的Job继承于SimpleJob，SimpleJob是一个空实现AbstractJob的子类。

在抽象Job中，我们定义了前置和后置操作。在前置后置之间执行真正的操作。

因为AbsSimpleJob是一个抽象类，是无法实例化为bean实例，然后注入到容器中的，所以，需要创建子类，子类实现了AbsSimpleJob,并且，注入到容器中。

第一个子类，PlaySimpleJob
    
    
    public class PlaySimpleJob extends AbsSimpleJob {
        @Override
        void beforeExec() {
            log.info(" play simple before ");
        }
    
        @Override
        void afterExec() {
            log.info(" play simple after ");
        }
    }
    

PlaySimpleJob子类，实现了父类定义的前置后置操作。

前置后置操作也是非常的简单，只是打印一句话。

接着，我们创建第二个子类SleepSimpleJob
    
    
    public class SleepSimpleJob extends AbsSimpleJob {
        @Override
        void beforeExec() {
            log.info(" sleep simple before ");
        }
    
        @Override
        void afterExec() {
            log.info(" sleep simple after ");
        }
    }
    

SleepSimpleJob中的操作和PlaySimpleJob中的操作相同，都是打印日志。

目前为止，我们创建了1个抽象类，2个抽象类的子类。

那么，抽象的job和实现的job如何使用呢？

我们创建job配置类
    
    
    @Configuration
    @EnableBatchProcessing
    public class AbsJobConf {
    
        @Bean
        public String runAbsJob(PlaySimpleJob playSimpleJob, SleepSimpleJob sleepSimpleJob, JobLauncher jobLauncher) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(playSimpleJob, new JobParametersBuilder().addLong("id", 2L).toJobParameters());
            jobLauncher.run(sleepSimpleJob, new JobParametersBuilder().addLong("id", 2L).toJobParameters());
            return "";
        }
    
        @Bean
        public PlaySimpleJob playSimpleJob(JobRepository jobRepository, Step absStep) {
            PlaySimpleJob playSimpleJob = new PlaySimpleJob();
            playSimpleJob.setName("study4-play-simple-job");
            playSimpleJob.setJobRepository(jobRepository);
            playSimpleJob.addStep(absStep);
            return playSimpleJob;
        }
    
        @Bean
        public SleepSimpleJob sleepSimpleJob(JobRepository jobRepository, Step absStep) {
            SleepSimpleJob sleepSimpleJob = new SleepSimpleJob();
            sleepSimpleJob.setName("study4-sleep-simpleo-job");
            sleepSimpleJob.setJobRepository(jobRepository);
            sleepSimpleJob.addStep(absStep);
            return sleepSimpleJob;
        }
    
        @Bean
        public Step absStep(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-abs-step")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("exec ! " + LocalTime.now());
                            return RepeatStatus.FINISHED;
                        }
                    }).build();
    
        }
    
    }
    

我们创建了一个step，然后使用注解，将实现了AbsSimpleJob的子类，注入到容器中。需要注意装配Job。如果Job在内部自己已经实现装配，那么我们在注入到容器中时，只需要注入JobRepository属性即可。

最后调度。

![image-20201111162706365](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d15dd9af450c9ba5f144b6a173c15b39.png)

### 继承Job

在抽象Job中就知道，Job就是一个Bean，那么就可以实现继承。

比如我们新增PlayGameSimpleJob继承PlaySimpleJob
    
    
    public class PlayGameSimpleJob extends PlaySimpleJob {
    
        public PlayGameSimpleJob() {
            this("play-game-simple-job");
        }
    
        public PlayGameSimpleJob(String name) {
            setName(name);
        }
    
    }
    

然后调度即可
    
    
    @EnableBatchProcessing
    @Configuration
    public class ExJobConf {
    
        @Bean
        public String runExJob(JobLauncher jobLauncher, Job playGameSimpleJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(playGameSimpleJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public PlayGameSimpleJob playGameSimpleJob(JobRepository jobRepository, Step playGameStep) {
            PlayGameSimpleJob playGameSimpleJob = new PlayGameSimpleJob("study4-play-game-job");
            playGameSimpleJob.setJobRepository(jobRepository);
            playGameSimpleJob.setSteps(Arrays.asList(playGameStep));
            return playGameSimpleJob;
        }
    
        @Bean
        public Step playGameStep(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-play-game-step")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println(" play game step ");
                            return RepeatStatus.FINISHED;
                        }
                    }).build();
        }
    }
    

启动

![image-20201111163957358](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0b27090859a804810c70f1706a81199e.png)

## 作用域绑定

### 作用域

Scope用来声明Ioc容器中对象的存活空间，即在Ioc容器在对象进入相应的Scope之前，生成并装配这些对象，在该对象不再处于这些Scope的限定范围之后，容器通常会销毁这些对象。

StepScope是spring batch框架提供的自定义的Scope，将spring bean定义为StepScope，支持spring bean在Step开始的时候初始化，在Step结束的时候销毁spring bean，将spring bean的生命周期与Step绑定。

JobScope是spring batch框架提供的自定义的Scope，将spring bean定义为JobScope，支持spring bean在Job开始的时候初始化，在Job结束的时候销毁spring bean，将spring bean的生命周期与Job绑定。

### 参数绑定–LateBinding

在之前配置job的时候，job的名字，step的名字，以及一些参数等等，都是直接写死的。

这样非常不利于扩展，而且也不符合实际，很多东西，是只有在运行的时候，才知道的。

spring batch可以通过属性后绑定的技术，支持在运行期间获取属性的值。

spring batch框架通过特定的表达式支持为Job或者Step关联的实体使用后绑定技术。在StepScope和JobScope中spring batch框架提供的可以使用的实体包括jobParameters,jobExecutionContext,stepExecutionContext.

### 举例
    
    
    @EnableBatchProcessing
    @Configuration
    public class ScoJobConf {
    
        private AtomicInteger atomicInteger = new AtomicInteger();
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addString("stepName", "study4-sco-step")
                    .addString("name", "   sco   ")
                    .addLong("time", 10L)
                    .addString("writeMessage", " write msg ")
                    .addString("processMessage", " process msg")
                    .addDate("date", new Date())
                    .toJobParameters());
            return "";
        }
    
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("study4-sco-job")
                    .start(step)
                    .validator(new DefaultJobParametersValidator(new String[]{"stepName", "name", "time"}, new String[]{"writeMessage", "processMessage", "date"}))
                    .build();
        }
    
        @Bean
        @JobScope
        public Step step(ItemReader<String> reader, ItemProcessor<String, String> processor, ItemWriter<String> writer, StepBuilderFactory stepBuilderFactory, @Value("#{jobParameters['stepName']}") String stepName) {
            return stepBuilderFactory.get(stepName)
                    .<String, String>chunk(10)
                    .reader(reader)
                    .processor(processor)
                    .writer(writer)
                    .build();
        }
    
        @Bean
        @StepScope
        public ItemProcessor<String, String> processor(@Value("#{jobParameters['processMessage']}") String processMessage) {
            return new ItemProcessor<String, String>() {
                @Override
                public String process(String item) throws Exception {
                    System.out.println(processMessage + item);
                    return item;
                }
            };
        }
    
        @Bean
        @StepScope
        public ItemWriter<String> writer(@Value("#{jobParameters['writeMessage']}") String writeMessage) {
            return new ItemWriter<String>() {
                @Override
                public void write(List<? extends String> items) throws Exception {
                    items.stream().forEach(x -> System.out.println(writeMessage + x));
                }
            };
        }
    
        @Bean
        @StepScope
        public ItemReader<String> reader(@Value("#{jobParameters['name']}") String name, @Value("#{jobParameters['time']}") int time) {
            ItemReader reader = new ItemReader<String>() {
                @Override
                public String read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                    if (atomicInteger.incrementAndGet() > 20) {
                        return null;
                    }
                    return name;
                }
            };
            return reader;
        }
    
    }
    

运行

![image-20201112193329361](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/46af417c84204dd631b0effe362922d4.png)

数据库记录

![image-20201112193357874](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/863d84c7b63820e53a74f66d4fc1cd93.png)

## Job运行

spring batch框架提供一组执行job的接口。包括JobLauncher，JobExplorer和JobOperator三个操作Job的接口。

![image-20201112193639458](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a3a31dc5a78a5300932425c667241749.png)

JobLauncher是最常用的作业调度器，通过给定的Job Name和Job Parameters可以执行Job；JobExplorer主要负责从JobRepository中获取执行的信息，包括获取作业实例、获取作业执行器、获取正在运行的作业执行器，获取作业列表等操作；JobOperator包含了JobLauncher和JobExplorer中的大部分操作。

### 调度作业

配置好的Job需要调度才能运行，一般可以通过外部框架，结合使用，调度Job运行。

当然，也可以使用JobLauncher调度Job
    
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addString("stepName", "study4-sco-step")
                    .addString("name", "   sco   ")
                    .addLong("time", 10L)
                    .addString("writeMessage", " write msg ")
                    .addString("processMessage", " process msg")
                    .addDate("date", new Date())
                    .toJobParameters());
            return "";
        }
    

### 同步异步

默认情况下，JobLauncher的run操作通过同步方式调用Job，任何调用Job的客户端需要等待Job的执行结果返回后才能结束。

![image-20201112195010781](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3fa7f1019e9bdb50e2ff93b8ee6d42b7.png)

比如
    
    
    @EnableBatchProcessing
    @Configuration
    public class LauJobConf {
    
        @Bean
        public String runLauJob(JobLauncher jobLauncher,Job lauJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            System.out.println("start " + LocalDateTime.now());
            jobLauncher.run(lauJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            System.out.println("over " + LocalDateTime.now());
            return "";
        }
    
    
    
        @Bean
        public Job lauJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory) {
            return jobBuilderFactory.get("study4-lau-job")
                    .start(stepBuilderFactory.get("study4-lau-step")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            TimeUnit.SECONDS.sleep(10);
                            return RepeatStatus.FINISHED;
                        }
                    }).build())
                    .build();
        }
    
    }
    

这就是同步执行，任务中的线程睡眠，调度操作也会被阻塞

![image-20201112200426332](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/48ba876f218d89052b4b435211e2c5dd.png)

![image-20201112200536996](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ae2670c3b6cf2cca82cdf3482f0379f8.png)

同步操作的优势在于作业一旦执行完毕，调用客户端能够立刻收到返回值。但在实际的使用中，往往Job的执行所需时间太长，不能一直等下去。

所以，为了提高资源利用率，需要使用异步的方式调用Job。

JobLauncher提供了异步执行Job的能力。

比如
    
    
    @EnableBatchProcessing
    @Configuration
    public class SyncJobConf {
    
        @Bean
        public String runLauJob(JobLauncher jobLauncher, Job lauJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            System.out.println("start " + LocalDateTime.now());
            jobLauncher.run(lauJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            System.out.println("over " + LocalDateTime.now());
            return "";
        }
    
        @Bean
        public JobLauncher jobLauncher(JobRepository jobRepository) {
            SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
            jobLauncher.setJobRepository(jobRepository);
            jobLauncher.setTaskExecutor(new ConcurrentTaskExecutor());
            return jobLauncher;
        }
    
        @Bean
        public Job lauJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory) {
            return jobBuilderFactory.get("study4-lau-job")
                    .start(stepBuilderFactory.get("study4-lau-step")
                            .tasklet(new Tasklet() {
                                @Override
                                public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                                    TimeUnit.SECONDS.sleep(10);
                                    return RepeatStatus.FINISHED;
                                }
                            }).build())
                    .build();
        }
    
    }
    

执行结果

![image-20201112203419237](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a5dff53363aff9dbe41ecd2a26079e59.png)

### 定时任务执行

spring 中也有一个轻量级的调度器，所以，我们可以借助spring中的调度，实现任务的执行。

首先开启调度

![image-20201114130927310](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/efb8d0c0761be33ede371c6316d59d0a.png)

接着创建调度执行的目标job
    
    
    @EnableBatchProcessing
    @Configuration
    @Component
    public class SchJobConf {
    
        @Autowired
        private JobLauncher jobLauncher;
    
        @Autowired
        @Qualifier("schJob")
        private Job schJob;
    
        @Scheduled(cron = "* * * * * *")
        @Autowired
        public void runJob() throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(schJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
        }
    
        @Bean
        public JobLauncher jobLauncher(JobRepository jobRepository) {
            SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
            jobLauncher.setJobRepository(jobRepository);
            ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
            threadPoolTaskScheduler.setPoolSize(3);
            threadPoolTaskScheduler.initialize();
            jobLauncher.setTaskExecutor(threadPoolTaskScheduler);
            return jobLauncher;
        }
    
        @Bean
        public Job schJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory){
            return jobBuilderFactory.get("study4-sch-job")
                    .start(stepBuilderFactory.get("study4-sch-step")
                    .tasklet(new Tasklet() {
                        @Override
                        public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                            System.out.println("study-sch-execute");
                            return RepeatStatus.FINISHED;
                        }
                    }).build())
                    .build();
        }
    
    }
    

在这里我们主要做了三个操作：

  1. 通过注解，向spring容器中注入了我们的目标操作schJob。
  2. 因为调度是异步执行，所以，执行器需要初始化线程池，所以我们给jobLauncher初始化了线程池
  3. 拥有`@Scheduled`注解的调度方法，需要注意，调度方法不能有参数，所以，将调度方法中用到的执行器和任务，全部以属性的方式注入。

接着启动微服务，就会每一秒中执行一次任务：

![image-20201114131301537](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0fe24096789376062d49c47ef164660b.png)

### Web接口启动任务

通过上述几种方式，我们可以看出，不管是同步还是异步，不管是定时任务，还是命令行，还是Web应用，其实际上都是调用JobLauncher去启动任务。

所以，Web接口启动的原因也非常的简单，通过暴露Controller接口，在Controller接口中，调用jobLauncher启动任务。

## Job 终止

在前面体验异步、定时任务、Web接口启动任务的代码中，启动之后就无法停止了，只能通过关闭JVM的方式，来停止Job的调度。

但是，关闭JVM的方式来终止Job，终归不是一个好的退出方式。

所以，spring batch提供了停止正在运行Job的能力，通过接口JobOperator中的stop方法实现。

### 代码中终止

首先创建一个执行步的操作
    
    
        class StopTasklet implements Tasklet {
    
            @Override
            public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                int sum = 10;
                while (sum > 0) {
                    TimeUnit.SECONDS.sleep(1);
                    System.out.println("stop step " + chunkContext.getStepContext().getStepName() + " exec " + LocalDateTime.now());
                    sum--;
                }
                return RepeatStatus.FINISHED;
            }
        }
    

我们的执行步中的操作，就是每隔1秒，打印执行步的名字和时间。

接着创建三个执行步
    
    
        @Bean
        public Step stopStep1(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-stop-step-1")
                    .tasklet(new StopTasklet()).build();
        }
    
        @Bean
        public Step stopStep2(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-stop-step-2")
                    .tasklet(new StopTasklet()).build();
        }
    
        @Bean
        public Step stopStep3(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-stop-step-3")
                    .tasklet(new StopTasklet()).build();
        }
    

接着创建job
    
    
        @Bean
        public Job stopJob(JobBuilderFactory jobBuilderFactory, Step stopStep1, Step stopStep2, Step stopStep3) {
            return jobBuilderFactory.get("study4-stop-job")
                    .start(stopStep1)
                    .next(stopStep2)
                    .next(stopStep3)
                    .build();
        }
    

因为是异步启动，所以需要给JobLauncher注入线程池
    
    
        @Bean
        public JobLauncher jobLauncher(JobRepository jobRepository) {
            SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
            jobLauncher.setJobRepository(jobRepository);
            jobLauncher.setTaskExecutor(new ConcurrentTaskExecutor());
            return jobLauncher;
        }
    

接着调度执行job
    
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job stopJob, JobOperator jobOperator) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException, InterruptedException, NoSuchJobException, NoSuchJobExecutionException, JobExecutionNotRunningException {
            jobLauncher.run(stopJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            TimeUnit.SECONDS.sleep(15);
            jobOperator.stop(jobOperator.getRunningExecutions(stopJob.getName()).iterator().next());
            return "";
        }
    

在执行的方法中，我们注入了JobOperator接口。

在job中，我们有三个执行步，每个执行步10秒，每个1秒会打印执行步的名字和时间。

在调度中，我们主线程调度成功了job后，会暂时睡眠15秒，此时执行步正好执行在第二个执行步。

因为job终止的最小单位就是执行步，如果我们停止的时候，job正好执行在执行步中，那么是不会立刻终止的，而是等当前执行步执行完成后，终止。

所以，在第15秒的时候，终止job，job会将第二个执行部执行完成，然后终止job。

也就是预期结果中，第三个执行步不会执行。

![image-20201114141543826](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6884614b25a878ef7d2844b9631d1863.png)

![image-20201114141556465](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e3cf608c37158132cfd20ea448061945.png)

第三个执行步没有执行，在数据库中查询job执行结果，也是终止

![image-20201114141655638](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a5731ee4f01e735903421e7fcf438fed.png)

### JMX终止

我们创建一个和代码中终止的相同的job
    
    
        @Bean
        public Job stopJob(JobBuilderFactory jobBuilderFactory, Step stopStep1, Step stopStep2, Step stopStep3) {
            return jobBuilderFactory.get("study4-jmx-job")
                    .start(stopStep1)
                    .next(stopStep2)
                    .next(stopStep3)
                    .build();
        }
    
    
        @Bean
        public Step stopStep1(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-jmx-step-1")
                    .tasklet(new StopTasklet()).build();
        }
    
        @Bean
        public Step stopStep2(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-jmx-step-2")
                    .tasklet(new StopTasklet()).build();
        }
    
        @Bean
        public Step stopStep3(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-jmx-step-3")
                    .tasklet(new StopTasklet()).build();
        }
    
        class StopTasklet implements Tasklet {
    
            @Override
            public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                int sum = 30;
                while (sum > 0) {
                    TimeUnit.SECONDS.sleep(1);
                    System.out.println("stop step " + chunkContext.getStepContext().getStepName() + " exec " + LocalDateTime.now());
                    sum--;
                }
                return RepeatStatus.FINISHED;
            }
        }
    

使用异步启动
    
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job stopJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(stopJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public JobLauncher jobLauncher(JobRepository jobRepository) {
            SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
            jobLauncher.setJobRepository(jobRepository);
            jobLauncher.setTaskExecutor(new ConcurrentTaskExecutor());
            return jobLauncher;
        }
    

然后将JobOperator暴露为jmx操作
    
    
        @Bean
        public MBeanExporter mBeanExporter(JobOperator jobOperator) {
            MBeanExporter exporter = new MBeanExporter();
            exporter.setBeans(Map.of("com.study.study4.job:name=jobOperator", jobOperator));
            return exporter;
        }
    

接着启动

![image-20201114150410786](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4a8deaf5f620535ec19c07941b0fdc3f.png)

然后在控制台启动jconsole

[图片]![image-20201114153932286](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e247869af4474e217cfc4db7f735468c.png)

并连接我们的应用

![image-20201114150514267](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8115b270e4228210bae72c92ca405c74.png)

接着在数据库中找到我们的job的execution

![image-20201114150544528](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ddd4e33823430a76811fe75ded95612e.png)

等待程序执行到第30~60秒期间

调用jobOperatoer的stop操作，传入job_execution_id

![image-20201114150638727](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/70d967843746274fbc63a2f86e002360.png)

此时程序会将第30~60秒的stopStep2执行完毕，但是不会执行stopStep3

![image-20201114150730297](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/12de4d240a6a21dfc59ca0c934bebc18.png)

![image-20201114150737608](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8f41ef53b7c5c31e7ec7b2eca0984aff.png)

此时在查询数据库

![image-20201114150758617](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4977ad054835325f71b377f89fabf84f.png)

job_execution就是stop状态了。

### 业务终止

当业务代码出现异常时，通过业务判断，job不能再继续执行下去(越执行，错误越多)

此时业务代码通过调用StepExecution.setTerminateOnly()，发送一个停止消息给框架，一旦spring batch框架接收到停止消息，并且框架获取到作业的控制权，spring batch就会终止job.

我们继续以代码中终止的job为例
    
    
    @EnableBatchProcessing
    @Configuration
    public class BusJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job stopJob) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(stopJob, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public JobLauncher jobLauncher(JobRepository jobRepository) {
            SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
            jobLauncher.setJobRepository(jobRepository);
            jobLauncher.setTaskExecutor(new ConcurrentTaskExecutor());
            return jobLauncher;
        }
    
        @Bean
        public Job stopJob(JobBuilderFactory jobBuilderFactory, Step stopStep1, Step stopStep2, Step stopStep3) {
            return jobBuilderFactory.get("study4-jmx-job")
                    .start(stopStep1)
                    .next(stopStep2)
                    .next(stopStep3)
                    .build();
        }
    
    
        @Bean
        public Step stopStep1(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-jmx-step-1")
                    .tasklet(new StopTasklet())
                    .build();
        }
    
        @Bean
        public Step stopStep2(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-jmx-step-2")
                    .tasklet(new StopTasklet())
                    .build();
        }
    
        @Bean
        public Step stopStep3(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("study4-jmx-step-3")
                    .tasklet(new StopTasklet())
                    .build();
        }
    
        class StopTasklet implements Tasklet {
    
            @Override
            public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                int sum = 30;
                if (chunkContext.getStepContext().getStepName().equals("study4-jmx-step-2")) {
                    chunkContext.getStepContext().getStepExecution().setTerminateOnly();
                }
                while (sum > 0) {
                    TimeUnit.SECONDS.sleep(1);
                    System.out.println("stop step " + chunkContext.getStepContext().getStepName() + " exec " + LocalDateTime.now());
                    sum--;
                }
                return RepeatStatus.FINISHED;
            }
        }
    
    }
    

但是做了一个小改动，在执行步的操作中，业务代码中，判断当前如果是第二个执行步，那么调用停止操作，发送停止消息给spring batch框架。

需要记住的是，spring batch调度的单位是执行步，而且需要框架得到作业的控制权。

我们在第二个执行步中发送消息，但是第二个执行步已经进入，所以，需要等第二个执行步执行完成，框架得到控制权，就会终止job，不会执行第三个执行步。

启动

![image-20201114152832209](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/858182b3fb6879c42f6a1ae5547e76c7.png)

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/614e4728a73b840960029e37b7ece361.png)

第三个执行步不会执行，而且日志中也显示，得到了停止消息。

我们可以在异常拦截器中，发送停止消息，从而终止job.
