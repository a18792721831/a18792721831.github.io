---
layout: post
title: "spring boot config 集成rabbitmq实现总线bus实时刷新配置"
date: 2020-03-09 19:33:32 +0800
categories: [rabbitmq与bus集成, 微服务实现实时刷新配置, 微服务集成mq与消息总线, 如何触发消息实时刷新]
description: "本文详细介绍了如何使用SpringBoot集成RabbitMQ实现配置总线Bus，通过Docker和Kubernetes部署RabbitMQ，并在SpringBoot应用中实现配置的实时刷新。"
keywords: rabbitmq与bus集成, 微服务实现实时刷新配置, 微服务集成mq与消息总线, 如何触发消息实时刷新
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/104737243
> - 发布时间：2020-03-09 19:33:32
> - 阅读量：1
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, spring boot
> - 标签：#rabbitmq与bus集成, #微服务实现实时刷新配置, #微服务集成mq与消息总线, #如何触发消息实时刷新

## 摘要

文章浏览阅读1.1k次。本文详细介绍了如何使用SpringBoot集成RabbitMQ实现配置总线Bus，通过Docker和Kubernetes部署RabbitMQ，并在SpringBoot应用中实现配置的实时刷新。

---

#### spring boot config 集成rabbitmq实现总线bus实时刷新配置

  * [1\. rabbitmq安装](<#1_rabbitmq_3>)
  *     * [1.1 选择docker 镜像](<#11_docker__4>)
    * [1.2 k8s 命名空间创建](<#12_k8s__13>)
    * [1.3 k8s 服务创建](<#13_k8s__26>)
    * [1.4 k8s daemonset 的 deployment](<#14_k8s_daemonset__deployment_65>)
    * [1.5 验证](<#15__123>)
  * [2\. spring boot config bus server 集成](<#2_spring_boot_config_bus_server__127>)
  *     * [2.1 创建项目](<#21__128>)
    * [2.2 配置](<#22__130>)
    * [2.3 注解](<#23__194>)
    * [2.4 远程配置](<#24__196>)
    * [2.5 启动](<#25__199>)
    * [2.6 验证实时刷新](<#26__202>)
  * [2\. spring boot config bus client 集成](<#2_spring_boot_config_bus_client__219>)
  *     * [2.1 创建项目](<#21__220>)
    * [2.2 配置](<#22__222>)
    * [2.4 启动](<#24__227>)
    * [2.5 修改远程配置](<#25__229>)
    * [2.6 post 刷新触发](<#26_post__231>)

  
git地址：   
https://github.com/a18792721831/studySpringCloud.git 

## 1\. rabbitmq安装

### 1.1 选择docker 镜像

因为使用docker 进行启动rabbitmq，那么一个合适的镜像是我们的基础，如果选择的镜像不合适，那么就相当于大楼的地基错误。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c4ec7f565491463429732a173f59d165.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a3835b372671cdb96f53fe4b677180b5.png)  
看到这个是不是很兴奋，so easy!  
但是千万不要用这个镜像，我们继续往下翻。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9d371b93191e01238a62fa4892bbfa01.png)  
发现如果要使用rabbitmq的管理界面，那么需要使用集成了插件的镜像，而不是latest的镜像。  
这个就比较坑，否则无法访问界面。

### 1.2 k8s 命名空间创建

rabbitmq-namespace.yaml
    
    
    apiVersion : v1
    kind : Namespace
    metadata : 
      name : rabbitmq
      labels : 
        app : rabbitmq
    

执行kubectl apply -f rabbitmq-namespace.yaml  
验证  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c800e56b1498500c02eee833efe3c38c.png)

### 1.3 k8s 服务创建

创建k8s服务主要是统一对外开放，但是本次只是各个独立的rabbitmq,未统一化集群部署。  
所以这部可有可无。  
但是统一了服务，后续对rabbitmq集群化后，使用到rabbitmq的地方是不需要做任何修改的。  
开放端口,需要开放两个端口（其余的我未开放）  
5762:消费者，生产者的端口  
15762:管理界面的端口。  
注意，这里说的是rabbitmq默认的端口，也就是容器内的端口(可以自定义配置)  
因为我们主要是学习，测试的rabbitmq，所以未做目录的挂载，意味着，当容器或者说pod重新启动或者调度后，rabbitmq所有的信息将会丢失(主要就是里面的路由和消息)  
rabbitmq-service.yaml
    
    
    apiVersion: v1
    kind: Service
    metadata:
      name: rabbitmq-service-node
      namespace: rabbitmq
      labels:
        name: rabbitmq-service
    spec: 
      selector:
        rabbitmq: template
      type: NodePort
      ports:
      - name: rabbitmq-service-15672
        protocol: TCP
        port: 15672
        targetPort: 15672
        nodePort: 31672
      - name: rabbitmq-service-5672
        protocol: TCP
        port: 5672
        targetPort: 5672
        nodePort: 30672
    

端口映射关系：

| 容器内   | node  | 对外    |
|-------|-------|-------|
| 5672  | 5672  | 30672 |
| 15672 | 15672 | 31672 |

### 1.4 k8s daemonset 的 deployment

因为k8s集群只有两个节点(穷)，所以采集deamonset就行，保证node节点必须有一个。  
rabbitmq-daemonset.yaml
    
    
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: rabbitmq-deployment-daemonset
      namespace: rabbitmq
      labels:
        rabbitmq: deployment-daemonset
    spec:
      #replicas: 2
      selector:
        matchLabels:
          rabbitmq: template
      #strategy: 
      updateStrategy:
        rollingUpdate:
          #maxSurge: 1
          maxUnavailable: 1
      template:
        metadata:
          labels:
            rabbitmq: template
        spec:
          #hostNetwork: true
          containers:
          - name: rabbitmq-pod
            image: rabbitmq:3-management
            imagePullPolicy: IfNotPresent
            livenessProbe:
              tcpSocket:
                port: 5672
              initialDelaySeconds: 180
              timeoutSeconds: 30
              periodSeconds: 600
            resources:
              limits:
                cpu: 4000m
                memory: 4096Mi
              requests:
                cpu: 1000m
                memory: 2048Mi
            ports:
            - name: rabbitmq-15672
              containerPort: 15672
              hostPort: 15672
              protocol: TCP
            - name: rabbitmq-5672
              containerPort: 5672
              hostPort: 5672
              protocol: TCP
    

执行kubectl apply -f rabbitmq-daemonset.yaml  
等待k8s调度，然后在节点上pull镜像。  
可能需要镜像加速。  
pull完后，就会启动。

### 1.5 验证

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f82a7a85ee2a80fb8f3547c8a57901e3.png)  
这个名字就是启动的pod的名字  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/03e704ce0237d2f591aa1445bb146516.png)

## 2\. spring boot config bus server 集成

### 2.1 创建项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/acd7a407a4494adc25382ce9ac1078da.png)

### 2.2 配置

application.yml
    
    
    server:
      port: 8011
    
    logging:
      level:
        org:
          springframework:
            web:
              servlet:
                mvc:
                  method:
                    annotation:
                      RequestMappingHandlerMapping: trace
    
    spring:
      freemarker:
        template-loader-path: classpath:/templates/
        prefer-file-system-access: false
      application:
        name: config-eureka-server
    #  profiles:
    #    active: native
    #  cloud:
    #    config:
    #      server:
    #        native:
    #          search-locations: classpath:/portconfig/
      profiles:
        active: git
      cloud:
        config:
          server:
            git:
              uri: https://github.com/a18792721831/studySpringCloud # 仓库跟地址
              search-paths: springbootconfigserver/src/main/resources/portconfig # 以studySpringCloud为根路径进行搜索的文件夹地址
              username: a18792721831 # 用户名
              password: # 公有仓库可以不写密码
              timeout: 60
          label: master # 指定分支
      rabbitmq:
        host: 10.0.228.93
        port: 30672
        username: guest
        password: guest
    eureka:
      client:
    #    register-with-eureka: false
    #    fetch-registry: false
      service-url:
        defaultZone: http://127.0.0.1:8761/eureka/
    management:
      security:
        enabled: false
      endpoints:
        web:
          exposure:
            include: '*'
    

这里有个坑：  
management.endpoints.web.exposure.include必须配置，可以配置其他字符。  
management.security.enabled=false表示rabbitmq访问时，不进行安全验证。

### 2.3 注解

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e6d794771040eed47d952ae8d764ab6d.png)

### 2.4 远程配置

远程配置需要注释掉port配置，因为我们需要多实例启动。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b9186e5e3d9326077e5011de946078d4.png)

### 2.5 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ca42dde7d21860910d381d7363bbf3c6.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2e3f75ff576872f6e251c7aa8a13b3ce.png)

### 2.6 验证实时刷新

当springbootconfigbusserver启动后，使用postMan发送请求；  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d1462919b6e8323d15ae346eeaa0206.png)  
这里有个坑，好多书本上资料上写的是http://[ip]:[port]/bus/refresh  
且是发送给client。  
可能是旧版本支持吧。  
或者旧版本资料上说请求http://[ip]:[port]/bus-refresh  
新版本请求是http://[ip]:[port]/actuator/bus-refresh  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec6204b5905d83ecc7c8923ce638e0e9.png)  
日志如下  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/696d8198a4db2e807d7fc4af059a6c72.png)  
这是远程没有修改时刷新的日志。  
接下来我们远程修改后，在次刷新。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8126844368e0285358f5f12107806835.png)  
刷新  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/396b2aba3e6dd813ce52b6304de26499.png)  
有了，更新了一个文件。

## 2\. spring boot config bus client 集成

### 2.1 创建项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4874af389a5f749b196074d547422616.png)

### 2.2 配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6b8cd41a979fc118ab8cd4c91e6ea072.png)  
这个配置文件是springbootconfigeurekaclient中拷贝即可。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe2fa0adfdb94c9bad20832ed435bbdd.png)  
management.security.enabled=false表示rabbitmq访问时，不进行安全验证。

### 2.4 启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f1afbc914450b199f5ebae9e9b4181c5.png)

### 2.5 修改远程配置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/809cad3c9ffddf8cd1a1c98985be9d96.png)

### 2.6 post 刷新触发

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad48bd4a9a76314ce9f2dff79c3f9334.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25f478c592c0a0dd81937dc937734ecd.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c05c332ffeef5acab436a49701d2f412.png)  
config client从config server得到了配置。  
而且实现了实时刷新，未重启服务，但是配置信息已经发生修改。