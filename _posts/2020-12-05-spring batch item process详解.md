---
layout: post
title: "spring batch item process详解"
date: 2020-12-05 14:23:32 +0800
categories: [batch process, ItemProcessor, batch处理操作, batch转换数据, batch过滤和跳过]
description: "spring batch item process详解ItemProcessorItemProcessor系统处理组件数据转换部分数据转换数据类型转换数据过滤数据Filter数据过滤统计数据校验ValidatorValidatingItemProcessor组合处理器服务复用拦截器接口异常执行顺序AnnotationMergegithub地址：https://github.com/a18792721831/studybatch.git文章列表：spring batch 入门spring batch_springbatch process"
keywords: batch process, ItemProcessor, batch处理操作, batch转换数据, batch过滤和跳过
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/110681958
> - 发布时间：2020-12-05 14:23:32
> - 阅读量：1
> - 分类：spring batch专栏收录该内容, 订阅专栏
> - 标签：#batch process, #ItemProcessor, #batch处理操作, #batch转换数据, #batch过滤和跳过

## 摘要

文章浏览阅读1.7k次。spring batch item process详解ItemProcessorItemProcessor系统处理组件数据转换部分数据转换数据类型转换数据过滤数据Filter数据过滤统计数据校验ValidatorValidatingItemProcessor组合处理器服务复用拦截器接口异常执行顺序AnnotationMergegithub地址：https://github.com/a18792721831/studybatch.git文章列表：spring batch 入门spring batch_springbatch process

---

#### spring batch item process详解

  * ItemProcessor
  *     * ItemProcessor
    * 系统处理组件
  * 数据转换
  *     * 部分数据转换
    * 数据类型转换
  * 数据过滤
  *     * 数据Filter
    * 数据过滤统计
  * 数据校验
  *     * Validator
    * ValidatingItemProcessor
  * 组合处理器
  * 服务复用
  * 拦截器
  *     * 接口
    * 异常
    * 执行顺序
    * Annotation
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

## ItemProcessor

批处理通过Tasklet完成具体的任务，chunk类型的tasklet定义了标准的读、处理、写的执行步骤。批处理在读取数据后，写入数据之前，希望能够提供一个处理数据的阶段，ItemProcessor是实现处理阶段的重要组件，spring batch框架提供了丰富的处理组件，包括数据转换、组合处理、数据过滤、数据校验等能力。

### ItemProcessor

ItemProcessor是step中对资源的处理阶段，spring batch框架已经提供了各种类型的处理实现。包括数据转换、组合处理、数据过滤、数据校验等。

需要注意的是处理阶段是可选的，也就是说可以只有读、写操作而没有中间的处理数据的阶段，这种情况下读的数据会直接交给写阶段处理。

![image-20201204185859006](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/559c27848cd0554d08df165bef23ce0a.png)

对于较为简单的处理，也可以使用jdk8支持的lambda表达式写。
    
    
    @EnableBatchProcessing
    @Configuration
    public class ItemProcessJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("item-processor-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicInteger atomicInteger = new AtomicInteger();
            return stepBuilderFactory.get("item-processor-step")
                    .<Integer, Integer>chunk(3)
                    .reader(() -> atomicInteger.get() > 20 ? null : atomicInteger.getAndIncrement())
                    .processor((Function<Integer, Integer>) item -> {
                        System.out.println("processor : " + item);
                        return item;
                    })
                    .writer(items -> System.out.println("writer : " + items.size()))
                    .build();
        }
    
    }
    

执行结果

![image-20201204192434684](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3cea384dba4e83916fda7d3e91519134.png)

### 系统处理组件

spring batch提供的处理组件。

| ItemProcessor            | 说明                                                 |
|--------------------------|----------------------------------------------------|
| CompositeItemProcessor   | 组合处理器，可以封装多个业务处理服务                                 |
| ItemProcessor            | ItemProcessor适配器，可以复用现有的业务处理服务                     |
| PassThroughItemProcessor | 不做任何业务处理，直接返回读到的数据                                 |
| ValidatingItemProcessor  | 数据校验处理器，支持对数据的校验，如果校验不通过可以进行过滤掉或者通过skip的方式跳过对记录的处理 |

## 数据转换

ItemProcessor的一个核心作用是对读阶段的数据进行转换，其中包括对部分数据进行更改，还可以根据读的数据完全返回一个不同类型的数据给处理阶段。

### 部分数据转换

在部分转换的情况下，不会更改读入阶段的数据类型，可以针对读出的数据进行属性值的重新修订或者重新计算。

比如输入一个对象，在处理中设置对象的属性

首先创建实体

![image-20201204193647791](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/dd9cb3f9ab1b394ced5af7725b6a099a.png)

接着使用实体：
    
    
    @EnableBatchProcessing
    @Configuration
    public class NoTypeItemProcessorJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("no-type-item-processor-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("no-type-item-processor-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 20 ? null : new People(atomicLong.getAndIncrement(),""))
                    .processor((Function<People, People>) item -> {
                        item.setName("name" + item.getId());
                        System.out.println("processor : " + item);
                        return item;
                    })
                    .writer(items -> items.stream().forEach(x->System.out.println("writer : " + x)))
                    .build();
        }
    
    }
    

执行结果

![image-20201204195129698](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3ba61154cf7e9b0abd196aaa64c30235.png)

### 数据类型转换

在数据类型变换的情况下，通常是读、写阶段需要处理的数据类型不一致，此时可以通过ItemProcessor进行数据的适配。

比如输入一个整型，输出一个字符串
    
    
    @EnableBatchProcessing
    @Configuration
    public class TypeItemProcessorJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("type-item-processor-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicInteger atomicInteger = new AtomicInteger();
            return stepBuilderFactory.get("type-item-processor-step")
                    .<Integer, String>chunk(3)
                    .reader(() -> atomicInteger.get() > 20 ? null : atomicInteger.getAndIncrement())
                    .processor((Function<Integer, String>) item -> {
                        System.out.println("processor : " + item);
                        return item.toString() + " process ";
                    })
                    .writer(items -> items.stream().forEach(x->System.out.println("writer : " + x)))
                    .build();
        }
    
    }
    

执行结果

![image-20201204193440106](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/16e284b6d55a7947da1df2b03cd386f0.png)

## 数据过滤

数据处理除了支持数据转换功能外，同样对数据提供了过滤的能力。过滤是指如果对读入阶段的数据不期望在写入阶段被写入，可以通过返回null来阻止该Item数据被写入。

### 数据Filter

数据过滤就是不符合规则数据，返回null。这样spring batch就不会将数据写入。

比如我们对id为3和3的倍数的数据进行过滤
    
    
    @EnableBatchProcessing
    @Configuration
    public class FilterItemProcessorJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder().addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("filter-item-processor-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicLong atomicLong = new AtomicLong();
            return stepBuilderFactory.get("filter-item-processor-step")
                    .<People, People>chunk(3)
                    .reader(() -> atomicLong.get() > 20 ? null : new People(atomicLong.getAndIncrement(), ""))
                    .processor((Function<People, People>) item -> {
                        item.setName("name" + item.getId());
                        System.out.println("processor : " + item);
                        if (item.getId() % 3 == 0) {
                            return null;
                        }
                        return item;
                    })
                    .writer(items -> items.stream().forEach(x -> System.out.println("writer : " + x)))
                    .build();
        }
    
    }
    

可以看到，读入的时候是有id为3和3的倍数的，但是在写入的时候就没有了

![image-20201204195753799](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/53ca8c163ef6e57a4ea438cabc1f79c5.png)

**数据过滤和异常跳过的区别**

数据过滤是通过返回null值，让程序无值可写，实现数据过滤。

异常跳过是正常情况下应该处理，结果出现了允许跳过的异常，从而跳过这个数据。

这两个来源和原因是不同的。

数据过滤是不符合处理规则，但是数据却又客观存在，因此需要进行数据过滤。

异常跳过是发生了预期异常，预期异常又在允许之内，因此跳过异常数据。

### 数据过滤统计

无论是数据过滤还是异常跳过处理，在Job执行期间都会吧源信息存入到JobRepository中，可以通过作业步执行器获取被过滤的记录总数、异常跳过的记录总数等信息。

StepExecution.getFilterCount()可以获取被过滤的记录总数。

StepExecution.getSkipCount()可以获取异常跳过的记录总数。

我们在processor中，将%3的抛出异常，%2的过滤。

![image-20201205122122670](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/26c59937df2d1788af817bb0b8ecfa7c.png)

然后允许跳过

![image-20201205122418273](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7100b3c0d322b0ef8b419b614ac6b9d2.png)

完整代码：
    
    
    @EnableBatchProcessing
    @Configuration
    public class CountItemProcessorJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("count-item-processor-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicInteger atomicInteger = new AtomicInteger();
            return stepBuilderFactory.get("count-item-processor-step")
                    .<Integer, Integer>chunk(3)
                    .reader(new ItemReader<Integer>() {
                        @Override
                        public Integer read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                            if (atomicInteger.get() > 50) {
                                return null;
                            }
                            return atomicInteger.getAndIncrement();
                        }
                    })
                    .processor(new ItemProcessor<Integer, Integer>() {
                        @Override
                        public Integer process(Integer item) throws Exception {
                            if (item % 3 == 0) {
                                throw new Exception(" item % 3 is 0");
                            } else if (item % 2 == 0) {
                                return null;
                            }
                            return item;
                        }
                    })
                    .writer(items -> items.forEach(x -> System.out.println("write : " + x)))
                    .faultTolerant()
                    .skipLimit(10)
                    .skip(Exception.class)
                    .listener(new CountItemProcessorLis())
                    .build();
        }
    
    }
    

执行结果

![image-20201205122451031](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a778030710c5a8785ea010a698235640.png)

## 数据校验

在业务数据处理过程中经常需要对输入的数据进行有效性校验。spring框架提供了对数据校验的接口，如果校验不通过可以抛出给定类型的异常。spring batch框架提供了数据校验处理类ValidatingItemProcessor，可以在处理的阶段进行数据校验，ValidatingItemProcessor本身支持过滤的功能和跳过两种能力。

### Validator

spring 框架提供的校验接口为Validator，只有一个操作validate，对输入的参数进行数据校验，如果校验不通过可以抛出类型为ValidationException的异常。

接口定义

![image-20201205125137537](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/33d34d55b1f59165d2f00c2e4f722343.png)

我们创建一个自己的校验器

![image-20201205125741768](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/431be15c5854fc78df33f8a4554f6c7d.png)

创建好了校验器在什么地方使用呢

在ValidatingItemProcessor中使用。

### ValidatingItemProcessor

ValidatingItemProcessor实现了接口ItemProcess，通过引用接口Validator进行数据校验功能，根据业务需要自定义实现符合业务需求的校验器。ValidatingItemProcessor支持过滤的功能和跳过两种能力，通过属性filter进行表示。true表示校验不通过的时候直接返回null,用于过滤；而设置为false则表示校验不通过，则抛出异常。通过配置step的异常跳过策略，也可以实现跳过。

![image-20201205130200030](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fc63e041b9bb74ba9fb6ba6786afc1e6.png)

我们使用上面创建的校验器

![image-20201205130240437](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/07c24ad932d39840d052f072c0c700d2.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class ValidatorItemProcessorJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("count-item-processor-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicInteger atomicInteger = new AtomicInteger();
            ValidatingItemProcessor<Integer> itemProcessor = new ValidatingItemProcessor<>(new MyValidator());
            itemProcessor.setFilter(true);
            return stepBuilderFactory.get("count-item-processor-step")
                    .<Integer, Integer>chunk(3)
                    .reader(new ItemReader<Integer>() {
                        @Override
                        public Integer read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                            if (atomicInteger.get() > 50) {
                                return null;
                            }
                            return atomicInteger.getAndIncrement();
                        }
                    })
                    .processor(itemProcessor)
                    .writer(items -> items.forEach(x -> System.out.println("write : " + x)))
                    .faultTolerant()
                    .listener(new CountItemProcessorLis())
                    .build();
        }
    
    }@EnableBatchProcessing
    @Configuration
    public class ValidatorItemProcessorJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("count-item-processor-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicInteger atomicInteger = new AtomicInteger();
            ValidatingItemProcessor<Integer> itemProcessor = new ValidatingItemProcessor<>(new MyValidator());
            itemProcessor.setFilter(true);
            return stepBuilderFactory.get("count-item-processor-step")
                    .<Integer, Integer>chunk(3)
                    .reader(new ItemReader<Integer>() {
                        @Override
                        public Integer read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                            if (atomicInteger.get() > 50) {
                                return null;
                            }
                            return atomicInteger.getAndIncrement();
                        }
                    })
                    .processor(itemProcessor)
                    .writer(items -> items.forEach(x -> System.out.println("write : " + x)))
                    .faultTolerant()
                    .listener(new CountItemProcessorLis())
                    .build();
        }
    
    }
    

接着启动

![image-20201205130312440](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ff803a2fbd88d985fda2d5736d66d144.png)

可以看到将%5==0的数据全部过滤了。

## 组合处理器

在spring batch框架中对于chunk智能配置一个ItemProcessor，但是在有些业务场景中需要将一个item同时执行多个不同的处理器，spring batch框架提供了组合ItemProcessor的模式满足这个需求。

![image-20201205130608893](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b7c25e2ec77f6cacf10c7771d3d6975b.png)

我们首先创建两个处理器

![image-20201205131133483](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6204d03a13dc8fbe61391281928a908f.png)

接着使用这两个处理器，将这两个处理器设置到组合处理器中，然后将组合处理器设置给step

![image-20201205131220653](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/173cd584d6ce0bb6758ebd4015b6d4ae.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class ComposeiteItemProcessorJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("composite-item-processor-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory) {
            AtomicInteger atomicInteger = new AtomicInteger();
            CompositeItemProcessor<Integer, Integer> compositeItemProcessor = new CompositeItemProcessor<>();
            compositeItemProcessor.setDelegates(Arrays.asList(processor1(), processor2()));
            return stepBuilderFactory.get("composite-item-processor-step")
                    .<Integer, Integer>chunk(3)
                    .reader(new ItemReader<Integer>() {
                        @Override
                        public Integer read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                            if (atomicInteger.get() > 10) {
                                return null;
                            }
                            return atomicInteger.getAndIncrement();
                        }
                    })
                    .processor(compositeItemProcessor)
                    .writer(items -> items.forEach(x -> System.out.println("write : " + x)))
                    .build();
        }
    
        private ItemProcessor<Integer, Integer> processor1() {
            return new ItemProcessor<Integer, Integer>() {
                @Override
                public Integer process(Integer item) throws Exception {
                    System.out.println("processor1 : " + item);
                    return item;
                }
            };
        }
    
        private ItemProcessor<Integer, Integer> processor2() {
            return new ItemProcessor<Integer, Integer>() {
                @Override
                public Integer process(Integer item) throws Exception {
                    System.out.println("processor2 : " + item);
                    return item;
                }
            };
        }
    }
    

执行结果

![image-20201205131300768](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d3732a7b852b8d95a297d410ab8390b4.png)

## 服务复用

在spring batch框架中，处理器也支持服务复用，ItemProcessorAdapter。

ItemProcessorAdapter持有服务对象，并调用指定的操作来完成ItemProcessor中定义的process功能。需要注意的是:已经存在的服务需要能够直接处理提供的Item对象，即参数必须是读输出的Item的具体类型；返回值类型必须是ItemWriter阶段输入的参数类型

![image-20201205131652188](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a3d9d1e3e803ef0773630a705530ea36.png)

ItemProcessorAdapter的关键属性

| ItemProcessorAdapter属性 | 类型       | 说明                                           |
|------------------------|----------|----------------------------------------------|
| targetObject           | Object   | 需要调用的目标服务对象                                  |
| targetMethod           | String   | 需要调用的目标操作名称                                  |
| arguments              | Obejct[] | 需要调用的操作参数，默认不需要传递该参数，默认情况下将每次处理的Item对象作为参数传入 |

首先创建一个服务，作为系统现有服务

![image-20201205132041416](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fb69d131672911c90debfc6b60867b27.png)

接着创建并使用ItemProcessorAdapter

![image-20201205132350805](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/05a358bea36f8ebc647943b233ecfd42.png)

完整代码
    
    
    @EnableBatchProcessing
    @Configuration
    public class ItemProcessorAdapterJobConf {
    
        @Bean
        public String runJob(JobLauncher jobLauncher, Job job) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
            jobLauncher.run(job, new JobParametersBuilder()
                    .addDate("date", new Date()).toJobParameters());
            return "";
        }
    
        @Bean
        public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
            return jobBuilderFactory.get("item-processor-adapter-job")
                    .start(step)
                    .build();
        }
    
        @Bean
        public Step step(StepBuilderFactory stepBuilderFactory, IntegerService integerService) {
            AtomicInteger atomicInteger = new AtomicInteger();
            ItemProcessorAdapter<Integer, Integer> processorAdapter = new ItemProcessorAdapter<>();
            processorAdapter.setTargetObject(integerService);
            processorAdapter.setTargetMethod("addOne");
            return stepBuilderFactory.get("item-processor-adapter-step")
                    .<Integer, Integer>chunk(3)
                    .reader(new ItemReader<Integer>() {
                        @Override
                        public Integer read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
                            if (atomicInteger.get() > 10) {
                                return null;
                            }
                            return atomicInteger.getAndIncrement();
                        }
                    })
                    .processor(processorAdapter)
                    .writer(items -> items.forEach(x -> System.out.println("write : " + x)))
                    .build();
        }
    
    }
    

执行结果

![image-20201205132430066](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/82781caeebda1c5774ef3b773b9b67ff.png)

## 拦截器

spring batch框架在ItemProcessor执行阶段提供了拦截器，使得在ItemProcessor执行前后能够加入自定义的业务逻辑。接口为ItemProcessListener<T,S>。

### 接口

接口定义

![image-20201205132622832](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/43f24c181a756b6df672639e26166913.png)

我们基于实现接口，创建一个拦截器

![image-20201205132926452](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d266809cb61aac9998c82cf5af131f0e.png)

然后使用

![image-20201205133133133](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f3abb655a61b05295341565540418dae.png)

执行结果

![image-20201205133219771](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3ef7f3f0424fd0120ab32af417598aa8.png)

### 异常

拦截器方法如果抛出异常会影响job的执行，所以在执行自定义的拦截器的时候，需要考虑对拦截器发生的异常做处理，避免影响业务。

我们在原来的例子中，主动抛出异常

![image-20201205141743432](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d6d60cdd783bece097886f576fa95a87.png)

启动执行

![image-20201205141921368](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2f744a19c792270dfc0ce4a61d1a30ba.png)

Job失败

![image-20201205141938648](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c4f446423911a8877ac5757609c93dbb.png)

### 执行顺序

根据配置的顺序。before和配置的顺序相同，after和配置的顺序相反。

![image-20201205141837462](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7e0dc68ebd9a63332eb92e6205a0ed4c.png)

error和配置的顺序相反

![image-20201205141900796](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9d26920730f43da7f3bc97aba49b3615.png)

### Annotation

除了实现接口，也可以使用注解

![image-20201205133342163](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d476c32941e85f3226be944664a7a3eb.png)

加入到拦截器列表中

![image-20201205133416052](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ce643916387992c6750ecc4ac1c9ae29.png)

执行结果

![image-20201205133449416](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c7d19d246a98dc25cd83ad579dd8fdaf.png)

### Merge

spring batch框架提供了多处配置拦截器执行，可以在chunk配置，也可以在tasklet配置。而且基于step的抽象和继承，可以在子step中控制是否执行父step。

通过在子step中使用super调用父step的监听，就可以实现将父、子step的拦截器全部注册。

如果在子step中没有调用父step中注册拦截器的方法，那么父step中的拦截器就不会注册，也就不会执行。
