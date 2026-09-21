---
layout: post
title: "【转载】kunbernetes-基于NFS的存储"
date: 2019-10-21 20:08:36 +0800
categories: [kubernetes, nfs, pv, pvc, nfs参数]
description: "本文详述了NFS在网络文件系统中的角色及其在Kubernetes环境下的配置与使用，包括服务端配置、作为volume及PersistentVolume的应用，以及通过nfs-provisioner实现动态存储供给的过程。"
keywords: kubernetes, nfs, pv, pvc, nfs参数
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/102670859
> - 发布时间：2019-10-21 20:08:36
> - 阅读量：491
> - 分类：kubenetes专栏收录该内容, 订阅专栏
> - 标签：#kubernetes, #nfs, #pv, #pvc, #nfs参数

## 摘要

文章浏览阅读491次。本文详述了NFS在网络文件系统中的角色及其在Kubernetes环境下的配置与使用，包括服务端配置、作为volume及PersistentVolume的应用，以及通过nfs-provisioner实现动态存储供给的过程。

---

转自：https://www.kubernetes.org.cn/4022.html

## **1****、NFS介绍**

NFS是Network File System的简写，即网络文件系统，NFS是FreeBSD支持的文件系统中的一种。NFS基于RPC(Remote Procedure Call)远程过程调用实现，其允许一个系统在网络上与它人共享目录和文件。通过使用NFS，用户和程序就可以像访问本地文件一样访问远端系统上的文件。NFS是一个非常稳定的，可移植的网络文件系统。具备可扩展和高性能等特性，达到了企业级应用质量标准。由于网络速度的增加和延迟的降低，NFS系统一直是通过网络提供文件系统服务的有竞争力的选择 。

### **1.1 nfs****原理**

NFS 使用RPC(Remote Procedure Call)的机制进行实现，RPC使得客户端可以调用服务端的函数。同时，由于有 VFS 的存在，客户端可以像使用其它普通文件系统一样使用 NFS 文件系统。经由操作系统的内核，将 NFS 文件系统的调用请求通过 TCP/IP 发送至服务端的 NFS 服务。NFS服务器执行相关的操作，并将操作结果返回给客户端。

![image](https://i-blog.csdnimg.cn/blog_migrate/0dd67171e5e7023bdd54e52eff00983c.png)

NFS服务主要进程包括：

  * rpc.nfsd：最主要的NFS进程，管理客户端是否可登录
  * rpc.mountd：挂载和卸载NFS文件系统，包括权限管理
  * rpc.lockd：非必要，管理文件锁，避免同时写出错
  * rpc.statd：非必要，检查文件一致性，可修复文件

nfs的关键工具包括：

  * 主要配置文件：/etc/exports；
  * NFS文件系统维护命令：/usr/bin/exportfs；
  * 共享资源的日志文件： /var/lib/nfs/*tab；
  * 客户端查询共享资源命令： /usr/sbin/showmount；
  * 端口配置： /etc/sysconfig/nfs。

### **1.2****共享配置**

在NFS服务器端的主要配置文件为/etc/exports时，通过此配置文件可以设置共享文件目录。每条配置记录由NFS共享目录、NFS客户端地址和参数这3部分组成，格式如下：

[NFS共享目录] [NFS客户端地址1(参数1,参数2,参数3……)] [客户端地址2(参数1,参数2,参数3……)]

  * NFS共享目录：服务器上共享出去的文件目录；
  * NFS客户端地址：允许其访问的NFS服务器的客户端地址，可以是客户端IP地址，也可以是一个网段(192.168.64.0/24)；
  * 访问参数：括号中逗号分隔项，主要是一些权限选项。

**1****）访问权限参数**

| **序号** |  **选项** |  **描述**                 |
|--------|---------|-------------------------|
| 1      | ro      | 客户端对于共享文件目录为只读权限。（默认设置） |
| 2      | rw      | 客户端对共享文件目录具有读写权限。       |

**2****）用户映射参数**

| 序号  |  选项            |  描述                                                  |
|-----|----------------|------------------------------------------------------|
| 1   | root_squash    | 使客户端使用root账户访问时，服务器映射为服务器本地的匿名账号。                    |
| 2   | no_root_squash | 客户端连接服务端时如果使用的是root的话，那么也拥有对服务端分享的目录的root权限。         |
| 3   | all_squash     | 将所有客户端用户请求映射到匿名用户或用户组（nfsnobody）。                    |
| 4   | no_all_squash  | 与上相反（默认设置）。                                          |
| 5   | anonuid=xxx    | 将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）。             |
| 6   | anongid=xxx    | 将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）。 |

**3****）其它配置参数**

| 序号  |  选项        |  描述                                                 |
|-----|------------|-----------------------------------------------------|
| 1   | sync       | 同步写操作，数据写入存储设备后返回成功信息。（默认设置）                        |
| 2   | async      | 异步写操作，数据在未完全写入存储设备前就返回成功信息，实际还在内存。                  |
| 3   | wdelay     | 延迟写入选项，将多个写操请求合并后写入硬盘，减少I/O次数，NFS非正常关闭数据可能丢失（默认设置）。 |
| 4   | no_wdelay  | 与上相反，不与async同时生效，如果NFS服务器主要收到小且不相关的请求，该选项实际会降低性能。   |
| 5   | subtree    | 若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)；                |
| 6   | no_subtree | 即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率             |
| 7   | secure     | 限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）。              |
| 8   | insecure   | 允许客户端从大于1024的tcp/ip端口连接服务器。                         |

## **2****、nfs服务端配置**

在nfs作为网络文件存储系统前，首先，需要安装nfs和rpcbind服务；接着，需要创建使用共享目录的用户；然后，需要对共享目录进行配置，这是其中相对重要和复杂的一个步骤；最后，需要启动rpcbind和nfs服务，以供应用使用。

### **2.1****安装nfs服务**

1）通过yum目录安装nfs服务和rpcbind服务：
    
    
    $ yum -y install nfs-utils rpcbind

2）检查nfs服务是否正常安装
    
    
    $ rpcinfo -p localhost

### **2.2****创建用户**

为NFS服务其添加用户，并创建共享目录，以及设置用户设置共享目录的访问权限：
    
    
    $ useradd -u nfs
    
    
    $ mkdir -p /nfs-share 
    $ chmod a+w /nfs-share

### **2.3****配置共享目录**

在nfs服务器中为客户端配置共享目录：
    
    
    $ echo "/nfs-share *(rw,async,no_root_squash)" >> /etc/exports

通过执行如下命令是配置生效：
    
    
    $exportfs -r

### **2.4****启动服务**

1）由于必须先启动rpcbind服务，再启动nfs服务，这样才能让nfs服务在rpcbind服务上注册成功：
    
    
    $ systemctl start rpcbind

2）启动nfs服务：
    
    
    $ systemctl start nfs-server

3）设置rpcbind和nfs-server开机启动：
    
    
    $ systemctl enable rpcbind
    

$ systemctl enable nfs-server

### **2.5****检查nfs服务是否正常启动**
    
    
    $ showmount -e localhost
    

$ mount -t nfs 127.0.0.1:/data /mnt

## **3****、nfs作为volume**

nfs可以直接作为存储卷使用，下面是一个redis部署的YAML配置文件。在此示例中，redis在容器中的持久化数据保存在/data目录下；存储卷使用nfs，nfs的服务地址为：192.168.8.150，存储路径为：/k8s-nfs/redis/data。容器通过 _volumeMounts.name_ 的值确定所使用的存储卷。
    
    
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: redis
    spec:
      selector:
        matchLabels:
          app: redis
      revisionHistoryLimit: 2
      template:
        metadata:
          labels:
            app: redis
        spec:
          containers:
          # 应用的镜像
          - image: redis
            name: redis
            imagePullPolicy: IfNotPresent
            # 应用的内部端口
            ports:
            - containerPort: 6379
              name: redis6379
            env:
            - name: ALLOW_EMPTY_PASSWORD
              value: "yes"
            - name: REDIS_PASSWORD
              value: "redis"   
            # 持久化挂接位置，在docker中 
            volumeMounts:
            - name: redis-persistent-storage
              mountPath: /data
          volumes:
          # 宿主机上的目录
          - name: redis-persistent-storage
            nfs:
              path: /k8s-nfs/redis/data
              server: 192.168.8.150

## **4****、nfs作为PersistentVolum**

在Kubernetes当前版本的中，可以创建类型为nfs的持久化存储卷，用于为PersistentVolumClaim提供存储卷。在下面的PersistenVolume YAML配置文件中，定义了一个名为nfs-pv的持久化存储卷，此存储卷提供了5G的存储空间，只能由一个PersistentVolumClaim进行可读可写操作。此持久化存储卷使用的nfs服务器地址为192.168.5.150，存储的路径为/tmp。
    
    
    apiVersion: v1 
    kind: PersistentVolume 
    metadata: 
      name: nfs-pv 
    spec: 
      capacity: 
        storage: 5Gi 
      volumeMode: Filesystem 
      accessModes: 
      - ReadWriteOnce 
      persistentVolumeReclaimPolicy: Recycle 
      storageClassName: slow 
      mountOptions: 
      - hard 
      - nfsvers=4.1
      # 此持久化存储卷使用nfs插件 
      nfs:
        # nfs共享目录为/tmp 
        path: /tmp 
        # nfs服务器的地址
        server: 192.168.5.150

通过执行如下的命令可以创建上述持久化存储卷：
    
    
    $ kubectl create -f {path}/nfs-pv.yaml

存储卷创建成功后将处于可用状态，等待PersistentVolumClaim使用。PersistentVolumClaim会通过访问模式和存储空间自动选择合适存储卷，并与其进行绑定。

## **5****、nfs作为动态存储提供**

### **5.1****部署nfs-provisioner**

为nfs-provisioner实例选择存储状态和数据的存储卷，并将存储卷挂接到容器的/export 命令。
    
    
    ...
     volumeMounts:
        - name: export-volume
          mountPath: /export
    volumes:
      - name: export-volume
        hostPath:
          path: /tmp/nfs-provisioner
    ...

为StorageClass选择一个供应者名称，并在deploy/kubernetes/deployment.yaml进行设置。
    
    
    args:
      - "-provisioner=example.com/nfs"
    ...

完整的deployment.yaml文件内容如下：
    
    
    kind: Service
    apiVersion: v1
    metadata:
      name: nfs-provisioner
      labels:
        app: nfs-provisioner
    spec:
      ports:
        - name: nfs
          port: 2049
        - name: mountd
          port: 20048
        - name: rpcbind
          port: 111
        - name: rpcbind-udp
          port: 111
          protocol: UDP
      selector:
        app: nfs-provisioner
    ---
    

apiVersion: extensions/v1beta1  
kind: Deployment  
metadata:  
name: nfs-provisioner  
spec:  
replicas: 1  
strategy:  
type: Recreate  
template:  
metadata:  
labels:  
app: nfs-provisioner  
spec:  
containers:  
- name: nfs-provisioner  
image: quay.io/kubernetes_incubator/nfs-provisioner:v1.0.8  
ports:  
- name: nfs  
containerPort: 2049  
- name: mountd  
containerPort: 20048  
- name: rpcbind  
containerPort: 111  
- name: rpcbind-udp  
containerPort: 111  
protocol: UDP  
securityContext:  
capabilities:  
add:  
- DAC_READ_SEARCH  
- SYS_RESOURCE  
args:  
# 定义提供者的名称，存储类通过此名称指定提供者  
- “-provisioner=nfs-provisioner”  
env:  
- name: POD_IP  
valueFrom:  
fieldRef:  
fieldPath: status.podIP  
- name: SERVICE_NAME  
value: nfs-provisioner  
- name: POD_NAMESPACE  
valueFrom:  
fieldRef:  
fieldPath: metadata.namespace  
imagePullPolicy: “IfNotPresent”  
volumeMounts:  
- name: export-volume  
mountPath: /export  
volumes:  
\- name: export-volume  
hostPath:  
path: /srv

在设置好deploy/kubernetes/deployment.yaml文件后，通过kubectl create命令在Kubernetes集群中部署nfs-provisioner。
    
    
    $ kubectl create -f {path}/deployment.yaml

### **5.2****创建StorageClass**

下面是example-nfs的StorageClass配置文件，此配置文件定义了一个名称为nfs-storageclass的存储类，此存储类的提供者为nfs-provisioner。
    
    
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: nfs-storageclass
      provisioner: nfs-provisioner

通过kubectl create -f命令使用上面的配置文件创建：
    
    
    $ kubectl create -f deploy/kubernetes/class.yaml

storageclass “example-nfs” created

在存储类被正确创建后，就可以创建PersistenetVolumeClaim来请求StorageClass，而StorageClass将会为PersistenetVolumeClaim自动创建一个可用PersistentVolume。

### 5.3 创建PersistenetVolumeClaim

PersistenetVolumeClaim是对PersistenetVolume的声明，即PersistenetVolume为存储的提供者，而PersistenetVolumeClaim为存储的消费者。下面是PersistentVolumeClaim的YAML配置文件，此配置文件通过 _metadata.annotations[].volume.beta.kubernetes.io/storage-class_ 字段指定所使用的存储储类。

在此配置文件中，使用 _nfs-storageclass_ 存储类为PersistenetVolumeClaim创建PersistenetVolume，所要求的PersistenetVolume存储空间大小为1Mi，可以被多个容器进行读取和写入操作。
    
    
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nfs-pvc
      annotations:
        volume.beta.kubernetes.io/storage-class: "nfs-storageclass"
    spec:
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 1Mi

通过kubectl create命令创建上述的持久化存储卷声明：
    
    
    $ kubectl create -f {path}/claim.yaml

### **5.4 创建使用PersistenVolumeClaim的部署**

**** 在这里定义名为busybox-deployment的部署YAML配置文件，使用的镜像为busybox。基于busybox镜像的容器需要对**/mnt** 目录下的数据进行持久化，在YAML文件指定使用名称为nfs的PersistenVolumeClaim对容器的数据进行持久化。
    
    
    # This mounts the nfs volume claim into /mnt and continuously
    # overwrites /mnt/index.html with the time and hostname of the pod. 
    apiVersion: v1
    kind: Deployment
    metadata:  
      name: busybox-deployment
    spec:  
      replicas: 2  
      selector:    
        name: busybox-deployment
      template:    
        metadata:      
          labels:        
            name: busybox-deployment    
        spec:      
          containers:      
          - image: busybox        
            command:          
            - sh          
            - -c          
            - 'while true; do date > /mnt/index.html; hostname >> /mnt/index.html; sleep $(($RANDOM % 5 + 5)); done'        
            imagePullPolicy: IfNotPresent        
            name: busybox        
            volumeMounts:          
            # name must match the volume name below          
            - name: nfs            
              mountPath: "/mnt"     
         # 
         volumes:      
         - name: nfs        
           persistentVolumeClaim:          
             claimName: nfs-pvc

通过kubectl create创建busy-deployment部署：
    
    
    $ kubectl create -f {path}/nfs-busybox-deployment.yaml

## **参考资料**

  1. 《Persistent Volumes》地址：<https://kubernetes.io/docs/concepts/storage/persistent-volumes/>
  2. 《Storage Classes》地址：<https://kubernetes.io/docs/concepts/storage/storage-classes/>
  3. 《Dynamic Volume Provisioning》地址：<https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/>
  4. 《Volums》地址：<https://kubernetes.io/docs/concepts/storage/volumes/>
  5. 《nfs》地址：https://github.com/kubernetes-incubator/external-storage/tree/master/nfs

作者简介：  
季向远，北京神舟航天软件技术有限公司产品经理。本文版权归原作者所有。