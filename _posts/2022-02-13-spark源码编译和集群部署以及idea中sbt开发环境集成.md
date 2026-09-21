---
layout: post
title: "spark源码编译和集群部署以及idea中sbt开发环境集成"
date: 2022-02-13 21:18:07 +0800
categories: [spark, spark源码编译, spark单机集群部署配置, sbtscala的开发环境, sbt打包提交spark]
description: "spark源码编译和集群部署以及idea中sbt开发环境集成源码下载源码编译maven 下载scala 下载编译参数编译编译分发的二进制包单机启动集群部署开发环境集成源码编译的3.2.0版本无法在window上直接用spark-shell启动总结项目地址：https://gitee.com/jyq_18792721831/studyspark.git源码下载打开Apache Spark™ - Unified Engine for large-scale data analytics，下载源码在下载_spark 源码编译与部署"
keywords: spark, spark源码编译, spark单机集群部署配置, sbtscala的开发环境, sbt打包提交spark
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/122914322
> - 发布时间：2022-02-13 21:18:07
> - 阅读量：1
> - 分类：scala同时被 3 个专栏收录, 订阅专栏, 大数据, spark
> - 标签：#spark, #spark源码编译, #spark单机集群部署配置, #sbtscala的开发环境, #sbt打包提交spark

## 摘要

文章浏览阅读1.6k次。spark源码编译和集群部署以及idea中sbt开发环境集成源码下载源码编译maven 下载scala 下载编译参数编译编译分发的二进制包单机启动集群部署开发环境集成源码编译的3.2.0版本无法在window上直接用spark-shell启动总结项目地址：https://gitee.com/jyq_18792721831/studyspark.git源码下载打开Apache Spark™ - Unified Engine for large-scale data analytics，下载源码在下载_spark 源码编译与部署

---

#### spark源码编译和集群部署以及idea中sbt开发环境集成

  * 源码下载
  * 源码编译
  *     * maven 下载
    * scala 下载
    * 编译参数
    * 编译
    * 编译分发的二进制包
  * 单机启动
  * 集群部署
  * 开发环境集成
  * 源码编译的3.2.0版本无法在window上直接用spark-shell启动
  * 总结

  
项目地址：https://gitee.com/jyq_18792721831/studyspark.git 

## 源码下载

打开[Apache Spark™ - Unified Engine for large-scale data analytics](<https://spark.apache.org/>)，下载源码

![image-20220121204543201](https://i-blog.csdnimg.cn/blog_migrate/938e855861f0c6c2c93fa2cc413e1c07.png)

在下载界面中选择源码下载

![image-20220121204718586](https://i-blog.csdnimg.cn/blog_migrate/116e3a361d9cd6b0888a071d1f910620.png)

随便选择哪个地址

![image-20220121204750811](https://i-blog.csdnimg.cn/blog_migrate/178cd63b9df4056dbd39c9a81325bfba.png)

下载源码后上传到hadoop01服务器的`/spark`目录下

![image-20220121210309639](https://i-blog.csdnimg.cn/blog_migrate/4baee76014654b6521b857b5dcffdb26.png)

使用`tar -zxvf spark-3.2.0.tgz`解压到`/spark`目录下

![image-20220121213143024](https://i-blog.csdnimg.cn/blog_migrate/14df60bb85e9aaeec65b47cefd5ebd98.png)

## 源码编译

返回下载界面，选择文档，选择最新版或者你下载的版本

![image-20220121210354072](https://i-blog.csdnimg.cn/blog_migrate/51dd19a1d6f24ed2f58cf175984ed130.png)

或者你可以直接打开[Overview - Spark 3.2.0 Documentation (apache.org)](<https://spark.apache.org/docs/latest/>)界面

选择跳转到打包构建的文档

![image-20220121210519109](https://i-blog.csdnimg.cn/blog_migrate/ed87070fdba9e29c359c016be76b73f5.png)

有两种构建方式：maven和sbt(scala build tool)

![image-20220121210612658](https://i-blog.csdnimg.cn/blog_migrate/378980e4c7445f39093fd7b0920ac2b5.png)

我们选择maven

### maven 下载

在编译文档中，要求我们尽可能使用新的maven

![image-20220122021257900](https://i-blog.csdnimg.cn/blog_migrate/34ea429848f9490959b3c045a6af98c9.png)

所以我们使用手动安装的方式，安装最新版

首先打开[Maven – Welcome to Apache Maven](<https://maven.apache.org/>)，跳转到下载页

![image-20220122022722255](https://i-blog.csdnimg.cn/blog_migrate/d993ca7a40e9b8b4800e2d0fa39472b6.png)

我们选择最新的二进制包

![image-20220122022912923](https://i-blog.csdnimg.cn/blog_migrate/80027e1c8a7b62c0c9c01bcc10457370.png)

右键拷贝链接，直接在hadoop01上下载

在根目录下创建`/maven`目录，然后执行`curl -O https://dlcdn.apache.org/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.tar.gz`下载

![image-20220122023111638](https://i-blog.csdnimg.cn/blog_migrate/cdb7b9171f394389485f4971cb6248ae.png)

使用`tar -zxvf apache-maven-3.8.4-bin.tar.gz`解压

![image-20220122023148438](https://i-blog.csdnimg.cn/blog_migrate/d55050642a090418902ea257721a2fe5.png)

删除压缩包，并把maven里面的内容移动出来，使得maven的home为`/maven`

![image-20220122023253300](https://i-blog.csdnimg.cn/blog_migrate/0d17cc8c71f4e2347dacdb72c282346b.png)

因为之前就在hadoop01上安装过了java，并且配置了环境变量，所以可以直接配置maven的环境变量即可，高版本需要jdk1.7以上

![image-20220122023437042](https://i-blog.csdnimg.cn/blog_migrate/e6a523f4057e155149e02e42b95cc816.png)

配置环境变量

![image-20220122023604730](https://i-blog.csdnimg.cn/blog_migrate/0e1db6bfa1263d200cd6dc7839fb6631.png)

然后使用`source ~/.bash_profile`生效

使用`mvn --version`验证版本

![image-20220122023711359](https://i-blog.csdnimg.cn/blog_migrate/f2d256e15a92b22633d4d4fe2d900128.png)

### scala 下载

到[The Scala Programming Language (scala-lang.org)](<https://scala-lang.org/>)进入下载界面

![image-20220122025616569](https://i-blog.csdnimg.cn/blog_migrate/5981ce5e32eaa1af78c6ca59c602ca07.png)

选择scala2吧，scala3实在没用过。

拉到最下面，下载二进制包

![image-20220122025754631](https://i-blog.csdnimg.cn/blog_migrate/7ed7714c0313eb6c077a291c74ddc55b.png)

相同的方式，拷贝链接，在hadoop01中下载

![image-20220122030002004](https://i-blog.csdnimg.cn/blog_migrate/19d54a5860e278407e1b613a861df173.png)

然后配置环境变量

![image-20220122030842368](https://i-blog.csdnimg.cn/blog_migrate/1a85d2ed767433d4e9a59694194c3387.png)

使用`source ~/.bash_profile`生效后验证版本

![image-20220122030929715](https://i-blog.csdnimg.cn/blog_migrate/8128e250118c95defe251755b66b8bf4.png)

### 编译参数

根据文档，首先设置maven的信息

![image-20220121212426892](https://i-blog.csdnimg.cn/blog_migrate/0762d55ee24435c913dc3c6644523073.png)

我们将`MAVEN_OPTS`放入`~/.bash_profile`中

![image-20220122023822541](https://i-blog.csdnimg.cn/blog_migrate/3e8b766660dd345e588acbb3f8fcbbfa.png)

使用`source ~/.bash_profile`使之生效。

接着往下看构建文档，找到支持hive和jdbc的构建

![image-20220121212754471](https://i-blog.csdnimg.cn/blog_migrate/e5c033aedc77e30ead249e12ce107bba.png)

这是maven命令，使用源码包中自带的maven，其中`-P`和`-D`是一些参数。

这些参数和pom.xml文件中的profile和properties有关。

`-D`对应properties

`-P`对应profile

我们查看pom.xml文件

这只是一部分，具体的可以自行查看
    
    
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <!-- 我们使用的是java 11，这个需要进行替换 -->
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <!-- 因为我们使用的是3.8.1的maven，所以这个也需要替换 -->
        <maven.version>3.6.3</maven.version>
        <exec-maven-plugin.version>1.6.0</exec-maven-plugin.version>
        <sbt.project.name>spark</sbt.project.name>
        <slf4j.version>1.7.30</slf4j.version>
        <log4j.version>1.2.17</log4j.version>
        <!-- hadoop 的版本是2.9.2 -->
        <hadoop.version>3.3.1</hadoop.version>
        <protobuf.version>2.5.0</protobuf.version>
        <yarn.version>${hadoop.version}</yarn.version>
        <zookeeper.version>3.6.2</zookeeper.version>
        <curator.version>2.13.0</curator.version>
        <hive.group>org.apache.hive</hive.group>
        <hive.classifier>core</hive.classifier>
        <!-- Version used in Maven Hive dependency -->
        <!-- hive的版本是2.3.9，这个可以不用修改 -->
        <hive.version>2.3.9</hive.version>
        <hive23.version>2.3.9</hive23.version>
        <!-- Version used for internal directory structure -->
        <hive.version.short>2.3</hive.version.short>
        <!-- note that this should be compatible with Kafka brokers version 0.10 and up -->
        <kafka.version>2.8.0</kafka.version>
        <!-- After 10.15.1.3, the minimum required version is JDK9 -->
        <derby.version>10.14.2.0</derby.version>
        <parquet.version>1.12.1</parquet.version>
        <orc.version>1.6.11</orc.version>
        <jetty.version>9.4.43.v20210629</jetty.version>
        <jakartaservlet.version>4.0.3</jakartaservlet.version>
        <chill.version>0.10.0</chill.version>
        <ivy.version>2.5.0</ivy.version>
        <oro.version>2.0.8</oro.version>
        <!--
        If you changes codahale.metrics.version, you also need to change
        the link to metrics.dropwizard.io in docs/monitoring.md.
        -->
        <codahale.metrics.version>4.2.0</codahale.metrics.version>
        <avro.version>1.10.2</avro.version>
        <aws.kinesis.client.version>1.12.0</aws.kinesis.client.version>
        <!-- Should be consistent with Kinesis client dependency -->
        <aws.java.sdk.version>1.11.655</aws.java.sdk.version>
        <!-- the producer is used in tests -->
        <aws.kinesis.producer.version>0.12.8</aws.kinesis.producer.version>
        <!--  org.apache.httpcomponents/httpclient-->
        <commons.httpclient.version>4.5.13</commons.httpclient.version>
    </properties>
    

所以，我们的构建命令中`-D`要设置的如下

`mvn -Djava.version=11 -Dmaven.version=3.8.1 -Dhadoop.version=2.9.2 -Dscala.version=2.13.8 -DskipTests clean package`

接下来是`-P`参数

`-P`参数对应的是pom.xml中的profile属性
    
    
        <profile>
          <id>hadoop-2.7</id>
          <properties>
            <hadoop.version>2.7.4</hadoop.version>
            <curator.version>2.7.1</curator.version>
            <commons-io.version>2.4</commons-io.version>
            <hadoop-client-api.artifact>hadoop-client</hadoop-client-api.artifact>
            <hadoop-client-runtime.artifact>hadoop-yarn-api</hadoop-client-runtime.artifact>
            <hadoop-client-minicluster.artifact>hadoop-client</hadoop-client-minicluster.artifact>
          </properties>
        </profile>
    	<profile>
          <id>hive-2.3</id>
        </profile>
    	<profile>
          <id>yarn</id>
          <modules>
            <module>resource-managers/yarn</module>
            <module>common/network-yarn</module>
          </modules>
        </profile>
    	<profile>
          <id>hive-thriftserver</id>
          <modules>
            <module>sql/hive-thriftserver</module>
          </modules>
        </profile>
    	<profile>
          <id>scala-2.13</id>
          <properties>
            <scala.version>2.13.5</scala.version>
            <scala.binary.version>2.13</scala.binary.version>
          </properties>
          <build>
            <pluginManagement>
              <plugins>
                <plugin>
                  <groupId>net.alchim31.maven</groupId>
                  <artifactId>scala-maven-plugin</artifactId>
                  <configuration>
                    <args>
                      <arg>-unchecked</arg>
                      <arg>-deprecation</arg>
                      <arg>-feature</arg>
                      <arg>-explaintypes</arg>
                      <arg>-target:jvm-1.8</arg>
                      <arg>-Wconf:cat=deprecation:wv,any:e</arg>
                      <arg>-Wconf:cat=scaladoc:wv</arg>
                      <arg>-Wconf:cat=lint-multiarg-infix:wv</arg>
                      <arg>-Wconf:cat=other-nullary-override:wv</arg>
                      <arg>-Wconf:cat=other-match-analysis&amp;site=org.apache.spark.sql.catalyst.catalog.SessionCatalog.lookupFunction.catalogFunction:wv</arg>
                      <arg>-Wconf:cat=other-pure-statement&amp;site=org.apache.spark.streaming.util.FileBasedWriteAheadLog.readAll.readFile:wv</arg>
                      <arg>-Wconf:cat=other-pure-statement&amp;site=org.apache.spark.scheduler.OutputCommitCoordinatorSuite.&lt;local OutputCommitCoordinatorSuite&gt;.futureAction:wv</arg>
                      <arg>-Wconf:msg=^(?=.*?method|value|type|object|trait|inheritance)(?=.*?deprecated)(?=.*?since 2.13).+$:s</arg>
                      <arg>-Wconf:msg=^(?=.*?Widening conversion from)(?=.*?is deprecated because it loses precision).+$:s</arg>
                      <arg>-Wconf:msg=Auto-application to \`\(\)\` is deprecated:s</arg>
                      <arg>-Wconf:msg=method with a single empty parameter list overrides method without any parameter list:s</arg>
                      <arg>-Wconf:msg=method without a parameter list overrides a method with a single empty one:s</arg>
                      <arg>-Wconf:cat=deprecation&amp;msg=procedure syntax is deprecated:e</arg>
                    </args>
                    <compilerPlugins combine.self="override">
                    </compilerPlugins>
                  </configuration>
                </plugin>
              </plugins>
            </pluginManagement>
          </build>
    	</profile>
    

所以`-P`的参数大概就这么多

`mvn -Phadoop-2.7 -Phive-2.3 -Pyarn -Phive-thriftserver -Pscala-2.13 -Djava.version=11 -Dmaven.version=3.8.4 -Dhadoop.version=2.9.2 -Dscala.version=2.13.8 -Dscala.binary.version=2.13 -DskipTests clean package`

### 编译

如果使用的是scala-2.13版本，那么需要切换scala版本

![image-20220122171302264](https://i-blog.csdnimg.cn/blog_migrate/6447b79b3d298c9e68b1921a241c4389.png)

查看`change-scala-version.sh`的内容，主要是环境变量的配置和pom.xml文件中的版本号的修改。

在`pom.xml`中，scala相关的配置是properties，理论上我们使用`-D`参数就能修改，不过可能有其他的配置，不一定是pom.xml的，反正执行一次最好。

而且，需要注意的是，执行这个命令，必须在spark主目录下，因为在`change-scala-version.sh`脚本中使用相对路径执行其他脚本，所以如果你在其他位置执行，那么会因为上下文的问题，导致执行失败。

实际上，`change-scala-version.sh`脚本还会使用spark主目录下`build/mvn`程序。

我们使用上面拼接好的命令，在`SPARK_HOME`下执行

第一次执行，maven 会下载相关的依赖，比较慢

![image-20220122141748723](https://i-blog.csdnimg.cn/blog_migrate/b3488c57f90a19359c6a80f03172f290.png)

哎，依赖是在是多，耐心等待吧

![image-20220122142400052](https://i-blog.csdnimg.cn/blog_migrate/c92a4394ceb603d75901fc229765f7d2.png)

等待了一段时间，发现编译错误了

![image-20220122144324392](https://i-blog.csdnimg.cn/blog_migrate/0d79fc21d9930a482c63afab2c96c42b.png)

很正常，一次成功才不正常。

我们查看错误的堆栈，发现是因为找不到一个为`RangePartitioner`的类，查看源码，这是一个在`apache/spark/blob/master/core/src/main/scala/org/apache/spark/Partitioner.scala`中定义的类

![image-20220122171035787](https://i-blog.csdnimg.cn/blog_migrate/7eea7a7294e65b7143b13149503f01c1.png)

问题在于scala-2.13中没有打包生成class文件，而scala-2.12中就会打包生成calss文件

![image-20220122171144984](https://i-blog.csdnimg.cn/blog_migrate/69f194d401989731a0b7fae5c9f83b22.png)

生成的class文件存储在`/spark/core/target/scala-2.12/classes/org/apache/spark`目录下

不知道是不是scala和spark的兼容性问题。（我自己试了好多次，2.12.15可以编译成功，2.13.8就编译失败了。）

在使用scala-2.12编译的时候，编译到spark-sql的时候，卡主，怀疑是内存不足了

![image-20220122172055296](https://i-blog.csdnimg.cn/blog_migrate/886813aee3cc96c71c90cf7b0a17daf7.png)

幸好是虚拟机，可以增加内存，我们尝试将内存增加到4G

然后终止编译，重新编译，并使用`-X`启动debug输出

经过尝试，调大内存在一定程度上能加快编译速度。

![image-20220122173452320](https://i-blog.csdnimg.cn/blog_migrate/c21df5569d536d28916ca55d5d8c1093.png)

第一次编译可以采用一步一步执行，可以先编译后打包，其实打包也包含编译，我单独执行编译是为了验证scala-2.12是否会生成相关的class文件，打包成功如下

![image-20220122195503715](https://i-blog.csdnimg.cn/blog_migrate/ff2e477c342c70e0ff5c7fc91b662127.png)

编译成功就会生成二进制文件，存储于spark根目录的bin目录下

![image-20220122205137365](https://i-blog.csdnimg.cn/blog_migrate/23acbdb99a3d11f6136129d7344075c7.png)

其实这只是一部分，这些脚本调用的jar包也会生成。

我们在源码的bin目录下就可以开心的使用spark了

直接启动spark-shell

![image-20220122205724382](https://i-blog.csdnimg.cn/blog_migrate/1ccf6cd18d0409229b9959e18fd3ddc0.png)

这些版本信息和我们在编译的时候指定的一模一样。

### 编译分发的二进制包

编译好只是这个环境编译好了，里面既有源码文件，也有可执行文件，我们不能在集群环境的每个节点上都进行编译一次，那么太过费时，而且也不可行。

所以我们需要编译可分发的二进制包，就像官网提供的spark一样，没有源码，全部是执行文件。

在官网中是建议我们使用工具编译分发的二进制包

![image-20220122031631008](https://i-blog.csdnimg.cn/blog_migrate/db409e7f2a1da569923580d0d023c106.png)

查看这个脚本，发现还是maven命令

![image-20220122031850810](https://i-blog.csdnimg.cn/blog_migrate/59c84723c958c719d8f3351491c439db.png)

最后将编译后在bin目录中的内容进行打包而已

![image-20220122141625010](https://i-blog.csdnimg.cn/blog_migrate/c8dc2c5a4274145bd334d07b53da1875.png)

我们在spark的根目录下执行`./dev/make-distribution.sh --name 2.12 --tgz --mvn mvn -Phadoop-2.7 -Phive-2.3 -Pyarn -Phive-thriftserver -Pscala-2.12 -Djava.version=11 -Dmaven.version=3.8.4 -Dhadoop.version=2.9.2 -Dscala.version=2.12.15 -Dscala.binary.version=2.12 -DskipTests clean package`

打包完成后就会生成`psark-2.9.2-bin-2.12.tgz`压缩包。

![image-20220122210318410](https://i-blog.csdnimg.cn/blog_migrate/954f086e8b92483322c7fd764de1fc29.png)

并不会打印maven的日志，等待就行了，可能需要和maven打包同样长的时间。(2核，8G，大约40分钟)

完成后会在spark根目录下生成可分发二进制文件包

![image-20220123153440615](https://i-blog.csdnimg.cn/blog_migrate/c595e64db5ada5a3289cd3854dd0d170.png)

实际上当我们使用maven命令完成打包后，只是单独的想打包分发二进制包，那么是完全不需要重新执行maven打包的，毕竟maven打包太慢了。我们可以将spark根目录下的`./dev/make-distribution.sh`文件中的相关maven的命令跳过，注释掉脚本中的这两行即可。

![image-20220122235213609](https://i-blog.csdnimg.cn/blog_migrate/5ab9ce896c2229886f71f86075118b82.png)

然后把原来脚本中使用maven获取版本号的方式自定义指定即可

![image-20220123153942543](https://i-blog.csdnimg.cn/blog_migrate/bae5ea3962290aed769379960888d676.png)

这样就不会重复执行maven命令了(maven命令太费时间了)

然后在maven打包完成的前提下，执行分发的二进制包的命令`./dev/make-distribution.sh --name 2.12 --tgz --mvn mvn -Phadoop-2.7 -Phive-2.3 -Pyarn -Phive-thriftserver -Pscala-2.12 -Djava.version=11 -Dmaven.version=3.8.4 -Dhadoop.version=2.9.2 -Dscala.version=2.12.15 -Dscala.binary.version=2.12 -DskipTests clean package`

![image-20220123153501705](https://i-blog.csdnimg.cn/blog_migrate/dcda60818122096ed70bc9708125dc51.png)

spark源码编译实际上就是maven编译，如果你了解maven编译，那么在spark源码编译中遇到的问题，你都能很好的解决。

而二进制分发包的打包脚本，就是对maven编译的使用，然后在把编译后的文件重新组合。

## 单机启动

我们将编译好的二进制包，从hadoop01分发到hadoop02上，使用`scp spark-3.2.0-bin-2.12.tgz hadoop02:/spark/`

![image-20220123154928367](https://i-blog.csdnimg.cn/blog_migrate/1cb2fc6aa387e400bba56586a56981df.png)

然后在hadoop02上解压,`tar -zxvf spark-3.2.0-bin-2.12.tgz`

我们选择在hadoop02上验证二进制分发包是否可用。

![image-20220123155536201](https://i-blog.csdnimg.cn/blog_migrate/1ac4b8c86d934212f9da163a2b62ed15.png)

然后将`SPARK_HOME`设置到环境变量，并使之生效

![image-20220123155650929](https://i-blog.csdnimg.cn/blog_migrate/e0d2c692da8a5fe570f3e4fc8e519b03.png)

然后你可以在任意位置使用`spark-shell`启动

相关的文档在这里[Quick Start - Spark 3.2.0 Documentation (apache.org)](<https://spark.apache.org/docs/latest/quick-start.html>)

![image-20220123155820491](https://i-blog.csdnimg.cn/blog_migrate/88fd593d0dfb2242b5499002f86f1102.png)

![image-20220123155949101](https://i-blog.csdnimg.cn/blog_migrate/ed0e68a3e88606efa44cab23cb759e8c.png)

启动后如下

![image-20220123160036035](https://i-blog.csdnimg.cn/blog_migrate/250fbe9f19fd7e7d90e726afdbd660e6.png)

可见启动spark并不需要scala环境，应该把scala的相关的包集成进去了

我们尝试一下quick-started中的例子

![image-20220123160301066](https://i-blog.csdnimg.cn/blog_migrate/ecce5e17ca35628f6df138600d6651e7.png)

然后打开`http://hadoop02:4040`查看执行历史

![image-20220123160349548](https://i-blog.csdnimg.cn/blog_migrate/71ce94ec06f3e6bdcfa47882b90df66e.png)

这种启动方式，在启动中会打印如下数据

![image-20220123160540572](https://i-blog.csdnimg.cn/blog_migrate/96f0f396de0dd9d5bd466a785da55a8d.png)

这其实是spark的standalone的启动方式

相关的文档在这[Overview - Spark 3.2.0 Documentation (apache.org)](<https://spark.apache.org/docs/latest/>)

![image-20220123160745708](https://i-blog.csdnimg.cn/blog_migrate/b159fe8df63798c82e8db6b56fb48977.png)

关于standalone的模式的相关文档

![image-20220123160820231](https://i-blog.csdnimg.cn/blog_migrate/c3380ae38df1b2f120628e742d78f3d7.png)

[Spark Standalone Mode - Spark 3.2.0 Documentation (apache.org)](<https://spark.apache.org/docs/latest/spark-standalone.html>)

## 集群部署

为什么直接启动`spark-shell`就是单机版的模式呢？

spark的配置和hadoop差不多，有个slaves文件，里面会指定从节点的域名

默认的配置在`/spark/conf`文件里面

![image-20220123161033850](https://i-blog.csdnimg.cn/blog_migrate/f26f7e71d65c47815a0e555ae004b014.png)

但是你在这里面是找不到slaves文件的，不过有一个workers.template文件

查看这个文件，默认是localhost

![image-20220123162059319](https://i-blog.csdnimg.cn/blog_migrate/b99efc84fb552e8ec1cc46253aeb6f07.png)

在sbin目录下有一个`slaves.sh`文件

![image-20220123161403537](https://i-blog.csdnimg.cn/blog_migrate/e081139333649bd7e16ceb42dcb1d48b.png)

`slaves.sh`文件跳转到了`workers.sh`文件中了

![image-20220123161739953](https://i-blog.csdnimg.cn/blog_migrate/81465a9b99b71c04b187049159c9a636.png)

通过这个脚本基本上就能明白了，spark的从节点配置在`/sbin/spark-config.sh`和`/conf`中都可以配置

其中所有的配置都可以在`/conf/spark-env.sh`中配置

这些是一些standalone的配置

![image-20220123162947842](https://i-blog.csdnimg.cn/blog_migrate/a1e2fec19784c887cf4206f2c92cdac9.png)

基本上都是通用的。

我们搭建一个spark的集群，集群信息如下

| 节点       | 角色  | 其他    |
|----------|-----|-------|
| hadoop01 | 主节点 | 一个执行器 |
| hadoop02 | 从节点 | 一个执行器 |
| hadoop03 | 从节点 | 一个执行器 |

完整的`/conf/spark-env.sh`如下
    
    
    # Options read when launching programs locally with
    # ./bin/run-example or ./bin/spark-submit
    # - HADOOP_CONF_DIR, to point Spark towards Hadoop configuration files
    # - SPARK_LOCAL_IP, to set the IP address Spark binds to on this node
    # - SPARK_PUBLIC_DNS, to set the public dns name of the driver program
    
    # hadoop 的配置信息
    HADOOP_CONF_DIR=/hadoop/etc/hadoop
    
    # Options read by executors and drivers running inside the cluster
    # - SPARK_LOCAL_IP, to set the IP address Spark binds to on this node
    # - SPARK_PUBLIC_DNS, to set the public DNS name of the driver program
    # - SPARK_LOCAL_DIRS, storage directories to use on this node for shuffle and RDD data
    # - MESOS_NATIVE_JAVA_LIBRARY, to point to your libmesos.so if you use Mesos
    
    # spark shuffle的数据目录，
    SPARK_LOCAL_DIRS=/spark/shuffle_data
    
    # Options read in YARN client/cluster mode
    # - SPARK_CONF_DIR, Alternate conf dir. (Default: ${SPARK_HOME}/conf)
    # - HADOOP_CONF_DIR, to point Spark towards Hadoop configuration files
    # - YARN_CONF_DIR, to point Spark towards YARN configuration files when you use YARN
    # - SPARK_EXECUTOR_CORES, Number of cores for the executors (Default: 1).
    # - SPARK_EXECUTOR_MEMORY, Memory per Executor (e.g. 1000M, 2G) (Default: 1G)
    # - SPARK_DRIVER_MEMORY, Memory for Driver (e.g. 1000M, 2G) (Default: 1G)
    
    # yarn的配置文件在hadoop的配置文件中
    YARN_CONF_DIR=$HADOOP_CONF_DIR
    # 每个节点启动几个执行器
    SPARK_EXECUTOR_CORES=1
    # 每个执行器可以使用多大内存
    SPARK_EXECUTOR_MEMORY=1800M
    # 每个driver可以使用多大内存
    SPARK_DRIVER_MEMORY=1800M
    
    # Options for the daemons used in the standalone deploy mode
    # - SPARK_MASTER_HOST, to bind the master to a different IP address or hostname
    # - SPARK_MASTER_PORT / SPARK_MASTER_WEBUI_PORT, to use non-default ports for the master
    # - SPARK_MASTER_OPTS, to set config properties only for the master (e.g. "-Dx=y")
    # - SPARK_WORKER_CORES, to set the number of cores to use on this machine
    # - SPARK_WORKER_MEMORY, to set how much total memory workers have to give executors (e.g. 1000m, 2g)
    # - SPARK_WORKER_PORT / SPARK_WORKER_WEBUI_PORT, to use non-default ports for the worker
    # - SPARK_WORKER_DIR, to set the working directory of worker processes
    # - SPARK_WORKER_OPTS, to set config properties only for the worker (e.g. "-Dx=y")
    # - SPARK_DAEMON_MEMORY, to allocate to the master, worker and history server themselves (default: 1g).
    # - SPARK_HISTORY_OPTS, to set config properties only for the history server (e.g. "-Dx=y")
    # - SPARK_SHUFFLE_OPTS, to set config properties only for the external shuffle service (e.g. "-Dx=y")
    # - SPARK_DAEMON_JAVA_OPTS, to set config properties for all daemons (e.g. "-Dx=y")
    # - SPARK_DAEMON_CLASSPATH, to set the classpath for all daemons
    # - SPARK_PUBLIC_DNS, to set the public dns name of the master or workers
    
    # 主节点
    SPARK_MASTER_HOST=hadoop01
    # 端口，不要和worker的端口重复，通信端口
    SPARK_MASTER_PORT=4040
    # 界面端口
    SPARK_MASTER_WEBUI_PORT=8089
    # 每个从节点启动的执行器
    SPARK_WORKER_CORES=1
    # 每个从节点可使用最大内存
    SPARK_WORKER_MEMORY=1800M
    # 从节点的通信端口
    SPARK_WORKER_PORT=4040
    # 界面端口
    SPARK_WORKER_WEBUI_PORT=8089
    # 从节点工作目录
    SPARK_WORKER_DIR=/spark/worker
    
    # Options for launcher
    # - SPARK_LAUNCHER_OPTS, to set config properties and Java options for the launcher (e.g. "-Dx=y")
    
    # Generic options for the daemons used in the standalone deploy mode
    # - SPARK_CONF_DIR      Alternate conf dir. (Default: ${SPARK_HOME}/conf)
    # - SPARK_LOG_DIR       Where log files are stored.  (Default: ${SPARK_HOME}/logs)
    # - SPARK_LOG_MAX_FILES Max log files of Spark daemons can rotate to. Default is 5.
    # - SPARK_PID_DIR       Where the pid file is stored. (Default: /tmp)
    # - SPARK_IDENT_STRING  A string representing this instance of spark. (Default: $USER)
    # - SPARK_NICENESS      The scheduling priority for daemons. (Default: 0)
    # - SPARK_NO_DAEMONIZE  Run the proposed command in the foreground. It will not output a PID file.
    # Options for native BLAS, like Intel MKL, OpenBLAS, and so on.
    # You might get better performance to enable these options if using native BLAS (see SPARK-21305).
    # - MKL_NUM_THREADS=1        Disable multi-threading of Intel MKL
    # - OPENBLAS_NUM_THREADS=1   Disable multi-threading of OpenBLAS
    
    # spark 的日志目录
    SPARK_LOG_DIR=/spark/logs
    

相关的配置项说明在这[Spark Standalone Mode - Spark 3.2.0 Documentation (apache.org)](<https://spark.apache.org/docs/latest/spark-standalone.html>)

![image-20220123173458695](https://i-blog.csdnimg.cn/blog_migrate/3f75fba9938edac8005c7a27deb4569e.png)

然后配置workers

![image-20220123164007658](https://i-blog.csdnimg.cn/blog_migrate/25a5396ce1ddf496066ebb6fe2e6d50b.png)

接着把这个配置好的文件分发到hadoop02的`/spark目录下`

使用`scp -r /spark hadoop02:/spark/`

![image-20220123164159436](https://i-blog.csdnimg.cn/blog_migrate/ebf4972d39606b7114c05025a7787446.png)

然后在hadoop01上启动（当然如果你在hadoop01上配置那么就可以不需要从hadoop02分发到hadoop01上了）

别忘记`SPARK_HOME`

![image-20220123164856859](https://i-blog.csdnimg.cn/blog_migrate/28c044e247bdf5b8d98f0b931bb684b0.png)

接着切换到`/spark/sbin`目录下，执行启动脚本

![image-20220123164928839](https://i-blog.csdnimg.cn/blog_migrate/08d48d798d96dfc5a1874178808b63ea.png)

启动过程中提示hadoop03还没有相关文件

![image-20220123165024129](https://i-blog.csdnimg.cn/blog_migrate/1f6b9644705e1e90c637a1c47db73c07.png)

并且提示我们hadoop02的JAVA_HOME没有设置，这就离谱了，我们第一个配置的就是JAVA_HOME。

错误信息中给出了启动worker的命令，我们直接拷贝到hadoop02上执行看下，发现是可以执行的

![image-20220123165423171](https://i-blog.csdnimg.cn/blog_migrate/2bd6d5d78636290a1de6fff1220d36b9.png)

接着我们在界面中看下

![image-20220123165507725](https://i-blog.csdnimg.cn/blog_migrate/5af34fce84ee2389d747a499eb2ffe6a.png)

到worker里面看下，发现和我们配置的一样

![image-20220123165626575](https://i-blog.csdnimg.cn/blog_migrate/64ffec5bec154e3cf48eae4a579e8827.png)

查看java进程，发现master已经在hadoop01上启动

![image-20220123165845922](https://i-blog.csdnimg.cn/blog_migrate/2fa9e573e8d77733577b00da049a4ef3.png)

hadoop02启动的是worker

![image-20220123165911512](https://i-blog.csdnimg.cn/blog_migrate/26aaf62c83f56cd6df5079d5c680a1e7.png)

现在剩下一个问题，在hadoop01上启动，无法启动hadoop02的worker

我们查看下`start-all.sh`

实际上`start-all.sh`启动了`start-master.sh`和`start-worker.sh`

![image-20220123170101479](https://i-blog.csdnimg.cn/blog_migrate/bf903bdb104139a20db287c8236528d1.png)

重点看`start-worker.sh`

![image-20220123171400069](https://i-blog.csdnimg.cn/blog_migrate/4be40d31dbbe4ef1b5775c473d9ffafc.png)

可以看到会执行这两个脚本，用于处理配置，我们把java环境在`spark-config.sh`中设置下

![image-20220123171936168](https://i-blog.csdnimg.cn/blog_migrate/58b021e7632a836e098c6c6ae3fb9d34.png)

然后重新分发并重启

![image-20220123172056009](https://i-blog.csdnimg.cn/blog_migrate/2cb0d70e495f7747146baa35e05f3d52.png)

就没有报错，并且在hadoop02上成功启动了

![image-20220123172128117](https://i-blog.csdnimg.cn/blog_migrate/4dc49e7313e16d3126aa3edd11d6c3d3.png)

将`/spark`目录分发到hadoop03并进行重启

![image-20220123172802622](https://i-blog.csdnimg.cn/blog_migrate/f3bf9b2a3f4d5e75fa7a22987f40b82c.png)

![image-20220123172827088](https://i-blog.csdnimg.cn/blog_migrate/756dc6414973b3576b43373736dd62bb.png)

需要注意，我们配置主节点和从节点的通信端口都是4040,界面访问端口都是8089。这也就是意味着，如果要在hadoop01上启动Worker ，会出现端口冲突。

不过我们有三个节点，本来就是打算让hadoop01成为master的。

## 开发环境集成

我们开发spark程序是在idea中开发的，所以需要在idea中安装scala的插件

![image-20220123173920675](https://i-blog.csdnimg.cn/blog_migrate/b62e67d0cf314068ec1177d06e8d16d0.png)

然后在windows环境中配置scala的环境

先把scala的文件从hadoop01上下载到windows中

![image-20220123174115624](https://i-blog.csdnimg.cn/blog_migrate/79332099e41dd3ac9f2079315cf68ee1.png)

然后配置环境变量

![image-20220123174505494](https://i-blog.csdnimg.cn/blog_migrate/333221df07109500046697f77a180f56.png)

接着在idea中指定scala的sdk

![image-20220123174639707](https://i-blog.csdnimg.cn/blog_migrate/9ac0a280c5f102673cd684357f3957cc.png)

![image-20220123174829790](https://i-blog.csdnimg.cn/blog_migrate/1f7740e76bcfaf12c39b291531ea7e5b.png)

idea会自动扫描全部的scala的sdk

![image-20220123174926527](https://i-blog.csdnimg.cn/blog_migrate/b3e94c0ff40ac43c2557b8b6eab4d826.png)

选择scala目录

![image-20220123174952026](https://i-blog.csdnimg.cn/blog_migrate/42368f70ccf5c05ca1c1492aef50915e.png)

有了scala就可以创建spark的项目了，不过还需要scala的打包构建工具，也就是sbt

![image-20220123175051429](https://i-blog.csdnimg.cn/blog_migrate/12d631a490aeb79ed8872bff807f04f9.png)

我们到[sbt - The interactive build tool (scala-sbt.org)](<https://www.scala-sbt.org/>)下载sbt

![image-20220123175253812](https://i-blog.csdnimg.cn/blog_migrate/6eda6389659bcd83161f32245654715d.png)

下载zip包就行

![image-20220123175332397](https://i-blog.csdnimg.cn/blog_migrate/b70fe4c3459e021dcf4013fdfbea1978.png)

将下载的sbt解压到windows中，并配置环境变量

![image-20220123195617714](https://i-blog.csdnimg.cn/blog_migrate/a81fd64244c19d4d2a0bdc7e0ff4f5dc.png)

继续创建项目(scala版本最好选择和我们编译spark相同的版本，当然创建好项目后在修改也是可以的)

![image-20220123195725733](https://i-blog.csdnimg.cn/blog_migrate/f32d56fb4927e25713186921481b69d0.png)

关于sbt的更多资料请看

[sbt入门_a18792721831的博客-CSDN博客](<https://blog.csdn.net/a18792721831/article/details/122871070>)

[sbt使用教程_a18792721831的博客-CSDN博客](<https://blog.csdn.net/a18792721831/article/details/122904465>)

根据上面这两篇资料的内容，简单了解下sbt。不希望能做到更好，只要能正常使用就行了。

然后重新打开idea，打开新建的studyspark项目

sbt就开始下载依赖了

![image-20220123200339817](https://i-blog.csdnimg.cn/blog_migrate/59b490dc5c1f35d10d6ebea673fa111e.png)

此时sbt加载项目会异常的，因为idea默认使用自己的sbt，而不是我们安装的sbt

![image-20220123200709091](https://i-blog.csdnimg.cn/blog_migrate/ba5ecfa07e27b40bc6ea46c230dce276.png)

需要在settings中设置我们自己的sbt

![image-20220127180518468](https://i-blog.csdnimg.cn/blog_migrate/0fcb7c1386e17e9261d135fce7414d1e.png)

重新构建项目

![image-20220123200935437](https://i-blog.csdnimg.cn/blog_migrate/f613fd82c6825f1a6a579c02e6e8163a.png)

会弹出sbt-shell（如果你没有把sbt配置的全部勾选，那么应该是不会弹出sbt-shell的，那么这步可以跳过）

`/*************************跳过开始*****************************`

![image-20220123200957189](https://i-blog.csdnimg.cn/blog_migrate/76fe096cd36258ee33364d801b4877af.png)

如果你也出现上述提示，那么表示在等待sbt使用默认的配置，从国外下载依赖

我们在sbt的配置中增加如下参数
    
    
    -Dsbt.override.build.repos=true
    -Dsbt.repository.config=E:\sbt\conf\repo.properties
    

第一行表示使用全局的仓库配置

第二行则是指定使用的仓库配置的文件路径

![image-20220213125747234](https://i-blog.csdnimg.cn/blog_migrate/96c9ae55b6ae1da36ae5f8f5a2fd3bea.png)

记得取消这个勾选

![image-20220123204419388](https://i-blog.csdnimg.cn/blog_migrate/f899eaff2f5cd07040ba6cba0c4bb18a.png)

然后重新导入项目

![image-20220123202717437](https://i-blog.csdnimg.cn/blog_migrate/9547399006cbc242917e52d13bd782ee.png)

这里发现sbt-shell中文乱码了，很可惜，我尝试了网上找到的方式，都失败了，或许你可以试试，在sbt的参数中增加
    
    
    -Dfile.encoding=UTF-8
    -Dconsold.encoding=GBK
    

对我都是无效的😆

`********************跳过结束*********************`

此时项目结构如下

![image-20220123202733192](https://i-blog.csdnimg.cn/blog_migrate/fe1ebaa0d4b9898ab6f4692b20c9c687.png)

项目相关的配置可以在`build.sbt`中修改

build.sbt如下
    
    
    // 创建配置，用于标识spark的版本
    lazy val sparkVersion = SettingKey[String]("spark-version", "spark version")
    // 定义公共的包基础
    lazy val packageBaseDir = SettingKey[String]("package-base-dir", "package base dir")
    // 抽取公共配置
    lazy val commonSettings = Seq(
      version := "1.0",
      scalaVersion := "2.12.15",
      sbtVersion := "1.6.1",
      sparkVersion := "3.2.0",
      // 定义项目包基础
      packageBaseDir := "com.study.spark",
      // 定义每个项目的包前缀
      ThisProject / idePackagePrefix := Some(packageBaseDir.value + "." + name.value),
      // 定义输出目录
      ideOutputDirectory := Some(file("target")),
      organization := "com.study.spark",
      // 申明源码路径
      sourceDirectories := Seq(
        file("src/main/scala"),
        file("src/main/java"),
        file("src/test/scala"),
        file("src/test/java")
      ),
      // 声明资源路径
      resourceDirectories := Seq(
        file("src/main/resources"),
        file("src/test/resources")
      ),
      libraryDependencies ++= Seq(
        "org.apache.spark" %% "spark-core" % sparkVersion.value % "provided",
        "org.apache.spark" %% "spark-sql" % sparkVersion.value % "provided",
        "org.apache.spark" %% "spark-mllib" % sparkVersion.value % "provided",
        "org.apache.spark" %% "spark-streaming" % sparkVersion.value % "provided"
      )
    )
    // 设置根项目
    lazy val root = (project in file("."))
      .settings(
        name := "studyspark",
        commonSettings
      )
    

接着我们把`sbt-assembly`插件加入到项目中

![image-20220213132250499](https://i-blog.csdnimg.cn/blog_migrate/4573c78c373e292db1801a3835cf31f8.png)

等待sbt刷新项目后如下

![image-20220213132344456](https://i-blog.csdnimg.cn/blog_migrate/b6d46797f2a8d69048da21a08de16494.png)

因为我们在根项目中是不会进行任何编码的，所以我们可以将根项目中的src目录删除

最终我们的代码是需要上传到git的，所以创建`.gitignore`文件，将`target/`排除

我们创建一个helloworld的模块，验证sbt下的scala是否可用

![image-20220123203026273](https://i-blog.csdnimg.cn/blog_migrate/090c72d33bb1e389853fa10f9b0e3fbe.png)

![image-20220213132439437](https://i-blog.csdnimg.cn/blog_migrate/60fa55e6ba1d35365481cd8aed440adb.png)

别忘记在build.sbt中定义helloworld项目
    
    
    lazy val helloworld = (project in file("helloworld"))
      .settings(
        name := "helloworld",
        commonSettings
      )
    

别忘记创建目录`main,test,scala,java,resources`等.

创建好后，我们创建如下代码

![image-20220213144327277](https://i-blog.csdnimg.cn/blog_migrate/a1c78c90b8e9e90e4630c55204eb9ec9.png)

在运行之前需要先执行compile操作

你可以在sbt-shell中执行

![image-20220213140106153](https://i-blog.csdnimg.cn/blog_migrate/e45311718000256836fb06c5e57f7a16.png)

也可以在sbt工具窗口中执行(我比较喜欢在sbt-shell中操作)

![image-20220213140139499](https://i-blog.csdnimg.cn/blog_migrate/17ab0f18c418f8d3d7a97568306c310b.png)

点击运行

![image-20220123212342309](https://i-blog.csdnimg.cn/blog_migrate/06883d3a4d8baf18ab7ab629168aa02c.png)

但是现在我们只是能在ide中执行，我们的目的是打出jar包的，所以我们还需要配置`sbt-assembly`插件

打开`sbt-assembly`插件的官方文档[sbt-assembly (scala-lang.org)](<https://index.scala-lang.org/sbt/sbt-assembly/sbt-assembly/1.1.1?target=_2.12_1.0>)
    
    
    lazy val helloworld = (project in file("helloworld"))
      .settings(
        name := "helloworld",
        // 定义主类
    	assembly / mainClass := Some(idePackagePrefix.value.get + ".Hello"),
        // 定义jar包名 项目名字_scala版本_项目版本.jar
        assembly / assemblyJarName := name.value + "_" + scalaVersion.value + "_" + version.value + ".jar",
        // 依赖合并策略
        assemblyMergeStrategy := {
          case PathList("javax", "servlet", xs @ _*)         => MergeStrategy.first
          case PathList(ps @ _*) if ps.last endsWith ".html" => MergeStrategy.first
          case "application.conf"                            => MergeStrategy.concat
          case "unwanted.txt"                                => MergeStrategy.discard
          case x =>
            val oldStrategy = (ThisBuild / assemblyMergeStrategy).value
            oldStrategy(x)
        },
        commonSettings
      )
    

然后执行`assembly`操作,记得切换到helloworld项目下，你也可以在sbt工具窗口执行

![image-20220213144704313](https://i-blog.csdnimg.cn/blog_migrate/45fcf7673a42dd5fdb3fe10d2b215417.png)

不过我喜欢在sbt-shell中执行

![image-20220213144554554](https://i-blog.csdnimg.cn/blog_migrate/5659bf99dedd4d7587a2973fe9c0edbb.png)

然后耐心的等待吧，第一次会比较慢的

![image-20220213150049914](https://i-blog.csdnimg.cn/blog_migrate/607accdb36f8daf953986e2f9841d741.png)

可以看到jar包已经生成了

![image-20220213150109947](https://i-blog.csdnimg.cn/blog_migrate/42124f5750552124d2a4ac0f271d069c.png)

我们使用java -jar 执行

![image-20220213150621695](https://i-blog.csdnimg.cn/blog_migrate/9c49f8880d94e61b3a7e57a0c7396f83.png)

可以看到打出来的jar包成功执行了

到了这里scala和sbt都集成完毕，此时需要集成spark了

spark的依赖下载会比较慢，而且新版本国内的镜像不一定有，需要从国外下载

![image-20220123231552302](https://i-blog.csdnimg.cn/blog_migrate/d76261a5df091d9392c316737aa14ea8.png)

这里推荐一个Idea的插件：`Big Data Tools`

![image](https://i-blog.csdnimg.cn/blog_migrate/ca12d373d39a6aab75e5f86f66bc7b3b.png)

它可以连接到远程的hadoop,hdfs,spark的集群上，用于监控和交互，安装后重启idea会在下面出现工具窗口

![image-20220127154515533](https://i-blog.csdnimg.cn/blog_migrate/d581483d3c86c508244401c416b47bf3.png)

支持的还是挺多的

![image-20220127154653531](https://i-blog.csdnimg.cn/blog_migrate/371bb9f10510fa8e1d85a3c04a62567f.png)

首先我们将插件和hadoop01进行连接

![image-20220127164004319](https://i-blog.csdnimg.cn/blog_migrate/236a43b9670a27cc317716d3dc051317.png)

展示如下，和我们在浏览器查看的效果是一样的

![image-20220127164032048](https://i-blog.csdnimg.cn/blog_migrate/181e597f4fe2d9fa518689760fdd6d04.png)

连接hdfs,需要注意，填写的hdfs的通信地址，不是浏览器地址

![image-20220127164819223](https://i-blog.csdnimg.cn/blog_migrate/b979c9de057de3a9194d41cf51e07677.png)

连接上之后，就可以像操作idea里面的项目文件一样，上传下载等

![image-20220127164949014](https://i-blog.csdnimg.cn/blog_migrate/5a2e85cca646033b27b1b86d416c0e4e.png)

接着我们启动hadoop01的spark集群

![image-20220127170945385](https://i-blog.csdnimg.cn/blog_migrate/3c19e2d4360bed0e13d8659c70dad56b.png)

可惜了，连接spark还是有问题

![image-20220127170929039](https://i-blog.csdnimg.cn/blog_migrate/a0e246ae80e0fad151eafc8191857920.png)

**到了这里，工具装了一大堆，但是还没开始开发一行代码，接下来就开发一个小案例wordcount试试吧**

当我们安装了`big data tools`插件后，新建module的时候，就可以选择`big data tools`下的`spark`模板了

![image-20220127172046605](https://i-blog.csdnimg.cn/blog_migrate/fbdc8d6b64d51fa1c6ab9869e2ae5db0.png)

这是一个类似于spring boot的模板，里面自动创建一些文件和配置，更准确应该是和maven比较像

![image-20220127180821899](https://i-blog.csdnimg.cn/blog_migrate/fbb5103a6f2bdeb3ae316274cb2ebb35.png)

加载完毕后项目结构如下

![image-20220213150830026](https://i-blog.csdnimg.cn/blog_migrate/00dd8af59d6f51fd01d368774367c2fe.png)

不过这种创建项目只是适合创建顶级项目，对于子级项目是不行的，会覆盖已有的build.sbt等信息的

我们前面已经搭建好了顶级项目，所以我们可以直接创建一个scala项目就行了

![image-20220213152901077](https://i-blog.csdnimg.cn/blog_migrate/da473c4b7da9b50eae86035ada84ab9c.png)

然后指定模块的名字就行了

![image-20220213152934987](https://i-blog.csdnimg.cn/blog_migrate/5494c24f090198c3fa72c85f4806e45f.png)

我们做个优化，把打包的配置，除了主类之外的，都放在公共配置中
    
    
    // 抽取公共配置
    lazy val commonSettings = Seq(
      version := "1.0",
      scalaVersion := "2.12.15",
      sbtVersion := "1.6.1",
      sparkVersion := "3.2.0",
      // 定义项目包基础
      packageBaseDir := "com.study.spark",
      // 定义每个项目的包前缀
      ThisProject / idePackagePrefix := Some(packageBaseDir.value + "." + name.value),
      // 定义输出目录
      ideOutputDirectory := Some(file("target")),
      organization := "com.study.spark",
      // 申明源码路径
      sourceDirectories := Seq(
        file("src/main/scala"),
        file("src/main/java"),
        file("src/test/scala"),
        file("src/test/java")
      ),
      // 声明资源路径
      resourceDirectories := Seq(
        file("src/main/resources"),
        file("src/test/resources")
      ),
      libraryDependencies ++= Seq(
        "org.apache.spark" %% "spark-core" % sparkVersion.value % "provided",
        "org.apache.spark" %% "spark-sql" % sparkVersion.value % "provided",
        "org.apache.spark" %% "spark-mllib" % sparkVersion.value % "provided",
        "org.apache.spark" %% "spark-streaming" % sparkVersion.value % "provided"
      ),
      // 定义jar包名 项目名字_scala版本_项目版本.jar
      ThisProject / assembly / assemblyJarName := name.value + "_" + scalaVersion.value + "_" + version.value + ".jar",
      // 依赖合并策略
      ThisBuild / assemblyMergeStrategy := {
        case PathList("javax", "servlet", xs @ _*)         => MergeStrategy.first
        case PathList(ps @ _*) if ps.last endsWith ".html" => MergeStrategy.first
        case "application.conf"                            => MergeStrategy.concat
        case "unwanted.txt"                                => MergeStrategy.discard
        case x =>
          val oldStrategy = (ThisBuild / assemblyMergeStrategy).value
          oldStrategy(x)
      },
    )
    

然后把wordcount加入到根项目中，主类先不写
    
    
    lazy val wordcount = (project in file("wordcount"))
      .settings(
        name := "wordcount",
        commonSettings
      )
    

然后创建目录

![image-20220213153343245](https://i-blog.csdnimg.cn/blog_migrate/541954961fb0fe293613bfd002255738.png)

我们在scala目录下创建主类

![image-20220213153625245](https://i-blog.csdnimg.cn/blog_migrate/1623cfd465da056dbf8f99870494645b.png)

首先使用本地进行执行
    
    
    package com.study.spark.wordcount
    
    import org.apache.spark.{SparkConf, SparkContext}
    
    object WordCount {
    
      def main(args: Array[String]): Unit = {
        // 配置spark
        val conf = new SparkConf().setMaster("local").setAppName("wordcount")
        // 获取sparkContext
        val sc = new SparkContext(conf)
        // 从本地读取文件，其实就是 build.sbt 文件
        sc.textFile("file:///E:\\java\\studyspark\\build.sbt")
          // 按照空格拆分
          .flatMap(_.split(" "))
          // 每个单词计数1
          .map(_->1)
          // 按单词分组统计
          .reduceByKey(_+_)
          // 计算
          .collect()
          // 打印统计结果
          .foreach(println(_))
      }
    
    }
    

别忘记在build.sbt中配置主类(之前我们只有一个有效的项目，所以我们可以直接取参数的值，但是现在有了多个项目，就需要在取值的时候，限定作用域)

![image-20220213154637710](https://i-blog.csdnimg.cn/blog_migrate/82426b5ea537c83ebc48c019759dbc18.png)

需要注意的是我们配置的spark依赖是不会在classpath中使用的，因为默认当我们把spark任务的jar包提交给spark集群的时候，spark相关的依赖是不需要在jar包中的。

但是当我们需要本地启动的时候，如果没有spark的依赖，会导致spark相关的类找不到。

所以我们需要在本地启动的spark项目中，再次配置spark的依赖，区别是不写后面`provided`作用域

![image-20220213155250683](https://i-blog.csdnimg.cn/blog_migrate/617f105cff4984cde1c6e30ff6d2acee.png)

然后切换到wordcount项目下，编译并执行(使用run执行，而不是使用ide的执行按钮)

![image-20220213155341626](https://i-blog.csdnimg.cn/blog_migrate/43accb71b2683d1b458b0093a23fd957.png)

如果一切顺利，就会看到执行的日志

![image-20220213155403722](https://i-blog.csdnimg.cn/blog_migrate/d9b250239da91429e43cd8b5c821dfa4.png)

我们也可以打包的，使用`sbt-assembly`打包

![image-20220213155504796](https://i-blog.csdnimg.cn/blog_migrate/df2394e51daf17240d9d71e762c832ba.png)

然后使用java -jar 执行

执行会抛出异常，这是因为缺少了scala的依赖，我们需要把scala的依赖也加入

![image-20220213162136720](https://i-blog.csdnimg.cn/blog_migrate/24562309c63d42bbff6d1335cbbb4e9c.png)

我们要求scala的依赖每次都必须打包，注意这里是一个`%`

打出来的jar包不能简单的使用java -jar 运行，需要提交到spark环境中执行，因为我们在打包的时候并没有打spark相关的包。

为了能在服务器上使用，我们需要修改我们的代码，需要链接到服务器上的spark集群，而且读取文件应该读取hdfs中的文件

首先启动服务器上的hdfs服务

![image-20220213182838050](https://i-blog.csdnimg.cn/blog_migrate/28a60c9618f8821acfb8fec97c35ec85.png)

接着启动spark集群

![image-20220213182936044](https://i-blog.csdnimg.cn/blog_migrate/2fb3b3559832eb9ff07728223f24d825.png)

接着把build.sbt上传到hdfs中

![image-20220213183221032](https://i-blog.csdnimg.cn/blog_migrate/a9d54ed6f0c47c812ebf64c08c4a4225.png)

然后重新打包，并上传到服务器

![image-20220213202524249](https://i-blog.csdnimg.cn/blog_migrate/f99d2ede64c7b9d314b7e79f31b91df1.png)

然后使用
    
    
    spark-submit --master spark://hadoop01:4040 --class com.study.spark.wordcount.WordCount /spark/wordcount_2.12.15_1.0.jar
    

提交到spark集群，spark就开始执行了

![image-20220213202642189](https://i-blog.csdnimg.cn/blog_migrate/097c2a5c34092da2ed1518451e3e6c09.png)

执行结果和在本地执行一模一样

![image-20220213202706113](https://i-blog.csdnimg.cn/blog_migrate/183189a167a6ba243d044c6187ee7b42.png)

而且在spark的界面中也能看到

![image-20220213202735174](https://i-blog.csdnimg.cn/blog_migrate/cdd8728d0c88f2ba9ed14d2565c89993.png)

如果你还没有配置历史服务，那么会启动报错的，提示没有配置历史数据存储路径

![image-20220213204501880](https://i-blog.csdnimg.cn/blog_migrate/2c70d65f94a5f1c7cf203990a6eeaf44.png)

拷贝spark-defaults.conf.template为spark-defaults.conf

![image-20220213204431531](https://i-blog.csdnimg.cn/blog_migrate/5cb64c0916e17a71a91b44263f0af9e9.png)

内容如下
    
    
    # 是否开启日志
    spark.eventLog.enabled true
    # 日志存储路径
    spark.eventLog.dir hdfs://hadoop01:8020/logs
    # 是否压缩
    spark.eventLog.compress true
    

同时在spark-env.sh中配置以下内容
    
    
    # 配置spark历史服务
    SPARK_HISTORY_OPTS="-Dspark.history.ui.port=8086 -Dspark.history.retainedApplications=10 -Dspark.history.fs.logDirectory=hdfs://hadoop01:8020/logs"
    

我们启动spark历史记录服务

![image-20220213203008158](https://i-blog.csdnimg.cn/blog_migrate/9c8778b87ccb73a253cb4e7cbd5fb9ca.png)

启动历史记录服务不报错

![image-20220213205442039](https://i-blog.csdnimg.cn/blog_migrate/2cdc80501889b396d3a671bd4eccec3a.png)

而且界面能打开

![image-20220213205510642](https://i-blog.csdnimg.cn/blog_migrate/b9a6a8b355072d1211c36499585b3c21.png)

然后在执行一次，因为我们的任务很快，所以开启历史记录，方便我们查看job的信息

这时候就能在历史界面查看了

![image-20220213205630095](https://i-blog.csdnimg.cn/blog_migrate/a01d6219eb0fe6b2a59fd363302e40e2.png)

![image-20220213205650355](https://i-blog.csdnimg.cn/blog_migrate/4134cfbf587a2dcf3e30f8f28d787adb.png)

我们实际上是想看看dag图

![image-20220213205808647](https://i-blog.csdnimg.cn/blog_migrate/c71dbb70ff3ebe0f0494289d0136ac18.png)

很漂亮。

## 源码编译的3.2.0版本无法在window上直接用spark-shell启动

这个问题我发现不仅仅是我们自己编译的二进制分发包有问题，就连官方下载的二进制分发包，如果你直接使用`spark-shell`启动，也是会无法启动的。这个问题困扰我了好几天，我在网上找到了相关的讨论[windows - Spark illegal character in path - Stack Overflow](<https://stackoverflow.com/questions/69669524/spark-illegal-character-in-path>)

但是也没给出问题原因和解决方案，只是说降低版本。

之前是源码编译二进制包，所以我也有spark的源码，我自己根据堆栈找了好长时间，奈何自己水平太低，从源码中没有找到相关的代码。

不过从我自己理解的源码来看，目前好像就是`spark-shell`的repl的启动存在问题，而提交作业等貌似是没有问题的。(我自己没试)

## 总结

整个文章的过程比较坎坷，比较艰难。

我是在看书的时候，知道了scala的sdk的二进制不兼容，以及目前市面上大多数公司使用cdh的发行包，于是我也到cdh的官网想下载最新的发行包，结果发现cdh版本貌似也开始收费了，于是只能自己使用源码编译。

源码编译时第一个阶段，当我历经艰难，完成了源码编译后，发现不能在windows上使用spark-shell，于是在服务器集群上搭建spark集群。这里主要学习了spark的启动和集群部署，以及一些配置等。

上述这两个阶段基本上是年前完成的，在搭建spark开发环境的时候，才知道spark或者说scala推荐使用sbt编译工具，可怜我maven都不是很熟悉，我哪会sbt呢。没辙，想办法研究sbt，研究sbt刚开始，就过年了，中间大概10多天吧，就是啥都没做，新年开工，才开始正式研究sbt，研究几天，基本上做到了入门，就继续在idea中集成spark，sbt环境。

最终实现了idea本地开发编码，本地在sbt的基础上执行，然后打包提交到spark集群中执行。

整个过程还是比较长的，但是收获也很多。

当然，我自己深深的知道，自己对上面这些，也仅仅是入门。还需努力，少年！
