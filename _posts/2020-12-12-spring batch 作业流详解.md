---
layout: post
title: "spring batch 作业流详解"
date: 2020-12-12 18:49:59 +0800
categories: [批处理Flow, 条件分支Step, step流程控制, 并行执行step, 批处理job的停止]
description: "spring batch 作业流详解顺序Flow条件Flow条件Flwo配置ExitStatus & BatchStatusdecision并行FlowStep Job Flow关系FlowFlowStepJobStep数据共享终止Jobendstopfailgithub地址：https://github.com/a18792721831/studybatch.git文章列表：spring batch 入门spring batch连接数据库spring batch元数据spring b_springbatch详解"
keywords: 批处理Flow, 条件分支Step, step流程控制, 并行执行step, 批处理job的停止
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/111083504
> - 发布时间：2020-12-12 18:49:59
> - 阅读量：2
> - 分类：spring batch专栏收录该内容, 订阅专栏
> - 标签：#批处理Flow, #条件分支Step, #step流程控制, #并行执行step, #批处理job的停止

## 摘要

文章浏览阅读2.3k次，点赞4次，收藏6次。spring batch 作业流详解顺序Flow条件Flow条件Flwo配置ExitStatus & BatchStatusdecision并行FlowStep Job Flow关系FlowFlowStepJobStep数据共享终止Jobendstopfailgithub地址：https://github.com/a18792721831/studybatch.git文章列表：spring batch 入门spring batch连接数据库spring batch元数据spring b_springbatch详解

---

#### spring batch 作业流详解

  * 顺序Flow
  * 条件Flow
  *     * 条件Flwo配置
    * ExitStatus & BatchStatus
    * decision
  * 并行Flow
  * Step Job Flow关系
  *     * Flow
    * FlowStep
    * JobStep
  * 数据共享
  * 终止Job
  *     * end
    * stop
    * fail

  
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

## 顺序Flow

顺序Flow是指在Job中定义多个Step，每个Step之间按照定义好的顺序执行，任何一个Step的失败都会导致Job的失败。

![image-20201209185108457](https://i-blog.csdnimg.cn/blog_migrate/8aa9c6ccde9943f8a813caede4bd23b2.png)

比如

我们定义这样的4个step

![image-20201209192410564](https://i-blog.csdnimg.cn/blog_migrate/c537ce116606e4c926f027a590572d2c.png)

定义完成之后，在job中使用

![image-20201209192441494](https://i-blog.csdnimg.cn/blog_migrate/9af03f5dd59c607a8c986eaac2f74b8d.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class MyFlowStepJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory) {
            return jobBuilderFactory.get("my-flow-step-job")
                    .start(step1(stepBuilderFactory))
                    .next(step2(stepBuilderFactory))
                    .next(step3(stepBuilderFactory))
                    .next(step4(stepBuilderFactory))
                    .build();
        }
    
        @Bean
        public Step step1(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("my-flow-step-step1")
                    .tasklet((sc, cc) -> {
                        System.out.println(sc.getStepExecution().getStepName() + " run >>>>> ");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    
        @Bean
        public Step step2(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("my-flow-step-step2")
                    .tasklet((sc, cc) -> {
                        System.out.println(sc.getStepExecution().getStepName() + " run >>>>> ");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    
        @Bean
        public Step step3(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("my-flow-step-step3")
                    .tasklet((sc, cc) -> {
                        System.out.println(sc.getStepExecution().getStepName() + " run >>>>> ");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    
        @Bean
        public Step step4(StepBuilderFactory stepBuilderFactory) {
            return stepBuilderFactory.get("my-flow-step-step4")
                    .tasklet((sc, cc) -> {
                        System.out.println(sc.getStepExecution().getStepName() + " run >>>>> ");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    
    }
    

执行结果

![image-20201209193128082](https://i-blog.csdnimg.cn/blog_migrate/e3b6fd2605519ece228d766cbefb8bb2.png)

## 条件Flow

更多的业务场景需要根据作业步的执行结果决定后续调用哪个作业步，而不是像上面就事先定义好了作业的执行顺序。spring batch框架提供了条件Flow来满足有选择的执行作业步的功能。

![image-20201209194201598](https://i-blog.csdnimg.cn/blog_migrate/f2f1a351b9c7b6533157e86f93e6e74f.png)

### 条件Flwo配置

条件Flow配置的关键方法

| 方法   | 说明                                                                                                                                                   |
|------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| on   | 定义作业步的ExitStatus(退出状态）和 on属性指定的值匹配的时候，则执行to指定的作业步。on属性的值可以是任意的字符串，同时支持通配符"_”、"?""_ ":表示退出状态为任何值都满足。"?":表示匹配一个字符，如 c?t，当作业的退出状态为cat 的时候满足，如果是cabt则不满足 |
| to   | 当前Step执行完成后，to属性元素指定下个需要执行的Step                                                                                                                      |
| from | 进行匹配的时候，匹配哪一个step的退出状态                                                                                                                               |
| end  | 配置结束，返回构造器构造。                                                                                                                                        |

比如我们创建5个step

![image-20201209200620686](https://i-blog.csdnimg.cn/blog_migrate/34a287ffcb1dbf7fd3a18afeb530c52b.png)

接着我们创建job的时候，指定，当0执行完后执行1,1执行完后执行3.如果0执行的有误，执行2，如果1执行有误，执行4。

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class ChangeFlowStepJobConf {
    
        @Autowired
        private StepBuilderFactory stepBuilderFactory;
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory) {
            Step[] st = new Step[5];
            Step step = stepBuilderFactory.get("change-flow-step-6").tasklet((sc, cc) -> {
                System.out.println(sc.getStepExecution().getStepName() + " run >>>>> ");
                sc.setExitStatus(ExitStatus.FAILED);
                return RepeatStatus.FINISHED;
            }).build();
            steps().toArray(st);
            return jobBuilderFactory.get("change-flow-step-job")
                    .start(step)
                    // 配置规则 on:配置匹配的字符串; to:匹配后执行; from:匹配谁
                    // 简单来说 from 谁 on 配置了 to 执行
                    // 建议使用 from-on-to
                    // 否则这个关系很容易配错
                    // step -> "COMPLETED" -> st[1]
                    .on(FlowExecutionStatus.COMPLETED.getName()).to(st[1])
                    // step -> "FAILED" -> st[0]
                    .from(step).on(FlowExecutionStatus.FAILED.getName()).to(st[0])
                    // st[0] -> "COMPLETED" -> st[1]
                    .from(st[0]).on("COMPLETED").to(st[1])
                    // step -> "*" -> st[2]
                    .from(step).on("*").to(st[2])
                    // st[1] -> "COMPLETED" -> st[3]
                    .from(st[1]).on(FlowExecutionStatus.COMPLETED.getName()).to(st[3])
                    // st[1] -> "*" -> st[4]
                    .from(st[1]).on("*").to(st[4])
                    .end()
                    .build();
        }
    
        private List<Step> steps() {
            return Arrays.asList(0, 1, 2, 3, 4).stream().map(
                    x -> stepBuilderFactory.get("change-flow-step-" + x).tasklet((sc, cc) -> {
                        System.out.println(sc.getStepExecution().getStepName() + " run >>>>> ");
                        return RepeatStatus.FINISHED;
                    }).build())
                    .collect(Collectors.toList());
        }
    
    }
    

COMPLETED

FAILED

COMPLETED

*

COMPLETED

*

step 

st1 

st0 

st2 

st3 

st4 

执行结果

![image-20201212125925833](https://i-blog.csdnimg.cn/blog_migrate/df37a205ddd4ce490e3015d31494f4be.png)

这么写，虽然也不错，但是，判断的必须是指定的字符串。所以写起来还是比较麻烦的。

所以，我们挑战一个更高难度的。

0

1

2

3

4

1

2

3

4

2

3

4

3

4

4

y

n

s 

s0 

s1 

s2 

s3 

s4 

y 

n 

假设我们的分支如此的复杂，真是闲的。

首先定义两个方法，用于创建step

![image-20201212134613208](https://i-blog.csdnimg.cn/blog_migrate/325f2830e404065d2b22557168e6b5ba.png)

接着统一创建step

![image-20201212134648263](https://i-blog.csdnimg.cn/blog_migrate/78c5472a40bf657f5fbfb5dd424cefe7.png)

然后就是配置规则

![image-20201212134707809](https://i-blog.csdnimg.cn/blog_migrate/455910bd6b83a1e0cf314bb015df0db0.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class ChangesFlowStepJobConf {
    
        @Autowired
        private StepBuilderFactory stepBuilderFactory;
    
        @Autowired
        private JobBuilderFactory jobBuilderFactory;
    
        @Autowired
        private JobLauncher jobLauncher;
    
        @PostConstruct
        public void runJob() throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job(), new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
        }
    
        private Job job() {
            Map<Integer, Step> temps = new HashMap<>();
            temps.put(-1, getStep(null, "0"));
            temps.put(0, getStep(0, "1"));
            temps.put(1, getStep(1, "2"));
            temps.put(2, getStep(2, "3"));
            temps.put(3, getStep(3, "4"));
            temps.put(4, getStep(4, "y"));
            return jobBuilderFactory.get("changes-flow-step-job")
                    // s 开始节点,返回 0
                    .start(temps.get(-1))
                    // s -> "0" -> s0
                    .on("0").to(temps.get(0))
                    // s -> "1" -> s1
                    .from(temps.get(-1)).on("1").to(temps.get(1))
                    // s -> "2" -> s2
                    .from(temps.get(-1)).on("2").to(temps.get(2))
                    // s -> "3" -> s3
                    .from(temps.get(-1)).on("3").to(temps.get(3))
                    // s -> "4" -> s4
                    .from(temps.get(-1)).on("4").to(temps.get(4))
                    // s0 -> "1" -> s1
                    .from(temps.get(0)).on("1").to(temps.get(1))
                    // s0 -> "1" -> s1
                    .from(temps.get(0)).on("2").to(temps.get(2))
                    // s0 -> "1" -> s1
                    .from(temps.get(0)).on("3").to(temps.get(3))
                    // s0 -> "1" -> s1
                    .from(temps.get(0)).on("4").to(temps.get(4))
                    // s1 -> "2" -> s2
                    .from(temps.get(1)).on("2").to(temps.get(2))
                    // s1 -> "3" -> s3
                    .from(temps.get(1)).on("3").to(temps.get(3))
                    // s1 -> "4" -> s4
                    .from(temps.get(1)).on("4").to(temps.get(4))
                    // s2 -> "3" -> s3
                    .from(temps.get(2)).on("3").to(temps.get(3))
                    // s2 -> "4" -> s4
                    .from(temps.get(2)).on("4").to(temps.get(4))
                    // s3 -> "4" -> s4
                    .from(temps.get(3)).on("4").to(temps.get(4))
                    // s4 -> "y" -> y
                    .from(temps.get(4)).on("y").to(getSteps("y", FlowExecutionStatus.COMPLETED.getName()))
                    // s4 -> "n" -> n
                    .from(temps.get(4)).on("n").to(getSteps("n", FlowExecutionStatus.FAILED.getName()))
                    .end()
                    .build();
        }
    
        private Step getStep(Integer x, String result) {
            return stepBuilderFactory.get("s" + (x == null ? "" : x))
                    .tasklet((sc, cc) -> {
                        System.out.println(sc.getStepExecution().getStepName() + " run  >>>>>>>  ");
                        sc.setExitStatus(new ExitStatus(result));
                        return RepeatStatus.FINISHED;
                    }).build();
        }
    
        private Step getSteps(String x, String result) {
            return stepBuilderFactory.get(x)
                    .tasklet((sc, cc) -> {
                        System.out.println(sc.getStepExecution().getStepName() + " run  >>>>>>>  ");
                        sc.setExitStatus(new ExitStatus(result));
                        return RepeatStatus.FINISHED;
                    }).build();
        }
    }
    

执行结果

![image-20201212134751546](https://i-blog.csdnimg.cn/blog_migrate/de76b6cb57a457dc6b821c9956038777.png)

### ExitStatus & BatchStatus

Spring Batch框架有两个重要的状态，一个是退出状态（ExitStatus)，另一个是批处理状态（BatchStatus）。

**BatchStatus**

批处理状态通常由批处理框架使用,用来记录Job或者Step的执行情况,在Job或者Step的重启中，批处理的状态起到关键作用（Job或者Step 的重启状态判断使用的就是BatchStatus)。可以通过Job Execution和Step Execution 获取当前Job、Step 的批处理状态。

JobExecution.getStatus()操作可以获取作业Job 的批处理状态。

StepExecution.getStatus()操作可以获取作业步Step的批处理状态。

批处理的状态是枚举类型，目前框架定义了8种类型，分别是完成COMPLETED、启动中STARTING、已启动STARTED、停止中STOPPING、已停止STOPPED、失败FAILED、废弃ABANDONED和未知UNKOWN。

| 状态        | 说明                                     |
|-----------|----------------------------------------|
| COMPLETED | 表示完成状态，所有的Step都标记为COMPLETED后，Job会处于此状态 |
| STARTING  | 表示作业正在启动状态，还没有启动完毕                     |
| STARTED   | 表示作业已经成功启动                             |
| STOPPING  | 表示作业正在停止中                              |
| STOPPED   | 表示作业停止完成                               |
| FAILED    | 表示作业执行失败                               |
| ABANDONED | 表示当前下次重启Job 时候需要废弃掉的 Step，即不会被再次执行     |
| UNKOWN    | 表示未知的状态，该状态下重启Job会抛出错误                 |

批处理状态在Job或者Step执行期间通过Job上下文或者Step上下文持久化到DB中，可以在表 batch _job_execution查看Job的批处理状态（字段STATUS表示该状态），在表batch_step_execution查看Step 的批处理状态（字段STATUS表示该状态）。

**ExitStatus**

退出状态表示Step执行后或者Job执行后的状态，该状态值可以被修改，通常用于条件Flow中。可以通过拦截器StepExecutionListener 的 afterStep()操作来修改退出状态的值。

退出状态在Job或者Step执行期间通过Job上下文或者Step上下文持久化到DB中，可以在表batch _job_execution查看Job 的退出状态（字段EXIT_CODE表示该状态），在表batch_step_execution查看Step 的退出状态（字段EXIT_CODE表示该状态）。

### decision

在条件Flow中，我们根据ExitStatus的字符串，进行匹配，然后决定执行哪一个step。但是这样有一个不好的地方，在step中混杂了step控制的代码。

也就是说，step不纯粹了，step中不仅仅有业务代码，还有了流程控制代码。step就不是单纯的step了，step不仅仅要干业务的代码，还需要处理流程控制的代码。

因此，在ExitStatus之外，有了FlowExecutionStatus。专门用于step流程控制。也就是JobExecutionDecider。JobExecutionDecider不仅仅可以用于控制step，还可以用于控制job。

JobExecutionDecider接口只有一个方法

![image-20201212140506304](https://i-blog.csdnimg.cn/blog_migrate/f86ebac70814bcc327ea006e058a63c3.png)

也就是说，我们可以使用lambda创建接口内容对象。

有了JobExecutionDecider，我们就可以将业务代码和流程控制代码分离。

在step中就是业务代码。在JobExecutionDecider中就是流程控制代码。

使用

跳转

使用

跳转

使用

跳转

使用

跳转

使用

跳转

s0 

dec0 

s1 

dec1 

s2 

dec2 

s3 

dec3 

s4 

dec4 

y 

在理解上，可以认为我们将一部分step的进行了特殊化，只处理流程跳转，而且，将这些step做了简化。比如JobExecutionDecider不需要我们传入名字，不需要JobRepository等等。实际上，在Builder里面，进行了处理，在Builder里面，JobExecutionDecider和Step是相等的。JobExecutionDecider的名字也是由Builder来指定的。

![image-20201212143148432](https://i-blog.csdnimg.cn/blog_migrate/b246c4bf7d52ea5614a447a96b3d9594.png)

![image-20201212143215930](https://i-blog.csdnimg.cn/blog_migrate/a16c536fc7e150b60e57ae29fbe395b6.png)

多说无益，我们还是以事实说话

创建两个方法，分别用于创建step和JobExecutionDecider

![image-20201212143342125](https://i-blog.csdnimg.cn/blog_migrate/4d1ae6c2c33e837dab4ea206fed9e005.png)

接着统一创建step和JobExecutionDecider（真正使用中，这些step和JobExecutionDecider可能非常的复杂，而且每一个都不一样）

![image-20201212143412812](https://i-blog.csdnimg.cn/blog_migrate/f362168a86e9bc6d29643acc8f297fe4.png)

接着定义关系

![image-20201212143532873](https://i-blog.csdnimg.cn/blog_migrate/72399b5e8bff3af89abaf59bbf6d3a0a.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class DeciFlowJobConf {
    
        @Autowired
        private StepBuilderFactory stepBuilderFactory;
    
        @Autowired
        private JobBuilderFactory jobBuilderFactory;
    
        @Autowired
        private JobLauncher jobLauncher;
    
        @PostConstruct
        public void runJob() throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job(), new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
        }
    
        private Job job() {
            Map<Integer, Step> temps = new HashMap<>();
            temps.put(-1, getStep("s"));
            temps.put(0, getStep("s0"));
            temps.put(1, getStep("s1"));
            temps.put(2, getStep("s2"));
            temps.put(3, getStep("s3"));
            temps.put(4, getStep("s4"));
            temps.put(5, getStep("y"));
            temps.put(6, getStep("n"));
            Map<Integer, JobExecutionDecider> deciderMap = new HashMap<>();
            deciderMap.put(-1, getDec("start"));
            deciderMap.put(0, getDec("1"));
            deciderMap.put(1, getDec("2"));
            deciderMap.put(2, getDec("3"));
            deciderMap.put(3, getDec("4"));
            deciderMap.put(4, getDec("y"));
            deciderMap.put(5, getDec("n"));
            return jobBuilderFactory.get("changes-flow-step-job")
                    // step[s] 开始节点
                    .start(temps.get(-1))
                    // dec[start] 开始节点的控制节点
                    .start(deciderMap.get(-1))
                    // dec[start] -> step[s0]
                    .from(deciderMap.get(-1)).on("start").to(temps.get(0))
                    // dec[*] -> step[n]
                    .from(deciderMap.get(-1)).on("*").to(temps.get(6))
                    // step[s0] -> dec[0]
                    .from(temps.get(0)).on("*").to(deciderMap.get(0))
                    // dec[0] -> step[s1]
                    .from(deciderMap.get(0)).on("1").to(temps.get(1))
                    // step[s1] -> dec[1]
                    .from(temps.get(1)).on("*").to(deciderMap.get(1))
                    // dec[1] -> step[s2]
                    .from(deciderMap.get(1)).on("2").to(temps.get(2))
                    // step[s2] -> dec[2]
                    .from(temps.get(2)).on("*").to(deciderMap.get(2))
                    // dec[2] -> step[s3]
                    .from(deciderMap.get(2)).on("3").to(temps.get(3))
                    // step[s3] -> dec[3]
                    .from(temps.get(3)).on("*").to(deciderMap.get(3))
                    // dec[3] -> step[s4]
                    .from(deciderMap.get(3)).on("4").to(temps.get(4))
                    // step[s4] -> dec[4]
                    .from(temps.get(4)).on("*").to(deciderMap.get(4))
                    // dec[4] -> step[y]
                    .from(deciderMap.get(4)).on("y").to(temps.get(5))
                    .end()
                    .build();
        }
    
        private Step getStep(String x) {
            return stepBuilderFactory.get(x)
                    .tasklet((sc, cc) -> {
                        System.out.println(sc.getStepExecution().getStepName() + " run  >>>>>>>  ");
                        return RepeatStatus.FINISHED;
                    }).build();
        }
    
        private JobExecutionDecider getDec(String result) {
            return (je,se) -> {
                if (se == null) {
                    System.out.println(je.getJobInstance().getJobName() + " -> " + result);
                    return new FlowExecutionStatus(result);
                } else {
                    System.out.println(se.getStepName() + " status : " + se.getExitStatus().getExitCode() + " -> " + result);
                    return new FlowExecutionStatus(result);
                }
            };
        }
    
    }
    

执行结果

![image-20201212143623277](https://i-blog.csdnimg.cn/blog_migrate/f95f31199bb32031bddbc0eb9744547d.png)

## 并行Flow

批处理任务中有些任务有先后的执行顺序，还有些Step没有先后执行顺序的要求，可以在同一时刻并行作业，批处理框架提供了并行处理Step 的能力，通过Split元素可以定义并行的作业流，为split定义执行的线程池，从而提高Job的执行效率。

Spring Batch框架提供了split元素来执行并行作业的能力。

split元素关键属性

| 属性            | 说明                                       | 默认值                                  |
|---------------|------------------------------------------|--------------------------------------|
| id            | 定义split的唯一ID，全局需要保证 id 唯一                |                                      |
| task-executor | 任务执行处理器，定义后表示采用多线程执行任务，需要考虑多线程执行任务时候的安全性 | 如果不定义的话，默认使用同步线程执行器:SyncTaskExecutor |
| next          | 当前split中所有的flow执行完毕后，接下来执行的Step          |                                      |

split元素

| 属性   | 说明                                                    |
|------|-------------------------------------------------------|
| flow | 用来定义并行处理的作业，并列的flow表示可以并行处理的任务; split元素下面可以定义多个flow节点 |
| next | 根据退出状态定义下一步需要执行的Step                                  |
| stop | 根据退出状态决定是否退出当前的任务，同时Job也会停止，作业状态为"“STOPPED”           |
| fail | 根据退出状态决定是否失败当前的任务，同时Job也会停止，作业状态为"FAILED"             |
| end  | 根据退出状态决定是否结束当前的任务，同时Job也会停止，作业状态为"COMPLETED"          |

我们用一个例子体验。有两组step需要执行，每组step有3个step，每个step需要睡眠10秒钟。

这两组step并行执行。组内的step串行执行。

换句话说，我们有两个Flow，每个Flow包含3个step。每个step睡眠10秒。

首先我们创建一个睡眠step的创建方法

![image-20201212160312352](https://i-blog.csdnimg.cn/blog_migrate/2f96cf9170d4afd280106659b8469146.png)

接着创建一个线程池(并行执行，肯定需要线程池)

![image-20201212160351157](https://i-blog.csdnimg.cn/blog_migrate/2be0dc98b46105c09555c246a63b258e.png)

然后我们创建两个flow

![image-20201212160408471](https://i-blog.csdnimg.cn/blog_migrate/2c402bf70575e30f8e66029a037ac8b4.png)

为了保证服务正确的被停止，我们增加一个step，当并行任务执行完毕后，用于关闭线程池

![image-20201212160446892](https://i-blog.csdnimg.cn/blog_migrate/3d89861926bd709e91033799d72d07f6.png)

所以，完整的定义如下

![image-20201212160504555](https://i-blog.csdnimg.cn/blog_migrate/ac1ad2adecbce46ff225a1f58928a3f8.png)

完整的代码
    
    
    @Component
    public class SplitFlowStepJobConf {
    
        @Autowired
        private JobBuilderFactory jobBuilderFactory;
    
        @Autowired
        private StepBuilderFactory stepBuilderFactory;
    
        @Autowired
        private JobLauncher jobLauncher;
    
        @PostConstruct
        public void runJob() throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job(), new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
        }
    
        private Job job() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            executor.setMaxPoolSize(3);
            executor.setCorePoolSize(3);
            executor.initialize();
            return jobBuilderFactory.get("split-flow-step-job")
                    .start(getStep("init", 5))
                    .split(executor)
                    .add(new FlowBuilder<Flow>("flow1").start(getStep("start", 10))
                                    .next(getStep("fs1", 10))
                                    .next(getStep("fs2", 10))
                                    .build(),
                            new FlowBuilder<Flow>("flow2").start(getStep("start", 10))
                                    .next(getStep("fs1", 10))
                                    .next(getStep("fs2", 10))
                                    .build())
                    .next(stepBuilderFactory.get("clean")
                            .tasklet((sc, cc) -> {
                                executor.shutdown();
                                return RepeatStatus.FINISHED;
                            }).build())
                    .on("x").stop()
                    .on("xx").fail()
                    .on("xxx").end()
                    .build().build();
    
        }
    
        private Step getStep(String name, Integer seconds) {
            return stepBuilderFactory.get(name)
                    .tasklet((sc, cc) -> {
                        System.out.println(Thread.currentThread().getName() + " step : " + sc.getStepExecution().getStepName() + " time : " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("YYYY-MM-dd HH:MM:SS")) + " run >>>>> sleep : " + seconds);
                        TimeUnit.SECONDS.sleep(seconds);
                        return RepeatStatus.FINISHED;
                    }).build();
        }
    
    }
    

执行结果

![image-20201212160545758](https://i-blog.csdnimg.cn/blog_migrate/7ec77aff48e0158f5812982aed196222.png)

## Step Job Flow关系

Step和Flow和Job的关系

![image-20201212161852982](https://i-blog.csdnimg.cn/blog_migrate/e6451053c99c332f4d9567c08df77716.png)

### Flow

spring batch框架提供了Flow用于声明一组step。job可以直接使用Flow。Flow和Job和Step的地位相同，都可以创建抽象的Flow等。

Flow的组成：1.step;2.split;3.flow;4.decision。

也就是说flow可以包含step，Flow，并行的flow，以及流程控制dec。

引入Flow的意义：我认为是为了复用step。

如果我们将step拆分的特别粗大，那么在组装job的时候很容易，但是step里面的逻辑或者功能多了，step本身的灵活性，复用性就降低了。

而spring batch是提倡复用step的。

但是如果我们将step拆分的特别小的时候，灵活性和复用性提高了，但是，组装job的复杂程度也跟着增加了。

因此，引入Flow，可以进一步对step进行封装。

而且flow,step是同等地位：在flow中可以包含引用flow和step，在step中也可以包含flow和step。

而对于job来说，step和flow是相同的。

这样就在很大程度上增加了step的复用性。

### FlowStep

FlowStep是指Step可以使用定义好的Flow。FlowStep和直接使用Flow的区别在于前者将Flow作为一个单独的Step在Job执行期间出现(该单独的Step是将Flow中定义的所有的Step包装在一个大的Step中)；后者没有单独的Step，Flow中定义了几个step，则执行期间有多少step。

使用FlowStep的好处 在于，在Job中将外部定义的Flow作为一个完整的step，同时Flow中定义的多个step在job执行期间作为完整step的一个子活动，当Flow中所有的step执行完毕后FlowStep才会执行完成。

### JobStep

JobStep是指step元素使用外部定义的job，JobStep和FlowStep的区别在于前者将Job作为一个单独的Step在Job执行期间出现（该单独的step将job中定义的所有的step包装在一个大的step中；同时内部的job在运行期间是一个完整的job执行）；而后者将Flow作为一个单独的step在job执行期间出现（该单独的step将Flow中定义的所有的step包装在一个大的step中）

## 数据共享

Execution Context是 Spring Batch框架提供的持久化与控制的key/value对，能够让开发者在Step Execution 或 Job Execution中保存需要进行持久化的状态。

Execution Context分为两类:一类是 Job Execution 的上下文（对应表:BATCH_JOB_EXECUTION_CONTEXT)﹔另一类是Step Execution 的上下文（对应表:BATCH_STEP_EXECUTION_CONTEXT)。两类上下文之间的关系:一个Job Execution对应一个Job Execution的上下文;每个Step Execution对应一个Step Execution 上下文;同一个Job中的Step Execution公用Job Execution的上下文。

因此如果同一个Job的不同Step间需要共享数据，则可以通过Job Execution的上下文来共享数据。利用Execution Context 中的 key/value对可以重新启动Job;也可以利用ExecutionContext在不同的作业步Step之间进行数据功能。

Spring Batch中可以通过tasklet、 reader. write、 processor、 listener中访问Execution Context对象，在不同的Step中可以将数据写入 Context或者从 Context 读取。

Job Execution Context在整个Job的执行期间存在，不同的Step可以将数据存入Job上下文中，Job Execution Context在执行期间会将数据保存到DB中，在Job重启的时候能够恢复Job 的状态。

**StepExecutionContext**

![image-20201212170432488](https://i-blog.csdnimg.cn/blog_migrate/8ee1be64c93238270c616028b5b54442.png)

执行结果

![image-20201212170448143](https://i-blog.csdnimg.cn/blog_migrate/5a6ced7071808f5e387fe0406d8edada.png)

数据库中的记录

![image-20201212170550320](https://i-blog.csdnimg.cn/blog_migrate/34e636c0baaa06710deb872ad4ae9b44.png)

**JobExecutionContext**

我们创建两个step，在第一个step中放入信息，在第二个step中取出

![image-20201212170848715](https://i-blog.csdnimg.cn/blog_migrate/b280948c6431422a254494aa71794d8a.png)

执行结果

![image-20201212171103401](https://i-blog.csdnimg.cn/blog_migrate/ee351e31d7090b0642f8d720c205e296.png)

数据库记录

![image-20201212171135129](https://i-blog.csdnimg.cn/blog_migrate/2185d0ee1cda26f09cb2e96ae53a366b.png)

完整代码
    
    
    @Component
    public class ContextJobConf {
    
        @Autowired
        private StepBuilderFactory stepBuilderFactory;
    
        @Autowired
        private JobBuilderFactory jobBuilderFactory;
    
        @Autowired
        private JobLauncher jobLauncher;
    
        @PostConstruct
        public void runJob() throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job(), new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
        }
    
        private Job job(){
            return jobBuilderFactory.get("context-job")
                    .start(step1())
                    .next(step2())
                    .next(step3())
                    .build();
        }
    
        private Step step1() {
            return stepBuilderFactory.get("context-job-step")
                    .tasklet((sc,cc)->{
                        sc.getStepExecution().getExecutionContext().put("message", "tasklet!!!");
                        System.out.println(sc.getStepExecution().getExecutionContext().get("message"));
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    
        private Step step2() {
            return stepBuilderFactory.get("context-job-step")
                    .tasklet((sc,cc)->{
                        sc.getStepExecution().getJobExecution().getExecutionContext().put("jmsg","job message!!");
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    
        private Step step3() {
            return stepBuilderFactory.get("context-job-step")
                    .tasklet((sc,cc)->{
                        System.out.println(sc.getStepExecution().getJobExecution().getExecutionContext().get("jmsg"));
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    }
    

## 终止Job

Spring Batch框架中支持在Job 的某一个作业步终止作业。截止到目前，前面所有的Job均是在Job的最后一个Step完成Job 的执行。本节我们展示如何使用end、stop、fail元素来根据ExitStatus完成Job的终止。

end用来根据 ExitStatus来正确的完成Job，使用end 结束后的Job 的BatchStatus是COMPLETED，不能重新启动。

stop用来根据ExitStatus来停止Job，使用sto结束后的Job的 BatchStatus 是 STOPPED,可以重新启动。

fail用来根据ExitStatus 来让Job 失败，使用fail结束后的Job的 BatchStatus 是FAILED,可以重新启动。

end,stop,fail的区别

| 元素   | BatchStatus | ExitStatus | ExitStatus是否更改 | 能否重启 | 说明                 |
|------|-------------|------------|----------------|------|--------------------|
| end  | COMPLETED   | COMPLETED  | 是              | 否    | 完成当前的step后，完成当前Job |
| stop | STOPPED     | STOPPED    | 否              | 是    | 完成当前step后，停止当前Job  |
| fail | FAILED      | FAILED     | 否              | 是    | 完成当前Step后，Job失败    |

### end

可以根据给定的退出状态将Job正常的终止掉;通常可以根据业务状态的值将Job终止，默认情况下Job的退出状态为COMPLETED，批处理状态为COMPLETED;可以根据属性exit-code来指定Job的批处理退出状态值。

end的使用方式

| 方法  | 说明                                                                                                                                  | 默认值       |
|-----|-------------------------------------------------------------------------------------------------------------------------------------|-----------|
| on  | 定义作业步的ExitStatus(退出状态)和on指定的值匹配的时候，正常完成Job。on可以是任意的字符串，同时支持通配符`*,?`.`*`表示退出状态为任何值都满足。`?`表示匹配一个字符，如c?t，当作业的退出状态为cat的时候满足，如果是caat则不满足 |           |
| 参数  | 设置Job的退出状态                                                                                                                          | COMPLETED |

首先创建两个step，第一个step的退出码是quit,第二个step的退出码是ok。

![image-20201212175622794](https://i-blog.csdnimg.cn/blog_migrate/6d12e480108600229a522534352a160a.png)

然后配置关系：如果step1的退出状态是quit，那么end这个job，退出码是step1.如果step2的退出状态是ok，那么end这个job,退出码是step2

![image-20201212175746139](https://i-blog.csdnimg.cn/blog_migrate/aca2450f892fd7cc30f26bbfdb9b11b6.png)

执行结果

![image-20201212175800256](https://i-blog.csdnimg.cn/blog_migrate/d05d487b6b954ff7b7b2d88a466d9bf7.png)

数据库中step执行结束，退出码是quit

![image-20201212173200624](https://i-blog.csdnimg.cn/blog_migrate/e3c43f5fbfc83edc7f426d772ba2a2fe.png)

end的参数是设置Job的退出状态

![image-20201212175524876](https://i-blog.csdnimg.cn/blog_migrate/9d8ded3546a68695c0bb359e7e463604.png)

如果我们注释掉step1中的设置退出状态的代码

![image-20201212175842176](https://i-blog.csdnimg.cn/blog_migrate/5b39e04ba01585131f85f72fcaf2649a.png)

然后执行

![image-20201212175931432](https://i-blog.csdnimg.cn/blog_migrate/965bdcdf8df50b39b5736a935071c0d6.png)

数据库中Job的退出码

![image-20201212175958636](https://i-blog.csdnimg.cn/blog_migrate/498587e3466f6d6932e5af0f4e01215f.png)

数据库中step的退出码

![image-20201212180030624](https://i-blog.csdnimg.cn/blog_migrate/94e13632659ac26d38a5ef2bd9d66b58.png)

需要注意，我们通过end结束job，job的状态是COMPLETED。

完整代码
    
    
    @Component
    public class EndJobConf {
    
        @Autowired
        private JobBuilderFactory jobBuilderFactory;
    
        @Autowired
        private StepBuilderFactory stepBuilderFactory;
    
        @Autowired
        private JobLauncher jobLauncher;
    
        @PostConstruct
        public void runJob() throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job(), new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
        }
    
        private Job job() {
            Step step1 = step1();
            Step step2 = step2();
            return jobBuilderFactory.get("end-job")
                    .start(step1)
                    .on("quit").end("step1")
                    .from(step1).on("*").to(step2)
                    .next(step2)
                    .from(step2).on("ok").end("step2")
                    .build().build();
        }
    
        private Step step1(){
            return stepBuilderFactory.get("end-step1")
                    .tasklet((sc,cc)->{
    //                    sc.setExitStatus(new ExitStatus("quit"));
                        System.out.println(sc.getStepExecution().getStepName());
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    
        private Step step2() {
            return stepBuilderFactory.get("end-step2")
                    .tasklet((sc,cc)->{
                        sc.setExitStatus(new ExitStatus("ok"));
                        System.out.println(sc.getStepExecution().getStepName());
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    
    }
    

### stop

使用stop元素，可以根据给定的退出状态将Job停止掉;通常可以根据业务状态的值将Job停止，默认情况下Job的退出状态为STOPPED，批处理状态也为STOPPED;可以根据属性restart来指定Job重启时候从哪个Step开始执行。

stop的属性

| 方法             | 说明                                                             |
|----------------|----------------------------------------------------------------|
| on             | 定义作业步的ExitStatus和on指定的值匹配的时候，停止当前Job。on属性的值可以是任意的字符串，同时支持`*,?` |
| stop           | 停止job                                                          |
| stopAndRestart | 停止job并指定重启的step                                                |

首先创建两个step,用于设置退出码

![image-20201212180707865](https://i-blog.csdnimg.cn/blog_migrate/229b5d1183cfd3af802147d180a51a23.png)

接着设置停止关系

如果step1退出码是stop1，那么停止job

如果step1退出码是stop1r，那么停止，然后重新启动，并从step2开始执行

如果step1退出码是其他值，那么继续执行step2

如果step2退出码是stop2那么停止。

![image-20201212180901237](https://i-blog.csdnimg.cn/blog_migrate/cf57f86af0761aaf0e2a56b7d24c159e.png)

第一次直接运行，会在step1哪里停止，退出码是stop1

![image-20201212180932487](https://i-blog.csdnimg.cn/blog_migrate/718f709c75d2c6869ac2d6cdc62ac731.png)

step的数据库记录

![image-20201212181010594](https://i-blog.csdnimg.cn/blog_migrate/70fe3c4fcd34c1a86c28c116ad93a26a.png)

job的数据库记录STOPPED

![image-20201212181111540](https://i-blog.csdnimg.cn/blog_migrate/7e18240b9149923c866a7feb0f535a01.png)

接着我们修改step1的退出码为stop1r

![image-20201212181157775](https://i-blog.csdnimg.cn/blog_migrate/d958d184442e65b72e4fbf95064f3b37.png)

然后运行

![image-20201212181456119](https://i-blog.csdnimg.cn/blog_migrate/544d609fc821fcc93b41205552bfaa2b.png)

step数据库中的记录

![image-20201212181450803](https://i-blog.csdnimg.cn/blog_migrate/7b40950739a3fcf876cc40beecb2f1e9.png)

此时Job的状态是STOPPED，我们重启这个Job实例

![image-20201212181406112](https://i-blog.csdnimg.cn/blog_migrate/728f49fc4c6e6a477a5183f7668108d9.png)

重启，重启之后就不会 在从step1开始了，而是直接从step2开始了。

![image-20201212181528645](https://i-blog.csdnimg.cn/blog_migrate/fc168e081345e7ff28f1e667a17d13d3.png)

因为在第一次运行的时候step1的状态是COMPLETED，重启当然就不会在继续执行了。

如果这么认为我们不设置，也是会从step2开始运行。那么我们修改为step1执行的退出码是stop1r，那么重启后，再次从step1开始运行

![image-20201212184428322](https://i-blog.csdnimg.cn/blog_migrate/33c3dfdd3a694618075c5a0aa3adf866.png)

再次重启，因为在step2中，也是stop，所以job本身是stop的

![image-20201212184552630](https://i-blog.csdnimg.cn/blog_migrate/1c8cea09e9994721b6b1f7a2eebe2b52.png)

如果我们注释掉step1的退出码设置

![image-20201212182057411](https://i-blog.csdnimg.cn/blog_migrate/80009c771656963ebfda36fe8504808f.png)

然后重新执行(需要修改参数，之前的job实例是一个死循环，在数据库中的记录是运行状态，不能重启)(不是死循环了)

![image-20201212182241421](https://i-blog.csdnimg.cn/blog_migrate/d8d2134d4b4976059abdfb0c4e2d1fde.png)

数据库中job还是stop

![image-20201212182316761](https://i-blog.csdnimg.cn/blog_migrate/546854126411148a6e0e4a8a2c24437c.png)

完整代码
    
    
    @Component
    public class StopJobConf {
    
        @Autowired
        private JobBuilderFactory jobBuilderFactory;
    
        @Autowired
        private StepBuilderFactory stepBuilderFactory;
    
        @Autowired
        private JobLauncher jobLauncher;
    
        @PostConstruct
        public void runJob() throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job(), new JobParametersBuilder().addLong("id", 1L).toJobParameters());
        }
    
        private Job job() {
            Step step1 = step1();
            Step step2 = step2();
            return jobBuilderFactory.get("stop-job")
                    .start(step1)
                    .on("stop1").stop()
                    .from(step1).on("stop1r").stopAndRestart(step1)
                    .from(step1).on("*").to(step2)
                    .next(step2)
                    .from(step2).on("stop2").stop()
                    .build().build();
        }
    
        private Step step1() {
            return stepBuilderFactory.get("stop-step1")
                    .tasklet((sc, cc) -> {
    //                    sc.setExitStatus(new ExitStatus("stop1r"));
                        System.out.println(sc.getStepExecution().getStepName());
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    
        private Step step2() {
            return stepBuilderFactory.get("stop-step2")
                    .tasklet((sc, cc) -> {
                        sc.setExitStatus(new ExitStatus("stop2"));
                        System.out.println(sc.getStepExecution().getStepName());
                        return RepeatStatus.FINISHED;
                    })
                    .build();
        }
    }
    

### fail

使用fail元素，可以根据给定的退出状态将Job正确地终止掉;通常可以根据业务状态的值将Job终止，默认情况下Job 的退出状态为FAILED，批处理状态为FAILED;可以根据属性exit-code 来指定Job 的批处理退出状态值。

fail属性

| 方法   | 说明                                                                    |    |
|------|-----------------------------------------------------------------------|----|
| on   | 定义作业步的ExitStatus和on指定的值匹配的时候，以失败的方式完成Job。on属性的值可以是任意的字符串，同时支持通配符`*,?` |    |
| fail | 设置Job的退出状态                                                            |    |

首先创建两个step1，step2.

![image-20201212184046689](https://i-blog.csdnimg.cn/blog_migrate/2e9a95c9d84e604415c913109efcd14c.png)

然后定义关系

如果step1的退出码是fail，那么结束job，设置状态为fail

如果step1的退出码是其他，那么继续执行step2

如果step2的退出码是fail，那么结束job,设置状态为fail

![image-20201212183054880](https://i-blog.csdnimg.cn/blog_migrate/9d46f5418b7e93b7a2078b4add780e64.png)

第一次执行

![image-20201212183332941](https://i-blog.csdnimg.cn/blog_migrate/8e34a9a1cf54b0a29a202e8043b9fe50.png)

step的数据库记录COMPLETED，退出码是fail(退出码是无法修改的)

![image-20201212183412326](https://i-blog.csdnimg.cn/blog_migrate/cfd96642b2f182828ce6bfa84e59bafe.png)

job的数据库记录

![image-20201212183521092](https://i-blog.csdnimg.cn/blog_migrate/8046dbcd451536da103adce67d43417e.png)

我们修改step1的退出码，然后重启

![image-20201212183556530](https://i-blog.csdnimg.cn/blog_migrate/524f0da0fe0c103edb16de405266e7b2.png)

执行结果

![image-20201212184149513](https://i-blog.csdnimg.cn/blog_migrate/de635edb2ffb59525094b98a4bfd91e2.png)

此时数据库中还是失败

![image-20201212184218739](https://i-blog.csdnimg.cn/blog_migrate/b774dc8b31393f2f1eb33da10c648f2c.png)

![image-20201212184300828](https://i-blog.csdnimg.cn/blog_migrate/9735f1176d23d10c1a85a776b5c055a0.png)

完整代码
    
    
    @Component
    public class FailJobConf {
        @Autowired
        private JobBuilderFactory jobBuilderFactory;
    
        @Autowired
        private StepBuilderFactory stepBuilderFactory;
    
        @Autowired
        private JobLauncher jobLauncher;
    
        @PostConstruct
        public void runJob() throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job(), new JobParametersBuilder().addLong("id", 2L).toJobParameters());
        }
    
        private Job job() {
            Step step1 = step1();
            Step step2 = step2();
            return jobBuilderFactory.get("fail-job")
                    .start(step1)
                    .on("fail").fail()
                    .from(step1).on("*").to(step2)
                    .next(step2)
                    .from(step2).on("fail").fail()
                    .build().build();
        }
    
        private Step step1() {
            return stepBuilderFactory.get("fail-step1")
                    .tasklet((sc, cc) -> {
                        sc.setExitStatus(new ExitStatus("fail1"));
                        System.out.println(sc.getStepExecution().getStepName());
                        return RepeatStatus.FINISHED;
                    })
                    .allowStartIfComplete(true)
                    .build();
        }
    
        private Step step2() {
            return stepBuilderFactory.get("fail-step2")
                    .tasklet((sc, cc) -> {
                        sc.setExitStatus(new ExitStatus("fail"));
                        System.out.println(sc.getStepExecution().getStepName());
                        return RepeatStatus.FINISHED;
                    })
                    .allowStartIfComplete(true)
                    .build();
        }
    }
