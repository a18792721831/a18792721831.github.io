---
layout: post
title: "n8n数据存储在postgres"
date: 2025-10-17 18:22:40 +0800
categories: [1024程序员节, 数据库, n8n, postgresql, ai]
description: "本文介绍了如何使用PostgreSQL作为n8n工作流自动化工具的后端数据库。首先概述了PostgreSQL作为开源关系型数据库的优势和特点，包括其扩展性和复杂数据类型支持。然后详细讲解了通过Docker部署PostgreSQL的步骤，包括目录挂载和常见问题解决。接着指导如何在n8n中配置PostgreSQL数据库连接，通过环境变量指定数据库参数。最后验证了n8n成功将工作流数据存储到PostgreSQL中，包括创建流程、凭证和执行记录等数据。该方案为n8n提供了更可靠的数据存储方案，适合生产环境使用。_n8n postgresql"
keywords: 1024程序员节, 数据库, n8n, postgresql, ai
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/153475669
> - 发布时间：2025-10-17 18:22:40
> - 阅读量：1
> - 标签：#1024程序员节, #数据库, #n8n, #postgresql, #ai

## 摘要

文章浏览阅读1.1k次，点赞27次，收藏10次。本文介绍了如何使用PostgreSQL作为n8n工作流自动化工具的后端数据库。首先概述了PostgreSQL作为开源关系型数据库的优势和特点，包括其扩展性和复杂数据类型支持。然后详细讲解了通过Docker部署PostgreSQL的步骤，包括目录挂载和常见问题解决。接着指导如何在n8n中配置PostgreSQL数据库连接，通过环境变量指定数据库参数。最后验证了n8n成功将工作流数据存储到PostgreSQL中，包括创建流程、凭证和执行记录等数据。该方案为n8n提供了更可靠的数据存储方案，适合生产环境使用。_n8n postgresql

---

n8n数据存储在postgres

## postgres 介绍

PostgreSQL（简称 Postgres）作为一款功能强大的开源关系型数据库管理系统，因其高扩展性、标准兼容性和卓越的性能，受到了全球开发者和企业的广泛青睐。Postgres 最初的设计目标是为了解决传统关系型数据库在可扩展性、复杂数据类型支持以及事务处理等方面的局限性。自1986年由加州大学伯克利分校的 POSTGRES 项目起步，经过数十年的持续发展，Postgres 已经从一个学术研究项目成长为企业级应用的首选数据库之一。

与 MySQL、Oracle、SQL Server 等主流数据库相比，Postgres 在许多方面展现出独特的优势。首先，Postgres 完全遵循 ACID 原则，支持多版本并发控制（MVCC），在高并发场景下依然能够保证数据一致性和事务隔离。其次，Postgres 拥有极强的扩展能力，用户可以自定义数据类型、函数、操作符，甚至可以通过插件机制扩展数据库的核心功能。此外，Postgres 对地理空间数据（PostGIS）、JSON、XML 等复杂数据类型的原生支持，使其在大数据、地理信息系统、金融等领域表现出色。

当然，Postgres 也有一些不足之处。例如，在高写入压力和极大规模分布式场景下，Postgres 的原生集群和分片能力相较于某些 NoSQL 数据库（如 MongoDB、Cassandra）略显不足。不过，随着社区的不断发展，相关的扩展和工具（如 Citus、Patroni 等）也在不断完善。总体而言，Postgres 以其开源、稳定、灵活和强大的特性，成为现代应用架构中不可或缺的数据库解决方案。

## 使用 docker 部署

在 docker hub 中找到 postgres

https://hub.docker.com/_/postgres

创建数据存储目录

![image-20251016113049187](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20251016113049187.png)

使用如下命令启动
    
    
    docker run -d \
    --name pg \
    -v /data/postgres:/var/lib/postgresql \
    -p 5432:5432 \
    -e POSTGRES_USER=root \
    -e POSTGRES_PASSWORD=aA123456 \
    postgres
    

> 挂载目录使用 `docker imspect postgres` 查看
> 
> ![image-20251017154054005](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20251017154054005.png)

启动报错

![image-20251016113148936](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20251016113148936.png)

应该是挂载目录的版本问题

![Clipboard_Screenshot_1760686896](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1760686896.png)

postgres 17及之前的，挂载 `/var/lib/postgresql/data`目录，之后的挂载`/var/lib/postgresql`目录

![image-20251017154245268](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20251017154245268.png)

使用 dbever 连接测试

![Clipboard_Screenshot_1760687380](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1760687380.png)

## n8n使用postgres 数据库

文档地址：https://docs.n8n.io/hosting/installation/docker/#using-with-postgresql

主要是在创建 n8n 镜像的时候，通过环境变量指定pg 数据库
    
    
    docker volume create n8n_data
    docker run -it --rm \
     --name n8n \
     -p 5678:5678 \
     -e GENERIC_TIMEZONE="<YOUR_TIMEZONE>" \
     -e TZ="<YOUR_TIMEZONE>" \
     -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
     -e N8N_RUNNERS_ENABLED=true \
     -e DB_TYPE=postgresdb \
     -e DB_POSTGRESDB_DATABASE=<POSTGRES_DATABASE> \
     -e DB_POSTGRESDB_HOST=<POSTGRES_HOST> \
     -e DB_POSTGRESDB_PORT=<POSTGRES_PORT> \
     -e DB_POSTGRESDB_USER=<POSTGRES_USER> \
     -e DB_POSTGRESDB_SCHEMA=<POSTGRES_SCHEMA> \
     -e DB_POSTGRESDB_PASSWORD=<POSTGRES_PASSWORD> \
     -v n8n_data:/home/node/.n8n \
     docker.n8n.io/n8nio/n8n
    

我们之前启动 n8n 的命令
    
    
    docker run -d \
     --name n8n \
     -p 5678:5678 \
     -e N8N_SECURE_COOKIE=false \
     -e NODE_FUNCTION_ALLOW_BUILTIN=* \
     -e NODE_FUNCTION_ALLOW_EXTERNAL=* \
     -v n8n_data:/data/n8n \
    n8nio/n8n
    

首先在 pg 数据库中创建 n8n 的 database

![image-20251017171529960](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20251017171529960.png)

选择展示所有的数据库

![Clipboard_Screenshot_1760693081](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1760693081.png)

![Clipboard_Screenshot_1760693044](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1760693044.png)

按照需要进行修改
    
    
    docker run -d \
     --name n8n \
     -p 5678:5678 \
     -e N8N_SECURE_COOKIE=false \
     -e NODE_FUNCTION_ALLOW_BUILTIN=* \
     -e NODE_FUNCTION_ALLOW_EXTERNAL=* \
     -e DB_TYPE=postgresdb \
     -e DB_POSTGRESDB_DATABASE=n8n \
     -e DB_POSTGRESDB_HOST=ip \
     -e DB_POSTGRESDB_PORT=5432 \
     -e DB_POSTGRESDB_USER=root \
     -e DB_POSTGRESDB_PASSWORD=aA123456 \
     -e N8N_DEFAULT_LOCALE=zh-CN \
     -v n8n_data:/data/n8n \
     -v /data/n8n_zh:/usr/local/lib/node_modules/n8n/node_modules/n8n-editor-ui/dist \
    n8nio/n8n
    

启动成功

![image-20251017172747210](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20251017172747210.png)

登录验证

登录后，创建一个n8n 的流程

![Clipboard_Screenshot_1760693795](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1760693795.png)

包括创建一些凭证之类的

我使用的是腾讯云的 deepseek 能力

![Clipboard_Screenshot_1760693914](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1760693914.png)

![Clipboard_Screenshot_1760694006](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1760694006.png)

尝试触发请求 ai

![Clipboard_Screenshot_1760694060](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1760694060.png)

尝试执行一下

![Clipboard_Screenshot_1760695278](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/Clipboard_Screenshot_1760695278.png)

在postgres 中验证

![image-20251017180155449](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20251017180155449.png)

数据库中已经有数据了

![image-20251017180231087](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/image-20251017180231087.png)