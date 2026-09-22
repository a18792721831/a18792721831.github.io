---
layout: post
title: "k8s-kubenetes安装"
date: 2019-10-14 18:57:22 +0800
categories: [k8s入门, kubenates入门, kubeadm, docker, kubenetes安装使用]
description: "本文详细介绍如何使用kubeadm在虚拟机环境下快速部署Kubernetes集群，包括准备工作、k8s介绍、安装步骤、常用命令及异常处理。适用于初学者快速上手Kubernetes。"
keywords: k8s入门, kubenates入门, kubeadm, docker, kubenetes安装使用
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/102554321
> - 发布时间：2019-10-14 18:57:22
> - 阅读量：1
> - 分类：kubenetes专栏收录该内容, 订阅专栏
> - 标签：#k8s入门, #kubenates入门, #kubeadm, #docker, #kubenetes安装使用

## 摘要

文章浏览阅读1k次。本文详细介绍如何使用kubeadm在虚拟机环境下快速部署Kubernetes集群，包括准备工作、k8s介绍、安装步骤、常用命令及异常处理。适用于初学者快速上手Kubernetes。

---

#### k8s-kubenetes安装

  * 1.准备工作
  * 2.k8s介绍
  * 3.k8s安装
  *     * 3.1初始化hosts
    * 3.2更新yum
    * 3.3添加kubeadm的yum源
    * 3.4下载kubeadm
    * 3.5关闭linux的swap
    * 3.6关闭防火墙
    * 3.8设置开机启动kubelet
    * 3.9添加docker的yum源
    * 3.10安装docker
    * 3.11设置开机启动docker
    * 3.12配置docker参数
    * 3.13设置iptables的规则
    * 3.14使用kubeadm初始化环境
    * 3.15k8s访问设置
  * 4.k8s常用命令
  *     * 4.1查看集群组成
    * 4.2查看集群状态
    * 4.3安装网络服务(master)
    * 4.4查看k8s所有命名空间
    * 4.5查看命名空间中pod
    * 4.6查看k8s的deployment
    * 4.7重新生成token
  * 5.kubeadm安装过程中发生异常

## 1.准备工作

因为没有机器，且计算机硬件限制，所以使用vm虚拟机模拟。  
硬件方面：  
k8s_master 2G 2core 20G  
k8s_node1 1G 2core 20G  
k8s_node2 1G 2core 20G  
操作系统：  
CentOS Linux release 7.6.1810 (Core)  
网络：  
虚拟机需要连接外网  
静态ip,桥接模式，虚拟机直接访问外网。  
k8s_master 192.168.20.71  
k8s_node1 192.168.20.72  
k8s_node2 192.168.20.73  
用户：  
三台虚拟机都使用root登录，操作  
密码都是centos  
设置好了后，关闭vm，让虚拟机后台运行，然后在本地用ssh连接，ssh工具xshell6。

到此，准备工作完成。

设置虚拟机静态IP：  
vi /etc/sysconfig/network-scripts/ifcfg-ens33  
名称不一定是ens33  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/df7dac891034767f4b32eba4a6452e34.png)  
完成后保存，重启，然后就可以使用ssh连接了。  
然后关闭vm软件，在弹出的提示中选择后台运行。  
需要关闭的时候再次打开vm软件即可。

## 2.k8s介绍

k8s是干什么的，什么原理等等，网上多的是。  
k8s安装方式有以下几种：  
1.从网上下载二进制文件，然后启动（不推荐）  
2.使用kubeadm（推荐）  
3.从github下载源码，自行编译、启动（不推荐）  
4.使用docker从docker仓库拉取（推荐）

基本上安装方式就上述4类，实际上2、4是一类方式。

不过最推荐的就是使用kubeadm进行安装，因为k8s涉及到多个服务共同协作，每个服务的配置比较多，如果手动一个一个的启动就非常的麻烦。

而且kubeadm一开始只是社区自发开发的安装脚本，后续经过考验，官方也开始支持并推荐使用kubeadm进行安装。

kubeadm的安装方式就是从google仓库下载二进制文件，然后生成配置文件，将一些初始化配置写入配置文件，然后启动服务即可。

## 3.k8s安装

### 3.1初始化hosts

因为我们是1主2副，所以需要在hosts配置主机名。配置了主机名，在需要IP的地方就可以用主机名代替，后续换IP只需要更新hosts即可，其他的无需修改。
    
    
    #!/bin/bash
    echo -e "192.168.20.71 k8s-master\
    \n192.168.20.72 k8s-node1\
    \n192.168.20.73 k8s-node2\
    \n127.0.0.1 k8s-master" >> /etc/hosts
    
    

执行上述脚本后，重启网络服务`service network restart`  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/72efc3d428e9d5a6fce311bf828a6987.png)  
查看修改结果```cat /etc/hosts  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2c326e269df5574dd3dd2dfdd5ad5bc8.png)

注意 ： 需要修改127对应的主机名  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ce1b14438ba1abfde056207295ea5004.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/74eb25831a109ef84cfad12a07a8b6a5.png)  
测试连通性  
ping  
这三个虚拟机能够相互ping通  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3d445a12b172a69bc3661d1cae0afddb.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/80dc19898969ca48c1adeb3a687d765a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a93f75e185378c7e3259c1cfd4bde3f8.png)

### 3.2更新yum
    
    
    yum clean all
    yum makecache
    yum update -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/77271cc017f3465714864a208af7f4c7.png)  
![发生异常重启重新执行命令](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b066750b3cf7a3ca25d81ceea385943e.png)

### 3.3添加kubeadm的yum源
    
    
    echo -e "[kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=0" >> /etc/yum.repos.d/kubernetes.repo
    yum makecacahe fast
    yum update -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/761be55a394270197224047e6543cdb4.png)

### 3.4下载kubeadm
    
    
    #查看kubeadm的信息
    yum list kubeadm --showduplicates|sort -r
    yum install -y kubeadm 版本
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/499b629a5e6ff17e5e5c7b7f2ec95886.png)

我们下载最新的  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cac8ee95c0585d77bc0ad5c44d9d3858.png)  
`yum install -y kubeadm 1.9.9-0`  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/749203aedb3913d2b5192eb6ddc0e7d1.png)  
可以看出，安装kubeadm将会安装kubelet、kubernetes-cni、kubectl等服务

### 3.5关闭linux的swap

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/94edd72be2aa7961b3477b171164e600.png)
    
    
    vi /etc/fstab
    swapoff -a && swapon -a
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/677358f01870b93385a88ebab0437178.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/58dec88370e858002b42da2073ba01f8.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/56314a8e5e5e9f761fc5149f177a9a80.png)

### 3.6关闭防火墙
    
    
    systemctl disable firewalld
    systemctl stop firewalld
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9d2b0eff729f268093915f56b5112783.png)

### 3.8设置开机启动kubelet
    
    
    systemctl stop kubelet
    systemctl enable kubelet
    systemctl start kubelet
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8438fd8c0d86a67b76c2b6b27c744bc0.png)  
这个时候启动失败是正确的，因为还没有使用kubeadm初始化环境。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8af9d8bc0cf7bebe230d51eb70d328bf.png)

### 3.9添加docker的yum源
    
    
    yum install yum-utils -y
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum makecacahe fast
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/24c049b0866de0842ce598492cdbe1c9.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b802053279eace3c0bf4ca50c95f2598.png)  
也可以使用wget手动添加yum源，见3.3  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7d8f62baebbb7105dc81305f9a3eb1c4.png)

### 3.10安装docker
    
    
    yum list docker-ce --showduplicates|sort -r
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4735ad37ceab506796e5d3be7745e3e6.png)  
我们安装最新的
    
    
    yum install -y docker-ce 3:19.03.3-3.el7
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ffcbcb32f918e9bd91e31b14f5ccb48b.png)

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/afd512fde973d468b2191ef62d877f01.png)  
使用yum安装的时候遇到错误，导致安装失败，一般清除yum缓存，更新yum，重新安装即可解决大多数的问题。（前提不是网络、硬件等问题）

### 3.11设置开机启动docker
    
    
    systemctl stop docker
    systemctl enable docker
    systemctl start docker
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/771eaf8221e4890765f753c9d237888c.png)

### 3.12配置docker参数
    
    
    echo -e "{\n\
    \"registry-mirrors\": [\"https://xxxx.mirror.aliyuncs.com\"],\n\
    \"exec-opts\": [\"native.cgroupdriver=systemd\"]\n\
    }" >> /etc/docker/daemon.json
    

这里配置docker使用阿里的仓库  
然后将docker服务设置为系统服务  
然后重启docker服务
    
    
    systemctl daemon-reload
    systemctl restart docker
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2f75fe10bdf4f911822afb23e39df80c.png)

### 3.13设置iptables的规则
    
    
    modprobe br_netfilter
    echo 1 >> /proc/sys/net/bridge/bridge-nf-call-iptables
    echo 1 >> /proc/sys/net/bridge/bridge-nf-call-ip6tables
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e25eec87e1d1b6fb7ff7cf5d358d1524.png)

一定记得重启。

### 3.14使用kubeadm初始化环境

注意：上述13个步骤需要在所有的虚拟机上执行，也就是说不管是master还是node都需要做的操作。
    
    
    kubeadm init \
    --apiserver-advertise-address=192.168.20.71 \
    --image-repository=registry.aliyuncs.com/google_containers \
    --kubernetes-version v1.16.1 \
    --pod-network-cidr=10.244.0.0/16
    

其中–apiserver-advertise-address表示k8s_master,  
–image-repository表示使用阿里的云仓库(因为kubenetes的官方镜像在google服务中，因为GFW的缘故，需要vpn才能从google下载，而且速度很慢)  
–kubernetes-version表示我们将要初始化的是v1.16.1的环境，最新的版本可以在github上查看  
[传送门](<https://github.com/kubernetes/kubernetes>)  
–pod-network-cidr表示网关

最新的就是![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ff401faa2e9066d62b5d4b3e00d8327f.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bcee510515aa8746bf3371e14496961b.png)  
然后kubeadm就会从设置的镜像仓库下载镜像了，这里也就能说明为什么使用kubeadm安装k8s和使用docker镜像安装是一回事了，他们都是一样的。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f07f0847795c6c71b4ef60609a2d84bb.png)  
到此，我们的master就安装好了。

### 3.15k8s访问设置

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f5a64ad7b02cfaf278824300fef6263c.png)  
在这张图中，提示中说明了访问k8s集群需要做哪些事情：
    
    
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    

用大白话说就是哪个用户需要访问集群，就需要将master中的admin.conf文件作为用户的访问配置。  
因为kubeadm在初始化环境的时候会初始化证书，然后k8s集群在交换数据时，都是使用restful请求完成的。

所以普通用户需要访问就需要证书。  
这和你第一次在12306买火车票时，12306要求你安装证书是一样的。  
所以，就需要将这个文件拷贝到用户下，然后赋予权限即可。  
我们都是用root用户操作的。  
k8s-master
    
    
    #删除原来的目录
    rm -rf /root/.kube
    #创建需要的目录
    mkdir /root/.kube
    #拷贝文件
    cp -i /etc/kubernetes/admin.conf /root/.kube/config
    #更改文件所有者
    chown root:root /root/.kube/config
    

接下来看下这个文件到底是啥  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ceec92902cbc1203b9dde22734f9a786.png)  
一些配置，和三个证书。  
一定记下这个token.  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f5a64ad7b02cfaf278824300fef6263c.png)  
拷贝出来放到其他地方。  
接下来安装k8s-node1,k8s-node2加入k8s-master集群，使用上图红框中的命令即可
    
    
    kubeadm join 192.168.20.71:6443 --token ihj7xn.256lev80oztuz346 \
        --discovery-token-ca-cert-hash sha256:69a495502f215f3352c863e5ef52fa7229d24c3a74481a0661aa4ad65f3a2de3
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/549618931d93b18394ea0a1dc23b1ff0.png)  
到了这里一套k8s的环境就搭建完了。

## 4.k8s常用命令

### 4.1查看集群组成
    
    
    kubectl get nodes
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ac250c7ab26e04299dc0afbe96121522.png)

### 4.2查看集群状态
    
    
    kubectl get cs
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c45adb9967a5ca3176c33cac081827e4.png)  
显示这个网上找了一些资料，大概就是说这个不影响正常使用。是因为集群条件下启动，那么controller-manager服务、scheduler服务、etcd服务可能不在一个主机上，所以这里都是unknown，不过这些服务都是正常运行的。

不管k8s怎么操作，k8s都是基于docker 运行的，所以查看docker的容器使用情况也可以看出k8s服务是否正常运行
    
    
    docker ps -a
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bc4eb2a89770c087d9e2e72d273d5f0a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ba69344702ae4a9debe4bdd591eb8a74.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9f923e9de63e2f944fdfa793231d3bfe.png)  
可以看到这些服务都是正常启动的。

还可以通过pod进行查看
    
    
    kubectl get pod --all-namespaces -o wide
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bf7c40e361ab8012c65926e3baa9d2b1.png)  
因为还未安装网络相关的服务，所以coredns服务是不运行的。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c1cb6f8d7aac92a7d0f0ba7b8196d702.png)  
在node上查询的结果相同。

### 4.3安装网络服务(master)
    
    
    cd /etc/kubernetes/
    wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    kubectl apply -f  kube-flannel.yml
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4fa88321308d51d0bfa8f0674d494c42.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e2bca5a42aa87f6d8431b67120675ff2.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/05026f2c791e4e87d901b495d437c58c.png)

### 4.4查看k8s所有命名空间
    
    
    kubectl get namespaces --all-namespaces
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/eaf5a0f25534fd112a5e696f49b3bd6c.png)

### 4.5查看命名空间中pod
    
    
    kubectl get pod --namespace kube-system
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4a0d08b196131aaf24858632284817b2.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/14fa3c407d144e65071733dd24f92885.png)

### 4.6查看k8s的deployment
    
    
    kubectl get deployment --all-namespaces
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e5546a8676f06aabeb1b84a27a1119a1.png)

### 4.7重新生成token

因为默认的token是24小时有效期，过期了就无法在使用token加入集群了，所以当token过期后，需要重新生成token,或者设置tocken永不过期(自己玩玩就行，生产环境千万不能用)
    
    
    kubeadm join 192.168.20.71:6443 --token rnnstv.3s9sc9memaiaeakp \
        --discovery-token-ca-cert-hash sha256:45c8acbd18490a5ab9a7db672c09692fc81e2bd718a9ce95f35b6e0f7cb3f6cc
    

这个命令大致包含三个关键参数  
1.master的ip  
2.token  
3.token-hash
    
    
    #如果过期可先执行此命令
    kubeadm token create    #重新生成token
    #列出token
    kubeadm token list  | awk -F" " '{print $1}' |tail -n 1
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7db719ee615e3a63c7dde42d64a5afea.png)

## 5.kubeadm安装过程中发生异常

如果在kubeadm初始化之后发生错误，可以使用
    
    
    kubeadm reset
    

进行重置kubeadm环境，然后重新使用kubeadm init进行初始化。
