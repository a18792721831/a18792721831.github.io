---
layout: post
title: "打包服务器docker化与jenkins集成"
date: 2020-01-04 18:09:34 +0800
categories: [docker, centos, dockerfile, 自定义dockerimage, docker+jenkins]
description: "本文详细记录了从CentOS 6.1搭建基础镜像开始，通过离线安装方式完成必要软件包的安装，逐步实现服务器的Docker化，并最终集成Jenkins的过程。涵盖软件下载、镜像构建、环境配置、Jenkins安装等关键步骤。"
keywords: docker, centos, dockerfile, 自定义dockerimage, docker+jenkins
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/103786656
> - 发布时间：2020-01-04 18:09:34
> - 阅读量：764
> - 分类：docker同时被 2 个专栏收录, 订阅专栏, jenkins
> - 标签：#docker, #centos, #dockerfile, #自定义dockerimage, #docker+jenkins

## 摘要

文章浏览阅读764次。本文详细记录了从CentOS 6.1搭建基础镜像开始，通过离线安装方式完成必要软件包的安装，逐步实现服务器的Docker化，并最终集成Jenkins的过程。涵盖软件下载、镜像构建、环境配置、Jenkins安装等关键步骤。

---

#### 打包服务器docker化

  * 1.基础镜像
  *     * 1.1安装系统
    * 1.2打包系统
    * 1.3导入镜像
  * 2.下载软件
  *     * 2.1准备
    * 2.2 设置yum源
    * 2.3 libxml2
    * 2.4 libxslt-devel
    * 2.5 net-snmp
    * 2.6 net-snmp-utils
    * 2.7 net-snmp-devel
    * 2.8 dos2unix
    * 2.9 zlib-devel
    * 2.10 libxml2
    * 2.11 gcc
    * 2.12 gcc-c++
    * 2.13 automake
    * 2.14 make
    * 2.15 libtool
    * 2.16 byacc
    * 2.17 bison
    * 2.18 flex
    * 2.19 zlib
  * 3.安装软件
  * 4.安装oracle
  * 5\. env
  * 6\. gsoap 2.7.7
  * 7\. java
  * 8\. jenkins
  * 9\. dockerfile合并
  *     * 9.1 6->5
    * 9.2 5->4
    * 9.3 4->3
    * 9.4 3->2
    * 9.5 2->1
  * 10\. 总结

## 1.基础镜像

### 1.1安装系统

首先将CentOS-6.1-x86_64-bin-DVD1安装至虚拟机  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/97873c06cc08e88bd76b20f3c2d442b8.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d4f560f6ac15148ade38cf90074a0baf.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d8152cec004a28924273bc1d1f906f6a.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3eb10971497badaa9ca07a9b7290d2b6.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/037c452ce4d10d28fcf4e6b755de3df4.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/42f477a56073511e5adc4fbb85aae7e6.png)  
选择最小安装。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7fd38e121530d5cec151e68438a222c4.png)  
等待完成即可。

### 1.2打包系统

首先设置linux网络，可以使用ssh连接centos6-64-base
    
    
    vi /etc/sysconfig/network-scripts/ifcfg-eth0
    

保存，使用
    
    
    service network restart
    

重启网络  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7c73ea9787a988fd1961869a381ab37e.png)  
然后查看IP，并使用ssh连接  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/40180f3e7eca51d3e52b53f3bbd5a427.png)

在ssh中使用如下命令打包系统
    
    
    tar -cvpf /tmp/base.tar --directory=/ --exclude=proc --exclude=sys --exclude=dev --exclude=ru
    n --exclude=boot /
    

说明，/tmp/base.tar是打包后的文件存放的目录与名称；  
–directory是打包的路劲  
–exclude是忽略的目录  
/是将根目录作为上下文传入  
执行完成后会生成  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6dc36eb50bfa4eb835ad6fd7b99153f4.png)  
base.tar文件。  
然后将这个base.tar拷贝出来，准备工作就完成了。  
（虚拟机就没有用了，可以关闭了，或者删除掉）

### 1.3导入镜像

既然是要做docker镜像的，所以需要把1.2中的base.tar放到装有docker的Linux中。  
然后执行
    
    
    docker import /tmp/base.tar centos6-64:base
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a10b6dd844a6face5587a036f65edb71.png)

## 2.下载软件

### 2.1准备

在之前安装的虚拟机中安装yum-plugin-downloadonly
    
    
    yum install yum-plugin-downloadonly -y
    

然后对整个yum进行更新
    
    
    yum clean all && yum update -y
    

在update时会出现异常：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8113051378cab875e438ce12f16197e4.png)  
使用
    
    
    yum -y remove matahari*
    

去除依赖  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6de51c5aceb8789a711384ecd188811a.png)  
然后在更新（时间比较长，等待完成即可）

### 2.2 设置yum源

因为官方的yum源在国外，且速度比较慢，所以使用阿里和163的yum源。  
首先下载wget
    
    
    yum install wget -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/53ceec2986f207d16a4b3dbcded54514.png)  
下载阿里和163的yum源
    
    
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
    

（阿里和163的yum源随便一个即可）  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d177113a2d810da03f817d76ab49edb8.png)  
然后执行：
    
    
    yum clean all && yum makecache && yum update -y
    

### 2.3 libxml2

<http://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/libxml2-2.7.6-21.el6_8.1.x86_64.rpm>

### 2.4 libxslt-devel
    
    
    yum install libxslt-devel --downloadonly --downloaddir=/tmp libxslt-devel -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c933f58b050a618b97ebf8644421703a.png)

### 2.5 net-snmp
    
    
    yum install net-snmp --downloadonly --downloaddir=/tmp net-snmp -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d7bd76a0bb7d38dec05beb6fe683b900.png)

### 2.6 net-snmp-utils
    
    
    yum install net-snmp-utils --downloadonly --downloaddir=/tmp net-snmp-utils -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8b4c27cb0e93cc64eb8c94d3daff83e5.png)

### 2.7 net-snmp-devel
    
    
    yum install net-snmp-devel --downloadonly --downloaddir=/tmp net-snmp-devel -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2713ea8a713dc71d54adf6783dc32598.png)

### 2.8 dos2unix
    
    
    yum install dos2unix --downloadonly --downloaddir=/tmp dos2unix -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e5a89b471aa58a8f9430e020985334a8.png)

### 2.9 zlib-devel
    
    
    yum install zlib-devel --downloadonly --downloaddir=/tmp zlib-devel -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c08e4916ca4f8707e1b9675dc482b292.png)

### 2.10 libxml2

这个需要手动下载，然后放在其他文件一起。  
下载地址：  
<http://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/libxml2-2.7.6-21.el6_8.1.x86_64.rpm>

### 2.11 gcc
    
    
    yum install gcc --downloadonly --downloaddir=/tmp gcc -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/15c06d7a0f73ca7de09bfcbad12c1879.png)

### 2.12 gcc-c++
    
    
    yum install gcc-c++ --downloadonly --downloaddir=/tmp gcc-c++ -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c40aabdb1c466bd8a92df02f0fe944c5.png)

### 2.13 automake
    
    
    yum install automake --downloadonly --downloaddir=/tmp automake -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a1bd587ce7275d9ed17e78a292cc7270.png)

### 2.14 make

手动下载，地址  
<http://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/make-3.81-23.el6.x86_64.rpm>

### 2.15 libtool
    
    
    yum install libtool --downloadonly --downloaddir=/tmp libtool -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7b740c4d3f33453cf45c3d7c9d1ffd8e.png)

### 2.16 byacc
    
    
    yum install byacc --downloadonly --downloaddir=/tmp byacc -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/574ab1755dc6ef41bc19a04e7fdd059d.png)

### 2.17 bison
    
    
    yum install bison --downloadonly --downloaddir=/root/tmp/ bison -y
    

### 2.18 flex
    
    
    yum install flex --downloadonly --downloaddir=/root/tmp/ flex -y
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c2d3b41bd4faf54f87edbd3aa8830591.png)

### 2.19 zlib

<http://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/zlib-1.2.3-29.el6.x86_64.rpm>

总计需要安装71个软件，其具体的软件名称与版本如下：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fc3e07030d309f2cd675a6e2e8e70845.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f289340c201e8f053b2bacc71fce86b6.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4c2030e54a246715c59466b0e48d871a.png)

## 3.安装软件

将2下载的文件放到docker服务器中。  
在docker服务器的rpm文件夹同级目录中创建文件dockerfile  
(注意：必须是rpm文件夹的同级目录，因为docker build需要使用rpm同级目录作为构建上下文，同时不建议将rpm放在根目录下，如果放在根目录下表示将docker )  
然后编写dockerfile文件：
    
    
    #based as centos6-64:base os
    FROM centos6-64:base
    # auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    # user root
    USER root
    # copy rpm files
    COPY /rpm /rpm
    # install rpm
    RUN find /rpm -type f|xargs rpm -i --force --nodeps
    # clean rpm
    RUN find /rpm -type f|xargs rm -rf
    # /bin/bash cmd
    CMD ["/bin/bash"]
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/90abd0a4bc62641f42f79b74bb88ee21.png)

FROM 表示将centos6-64:base作为本次构建的基础镜像。  
MAINTAINER表示作者以及邮箱。  
COPY表示将我们上传的rpm文件夹以及里面的文件全部拷贝到容器中的rpm目录中  
RUN表示执行shell命令。  
shell命令分为两部分，第一部分是遍历rpm文件夹中所有的安装包  
第二部分是使用rpm -i命令安装遍历得到的所有软件包，忽略检测，强制安装。  
CMD 表示执行cmd命令，执行/bin/bash命令，否则启动异常，无法启动容器进行验证。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/02f563cd115fcacd9c93375cf215de59.png)

## 4.安装oracle

基于3生成的镜像编写新的dockerfile：
    
    
    #based as centos6-64:base os
    FROM centos6-64:v0
    #auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    #create oracle
    RUN mkdir /opt/oracle
    #COPY oracle files
    COPY /file/oracle /opt/oracle
    #change jur
    RUN chmod -R 755 /opt/oracle
    # /bin/bash cmd
    CMD ["/bin/bash"]
    

然后进行构建  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cbac34955f0fdbb9993c3d0eac8496c0.png)

## 5\. env

在4的基础镜像上，编写如下dockerfile:
    
    
    #based as centos6-64:v0 os
    FROM centos6-64:v1
    
    #auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    
    # set env
    ENV alias vi=vim
    ENV PATH=$PATH:$HOME/bin
    
    # SYSTEM ENV
    ENV LANG=en_US
    ENV LC_ALL=en_US
    ENV EDITOR=vi
    ENV MANPATH=$MANPATH:/usr/share/man:/usr/share/locale/man
    ENV LD_LIBRARY_PATH=/usr/lib:/usr/dt/lib:/usr/openwin/lib:/usr/sfw/lib:/usr/local/lib:/usr/local/ssl/lib:/usr/local/apr/lib:.
    ENV PATH=/bin:/sbin:/usr/bin:/usr/sbin:/etc:/usr/local/bin:/usr/local/sbin:/usr/ccs/bin:/usr/ucb:/usr/sfw/bin:.
    ENV CC=gcc
    
    # JAVA ENV
    #ENV JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-1.el6_10.x86_64/jre
    #ENV CLASSPATH=.:$JAVA_HOME/lib/rt.jar
    #ENV PATH=$PATH:$JAVA_HOME/bin
    
    # ORACLE ENV
    ENV ORACLE_BASE=/opt/oracle
    ENV ORACLE_HOME=$ORACLE_BASE/instantclient_11_2
    ENV ORACLE_SID=starboss
    ENV PATH=$ORACLE_HOME/sdk:$ORACLE_HOME:$PATH
    ENV LD_LIBRARY_PATH=$ORACLE_HOME/:/lib64:/usr/lib64
    ENV CLASSPATH=$CLASSPATH:$ORACLE_HOME/jre:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
    ENV NLS_LANG=AMERICAN_AMERICA.ZHS16GBK
    
    # LIBXML2 ENV
    ENV LIBXML2_HOME=/usr
    ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
    ENV PATH=$PATH:$ORACLE_HOME/lib:$ORACLE_HOME/bin:$PATH:$LIBXML2_HOME/bin
    
    #gcc head file and lib path
    ENV LOCAL_LIB_PATH=/usr/local/lib
    ENV LIBRARY_PATH=$LIBXML2_HOME/lib:$ORACLE_HOME
    ENV CPLUS_INCLUDE_PATH=$ORACLE_HOME/sdk/include:$ORACLE_HOME/rdbms/public:$LIBXML2_HOME/include/libxml2
    ENV C_INCLUDE_PATH=$LIBXML2_HOME/include/libxml2
    
    ENV ulimit -S -c unlimited
    ENV OS=LINUX
    
    # /bin/bash cmd
    CMD ["/bin/bash"]
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1197878f4221b7d978c3546d53509f3d.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5f41855d4deeaf5b60259910168fdd78.png)

## 6\. gsoap 2.7.7

在6的基础上进行构建，dockerfile如下：
    
    
    #based as centos6-64:v1 os
    FROM centos6-64:v2
    #auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    #use add command to copy and unzip gsoap file
    ADD /file/gsoap_2.7.7.tar.gz /root/
    #change jur
    RUN chmod +x /root/gsoap-2.7
    #change workdir
    WORKDIR /root/gsoap-2.7
    #configure
    RUN ./configure
    #make
    RUN make
    #make install
    RUN make install
    # /bin/bash cmd
    CMD ["/bin/bash"]
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/de8f91c6e7254bedd7516482457b6a46.png)

## 7\. java
    
    
    #based as centos6-64:base os
    FROM centos6-64:v3
    #auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    #add jdk files
    ADD /file/jdk-8u111-linux-x64.tar.gz /
    # set env
    ENV JAVA_HOME=/jdk1.8.0_111
    ENV CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ENV PATH=$PATH:$JAVA_HOME/bin
    # /bin/bash cmd
    CMD ["/bin/bash"]
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9074bd44cc52f06d857e787d92fd736b.png)

## 8\. jenkins

首先去jenkins的官网下载jenkins的war包  
<http://ftp-chi.osuosl.org/pub/jenkins/war-stable/2.204.1/jenkins.war>  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/17747c42b582cefd2b746958ac593141.png)  
将下载的jenkins.war包放到docker 服务器中  
然后编写dockerfile:
    
    
    #based as centos6-64:v1 os
    FROM centos6-64:v4
    #auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    # mkdir /tmp,jenkins will be used /tmp file
    RUN mkdir /tmp
    # use copy command to copy and unzip gsoap file
    COPY /file/jenkins.war /
    # use 8080 port
    EXPOSE 8080
    # start jenkins 
    ENTRYPOINT ["java","-jar","/jenkins.war","--httpPort=8080"]
    # workdir /root
    WORKDIR /root
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ebf5b90fa9066bf8072a2bfbdc9e2498.png)  
启动jenkins镜像，验证jenkins是否安装成功  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1693791e33be82b2d158165e4dddd9c2.png)  
然后访问  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/311a6e2760469784c5e42947051be579.png)  
正常不是这样的，应该是需要去密码文件，以管理员登录，然后下载插件，创建用户等等。

我这个是之前启动的jenkins时，保留的/root/.jenkins文件夹，这样不用每次重启镜像都需要重新下载插件了。  
但是如果两个jenkins的版本不一样，即使保留了jenkins文件夹，仍然需要重新下载插件的。  
到此，整个环境已经搞定。

## 9\. dockerfile合并

之前每一小步骤都分为1个dockerfile是因为如果一个小步骤出现问题，不太容易差错。现在可以证明每一个小步骤都是ok的，因此，将之前的dockerfile合并为1个。  
采用向前合并。  
现在我们有1,2,3,4,5,6个dockerfile

### 9.1 6->5

首先将6与5进行合并
    
    
    # based as centos6-64:base os
    FROM centos6-64:v3
    # auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    # mkdir /tmp, jenkins will be used /tmp file
    RUN mkdir /tmp
    # add jdk files
    ADD /file/jdk-8u111-linux-x64.tar.gz /
    # copy jenkins.war
    COPY /file/jenkins.war /
    # set env
    ENV JAVA_HOME=/jdk1.8.0_111
    ENV CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ENV PATH=$PATH:$JAVA_HOME/bin
    # use 8080 port
    EXPOSE 8080
    # start jenkins 
    ENTRYPOINT ["java","-jar","/jenkins.war","--httpPort=8080"]
    # workdir /root
    WORKDIR /root
    

然后重新构建，并启动测试验证  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/319e2597e2ae9ee11450d786c7373b5e.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6d4d19d8a5d94de6f12fb45c05c842eb.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3acf2a5b59cae5664b08d07455fefb37.png)

### 9.2 5->4
    
    
    #based as centos6-64:v1 os
    FROM centos6-64:v2
    
    #auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    
    #use add command to copy and unzip gsoap file
    ADD /file/gsoap_2.7.7.tar.gz /root/
    # add jdk files
    ADD /file/jdk-8u111-linux-x64.tar.gz /
    
    # copy jenkins.war
    COPY /file/jenkins.war /
    
    # set env
    ENV JAVA_HOME=/jdk1.8.0_111
    ENV CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ENV PATH=$PATH:$JAVA_HOME/bin
    
    #change workdir
    WORKDIR /root/gsoap-2.7
    
    # mkdir /tmp, jenkins will be used /tmp file
    RUN mkdir /tmp
    #change jur
    RUN chmod +x /root/gsoap-2.7
    #configure
    RUN ./configure
    #make
    RUN make
    #make install
    RUN make install
    
    # use 8080 port
    EXPOSE 8080
    # start jenkins 
    ENTRYPOINT ["java","-jar","/jenkins.war","--httpPort=8080"]
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f2523a4eb4fee60acb5f5c9ebd8626d5.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/29c97f361a7c590114066299eb631efe.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/140f5a13f7b970e3124e5351fd1576f5.png)

### 9.3 4->3
    
    
    #based as centos6-64:v0 os
    FROM centos6-64:v1
    
    #auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    
    #use add command to copy and unzip gsoap file
    ADD /file/gsoap_2.7.7.tar.gz /root/
    # add jdk files
    ADD /file/jdk-8u111-linux-x64.tar.gz /
    
    # copy jenkins.war
    COPY /file/jenkins.war /
    
    # set env
    ENV alias vi=vim
    ENV PATH=$PATH:$HOME/bin
    # SYSTEM ENV
    ENV LANG=en_US
    ENV LC_ALL=en_US
    ENV EDITOR=vi
    ENV MANPATH=$MANPATH:/usr/share/man:/usr/share/locale/man
    ENV LD_LIBRARY_PATH=/usr/lib:/usr/dt/lib:/usr/openwin/lib:/usr/sfw/lib:/usr/local/lib:/usr/local/ssl/lib:/usr/local/apr/lib:.
    ENV PATH=/bin:/sbin:/usr/bin:/usr/sbin:/etc:/usr/local/bin:/usr/local/sbin:/usr/ccs/bin:/usr/ucb:/usr/sfw/bin:.
    ENV CC=gcc
    # ORACLE ENV
    ENV ORACLE_BASE=/opt/oracle
    ENV ORACLE_HOME=$ORACLE_BASE/instantclient_11_2
    ENV ORACLE_SID=starboss
    ENV PATH=$ORACLE_HOME/sdk:$ORACLE_HOME:$PATH
    ENV LD_LIBRARY_PATH=$ORACLE_HOME/:/lib64:/usr/lib64
    ENV CLASSPATH=$CLASSPATH:$ORACLE_HOME/jre:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
    ENV NLS_LANG=AMERICAN_AMERICA.ZHS16GBK
    # LIBXML2 ENV
    ENV LIBXML2_HOME=/usr
    ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
    ENV PATH=$PATH:$ORACLE_HOME/lib:$ORACLE_HOME/bin:$PATH:$LIBXML2_HOME/bin
    #gcc head file and lib path
    ENV LOCAL_LIB_PATH=/usr/local/lib
    ENV LIBRARY_PATH=$LIBXML2_HOME/lib:$ORACLE_HOME
    ENV CPLUS_INCLUDE_PATH=$ORACLE_HOME/sdk/include:$ORACLE_HOME/rdbms/public:$LIBXML2_HOME/include/libxml2
    ENV C_INCLUDE_PATH=$LIBXML2_HOME/include/libxml2
    ENV ulimit -S -c unlimited
    ENV OS=LINUX
    # set env
    ENV JAVA_HOME=/jdk1.8.0_111
    ENV CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ENV PATH=$PATH:$JAVA_HOME/bin
    
    #change workdir
    WORKDIR /root/gsoap-2.7
    
    # mkdir /tmp, jenkins will be used /tmp file
    RUN mkdir /tmp
    #change jur
    RUN chmod +x /root/gsoap-2.7
    #configure
    RUN ./configure
    #make
    RUN make
    #make install
    RUN make install
    
    # use 8080 port
    EXPOSE 8080
    # start jenkins
    ENTRYPOINT ["java","-jar","/jenkins.war","--httpPort=8080"]
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3d62cae58e9d04830a6560453effd647.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9a56416b87c9f233cba7768a04297343.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bc2d6f10e4310a4fb1c3808ba507ab77.png)

### 9.4 3->2
    
    
    #based as centos6-64:v0 os
    FROM centos6-64:v0
    
    #auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    
    #use add command to copy and unzip gsoap file
    ADD /file/gsoap_2.7.7.tar.gz /root/
    # add jdk files
    ADD /file/jdk-8u111-linux-x64.tar.gz /
    
    #create oracle
    RUN mkdir /opt/oracle
    # mkdir /tmp, jenkins will be used /tmp file
    RUN mkdir /tmp
    
    # copy jenkins.war
    COPY /file/jenkins.war /
    #COPY oracle files
    COPY /file/oracle /opt/oracle
    
    # set env
    ENV alias vi=vim
    ENV PATH=$PATH:$HOME/bin
    # SYSTEM ENV
    ENV LANG=en_US
    ENV LC_ALL=en_US
    ENV EDITOR=vi
    ENV MANPATH=$MANPATH:/usr/share/man:/usr/share/locale/man
    ENV LD_LIBRARY_PATH=/usr/lib:/usr/dt/lib:/usr/openwin/lib:/usr/sfw/lib:/usr/local/lib:/usr/local/ssl/lib:/usr/local/apr/lib:.
    ENV PATH=/bin:/sbin:/usr/bin:/usr/sbin:/etc:/usr/local/bin:/usr/local/sbin:/usr/ccs/bin:/usr/ucb:/usr/sfw/bin:.
    ENV CC=gcc
    # ORACLE ENV
    ENV ORACLE_BASE=/opt/oracle
    ENV ORACLE_HOME=$ORACLE_BASE/instantclient_11_2
    ENV ORACLE_SID=starboss
    ENV PATH=$ORACLE_HOME/sdk:$ORACLE_HOME:$PATH
    ENV LD_LIBRARY_PATH=$ORACLE_HOME/:/lib64:/usr/lib64
    ENV CLASSPATH=$CLASSPATH:$ORACLE_HOME/jre:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
    ENV NLS_LANG=AMERICAN_AMERICA.ZHS16GBK
    # LIBXML2 ENV
    ENV LIBXML2_HOME=/usr
    ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
    ENV PATH=$PATH:$ORACLE_HOME/lib:$ORACLE_HOME/bin:$PATH:$LIBXML2_HOME/bin
    #gcc head file and lib path
    ENV LOCAL_LIB_PATH=/usr/local/lib
    ENV LIBRARY_PATH=$LIBXML2_HOME/lib:$ORACLE_HOME
    ENV CPLUS_INCLUDE_PATH=$ORACLE_HOME/sdk/include:$ORACLE_HOME/rdbms/public:$LIBXML2_HOME/include/libxml2
    ENV C_INCLUDE_PATH=$LIBXML2_HOME/include/libxml2
    ENV ulimit -S -c unlimited
    ENV OS=LINUX
    # set env
    ENV JAVA_HOME=/jdk1.8.0_111
    ENV CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ENV PATH=$PATH:$JAVA_HOME/bin
    
    #change workdir
    WORKDIR /root/gsoap-2.7
    
    #change jur
    RUN chmod +x /root/gsoap-2.7
    #configure
    RUN ./configure
    #make
    RUN make
    #make install
    RUN make install
    #change jur
    RUN chmod -R 755 /opt/oracle
    
    # use 8080 port
    EXPOSE 8080
    # start jenkins
    ENTRYPOINT ["java","-jar","/jenkins.war","--httpPort=8080"]
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/14f72aa991a474f23f0100bf3e2217a6.png)  
(这里镜像tag写错了，应该是centos6-64:v1,不是centos6-64:v0)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c0d7893a4f9da5ab2d021f7515f67360.png)  
这是刷新后的图。。。。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7d58bf66c4cb0fea9b99de7fa12dfe23.png)

### 9.5 2->1
    
    
    #based as centos6-64:base os
    FROM centos6-64:base
    
    #auth is jiayq jiayq@startimes.com.cn
    MAINTAINER jiayq <jiayq@startimes.com.cn>
    
    # user root
    USER root
    
    #use add command to copy and unzip gsoap file
    ADD /file/gsoap_2.7.7.tar.gz /root/
    # add jdk files
    ADD /file/jdk-8u111-linux-x64.tar.gz /
    
    # copy jenkins.war
    COPY /file/jenkins.war /
    #COPY oracle files
    COPY /file/oracle /opt/oracle
    # copy rpm files
    COPY /rpm /rpm
    
    # set env
    ENV alias vi=vim
    ENV PATH=$PATH:$HOME/bin
    # SYSTEM ENV
    ENV LANG=en_US
    ENV LC_ALL=en_US
    ENV EDITOR=vi
    ENV MANPATH=$MANPATH:/usr/share/man:/usr/share/locale/man
    ENV LD_LIBRARY_PATH=/usr/lib:/usr/dt/lib:/usr/openwin/lib:/usr/sfw/lib:/usr/local/lib:/usr/local/ssl/lib:/usr/local/apr/lib:.
    ENV PATH=/bin:/sbin:/usr/bin:/usr/sbin:/etc:/usr/local/bin:/usr/local/sbin:/usr/ccs/bin:/usr/ucb:/usr/sfw/bin:.
    ENV CC=gcc
    # ORACLE ENV
    ENV ORACLE_BASE=/opt/oracle
    ENV ORACLE_HOME=$ORACLE_BASE/instantclient_11_2
    ENV ORACLE_SID=starboss
    ENV PATH=$ORACLE_HOME/sdk:$ORACLE_HOME:$PATH
    ENV LD_LIBRARY_PATH=$ORACLE_HOME/:/lib64:/usr/lib64
    ENV CLASSPATH=$CLASSPATH:$ORACLE_HOME/jre:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
    ENV NLS_LANG=AMERICAN_AMERICA.ZHS16GBK
    # LIBXML2 ENV
    ENV LIBXML2_HOME=/usr
    ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
    ENV PATH=$PATH:$ORACLE_HOME/lib:$ORACLE_HOME/bin:$PATH:$LIBXML2_HOME/bin
    #gcc head file and lib path
    ENV LOCAL_LIB_PATH=/usr/local/lib
    ENV LIBRARY_PATH=$LIBXML2_HOME/lib:$ORACLE_HOME
    ENV CPLUS_INCLUDE_PATH=$ORACLE_HOME/sdk/include:$ORACLE_HOME/rdbms/public:$LIBXML2_HOME/include/libxml2
    ENV C_INCLUDE_PATH=$LIBXML2_HOME/include/libxml2
    ENV ulimit -S -c unlimited
    ENV OS=LINUX
    # set env
    ENV JAVA_HOME=/jdk1.8.0_111
    ENV CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ENV PATH=$PATH:$JAVA_HOME/bin
    
    #change workdir
    WORKDIR /root/gsoap-2.7
    
    # install rpm
    RUN find /rpm -type f|xargs rpm -i --force --nodeps
    #change jur
    RUN chmod +x /root/gsoap-2.7
    #configure
    RUN ./configure
    #make
    RUN make
    #make install
    RUN make install
    #change jur
    RUN chmod -R 755 /opt/oracle
    # mkdir /tmp, jenkins will be used /tmp file
    RUN mkdir /tmp
    
    # use 8080 port
    EXPOSE 8080
    # start jenkins
    ENTRYPOINT ["java","-jar","/jenkins.war","--httpPort=8080"]
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e1b619d34db8118c249086ff6ba88e05.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e97d5afd4400b9b701c5a4f9a4f45485.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2eb00c39d516e84d2cb4574c69570b1d.png)  
至此，就完成了docker化与Jenkins集成。

## 10\. 总结

  * 1.打包服务器的操作系统有限制，必须使用centos6.1，但是因为centos6.1比较老旧，网上几乎找不到现有的centos6.1的基础镜像。所以只能自行创建基础镜像。
  * 2.docker build 时，因为没有涉及到网络（考虑到后续如果生产环境升级docker，那么生产机器有网络隔离，提前试水），所以，安装一些必要的软件就无法使用yum去直接安装。只能下载对应的rpm包，通过离线安装完成所有软件及依赖软件的安装。
  * 3.因为centos6.1的操作系统是在是太老旧，导致centos6.1通过yum下载的git最高版本是1.7.1,因为git太旧，导致无法克隆代码。所以需要安装ius第三方yum源，通过第三方yum源得到git2.16的rpm离线安装包，然后安装到镜像中。
  * 4.在获取离线安装包时，使用yum的一个工具download插件，但是有一些软件是操作系统自带，但是打tar包时，为打进去，导致有些依赖的软件无法通过yum和插件进行下载。只能通过网络中的rpm搜索站，手动下载。
  * 5.rpm离线安装需要联网进行检查与验证，但是镜像构建无法使用网络，所以使用rpm的–force --nodeps参数，强制安装，跳过检测验证。
  * 6.gsoap只能使用2.7.7，但是现在网上没有2.7.7的离线安装包，只能使用源码安装，使用源码安装，需要下载安装时的依赖库，而一些依赖库无法使用yum下载的。所以只能一次一次的试，然后一点一点的加进去。
  * 7.openjdk原来使用离线安装包进行安装，但是在构建过程中安装完成后，存在环境变量未设置，以及安装的是jre，等等原因，最后使用jdk1.8u111的linux包作为java安装。
  * 8.git安装成功后，进行clone测试，发现git有运行时依赖，所以只能推倒全部重新来。。（很恶心）
  * 9.安装8依赖的软件后，问题依然存在，但是可以从内网下载代码（外网不行，因为没有配置解析）。
  * 10.jenkins启动成功，才能说明本次集成成功，但是有好些参数，在最后一步才能发现问题，发现问题，意味着之前做的工作全部需要重新来过。
  * 11.dockerfile合并，dockerfile不能太长，对于命令的编排，命令的合并，都需要不断的优化。

最后，不要灰心，即使这条路走不通，依然有其他的路可以到达。
