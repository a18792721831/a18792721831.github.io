---
layout: post
title: "k8s-kubernetes-configmap"
date: 2019-10-28 19:17:29 +0800
categories: [k8s, kubernetes, configMap, 统一配置, pod内如何使用configMap]
description: "本文深入探讨Kubernetes ConfigMap的创建与使用，包括四种创建方式：字符串、env文件、目录和定义文件。并详细讲解了ConfigMap在Pod中的三种应用方式：作为环境变量、command参数和volume挂载。"
keywords: k8s, kubernetes, configMap, 统一配置, pod内如何使用configMap
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/102786860
> - 发布时间：2019-10-28 19:17:29
> - 阅读量：573
> - 分类：kubenetes专栏收录该内容, 订阅专栏
> - 标签：#k8s, #kubernetes, #configMap, #统一配置, #pod内如何使用configMap

## 摘要

文章浏览阅读573次。本文深入探讨Kubernetes ConfigMap的创建与使用，包括四种创建方式：字符串、env文件、目录和定义文件。并详细讲解了ConfigMap在Pod中的三种应用方式：作为环境变量、command参数和volume挂载。

---

#### k8s-kubernetes-configmap

  * 1.configmap
  * 2.configmap创建
  *     * 2.1 key-value字符串创建
    * 2.2 env文件创建
    * 2.3 从目录创建
    * 2.4 yaml/json创建
  * 3.使用
  *     * 3.1 pod内env
    * 3.2 command
    * 3.3 volume挂载
  * 4.总结

## 1.configmap

configmap用于保存配置数据的键值对，可以用来保存单个属性，也可以用来保存配置文件。configmap和secret很类似，但他可以更方便的处理不包含敏感信息的字符串。

## 2.configmap创建

### 2.1 key-value字符串创建
    
    
    kubectl create configmap test -n study --from-literal=test.hello=hello --from-literal=testhello.hi=hi
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/01d72ad0f2d74c338c5f533dcd13df68.png)  
当然也可以使用格式化输出形式查看  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8d2d9976e3989648c8a377028e1905c4.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4d0c234af36a8f426360db15c7d19b73.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b65bc4ba64e3a31431391e990a5d6a3e.png)  
为了后面例子能够串起来，我们删除这个configmap，然后重新创建一个：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2ae7c8a4dcc87e58a06b67f4e1aa2606.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6f307aa2d57934ed6914a51f204b2ce4.png)

### 2.2 env文件创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eadfa8e5e0591081ddd5cd8ed29ee14c.png)

### 2.3 从目录创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/92eea4e0842fdd7bc92e044636217edd.png)

### 2.4 yaml/json创建

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf8c09692b648f99febec8b5199e8038.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b912d6a025bfeced03c440c74d9583e5.png)

## 3.使用

### 3.1 pod内env

创建一个pod:  
useforenv.yaml
    
    
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
        env:
        - name: test_command_hello
          valueFrom:
            configMapKeyRef:
              name: testcommand
              key: test.command.hello
        - name: test_command_hi
          valueFrom:
            configMapKeyRef:
              name: testcommand
              key: test.command.hi
        - name: test_env_hello
          valueFrom:
            configMapKeyRef:
              name: testenv
              key: test.env.hello
        - name: testenvhello
          valueFrom:
            configMapKeyRef:
              name: testenv
              key: testenvhello
        - name: test_env_hi
          valueFrom:
            configMapKeyRef:
              name: testenv
              key: test.env.hi
        image: tomcat
        imagePullPolicy: IfNotPresent
        command: ["/usr/local/tomcat/bin/catalina.sh","run"]
        workingDir: /usr/local/tomcat/
        volumeMounts:
        - name: tomcat-log
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
          server: 10.0.228.93
          path: /userdata/testtomcatlog
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5456101ba5aef05d9836c872d867177c.png)

### 3.2 command

先创建一个configmap  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e43d7f2b9b9884e6bf1c9d69fa85ac8e.png)  
在command中使用configmp需要将configmap先用3.1的方式设置为环境变量，然后在command中用$(envName)的方式使用。  
useforcommand.yaml
    
    
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
        command: ["/usr/local/tomcat/bin/catalina.sh","$(command_test_run)"]
        workingDir: /usr/local/tomcat/
        env:
        - name: command_test_run
          valueFrom:
            configMapKeyRef:
              name: command-run
              key: command.test.run
        volumeMounts:
        - name: tomcat-log
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
          server: 10.0.228.93
          path: /userdata/testtomcatlog
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9b07c0442265f58f26757d3e425b2ad4.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/29e7dfcb2580c614518a09e099a08f9f.png)  
已经启动了。。

### 3.3 volume挂载

创建configmap可以根据文件及目录进行创建，同样的，在使用的时候，可以根据configmap恢复成文件目录  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25fe8395978b9b08a4abb8759ebb1699.png)  
根据这个目录创建configmap  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c21efa2a2aa6a3160a3d8905153d849c.png)  
在pod中使用  
useforvolume.yaml
    
    
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
        env:
        - name: test_command_hello
          valueFrom:
            configMapKeyRef:
              name: testcommand
              key: test.command.hello
        - name: test_command_hi
          valueFrom:
            configMapKeyRef:
              name: testcommand
              key: test.command.hi
        - name: test_env_hello
          valueFrom:
            configMapKeyRef:
              name: testenv
              key: test.env.hello
        - name: testenvhello
          valueFrom:
            configMapKeyRef:
              name: testenv
              key: testenvhello
        - name: test_env_hi
          valueFrom:
            configMapKeyRef:
              name: testenv
              key: test.env.hi
        image: tomcat
        imagePullPolicy: IfNotPresent
        command: ["/usr/local/tomcat/bin/catalina.sh","run"]
        workingDir: /usr/local/tomcat/
        volumeMounts:
        - name: tomcat-log
          mountPath: /usr/local/tomcat/logs/
          readOnly: false
        - name: testdir
          mountPath: /usr/local/test/
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
          server: 10.0.228.93
          path: /userdata/testtomcatlog
      - name: testdir
        configMap:
          name: testdir
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d21d9b38f85806a846bdd4984eec5f7.png)

## 4.总结

  * configmap就是k8s集群中公共的key-value集合，所有的pod都可以使用。
  * configmap也是k8s中一类资源
  * configmap有4种基本的创建方式：字符串、env、文件目录、定义文件
  * configmap在pod中有3种使用方式：env、command、volume挂载
