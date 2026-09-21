---
layout: post
title: "MyEclipse环境下的JavaWeb项目打包成war包部署到tomcat服务器发生jstl错误解决办法"
date: 2017-10-04 15:39:13 +0800
categories: [tomcat, 服务器, myeclipse, java web, jstl]
description: "本文介绍如何解决从MyEclipse环境中打包并部署到Tomcat服务器时出现的JSTL错误问题。具体步骤包括复制必要的JAR文件至Tomcat的lib目录，并配置web.xml文件。"
keywords: tomcat, 服务器, myeclipse, java web, jstl
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/78158311
> - 发布时间：2017-10-04 15:39:13
> - 阅读量：1
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#tomcat, #服务器, #myeclipse, #java web, #jstl

## 摘要

文章浏览阅读1.6k次。本文介绍如何解决从MyEclipse环境中打包并部署到Tomcat服务器时出现的JSTL错误问题。具体步骤包括复制必要的JAR文件至Tomcat的lib目录，并配置web.xml文件。

---

在MyEclipse环境下的一个JavaWeb项目打包成war包，然后单独部署到tomcat服务器会发生jstl错误，前提是jsp中使用了jstl。

比如把一个项目部署到Linux系统中的tomcat服务器上：   
![这里写图片描述](https://img-blog.csdn.net/20171004152024946?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
重启tomcat服务器，   
然后访问使用了jstl的jsp页面：   
![这里写图片描述](https://img-blog.csdn.net/20171004154558619?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
发生了jstl错误，打开源码：   
![这里写图片描述](https://img-blog.csdn.net/20171004152512698?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
在file_load.jsp中使用了jstl语言：   
![这里写图片描述](https://img-blog.csdn.net/20171004152624536?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

接下来，就是解决的办法：   
1.找到2个jar包：   
第一个是：   
![这里写图片描述](https://img-blog.csdn.net/20171004152747036?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
路劲在后面可以看到，这个图片是在MyEclipse的环境下查看，找到这个jar包，复制到tomcat服务器项目下的WEB-INF下的lib文件夹里：   
![这里写图片描述](https://img-blog.csdn.net/20171004153012413?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
第二个是：   
![这里写图片描述](https://img-blog.csdn.net/20171004153223344?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
路劲在：   
安装目录MyEclipse\Common\plugins\com.genuitec.eclipse.j2eedt.core_10.0.0.me201110301321\data\libraryset\JSTL1.1\lib   
下   
同样拷贝到WEB-INF下的lib里

最后一步，修改web.xml文件：   
![这里写图片描述](https://img-blog.csdn.net/20171004153558091?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    
    
     <jsp-config>  
          <taglib>  
                <taglib-uri>http://java.sun.com/jstl/fmt</taglib-uri>  
                <taglib-location>/WEB-INF/tld/fmt.tld</taglib-location>  
            </taglib>  
            <taglib>  
                <taglib-uri>http://java.sun.com/jstl/core</taglib-uri>  
                <taglib-location>/WEB-INF/tld/c.tld</taglib-location>  
            </taglib>  
            <taglib>  
                <taglib-uri>http://java.sun.com/jstl/sql</taglib-uri>  
                <taglib-location>/WEB-INF/tld/sql.tld</taglib-location>  
            </taglib>  
            <taglib>  
                <taglib-uri>http://java.sun.com/jstl/x</taglib-uri>  
                <taglib-location>/WEB-INF/tld/x.tld</taglib-location>  
            </taglib>  
        </jsp-config>

最后重启服务器，访问：   
![这里写图片描述](https://img-blog.csdn.net/20171004155505117?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
搞定！