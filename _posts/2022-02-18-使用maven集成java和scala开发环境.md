---
layout: post
title: "使用maven集成java和scala开发环境"
date: 2022-02-18 01:11:43 +0800
categories: [scala, java, maven, maven插件使用, scala开发环境搭建]
description: "本文档详细介绍了如何使用Maven搭建一个包含Scala和Spark的开发环境。首先创建一个普通的Maven项目，然后在父项目中配置Scala依赖，并在子项目中引入。接着配置Scala插件，安装Scala SDK，编写并运行Scala的HelloWorld程序。随后，配置Maven的各种插件，如maven-compile-plugin、maven-scala-plugin、maven-jar-plugin、maven-dependency-plugin和maven-assembly-plugin，用于编译、打包和依赖管理。最后，通过maven-assembly-plugin创建可执行的jar包，并演示了如何在Spark集群上运行该程序。"
keywords: scala, java, maven, maven插件使用, scala开发环境搭建
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/122995011
> - 发布时间：2022-02-18 01:11:43
> - 阅读量：3
> - 分类：scala同时被 3 个专栏收录, 订阅专栏, 大数据, spark
> - 标签：#scala, #java, #maven, #maven插件使用, #scala开发环境搭建

## 摘要

文章浏览阅读3.3k次，点赞4次，收藏27次。本文档详细介绍了如何使用Maven搭建一个包含Scala和Spark的开发环境。首先创建一个普通的Maven项目，然后在父项目中配置Scala依赖，并在子项目中引入。接着配置Scala插件，安装Scala SDK，编写并运行Scala的HelloWorld程序。随后，配置Maven的各种插件，如maven-compile-plugin、maven-scala-plugin、maven-jar-plugin、maven-dependency-plugin和maven-assembly-plugin，用于编译、打包和依赖管理。最后，通过maven-assembly-plugin创建可执行的jar包，并演示了如何在Spark集群上运行该程序。

---

#### 使用maven集成java和scala开发环境

  * 创建项目
  * 增加scala依赖
  * 创建目录
  * 安装scala插件
  * scala的hello world
  * maven 插件
  *     * 配置仓库
    * maven-compile-plugin
    * maven-scala-plugin
    * maven-jar-plugin
    * maven-dependency-plugin
    * maven-assembly-plugin
  * spark 开发环境

  
git地址：https://gitee.com/jyq_18792721831/sparkmaven.git 

## 创建项目

我们首先创建一个普通的maven项目

![image-20220217205350430](https://i-blog.csdnimg.cn/blog_migrate/28f94b45003d52c609dd4279700eaee8.png)

创建项目后接着创建一个hello的模块

![image-20220217212046739](https://i-blog.csdnimg.cn/blog_migrate/4ef32bd7650b667a2f1dbe29497dcb12.png)

也是普通的maven模块

## 增加scala依赖

我们在父项目中是不会写代码的，父项目只是为了管理子项目的，所以父项目的src目录可以删除

我们在父项目的pom.xml中增加依赖

![image-20220217212546275](https://i-blog.csdnimg.cn/blog_migrate/ea5ae992aa59f4bf87e16ce1fe00f7cc.png)

如下
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>com.study.sparkmaven</groupId>
        <artifactId>sparkmaven</artifactId>
        <packaging>pom</packaging>
        <version>1.0</version>
        <modules>
            <module>hello</module>
        </modules>
    
        <properties>
            <maven.compiler.source>11</maven.compiler.source>
            <maven.compiler.target>11</maven.compiler.target>
            <!-- scala 的版本 -->
            <version.scala>2.12.15</version.scala>
        </properties>
    
        <dependencyManagement>
            <dependencies>
                <!-- scala 库 -->
                <dependency>
                    <groupId>org.scala-lang</groupId>
                    <artifactId>scala-library</artifactId>
                    <version>${version.scala}</version>
                </dependency>
                <!-- scala 编译 -->
                <dependency>
                    <groupId>org.scala-lang</groupId>
                    <artifactId>scala-compiler</artifactId>
                    <version>${version.scala}</version>
                </dependency>
                <!-- scala 映射 -->
                <dependency>
                    <groupId>org.scala-lang</groupId>
                    <artifactId>scala-reflect</artifactId>
                    <version>${version.scala}</version>
                </dependency>
            </dependencies>
        </dependencyManagement>
    </project>
    

接着在hello模块的pom.xml中将scala相关的依赖引入

![image-20220217212703355](https://i-blog.csdnimg.cn/blog_migrate/873cccf0ec978b5bd51b35bcb0222003.png)

## 创建目录

我们需要在src目录下创建我们的源码目录和资源目录

![image-20220217212809726](https://i-blog.csdnimg.cn/blog_migrate/1477a864cd99838575fd2600cc13f2cf.png)

并使用右键标注为源码和资源

![image-20220217212837463](https://i-blog.csdnimg.cn/blog_migrate/898539bdfa0c84eecdaabeb11bea2344.png)

标注完成后如下

![image-20220217212910013](https://i-blog.csdnimg.cn/blog_migrate/cd2b06b41a154dd392f87c254c6558e5.png)

接着创建我们的包目录

![image-20220217212952030](https://i-blog.csdnimg.cn/blog_migrate/1806ccfc2a8204b72d701fa72292c62e.png)

## 安装scala插件

首先打开设置

![image-20220217213448727](https://i-blog.csdnimg.cn/blog_migrate/ad38c56455e0a2c7cb120fcf95c7412c.png)

在插件处查询scala插件并安装，可能需要多试几次

![image-20220217213710554](https://i-blog.csdnimg.cn/blog_migrate/af804e295343fa13b1c40114e3e0c9ab.png)

而且scala插件比较大的，下载不一定能一次成功。如果确实无法下载，可以去idea的插件市场中离线下载，然后离线安装。

## scala的hello world

我们在scala的源码下进行开发

此时在新建文件的时候，是找不到scala的选项的

![image-20220217213051612](https://i-blog.csdnimg.cn/blog_migrate/91b6fdcb883d127428dfff5acc06583d.png)

我们刷新整个maven项目，让maven下载依赖

![image-20220217213135498](https://i-blog.csdnimg.cn/blog_migrate/432545e47b027f913f484cd67ba64542.png)

当然，刷新完还是无法创建scala的项目

![image-20220217213201116](https://i-blog.csdnimg.cn/blog_migrate/34462bb46179e274d5258bdd55273de0.png)

我们需要告诉idea，我们的项目需要支持scala，所以我们需要把scala的sdk加入

![image-20220217213234951](https://i-blog.csdnimg.cn/blog_migrate/22dabd9ce41747f01992494920be1ccb.png)

首先保证你的全局的sdk有scala，如果没有需要点击+增加

![image-20220217213318784](https://i-blog.csdnimg.cn/blog_migrate/1ab3136a8f42266d5093e54b156edc11.png)

比如

![image-20220217213338109](https://i-blog.csdnimg.cn/blog_migrate/9577108808050dae5ad486ff68878aae.png)

当然，最最前提是你需要安装scala的插件，只有安装了scala的插件，才能开发与scala有关的内容。

我们在模块设置将scala加入

![image-20220217213907139](https://i-blog.csdnimg.cn/blog_migrate/1c7df198fd54fe92ab59665c1db9d54d.png)

选择我们的scala版本的sdk,如果你有多个版本的scala的sdk，一定注意版本

![image-20220217213937037](https://i-blog.csdnimg.cn/blog_migrate/caec43a0410e6065614c0dcffaf4119e.png)

我们也可以把scala的sdk加入到根项目中

![image-20220217214058338](https://i-blog.csdnimg.cn/blog_migrate/6932b4989a63936ae71e37a20a26c8ac.png)

当然，你在加入到根项目中后，在子项目中还是需要增加一次

我们这里增加实际上与maven增加scala的依赖是没有任何冲突的

我们在这里增加依赖，只是告诉idea，在我们的项目中需要用到scala的一些功能而已

![image-20220217214243988](https://i-blog.csdnimg.cn/blog_migrate/0bec2b2ddb97048ade7f97fe79d6207e.png)

我们选择增加一个object，写如下内容
    
    
    package com.study.sparkmaven.hello
    
    object Hello {
    
      def main(args: Array[String]): Unit = {
        println("hello scala")
      }
    
    }
    

然后点击运行

![image-20220217214356937](https://i-blog.csdnimg.cn/blog_migrate/e5af3803a36ec7a2758a30161a698d9c.png)

如下

![image-20220217214432952](https://i-blog.csdnimg.cn/blog_migrate/c3153d37546540aedf1075394287bd8d.png)

## maven 插件

我们虽然开发了scala的hello world，但是这没啥意义，只是说明我们的idea目前支持了scala的语法，我们还需要安装一些maven的插件，用于帮助我们做其他的处理。

还是和依赖一样的管理方式，我们在父项目的pom.xml中定义依赖和插件，包括一些插件的通用配置，然后在子项目中增加子项目个性化的依赖，以及子项目个性化的插件配置。

约定所有的版本号都在`properties`中统一配置。

插件都在`build-pluginManage-plugins`下配置

### 配置仓库

如果不配置仓库，默认是从maven的中央仓库下载依赖和插件，会比较慢，我们可以配置国内镜像，加快下载速度
    
    
        <!-- 私有仓库配置 -->
        <pluginRepositories>
            <pluginRepository>
                <id>maven-net-cn</id>
                <name>Maven China Mirror</name>
                <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
            </pluginRepository>
        </pluginRepositories>
        <repositories>
            <repository>
                <id>central</id>
                <name>Maven China Mirror</name>
                <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
            </repository>
        </repositories>
    

我们在父项目的pom.xml中配置即可。

### maven-compile-plugin

我们在父项目的pom.xml中加入maven-compile-plugin插件

首先定义插件的版本
    
    
        <properties>
            <maven.compiler.source>11</maven.compiler.source>
            <maven.compiler.target>11</maven.compiler.target>
            <!-- scala 的版本 -->
            <version.scala>2.12.15</version.scala>
            <!-- maven-compile-plugin 的版本号 -->
            <version.maven.compile.plugin>3.9.0</version.maven.compile.plugin>
        </properties>
    

接着配置插件
    
    
    <build>
            <pluginManagement>
                <plugins>
                    <!-- maven-compile-plugin 插件 -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <version>${version.maven.compile.plugin}</version>
                        <!-- 配置信息 -->
                        <configuration>
                            <!-- 源码 -->
                            <source>${maven.compiler.source}</source>
                            <target>${maven.compiler.target}</target>
                            <!-- 编码方式 -->
                            <encoding>UTF-8</encoding>
                            <!-- 支持调试 -->
                            <debug>true</debug>
                        </configuration>
                    </plugin>
                </plugins>
            </pluginManagement>
        </build>
    

或许你可能会比较疑惑，我们怎么知道我可以使用哪些插件，以及这些插件的版本是什么呢？  
你可以在[Maven – Available Plugins (apache.org)](<https://maven.apache.org/plugins/index.html>)查询所有可用的插件，点击插件名字可以进入到插件的文档界面，里面会有详细的版本号，以及如何使用等信息

![image-20220217222140540](https://i-blog.csdnimg.cn/blog_migrate/f52eef9cefd2f9a9722f16c6867e9826.png)

详细信息

![image-20220217222242540](https://i-blog.csdnimg.cn/blog_migrate/3241723c4884679c2bd594b2dbb44936.png)

千万记得，我们在父项目中配置插件后，还需要在子项目中引入

我们在hello项目的pom.xml中引入

![image-20220217222706562](https://i-blog.csdnimg.cn/blog_migrate/542890aebd4baf51c58f3548ba476c29.png)

这样做的好处是在一个pom.xml中统一管理项目中的插件的版本等信息。一些通用的配置也可以在父项目的pom.xml中配置。

### maven-scala-plugin

我们接着配置scala的插件

定义版本
    
    
            <!-- maven-scala-plugin 的版本号 -->
            <version.maven.scala.plugin>2.15.2</version.maven.scala.plugin>
    

接着引入和配置插件
    
    
    <plugin>
                        <groupId>org.scala-tools</groupId>
                        <artifactId>maven-scala-plugin</artifactId>
                        <version>${version.maven.scala.plugin}</version>
                        <configuration>
                            <!-- scala的版本号 -->
                            <scalaVersion>${version.scala}</scalaVersion>
                        </configuration>
                        <!-- 配置监听器 -->
                        <executions>
                            <!-- 监听器 -->
                            <execution>
                                <!-- 如果有多个监听器则必须设置id，而且不能重复 -->
                                <id>scala-compile</id>
                                <!-- 监听的操作 -->
                                <phase>compile</phase>
                                <!-- 监听器触发后执行的操作 -->
                                <goals>
                                    <goal>compile</goal>
                                </goals>
                            </execution>
                            <execution>
                                <id>scala-test-compile</id>
                                <phase>test-compile</phase>
                                <goals>
                                    <goal>testCompile</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
    

接着在子项目中引入插件，并配置执行(子项目的pom.xml)
    
    
     <plugin>
                    <groupId>org.scala-tools</groupId>
                    <artifactId>maven-scala-plugin</artifactId>
                    <!-- 子项目个性化配置 -->
                    <configuration>
                        <!-- 配置执行 -->
                        <launchers>
                            <!-- 可以配置多个执行 -->
                            <launcher>
                                <!-- 可以使用 run id 选择不同的执行 -->
                                <id>hello</id>
                                <!-- 定义执行的入口,这个是核心 -->
                                <mainClass>com.study.sparkmaven.hello.Hello</mainClass>
                                <!-- 执行的附加参数等 -->
                                <jvmArgs>
                                    <jvmArg>-Xmx128m</jvmArg>
                                    <jvmArg>-Djava.library.path=...</jvmArg>
                                </jvmArgs>
                            </launcher>
                        </launchers>
                    </configuration>
                </plugin>
    

我们此时刷新maven

如果一切正常，此时会有scala的一些操作

![image-20220217223657750](https://i-blog.csdnimg.cn/blog_migrate/d2c677fcbd912854830c58e407df87b8.png)

因为我们只配置了一个scala的执行，所以可以直接使用这个run，在只有一个执行的时候，是不需要指定执行的id，如果有多个，不指定执行id，也是会取第一个。

我们双击运行`scala:run`

![image-20220217223846885](https://i-blog.csdnimg.cn/blog_migrate/fe97fb253d55be010f8dda49a0d2af1f.png)

第一次会比较慢，因为需要下载依赖。

此时虽然可以运行，但是也是基于idea的，如果我们直接打jar包，并在java环境运行是不行的。

会提示我们没有配置主类

![image-20220217224255908](https://i-blog.csdnimg.cn/blog_migrate/b83069077fbd3769f709def8af150dae.png)

### maven-jar-plugin

这个插件就是用于指定主类的，打包出来的jar包可以运行

指定版本
    
    
            <!-- maven-jar-plugin 的版本号 -->
            <version.maven.jar.plugin>3.2.2</version.maven.jar.plugin>
    

配置插件
    
    
    <!-- maven-jar-plugin -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-jar-plugin</artifactId>
                        <version>${version.maven.jar.plugin}</version>
                        <executions>
                            <execution>
                                <!-- 监听打包操作 -->
                                <phase>package</phase>
                                <goals>
                                    <!-- 打包后执行 jar 操作 -->
                                    <goal>jar</goal>
                                </goals>
                                <configuration>
                                    <!-- 生成的jar包的附加字段，为了防止重复 -->
                                    <classifier>client</classifier>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
    

子项目中引入
    
    
    <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-jar-plugin</artifactId>
                    <configuration>
                        <archive>
                            <manifest>
                                <!-- 增加运行时环境 -->
                                <addClasspath>true</addClasspath>
                                <!-- 指定依赖的位置为lib目录 -->
                                <classpathPrefix>lib/</classpathPrefix>
                                <!-- 指定主类 -->
                                <mainClass>com.study.sparkmaven.hello.Hello</mainClass>
                            </manifest>
                        </archive>
                    </configuration>
                </plugin>
    

我们刷新项目，并clean后执行package

![image-20220217225039204](https://i-blog.csdnimg.cn/blog_migrate/e1f889e69c020c2c5d59cb82535c0e1f.png)

执行后会生成两个jar包

![image-20220217225111633](https://i-blog.csdnimg.cn/blog_migrate/bc83a45852995cc3a9711760cb2d03b0.png)

第二个jar包就是插件生成的，尝试执行

![image-20220217225201350](https://i-blog.csdnimg.cn/blog_migrate/842f514c5aa6f187148e7402e5bef41b.png)

我们依次执行两个jar包，发现还是无法执行，但是至少不是找不到主类的错误了，而是找不到scala的相关库。

这是因为我们虽然指定了依赖在`lib`目录，但是现在lib目录可是空的。

也就是没有依赖。

此时就需要下面这个插件了。

### maven-dependency-plugin

依赖拷贝插件，可以将项目的依赖拷贝到指定位置，为打包做准备。

前面我们打包，因为`lib`目录为空，导致打包的jar中缺少相关的依赖。

我们首先定义版本
    
    
            <!-- maven-dependency.plugin 的版本号 -->
            <version.maven.dependency.plugin>3.2.0</version.maven.dependency.plugin>
    

接着配置插件
    
    
    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-dependency-plugin</artifactId>
                        <version>${version.maven.dependency.plugin}</version>
                        <executions>
                            <execution>
                                <id>dependency-copy</id>
                                <!-- 监听打包操作 -->
                                <phase>package</phase>
                                <goals>
                                    <!-- 打包操作前执行依赖拷贝操作 -->
                                    <goal>copy-dependencies</goal>
                                </goals>
                                <configuration>
                                    <!-- 依赖拷贝的目的目录 -->
                                    <outputDirectory>${project.build.directory}/lib</outputDirectory>
                                    <!-- 是否可以覆盖release版本的依赖 -->
                                    <overWriteReleases>false</overWriteReleases>
                                    <!-- 是否可以覆盖snapshots版本的依赖 -->
                                    <overWriteSnapshots>false</overWriteSnapshots>
                                    <!-- 是否可以覆盖，当有新版本时 -->
                                    <overWriteIfNewer>true</overWriteIfNewer>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
    

我们在子项目中引入
    
    
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-dependency-plugin</artifactId>
                </plugin>
    

因为依赖拷贝算是一个通用的，所以这里把相关的设置放到父项目中的pom.xml中了

刷新maven,先后执行clean和package

![image-20220217230255368](https://i-blog.csdnimg.cn/blog_migrate/001b61238dcd9e8c8fcab8e8067f4403.png)

接着尝试执行jar包

![image-20220217230335843](https://i-blog.csdnimg.cn/blog_migrate/f9ce0dfd97afc00c905788952c3b5aca.png)

### maven-assembly-plugin

在上面中，实际上是两个插件配合使用，第一个插件用于拷贝依赖，第二个插件用于打包。

但是打包只是能打包jar包，如果你想打war包，或者其他的就不行了。

所以还有一个插件，不仅仅能打jar包，还能打其他的包，而且是集成了这两个插件的功能。

就是大名鼎鼎的`assembly`插件。

版本定义
    
    
            <!-- maven-assembly-plugin 的版本号 -->
            <version.maven.assembly.plug>3.3.0</version.maven.assembly.plug>
    

插件定义
    
    
    <!-- maven-assembly-plugin -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-assembly-plugin</artifactId>
                        <version>${version.maven.assembly.plug}</version>
                        <!-- 配置 -->
                        <configuration>
                            <!-- 启动的功能 -->
                            <descriptorRefs>
                                <!-- 打jar包自动拷贝依赖 -->
                                <descriptorRef>jar-with-dependencies</descriptorRef>
                            </descriptorRefs>
                        </configuration>
                        <executions>
                            <!-- 监听器 -->
                            <execution>
                                <id>make-assembly</id>
                                <!-- 监听 打包命令 -->
                                <phase>package</phase>
                                <goals>
                                    <!-- 目前只有这个操作 -->
                                    <goal>single</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
    

子项目依赖，我们去掉之前的`maven-jar-plugin`和`maven-dependency-plugin`插件
    
    
    <!--            <plugin>-->
    <!--                <groupId>org.apache.maven.plugins</groupId>-->
    <!--                <artifactId>maven-jar-plugin</artifactId>-->
    <!--                <configuration>-->
    <!--                    <archive>-->
    <!--                        <manifest>-->
    <!--                            &lt;!&ndash; 增加运行时环境 &ndash;&gt;-->
    <!--                            <addClasspath>true</addClasspath>-->
    <!--                            &lt;!&ndash; 指定依赖的位置为lib目录 &ndash;&gt;-->
    <!--                            <classpathPrefix>lib/</classpathPrefix>-->
    <!--                            &lt;!&ndash; 指定主类 &ndash;&gt;-->
    <!--                            <mainClass>com.study.sparkmaven.hello.Hello</mainClass>-->
    <!--                        </manifest>-->
    <!--                    </archive>-->
    <!--                </configuration>-->
    <!--            </plugin>-->
    <!--            <plugin>-->
    <!--                <groupId>org.apache.maven.plugins</groupId>-->
    <!--                <artifactId>maven-dependency-plugin</artifactId>-->
    <!--            </plugin>-->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-assembly-plugin</artifactId>
                    <configuration>
                        <archive>
                            <manifest>
                                <addClasspath>true</addClasspath>
                                <classpathPrefix>lib/</classpathPrefix>
                                <mainClass>com.study.sparkmaven.hello.Hello</mainClass>
                            </manifest>
                        </archive>
                    </configuration>
                </plugin>
    

刷新项目并执行clean和package

![image-20220217231510243](https://i-blog.csdnimg.cn/blog_migrate/e33313ba3ac79f9fa875e700bec12fa1.png)

我们尝试执行

![image-20220217231554880](https://i-blog.csdnimg.cn/blog_migrate/1a1ece4791fe1738d8828696c3fadb9f.png)

因为我们注释了`maven-jar-plugin`的配置，所以打包的默认的包是找不到主类的。

而`maven-assembly-plugin`因为我们配置了主类，而且还启动了依赖拷贝功能，所以打出来的jar不仅仅有主类，还有scala的依赖，可以直接运行。

## spark 开发环境

我们创建一个空的maven项目

![image-20220217231813445](https://i-blog.csdnimg.cn/blog_migrate/26892c820eb5486859ae39523280aaf2.png)

别忘记告诉idea我们需要scala环境

![image-20220217231923767](https://i-blog.csdnimg.cn/blog_migrate/acb7153d870ba93cc0bab0eab7e40e58.png)

创建目录

![image](https://i-blog.csdnimg.cn/blog_migrate/acb7153d870ba93cc0bab0eab7e40e58.png)

包目录

![image-20220217232046096](https://i-blog.csdnimg.cn/blog_migrate/6cd928ac86dd5115a3d3f74d2c1732b9.png)

主类

![image-20220217232114062](https://i-blog.csdnimg.cn/blog_migrate/c826929dda606f16684a341489cb90c3.png)

加入spark依赖和scala依赖，以及相关的插件

spark的版本

我在定义properties时遵循一个习惯：

依赖的版本号以`.version`结尾

插件的版本号以`version`开头，`.plugin`结尾
    
    
            <!-- spark 依赖的版本 -->
            <spark.version>3.2.0</spark.version>
    

增加`spark-core_2.12`依赖，因为scala二进制不兼容，所以需要注意哦
    
    
            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-core_2.12</artifactId>
                <version>${spark.version}</version>
                <!-- idea运行需要注释掉，打包需要放开 -->
    <!--            <scope>provided</scope>-->
            </dependency>
    

因为spark打的包最终是提交给spark执行的，本身就有spark相关的依赖，所以我们打包的时候不需要将spark的依赖打进去，否则会非常大，而且因为spark本身的依赖关系比较复杂，处理依赖的传递等问题会比较麻烦。

子项目的插件
    
    
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>org.scala-tools</groupId>
                    <artifactId>maven-scala-plugin</artifactId>
                    <configuration>
                        <launchers>
                            <launcher>
                                <id>wordcount</id>
                                <mainClass>com.study.spark.maven.wordcount.WordCount</mainClass>
                            </launcher>
                        </launchers>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-assembly-plugin</artifactId>
                    <configuration>
                        <archive>
                            <manifest>
                                <addClasspath>true</addClasspath>
                                <classpathPrefix>lib/</classpathPrefix>
                                <mainClass>com.study.spark.maven.wordcount.WordCount</mainClass>
                            </manifest>
                        </archive>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    

刷新maven项目后就可以在主类中编写spark代码了

如果你在开发代码的时候，发现scala的相关关键词无法识别，请重新刷新idea的缓存

![image-20220217234125971](https://i-blog.csdnimg.cn/blog_migrate/395972f236c04bba2b779c472b8d6e96.png)

选择这个重启idea即可

![image-20220217234147243](https://i-blog.csdnimg.cn/blog_migrate/5e6ad6e80d580b34c47eba8f200e5c99.png)

wordcount的代码如下
    
    
    package com.study.spark.maven.wordcount
    
    import org.apache.spark.{SparkConf, SparkContext}
    
    object WordCount {
    
      def main(args: Array[String]): Unit = {
        val conf = new SparkConf().setMaster("local").setAppName("wordcount").set("spark.testing.memory","2147480000")
        val sc = new SparkContext(conf)
        sc.textFile("E:\\java\\sparkmaven\\pom.xml")
          .flatMap(_.split(" "))
          .map(_->1)
          .reduceByKey(_+_)
          .collect()
          .foreach(println(_))
      }
    
    }
    

![image-20220217234415892](https://i-blog.csdnimg.cn/blog_migrate/d91844569093a65a18487532b5e5cc9a.png)

我们首先直接点击运行

![image-20220217234509876](https://i-blog.csdnimg.cn/blog_migrate/13de136f88e48781c639aaaaf49f6225.png)

接着使用`scala:run`执行

![image-20220217234610989](https://i-blog.csdnimg.cn/blog_migrate/38b0b2f48770957cd4ab90c38798d47c.png)

接着放开spark-core的scope注释，进行打包

第一次运行会比较慢，而且我们基本上都是用的最新的spark的依赖，镜像库可能还没有同步，所以更慢。

![image-20220217234930157](https://i-blog.csdnimg.cn/blog_migrate/5f8e9c877c96ac55645d086a0fbe29f9.png)

30多分钟，我是楞是等它结束了

![image-20220218003602429](https://i-blog.csdnimg.cn/blog_migrate/ac68c63fcda16abdf10058e3af705a66.png)

看看打的包能不能运行

![image-20220218003653251](https://i-blog.csdnimg.cn/blog_migrate/67f8905562572be0f5919db65c61c532.png)

也是可以运行的

![image-20220218003711076](https://i-blog.csdnimg.cn/blog_migrate/28b5e0f1467d268e32e8139902ab4636.png)

maven-assembly-plugin活生生的把spark打包打进去了

如果我们不需要将spark打包，那么应该会快很多的

![image-20220218003859906](https://i-blog.csdnimg.cn/blog_migrate/d61c4df4c7d40c87d0aafd87bbe19d28.png)

不过会提示找不到spark的类

![image-20220218003932269](https://i-blog.csdnimg.cn/blog_migrate/3e30680cf6e3d985d2ac1a72f06285be.png)

不过第二次打包就会快很多了

![image-20220218004105583](https://i-blog.csdnimg.cn/blog_migrate/31a637b04f81a9486372f5fd4a9f9322.png)

而且不管我们有没有把spark的依赖打包，提交给spark执行一般不会有问题。

但是需要注意jar包内的spark和spark环境的版本之间可能会存在冲突，所以不建议把spark的依赖进行打包。

我们尝试把打的jar包，提交到spark集群中运行

首先启动集群环境

![image-20220218004345596](https://i-blog.csdnimg.cn/blog_migrate/29512e7180996d28640dc789adc591a3.png)

然后使用xshell链接(家庭版免费)

![image-20220218004413432](https://i-blog.csdnimg.cn/blog_migrate/e9f42bcc8528307acbb6b7ce93277027.png)

启动hdfs集群

![image-20220218004529294](https://i-blog.csdnimg.cn/blog_migrate/3c607b3137fb4eb8683f39456cc306a3.png)

接着启动spark集群

![image-20220218004640701](https://i-blog.csdnimg.cn/blog_migrate/418a74f7675b5d7b3a076894ea4484a3.png)

启动spark历史记录服务

![image-20220218004724261](https://i-blog.csdnimg.cn/blog_migrate/136f42360fd9e5fa2d2de812bc56a7e4.png)

这个wordcount是在sbt构建中执行的，详见[spark源码编译和集群部署以及idea中sbt开发环境集成_a18792721831的博客-CSDN博客](<https://blog.csdn.net/a18792721831/article/details/122914322>)

因为我们的hdfs上面已经有文件了，所以需要修改代码，使用hdfs的文件，而不是本地的文件，而且使用集群的spark而不是本地的spark

修改后的WordCount如下
    
    
    package com.study.spark.maven.wordcount
    
    import org.apache.spark.{SparkConf, SparkContext}
    
    object WordCount {
    
      def main(args: Array[String]): Unit = {
        val conf = new SparkConf().setMaster("spark://hadoop01:4040").setAppName("wordcount.maven")
        val sc = new SparkContext(conf)
        sc.textFile("hdfs://hadoop01:8020/input/build.sbt")
          .flatMap(_.split(" "))
          .map(_->1)
          .reduceByKey(_+_)
          .collect()
          .foreach(println(_))
      }
    
    }
    
    

打包并将jar包上传到服务器，我们不需要spark的依赖

![image-20220218005149986](https://i-blog.csdnimg.cn/blog_migrate/9e57eeeb4ac91b7bf302e9da7f8fa0c1.png)

使用如下命令提交
    
    
    spark-submit --class com.study.spark.maven.wordcount.WordCount --master spark://hadoop01:4040 wordcount-1.0-jar-with-dependencies.jar
    

成功执行

![image-20220218005338444](https://i-blog.csdnimg.cn/blog_migrate/5fc170a719fd5c3f226669bb1385a950.png)

从spark历史记录中也能查看

![image-20220218005408277](https://i-blog.csdnimg.cn/blog_migrate/bded2d4245ccbab36d39c0460ea5cda2.png)

![image-20220218005438370](https://i-blog.csdnimg.cn/blog_migrate/824efcbbe85b31f11f59fc019e051574.png)

其实我也有个疑问，如果我们把spark的依赖打包了，还能执行吗？

试试！

含有spark依赖的包，128M，😆

![image-20220218005652010](https://i-blog.csdnimg.cn/blog_migrate/7446f364ea6e623de2ecb959a70e5eee.png)

执行：

![image-20220218005804370](https://i-blog.csdnimg.cn/blog_migrate/7b02880c4716c66555093acea107a617.png)

没啥区别

![image-20220218005837909](https://i-blog.csdnimg.cn/blog_migrate/73217e61d779a79fb53b1341f5a3e5fa.png)

这是因为我服务器也是3.2.0版本，而且是使用最新的源码编译的，详见[spark源码编译和集群部署以及idea中sbt开发环境集成_a18792721831的博客-CSDN博客](<https://blog.csdn.net/a18792721831/article/details/122914322>)，本地的依赖也是3.2.0，所以jar包中有没有spark依赖都是可以的。

如果你无法准确的知道服务器的spark的版本，以及服务器的spark的scala版本，建议还是不要把spark依赖打包。
