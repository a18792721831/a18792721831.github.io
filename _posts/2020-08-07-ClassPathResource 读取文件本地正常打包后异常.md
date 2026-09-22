---
layout: post
title: "ClassPathResource 读取文件本地正常打包后异常"
date: 2020-08-07 13:46:40 +0800
categories: [springboot坑, ClassPathResour, SAXReader解析xml, 读取xml并解析, 文件读取排错]
description: "本文介绍了解决在本地环境下使用ClassPathResource读取文件正常，但在打包成jar后部署到服务器运行时出现异常的问题。通过使用InputStream替代file对象，并调整XML中DTD文件的路径引用方式，实现了在本地和服务器环境下的正常读取。"
keywords: springboot坑, ClassPathResour, SAXReader解析xml, 读取xml并解析, 文件读取排错
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107860718
> - 发布时间：2020-08-07 13:46:40
> - 阅读量：3
> - 分类：java同时被 3 个专栏收录, 订阅专栏, 微服务, spring boot
> - 标签：#springboot坑, #ClassPathResour, #SAXReader解析xml, #读取xml并解析, #文件读取排错

## 摘要

文章浏览阅读3.1k次，点赞2次，收藏4次。本文介绍了解决在本地环境下使用ClassPathResource读取文件正常，但在打包成jar后部署到服务器运行时出现异常的问题。通过使用InputStream替代file对象，并调整XML中DTD文件的路径引用方式，实现了在本地和服务器环境下的正常读取。

---

## ClassPathResource 读取文件本地正常打包后异常

代码：

![image-20200807115640657](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/930c514c520e71d48f938b2cf145ffe4.png)

里面使用了`classPathResource.getFile().listFiles()`获取一个目录下全部的文件，然后返回的是file数组。

文件放在了resource下的一个目录中

![image-20200807115906279](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3c558f8cb0cddac0f0eaa45173c9d4da.png)

在本地正常使用，但是打成jar包，部署到服务器，使用`java -jar`启动后，出现异常：

![image-20200807120051836](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f34b186b6b5752950d88be2b56e30746.png)

从异常中来看，大概是说：目标目录在一个jar包里面，我们使用的是`ClassPathResource`的getFile方法获取了目录的file对象，然后通过listFiles获取目录下全部的文件。

问题就在这里：

![image-20200807120335622](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3cd54ca98d3d7a48ec8cf9f23a6f769f.png)

通过转换绝对路径，然后直接读取

![image-20200807120447052](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5c0f1304b1240aa242f9daaf132b078e.png)

File类需要的路径是一个独立的文件的路径，但是我们给的是jar内的一个路径，就无法读取了。

网上也有很多这样的资料，我也没有一一尝试，或许网上的解决方案对于我来说也可行。

我说下我的解决方案，仅供参考：

因为我读取的是一个xml文件，后面需要使用SAXReader解析xml的。在xml中引入了.dtd文件。

这是解析的代码

![image-20200807120749108](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/9a6b97ba66642ed0139ca8fb8de33e40.png)

我直接给reader.read传入一个文件对象，就行了。

但是现在使用ClassPathResource无法直接获取到file对象。

通过查看网上的资料，说使用流可以。

于是就修改成：`reader.read(classPathResource.getInputSttream())`

这个编译到是也没有问题，但是在运行的时候，出现了异常：

![image-20200807121137010](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/da97a445fa17cd00bdd43ada460110fd.png)

后来想了想，应该是xml中配置的dtd是相对路径，但是解析的时候变成绝对路径的时候出错了

于是使用`reader.read(classPathResource.getInputStream(), "classpath:" + dtdpath);`本地又可以启动了。

同时服务器启动也正常了。

————————————————————————————2021-05-08  
有些小伙伴对具体的实现有些疑问，我就把源码发下：
    
    
        private Map<String, byte[]> readCommonQueryResource() throws CommQueryResourceException {
            Map<String, byte[]> result = new HashMap<>();
            try {
                // 获取当前类的类加载器，然后通过 类加载器得到指定目录的 uri,并且得到这个uri的链接对象
                // QUERY_FILES_PATH是resource下的目录。resource目录会被打到jar包里面
                URLConnection connection = LoadXml2RedisGenQuery.class.getClassLoader().getResource(QUERY_FILES_PATH).openConnection();
                // 如果连接对象是jar类型的链接对象，表示指定目录是jar包内的目录
                if (connection instanceof JarURLConnection) {
                    // 此时将链接对象转为jar类型的链接对象
                    JarURLConnection jarCon = (JarURLConnection) connection;
                    // 设置不使用缓存
                    jarCon.setUseCaches(false);
                    // 然后获取到jar文件对象
                    JarFile jarFile = jarCon.getJarFile();
                    // 这里不能使用stream,使用stream,会出现zipifile is closed异常。里面源码相同，应该是兼容性问题吧.
                    Enumeration<JarEntry> entries = jarFile.entries();
                    // 通过jarFile对象获取jar里面的文件
                    while (entries.hasMoreElements()) {
                        JarEntry jarEntry = entries.nextElement();
                        String jarEntryName = jarEntry.getName();
                        // 只要指定前缀文件的字节流，或者是指定格式的文件的字节流
                        // 这里使用循环处理
                        // 举个例子：假设我们的resource目录下有一个testxml目录
                        // testxml目录下有很多xml，里面有些是test_1_x.xml
                        // 有些是test_2_x.xml
                        // 我们这里的QUERY_FILES_PATH 就是test_1,QUERY_FILE_NAME_PATTERN就是x
                        // 如果还有test_1_a_x.xml和test_1_a_y.xml
                        // 那么最终会在map中存储test_1_x.xml，test_1_a_x.xml的文件的key以及这两个文件对应的字节流
                        // 我们拿到了字节流，就可以读取具体的文件内容了
                        // 如果需要修改文件，貌似应该也可以，因为我们能拿到jarFile对象，修改jar里面的文件，都是通过jarFile操作的
                        // jarFile是一个操作jar文件的工具类
                        // jar文件实际上就是一个压缩包
                        // 你如果想操作jar包，那么对比下操作压缩包，类似的实现
                        // 需要注意的是这里得到的文件名称都是uri格式的，而且是类似于jar:xx/xxx/xx/xml格式
                        if (jarEntryName.startsWith(QUERY_FILES_PATH) && !jarEntry.isDirectory() &&
                                jarEntryName.matches(QUERY_FILES_PATH + QUERY_FILE_NAME_PATTERN)) {
                            InputStream inputStream = jarFile.getInputStream(jarEntry);
                            result.put(jarEntryName, inputStream == null ? new byte[0] : inputStream.readAllBytes());
                        }
                    }
                } else {
                // 如果是其他的链接类型，那么至少不是jar包，那么就可以用读取文件的方式读取
                    ClassPathResource classPathResource = new ClassPathResource(QUERY_FILES_PATH);
                    // 我们通过链接得到了文件夹的字符串
                    String fileNames = new String(connection.getInputStream().readAllBytes());
                    if (classPathResource.exists()) {
                        for (String fileName : fileNames.split("\n")) {
                            if (fileName.matches(QUERY_FILE_NAME_PATTERN)) {
                                ClassPathResource resource = new ClassPathResource(QUERY_FILES_PATH + fileName);
                                if (resource.exists()) {
                                    InputStream inputStream = resource.getInputStream();
                                    result.put(fileName, inputStream == null ? new byte[0] : inputStream.readAllBytes());
                                }
                            }
                        }
                    }
                }
            } catch (IOException e) {
                throw CommQueryResourceException.getException(e);
            }
            return result;
        }
    
    

操作jar包，我们需要知道我们实际上操作的是压缩包，然后按照压缩包操作的流程处理即可。