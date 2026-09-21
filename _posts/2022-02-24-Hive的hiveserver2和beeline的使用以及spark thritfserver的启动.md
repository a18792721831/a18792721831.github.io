---
layout: post
title: "Hive的hiveserver2和beeline的使用以及spark thritfserver的启动"
date: 2022-02-24 23:17:45 +0800
categories: [hive, spark, thriftserver, beeline, hiverserver2]
description: "本文详细介绍了HiveServer2的配置，包括最小和最大工作线程数、监听端口和绑定地址，并展示了如何使用beeline连接HiveServer2。此外，还探讨了SparkThriftServer的配置，以及它与HiveServer2和spark-sql的对比，强调了其在应用复用方面的优势。最后，展示了如何通过beeline连接SparkThriftServer，并给出了使用sparksql程序连接ThriftServer的示例。"
keywords: hive, spark, thriftserver, beeline, hiverserver2
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/123123282
> - 发布时间：2022-02-24 23:17:45
> - 阅读量：6
> - 分类：scala同时被 3 个专栏收录, 订阅专栏, 大数据, spark
> - 标签：#hive, #spark, #thriftserver, #beeline, #hiverserver2

## 摘要

文章浏览阅读6.5k次，点赞8次，收藏30次。本文详细介绍了HiveServer2的配置，包括最小和最大工作线程数、监听端口和绑定地址，并展示了如何使用beeline连接HiveServer2。此外，还探讨了SparkThriftServer的配置，以及它与HiveServer2和spark-sql的对比，强调了其在应用复用方面的优势。最后，展示了如何通过beeline连接SparkThriftServer，并给出了使用sparksql程序连接ThriftServer的示例。

---

#### Hive的hiveserver2和beeline的使用以及spark thritfserver的启动

  * Hive 的hiveserver2介绍
  * hiveserver2 的配置
  * beeline连接hiveserver2
  * 配置hiveserver2的界面
  * spark thriftserver的配置
  * beeline 连接spark thriftserver
  * thriftserver和spark-sql对比
  * spark sql 程序连接thriftserver

## Hive 的hiveserver2介绍

HiveServer2 (HS2) 是一项使客户端能够针对 Hive 执行查询的服务。 HiveServer2 是已弃用的 HiveServer1 的继任者。 HS2 支持多客户端并发和认证。它旨在为开放 API 客户端（如 JDBC 和 ODBC）提供更好的支持。 HS2 是作为复合服务运行的单个进程，其中包括基于 Thrift 的 Hive 服务（TCP 或 HTTP）和用于 Web UI 的 Jetty Web 服务器。

## hiveserver2 的配置

需要在`/hive/conf/hive-site.xml`中配置
    
    
    hive.server2.thrift.min.worker.threads – 最小工作线程,默认 5.
    hive.server2.thrift.max.worker.threads – 最大工作线程,默认 500.
    hive.server2.thrift.port – 监听端口, 默认 10000.
    hive.server2.thrift.bind.host – 绑定的地址.
    

比如
    
    
    <?xml version="1.0"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    
    <configuration>
      <!-- hive 的元数据存储路径，使用mysql存储 -->
      <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop01:3306/hive?createDatabaseIfNotExist=true</value>
      </property>
      <!-- 数据库驱动 -->
      <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.cj.jdbc.Driver</value>
      </property>
      <!-- 数据库用户名 -->
      <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
      </property>
      <!-- 数据库密码 -->
      <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
      </property>
      <!-- 是否进行版本校验 -->
      <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
      </property>
      <!-- 权限处理 -->
      <property>
        <name>hive.server2.enable.doAs</name>
        <value>false</value>
      </property>
      <!-- 最小工作线程，默认5 -->
      <property>
        <name>hive.server2.thrift.min.worker.threads</name>
        <value>2</value>
      </property>
      <!-- 最大工作线程，默认500 -->
      <property>
        <name>hive.server2.thrift.max.worker.threads</name>
        <value>5</value>
      </property>
      <!-- 绑定端口，默认10000 -->
      <property>
        <name>hive.server2.thrift.port</name>
        <value>10000</value>
      </property>
      <!-- 绑定地址，默认0.0.0.0 -->
      <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>0.0.0.0</value>
      </property>
    </configuration>
    

因为使用了mysql数据库存储hive的元数据，所以需要从网上下载mysql的驱动包

从[Maven Repository: mysql » mysql-connector-java (mvnrepository.com)](<https://mvnrepository.com/artifact/mysql/mysql-connector-java>)下载就行

8.0.23版本

下载地址https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.23/mysql-connector-java-8.0.23.jar

下载后拷贝到`/hive/lib`目录下

![image-20220224213358642](https://i-blog.csdnimg.cn/blog_migrate/cb5ca4ef56d22f7e1e04a0f3f2e11132.png)

接着启动mysql

![image-20220224213509561](https://i-blog.csdnimg.cn/blog_migrate/a5c18c4459d2d546b7b127d878796f49.png)

接着启动hdfs

![image-20220224213531387](https://i-blog.csdnimg.cn/blog_migrate/fa08eac1cfdb54e6e765485f78f245fe.png)

启动spark集群

![image-20220224213554183](https://i-blog.csdnimg.cn/blog_migrate/a917a5302e7e917b5f81a30bb5874537.png)

启动spark-history

![image-20220224213611856](https://i-blog.csdnimg.cn/blog_migrate/640c2a5bd3263b9e23cddb6374b76588.png)

启动hiveserver2

![image-20220224213909185](https://i-blog.csdnimg.cn/blog_migrate/03593f72eb6ee137862f1dbfaab8cdc0.png)

也可以使用`nohup ./hiveserver2 &` 在后台启动hiveserver2，将输出写入nohup文件

![image-20220224214015547](https://i-blog.csdnimg.cn/blog_migrate/4f95bcb350ba83616bc1e5a29cacedcd.png)

可以使用`jps -m`查看运行的hiveserver2进程

![image-20220224214114606](https://i-blog.csdnimg.cn/blog_migrate/5f19022920d6a9b23e5092e59f077b39.png)

## beeline连接hiveserver2

当启动hiveserver2之后，就可以使用beeline进行连接了。

hiveserver2是服务端，而beeline是hive自带的一个客户端，除了使用beeline，还可以使用代码连接hiveserver2服务端。

使用beeline连接需要指定连接url`jdbc:hive2://hadoop01:10000`

比如使用如下命令连接`beeline -u jdbc:hive2://hadoop01:10000 -n hadoop01`

其中`-u`指定连接url，`-n`指定客户端的用户名

![image-20220224214423733](https://i-blog.csdnimg.cn/blog_migrate/17d3b2679bf4e2fb7a778afe625bbe81.png)

查询

![image-20220224214528376](https://i-blog.csdnimg.cn/blog_migrate/6c6b6dd89d3c0d8cfb143aa5191bfffa.png)

和使用mysql非常类似

输入`help`查看帮助

![image-20220224214617929](https://i-blog.csdnimg.cn/blog_migrate/8fd30e80d994a8ee7838f4b26138af1d.png)

可以看到beeline的命令都是以`!`开头的。

## 配置hiveserver2的界面

可以在`/hive/conf/hive-site.xml`中配置和webUI相关的配置
    
    
      <!-- hive 界面绑定地址 -->
      <property>
        <name>hive.server2.webui.host</name>
        <value>0.0.0.0</value>
      </property>
      <!-- hive 界面端口 -->
      <property>
        <name>hive.server2.webui.port</name>
        <value>8085</value>
      </property>
      <!-- hive 界面最大线程 -->
      <property>
        <name>hive.server2.webui.max.threads</name>
        <value>5</value>
      </property>
    

然后重新启动hiveserver2

需要先杀掉原来启动的hiveserver2，然后重新启动

![image-20220224215359703](https://i-blog.csdnimg.cn/blog_migrate/2d16adb839676b17e314a73ac2db2305.png)

接着就能访问hive的界面了

![image-20220224215424883](https://i-blog.csdnimg.cn/blog_migrate/e9ff8d7760c844f137a97824d905fb76.png)

启动一个beeline连接，使用用户名hadoop01

在启动beeline连接hiveserver2的时候，可以像jdbc的连接url一样，指定database

比如`beeline -u jdbc:hive2://hadoop01:10000/test_column -n hadoop01`

![image-20220224215622079](https://i-blog.csdnimg.cn/blog_migrate/d82a86f53aa5881f22c6a42d707fad82.png)

而且在连接url中指定database后，连接了就会自动切换到指定的database上的。

查看界面

![image-20220224215637171](https://i-blog.csdnimg.cn/blog_migrate/c57af275d576dff7575d30ace665992c.png)

还可以启动更多的beeline，在同一个机器上也可以的

![image-20220224215748225](https://i-blog.csdnimg.cn/blog_migrate/caec576303e415b253c2333bb184e853.png)

在同一个机器上启动两个beeline，使用`-n`参数区分

每个beeline上查询，然后查看界面

![image-20220224215932565](https://i-blog.csdnimg.cn/blog_migrate/36573cbe47562ad1a502ec7a8e3c5334.png)

## spark thriftserver的配置

使用spark-submit和spark-sql，每次执行一个sql，实际上都会产生一个spark application用于执行spark任务，如果执行的比较多的sql，就会频繁的创建spark application，这样就会在一定程度上耗费资源。为了实现spark application的复用，spark基本上把hive的hiveserver2照搬了过来，产生了 spark thriftserver，在使用thriftserver的时候，就能够复用spark application。

不过生产环境中每个查询因为数据量巨大，而且基本处于独立的查询操作，所以是不会使用thriftserver和hiveserver2的，这在开发中可能用的比较多。

简单点说，spark thriftserver和hiveserver2是一样的。

只需要把`/hive/conf/hive-site.xml`拷贝到`/spark/conf`目录下即可

![image-20220224220239234](https://i-blog.csdnimg.cn/blog_migrate/f0d853d63340832bb6d4bbdca270ff33.png)

然后启动spark thriftserver即可

在spark的官网文档中(spark-sql模块下)说明spark-thriftserver的脚本`start-thriftserver.sh`接受和spark-submit一样的参数

通过上面这句话，就基本上可以猜测出来启动thriftserver需要哪些参数了，当然也可以通过help查看

![image-20220224220633650](https://i-blog.csdnimg.cn/blog_migrate/99ea57e111f228c1ec36028253b3306a.png)

所以可以使用如下命令启动thriftserver

`./start-thriftserver.sh --master spark://hadoop01:4040 --jars /hive/lib/mysql-connector-java-8.0.27.jar --driver-class-path /hive/lib/mysql-connector-java-8.0.27.jar`

![image-20220224220912726](https://i-blog.csdnimg.cn/blog_migrate/9ecef142bf64fe2ab2b02966e3ce37eb.png)

启动后需要手动查看日志

启动出现了异常，提示地址已经被使用了，说白了就是端口冲突，因为hiveserver2还是启动，hiveserver2绑定了10000端口，而spark thriftserver启动也需要绑定10000端口，就冲突了。

所以修改spark thriftserver的端口为10001

![image-20220224221136872](https://i-blog.csdnimg.cn/blog_migrate/614516764b89871e7753902174c69fd3.png)

当然thriftserver的界面集成到了spark的界面中了，这里的8084应该是无效的

重新启动

![image-20220224221528565](https://i-blog.csdnimg.cn/blog_migrate/7df225b106dc65122ec6c7ed083a9eaa.png)

启动日志无报错后刷新spark集群的界面

![image-20220224221558613](https://i-blog.csdnimg.cn/blog_migrate/8e7fb80d93aa973883e7d38f74cd3c15.png)

进入application的界面后会多出来两个tab

![image-20220224221634321](https://i-blog.csdnimg.cn/blog_migrate/75c4d955d0167935bca582ba692f7ee4.png)

访问8084界面无响应，说明在`/spark/conf/hive-site.xml`中配置的界面端口无效

![image-20220224221731518](https://i-blog.csdnimg.cn/blog_migrate/4c99d40c884a627cb8dcb9f37e62733f.png)

## beeline 连接spark thriftserver

使用beeline连接spark thriftserver和使用beeline连接hiveserver2完全相同

使用`beeline -u jdbc:hive2://hadoop01:10001/test_column -n sparkbeeline`连接

![image-20220224222904282](https://i-blog.csdnimg.cn/blog_migrate/20322585aacf51332a505a3de8117bd9.png)

## thriftserver和spark-sql对比

thriftserver和spark-sql都能直接执行sql语句，区别在于多个beeline连接同一个thriftserver，共用一个application，而启动多个spark-sql就会启动多个application

比如首先在beeline连接的thriftserver中执行两次sql

![image-20220224223122221](https://i-blog.csdnimg.cn/blog_migrate/d08c55927cd8cca94b41957594ffe980.png)

刷新spark界面，并没有增加完成的application

![image-20220224223213175](https://i-blog.csdnimg.cn/blog_migrate/0344ce3ae72efb44c83562bac91c3e90.png)

停止thriftserver，因为thriftserver占用了全部集群中的两个核心(实际是我分配的核心太少了)，thriftserver会自动占用每个driver的一个核心，我配置每个worker只启动一个driver，每个driver只有一个核心，所以thriftserver就占用了全部的核心。

![image-20220224223445720](https://i-blog.csdnimg.cn/blog_migrate/a5c5db1af087d3b303069a9763844399.png)

接着启动spar-sql，占用一个核心

使用`spark-sql --num-executors 1 --master spark://hadoop01:4040 --jars /hive/lib/mysql-connector-java-8.0.27.jar --driver-class-path /hive/lib/mysql-connector-java-8.0.27.jar`启动spark-sql，要求只占用一个executor

![image-20220224224017305](https://i-blog.csdnimg.cn/blog_migrate/0200177ce5cf063ce623cfbe85c49473.png)

可惜的是，即使指定了`--num-executors`也没有生效，spark-sql又占用了全部核心

![image-20220224224134775](https://i-blog.csdnimg.cn/blog_migrate/d90439f1aeb43dfd86aedd34958c300a.png)

接着执行一个sql

执行发现，好像和预期的不一样，也是所有的sql使用同一个application

![image-20220224225442338](https://i-blog.csdnimg.cn/blog_migrate/60bbfd86bb464c333a2f40f4e382a701.png)

![image-20220224225502995](https://i-blog.csdnimg.cn/blog_migrate/a8633568cecd2a8de39d979a5fb8f12f.png)

在启动一个spark-sql

![image-20220224225909286](https://i-blog.csdnimg.cn/blog_migrate/c06be076463ee0b8bea7de3376022527.png)

此时查看spark界面，发现是两个application

![image-20220224225935745](https://i-blog.csdnimg.cn/blog_migrate/9040b37dabe99d5173e7f542f6d11e3d.png)

证明了每个spark-sql实例对应一个application（没有分配到资源请忽略）

接着启动thriftserver

![image-20220224230237771](https://i-blog.csdnimg.cn/blog_migrate/376362ebc89d57f99103b57ba09fd5ca.png)

接着启动一个beeline

![image-20220224231252714](https://i-blog.csdnimg.cn/blog_migrate/c4318ccdf70dbc84365c792869078560.png)

启动了两个beeline，连接同一个thriftserver,但是却只有一个application

![image-20220224231333300](https://i-blog.csdnimg.cn/blog_migrate/8efc362b2b0da47f5021150ba997b1fe.png)

这就是spark-sql与thriftserver最大的区别

## spark sql 程序连接thriftserver

使用spark-submit和spark-sql，每次执行一个sql，实际上都会产生一个spark application用于执行spark任务，如果执行的比较多的sql，就会频繁的创建spark application，这样就会在一定程度上耗费资源。为了实现spark application的复用，spark基本上把hive的hiveserver2照搬了过来，产生了 spark thriftserver，在使用thriftserver的时候，就能够复用spark application。

不过生产环境中每个查询因为数据量巨大，而且基本处于独立的查询操作，所以是不会使用thriftserver和hiveserver2的，这在开发中可能用的比较多。

要使用thriftserver就需要在依赖中加入hive-jdbc的依赖(前面说了，spark的thriftserver就是照搬hiveserver2的)

[Maven Repository: org.apache.hive » hive-jdbc » 3.1.2 (mvnrepository.com)](<https://mvnrepository.com/artifact/org.apache.hive/hive-jdbc/3.1.2>)

直接增加会因为无法下载`jdk-tools:jdk-toos.jar:1.6`报错，无法下载

在实际测试中，这个确实比较恶心，网上有一种解决方案是自己下载依赖，然后使用maven本地安装，然后在加载依赖，经过测试，没用。

我自己的解决方案是不管他，报红就使用排除
    
    
            <dependency>
                <groupId>org.apache.hive</groupId>
                <artifactId>hive-jdbc</artifactId>
                <version>${hive.jdbc.version}</version>
                <exclusions>
                    <exclusion>
                        <groupId>jdk.tools</groupId>
                        <artifactId>jdk.tools</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
    

然后在根目录下使用maven命令下载依赖`mvn dependency:sources`下载源码和依赖，然后重新加载maven项目，此时会把除了jdk.tools之外的依赖加入到项目中

![image-20220224013344093](https://i-blog.csdnimg.cn/blog_migrate/214d4897f912b3680d53b55ac1d48d56.png)

接着就可以开始编码了
    
    
    import java.sql.DriverManager
    
    object SparkThriftServerApp {
    
      def main(args: Array[String]): Unit = {
        // 1. 加载驱动
        Class.forName("org.apache.hive.jdbc.HiveDriver")
        // 2. 获取连接，用户名密码如果没有随便写，这里和beeline连接thriftserver是一样的
        val coon = DriverManager getConnection("jdbc:hive2://hadoop01:10000/test_column", "root", "")
        // 3. 编写查询sql
        val pstmt = coon prepareStatement "select * from columns limit 10"
        // 4. 查询
        val rs = pstmt executeQuery()
        // 5. 获取返回结果的元数据信息 --- 表头
        val metaData = rs getMetaData()
        // 6. 使用可变列表存储表头
        var columsArray = List[String]()
        // 7. 循环打印表头，切记从 1 开始，不是从 0 开始
        for(i <- Range(1, metaData getColumnCount) inclusive) {
          columsArray = columsArray :+ (metaData getColumnName i)
          print(s"${metaData getColumnName i}\t")
        }
        println
        // 8. 循环读取数据，也是从 1 开始，不是从 0 开始
        while (rs next()) {
          for (i <- Range(1, columsArray length) inclusive) {
            print(s"${rs getString(i)}\t")
          }
          println
        }
        // 9. 关闭连接
        rs close()
        pstmt close()
        coon close()
      }
    
    }
    

执行结果如下

![image-20220224013717788](https://i-blog.csdnimg.cn/blog_migrate/a78395d315c0e5a239e684eb603011c4.png)

也可以在thriftserver的监控界面查看执行的sql

![image-20220224013749506](https://i-blog.csdnimg.cn/blog_migrate/2369234e6ca8566fe7c1f9c8131c0c0d.png)

![image-20220224013812656](https://i-blog.csdnimg.cn/blog_migrate/1b8b61ef8ecc783e20ae565f11fcbe1b.png)
