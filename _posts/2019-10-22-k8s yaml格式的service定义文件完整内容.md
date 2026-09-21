---
layout: post
title: "k8s yaml格式的service定义文件完整内容"
date: 2019-10-22 17:36:26 +0800
categories: [kubernetes, k8s, service, yaml, yaml解析]
description: "本文深入解析Kubernetes中Service资源的配置细节，包括元数据、标签、类型、虚拟服务地址、会话亲和性、端口配置及外部负载均衡器设置等关键概念。"
keywords: kubernetes, k8s, service, yaml, yaml解析
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/102687141
> - 发布时间：2019-10-22 17:36:26
> - 阅读量：1
> - 分类：kubenetes专栏收录该内容, 订阅专栏
> - 标签：#kubernetes, #k8s, #service, #yaml, #yaml解析

## 摘要

文章浏览阅读1.8k次。本文深入解析Kubernetes中Service资源的配置细节，包括元数据、标签、类型、虚拟服务地址、会话亲和性、端口配置及外部负载均衡器设置等关键概念。

---

apiVersion: v1
    kind: Service
    matadata:                                #元数据
      name: string                           #service的名称
      namespace: string                      #命名空间
      labels:                                #自定义标签属性列表
        - name: string
      annotations:                           #自定义注解属性列表
        - name: string
    spec:                                    #详细描述
      selector: []                           #label selector配置，将选择具有label标签的Pod作为管理 范围
      type: string                           #service的类型，指定service的访问方式，默认为clusterIp
      clusterIP: string                      #虚拟服务地址
      sessionAffinity: string                #是否支持session
      ports:                                 #service需要暴露的端口列表
      - name: string                         #端口名称
        protocol: string                     #端口协议，支持TCP和UDP，默认TCP
        port: int                            #服务监听的端口号
        targetPort: int                      #需要转发到后端Pod的端口号
        nodePort: int                        #当type = NodePort时，指定映射到物理机的端口号
      status:                                #当spce.type=LoadBalancer时，设置外部负载均衡器的地址
        loadBalancer:                        #外部负载均衡器
          ingress:                           #外部负载均衡器
            ip: string                       #外部负载均衡器的Ip地址值
            hostname: string                 #外部负载均衡器的主机名