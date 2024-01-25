---
layout: post
title: tsf定时任务迁移到xxl-job
categories: [spring boot, 微服务, java]
description: tsf定时任务迁移到xxl-job
keywords: tsf, xxl-job, 定时任务迁移, spring boot
---

tsf定时任务迁移到xxl-job

xxl-job 项目gitee：https://gitee.com/xuxueli0323/xxl-job

xxl-job 项目github：https://github.com/xuxueli/xxl-job/

xxl-job 项目文档：https://www.xuxueli.com/xxl-job/



# 1. 介绍

XXL-JOB是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

> xxl-job 是一个国人开发的框架，于15年开源启动，到现在也有7年历史，从最开始的基础功能实现，到现在支持分布式，集群，任务重试，分片等等。基本上已经很全面了，很能打了。

# 2. 原理

xxl-job 分为两部分，xxl-job-admin,xxl-job-executor。调度中心，执行器。

- 调度中心提供管理、调度功能
- 执行器提供任务注册，任务执行功能

执行器启动后，扫描spring 容器中的执行类(`jobHandler`)，然后将执行类注册到调度中心。调度中心会保存执行类和执行器的关系，当调度触发某个执行类执行时，调度中心就会要求执行器执行指定的执行类，完成任务调度。

==在xxl-job框架中，目前主流有两种方案：一是独立的执行器；二是集成的执行器。==

**独立的执行器：**

单独部署spring boot 服务，专门用于执行定时任务，针对执行器服务，支持横向动态扩展，类似于资源池，用于执行任务。

独立执行器适合于定时任务较多，且同时存在任务执行时间差距较大，需要对定时任务做单独资源管理的场景。

> 因为可能部分定时任务耗费资源较多，一定程度上会争夺资源。
>
> 案例：账务模块中账单的生成，结算模块中结算单生成。这两种场景，都是指定时间的大并发场景，如果和其他业务放在一起，那么在指定时间，触发大并发的时候，会影响其他业务，因此抽离出来比较合适。

实现方案：

1. 使用xxl-job的在线任务配置，将源码直接在在线编辑器中执行。

   这种方式使用特别少，适合与系统依赖非常低的场景。

2. 使用 spring 远程调用的方式，在执行器中调用其他服务。

   这种方式使用普遍，比如定时任务中用到了其他服务的功能.

   这种是在执行器中开发任务逻辑。

3. 完全使用 spring 远程调用。

   这种方式与2的区别在于：在执行器外面开发任务逻辑，执行器只是一个调用的作用。

**集成的执行器：**

不单独部署 spring boot 服务，定时任务与业务接口部署在同一个服务中。

## 2.1 设计思想

将调度行为抽象形成“调度中心”公共平台，而平台自身并不承担业务逻辑，“调度中心”负责发起调度请求。

将任务抽象成分散的JobHandler，交由“执行器”统一管理，“执行器”负责接收调度请求并执行对应的JobHandler中业务逻辑。

因此，“调度”和“任务”两部分可以相互解耦，提高系统整体稳定性和扩展性；

## 2.2 系统组成

- **调度模块（调度中心）**：
  负责管理调度信息，按照调度配置发出调度请求，自身不承担业务代码。调度系统与任务解耦，提高了系统可用性和稳定性，同时调度系统性能不再受限于任务模块；
  支持可视化、简单且动态的管理调度信息，包括任务新建，更新，删除，GLUE开发和任务报警等，所有上述操作都会实时生效，同时支持监控调度结果以及执行日志，支持执行器Failover。
- **执行模块（执行器）**：
  负责接收调度请求并执行任务逻辑。任务模块专注于任务的执行等操作，开发和维护更加简单和高效；
  接收“调度中心”的执行请求、终止请求和日志请求等。

## 2.3 架构图

![img.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img.png)



> 原理更多资料请看：https://www.xuxueli.com/xxl-job/#%E4%BA%94%E3%80%81%E6%80%BB%E4%BD%93%E8%AE%BE%E8%AE%A1

# 3. 迁移方案

## 3.1 现状

现在我们使用的是 tsf 的任务框架。

tsf 的任务要求我们实现 `com.tencent.cloud.task.sdk.client.spi.ExecutableTask` 接口。

源码如下：

```java
package com.tencent.cloud.task.sdk.client.spi;

import com.tencent.cloud.task.sdk.client.model.ExecutableTaskData;
import com.tencent.cloud.task.sdk.client.model.ProcessResult;

public interface ExecutableTask {
    ProcessResult execute(ExecutableTaskData var1);
}
```

很简单，就是实现一个`execute`方法。

其参数是封装了一些任务信息，包括任务调度，任务重试，任务超时，任务参数等信息。

返回值封装了任务执行结果，成功失败，成功失败信息，以及任务元数据信息。

## 3.2 迁移方案

因为我们切换了新的任务调度框架，结合 tsf 的任务现状，迁移方案设计如下：

首先增加一个 spring 上下文容器属性操作，扫描 srping 容器中 `com.tencent.cloud.task.sdk.client.spi.ExecutableTask` 接口的实现类，然后将实现类注册到xxl-job。

在注册到xxl-job的时候，新建xxl-job的执行类，做 tsf 的任务的实现代理。

具体源码如下：

```java

import com.tencent.cloud.task.sdk.client.model.ExecutableTaskData;
import com.tencent.cloud.task.sdk.client.model.ProcessResult;
import com.tencent.cloud.task.sdk.client.model.ProcessResultCode;
import com.tencent.cloud.task.sdk.client.spi.ExecutableTask;
import com.xxl.job.core.context.XxlJobHelper;
import com.xxl.job.core.executor.XxlJobExecutor;
import com.xxl.job.core.handler.IJobHandler;
import java.util.Map;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * xxl-job registry
 *
 * @author 
 * @date 2022/10/17 15:27
 **/
@Slf4j
@Component
public class XxlJobRegistry implements ApplicationContextAware {

    @Value("${spring.application.name:}")
    private String appName;

    @Override
    public void setApplicationContext (ApplicationContext applicationContext) throws BeansException {
        // 1. 获取全部的 tsf 的任务
        Map<String, ExecutableTask> beansOfType = applicationContext.getBeansOfType(ExecutableTask.class);
        beansOfType.forEach((key, value) -> {
            // 2. 注册到 xxl-job
            // name = appName # className # execute 
            // name = appName # 任务 method 全引用路径
            // 使用 method 的全引用路径，防止 beanName 重复的任务
            // 使用 appName 前缀，防止跨项目 method 全引用路径重复
            // 使用 匿名实现类 做 tsf 到 xxl-job 的转换
            XxlJobExecutor.registJobHandler(appName + "#" + value.getClass().getName() + "#" + "execute", new IJobHandler() {
                @Override
                public void execute () throws Exception {
                    // XxlJobHelper.log 是 xxl-job 的 日志打印方式
                    XxlJobHelper.log(" job : {} start , param : {} ", key, XxlJobHelper.getJobParam());
                    // log.info 是 tsf 的日志打印方式，也是我们统一日志集成方式
                    log.info(" job : {} start , param : {} ", key, XxlJobHelper.getJobParam());
                    // 构造 tsf 任务触发请求参数
                    ExecutableTaskData executableTaskData = new ExecutableTaskData();
                    // 将 xxl-job 的任务参数 转发 到 tsf 任务请求参数中
                    // 兼容现有的 tsf 任务，同时支持后续任务开发继续使用 tsf 模式
                    executableTaskData.setTaskArgument(XxlJobHelper.getJobParam());
                    // 调用 stf 任务
                    ProcessResult execute = value.execute(executableTaskData);
                    // 对 tsf 任务做结果转换
                    if (ProcessResultCode.SUCCESS == execute.getResultCode()) {
                        XxlJobHelper.handleSuccess(execute.getResultMsg());
                    } else {
                        XxlJobHelper.handleFail(execute.getResultMsg());
                    }
                    XxlJobHelper.log("job : {} , executor : {} , message : {} ", key, execute.getResultCode(), execute.getResultMsg());
                    log.info("job : {} , executor : {} , message : {} ", key, execute.getResultCode(), execute.getResultMsg());
                }
            });
            log.info(" xxl-job registry {} success!", key);
        });
    }

}
```

使用这种方式，可以同时支持 tsf 和 xxl-job ,一方面是 xxl-job 试用阶段的双框架支持，另一方面是 现有 任务的 xxl-job 支持。

## 3.3 xxl-job 配置

xxl-job的配置主要是 xxl-job 执行器的配置。

```properties
### 调度中心部署根地址 [选填]：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"；为空则关闭自动注册；
xxl.job.admin.addresses=http://127.0.0.1:8080/xxl-job-admin
### 执行器通讯TOKEN [选填]：非空时启用；
xxl.job.accessToken=
### 执行器AppName [选填]：执行器心跳注册分组依据；为空则关闭自动注册
xxl.job.executor.appname=xxl-job-executor-sample
### 执行器注册 [选填]：优先使用该配置作为注册地址，为空时使用内嵌服务 ”IP:PORT“ 作为注册地址。从而更灵活的支持容器类型执行器动态IP和动态映射端口问题。
xxl.job.executor.address=
### 执行器IP [选填]：默认为空表示自动获取IP，多网卡时可手动设置指定IP，该IP不会绑定Host仅作为通讯实用；地址信息用于 "执行器注册" 和 "调度中心请求并触发任务"；
xxl.job.executor.ip=
### 执行器端口号 [选填]：小于等于0则自动获取；默认端口为9999，单机部署多个执行器时，注意要配置不同执行器端口；
xxl.job.executor.port=9999
### 执行器运行日志文件存储磁盘路径 [选填] ：需要对该路径拥有读写权限；为空则使用默认路径；
xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler
### 执行器日志文件保存天数 [选填] ： 过期日志自动清理, 限制值大于等于3时生效; 否则, 如-1, 关闭自动清理功能；
xxl.job.executor.logretentiondays=30
```

对应的配置类：

```java

import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * xxl-job config
 *
 * @author jiayongqi
 * @date 2022/10/17 14:27
 **/
@Slf4j
@RefreshScope
@Configuration
public class XxlJobConfig {
    @Value("${xxl.job.admin.addresses:}")
    private String adminAddresses;
    @Value("${xxl.job.executor.appname:}")
    private String appName;
    @Value("${xxl.job.accessToken:}")
    private String accessToken;
    @Value("${xxl.job.executor.address:}")
    private String address;
    @Value("${xxl.job.executor.ip:}")
    private String ip;
    @Value("${xxl.job.executor.port:0}")
    private int port;
    @Value("${xxl.job.executor.logpath:}")
    private String logPath;
    @Value("${xxl.job.executor.logretentiondays:-1}")
    private int logRetentionDays;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor () {
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setAppname(appName);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);
        log.info("xxl-job config init : {}", xxlJobSpringExecutor);
        return xxlJobSpringExecutor;
    }

}
```

## 3.4 调度配置

首先登陆到xxl-job的调度平台

![img_1.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_1.png)  


**现有任务：**

现有任务：`com.boss.communal.xxl.job.task.TsfTestTask`

![img_2.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_2.png)  


根据规则`appName # className # methodName`拼接出Task的唯一标识`boss-communal#com.boss.communal.xxl.job.task.TsfTestTask#execute`

然后在任务管理中创建任务

![img_3.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_3.png)


创建成功后，就可以执行了

![img_4.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_4.png)


如果有参数，那么就设置参数，和 tsf 的任务管理控制台任务参数相同，需要和任务逻辑中参数解析的格式保持一致

![img_5.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_5.png)


然后就可以查看执行日志了

![img_6.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_6.png)


如下

![img_7.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_7.png)


## 3.5 执行器配置

执行器只需要配置一次即可，与项目同步即可，与`applicationName`相同即可

![img_8.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_8.png)


spring 服务要注册到那个执行器，由配置项配置

![img_9.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_9.png)


# 4. 后续开发方案

服务同时支持 tsf 和 xxl-job 后，后续开发定时任务，有两种方式：tsf 方式、xxl-job 注解。

## 4.1 tsf 方式

继续实现`com.tencent.cloud.task.sdk.client.spi.ExecutableTask`接口，别忘记将接口的实现类让spring 管理。

例子：

![img_10.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_10.png)


## 4.2 xxl-job 注解

对定时任务的逻辑上增加`@XxlJob`的注解即可，`value`就是我们在新增任务时的唯一标识。请注意规则`appName # className # methodName`

为了方便统一管理已将 `appName` 封装为接口常量

![img_11.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_11.png)


后面写定时任务的时候，实现接口即可(接口没有需要实现的方法，只是提供常量)

然后在`@XxlJob`注解中使用即可

![img_12.png](/images/posts/2022-10-24-tsf定时任务迁移到xxl-job/img_12.png)


# 5. xxl-job 接入手册

## 5.1 maven 依赖

在需要接入的项目的最外层 `pom.xml` 中的`dependencies`标签下增加依赖

```xml
        <dependency>
            <groupId>com.xuxueli</groupId>
            <artifactId>xxl-job-core</artifactId>
            <version>${xxl-job.version}</version>
        </dependency>
```

别忘记在`properties`标签下增加版本号

```xml
<xxl-job.version>2.3.1</xxl-job.version>
```

刷新maven依赖

## 5.2 配置类

在配置包中，增加配置类（因为xxl-job需要指定执行器，加上xxl-job需要负责注册和监听任务执行，还是比较耗费资源的，所以，不建议将配置类和依赖集成到`common`中，后期如果全部接入后，可以考虑）

```java

import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * xxl-job config
 *
 * @author 
 * @date 2022/10/17 14:27
 **/
@Slf4j
@RefreshScope
@Configuration
public class XxlJobConfig {
    @Value("${xxl.job.admin.addresses:}")
    private String adminAddresses;
    @Value("${xxl.job.executor.appname:}")
    private String appName;
    @Value("${xxl.job.accessToken:}")
    private String accessToken;
    @Value("${xxl.job.executor.address:}")
    private String address;
    @Value("${xxl.job.executor.ip:}")
    private String ip;
    @Value("${xxl.job.executor.port:0}")
    private int port;
    @Value("${xxl.job.executor.logpath:}")
    private String logPath;
    @Value("${xxl.job.executor.logretentiondays:-1}")
    private int logRetentionDays;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor () {
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setAppname(appName);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);
        log.info("xxl-job config init : {}", xxlJobSpringExecutor);
        return xxlJobSpringExecutor;
    }

}
```

## 5.3 配置yml

在yml中增加配置

```yaml
xxl:
  job:
    accessToken: test
    admin:
      addresses: http://slaver:18080/xxl-job-admin
    executor:
      appname: boss-communal
#      address: http://master
      ip: 172.16.10.251
      port: 9999 # 默认 9999
      logpath: ./xxl-job/
      logretentiondays: -1
```



应用程序只能获取到k8s容器内的ip,在访问的时候需要注意，如果调度中心与执行器处于同一个k8s网络中，那么使用ip可以，否则就需要使用 address。

需要保证 执行器 和 调度器 之间能相互访问。

## 5.4 增加常量接口

在配置包或者工具包中增加常量接口

```java

/**
 * @author 
 * @date 2022/10/20 10:47
 **/
public interface ApplicationName {

    String APP_NAME = "boss-communal";

}
```

## 5.5 增加任务注册

在应用初始化的包中增加xxl-job任务注册类

```java

import com.boss.communal.xxl.job.common.ApplicationName;
import com.tencent.cloud.task.sdk.client.model.ExecutableTaskData;
import com.tencent.cloud.task.sdk.client.model.ProcessResult;
import com.tencent.cloud.task.sdk.client.model.ProcessResultCode;
import com.tencent.cloud.task.sdk.client.spi.ExecutableTask;
import com.xxl.job.core.context.XxlJobHelper;
import com.xxl.job.core.executor.XxlJobExecutor;
import com.xxl.job.core.handler.IJobHandler;
import java.util.Map;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * xxl-job registry
 *
 * @author 
 * @date 2022/10/17 15:27
 **/
@Slf4j
@Component
public class XxlJobRegistry implements ApplicationContextAware, ApplicationName {
    @Override
    public void setApplicationContext (ApplicationContext applicationContext) throws BeansException {
        // 1. 获取全部的 tsf 的任务
        Map<String, ExecutableTask> beansOfType = applicationContext.getBeansOfType(ExecutableTask.class);
        beansOfType.forEach((key, value) -> {
            // 2. 注册到 xxl-job
            // name = appName # className # execute
            // 使用 method 的全引用路径，防止 beanName 重复的任务
            // 使用 appName 前缀，防止跨项目 method 全引用路径重复
            // name = appName # 任务 method 全引用路径
            XxlJobExecutor.registJobHandler(APP_NAME + "#" + value.getClass().getName() + "#" + "execute", new IJobHandler() {
                @Override
                public void execute () throws Exception {
                    // XxlJobHelper.log 是 xxl-job 的 日志打印方式
                    XxlJobHelper.log(" job : {} start , param : {} ", key, XxlJobHelper.getJobParam());
                    // log.info 是 tsf 的日志打印方式，也是我们统一日志集成方式
                    log.info(" job : {} start , param : {} ", key, XxlJobHelper.getJobParam());
                    // 构造 tsf 任务触发请求参数
                    ExecutableTaskData executableTaskData = new ExecutableTaskData();
                    // 将 xxl-job 的任务参数 转发 到 tsf 任务请求参数中
                    // 兼容现有的 tsf 任务，同时支持后续任务开发继续使用 tsf 模式
                    executableTaskData.setTaskArgument(XxlJobHelper.getJobParam());
                    // 调用 stf 任务
                    ProcessResult execute = value.execute(executableTaskData);
                    // 对 tsf 任务做结果转换
                    if (ProcessResultCode.SUCCESS == execute.getResultCode()) {
                        XxlJobHelper.handleSuccess(execute.getResultMsg());
                    } else {
                        XxlJobHelper.handleFail(execute.getResultMsg());
                    }
                    XxlJobHelper.log("job : {} , executor : {} , message : {} ", key, execute.getResultCode(), execute.getResultMsg());
                    log.info("job : {} , executor : {} , message : {} ", key, execute.getResultCode(), execute.getResultMsg());
                }
            });
            log.info(" xxl-job registry {} success!", key);
        });
    }

}

```

# 6. 使用

要使用xxl-job，就需要先将执行器注册到调度中心。

如果执行器还未在调度中心创建，需要先创建，然后在创建基于执行器的任务，最后配置定时任务或者手动任务。

具体使用教程见文档：https://www.xuxueli.com/xxl-job/#%E5%9B%9B%E3%80%81%E6%93%8D%E4%BD%9C%E6%8C%87%E5%8D%97

# 7. 总结

xxl-job毕竟是国人开发的框架，其文档非常适合国人习惯，而且非常通俗易懂。遇到的问题在文档中都能找到。

https://www.xuxueli.com/xxl-job/

