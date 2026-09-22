---
layout: post
title: "windows下搭建hadoop"
date: 2022-01-13 21:26:23 +0800
categories: [hadoop, windows, hdfs]
description: "windows下搭建hadoop下载环境变量windows 脚本替换配置windows权限启动单词统计实例下载首先去Apache Hadoop下载hadoop的安装包选择二进制文件即可选择国内镜像增加下载速度下载后解压到文件夹环境变量设置环境变量HADOOP_HOME然后把HADOOP_HOME加入Path中打开cmd，输入hadoop version验证windows 脚本替换到cdarlint/winutils: winutils.exe hadoop.dll and_windows下搭建hadoop"
keywords: hadoop, windows, hdfs
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/122483139
> - 发布时间：2022-01-13 21:26:23
> - 阅读量：3
> - 分类：大数据同时被 2 个专栏收录, 订阅专栏, hadoop
> - 标签：#hadoop, #windows, #hdfs

## 摘要

文章浏览阅读3k次，点赞2次，收藏23次。windows下搭建hadoop下载环境变量windows 脚本替换配置windows权限启动单词统计实例下载首先去Apache Hadoop下载hadoop的安装包选择二进制文件即可选择国内镜像增加下载速度下载后解压到文件夹环境变量设置环境变量HADOOP_HOME然后把HADOOP_HOME加入Path中打开cmd，输入hadoop version验证windows 脚本替换到cdarlint/winutils: winutils.exe hadoop.dll and_windows下搭建hadoop

---

#### windows下搭建hadoop

  * 下载
  * 环境变量
  * windows 脚本替换
  * 配置
  * windows权限
  * 启动
  * 单词统计实例

## 下载

首先去[Apache Hadoop](<https://hadoop.apache.org/>)下载hadoop的安装包

![image-20220113204924262](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6f37bcc98ac565e269d37943c2d9238b.png)

选择二进制文件即可

![image-20220113204948024](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/58a2cea193ced6bfe84e6d84d3406412.png)

选择国内镜像增加下载速度

![image-20220113205014208](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f89c616ef4e9d0fd732566120a4bf066.png)

下载后解压到文件夹

![image-20220113205041639](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f3fcb20a99a048bf3475959c044dcb9f.png)

## 环境变量

设置环境变量`HADOOP_HOME`

![image-20220113205555625](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c38a2878daf59041e879149c3e8c6234.png)

然后把`HADOOP_HOME`加入`Path`中

![image-20220113205210882](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3d1230eacd1eaccb44ae7c9bae25dbf3.png)

打开cmd，输入`hadoop version`验证

![image-20220113205621711](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/cd4f42ad74cbcde5289e1ed1498f4ac4.png)

## windows 脚本替换

到[cdarlint/winutils: winutils.exe hadoop.dll and hdfs.dll binaries for hadoop windows (github.com)](<https://github.com/cdarlint/winutils>)下载全部版本的脚本，并解压

![image-20220113205423153](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/41524cd1b6e5367457ffdff53459e88e.png)

需要注意的是，尽可能选择这里面有的hadoop版本。比如2.9.2版本。

![image-20220113205650139](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/96a8dfb6e8e037a2ed103a7f655ee663.png)

将里面的文件全部拷贝到hadoop下的bin目录中，并选择替换

![image-20220113205729556](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3b200b57cb949069bef75f032f441c04.png)

## 配置

首先在hadoop目录下创建一个临时文件夹，用于hadoop存储临时文件。

否则hadoop默认会在c盘创建临时文件夹，这时会因为权限的问题，导致无法创建，所以最好是在hadoop目录下创建临时文件夹，而且尽可能不要把hadoop放在C盘。

配置`core-site.xml`
    
    
    <configuration>
      <property>
        <name>fs.defaultFS</name>
    	<value>hdfs://localhost:9820</value>
      </property>
      <property>
        <name>hadoop.tmp.dir</name>
    	<value>/E:/hadoop/hadoop-2.9.2/tmp</value>
      </property>
    </configuration>
    

接着在hadoop目录下创建data文件夹，用于hadoop存储持久化数据，在data目录下创建namenode和datanode文件夹

配置`hdfs-site.xml`
    
    
    <configuration>
      <property>
        <name>dfs.replication</name>
    	<value>1</value>
      </property>
      <property>
        <name>dfs.namenode.name.dir</name>
    	<value>file:///E:/hadoop/hadoop-2.9.2/data/namenode</value>
      </property>
      <property>
        <name>dfs.datanode.data.dir</name>
    	<value>file:///E:/hadoop/hadoop-2.9.2/data/datanode</value>
      </property>
      <property>
        <name>dfs.namenode.http-address</name>
    	<value>http://localhost:9870</value>
      </property>
    </configuration>
    

配置`mapred-site.xml`
    
    
    <configuration>
      <property>
        <name>mapreduce.framework.name</name>
    	<value>yarn</value>
      </property>
    </configuration>
    

配置`yarn-site.xml`
    
    
    <configuration>
    
    <!-- Site specific YARN configuration properties -->
      <property>
        <name>yarn.nodemanager.aux-services</name>
    	<value>mapreduce_shuffle</value>
    	<description>Yarn Node Manager Aux Service</description>
      </property>
    </configuration>
    

注意上述文件如果在hadoop/etc/hadoop下无法找到的，请找到相同文件名的`.template`文件拷贝，并删除`.tamplate`后缀

## windows权限

此时直接启动，会出现创建符号的异常，这是因为在window系统中，只有管理员才能创建符号链接。

此时有两种解决方式，第一种使用管理员的cmd启动`hadoop/sbar/start-all.cmd`；第二种是给自己登录的用户赋予创建符号链接的权限。

第一种，使用管理员的cmd

![image-20220113210523890](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2b2d93cd906fe51fb86fbfd50e8dead9.png)

第二种，使用win+r打开运行，输入`gpedit.msc`，在【计算机配置】-【Windows设置】-【安全设置】-【本地策略】-【用户权限分配】-【创建符号链接】中加入自己登录的用户或用户组，然后重启系统生效。

## 启动

第一次启动需要初始化名字节点，初始化名字节点之前请确保`hadoop/data/namenode`文件夹为空，然后在cmd(管理员)中输入`hadoop namenode -format`请注意空格。

![image-20220113210918858](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0456d8eef1dfa64098b894a4b7c713fd.png)

这样就初始化成功了。

然后切换到hadoop所在的驱动器，并切换到hadoop目录下

![image-20220113211008038](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a30891ec986884e9450433b80954a376.png)

接着进入sbin目录，并启动`start-all.cmd`

![image-20220113211039013](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/34a14d52a3214316a6f7e52baae38042.png)

接着会启动4个cmd窗口

![image-20220113211119634](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/1762fa4d874eeafb51c1ab8cbf3a085a.png)

这样就启动成功了

接着在浏览器中访问`http://localhost:9870`验证

![image-20220113211206943](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e962137c5517a2e480574757cf3b526c.png)

## 单词统计实例

我们首先创建三个txt文件

![image-20220113211259649](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/2293437171235112406ee44c9f9c2d8c.png)

然后给文件里面随便写点东西，接着在hadoop的hdfs中创建`/input`目录(这里可以使用普通的cmd)

`hadoop fs -mkdir /input`

![image-20220113211421727](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c59129062a82fe65ada128cffe25262f.png)

接着把三个txt文件上传到hdfs中

`hadoop fs -put yourpath\1.txt /input`

`hadoop fs -put yourpath\2.txt /input`

`hadoop fs -put yourpath\2.txt /input`

![image-20220113211549291](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/257e3e4cdab7de61357b67d5519ab1bc.png)

然后查看

`hadoop fs -ls /input`

![image-20220113211706327](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7883703c1b3bfd013e9af008e6284f1d.png)

你也可以在浏览器中查看

![image-20220113211735868](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6116295dc059a082f64faad724e0a053.png)

![image-20220113211747207](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/28dda07b15b0a4a856e8a365437fd740.png)

![image-20220113211756078](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/590c5e5d57f6e811b646f0dca283b10c.png)

![image-20220113211805426](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b55fd8a6598ca5c90fa988f55ce20552.png)

接着调用单词统计的例子

![image-20220113211853264](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/31e9c2147e87450178f3d4618b55b0c8.png)

执行命令`hadoop jar yourPath\share\hadoop\mapreduce\hadoop-mapreduce-examples-2.9.2.jar wordcount /input /output`

就会开始执行

![image-20220113212240902](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/e6ad883e061c808741e791d2762b60a3.png)

等待一会执行完毕即可

![image-20220113212308137](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/fdebd850078a8e87749d167d241b3c26.png)

执行的结果会保存在hdfs的`/output`目录下

`hadoop fs -ls /output`

![image-20220113212353519](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/53773684c67d90d2ab6aedc1bcc009a0.png)

查看结果

`hadoop fs -cat /output/part-r-ooooo`

![image-20220113212540290](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a83f7474d13358331f22f83cca543c28.png)
