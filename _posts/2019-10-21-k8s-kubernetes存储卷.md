---
layout: post
title: "k8s-kubernetes存储卷"
date: 2019-10-21 18:36:46 +0800
categories: [k8s存储, kubernetes, volume类型, nfs, pv]
description: "本文详细介绍了Kubernetes中各种存储卷类型，包括emptyDir、hostPath、NFS、gcePersistentDisk、awsElasticBlockStore等，以及如何使用subPath、FlexVolume、ProjectedVolume等高级特性，帮助读者理解容器数据持久化和共享数据的解决方案。"
keywords: k8s存储, kubernetes, volume类型, nfs, pv
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/102669382
> - 发布时间：2019-10-21 18:36:46
> - 阅读量：792
> - 分类：kubenetes专栏收录该内容, 订阅专栏
> - 标签：#k8s存储, #kubernetes, #volume类型, #nfs, #pv

## 摘要

文章浏览阅读792次。本文详细介绍了Kubernetes中各种存储卷类型，包括emptyDir、hostPath、NFS、gcePersistentDisk、awsElasticBlockStore等，以及如何使用subPath、FlexVolume、ProjectedVolume等高级特性，帮助读者理解容器数据持久化和共享数据的解决方案。

---

#### k8s-kubernetes存储卷

  * [1.Volume类型](<#1Volume_5>)
  * [2.emptyDir](<#2emptyDir_30>)
  * [3.hostPath](<#3hostPath_48>)
  * [4.NFS](<#4NFS_67>)
  * [5.gcePersistentDisk](<#5gcePersistentDisk_77>)
  * [6.awsElasticBlockStore](<#6awsElasticBlockStore_87>)
  * [7.gitRepo](<#7gitRepo_97>)
  * [8.使用subPath](<#8subPath_106>)
  * [9.FlexVolume](<#9FlexVolume_132>)
  * [10.Projected Volume](<#10Projected_Volume_145>)
  * [11.本地存储限额](<#11_184>)

  
我们知道默认情况下容器的数据都是非持久化的，在容器消亡以后数据也跟着丢失，所以Docker提供了Volume机制以便将数据持久化存储。   
类似的，Kubernetes提供了更强大的Volume机制和丰富的插件，解决了容器数据持久化和容器间共享数据的问题。   
与Docker不同，Kubernetes Volume的生命周期与Pod绑定容器挂掉后Kubelet再次重启容器时，Volume的数据依然还在而Pod删除时，Volume才会清理。   
数据是否丢失取决于具体的Volume类型，比如emptyDir的数据会丢失，而PV的数据则不会丢失。 

## 1.Volume类型

kubernetes支持一下Volume类型：

  * emptyDir
  * hostPath
  * gcePersistentDisk
  * awsElasticBlockStore
  * nfs
  * iscsi
  * flocker
  * glusterfs
  * rbd
  * cephfs
  * gitRepo
  * secret
  * persistentVolumeClaim
  * downwardAPI
  * azureFileVolume
  * azureDisk
  * vsphereVolume
  * Quobyte
  * PortworxVolume
  * ScaleIO
  * FlexVolume
  * StorageOS
  * local

## 2.emptyDir

如果Pod设置了emptyDir类型Volume， Pod 被分配到Node上时候，会创建emptyDir，只要Pod运行在Node上，emptyDir都会存在（容器挂掉不会导致emptyDir丢失数据），但是如果Pod从Node上被删除（Pod被删除，或者Pod发生迁移），emptyDir也会被删除，并且永久丢失。
    
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pd
    spec:
      containers:
      - image: gcr.io/google_containers/test-webserver
      name: test-container
      volumeMounts:
      - mountPath: /cache
        name: cache-volume
      volumes:
      - name: cache-volume
        emptyDir: {}
    

## 3.hostPath

hostPath允许挂载Node上的文件系统到Pod里面去。如果Pod需要使用Node上的文件，可以使用hostPath。
    
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pd
    spec:
      containers:
      - image: gcr.io/google_containers/test-webserver
        name: test-container
        volumeMounts:
        - mountPath: /test-pd
          name: test-volume
      volumes:
      - name: test-volume
        hostPath:
          path: /data
    

## 4.NFS

NFS 是Network File System的缩写，即网络文件系统。Kubernetes中通过简单地配置就可以挂载NFS到Pod中，而NFS中的数据是可以永久保存的，同时NFS支持同时写操作。
    
    
    volumes:
    - name: nfs
      nfs:
        # FIXME: use the right hostname
        server: 10.254.234.223
        path: "/"
    

## 5.gcePersistentDisk

gcePersistentDisk可以挂载GCE上的永久磁盘到容器，需要Kubernetes运行在GCE的VM中。
    
    
    volumes:
      - name: test-volume
        # This GCE PD must already exist.
        gcePersistentDisk:
          pdName: my-data-disk
          fsType: ext4
    

## 6.awsElasticBlockStore

awsElasticBlockStore可以挂载AWS上的EBS盘到容器，需要Kubernetes运行在AWS的EC2上。
    
    
    volumes:
      - name: test-volume
        # This AWS EBS volume must already exist.
        awsElasticBlockStore:
          volumeID: <volume-id>
          fsType: ext4
    

## 7.gitRepo

gitRepo volume将git代码下拉到指定的容器路径中
    
    
    volumes:
      - name: git-volume
        gitRepo:
          repository: "git@somewhere:me/my-git-repository.git"
          revision: "22f1d8406d464b0c0874075539c1f2e96c253775"
    

## 8.使用subPath

Pod的多个容器使用同一个Volume时，subPath非常有用
    
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-lamp-site
    spec:
      containers:
      - name: mysql
        image: mysql
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: site-data
          subPath: mysql
      - name: php
        image: php
        volumeMounts:
        - mountPath: /var/www/html
          name: site-data
          subPath: html
      volumes:
      - name: site-data
        persistentVolumeClaim:
          claimName: my-lamp-site-data
    

## 9.FlexVolume

如果内置的这些Volume不满足要求，则可以使用FlexVolume实现自的Volume插件。  
注意要把volume plugin放到 /usr/libexec/kubernetes/kubelet-plugins/volume/exec/<vendor~driver>/ ，plugin要实现 init/attach/detach/mount/umount 等命令（可参考lvm的示例）。
    
    
    - name: test
      flexVolume:
        driver: "kubernetes.io/lvm"
        fsType: "ext4"
        options:
          volumeID: "vol1"
          size: "1000m"
          volumegroup: "kube_vg"
    

## 10.Projected Volume

Projected volume将多个Volume源映射到同一个目录中，支持secret、downwardAPI和configMap。
    
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: volume-test
    spec:
      containers:
      - name: container-test
        image: busybox
        volumeMounts:
        - name: all-in-one
          mountPath: "/projected-volume"
          readOnly: true
      volumes:
      - name: all-in-one
        projected:
          sources:
          - secret:
            name: mysecret
            items:
              - key: username
                path: my-group/my-username
          - downwardAPI:
            items:
              - path: "labels"
                fieldRef:
                  fieldPath: metadata.labels
              - path: "cpu_limit"
                resourceFieldRef:
                  containerName: container-test
                  resource: limits.cpu
          - configMap:
            name: myconfigmap
            items:
              - key: config
                path: my-group/my-config
    

## 11.本地存储限额

v1.7+支持对基于本地存储（如hostPath, emptyDir, gitRepo等）的容量进行调度限额，可以通过 --feature-gates=LocalStorageCapacityIsolation=true 来开启这个特性。

为了支持这个特性，Kubernetes将本地存储分为两类

  * storage.kubernetes.io/overlay ，即 /var/lib/docker 的大小
  * storage.kubernetes.io/scratch ，即 /var/lib/kubelet 的大小  
Kubernetes根据 storage.kubernetes.io/scratch 的大小来调度本地存储空间，而根据 storage.kubernetes.io/overlay 来调度容器的存储。比如为容器请求64MB的可写层存储空间.

    
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: ls1
    spec:
      restartPolicy: Never
      containers:
      - name: hello
        image: busybox
        command: ["df"]
        resources:
          requests:
            storage.kubernetes.io/overlay: 64Mi
    

为empty请求64MB的存储空间
    
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: ls1
    spec:
      restartPolicy: Never
      containers:
      - name: hello
        image: busybox
        command: ["df"]
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        emptyDir:
          sizeLimit: 64Mi