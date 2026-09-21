---
layout: post
title: "系统占用 docker 容器内的用户授权失败启动失败& postgres 建议挂载的目录是回环目录挂载失败"
date: 2025-10-17 15:38:43 +0800
categories: [1024程序员节, docker, postgresql, docker 权限, 容器]
description: "摘要：Docker启动PostgreSQL容器时遇到挂载目录失败问题。初始报错显示目录权限和用户问题，检查发现宿主机UID 999已被占用。尝试修改目录权限为2000后仍失败。进一步排查发现PostgreSQL 17+版本需挂载/var/lib/postgresql目录而非旧版的/data目录。最终通过正确挂载目录并指定用户权限成功解决问题，提醒网络信息可能过时需注意版本差异。_windows11 docker postgres启动失败"
keywords: 1024程序员节, docker, postgresql, docker 权限, 容器
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/153471314
> - 发布时间：2025-10-17 15:38:43
> - 阅读量：375
> - 分类：docker同时被 2 个专栏收录, 订阅专栏, 技术分享
> - 标签：#1024程序员节, #docker, #postgresql, #docker 权限, #容器

## 摘要

文章浏览阅读375次，点赞3次，收藏6次。摘要：Docker启动PostgreSQL容器时遇到挂载目录失败问题。初始报错显示目录权限和用户问题，检查发现宿主机UID 999已被占用。尝试修改目录权限为2000后仍失败。进一步排查发现PostgreSQL 17+版本需挂载/var/lib/postgresql目录而非旧版的/data目录。最终通过正确挂载目录并指定用户权限成功解决问题，提醒网络信息可能过时需注意版本差异。_windows11 docker postgres启动失败

---

系统占用 docker 容器内的用户授权失败启动失败& postgres 建议挂载的目录是回环目录挂载失败

## 问题

使用docker 启动 postgres 容器，挂载指定目录
    
    
    docker run -d \
    --name pg \
    -v /data/postgres:/var/lib/postgresql/data \
    -p 5432:5432 \
    -e POSTGRES_USER=root \
    -e POSTGRES_PASSWORD=aA123456 \
    postgres
    

直接启动提示错误
    
    
    8f7edd72a4fbfb837ed60f92e55176f8cd9b2511274686479feaa03af1151188
    docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error mounting "/data/postgres" to rootfs at "/var/lib/postgresql/data": change mount propagation through procfd: open o_path procfd: open /data/docker/lib/overlay2/51a975357544cbbe15d37636a0cdc0a0ceb9798886f7fc6a3acbbdf7ace0f13f/merged/var/lib/postgresql/data: no such file or directory: unknown.
    

怀疑是挂载的目录，权限问题
    
    
    chmod -R 700 /data/postgres
    chown -R 999:999 /data/postgres
    

执行上述命令，启动还是失败
    
    
    # root @ test in /data [11:17:13] C:127
    $ chmod -R 700 postgres      
    (base) 
    # root @ test in /data [11:18:26] 
    $ chown -R 999:999 postgres
    (base) 
    # root @ test in /data [11:18:30] 
    $ ls -l
    总用量 36
    drwx--x--x 3 root             root    4096 3月   7 2025 docker
    drwxrwxrwx 2 root             root   16384 2月   5 2025 lost+found
    drwxr-xr-x 5 root             root    4096 5月   9 17:11 mysql
    drwxr-xr-x 4             1001 docker  4096 8月  11 23:29 n8n_zh
    drwxrwxrwx 4 root             root    4096 2月   5 2025 ollama_models
    drwx------ 2 systemd-coredump input   4096 10月 16 19:57 postgres
    (base) 
    

## 原因排查

使用 `ls -l`查看，发现显式的用户不是 999 而是 `systemd-coredump:input`

说明宿主机里面的uid=999 已经被占用了

使用 `getent passwd 999`验证

![image-20251017113106934](https://i-blog.csdnimg.cn/img_convert/da3098f2dad27ee80d7819bbb13d3b00.png)

**systemd-coredump:input**

**systemd-coredump:input** 是核心转储权限控制的核心组，与系统服务强相关，简单来说就是当程序崩溃了，用于生成崩溃的core 文件。

**postgres 容器内使用的用户uid **

  1. 使用 `docker inspect postgres` 查看

![image-20251017142356605](https://i-blog.csdnimg.cn/img_convert/a7555787beaef71ae6b5a83456616e69.png)

为空表示在容器内使用 root 账号

  2. 临时启动容器查看`docker run -it --rm postgres id`

![image-20251017142456859](https://i-blog.csdnimg.cn/img_convert/0ca47990c3816115c7fa742ae0a3790f.png)

表示使用root 账号

  3. 使用镜像构建查看`docker history --no-trunc postgres|grep -i "USER"`

![image-20251017142640110](https://i-blog.csdnimg.cn/img_convert/2d3b9eb775e21ffd24eacec8536f2b8a.png)

有的，这里表示增加 postgres 用户和组，使用uid:gid=999:999

  4. 通过 docker hub 查看

https://hub.docker.com/layers/library/postgres/latest/images/sha256-cdb16ede83438364ee7e4f30052a1b9c8c3e30df43ccfb6b6028a524adcd6001

![Clipboard_Screenshot_1760682775](https://i-blog.csdnimg.cn/img_convert/f3d1b88ad7c3ec24262731e4f0cf148e.png)

## 解决方案

**指定uid:gid**

使用 `cat /etc/passwd | cut -d: -f1,3,4` 查看当前系统中占用的 uid:gid

![image-20251017144436421](https://i-blog.csdnimg.cn/img_convert/a4113167fc8dca3ef68bf7ee4640c1ff.png)

基于这个，我们可以让 postgres 使用 2000 这个uid

`chown -R 2000:2000 /data/postgres`

![image-20251017144542159](https://i-blog.csdnimg.cn/img_convert/56ffe2de0c83525a61f26d90b1323e92.png)

然后在docker 启动的时候，指定 2000 用户
    
    
    docker run -d \
    --name pg \
    -v /data/postgres:/var/lib/postgresql/data \
    -p 5432:5432 \
    -e POSTGRES_USER=root \
    -e POSTGRES_PASSWORD=aA123456 \
    postgres
    

到了这里，我想着肯定成功了吧，现实狠狠地打了脸

还是报错

![image-20251017152708446](https://i-blog.csdnimg.cn/img_convert/7739bd465b187b6d0f17fc6325c2c68e.png)

接着分析容器内是不是有这个目录呢？

首先使用`docker inspect postgres`查看镜像的元数据

![image-20251017152846594](https://i-blog.csdnimg.cn/img_convert/3f548775b0e69a260cd6b986c0d816af.png)

发现镜像的元数据中的数据卷是 `/var/lib/postgresql`目录

但是网络上大多数都是建议挂载 `/var/lib/postgresql/data`目录

![Clipboard_Screenshot_1760686199](https://i-blog.csdnimg.cn/img_convert/2bd274ee2fc173e5b948e7874595834d.png)

在 docker hub 的主页中，也有写到这个变化

https://hub.docker.com/_/postgres

![Clipboard_Screenshot_1760686401](https://i-blog.csdnimg.cn/img_convert/34f34fb1497c1b1b5ad677907b8bea88.png)

> 简单一句话描述就是 postgres 17 及以下，挂载 `/var/lib/postgresql/data`目录
> 
> Postgres 17 以上的版本，挂载 `/var/lib/postgresql`目录

好吧，再次看到了希望

使用
    
    
    docker run -d \
    --name pg \
    -v /data/postgres:/var/lib/postgresql \
    -p 5432:5432 \
    -e POSTGRES_USER=root \
    -e POSTGRES_PASSWORD=aA123456 \
    postgres
    

启动

![image-20251017153707011](https://i-blog.csdnimg.cn/img_convert/4017ceb5f46dbc055f12843e80725c98.png)

完美解决

> 网络上的信息会过时呀~~