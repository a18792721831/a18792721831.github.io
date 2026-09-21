---
layout: post
title: "k8s--kubernetes架构"
date: 2019-10-15 16:40:53 +0800
categories: [kubernetes架构, 如何在k8s上进行一次部署, k8s部署需要哪些步骤, k8s组件分工, k8s命名空间]
description: "本文深入探讨Kubernetes(K8s)集群的架构组成与服务功能，包括master与node节点角色、核心服务如kube-apiserver、kube-scheduler及kube-proxy的工作原理。同时，通过实例演示命名空间的创建与pod部署流程。"
keywords: kubernetes架构, 如何在k8s上进行一次部署, k8s部署需要哪些步骤, k8s组件分工, k8s命名空间
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/102569906
> - 发布时间：2019-10-15 16:40:53
> - 阅读量：497
> - 分类：kubenetes专栏收录该内容, 订阅专栏
> - 标签：#kubernetes架构, #如何在k8s上进行一次部署, #k8s部署需要哪些步骤, #k8s组件分工, #k8s命名空间

## 摘要

文章浏览阅读497次。本文深入探讨Kubernetes(K8s)集群的架构组成与服务功能，包括master与node节点角色、核心服务如kube-apiserver、kube-scheduler及kube-proxy的工作原理。同时，通过实例演示命名空间的创建与pod部署流程。

---

#### k8s--kubernetes架构

  * 1.k8s集群组成
  * 2.k8s服务介绍
  *     * 2.1kubeadm
    * 2.2kubectl
    * 2.3kube-apiserver
    * 2.4kube-scheduler
    * 2.5kube-controller-manager
    * 2.6etcd
    * 2.7pod网络
    * 2.8kubelet
    * 2.9kube-proxy
  * 3.k8s的架构图
  * 4.一个小例子
  *     * 4.1命名空间
    *       * 4.1.1命名空间查询
      * 4.1.2增加命名空间
      * 4.1.3删除命名空间
    * 4.2部署

## 1.k8s集群组成
    
    
    kubectl get nodes
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/11daca323a716abffcc226194c828334.png)  
k8s集群由master节点和node节点组成。  
master节点是k8s集群的核心大脑，是调度、管理节点，运行着一些核心的服务。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f6f51b5f8b9edbda6439e6d815c7bd92.png)  
coredns、etcd、kube-apiserver、kube-controller-manager、kube-flannel-ds、kube-proxy、kube-scheduler  
node节点是工作节点，运行业务服务，当然也有配合master节点的服务  
kube-flannel-ds、kube-proxy

上述服务都是通过pod的方式在容器中运行。

还有两个服务，不管是master节点还是node节点都需要，它们通过system服务的方式运行。  
那就是kubeadm、kubectl。

在master节点上也有node节点中运行的服务，也就是说，master节点也可以充当node节点。  
即，master节点上也可以运行业务服务。

## 2.k8s服务介绍

### 2.1kubeadm

kubeadm用来初始化k8s集群的环境，包括检测安装k8s的一些必要的条件是否满足，下载文件等等。  
kubeadm类似windos系统中的安装包，安装软件，当k8s集群安装完成后，基本上这个服务的作用就比较小了(依然有，比如生成token等)。

### 2.2kubectl

kubectl在使用yum安装kubeadm是，作为依赖被安装。  
kubectl是一个很重要的服务，node节点与master节点通信、执行操作都是通过这个服务完成的。

### 2.3kube-apiserver

API server提供http/https restful api.  
API server是k8s的前端接口，各种客户端工具或者k8s的其他组件可以通过api server来管理k8s集群的资源等。

### 2.4kube-scheduler

kube-scheduler负责决定将pod放在哪个node上运行。scheduler在调度时，会采用各种调度策略，也可以通过配置，指定使用哪种调度策略，以实现应用的高可用、高性能。

### 2.5kube-controller-manager

kube-controller-manager负责管理k8s集群的各种资源，保证资源处于预期的状态。  
controller-manager由一下几个组成：  
replication controller、endpoints controller、namespaces controller、serviceaccounts controller等。  
replication controller管理Deployment、StatefulSet、DaemonSet的声明周期。  
namespaces controller管理namespaces资源。

### 2.6etcd

etcd负责保存k8s集群的配置信息和资源的状态信息。当数据发生变化时，etcd会快速的通知k8s集群的相关组件。

### 2.7pod网络

pod之间要能相互通信，就必须有pod网络。  
比如在node1上运行的pod1和pod2进行通信、在node2上运行的pod3进行通信等等。

### 2.8kubelet

kubelet是node节点的执行者，当kube-scheduler确定需要在node1上启动pod1后，就会将pod1的具体配置信息发送给node1的kubelet,kubelet根据这些信息创建、启动容器，并将结构报告给master

### 2.9kube-proxy

在k8s集群里有一个服务的概念。  
比如node1里面的pod1提供a服务  
node2里面的pod2也提供a服务  
那么对于整个k8s集群对外来说，k8s集群提供了a服务。  
但是当具体访问a服务时，到底走的是node1还是node2呢？  
就由kube-proxy实现流量转发。  
当然kube-proxy的作用远远不止这些。

## 3.k8s的架构图

这里引用一张k8s中文社区的架构图

![image](https://i-blog.csdnimg.cn/blog_migrate/cda5faf88a0c3b34e4c455ae636c7491.jpeg)

这张图非常清楚的说明了k8s集群中各个服务是如何协作的。

## 4.一个小例子

在这个例子中，我们的目标是创建一个单独的命名空间、然后在这个命名空间里面创建并启动一个pod然后验证这个pod所提供的服务是否成功启动。

### 4.1命名空间

#### 4.1.1命名空间查询
    
    
    kubectl get namespaces --all-namespaces
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/29bdee767581aeaedd74214b3a71f8a4.png)

#### 4.1.2增加命名空间

1.通过命令直接创建
    
    
    kubectl create namespace testcreatenamespaces1
    

注意：命名空间名称满足正则表达式[a-z0-9](<%5B-a-z0-9%5D*%5Ba-z0-9%5D>)?,最大长度为63位  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8e1d0257856707e043431e6c40477400.png)  
2.通过yml文件创建
    
    
    #切换到k8s的配置文件存储目录
    cd /etc/kubernetes/
    #创建用户的配置文件存储目录
    mkdir user-root
    #进入用户文件夹
    cd user-root/
    #创建命名空间二的配置文件
    touch testcreatenamespaces2.yml
    #使用vi编辑器将以下内容输入
    apiVersion: v1
    kind: Namespace
    metadata:
        name: testcreatenamespaces2
    #注意：yml文件需要严格的缩进，类似python语法一样，其数据结构与层级是通过缩进区分的，但是不必须几#个空格或者tab缩进，只要有缩进即可。
    #运行配置
    kubectl apply -f testcreatenamespaces2.yml
    #查看所有的命名空间
    kubectl get namespaces --all-namespaces
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6eb9a5e5db20f8756d16676150c269d2.png)

#### 4.1.3删除命名空间
    
    
    kubectl delete namespaces testcreatenamespaces1
    

1.删除一个namespace会自动删除所有属于该namespace的资源。  
2.default和kube-system命名空间不可删除。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e69dfa192ef12aeae4cebe162fb9a4d9.png)

### 4.2部署
    
    
    kubectl run httpd-app --image=httpd --replicas=2 -n testcreatenamespaces2
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dce1bd80a83b84e07a2c9e627ffc2c19.png)  
这条命令创建一个deployment，然后指定images是httpd，部署两个副本，在testcreatenamespaces2中执行。当然还未准备好，需要等待一段时间。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/56cbb58e22c94509bbabc9c4fd5d163d.png)  
然后在node1和node2的节点上就会运行对应的容器  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1fae48e8076a528c64378a75ea6f34b3.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3561b33322125590fa0d0b084fba0e4b.png)  
因为我的node2的网络出现了问题，ip地址重复。所以重启node2后，等待一段时间，完成。

  * a.kubectl发送部署请求到api server
  * b.api server通知controller manager创建deployment资源
  * c.kube-scheduler执行调度，将副本pod分发到k8s-node1和k8s-node2
  * d.k8s-node1和k8s-node2上的kubectl在各自的节点上创建并运行pod
