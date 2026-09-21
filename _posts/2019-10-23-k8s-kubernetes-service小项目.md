---
layout: post
title: "k8s-kubernetes-service小项目"
date: 2019-10-23 19:10:23 +0800
categories: [kubernetes, service, k8s负载均衡, k8s的高可用, k8s服务发现]
description: "本文深入探讨Kubernetes Service的工作原理，包括服务发现、负载均衡机制及四种类型：ClusterIP、NodePort、LoadBalancer和ExternalName。通过实例演示如何在Kubernetes集群中创建并访问Tomcat服务。"
keywords: kubernetes, service, k8s负载均衡, k8s的高可用, k8s服务发现
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/102708900
> - 发布时间：2019-10-23 19:10:23
> - 阅读量：424
> - 分类：kubenetes专栏收录该内容, 订阅专栏
> - 标签：#kubernetes, #service, #k8s负载均衡, #k8s的高可用, #k8s服务发现

## 摘要

文章浏览阅读424次。本文深入探讨Kubernetes Service的工作原理，包括服务发现、负载均衡机制及四种类型：ClusterIP、NodePort、LoadBalancer和ExternalName。通过实例演示如何在Kubernetes集群中创建并访问Tomcat服务。

---

#### k8s-kubernetes-service小项目

  * 1.service 简介
  * 2.service 定义
  * 3.实例
  * 4.总结

## 1.service 简介

Kubernetes在设计之初就充分考虑了针对容器的服务发现与负载均衡机制，提供了Service资源，并通过kube-proxy配合cloud provider来适应不同的应用场景。随着kubernetes用户的激增，用户场景的不断丰富，又产生了一些新的负载均衡机制。目前，kubernetes中的负载均衡大致可以分为以下几种机制，每种机制都有其特定的应用场景：

  * Service：直接用Service提供cluster内部的负载均衡，并借助cloud provider提供的LB提供外部访问
  * Ingress Controller：还是用Service提供cluster内部的负载均衡，但是通过自定义LB提供外部访问
  * Service Load Balancer：把load balancer直接跑在容器中，实现Bare Metal的Service Load Balancer
  * Custom Load Balancer：自定义负载均衡，并替代kube-proxy，一般在物理部署Kubernetes时使用，方便接入公司已有的外部服务

Service是对一组提供相同功能的Pods的抽象，并为它们提供一个统一的入口。借助Service，应用可以方便的实现服务发现与负载均衡，并实现应用的零宕机升级。Service通过标签来选取服务后端，一般配合Replication Controller或者Deployment来保Service证后端容器的正常运行。这些匹配标签的Pod IP和端口列表组成endpoints，由kube-proxy负责将服务IP负载均衡到这些endpoints上。  
Service有四种类型：

  * ClusterIP：默认类型，自动分配一个仅cluster内部可以访问的虚拟IP
  * NodePort：在ClusterIP基础上为Service在每台机器上绑定一个端口，这样就可以通过 `<NodeIP>:NodePort`来访问该服务
  * LoadBalancer：在NodePort的基础上，借助cloud provider创建一个外部的负载均衡器，并将请求转发到 `<NodeIP>:NodePort`
  * ExternalName：将服务通过DNS CNAME记录方式转发到指定的域名（通过 spec.externlName 设定）。需要kube-dns版本在1.7以上。另外，也可以将已有的服务以Service的形式加入到Kubernetes集群中来，只需要在创建Service的时候不指定Label selector，而是在Service创建好后手动为其添加endpoint。

## 2.service 定义

[k8s yaml格式的service定义文件完整内容](<https://blog.csdn.net/a18792721831/article/details/102687141>)

## 3.实例

在现有pod上创建服务。  
在这个实例[k8s-kubernetes入门-tomcat环境](<https://blog.csdn.net/a18792721831/article/details/102669307>)的基础上创建服务。
    
    
    touch testtomcat.yaml
    vi testtomcat.yaml
    
    
    
    apiVersion: v1
    kind: Service
    metadata:
      name: tomcat-service
      namespace: study
      labels:
        name: tomcat-service
    spec: 
      selector:
        mytomcat: study
      type: NodePort
      ports:
      - name: tomcat-service-8080
        protocol: TCP
        port: 30080
        targetPort: 8080
        nodePort: 30081
    
    
    
    kubectl apply -f testtomcat.yaml
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ba2ccdfeec6aef99607b8db76228ab17.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4cc95e4064ef168cda61e772423808c3.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ebb7be5aeba03b4291afb5151018c23.png)  
接下来在浏览器访问：  
http://master:30081  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c7907c621908a4f61b96fcf79cba73a1.png)

## 4.总结

现在的环境中有pod1个，service 1个，之前只有pod的时候，我们只能通过http://node:podPort访问  
现在有了service 我们就可以通过http://master:servicePort访问。  
不用关心pod具体被分配到哪个node上。

k8s是基于docker的，docker是因为虚拟机的解决方案太过臃肿，是一个轻量级的虚拟机。

现在环境如下：  
10.0.228.93:master  
10.0.228.117:node  
10.244.1.4:pod  
10.98.148.75:service  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c446e2aaddea83756afb504ae23d559b.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d2946a7fa9503fafacfe4cfe60216913.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/786ce81392a40bdbbd93c96889ba4516.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/04530538804db32e0d662460431819a2.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/015bf8194471b8246f23a9fc17bad730.png)  
pod对外开放18080端口，tomcat容器本身是8080端口  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7db8ff107c0982d2c09ec9d40659e8b6.png)  
service 监听8080对外30080，物理机映射30081  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bdf38d73e5eebd76919a4cc0adffa844.png)

所以，我们现在要访问到tomcat容器的8080端口。

没有service 时：  
内网(集群内部主机)：  
10.244.1.4:18080  
10.0.228.117:18080  
外网：  
10.0.228.117:18080

有service时：  
内网(集群内部主机)：  
10.244.1.4:18080无法访问  
10.98.148.75:30080  
外网：  
10.0.228.93:30081  
10.0.228.117:18080

这个实现是通过iptables的规则实现的。  
以10.98.148.75:30081为例
    
    
    iptables-save|grep 10.98.148.75
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dd37975df4d1dbec8466878d99c777b5.png)  
第一条表示10.244的都可以访问，即pod可以访问service

第二条是转到KUBE-SVC-AOO2BEF3XRFSYLKH  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/50c2157adfebcf5ca0af5673a03afc13.png)  
第一条是30081的端口。  
即：  
内网访问30080，外网30081

访问过程更详细参考：  
[k8s的service](<https://www.cnblogs.com/benjamin77/p/9908547.html>)
