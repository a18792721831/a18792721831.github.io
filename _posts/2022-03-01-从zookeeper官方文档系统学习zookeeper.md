---
layout: post
math: true
title: "从zookeeper官方文档系统学习zookeeper"
date: 2022-03-01 00:55:47 +0800
categories: [zookeeper, zookeeper搭建, zk选举Leader, zk的ZAB协议, zk命令使用]
description: "本文详细介绍了Zookeeper的安装、配置、启动、验证，以及单机和集群版本的区别。重点讲解了Zookeeper的Leader选举机制，包括启动时和运行中的选举过程，并探讨了节点状态和客户端命令。此外，还涵盖了Zookeeper的配置、权限ACL、监视器、数据结构、会话、数据同步和ZAB协议。最后，提到了Zookeeper在Java中的使用和分布式锁的应用。"
keywords: zookeeper, zookeeper搭建, zk选举Leader, zk的ZAB协议, zk命令使用
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/123196188
> - 发布时间：2022-03-01 00:55:47
> - 阅读量：1
> - 分类：大数据同时被 3 个专栏收录, 订阅专栏, hadoop, 技术分享
> - 标签：#zookeeper, #zookeeper搭建, #zk选举Leader, #zk的ZAB协议, #zk命令使用

## 摘要

文章浏览阅读1.4k次，点赞2次，收藏2次。本文详细介绍了Zookeeper的安装、配置、启动、验证，以及单机和集群版本的区别。重点讲解了Zookeeper的Leader选举机制，包括启动时和运行中的选举过程，并探讨了节点状态和客户端命令。此外，还涵盖了Zookeeper的配置、权限ACL、监视器、数据结构、会话、数据同步和ZAB协议。最后，提到了Zookeeper在Java中的使用和分布式锁的应用。

---

#### 从zookeeper官方文档系统学习zookeeper

  * 1\. zookeeper
  * 2\. zookeeper 文档
  * 3\. zookeeper 单机版
  *     * 3.1 配置
    * 3.2 启动
    * 3.3 验证
  * 4\. zookeeper 集群版
  *     * 4.1 配置
    * 4.2 启动
    * 4.3 验证
  * 5\. zookeeper 配置
  *     * 5.1 最小配置
    * 5.2 其他配置
  * 6\. zookeeper Leader 选举
  *     * 6.1 启动时的Leader选举
    * 6.2 运行中的leader选举
    * 6.3 zookeeper 节点状态
  * 7\. zookeeper 客户端命令
  *     * 7.0 文档
    * 7.1 zookeeper 客户端连接
    * 7.2 create
    * 7.3 ls
    * 7.4 get
    * 7.5 set
    * 7.6 delete
  * 8\. zookeeper 权限ACL
  *     * 8.1 ACL文档
    * 8.2 ACL命令
    * 8.3 ACL组成
    *       * 8.3.1 permissions
      * 8.3.2 schema
  * 9\. zookeeper 监视器
  *     * 9.1 zookeeper 监视器 文档
    * 9.2 zookeeper 客户端命令使用监视器
    * 9.3 zookeeper 监视器特点
    * 9.4 zookeeper 监视器事件类型
    * 9.5 zookeeper 客户端命令使用监视器
    * 9.6 zookeeper 程序使用监视器
  * 10\. zookeeper 数据结构
  *     * 10.1 zookeeper 存储数据
    * 10.2 zookeeper节点
  * 11\. zookeeper session
  * 12\. zookeeper 数据同步
  *     * 12.1 ZAB协议
    * 12.2 ZAB协议原理
    * 12.3 ZAB VS 流言
  * 13\. zookeeper 监控
  * 14\. zookeeper 集成 java
  *     * 14.1 zookeeper原生API
    * 14.2 zookeeper的Curator的API
  * 15\. zookeeper 分布式锁
  * 16\. 总结

## 1\. zookeeper

ZooKeeper 是一个集中式服务，用于维护配置信息、命名、提供分布式同步和提供组服务。

> ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services.  
>  [zookeeper官网](<http://zookeeper.apache.org/>)  
>  下载zookeeper  
>  ![image-20220228221238058](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c475b75aa6b4151ea59cf2e23e06330d.png)
> 
> ![image-20220228221339680](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bfc49401b15452cfd97af1604cd5474c.png)
> 
> zookeeper解压的文件目录：
> 
> ![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9486b42c94071c9f296b2f18b205212f.png)

## 2\. zookeeper 文档

zookeeper文档在下载目录的docs目录下。  
![image-20220228221851741](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cf1dc12b45dac007376a2b0417dea83d.png)

下载的二进制包里面的文档，一定是最准确的文档。  
有时候，比官网文档还准确。  
看文档，一般主要是这几个步骤。

  1. welcome或index界面：这里明确的定义了组件的组成、功能以及作用等信息。
  2. Getting Started：这里用一个最简单的例子，教我们入门组件。
  3. config：查询组件全部的配置，以及配置对应的含义。
  4. 集成：这一块主要是组件如何与其他系统进行集成。

一般的文档，都会有这四个部分的。

## 3\. zookeeper 单机版

### 3.1 配置

单机版的zookeeper需要在**conf/zoo.cfg** 配置三个参数即可。  
默认是没有**conf/zoo.cfg** 文件的，需要拷贝**zoo_sample.cfg** 文件。
    
    
    tickTime=2000
    dataDir=/var/lib/zookeeper
    clientPort=2181
    

tickTime默认就是2000，我们不需要做任何修改。  
因为本地启动，我们需要尽可能节省资源，因此将初始化的连接数，调整为2
    
    
    initLimit=2
    syncLimit=2
    

然后修改zookeeper数据存储路径。  
我们在zookeeper的主目录下创建`data`目录和`logs`目录  
![image-20220228222018966](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3453ba3f235a1cc9288a8862387b7245.png)

然后在**conf/zoo.cfg** 中配置`data`目录
    
    
    dataDir=/D:/Users/JIAYONGQI784/zookeeper/apache-zookeeper-3.7.0-bin/data
    

在`log4j.properties`中配置日志目录
    
    
    zookeeper.log.dir=/D:/Users/JIAYONGQI784/zookeeper/apache-zookeeper-3.7.0-bin/logs
    

OK，到此就配置完成了。

![image-20220228222125300](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d6ff6638df319490ffecb89267876cc9.png)

我们顺手把`ZOOKEEPER_HOME`配置了.  
![image-20220228222202212](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4873fe7012efa543395c005ad87ad3bd.png)  
别配置的太深入了,配置的目的是我们可以以非常快速的操作，切换到zookeeper相关的目录即可。

### 3.2 启动

执行命令
    
    
    cd %ZOOKEEPER_HOME%
    cd bin
    zkServer.cmd
    

当日志没有异常时，就启动成功了  
![image-20220228222251761](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/641fec9dbe6d268411418f1ff1cd41f6.png)

### 3.3 验证

zookeeper单机版启动成功了，如何验证呢?  
在zookeeper的文档中，给出了答案  
![image-20220228222332803](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cf6b404ee4407a1b6e41d41a2d2710f4.png)

使用客户端连接zookeeper服务。  
重新启动一个cmd，切换到zookeeper的bin目录下。
    
    
    cd %ZOOKEEPER_HOME%
    cd bin
    zkCli.cmd -server 127.0.0.1:2181
    

启动客户端，链接启动的zookeeper服务。  
![image-20220228222428431](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b11f3ae96299e8cfe938e357f0f5cfde.png)

zookeeper客户端的命令
    
    
    ZooKeeper -server host:port -client-configuration properties-file cmd args
    addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE
    addauth scheme auth
    close
    config [-c] [-w] [-s]
    connect host:port
    create [-s] [-e] [-c] [-t ttl] path [data] [acl]
    delete [-v version] path
    deleteall path [-b batch size]
    delquota [-n|-b|-N|-B] path
    get [-s] [-w] path
    getAcl [-s] path
    getAllChildrenNumber path
    getEphemerals path
    history
    listquota path
    ls [-s] [-w] [-R] path
    printwatches on|off
    quit
    reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
    redo cmdno
    removewatches path [-c|-d|-a] [-l]
    set [-s] [-v version] path data
    setAcl [-s] [-v version] [-R] path acl
    setquota -n|-b|-N|-B val path
    stat [-w] path
    sync path
    version
    whoami
    Command not found: Command not found help
    

我们尝试在zookeeper中创建一个`hello-world`目录，然后创建`hi`的文件，设置值`hi xiaomei!`
    
    
    create /hello-world
    create /hello-world/hi
    set /hello-world/hi "hi xiaomei\!"
    get /hello-world/hi
    

记得转义特殊符号。

![image-20220228222609293](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2ed68fd78a8a9913d049ba1dc5c2266f.png)

然后删除我们新增的内容
    
    
    delete /hello-world/hi
    delete /hello-world
    ls /
    

![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5f9801acb66148bb09ee8084ed2915f9.png)

至此，我们就搭建了一个单机版的zookeeper.

## 4\. zookeeper 集群版

> Running ZooKeeper in standalone mode is convenient for evaluation, some development, and testing. But in production, you should run ZooKeeper in replicated mode.  
>  单机版只适合与开发或者学习，或者测试，并不能在生产环境使用。

### 4.1 配置

在文档中也有如何搭建集群版的zookeeper。

![image](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f7bda83f35f1f4071c299ebedc4c7712.png)

首先，我们将zookeeper文件拷贝三份出来，并分别修改文件夹名字，加上端口号：

![image-20220228223254238](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a6641e963fe6b4e997007ac72cb6ba4b.png)

然后修改**conf/zoo.cfg** 文件，增加

![image-20220228223243525](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/13caa1a4982e61aaf4756cc2a32d6797.png)每个文件都是相同的
    
    
    server.1=0.0.0.0:2888:3888
    server.2=0.0.0.0:2889:3889
    server.3=0.0.0.0:2890:3890
    

> If you want to test multiple servers on a single machine, specify the servername as _localhost_ with unique quorum & leader election ports (i.e. 2888:3888, 2889:3889, 2890:3890 in the example above) for each server.X in that server’s config file.

增加端口,注意，每个节点的具体值不同
    
    
    clientPort=2181
    

增加日志配置，注意修改路径
    
    
    dataLogDir=/E:/zookeeper/zookeeper-2181/logs
    

因为我们有三个节点，因此需要将初始化的连接数修改为3，这样不用等待。
    
    
    initLimit=3
    syncLimit=3
    

现在出现了一个问题，节点如何知道自己是谁？

> The entries of the form _server.X_ list the servers that make up the ZooKeeper service. When the server starts up, it knows which server it is by looking for the file _myid_ in the data directory. That file has the contains the server number, in ASCII.

![image-20220228224150681](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/755c9daa12902d61dbdb4dd6b9678ed9.png)

在data目录下创建一个文件，文件名就是`myid`，这个文件里面的数字会告诉节点自己是哪个服务。

![image-20220228224236055](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/77a8c9ea077dc4d9e437a2cc2839f7e8.png)

千万不要有多余的空格等字符

这里数字是结合前面配置的节点列表来配置的，从1开始。(from _server.X_)

至此，就配置完了。

### 4.2 启动

切换到对应节点的bin目录下，启动服务即可。(启动前，先将data和logs目录清空，需要重新创建`myid`文件)
    
    
    cd %ZOOKEEPER_HOME%
    cd apache-zookeeper-3.7.0-bin-2181
    cd bin
    zkServer.cmd
    

依次启动三个节点即可，顺序没有关系。

第一个节点启动后，控制台会打印异常：

![image-20220228224423809](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/49a2e11df92fd1d8646f5048c3596006.png)

![image-20220228224432920](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fcf5ce5bd6ba39252c90d933955efdaa.png)

给其他节点发送消息的时候，连接失败(肯定了啦，还没启动呢)。

启动其他两个节点。

### 4.3 验证

还是用`zkCli.cmd`进行验证：

分别用三个客户端连接三个服务器：  
![image-20220228224832816](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/704afbf2385966fb892c33e42a796176.png)

然后用一个客户端连接集群：

![image-20220228224933472](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4f94baf5fe89204711bb17b391d6d5f2.png)

现在我有4个客户端。

我在第一个客户端上增加目录

![image-20220228225000337](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/72cc307d9a5d9fafcb2dfe0d328e76cb.png)

在第二个客户端上增加文件，并设置值

![image-20220228225111722](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8d7774ddbe22a52a8ae733b3927744de.png)

第三个客户端上查看文件的值(集群)

![image-20220228225135242](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5242f952b7f122889e8f1ba92dbb72dd.png)

在第四个客户端上删除增加的目录和值

![image-20220228225205856](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/816bd899227ecde26f8392813d446305.png)

证明了三个节点之间是联通的。同时也证明连接集群和连接集群内节点是等价的。

## 5\. zookeeper 配置

在Admin&… 目录下有配置参数说明：

![image-20220228225258745](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f47503c78252983644bb87b8e89f26f1.png)

可以看到，虽然zookeeper是3.7.0的版本，但是文档上面还是3.6.  
Zookeeper 3.6 Documenttation，看到了吗。  
或许可以向zookeeper发起pr。  
这个小节就是全部的zookeeper的配置  
![image-20220228225327995](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2b050b0b5e6c8c03927f3c056240bd82.png)

### 5.1 最小配置

  * clientPort: 端口
  * secureClientPort：https端口
  * observerMasterPort：master的端口
  * dataDir：数据存储目录
  * tickTime：时间单位(会话连接超时时间)  
在单机版中，使用的配置远远少于此。  
不过这个最小配置是对于zookeeper从节点来说的。  
在zookeeper中，有不同的角色。

### 5.2 其他配置

配置非常多，但是配置这一块，作为一个字典一样，遇到在这里查就可以了。

![image-20220228225417724](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e4eca0865cd84c003956addaa49431ed.png)

之前配置日志目录是在`log4j.properties`文件中配置的，这样就会导致配置有点分散。  
好在`zoo.cfg`文件中也能够配置日志路径。

![image-20220228225500890](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8babd71d18446738b6866b9f48c8c2d4.png)

## 6\. zookeeper Leader 选举

zookeeper 的选举非常有意思。

在看日志的时候，有一个小技巧：  
如果myid与n.sid相同，表示当前节点对外发出通知；  
如果myid与n.sid不同，表示当前节点收到其他节点的通知。

我截取了之前搭建的zookeeper集群的部分日志：
    
    
    server1: LOOKING -> FOLLOWING
    
    2021-05-28 16:20:59,966 [myid:1] - INFO [QuorumPeer[myid=1](plain=[0:0:0:0:0:0:0:0]:2181)(secure=disabled):QuorumPeer@1430] - LOOKING
    2021-05-28 16:21:16,464 [myid:1] - INFO [QuorumPeer[myid=1](plain=[0:0:0:0:0:0:0:0]:2181)(secure=disabled):QuorumPeer@1512] - FOLLOWING
    
    2021-05-28 16:20:59,976 [myid:1] - INFO [WorkerReceiver[myid=1]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:1, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    2021-05-28 16:21:16,259 [myid:1] - INFO [WorkerReceiver[myid=1]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:2, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:16,261 [myid:1] - INFO [WorkerReceiver[myid=1]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    2021-05-28 16:21:31,442 [myid:1] - INFO [WorkerReceiver[myid=1]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:FOLLOWING; n.sid:3, n.state:LOOKING, n.leader:3, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    
    
    server2: FOLLOWING -> LEADING
    
    2021-05-28 16:21:16,239 [myid:2] - INFO [QuorumPeer[myid=2](plain=[0:0:0:0:0:0:0:0]:2182)(secure=disabled):QuorumPeer@1430] - LOOKING
    2021-05-28 16:21:16,466 [myid:2] - INFO [QuorumPeer[myid=2](plain=[0:0:0:0:0:0:0:0]:2182)(secure=disabled):QuorumPeer@1524] - LEADING
    
    2021-05-28 16:21:16,251 [myid:2] - INFO [WorkerReceiver[myid=2]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:2, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:16,259 [myid:2] - INFO [WorkerReceiver[myid=2]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:1, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:16,262 [myid:2] - INFO [WorkerReceiver[myid=2]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    2021-05-28 16:21:31,442 [myid:2] - INFO [WorkerReceiver[myid=2]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LEADING; n.sid:3, n.state:LOOKING, n.leader:3, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    
    
    server3: LOOKING -> FOLLOWING
    
    2021-05-28 16:21:31,422 [myid:3] - INFO [QuorumPeer[myid=3](plain=[0:0:0:0:0:0:0:0]:2183)(secure=disabled):QuorumPeer@1430] - LOOKING
    2021-05-28 16:21:31,448 [myid:3] - INFO [QuorumPeer[myid=3](plain=[0:0:0:0:0:0:0:0]:2183)(secure=disabled):QuorumPeer@1512] - FOLLOWING
    
    2021-05-28 16:21:31,435 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:3, n.state:LOOKING, n.leader:3, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:31,442 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:31,443 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:2, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:31,446 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:2, n.state:LEADING, n.leader:2, n.round:0x1, n.peerEpoch:0xc, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:31,446 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:FOLLOWING, n.leader:2, n.round:0x1, n.peerEpoch:0xc, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    

如果直接看日志，是会觉得有点懵的。  
但是你如果按照时间线进行梳理，在结合启动节点的顺序，就能较为清晰的通过日志分析zookeeper的 leader选举过程。
    
    
    server1: LOOKING -> FOLLOWING
    
    2021-05-28 16:20:59,966 [myid:1] - INFO [QuorumPeer[myid=1](plain=[0:0:0:0:0:0:0:0]:2181)(secure=disabled):QuorumPeer@1430] - LOOKING
    2021-05-28 16:21:16,464 [myid:1] - INFO [QuorumPeer[myid=1](plain=[0:0:0:0:0:0:0:0]:2181)(secure=disabled):QuorumPeer@1512] - FOLLOWING
    
    2021-05-28 16:20:59,976 [myid:1] - INFO [WorkerReceiver[myid=1]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:1, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    2021-05-28 16:21:16,259 [myid:1] - INFO [WorkerReceiver[myid=1]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:2, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:16,261 [myid:1] - INFO [WorkerReceiver[myid=1]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    2021-05-28 16:21:31,442 [myid:1] - INFO [WorkerReceiver[myid=1]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:FOLLOWING; n.sid:3, n.state:LOOKING, n.leader:3, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    
    
    server2: FOLLOWING -> LEADING
    
    2021-05-28 16:21:16,239 [myid:2] - INFO [QuorumPeer[myid=2](plain=[0:0:0:0:0:0:0:0]:2182)(secure=disabled):QuorumPeer@1430] - LOOKING
    2021-05-28 16:21:16,466 [myid:2] - INFO [QuorumPeer[myid=2](plain=[0:0:0:0:0:0:0:0]:2182)(secure=disabled):QuorumPeer@1524] - LEADING
    
    2021-05-28 16:21:16,251 [myid:2] - INFO [WorkerReceiver[myid=2]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:2, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:16,259 [myid:2] - INFO [WorkerReceiver[myid=2]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:1, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:16,262 [myid:2] - INFO [WorkerReceiver[myid=2]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    2021-05-28 16:21:31,442 [myid:2] - INFO [WorkerReceiver[myid=2]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LEADING; n.sid:3, n.state:LOOKING, n.leader:3, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    
    
    server3: LOOKING -> FOLLOWING
    
    2021-05-28 16:21:31,422 [myid:3] - INFO [QuorumPeer[myid=3](plain=[0:0:0:0:0:0:0:0]:2183)(secure=disabled):QuorumPeer@1430] - LOOKING
    2021-05-28 16:21:31,448 [myid:3] - INFO [QuorumPeer[myid=3](plain=[0:0:0:0:0:0:0:0]:2183)(secure=disabled):QuorumPeer@1512] - FOLLOWING
    
    2021-05-28 16:21:31,435 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:3, n.state:LOOKING, n.leader:3, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:31,442 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:31,443 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:2, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0xb, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:31,446 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:2, n.state:LEADING, n.leader:2, n.round:0x1, n.peerEpoch:0xc, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    2021-05-28 16:21:31,446 [myid:3] - INFO [WorkerReceiver[myid=3]:FastLeaderElection$Messenger$WorkerReceiver@390] - Notification: my state:LOOKING; n.sid:1, n.state:FOLLOWING, n.leader:2, n.round:0x1, n.peerEpoch:0xc, n.zxid:0xb000001c4, message format version:0x2, n.config version:0x0
    
    
    20:59.966 -> server1 # LOOKING
    20:59.976 -> L1: sid = 1, leader = 1, peerEpoch = 0xb, zxid = 0xb000001c4 => server2 : timeout => server3 : timeout
    
    21:16.239 -> server1 # LOOKING, server2 # LOOKING
    21:16.251 -> L2: sid = 2, leader = 2, peerEpoch = 0xb, zxid = 0xb000001c4 => server1 : success => server3 : timeout
    21:16.259 -> L2: server2 => server1 @ sid = 2, leader = 2, peerEpoch = 0xb, zxid = 0xb000001c4 $$ sid : 2 > 1 ==> server1 accept ----> sid = 1, leader = 2, peerEpoch = 0xb, zxid = 0xb000001c4
    21:16.261 -> L2: server1 => server2 @ sid = 1, leader = 2, peerEpoch = 0xb, zxid = 0xb000001c4 $$ server1 accept server2 is leader.
    21:16:262 -> L2: sid = 1, leader = 2, peerEpoch = 0xb, zxid = 0xb000001c4 => server1 LOOKING
    21:16,464 -> L2: server1 --> LOOKING -> FOLLOWING
    21:16,466 -> L2: server2 --> LOOKING -> LEADING
    21:16,466 -> server1 # LOOKING, server2 # LEADING
    
    21:31,422 -> server3 # LOOKING
    21:31,435 -> L3: sid = 3, leader = 3, peerEpoch = 0xb, zxid = 0xb000001c4 => server1 : success => server2 : success
    21:31,442 -> L3: server1 => server3 @ sid = 3, leader = 3, peerEpoch = 0xb, zxid = 0xb000001c4 $$ FOLLOWING
    21:31,442 -> L3: server2 => server3 @ sid = 3, leader = 3, peerEpoch = 0xb, zxid = 0xb000001c4 $$ LEADING
    21:31,442 -> L3: server3 => server1 @ sid = 1, leader = 2, peerEpoch = 0xb, zxid = 0xb000001c4 $$ leader = 2
    21:31,443 -> L3: server3 => server2 @ sid = 2, leader = 2, peerEpoch = 0xb, zxid = 0xb000001c4 $$ leader = 2
    21:31,446 -> L3: server3 => server2 @ sid = 2, leader = 2, peerEpoch = 0xb, zxid = 0xb000001c4 $$ server2 -> LEADING # peerEpoch = 0xc
    21:31,446 -> L3: server3 => server1 @ sid = 1, leader = 2, peerEpoch = 0xb, zxid = 0xb000001c4 $$ server1 -> FOLLOWING # peerEpoch = 0xc
    21:31,448 -> server3 # FOLLOWING, server2 # LEADING, server1 # FOLLOWING
    
    

首先，集群中的zookeeper节点有4种状态：

  1. LOOKING: 寻找Leader状态，处于该状态需要进入选举流程
  2. LEADING:领导者状态，处于该状态的节点说明已经是Leader了
  3. FOLLOWING:跟随者状态，表示Leader已经选举出来了，当前节点跟随Leader
  4. OBSERVER:观察者状态，观察者状态的节点不参与Leader选举

### 6.1 启动时的Leader选举

分析三个节点的zookeeper启动时的Leader选举：

server1启动，此时server1的状态是LOOKING状态，然后server1进入选举流程。  
然后server1启动Leader选举流程

![image-20220228225639406](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/206f781f48da10f2aec8f3856eed33fc.png)

首先server1选举自己作为Leader，同时把自己的状态LOOKING通知给已知节点(配置文件中配置的节点)，并通知其他节点自己的选举事务id(使用选举讨论次数可能会好理解点)
    
    
    Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:1, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

因为Leader选举需要其他节点投票认可才能成为Leader，此时只有server1节点，是无法获得投票的，所以server1一直处于LOOKING状态，也一直无法选举成功。

接着server2启动了。  
server2启动后，状态是LOOKING状态，将进入选举流程。  
server2启动选举

![image-20220228225808145](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5cd34f66e05305e35d1abb3aa6811c3b.png)
    
    
    Notification: my state:LOOKING; n.sid:2, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

server2选举自己作为Leader，同时把自己的状态LOOKING通知给已知节点。

需要注意的一点是，在server2进行选举的时候，server1也在进行选举。  
server2启动后，将自己的选举信息通知给server1，期望获取server1的投票。

![image-20220228225856616](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ac85213c85cb3df38045e953c5faa1f3.png)
    
    
    Notification: my state:LOOKING; n.sid:2, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

server1收到server2的通知后，发现server2除了`n.leader`与server1不同外，没有其他的区别。  
此时server1和server2竞争leader。  
但是因为server1的myid小于server2的myid。也就是`n.sid`.所以，server1只能同意server2的选举。  
于是server1通知其他节点自己的投票

![image-20220228230242370](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c965ecf28cf29301ca86f5233210b5ef.png)
    
    
    Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

server1的投票中，`n.leader:2`表示server1选择server2作为leader。

当然，在leader选举出来之前，server1和server2是平等的，server1也会向server2发送通知，期望获取server2的投票

![image-20220228230204562](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d3b2dde439b034094aebeccd410792b3.png)
    
    
    Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:1, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

server2在收到server1要求投票的通知后，发现server1和server2的除了`n.leader`不同，其他的都相同。  
但是因为server2的sid大于server1的sid，因此，server2不会将票投给server1，也就不会发送投票结果了。

server2收到server1的投票后，会进行统计票数(实际上每一个节点都会统计)

![image-20220228230353041](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a6208a2d480a7bb4d4d3798139104aad.png)
    
    
    Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

此时server2已经有2个节点支持作为leader了。

在等待了一个会话时间段后(也就是配置文件中配置的`tickTime`)，没有收到其他节点的通知，表示当前集群中的节点都已投票。

然后集群内的全部节点进行切换。  
leader节点切换为LEADING状态，其他节点切换为FOLLOWING状态(OBSERVER节点除外)

![image-20220228230445739](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/063deabf40e34914ce11e5cc2448b46c.png)

![image-20220228230510289](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2251abfdbc67d4c1b292617233eb89a6.png)

当server3节点启动后，server3是LOOKING状态，进入leader选举流程。  
server3选举自己作为leader，将信息通知给已知节点，期望获取其他节点的投票。

![image-20220228230550934](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a91984219985e98a7519c696aec2eb80.png)
    
    
    Notification: my state:LOOKING; n.sid:3, n.state:LOOKING, n.leader:3, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

server1和server2在收到通知后，将现在的leader信息和自己的状态信息发送给server3  
server1收到的通知:

![image-20220228230647124](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3fc9a730c4ea85c96ac174861fe0a917.png)
    
    
    Notification: my state:FOLLOWING; n.sid:3, n.state:LOOKING, n.leader:3, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

收到通知后，因为现在的leader是2，所以server1将自己的投票结果通知给server3.

![image-20220228230913274](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/104573444c9bf47f5afee1494dd528a5.png)
    
    
    Notification: my state:LOOKING; n.sid:1, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

同样的server2也会收到server3要求投票的通知：
    
    
    Notification: my state:LEADING; n.sid:3, n.state:LOOKING, n.leader:3, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

server2也会将自己的投票结果发送给server3
    
    
    Notification: my state:LOOKING; n.sid:2, n.state:LOOKING, n.leader:2, n.round:0x1, n.peerEpoch:0x0, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

此时投票完成。

接着server3收到了server1和server2的状态同步通知

![image-20220228230823691](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e47da47329b27ff7907efb2e5583d1f4.png)
    
    
    Notification: my state:LOOKING; n.sid:1, n.state:FOLLOWING, n.leader:2, n.round:0x1, n.peerEpoch:0x1, n.zxid:0x0, message format version:0x2, n.config version:0x0
    Notification: my state:LOOKING; n.sid:2, n.state:LEADING, n.leader:2, n.round:0x1, n.peerEpoch:0x1, n.zxid:0x0, message format version:0x2, n.config version:0x0
    

每个节点会知道集群内每一个节点的状态。

server3修改自己的状态，跟随现有leader。  
因为目前集群中只有3个节点，server2作为leader得到了2个节点的投票，满足了半数原则(超过一半同意)。

![image-20220228231009528](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7fd8166ccd261cb195bcf78656f0af2e.png)

此时，整个集群已经完全OK了。

不知道你是否有这样的疑问，为什么每一次投票或者选举后，都需要发送通知呢？  
因为在实际的集群中，不仅仅有leader节点，还有Observer节点。  
做的例子中，每一个节点都可以参与选举，实际使用中，可能部分节点才能参与选举，其他节点是不会参与选举的。不参与选举，但是可以参与投票。  
所以就需要通知选举信息给这些节点，这些不参与选举的节点就会进行投票统计，然后根据投票结果进行跟随。

### 6.2 运行中的leader选举

在6.1中的zookeeper集群中，如果leader下线，此时会触发Leader选举，然后根据myid确定真正的leader。  
所以，如果让server2下线，重新选举后，leader应该是server3.  
试试：

![image-20220228231145333](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0dfdf234be0a57a3414d394b28d97a85.png)

![image-20220228231221387](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5e04b2d14cab0b9656ed99a2021100d2.png)

成功证实了的猜测。

### 6.3 zookeeper 节点状态

  1. LOOKING

处于LOOKING状态的节点将进行Leader选举流程。

LOOKING状态是zookeeper节点启动时处于的状态。

LOOKING状态是一个临时性的zookeeper节点状态。不是稳定状态。

LOOKING状态可以转为LEADING和FOLLOWING，FOLLOWING也可以转为LOOKING。

同样的，LEADING也可以转为LOOKING。(小集群加入大集群)

  2. LEADING

LEADING状态的节点在zookeeper集群中是唯一的。

一个zookeeper集群中，只会允许有一个LEADING状态的节点。(保证集群内的节点网络互通)

LEADING状态的节点会协调并管理(数据层面)FOLLOWING节点

  3. FOLLOWING

FOLLOWING状态的节点在zookeeper集群中占大多数，用于处理客户端的请求。

FOLLOWING状态的节点接受LEADING节点的调度，执行一些事务方面的操作。

  4. OBSERVER

OBSERVER节点是观察者节点，观察的对象是LEADING。

主要是用来减少LEADING的压力，因为OBSERVER与LEADING的数据完全相同，因此一些节点的信息同步，或者数据读取就会分配给OBSERVER，通过这种方式减轻LEADING的压力

## 7\. zookeeper 客户端命令

### 7.0 文档

官方文档是学习zookeeper的字典。

![image-20220228231313812](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/28ae97d6ae3921e3ad9ae56cbc739499.png)

### 7.1 zookeeper 客户端连接

连接zookeeper服务

常用格式：

`zkCli.cmd -server host:port`

![image-20220228231335028](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a0d94e361de9d24551de7ef6053bd438.png)

### 7.2 create

创建节点。在zookeeper中，可以将节点等价于文件系统中的文件夹或文件。

命令格式：

`create [-s] [-e] [-c] [-t ttl] path [data] [acl]`

  * -s : 创建顺序节点
  * -e : 创建临时节点
  * -c :创建存储于指定服务器的节点
  * -t :创建超时节点，需要`set zookeeper.extendedTypesEnabled=true`
  * path :节点路径
  * data :节点数据
  * acl :节点权限

注意点：

  1. 创建节点需要一层一层创建
  2. 临时节点不能有子节点
  3. 创建超时节点需要开启配置`zookeeper.extendedTypesEnabled=true`
  4. 创建顺序节点，创建的节点会自动增加排序值
  5. 创建节点时赋值，需要注意转义
  6. 创建节点有默认的访问权限

### 7.3 ls

查看节点结构。功能与linux的ls相同。

命令格式

`ls [-s] [-w] [-R] path`

  * -s: 显示详细信息
  * -R:显示子节点
  * -w:设置子节点监视器

### 7.4 get

获取节点的值。功能与cat类似。

命令格式

`get [-s] [-w] path`

  * -s:显示详细信息
  * -w:显示监视器

### 7.5 set

设置节点的值。

命令格式

`set [-s] [-v version] path data`

  * -s :显示详细信息
  * -v:设置版本号(cas：comple and set)
  * path:节点路径
  * data:节点值

### 7.6 delete

删除节点

命令格式

`delete [-v version] path`

  * -v:版本号(cas)
  * path:节点路径

命令格式

`deleteall path [-b batch size]`

  * path:节点路径
  * -b:节点数量

`deleteall`可以删除含有子节点的节点

`delete`必须删除子节点

## 8\. zookeeper 权限ACL

### 8.1 ACL文档

zookeeper 权限的文档

![image-20220228231418706](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/73143979318337dfc6f0a2fa10cc0a78.png)

### 8.2 ACL命令

getAcl：获取某个节点的权限信息

setAcl：设置某个节点的权限信息

create：创建节点的时候，也可以同时设置权限信息

addAuth：认证授权信息（配合权限使用，类似登录）

### 8.3 ACL组成

`[schema:id:permissions]`

  * schema: 使用哪种权限机制(可以理解为模式)
  * id: 权限认证的id
  * permissions: 权限(类似linux中的读写执行权限)

#### 8.3.1 permissions

  1. create:创建子节点，缩写c
  2. read:读取节点和子节点的值，缩写r
  3. write:给节点赋值，缩写w
  4. delete:删除节点或子节点，缩写d
  5. admin:给节点设置权限，缩写a

#### 8.3.2 schema

  1. world

默认权限，world只有一个id可用，anyone。

默认任何人拥有全部的权限

![image-20220228231524443](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2831f7eb5403c6303b60d709b7ce23f7.png)

  2. auth

授权节点的指定权限给指定用户。

先需要创建用户

![image-20220228231606641](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/886304d7aca8845c59c54b17fecf45d8.png)

然后进行授权

![image-20220228231717528](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ef8647c6f72031075494753b0fb6a3a0.png)

此时设置值

![image-20220228231742130](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f5a21c07e49974be07c3faf82e18ce93.png)

在启动一个客户端访问

![image-20220228231829718](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2fca5a09f56adb20fe691f6939ec1c2f.png)

登录，即可访问

![image-20220228231904543](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ec68a269e4153bbb1935921b48ffd06c.png)

auth 仅仅授权给当前认证的用户，也就是说，在未认证用户前，使用auth将会失败：

![image-20220228232243335](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/10120286444479e8c6ef8176f9e847df.png)

digest 可以授权给任何用户

  3. digest

与auth非常类似，区别在于，digest可以授权给任何用户。

授权的时候，使用 `digest:user:MD5(password):crdwa`

MD5(123456) = 6DY5WhzOfGsWQ1XFuIyzxkpwdPo=

![image-20220228232517249](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/16c1f55690fa8ee06852dc306a053552.png)

然后用其他客户端登录并访问

![image-20220228232609200](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2e5519dfad380899f5c03a4e44657322.png)

  4. ip

限制客户端的ip，只有客户端使用指定的ip才能访问指定的操作

  5. x509

没用过，不知道干啥的。

## 9\. zookeeper 监视器

### 9.1 zookeeper 监视器 文档

![image-20220228232633980](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/96ac5194275309b755457e78f3e9a1f7.png)

### 9.2 zookeeper 客户端命令使用监视器

![image-20220228232724649](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4deff12b9d1f3235d10fcb3a3f95b607.png)

`addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE`

查看cli的客户端命令，找到和监视器有关的命令：  
`addWatch [-m mode] path`

`printwatches on|off`

### 9.3 zookeeper 监视器特点

  * 一次触发： 当数据发生变化，达到监视器的条件后，只会触发一次，并不会重复发送触发消息。除非数据发生改变，且再一次达到监视器的条件。
  * 发送给客户端：当监视器被触发后，zookeeper服务端会发送触发事件给客户端。这些发送是异步的。但是服务端会保证每一个客户端接收的事件的顺序是一致的，时间可能不同(和客户端的网络等因素有关)
  * 轻量级：事件通知只是会告诉客户端发生了事件，但是变更的内容并不会提供。

因为一次触发的特性，导致了：如果客户端短暂的和服务端失去了链接，那么，在失去链接的这段时间内触发的监视器，在过了重试次数和链接时间后，客户端将丢失这期间的事件触发。

### 9.4 zookeeper 监视器事件类型

  1. NodeCreated : 节点创建
  2. NodeDataChanged : 节点数据发生变更
  3. NodeChildrentChanged : 子节点下发生变更
  4. NodeDeleted : 节点删除

### 9.5 zookeeper 客户端命令使用监视器

记得开启打印watches

![image-20220301000511865](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b150b7bdb6868ab793e8e1d0cb122edd.png)

![image-20220301000737861](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/22bb7745f8bf4019b5336f46ea19384b.png)

![image-20220301000902364](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0fbaa08580bca91d38eb6cfac9b73d6b.png)

### 9.6 zookeeper 程序使用监视器
    
    
    import org.apache.curator.utils.DefaultZookeeperFactory;
    import org.apache.zookeeper.AddWatchMode;
    import org.apache.zookeeper.Watcher;
    import org.apache.zookeeper.ZooKeeper;
    
    /**
     * @author jiayq
     * @Date 2022-03-01
     */
    public class Watch1 {
    
        public static void main(String[] args) throws Exception {
            Watcher watcher = watchedEvent -> {
                System.out.println("watched event : " + watchedEvent.getType().name());
                switch (watchedEvent.getType()) {
                    case NodeCreated:
                        System.out.println(" node create : " + watchedEvent.getPath());
                        break;
                    case NodeDataChanged:
                        System.out.println(" data changed : " + watchedEvent.getPath());
                        break;
                    case DataWatchRemoved:
                        System.out.println(" data remove : " + watchedEvent.getPath());
                        break;
                    case NodeDeleted:
                        System.out.println(" node remove : " + watchedEvent.getPath());
                        break;
                    case NodeChildrenChanged:
                        System.out.println(" node children changed : " + watchedEvent.getPath());
                        break;
                    case ChildWatchRemoved:
                        System.out.println(" node children remove : " + watchedEvent.getPath());
                        break;
                    default:
                        System.out.println(watchedEvent.getType().name() + " : " + watchedEvent.getPath());
                        break;
                }
            };
            ZooKeeper zooKeeper = new DefaultZookeeperFactory()
                    .newZooKeeper("127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183", 6000, sx -> {
                    }, false);
            zooKeeper.addWatch("/", watcher, AddWatchMode.PERSISTENT_RECURSIVE);
            System.in.read();
        }
    
    }
    
    

![image-20220301002251044](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/481e030649fb4b53689dc4d564d3dab5.png)

## 10\. zookeeper 数据结构

### 10.1 zookeeper 存储数据

在zookeeper 中，的数据都是存储在一个个的路径下面的，而这些路径就类似于linux的文件路径一样。

在linux中是文件夹和文件，而在zookeeper中，则被统一称为节点。

节点可以存储数据。但是，一个节点只能存储一个数据。如果给同一个节点赋值多次，那么已最后一次为准。

节点下面可以有下级节点。所有的节点都是从`/`节点生长出来的。

使用`ls,get`可以查看一个节点的详细信息

![image-20220301002342774](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2bf66a25a364697bb91e3359596b875b.png)

### 10.2 zookeeper节点

在zookeeper中，每一个节点都可以由这些属性描述

| 属性名字           | 属性释义                                           |
|----------------|------------------------------------------------|
| cZxid          | 创建节点的事务id                                      |
| ctime          | 创建节点的时间                                        |
| mZxid          | 最后修改节点时的事务id                                   |
| mtime          | 最后修改节点的时间                                      |
| pZxid          | 节点的子节点列表发生修改的事务id（增加或者删除子节点，修改子节点的数据不会修改子节点列表） |
| cversion       | 子节点版本号，子节点每次修改版本号+1（只统计直属子节点）                  |
| dataversion    | 数据版本号，数据每次修改版本号+1                              |
| aclversion     | 权限版本号，权限每次修改版本号+1                              |
| ephemeralOwner | 创建该临时节点的会话的sessionId（如果是持久节点，这个属性值为0）          |
| dataLength     | 节点数据长度                                         |
| numChildren    | 该节点的子节点数量(直属子节点)                               |

## 11\. zookeeper session

客户端与服务端之间的连接是基于tcp连接的。

这个tcp连接，就是一个session会话。

每个session会话，zookeeper服务端都会分配一个sessionId，这个sessionId全局唯一，并且不会重复。

**分桶管理策略**

zookeeper会根据当前时间，计算下次超时时间。在超时时间内，客户端如果给服务端成功发送了读取、写入或者ping数据包，那么zookeeper就会将这个客户端的会话移动到下一次超时的列表中。

可以将时间按照超时时间，划分为一个个的时间区间(桶)

每当在时间区间内，有客户端成功发送了(成功发送，表示服务端成功读取哦)读取、写入或者ping数据包，那么zookeeper就会将会话移动到下一个区间。

这样，当时间到了下一个区间内，下一个区间内的会话都可用，而没有移动到下一个区间内的会话，则会被关闭。

会话超时时间默认是tickTime。(也就是必须要配置的几个参数之一)

需要注意的是，实际上zookeeper并不是一个时间区间一个时间区间移动session的，而是会考虑会话超时时间和检测间隔时间进行跨多个时间分区进行迁移。

举个例子吧：

会话超时时间：sessionTimeout 为 3秒

会话检测时间：tickTime为1秒

那么在迁移的时候，就是每3秒迁移次，此时迁移跨越了3个时间区间。

## 12\. zookeeper 数据同步

zookeeper 有一个非常重要的功能，就是为分布式应用提供一致性服务。

但是zookeeper本身也是分布式应用，那么zookeeper是如何保证自身数据的一致性的，并对外提供一致性服务？

### 12.1 ZAB协议

![image-20220301002500359](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ee0682ce2ddc04961cca1df6c82be060.png)

ZAB协议全称：Zookeeper Atomic Broadcast(zookeeper原子播送)

很多资料，张口ZAB，闭口ZAB，但是ZAB究竟是什么？看看全称就比较清晰了。(当你熟悉的时候使用简称无可厚非，但是刚接触的时候，知道全称非常有助于理解)

但是，ZAB协议到底是个啥？

> At the heart of ZooKeeper is an atomic messaging system that keeps all of the servers in sync.

这是文档中对于ZAB协议的说明。😟

这不是没说吗。😂

zookeeper 使用的消息系统的功能：

  1. 可靠传递：一个服务器传递一个消息，最终这个消息将会被广播到所有的节点。(消息广播传播)
  2. 广播顺序：广播是有序的，即使广播传播，也是有序的。（比如水波，水波在传递的时候，第一个波峰永远在第二个波峰前面）（强调所有的服务器在传递消息时，使用的相同的传递顺序）
  3. 消息有序：消息有序广播。（强调消息的顺序性，不能乱序传递）

但是，保证了上述三个特性，就会出现如下两个问题：

  * 有序传递消息丢失：如果某个消息发生了消息丢失，那么因为消息的有序性，丢失的消息之后的消息也会丢失。
  * 一次性：因为zookeeper的会话是依赖于TCP连接的，所以如果TCP连接关闭，那么这个会话将永远不会有消息传递发生。

因为会话的超时和时间有关，所以，保证zookeeper集群中节点的时间同步非常重要。

总结来说：ZAB协议，就是zookeeper使用的消息系统，有可靠性，有序性和一致性的特点。

### 12.2 ZAB协议原理

还记得 zookeeper Leader的选举过程吗？

根据日志，详细的理解和描述了zookeeper Leader选举过程中的节点之间的通信。

现在，专业一点，给zookeeper Leader选举过程中的一些操作做个定义。

在选举中，某一个节点推荐自己为leader，然后将信息发送给其他节点，期望获取其他节点的投票。这个操作，换个更加有效的说话：提案。

提案：提交会议讨论决定的建议。(我觉得这个翻译很准确，文档中是Proposal)

在选举中，某一个节点发起一个提案，然后通过消息广播到全部的节点。

全部的节点进行投票（讨论提案，赞成或反对），然后所有节点一起汇总投票结果，决定提案是否通过（过半原则）。

投票需要保证节点投票的有效性。（比如在投票A的时候，不能投票B）

那么在分布式节点中，如何保证投票的有效性？

消息超时+事务id

在Leader选举中，就看到有zxid这个属性，在节点属性中，也有事务id这个值。

在zookeeper中，事务id是保证消息顺序性的关键。

zxid是一个64位数字，这个64位数字分两部分组成：高32位和低32位。

高32位表示纪元号（每次产生新的Leader，纪元号就会+1）

低32位表示提案计数器（每次产生新的提案，计数器+1）

> 哇哦，我第一次理解这个特性的时候，真的被震惊到了，这和现实中朝代的年代计法多么类似。

通过这种方法，就保证了事务id的唯一性，即使发生了Leader选举，也不会重复。

发生leader选举后，新leader会获取集群内节点中最大的zxid，然后取前32位，+1作为自己的纪元号。

**提案模式**

使用提案模式，是有一定的好处的，那就是在提案通过之前，节点已经获取到了提案内容。

通俗点说，就是集群内每一个节点先获取到了消息，然后决定消息是否保存。

因为是数据先行，所以在提交过程中发生宕机，也能尽最大可能恢复数据。

恢复的标准就是事务id。事务id最大的节点的数据一定是最新的。

这里借用网上流行的一张图。

客户端发送写请求给zookeeper集群，集群内任意节点收到写请求后，节点会将写请求转发给leader，leader将写请求进行转化为提案，然后将提案广播到集群内每一个节点，因为是写请求提案，每一个节点都会通过。leader收到过半的节点通过后，会创建一个提交事务的提案，然后将提案广播到集群内每一个节点，然后每一个节点就会知道哪些数据应该持久化，哪些数据应该丢弃。

![这里写图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/162050c5e79e01352fe7109ee6a82179.png)

当然，zookeeper也给出了一个示意图：

![Two phase commit](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/213954461c96dd4919df51a99e1c084f.png)

### 12.3 ZAB VS 流言

看到这里，想起了redis哨兵模式中的流言协议。

ZAB是集群内全部节点参与提案，提交提案进行通信。

而redis哨兵模式则是需要选择节点作为哨兵，类似第三人观察，观察集群内的节点，如果集群内节点有一半以上的节点说某个master下线了，哨兵就会主观认为master下线，当超过一半的哨兵都主观认为master下线，那么此时master就客观下线了。

ZAB协议比起流言协议，减少了中间角色，资源利用更好，响应更加快速。

不过，流言协议将角色进行分离，更方便管理。各有优劣。

## 13\. zookeeper 监控

在实际使用中，不可能使用`zkCli.cmd`去监控全部的节点。

好在zookeeper也给出了监控方案：

![image-20220301002716309](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0dd06fc80caed953d2e18a8917296974.png)

当然，zookeeper并没有自己开发监控，而是集成了`Prometheus`翻译过来就是大名鼎鼎的 普罗米修斯 。

话说，普罗米修斯，我听过大名，但是却没有详细了解过他。🚩标记下来后面有机会研究研究。

首先需要下载，去[官网下载](<https://prometheus.io/download/>)，但是吧，下载真心慢哦。

下载完成后，会得到一个压缩包。

我下载的是windows版的。

![image-20220301002832026](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5487b68eddd1af9de1f37d8baa6b3819.png)

  1. 配置zookeeper的配置文件zoo.cfg

    
    
    metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
    metricsProvider.httpPort=7000
    

其实就是放开两个配置(注意，如果是一个物理机启动，未使用虚拟机，那么需要注意端口冲突)：

![image-20220301003026138](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0d8126327fc1fcfe0f7c7c19d9c5458c.png)

修改完记得启动集群。

  2. 配置Prometheus配置prometheus.ym

![image-20220301003251515](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fa71246d4695225d038b92517aaf37b5.png)

![image-20220301004102572](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d8201d9bda5e465a7140be6e87f19418.png)

配置名字(scrape_configs.job_name)，配置地址(scrape_configs.static_configs.targets)

  3. 启动prometheus.exe

![image-20220301003418600](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d387382dbfaeaf6413aee6a60945f70c.png)

  4. 访问

![image-20220301004122598](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e3bdf155586f18cb3fe6f8094070985b.png)

现在共有39个节点

![image-20220301004219054](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a966bffdabeac2af9afe593ff3a05400.png)

增加一个试试

![image-20220301004327445](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/21e97f776768cd043ad359a2a88f2161.png)

发现zookeeper-1已经增加了

![image-20220301004345168](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6c41c3924ed9201b2cdfb213c6433bcd.png)

在执行一次，就会发现其他两个节点也有了

![image-20220301004359981](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/13726a4e1d809b0fe99ebe37464b36c3.png)

还不错，就是怎么会用😂

## 14\. zookeeper 集成 java

在zookeeper use case 中，可以看到zookeeper如何集成其他语言的应用程序。

![image-20220301004445932](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f681470f7f4d174d4f5c0f707ff2d9df.png)

zookeeper提供了两套原生的api，分别对应java语言和c语言。只以java为例。

### 14.1 zookeeper原生API

首先在maven依赖中加入：
    
    
            <dependency>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
                <version>3.6.2</version>
            </dependency>
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-client</artifactId>
                <version>2.13.0</version>
            </dependency>
    

然后就会下载依赖包。
    
    
    import org.apache.zookeeper.CreateMode;
    import org.apache.zookeeper.KeeperException;
    import org.apache.zookeeper.ZooDefs;
    import org.apache.zookeeper.ZooKeeper;
    import org.apache.zookeeper.data.Stat;
    
    import java.io.IOException;
    import java.util.concurrent.TimeUnit;
    
    /**
     * @author jiayq
     * @Date 2022-03-01
     */
    public class ZookeeperT {
    
        public static void main(String[] args) throws IOException {
            try {
                // 设置会话链接信息
                ZooKeeper zooKeeper = new ZooKeeper("127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183", 6000, null);
                // 每隔 1 秒轮询是否建立连接
                while (ZooKeeper.States.CONNECTED != zooKeeper.getState()) {
                    TimeUnit.SECONDS.sleep(1);
                    // 如果连接断开了，直接终止程序
                    if (ZooKeeper.States.CLOSED == zooKeeper.getState()) {
                        System.out.println("connection closed!");
                        System.exit(0);
                    }
                }
                // 判断是否存在
                Stat exists = zooKeeper.exists("/helloworld", false);
                // 如果不存在则创建
                if (exists == null) {
                    // 创建
                    zooKeeper.create("/helloworld", "hello-xiaomei".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
                }
                // 读取
                System.out.println(new String(zooKeeper.getData("/helloworld", false, new Stat())));
            } catch (IOException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (KeeperException e) {
                e.printStackTrace();
            }
        }
    
    }
    
    

![image-20220301004706630](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f7be6af44224715fa3d204f51e2c3801.png)

其他比如权限，监视器以及其他操作，和客户端命令相同。

### 14.2 zookeeper的Curator的API

Curator 是 Netflix 公司开源的一套 zookeeper 客户端框架，解决了很多 Zookeeper 客户端非常底层的细节开发工作，包括连接重连、反复注册 Watcher 和 NodeExistsException 异常等。

  * **curator-framework** ：对 zookeeper 的底层 api 的一些封装。
  * **curator-client** ：提供一些客户端的操作，例如重试策略等。
  * **curator-recipes** ：封装了一些高级特性，如：Cache 事件监听、选举、分布式锁、分布式计数器、分布式 Barrier 等。

curator-recipes 

curator-framework 

curator-client 

zookeeper 

其组成关系如上图，因此，可以根据需要增加依赖。

首先增加maven依赖
    
    
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-recipes</artifactId>
                <version>2.13.0</version>
            </dependency>
    
    
    
    import com.google.common.base.Preconditions;
    import org.apache.curator.framework.CuratorFramework;
    import org.apache.curator.framework.CuratorFrameworkFactory;
    import org.apache.curator.retry.ExponentialBackoffRetry;
    import org.apache.zookeeper.CreateMode;
    
    import java.nio.charset.StandardCharsets;
    import java.util.Objects;
    
    /**
     * @author jiayq
     * @Date 2022-03-01
     */
    public class CuratorTest {
    
        public static void main(String[] args) throws Exception {
            CuratorFramework zkClient = getZkClient();
            String path = "/hello";
            if (Objects.nonNull(zkClient.checkExists().forPath(path))) {
                System.out.println(zkClient.delete().forPath(path));
            }
            zkClient.create().withMode(CreateMode.PERSISTENT).forPath(path, null);
            Preconditions.checkState(null == zkClient.getData().forPath(path), "not null");
            zkClient.setData().forPath(path, "hello".getBytes(StandardCharsets.UTF_8));
            System.out.println(new String(zkClient.getData().forPath(path), StandardCharsets.UTF_8));
            Preconditions.checkNotNull(zkClient.getData().forPath(path), "is null");
        }
    
        private static CuratorFramework getZkClient() {
            String zkServerAddress = "127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183";
            ExponentialBackoffRetry retry = new ExponentialBackoffRetry(1000, 3, 5000);
            CuratorFramework zkClient = CuratorFrameworkFactory.builder()
                    .connectString(zkServerAddress)
                    .sessionTimeoutMs(5000)
                    .connectionTimeoutMs(5000)
                    .retryPolicy(retry)
                    .build();
            zkClient.start();
            return zkClient;
        }
    }
    
    

![image-20220301005050645](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/653b8b12692a4a8183fe20fe4068b9a5.png)

除了接口风格不同，其他都差不多的。

## 15\. zookeeper 分布式锁

zookeeper 中比较经典的使用就是分布式锁了，可以说，分布式锁完美的使用了zookeeper的特性。

**zookeeper分布式锁的原理**

利用zookeeper中临时顺序节点(zookeeper中共有4中节点，永久节点，永久顺序节点，临时节点，临时顺序节点)的特性：临时性(客户端断开连接，节点自动删除，这点非常重要，可以防止死锁)，顺序性(公平锁的顺序性)。

核心操作如下：

首先创建锁主节点`/lock`。

每当有节点需要获取锁，就在`/lock`下创建临时顺序节点。

然后判断创建的节点是否是序号最小的节点。如果是，获取锁。如果不是，注册节点监视器给创建节点的前一个节点。监视节点删除。如果监视器被触发，那么判断当前节点是否是序号最小的节点，如果是，获取锁，如果不是，注册监视器。

防止羊群效应(羊群效应，注册的监听大规模响应)

jdk锁的实现中，就存在羊群效应(每当锁释放，就会唤醒等待队列中的线程争抢锁)。

## 16\. 总结

在不了解zookeeper的时候，就是简单的认为zookeeper就是一个注册中心，配置中心，仅此而已。

但是当了解了zookeeper之后，才发现，zookeeper远远比想象中的强大多了。

甚至于里面的一些实现，都是非常精妙，高效的。

没有系统的学习过zookeeper，就不会理解zookeeper的妙。
