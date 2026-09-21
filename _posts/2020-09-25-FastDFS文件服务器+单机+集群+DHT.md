---
layout: post
title: "FastDFS文件服务器+单机+集群+DHT"
date: 2020-09-25 15:12:51 +0800
categories: [FastDFS单机版搭建, FastDFS集群搭建+DHT, FastDFS如何配置, FastDFS原理解析, FastDFS如何集成]
description: "FastDFS文件服务器+单机+集群+DHT1. 需求分析2. 现状2.1 tomcat静态资源2.2 ftp服务器2.3 数据库2.4 MogileFS2.5 FastDfs2.6 Zimg2.7 OSS3. FastDFS和MogileFS对比4. 具体实施措施4.1 docker镜像4.2 docker-compose文件4.2.1 tracker4.2.2 storage4.3 配置4.3.1 tracker-fdht4.3.2 tracker-fdfs4.3.3 storage5. 扩展5.1 水_fastdfs 单机 迁移集群"
keywords: FastDFS单机版搭建, FastDFS集群搭建+DHT, FastDFS如何配置, FastDFS原理解析, FastDFS如何集成
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/108797221
> - 发布时间：2020-09-25 15:12:51
> - 阅读量：1
> - 分类：微服务同时被 2 个专栏收录, 订阅专栏, 文件服务器
> - 标签：#FastDFS单机版搭建, #FastDFS集群搭建+DHT, #FastDFS如何配置, #FastDFS原理解析, #FastDFS如何集成

## 摘要

文章浏览阅读1k次。FastDFS文件服务器+单机+集群+DHT1. 需求分析2. 现状2.1 tomcat静态资源2.2 ftp服务器2.3 数据库2.4 MogileFS2.5 FastDfs2.6 Zimg2.7 OSS3. FastDFS和MogileFS对比4. 具体实施措施4.1 docker镜像4.2 docker-compose文件4.2.1 tracker4.2.2 storage4.3 配置4.3.1 tracker-fdht4.3.2 tracker-fdfs4.3.3 storage5. 扩展5.1 水_fastdfs 单机 迁移集群

---

#### FastDFS文件服务器+单机+集群+DHT

  * 1\. 需求分析
  * 2\. 现状
  *     * 2.1 tomcat静态资源
    * 2.2 ftp服务器
    * 2.3 数据库
    * 2.4 MogileFS
    * 2.5 FastDfs
    * 2.6 Zimg
    * 2.7 OSS
  * 3\. FastDFS和MogileFS对比
  * 4\. 具体实施措施
  *     * 4.1 docker镜像
    * 4.2 docker-compose文件
    *       * 4.2.1 tracker
      * 4.2.2 storage
    * 4.3 配置
    *       * 4.3.1 tracker-fdht
      * 4.3.2 tracker-fdfs
      * 4.3.3 storage
  * 5\. 扩展
  *     * 5.1 水平扩展
    * 5.2 垂直扩展
    * 5.3 查看FastDFS系统信息
    * 5.4 验证FastDFS系统
  * 6\. java程序连接FastDFS
  *     * 6.1 上传
    * 6.2 下载
    * 6.3 删除
  * 7\. 总结

  
github地址:https://github.com/a18792721831/fastdfs-docker-compose.git 

## 1\. 需求分析

因业务需求，现在需要搭建一个图片服务器，用于存储图片，而图片访问的地址则存储于数据库中。

而且，图片服务器需要考虑扩展，不仅仅需要存储图片，还有可能存储视频、文档等等。

其实就是自建一个文件服务器。

## 2\. 现状

作为图片服务器，目前的解决方案有很多种。

### 2.1 tomcat静态资源

自己写一个上传，下载的控制器，然后将图片存储于tomcat容器下的某个目录，图片存储于这个目录，访问时，直接指定路径即可。

缺点：

  1. 暴露服务器文件目录结构
  2. 性能差，安全程度低，如果存在客户恶意上传，那么很快会导致服务器宕机
  3. 没有备份，抗灾能力差
  4. 没有负载均衡，没有分布式，并发能力差
  5. 没有索引，搜索性能差
  6. 没有缓存，性能差
  7. 没有压缩，网络传输性能差
  8. 可扩展性差
  9. 没有权限，角色管理，安全性差

优点：

  1. 简单，无需额外成本

### 2.2 ftp服务器

直接将图片上传到ftp服务器，然后提供下载。

缺点：

  1. 暴露服务器文件目录结构
  2. 性能差，安全程度低，如果存在客户恶意上传，那么很快会导致服务器宕机
  3. 没有备份，抗灾能力差
  4. 没有负载均衡，没有分布式，并发能力差
  5. 没有索引，搜索性能差
  6. 没有缓存，性能差
  7. 没有压缩，网络传输性能差
  8. 可扩展性差
  9. 没有权限，角色管理，安全性差

优点：

  1. 简单，无需额外成本
  2. 成熟，解决方案成熟可用

### 2.3 数据库

在数据库中创建表，以clob的方式存储资源。

缺点：

  1. 性能差，安全程度低，如果存在客户恶意上传，那么很快会导致服务器宕机
  2. 没有备份，抗灾能力差
  3. 没有负载均衡，没有分布式，并发能力差
  4. 没有缓存，性能差
  5. 没有压缩，网络传输性能差
  6. 可扩展性差
  7. 没有权限，角色管理，安全性差

优点：

  1. 简单，无需额外成本

数据库就不应该存储资源。因为资源通常占用较大的空间，而服务器空间是很宝贵的。

### 2.4 MogileFS

MogileFS是一个开源的分布式文件存储系统，是由LiveJournal旗下的Danga Interactive公司开发。目前使用MogileFS的公司非常多，如日本排名先前的几个互联公司以及国内的Yupoo(又拍)、digg、豆瓣、大众点评、搜狗等，分别为所在的组织或公司管理着海量的图片。以大众点评为例，用户全部图片均有MogileFS存储，数据量已经达到500TB以上

缺点：

  1. 整个架构非常复杂，管理，实施起来不太容易
  2. 系统复杂，出现问题比较难解决

优点：

  1. 性能好，安全
  2. 有备份，抗灾能力强
  3. 有负载均衡，支持分布式，并发能力强
  4. 易扩展
  5. 大厂都在用

MogileFS由3个部分组成：

1、server：主要包括mogilefs和mogstored两个应用程序。

mogilefs实现的是tracker，它通过数据库来保存元数据信息，包括站点domain、class、hots等；mogstored是存储节点（storgenode’），它其实是个WsbDAV服务，默认监听在7500 端口，接受客户端的文件存储请求。在Mogilefs安装完后，要运行mogadm工具将所有storge node注册到mogilefs的数据库里，mogilefs会对这些节点进行管理和监控。

2、utils（工具集）：主要是Mogilefs的一些管理工具，例如mogadm等

3、客户端API：mogilefs的客户端API很多，例如Perl、PHP、java、python等，用这个模块可以编写客户端程序，实现文件的备份管理功能等

MogileFS的架构图：

![img](https://i-blog.csdnimg.cn/blog_migrate/477816fef351d0a48dda3125ef5195bc.png)

![img](https://i-blog.csdnimg.cn/blog_migrate/4302777cc7e338466421766191a69194.png)

上图为Mogilefs架构图，下面也描述了图中体现出一次数据请求过程。  
1、 客户端在发起一次数据请求，假设请求banner.jpg,请求首先到达前端代理perbal（当然此处可利用Nginx实现）  
2、 perbal或nginx会将请求代理至Mogilefs client（这里解释下，MogileFS本身就是一个Web服务可以提高返回数据信息，不过普通的浏览器或web客户端是不具备利用获取的Mogilefs返回的数据位置信息再次请求storage节点获取数据的，所以此处需要一个特定的客户端来访问Mogilefs，而此客户端在Nginx代理时，是nginx的特定的模块。）  
3、 mogilefs client模块将请求发往trackers节点，trackers向DB server发起查询  
4、 tracker将以banner.jpg为key查询到的vlaue值发给nginx。  
5、 Nginx通过Mogilefs API向storage 节点请求数据。

这就是一次完整的数据获取过程。

更多详情请见：[MogileFS](<https://blog.csdn.net/shangyuanlang/article/details/80865894>)，[mogilefs-docs](<https://github.com/mogilefs/mogilefs-docs>)，[github-mogilefs](<https://github.com/mogilefs>)

### 2.5 FastDfs

  * fastDFS 是以C语言开发的一项开源轻量级分布式文件系统，他对文件进行管理，主要功能有：文件存储，文件同步，文件访问（文件上传/下载）,特别适合以文件为载体的在线服务，如图片网站，视频网站等

  * 分布式文件系统：  
基于客户端/服务器的文件存储系统  
对等特性允许一些系统扮演客户端和服务器的双重角色，可供多个用户访问的服务器，比如，用户可以“发表”一个允许其他客户机访问的目录，一旦被访问，这个目录对客户机来说就像使用本地驱动器一样

FastDFS由跟踪服务器(Tracker Server)、存储服务器(Storage Server)和客户端(Client)构成。

FastDFS的系统结构图：

![img](https://i-blog.csdnimg.cn/blog_migrate/3a6a55af3c36d702e54748f55c65187b.png)

FastDFS网络时序图

![img](https://i-blog.csdnimg.cn/blog_migrate/59bb2d2a9134c605eab5adb6ab759c39.png)

### 2.6 Zimg

  * zimg是图像存储和处理服务器。您可以使用URL参数从zimg获得压缩和缩放的图像。  
http://demo.buaa.us/5f189d8ec57f5a5a0d3dcba47fa797e2?w=500&h=500&g=0&x=0&y=0&r=45&q=75&f=jpeg
  * 参数包括宽度，高度，调整大小类型，灰色，裁剪位置（x，y），旋转，质量和格式。您可以通过配置文件控制图像的默认类型。  
您可以像这样在zimg服务器中获取图像信息：[http](<http://demo.buaa.us/info?md5=5f189d8ec57f5a5a0d3dcba47fa797e2>) 😕/demo.buaa.us/info?md5=5f189d8ec57f5a5a0d3dcba47fa797e2
  * 如果要自定义图像的变换规则，可以编写zimg-lua脚本。`t=type`在URL中使用参数来获取特殊图像：[http](<http://demo.buaa.us/5f189d8ec57f5a5a0d3dcba47fa797e2?t=webp500>) :  
[//demo.buaa.us/5f189d8ec57f5a5a0d3dcba47fa797e2?t=webp500](<http://demo.buaa.us/5f189d8ec57f5a5a0d3dcba47fa797e2?t=webp500>)
  * zimg的并发I / O，分布式存储和及时处理能力非常出色。您不再需要在图像服务器中使用nginx。在基准测试中，zimg可以在高并发级别上每秒处理3000个以上的图像下载任务和每秒90000个以上的HTTP回显请求。性能高于PHP或其他图像处理服务器。

下面罗列zimg可以提供的常见功能：

  1. 所有图片默认返回质量为75%，JPEG格式的压缩图片，这样肉眼无法识辨，但是体积减小
  2. 获取宽度为x，被等比例缩放的图片
  3. 获取旋转后的图片
  4. 获取指定区域固定大小的图片
  5. 获取特定尺寸的图片，由于与原图比例不同，尽可能展示最多的图片内容，缩放之后多余的部分需要裁掉
  6. 获取特定尺寸的图片，要展示图片所有内容，因此图片会被拉伸到新的比例而变形
  7. 获取特定尺寸的图片，但是不需要缩放，只用展示图片核心内容即可
  8. 获取按指定百分比缩放的图片
  9. 获取指定压缩比的图片
  10. 获取去除颜色的图片
  11. 获取指定格式的图片
  12. 获取图片信息
  13. 删除指定图片

而以上这些功能的提供，仅需要一个url+特定的参数，通过get方式就可以完成，这才是简便之处。

zimg提供三种存储方式：本地磁盘，beansdb，ssdb三种。单机存储，依据其目录结构设计，可以存储1024 * 1024 * 1024 * 200KB = 200TB（单图200KB大小）数据量，切换成beansdb或ssdb，后续可扩展成更大容量的存储完全不是问题。

个人意见：

zimg是个人开源的一个C实现的集图片存储，图片简单编辑的轻量级的图片服务器。最新版本于2017.11.5发布后，再无后续， 目前github有98issues，5个pull request,last commit 是2018Otc.几乎可以想到，作者应该是已经放弃这个项目了，慎用。

更多详情请见：[zimg](<http://zimg.buaa.us/>)，[zimg-github](<https://github.com/buaazp/zimg>)

### 2.7 OSS

市面上各云服务厂商几乎都有OSS解决方案。基本上云服务厂商的OSS服务比自建的服务要可靠的多，速度也更快，安全性更高，运营成本更低。

如果有可能，还是尽可能选择OSS解决方案。

## 3\. FastDFS和MogileFS对比

基于目前的文件系统解决方案，排除OSS等之后，可能目前最好的选择就是FastDFS和MogileFS了。

这两个实现的原理和功能也很类似。

>   * 1 )支持多节点冗余
>   * 2）可实现自动的文件复制
>   * 3）使用名称空间（命名空间），每个文件通过key来确定，比如123.jpg是一个key，真正存储的位置可能是/000/000/00/01/md5hash.fid
>   * 4）不需要RAID，应用层可直接实现RAID，不共享任何东西，通过”集群接口”提供服务
>   * 5）工作于应用层，没有特殊的组件要求
>   * 6）不用共享任何数据，MogileFS不需要依靠昂贵的SAN来共享磁盘，每个机器只用维护好自己的磁盘
> 

MogileFS主要由三部分构成：tracker节点、database节点、storage节点

![这里写图片描述](https://i-blog.csdnimg.cn/blog_migrate/48f61a1db01d26e5f57912281734dca6.png)

>   * 1、分组存储，灵活简洁、对等结构，不存在单点
>   * 2、 文件ID由FastDFS生成，作为文件访问凭证。FastDFS不需要传统的name server
>   * 3、和流行的web server无缝衔接，FastDFS已提供apache和nginx扩展模块
>   * 4、大、中、小文件均可以很好支持，支持海量小文件存储
>   * 5、 支持多块磁盘，支持单盘数据恢复
>   * 6、 支持相同文件内容只保存一份，节省存储空间
>   * 7、 存储服务器上可以保存文件附加属性
>   * 8、 下载文件支持多线程方式，支持断点续传
> 

**FastDFS同步机制**  
采用binlog文件记录更新操作，根据binlog进行文件同步  
同一组内的storage server之间是对等的，文件上传、删除等操作可以在任意一台storage server上进行；  
文件同步只在同组内的storage server之间进行，采用push方式，即源服务器同步给目标服务器；  
源头数据才需要同步，备份数据不需要再次同步，否则就构成环路了；  
上述第二条规则有个例外，就是新增加一台storage server时，由已有的一台storage server将已有的所有数据（包括源头数据和备份数据）同步给该新增服务器。

![这里写图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb4795cbf9e60f5aeedd0272ae7de3c5.png)

![这里写图片描述](https://i-blog.csdnimg.cn/blog_migrate/107fb7fae656195356a44b847ee94917.png)

![这里写图片描述](https://i-blog.csdnimg.cn/blog_migrate/dab079250d232660b7de52900fabe69a.png)

基于这些原因，采用FastDFS作为文件存储的解决方案。

## 4\. 具体实施措施

### 4.1 docker镜像

FastDFS服务器的搭建方式不是很难，但是，为了更高的隐藏底层操作，同时也是更好的融入公司生态环境，采用docker镜像的方式部署FastDFS。

[FastDFS镜像](<https://hub.docker.com/search?q=fastdfs&type=image>)

综合考虑，采用如下docker镜像：

tracker:[FastDFS+FastDHT单机版](<https://hub.docker.com/r/qbanxiaoli/fastdfs>)

storage:[FastDFS官方镜像](<season/fastdfs>)(可能不是真正的官方镜像，但是这个却是下载次数最多的)

为什么采用不同的镜像？

首先，在FastDFS中，不管是tracker还是storage，都是需要同client进行通信的(他通过前面的网络时序图和系统结构图可以看出)，也就是说，FastDFS都需要使用host模式(其他模式获取的ip地址是docker内网地址，无法与client通信)。

在文件服务器中，如果有一个文件已经被存储了，那么，后续在存储相同的文件，就不应该重复存储。但是FastDFS没有集成任何DB，所以，对于重复文件，FastDFS是无法处理的。这个问题的解决方式就是使用DHT服务器(这个解决方案也是FastDFS的作者`余庆`提出的)，而且为了实现统一接口交互(至少对我们的业务代码来说)，tracker还需要nginx做代理。

因此，FastDFS+FastDHT单机版就是我们tracker的最佳选择。

作为storage，核心是提供存储即可，其他并不需要storage关心，因此，storage使用官方的纯净的镜像即可。

### 4.2 docker-compose文件

有了docker镜像，还远远不够。镜像提供的配置，仅仅是默认配置，并不一定能够完全满足我们的需求，因此，我们需要将镜像内的配置文件暴露出来，而且，作为文件存储服务器，最重要的就是文件数据，如果不使用目录挂载，那么所有的数据存储在容器内，这是不正确的。

因此需要挂载目录，配置启动参数等等，所以，使用docker-compose进行管理容器。

既然使用了docker镜像，我们希望docker-compose和相关的目录等，在所有的linux中都可以使用，需要忽略目录差异，因此，需要将docker-compose相关的目录和文件放置在`~`目录下。

#### 4.2.1 tracker

![image-20200925102302223](https://i-blog.csdnimg.cn/blog_migrate/24ca3fbc89372a65647011ceb7df7b17.png)

这个目录就是tracker的目录，将这个目录整个上传到`~`下即可。

`fdfs-tracker`目录内分为三部分：docker-compose.yaml，config，data

**config目录**

![image-20200925102413752](https://i-blog.csdnimg.cn/blog_migrate/fc731b64d4783f58c4603aa340185147.png)

其中config中是FastDFS中全部的配置文件，可以根据需要随时调整。

![image-20200925102503536](https://i-blog.csdnimg.cn/blog_migrate/66a57745e90c21eb8b91c921c606b1d5.png)

![image-20200925102513137](https://i-blog.csdnimg.cn/blog_migrate/8f8c11529a307542b452c29441ab6aa4.png)

![image-20200925102523132](https://i-blog.csdnimg.cn/blog_migrate/719cb159ca12fc564c68533776094718.png)

**docker-compose.yaml**
    
    
    version: '3'
    services:
      fastdfs:
        image: qbanxiaoli/fastdfs
        # 该容器是否需要开机启动+自动重启。若需要，则取消注释。
        restart: always
        container_name: fastdfs
        environment:
          # nginx服务端口,默认80端口，可修改
        - WEB_PORT=40080
          # tracker_server服务端口，默认22122端口，可修改
        - FDFS_PORT=22122
          # storage_server服务端口，默认23000端口，可修改
        - STORAGE_PORT=23000
          # fastdht服务端口，默认11411端口，可修改
        - FDHT_PORT=11411
          # docker所在的宿主机内网地址，默认使用eth0网卡的地址
        - IP=
        volumes:
        # 将本地目录映射到docker容器内的fastdfs数据存储目录，将fastdfs文件存储到主机上，以免每次重建docker容器，之前存储的文件就丢失了。
        - ~/fdfs-tracker/tracker/:/var/local
        - ~/fdfs-tracker/config/fdfs/:/etc/fdfs/
        - ~/fdfs-tracker/config/fdht/:/etc/fdht/
        # 网络模式为host，可不暴露端口，即直接使用宿主机的网络端口，只适用于linux系统
        network_mode: host
        tty: true
    

这里需要注意，docker-compose里面配置的端口和目录，与config中的配置应该保持一致。

如果物理机的eth0网卡的地址不是外网地址，应该配置IP环境变量。

docker-compose里面的目录都是以`~`为开始目录进行挂载的。

网络模式必须为host。

tracker里面包含traker和一个tracker下的storage.

**tracker目录**

tracker目录在创建时，保持空目录即可，容器会自己创建其他的目录和数据。

![image-20200925103321712](https://i-blog.csdnimg.cn/blog_migrate/510637800dd9995142d06ce8b1f651ae.png)

tracker目录下分为fdfs和fdht目录。

fdht目录下是相关数据：

![image-20200925103406393](https://i-blog.csdnimg.cn/blog_migrate/31449e091b0076bb6f7261fb40715e76.png)

fdht的日志在logs目录下。

![image-20200925103427318](https://i-blog.csdnimg.cn/blog_migrate/e5cf5a52755d86e4cf66e0d6ba22a149.png)

fdfs目录下分为两个目录：tracker+storage(因为tracker也有一个storage)

![image-20200925103521285](https://i-blog.csdnimg.cn/blog_migrate/f9463a12120327c86bbbb78e08dcd500.png)

tracker目录下分为两个目录：data+logs

![image-20200925103546926](https://i-blog.csdnimg.cn/blog_migrate/e4603db6f51a1ccfcacd16d6ad0a22f3.png)

data目录是整个FastDFS系统的一些交换信息

![image-20200925103617585](https://i-blog.csdnimg.cn/blog_migrate/459c69e694c6cb71ade5a888f398bd4d.png)

logs目录是整个FastDFS系统的调度等日志

![image-20200925103641830](https://i-blog.csdnimg.cn/blog_migrate/8393df003d766f381109c75d35dd3984.png)

storage目录是tracker服务器上的storage的相关数据和日志

![image-20200925103719960](https://i-blog.csdnimg.cn/blog_migrate/51392d33be1a98b735e19bfd9b6aee1d.png)

data就是实际存储文件的位置。

![image-20200925103745440](https://i-blog.csdnimg.cn/blog_migrate/8dc27a68ceb125caad570693d7c4f9de.png)

logs目录就是storage的日志

![image-20200925103810022](https://i-blog.csdnimg.cn/blog_migrate/647c1d76e1f58017af92b0ca8bd57d1c.png)

这些目录和文件都不需要创建，保证fdfs-tracker下存在tracker目录即可。

![image-20200925103856431](https://i-blog.csdnimg.cn/blog_migrate/f10c35cd3b167df5068e5743efe03752.png)

#### 4.2.2 storage

storage使用的是官方的纯净的镜像，和tracker类似，也是将数据和配置全部挂载。

![image-20200925104020791](https://i-blog.csdnimg.cn/blog_migrate/0bc0179aee06c44261e097ca1a1be9cf.png)

fdfs-storage就是storage相关的目录。

也是整体上传到服务器的`~`目录下就行。fdfs-storage和fdfs-tracker不能再同一个服务器上启动，存在端口冲突。

fdfs-storage包含三个部分：docker-compose.yaml，config和storage目录

![image-20200925104222059](https://i-blog.csdnimg.cn/blog_migrate/5bbd59b5110acebf9359dd7b071b8e48.png)

**config目录**

config目录下是storage的全部的FastDFSstorage的配置

![image-20200925104248205](https://i-blog.csdnimg.cn/blog_migrate/78ee7a24623420e9247d9b02f92a37ad.png)

**docker-compose.yaml**
    
    
    version: '3'
    services:
      storage:
        image: season/fastdfs
        command: storage
        # 该容器是否需要开机启动+自动重启。若需要，则取消注释。
        restart: always
        container_name: storage
        environment:
        - TRACKER_SERVER=10.0.228.153:22122
        volumes:
        # 将本地目录映射到docker容器内的fastdfs数据存储目录，将fastdfs文件存储到主机上，以免每次重建docker容器，之前存储的文件就丢失了。
        - ~/fdfs-storage/storage/:/fastdfs/storage/
        #- ./storage/:/fastdfs/store_path
        - ~/fdfs-storage/config/:/fdfs_conf/
        # 网络模式为host，可不暴露端口，即直接使用宿主机的网络端口，只适用于linux系统
        network_mode: host
        tty: true
    

和tracker的docker-compose文件类似。区别在于，storage需要指定tracker服务器的连接信息，也就是需要增加`TRACKER_SERVER`的环境变量。

**storage目录**

storage目录下需要手动创建data目录

![image-20200925104523865](https://i-blog.csdnimg.cn/blog_migrate/71d47c855fa4445db6d3f6bb66d94499.png)

data目录下需要手动创建一个pid文件，文件名必须是`fdfs_storaged.pid`里面填写一个大于1的数字即可。

![image-20200925104614183](https://i-blog.csdnimg.cn/blog_migrate/07b6d867772187147b102bd4ec5fe6ff.png)

![image-20200925104625102](https://i-blog.csdnimg.cn/blog_migrate/9a681a5f70a612e70c7915a210d58229.png)

为什么tracker不需要？

因为tracker使用镜像做了这件事，而官方的镜像没有那么智能，需要手动创建。这个pid文件就是storage主进程启动时使用的id号。

storage容器启动后，会创建logs目录

![image-20200925104837644](https://i-blog.csdnimg.cn/blog_migrate/7874356b3ef43fef6313c90c8ed96eee.png)

在data目录下会创建实际存储的文件目录：

![image-20200925104902256](https://i-blog.csdnimg.cn/blog_migrate/71e0cc7bb9d052db21152c3bcaaa2109.png)

logs目录下是storage的目录：

![image-20200925104919928](https://i-blog.csdnimg.cn/blog_migrate/9eb8d4c4d90f65c968acba54602e2eef.png)

实际使用非常简单：

  1. 上传相关的文件夹到服务器的`~`目录下
  2. 修改相关的配置
  3. 进入`fdfs-tracker`或者`fdfs-storage`目录
  4. 执行`docker-compose up -d`即可(需要服务器安装docker和docker-compose，而且对于配置的端口没有端口冲突)

### 4.3 配置

#### 4.3.1 tracker-fdht

dht包含三个配置文件：

![image-20200925105258118](https://i-blog.csdnimg.cn/blog_migrate/0f76510a57af6e91cca031ef2c51e46c.png)

**fdhtd.conf**

核心的配置：
    
    
    # 当前配置文件是否不允许(false表示允许)
    disabled=false
    # fdht服务的端口
    port=11411
    # fdht的目录(里面存储data和logs)
    base_path=/var/local/fdht
    # 访问白名单(如果是外网环境，可以在一定程度上保护dht服务器)
    allow_hosts=*
    

其他的配置在配置文件中有说明，可以随着使用，进行不断的调整。

**fdht_server.conf**

![image-20200925105858642](https://i-blog.csdnimg.cn/blog_migrate/6ccc855c269d3d4b55054f35dccd74b9.png)

里面就两个配置，第一个表示dht分为几组，第二个配置就是实际的分组了。

为什么会有分组？

因为dht服务器可以是集群，多个dht服务是对等的，在单机dht压力大的时候，可以配置多个。

如果配置多个，那么fdht_server.conf配置可能就是这样的：
    
    
    # 分组数量
    group_count = 3
    # 第一个分组
    group0=10.0.228.153:11411
    # 第二个分组
    group1=10.0.228.154:11411
    # 。。。
    group2=10.0.228.155:11411
    
    

**fdht_client.conf**

配置dht客户端连接的一些信息：

核心配置
    
    
    # dht的目录
    base_path=/var/local/fdht
    # 引入fdht_servers.conf文件
    #include fdht_servers.conf
    

#### 4.3.2 tracker-fdfs

里面的配置很多，并不是全部的配置文件都是tracker的，还有一些是storage的。

核心配置

tracker.conf
    
    
    # 端口，和docker-compose里面的端口保持一致
    port=22122
    # 目录，和docker-compose里面的目录保持一致(容器内目录)
    base_path=/var/local/fdfs/tracker
    # 留给操作系统和其他程序的空间(fdfs使用的空间等于=总空间-resered_storage_space)
    reserved_storage_space = 20g
    # 白名单
    allow_hosts=*
    

mod_fastdfs.conf
    
    
    # 连接超时时间
    connect_timeout=2
    # 等待时间
    network_timeout=30
    # 存储目录
    base_path=/var/local/fdfs/tracker
    # tracker服务的地址
    tracker_server=10.0.228.153:22122
    # 当前服务器上的storage的端口
    storage_server_port=23000
    # 当前服务器上的组名，可以有多个，用 / 区分
    group_name=group1
    # 请求文件时，是否包含组名
    url_have_group_name = true
    # 当前服务器挂载的硬盘或者目录个数
    store_path_count=1
    # 挂载目录地址
    store_path0=/var/local/fdfs/storage
    #store_path1=/home/yuqing/fastdfs1
    # 当前服务器上分组个数，与组名对应，如果只有一个写0即可，如果写1，需要配置这个分组的信息，写0不需要
    group_count = 0
    # 如果有多个分组，需要配置组名，端口和挂载的磁盘
    #[group1]
    #group_name=group1
    #storage_server_port=23000
    #store_path_count=2
    #store_path0=/home/yuqing/fastdfs
    #store_path1=/home/yuqing/fastdfs1
    

client.conf
    
    
    # 连接超时时间
    connect_timeout=30
    # 网络超时时间
    network_timeout=60
    # storage 的log存储目录
    base_path=/var/local/fdfs/storage
    # tracker服务的地址
    tracker_server=10.0.228.153:22122
    # tracker服务器的http端口(作用不大，主要是调试的时候，上传成功，使用浏览器访问，程序都是走23000端口的)
    http.tracker_server_port=80
    

#### 4.3.3 storage

storage 中的其他配置和tracker-fdfs中的一样，一个服务中的配置应该保持一致。

核心是storage.conf文件
    
    
    # 是否允许
    disabled=false
    # 当前storage中包含几个分组(一个服务器只能有一个storage，但是分组和storage是多对多关系，分组和storage_path是多对多关系)
    group_name=group1
    # 当前storage的端口(和分组有一定的关系，这个配置只在group_count=0时有效)
    port=23000
    # 容器内的目录
    base_path=/var/local/fdfs/storage
    # 当前服务器挂载的硬盘或者目录个数
    store_path_count=1
    # 挂载目录地址
    store_path0=/var/local/fdfs/storage
    #store_path1=/home/yuqing/fastdfs1
    # tracker服务的地址
    tracker_server=10.0.228.153:22122
    # 白名单
    allow_hosts=*
    # storage可以连接的dht服务的配置，和tracker-dht中fdht_servers.conf一致
    #include /etc/fdht/fdht_servers.conf
    # http访问storage的端口
    http.server_port=40080
    

## 5\. 扩展

随着业务的发展，当前的配置肯定存在不够使用的情况。当资源不足的时候，就需要进行扩展。

扩展就我理解，分为两个方向：水平扩展和垂直扩展。

**水平扩展** ：随着业务发展，系统增加了一个子系统，或者一个项目，或者按需存储不同类型的文件，现在增加了一种类型，那么就需要增加一个group。如果一个storage存储一个group，那么可以不用修改现有的配置，但是这个不安全，如果某个类型的storage的服务器损坏，那么这个类型的文件全部丢失。更好一些的是，每个group有多个storage进行备份，这些storage和group可以交叉。所以，水平扩展是增加不同数据的storage。

**垂直扩展** ：随着用户数量的增加，每个storage的并发量在增加，达到一定程度后，单个storage已经无法满足并发了，此时需要对storage做负载均衡。所以，垂直扩展是增加相同数据的storage的数量，达到负载均衡和数据备份的目的。

### 5.1 水平扩展

![image-20200925113552997](https://i-blog.csdnimg.cn/blog_migrate/129355012a24c742a1adb83a91bcd3ed.png)

增加的storage与原有的组名不同即可。(同一个服务器中的配置文件中相同的配置应该保持一致)

如果每个组名下只有一个storage，那么表示文件只存储了一份，没有备份，请小心。

如果新增一个storage需要备份全部的数据，那么新增的storage配置全部的组名即可。

### 5.2 垂直扩展

![image-20200925113936879](https://i-blog.csdnimg.cn/blog_migrate/04b45705c81e7114c0d3ec0f9b58fbd3.png)

增加的storage与原有的组名相同即可。(同一个组名的storage的端口应该保持一致)

同个组名下有几个storage，就是相同的数据有几份。

### 5.3 查看FastDFS系统信息

可以进入docker容器，也可以启动官方镜像，只启动shell.

![image-20200925114045010](https://i-blog.csdnimg.cn/blog_migrate/6a43b9566ec32aee6886d73de700abff.png)

然后使用fdfs_monitor程序

![image-20200925114211013](https://i-blog.csdnimg.cn/blog_migrate/db7f3fba3073d242aedbf5215564ef17.png)

输出结果

![image-20200925114341257](https://i-blog.csdnimg.cn/blog_migrate/690e34032ac015cbe36bb787e65187ba.png)  
![image-20200925114348940](https://i-blog.csdnimg.cn/blog_migrate/3afae503eda526b59b4e93727713672a.png)

![image-20200925114358256](https://i-blog.csdnimg.cn/blog_migrate/c37d716be7312ffd7d96d5a362dcd591.png)

这个可以在任意storage,tracker或者shell中执行。

### 5.4 验证FastDFS系统

在`fdfs-tracker/tracker/`下存储一张图片，用于测试。(任意一个storage,tracker或者shell都可以验证)

![image-20200925114502832](https://i-blog.csdnimg.cn/blog_migrate/4741e058f353a01e43d8f0f1a6200a05.png)

上传

然后使用`fdfs_upload_file`上传

![image-20200925114630599](https://i-blog.csdnimg.cn/blog_migrate/7b3f709f5ef288e63098f51f8ef12642.png)

返回`group1/M00/00/00/CgDkmV9taA6AMBn0AAhzLEdg5HE776.jpg`上传成功

接着我们访问`http://tracker_server:port/group1/M00/00/00/CgDkmV9taA6AMBn0AAhzLEdg5HE776.jpg`

因为tracker集成了nginx，所以http直接访问tracker就行了。

比如

![image-20200925114805996](https://i-blog.csdnimg.cn/blog_migrate/a7f59feae69a6e4f716b03b86fe36478.png)

下载

使用`fdfs_download_file`下载

![image-20200925114941221](https://i-blog.csdnimg.cn/blog_migrate/3af81d5c9a72828b59646d8a32cd59ff.png)

删除

使用`fdfs_delete_file`删除(删除服务器上的文件)

![image-20200925115134269](https://i-blog.csdnimg.cn/blog_migrate/9ad2aef7a29e93c25e1fd388e77df943.png)

浏览器依然能访问，可能是这几个原因：浏览器缓存，tracker中的nginx的缓存。  
演示使用的是tracker容器，storage容器中程序和配置的所在位置可能和tracker容器不同，可以使用`find`搜索。

## 6\. java程序连接FastDFS

java程序连接FastDFS，就需要用到fastdfs-client-java客户端。

[maven仓库fastdfs-client-java](<https://search.maven.org/search?q=fastdfs-client-java>)

![image-20200925134655412](https://i-blog.csdnimg.cn/blog_migrate/4c6833b0c6eee1ca8b8c5526fbf2bf0c.png)

随便挑一个加入依赖，当然，有些集成了starter.

[maven仓库fastdfs](<https://search.maven.org/search?q=fastdfs>)

![image-20200925134841508](https://i-blog.csdnimg.cn/blog_migrate/647914e0826e2c604bd58885f81b74d6.png)

然后将fastdfs-client-java包中的fdfs_client.conf.simple拷贝到resouce下(我演示的是spring boot web项目，gradler管理)，去掉.simple

可以先依赖fastdfs-client-java包，拿到了conf文件后，在依赖starter包。

![image-20200925135605774](https://i-blog.csdnimg.cn/blog_migrate/7e588f10b3794a3b662338e154f1cab5.png)

支持两种配置方式.conf和.properties。

主要还是配置tracker的ip和端口。

![image-20200925135744215](https://i-blog.csdnimg.cn/blog_migrate/2b45081063f7d644e42c9cac89edc359.png)

然后在resource目录下放一张图片，用于上传和下载。

### 6.1 上传
    
    
        @Test
        public void testUpload() {
            try {
                ClientGlobal.init("fdfs_client.conf");
                TrackerClient client = new TrackerClient();
                TrackerServer server = client.getConnection();
                StorageServer storageServer = null;
                StorageClient storageClient = new StorageClient(server, storageServer);
                ClassPathResource classPathResource = new ClassPathResource("test.jpg");
                String[] result = storageClient.upload_file("group1", classPathResource.getInputStream().readAllBytes(), "jpg", null);
                StringBuilder builder = new StringBuilder();
                builder.append("http://10.0.228.153:40080/")
                        .append(result[0])
                        .append("/")
                        .append(result[1]);
                System.out.println(builder.toString());
            } catch (IOException e) {
                e.printStackTrace();
            } catch (FastdfsException e) {
                e.printStackTrace();
            }
    
        }
    

![image-20200925140024832](https://i-blog.csdnimg.cn/blog_migrate/297d7b1abd12fecc6222eb3f2175a7c0.png)

我们也可以将这个图片上传到group0:

![image-20200925140100994](https://i-blog.csdnimg.cn/blog_migrate/2bb7a5b458da48a0dd3af35ce628ff9b.png)

什么情况，竟然异常了

![image-20200925140229787](https://i-blog.csdnimg.cn/blog_migrate/bdc6524975ac294ffb97b0e0cbbf7360.png)

根据前面我们查看的FastDFS系统的信息，应该是group0的storage离线了。

![image-20200925141340760](https://i-blog.csdnimg.cn/blog_migrate/98287d451a1f64bd328a8a8b5ed86f9d.png)

一般storage有7种状态

# FDFS_STORAGE_STATUS：INIT :初始化，尚未得到同步已有数据的源服务器

# FDFS_STORAGE_STATUS：WAIT_SYNC :等待同步，已得到同步已有数据的源服务器

# FDFS_STORAGE_STATUS：SYNCING :同步中

# FDFS_STORAGE_STATUS：DELETED :已删除，该服务器从本组中摘除

# FDFS_STORAGE_STATUS：OFFLINE :离线

# FDFS_STORAGE_STATUS：ONLINE :在线，尚不能提供服务

# FDFS_STORAGE_STATUS：ACTIVE :在线，可以提供服务

首先使用fdfs_monitor删除storage,然后清除全部数据，重新加入。

我猜测是tracker和storage是不同的网段，中间一段时间，tracker和storage无法通信，超过默认的时间了。

网络问题，暂时不深究了。

为什么group1没有问题？

一因为group1是tracker下的storage，相当于storage和tracker在同一个服务器上。

### 6.2 下载

还记得前面上传返回的地址吗，`http://10.0.228.153:40080/group1/M00/00/00/CgDkmV9tjpWAGxfmAAhzLHTDQrc462.jpg`

这个就是浏览器访问的地址，程序访问需要去掉group1前面的。`M00/00/00/CgDkmV9tjpWAGxfmAAhzLHTDQrc462.jpg`这是程序访问的地址
    
    
        @Test
        public void testDownload() {
            try {
                ClientGlobal.init("fdfs_client.conf");
                TrackerClient client = new TrackerClient();
                TrackerServer server = client.getConnection();
                StorageServer storageServer = null;
                StorageClient storageClient = new StorageClient(server, storageServer);
                int group1 = storageClient.download_file("group1", "M00/00/00/CgDkmV9tjpWAGxfmAAhzLHTDQrc462.jpg", "download.jpg");
                System.out.println(group1);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (FastdfsException e) {
                e.printStackTrace();
            }
        }
    

![image-20200925144952535](https://i-blog.csdnimg.cn/blog_migrate/6b7a4b9c701b16065c3c89054815ce91.png)

![image-20200925145001632](https://i-blog.csdnimg.cn/blog_migrate/82b6ad09eb3beb9e364e2e13c4c15e5c.png)

返回0表示成功。

### 6.3 删除

删除的地址和下载的地址相同，api也差不多。
    
    
        @Test
        public void testDeleteFile() {
            try {
                ClientGlobal.init("fdfs_client.conf");
                TrackerClient client = new TrackerClient();
                TrackerServer server = client.getConnection();
                StorageServer storageServer = null;
                StorageClient storageClient = new StorageClient(server, storageServer);
                int delete_file = storageClient.delete_file("group1", "M00/00/00/CgDkmV9tjpWAGxfmAAhzLHTDQrc462.jpg");
                System.out.println(delete_file);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (FastdfsException e) {
                e.printStackTrace();
            }
        }
    

![image-20200925145242802](https://i-blog.csdnimg.cn/blog_migrate/a7674cb5bdff77405656e5386b898163.png)

返回0表示成功。

然后在访问就404了

![image-20200925145335048](https://i-blog.csdnimg.cn/blog_migrate/5f3e7128a1555c6571cd4cf422d48b6f.png)

## 7\. 总结

通过这个项目，首先我了解了目前市面上的，关于文件服务器的解决方案，基本上了解了这些解决方案的优缺点。通过对比这些解决方案的优缺点，挑选出最适合的解决方案，FastDFS。

通过这个项目，基本上了解了FastDFS的实现原理，设计思想，安装实现以及如何操作。也基本上全面了解了FastDFS的配置指标以及如何配置优化。

通过试验，结合目前已经实现的docker镜像，在已有的docker镜像的基础上，进行了扩展，实现了我们需要的可扩展的FastDFS。

通过练习，学会了如何将FastDFS集成到项目中，进行上传、下载、删除等操作。

通过项目，更了解到，我们不能闭门造车，要学会站在巨人的肩膀上。
