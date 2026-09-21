---
layout: post
title: "在html,js,xml中使用Ajax"
date: 2017-07-22 17:17:54 +0800
categories: [javascript, html, xml, ajax, 服务器]
description: "本文介绍了一个使用Ajax从HTML、JS及XML文件中获取数据并实现局部刷新的小项目。通过具体的代码示例，展示了如何设置服务器端环境并在浏览器中测试。"
keywords: javascript, html, xml, ajax, 服务器
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/75780068
> - 发布时间：2017-07-22 17:17:54
> - 阅读量：796
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#javascript, #html, #xml, #ajax, #服务器

## 摘要

文章浏览阅读796次。本文介绍了一个使用Ajax从HTML、JS及XML文件中获取数据并实现局部刷新的小项目。通过具体的代码示例，展示了如何设置服务器端环境并在浏览器中测试。

---

因为Ajax的定义如下：

![image](https://img-blog.csdn.net/20170722171754805?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

需要使用到服务器端，所以，创建一个web项目

![image](https://img-blog.csdn.net/20170722172119961?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

新建一个files文件夹：

![image](https://img-blog.csdn.net/20170722172513550?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

在index.jsp中添加如下代码：
    
    
      <body>
        <button id="button">点击测试ajax</button><b id="b"></b>
      </body>

在添加Ajax代码： 
    
    
    <script type="text/javascript">
    	window.onload = function() {
    		var butn = document.getElementById("button");
    		butn.onclick = function() {
    			var request = new XMLHttpRequest();
    			var method="GET";
    			var url="files/getreturn.html";
    			request.open(method, url);
    			request.send(null);
    			request.onreadystatechange = function() {
    				if(request.readyState == 4 && request.status == 200)
    					document.getElementById("b").innerHTML = request.responseText;
    			}
    		}
    	}
    </script>

在files文件夹中建立一个HTML文件，在里面写入如下代码： 
    
    
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    这是GET返回的信息。
    

2.第一个例子到这里就写完了，使用Ajax调用HTML中的数据，但是现在还不能看到效果，需要把项目添加到服务器中 

![image](https://img-blog.csdn.net/20170722172934170?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
  
在图中，并没有我们的项目，所以需要把项目加入到服务器中：  
![image](https://img-blog.csdn.net/20170722173218392?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

加入我们的项目后，启动服务器：

![image](https://img-blog.csdn.net/20170722173313902?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

启动：

![image](https://img-blog.csdn.net/20170722173416505?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

在浏览器查看效果：

在浏览器输入：http://localhost:8080/test

![image](https://img-blog.csdn.net/20170722173538560?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

点击按钮：

![image](https://img-blog.csdn.net/20170722173627441?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

发现url没有改变，也就是说页面没有刷新，但是在button后面却刷新了一部分，这就是Ajax的方法，实现局部刷新

3.Ajax调用js文件中的数据：

在files文件夹下创建一个js文件，写入如下代码：
    
    
    {"person":{
    	
    	"name":"bill",
    	"value":"how are you"
    		}
    }

保存之后发现有错，解决方法： 

![image](https://img-blog.csdn.net/20170722174436438?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

选中js文件，右键->MyEclipse->Exclude From Validation,等下就OK了  
![image](https://img-blog.csdn.net/20170722175034785?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

在js文件中写入如下代码：
    
    
    {"person":{
    	
    	"name":"bill",
    	"value":"how are you"
    		}
    }

在index.jsp中写下如下代码： 
    
    
      <body>
        <button id="button">点击测试ajax</button><b id="b"></b><br>
        <button id="button" onclick="clk()">点击测试ajax与js</button><b id = "b1"></b>
      </body>

在index.jsp中添加代码： 
    
    
    function clk(){
    	var reqst = new XMLHttpRequest();
    	var method = "GET";
    	var url = "files/myjs.js";
    	reqst.open(method, url);
    	reqst.send(null);
    	reqst.onreadystatechange = function() {
    		if(reqst.readyState == 4 && reqst.status == 200)
    		{
    			var result = reqst.responseText;
    			
    			var obj = eval("(" + result + ")");
    			document.getElementById("b1").innerHTML ="name:" + obj.person.name + "      value:" +obj.person.value;
    		}
    	}
    }

4.OK，在浏览器上刷新界面： 

![image](https://img-blog.csdn.net/20170722175452776?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
点击第二个button：

![image](https://img-blog.csdn.net/20170722175558293?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
两个button都进行测试：

![image](https://img-blog.csdn.net/20170722175650290?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
  

5.现在该写Ajax在XML中获取数据：

在files文件夹下创建XML文件：

写入如下代码：
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <details>
      <name>Jeremy Keith</name>
      <value>你好啊</value>
    </details>

在index.jsp中添加如下代码： 
    
    
     <button onclick="ckk()">点击测试ajax与xml</button><b id = "b2"></b>

在index.jsp中添加如下代码： 
    
    
    function ckk(){
    	var request = new XMLHttpRequest();
    	var method = "GET";
    	var url = "files/MyXml.xml";
    	request.open(method, url);
    	request.send(null);
    	request.onreadystatechange = function() {
    		if(request.readyState == 4 && request.status == 200){
    			var result = request.responseXML;
    			var name = result.getElementsByTagName("name")[0].firstChild.nodeValue;
    			var v = result.getElementsByTagName("value")[0].firstChild.nodeValue;
    			document.getElementById("b2").innerHTML = "name:"+name+"    value:"+v;
    		}
    	}
    }

6.现在来进行测试： 

![image](https://img-blog.csdn.net/20170722190910177?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
  
点击点三个button：

![image](https://img-blog.csdn.net/20170722190948583?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
  

点击三个button：

![image](https://img-blog.csdn.net/20170722191028913?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

至此，这个小项目就完成了，

总结一下，这个项目主要实现了利用Ajax方法分别在HTML，js，XML数据源中获取数据，并对界面进行局部刷新。