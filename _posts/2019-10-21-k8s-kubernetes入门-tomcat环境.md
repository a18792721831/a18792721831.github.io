---
layout: post
title: "k8s-kubernetes入门-tomcat环境"
date: 2019-10-21 18:29:52 +0800
categories: [kubernetes, pod, yaml, k8s实例, k8s入门]
description: "k8s-kubernetes入门-tomcat环境1.命名空间2.选择tomcat镜像3.存储关系确定4.安装nfs服务5.yaml5.启动1.命名空间kubectl create namespace study2.选择tomcat镜像k8s是基于docker的，docker中是以images来管理的。所以我们需要的环境是一个有tomcat容器的images，docker sear..._kubernetes tomcat 前台"
keywords: kubernetes, pod, yaml, k8s实例, k8s入门
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/102669307
> - 发布时间：2019-10-21 18:29:52
> - 阅读量：1
> - 分类：kubenetes专栏收录该内容, 订阅专栏
> - 标签：#kubernetes, #pod, #yaml, #k8s实例, #k8s入门

## 摘要

文章浏览阅读1.2k次。k8s-kubernetes入门-tomcat环境1.命名空间2.选择tomcat镜像3.存储关系确定4.安装nfs服务5.yaml5.启动1.命名空间kubectl create namespace study2.选择tomcat镜像k8s是基于docker的，docker中是以images来管理的。所以我们需要的环境是一个有tomcat容器的images，docker sear..._kubernetes tomcat 前台

---

#### k8s-kubernetes入门-tomcat环境

  * [1.命名空间](<#1_1>)
  * [2.选择tomcat镜像](<#2tomcat_6>)
  * [3.存储关系确定](<#3_14>)
  * [4.安装nfs服务](<#4nfs_60>)
  * [5.yaml](<#5yaml_93>)
  * [5.启动](<#5_159>)

## 1.命名空间
    
    
    kubectl create namespace study
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e435be71176f55940904446be900c23c.png)

## 2.选择tomcat镜像

k8s是基于docker的，docker中是以images来管理的。  
所以我们需要的环境是一个有tomcat容器的images，
    
    
    docker search --no-trunc tomcat
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b24b5643f0d6a9844565407735d6b7f1.png)  
我们使用stars最多的就行。

## 3.存储关系确定

下载docker images到本地，然后用docker启动，进入容器，了解images的基本情况，然后确定哪些目录是需要永久保存的，哪些目录是临时保存的，哪些目录完全不需要。  
这个和k8s的存储配置有关。
    
    
    docker pull tomcat
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/047b1662cb488c218a1d6ea0fb00835f.png)  
接下来启动image,然后进入到image里面
    
    
    docker run -d -p 18080:8080 tomcat
    

-d表示后台运行  
-p进行端口映射表示将容器内8080端口映射到主机的18080上面。  
即请求主机的18080端口就是请求容器内的8080端口。  
tomcat是images的名字，如果是latest那么可以不写，否则需要  
tomcat:tag  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21af272c6547714af2ca930b4ecff2ea.png)
    
    
    docker exec -it d925784f0a /bin/bash
    

exec是执行命令/bin/bash  
-it表示有交互、前台  
d925784f0a是images启动实例。

docker通过dockerfile定义images,通过build将dockerfile打包成images；images通过docker run启动运行实例。即可以这样理解：  
dockerfile是人机交互的容器描述，images是容易静态持久，也是主要传播方式，运行实例是真正访问对象。

类似java：  
.java .class runtime

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3dc1b15bd95319b38352f3e4d87756fc.png)  
接下来我们查看文件目录  
对于tomcat 我们一般之关系两个目录：  
1.logs  
2.webapps  
当然，可能还有一个bin目录，但是一般需要用到bin目录进行重启时，直接重启整个容器即可。  
logs目录  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e085f93c8f37bd3d120de0a60f9cc9df.png)  
webapps目录  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1065c7edc724f69ada834f37688addd0.png)
    
    
    exit
    

退出。  
因为我们使用nfs的存储方式，且先只测试logs目录。  
在主机上创建/userdata/testtomcatlog目录，将主机目录和容器内目录进行映射。

## 4.安装nfs服务
    
    
    yum install -y rpcbind
    yum install -y nfs-utils
    

上述所有的k8s的节点都需要安装，不管是master还是node  
下载安装成功后，在共享目录主机（哪个主机的目录需要被共享就是哪个主机）中创建共享目录。  
然后在共享目录主机上配置
    
    
    vi /etc/exports
    
    
    
    # 共享目录  请求源（可以是IP，通配符，域名等）(读写权限，文件所属权限)
    /userdata/testtomcatlog *(rw,no_root_squash)
    

nfs配置详细信息见  
[kunbernetes-基于NFS的存储](<https://blog.csdn.net/a18792721831/article/details/102670859>)  
然后重启服务

注意：启动有先后顺序，先启动rpcbind,后启动nfs
    
    
    systemctl stop rpcbind
    systemctl stop nfs
    systemctl restart rpcbind
    systemctl restart nfs
    

验证：  
在共享目录的其他主机(非同一台机器)  
上执行
    
    
    showmount -e 共享主机ip
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cc0892328ee38b1203c68dc9c27a9be1.png)

## 5.yaml

新建文件：
    
    
    touch tomcat-study.yml
    
    
    
    apiVersion: v1
    kind: Pod
    metadata: 
      name: tomcat-study
      namespace: study
      labels:
        mytomcat: study
    spec: 
      containers:
      - name: tomcat
        image: tomcat
        imagePullPolicy: IfNotPresent
        command: ["/usr/local/tomcat/bin/catalina.sh","run"]
        #表示后面的命令都是基于workDir的，与dockerfile中的workdir一样
        workingDir: /usr/local/tomcat/
        volumeMounts:
        #name需要与volumes下的name相同
        - name: tomcat-log
          #mountPath需要与容器内的目录对应
          mountPath: /usr/local/tomcat/logs/
          readOnly: false
        ports:
        - name: tomcat-80
          containerPort: 80
          hostPort: 10080
          protocol: TCP
        - name: tomcat-8080
          containerPort: 8080
          hostPort: 18080
          protocol: TCP
        - name: tomcat-443
          containerPort: 443
          hostPort: 10443
          protocol: TCP
        env:
        - name: ORACLE_HOME
          value: oracle
        resources:
          limits:
            cpu: 500m
            memory: 500Mi
          requests:
            cpu: 200m
            memory: 50Mi
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 180
          timeoutSeconds: 30
          periodSeconds: 600
      volumes:
      - name: tomcat-log
        nfs:
          #server是共享目录主机的ip
          server: 10.0.228.93
          #path需要与共享的目录对应
          path: /userdata/testtomcatlog
    

文件格式等见  
[【转载】k8s yaml格式的pod定义文件完整内容](<https://blog.csdn.net/a18792721831/article/details/102611960>)

## 5.启动
    
    
    kubectl apply -f tomcat-study.yml
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f2e7b25d5c6778ad958f1b05f7c60b78.png)  
查看pod
    
    
    kubectl get pods -n study
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7f92aa0ad54dab150ab8883dea445928.png)  
查看详细信息
    
    
    kubectl describe pod tomcat-study -n study
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b5d03eef8e9888930bb42b19f6d734de.png)  
启动成功后，就可以访问node主机的映射端口，进而访问tomcat的管理项目.(因为我们没有进行service管理，即kube-proxy实际上还没有参与到集群管理中，所以需要访问实际启动images的node)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7075efefefc569533c93237d1a8d8844.png)  
然后去共享目录主机上查看日志  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a9e5c9a265686126e830b2efa11019c1.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fde91732e44d623c67e0539d5d7d85c5.png)  
这里的日志并不是实时刷新的，比如我们在浏览器上刷新下，那么实际上需要稍微等待一段时间，日志中才能刷出来。

至此，一个基于nfs的pod就被我们创建成功了。