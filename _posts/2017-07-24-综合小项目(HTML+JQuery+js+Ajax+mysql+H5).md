---
layout: post
title: "综合小项目(HTML+JQuery+js+Ajax+mysql+H5)"
date: 2017-07-24 18:20:06 +0800
categories: [javascript, css, mysql, jdbc, jquery]
description: "本文介绍如何使用HTML、CSS和JQuery构建带有数据库交互的登陆注册界面，包括前端表单验证及后端数据库操作。"
keywords: javascript, css, mysql, jdbc, jquery
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/76034259
> - 发布时间：2017-07-24 18:20:06
> - 阅读量：1
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#javascript, #css, #mysql, #jdbc, #jquery

## 摘要

文章浏览阅读1.2w次，点赞4次，收藏45次。本文介绍如何使用HTML、CSS和JQuery构建带有数据库交互的登陆注册界面，包括前端表单验证及后端数据库操作。

---

先来看一下最终效果图：

![image](https://img-blog.csdn.net/20170724182053454?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

1.首先，在以下例子中进行更改：

[查看文章](<http://blog.csdn.net/a18792721831/article/details/75314868>)  

这个例子是使用HTML和css建立的一个经典的界面，我们这次需要做的就是在登陆界面中，与数据库连接，实现登陆，注册功能：

首先更改登陆小窗口：

![image](https://img-blog.csdn.net/20170724182458238?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

更改为以下代码，实现上述效果：
    
    
    <table style="width: 300px;height: 180px;border: 0px;" align="center">
    <tr>
    	<td style="width: 100px;height: 40px;border: 0px;" align="center">
    		<b style="font-family: 楷体;font-size: 18px;color: black;">用户名:</b>
    	</td>
    	<td style="width: 200px;height: 40px;border: 0px;" align="center">
    		<input type="text" name="user" id="username">
    	</td>
    </tr>
    <tr>
    	<td colspan="3" style="width: 300px;height: 30px;border: 0px;" align="center">
    		<b id="user"></b>
    	</td>
    </tr>
    <tr>
    	<td style="width: 100px;height: 40px;border: 0px;" align="center">
    		<b style="font-family: 楷体;font-size: 18px;color: black;">密  码:</b>
    	</td>
    	<td style="width: 200px;height: 40px;border: 0px;" align="center">
    		<input type="password" name="password" id="password">
    	</td>
    </tr>
    <tr>
    	<td colspan="3" style="width: 300px;height: 30px;border: 0px;" align="center">
    		<b id = "pswd"></b>
    	</td>
    </tr>
    <tr>
    	<td colspan="3" style="width: 300px;height: 40px;border: 0px;"align="center">
    		<table style="width: 300px;height: 40px;border: 0px;" align="center">
    			<tr>
    				<td style="width: 100px;height: 40px;border: 0px;" align="center">
    					<input id="zhuce" type="button" value="注册" style="font-family: 楷体;font-size: 18px;">
    				</td>
    				<td style="width: 100px;height: 40px;border: 0px;"align="center">
    					<input id="chongzhi" type="button" value="重置" style="font-family: 楷体;font-size: 18px;">
    				</td>
    				<td style="width: 100px;height: 40px;border: 0px;" align="center">
    					<input id="denglu" type="button" value="登陆" style="font-family: 楷体;font-size: 18px;">
    					</td>
    				</tr>
    			</table>
    		</td>
    	</tr>
    </table>

2.因为要使用到JQuery，所以需要导入包之类的，具体请看： 

[查看文章](<http://blog.csdn.net/a18792721831/article/details/75729824>)  
  

为用户名输入框添加改变事件：
    
    
    	$("#username").change(function() {
    		$("#user").html("");
    		var val=$(this).val();
    val=$.trim(val);
    
    if(val != "")
    {
    var url="/serverlrtandhtml/servlet/IndexServerlet";
    var args={"method":"yanzhenguser","username":val,"time":new Date()};
    $.post(url,args,function(data){
    		$("#user").html(data);
    	});
    }
    	});

为密码输入框添加改变事件： 
    
    
     $("#password").change(function() {
    		var val=$(this).val();
    val=$.trim(val);
    if(val != "")
    {
    	if(val.length < 6 || val.length > 10){
    		$("#pswd").html("<font color='red'>密码长度为6-10位</font>");
    	}else{
    		$("#pswd").html("<font color='green'>密码格式正确</font>");
    	}
    }else{
    	$("#pswd").html("<font color='red'>密码不能为空</font>");
    }
    	}); 

  

为注册按钮添加点击事件： 
    
    
    $("#zhuce").click(function() {
    	var user = $("#username").val();
    	var pswd = $("#password").val();
    	if(user != "" && pswd != "")
    	{
    		if(pswd.length < 6 || pswd.length > 10){
    	$("#pswd").html("<font color='red'>密码长度为6-10位</font>");
    }else{
    	$("#pswd").html("<font color='green'>密码格式正确</font>");
    	var url="/serverlrtandhtml/servlet/IndexServerlet";
    	var args={"method":"zhuce","username":user,"password":pswd};
    	$.post(url,args,function(data){
    			alert(data);
    	});
    		
    		}
    	}
    });

  
为重置按钮添加点击事件： 
    
    
    $("#chongzhi").click(function() {
    	$("#username").val("");
    	$("#password").val("");
    	$("#user").html("");
    	$("#pswd").html("");
    });

  
为登陆按钮添加点击事件： 
    
    
    $("#denglu").click(function() {
    		var user = $("#username").val();
    		var pswd = $("#password").val();
    if(user != "" && pswd != "")
    {
    var url="/serverlrtandhtml/servlet/IndexServerlet";
    var args={"method":"denglu","username":user,"password":pswd};
    $.post(url,args,function(data){
    		alert(data);
    	});
    }
    	});

  
讲到这里，HTML文件中就OK了 

3.需要连接数据库，就需要在web Root文件夹下的WEB-INF文件夹下的lib文件夹中加入mysql的jar文件：

![image](https://img-blog.csdn.net/20170724190229585?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

4.在servlet类中连接数据库，并进行操作数据库：

首先创建一个servlet类：

①在项目的src文件夹下新建一个package，名字为：com.xust.jia

②在包上右键，new一个Servlet类，取名：IndexServerlet;

完成后如下：

![image](https://img-blog.csdn.net/20170724184035527?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  
5.我们在servlet类中进行以下修改：

定义准备变量：
    
    
    private String string = "com.mysql.jdbc.Driver";
    private ResultSet resultSet;
    private Connection connection = null;
    private Statement statement = null;
    private String result=null;
    private boolean userflag = false;
    private boolean pawdflag = false;

  
  

①在init方法中进行数据库的连接，因为此方法是服务器在启动时，创建servlet对象时调用的
    
    
    public void init() throws ServletException {
    	// Put your code here
    	try {
    		Class.forName(string);
    		String urlString = "jdbc:mysql://127.0.0.1:3306/xkd?&useSSL=true";
    		String userString = "root";
    		String passwordString = "a1871";//密码
    		connection = DriverManager.getConnection(urlString, userString, passwordString);
    		statement = connection.createStatement();
    	}catch (Exception e) {
    	// TODO: handle exception
    		e.printStackTrace();
    	}
    }

  

②在destroy方法中进行数据库的关闭，因为此方法是数据库重启或停止时调用的销毁方法：
    
    
    public void destroy() {
    	super.destroy(); // Just puts "destroy" string in log
    	// Put your code here
    	try{
    		if(connection != null)
    			connection.close();
    		if(statement != null)
    			statement.close();
    	}catch(Exception e){
    		e.printStackTrace();
    	}
    }

  
③在dopost方法中进行选择方法进行调用： 
    
    
    public void doPost(HttpServletRequest request, HttpServletResponse response)
    		throws ServletException, IOException {
    	String string = request.getParameter("method");
    	if(string.equals("yanzhenguser")){
    		result = null;
    		yanzhenguser(request, response);
    		if(userflag){
    			result="<font color='red'>无此用户</font>";
    		}else{
    			result="<font color='green'>该用户可以使用</font>";
    		}
    	}
    		
    	else if(string.equals("denglu")){
    		result = null;
    		denglu(request, response);
    		if(!userflag && pawdflag){
    			result = "登陆成功";
    		}
    		else if(!userflag && !pawdflag){
    			result = "密码错误";
    		}
    		else if(userflag && pawdflag){
    			result = "用户名错误";
    		}
    		else if(userflag && !pawdflag){
    			result = "用户名密码错误";
    		}
    		else{
    			result = "未知错误";
    		}
    	}
    	else if(string.equals("zhuce")){
    		result = null;
    		zhuce(request, response);
    	}
    	response.setCharacterEncoding("UTF-8");
    	response.setContentType("text/html;charset=UTF-8");
    	response.getWriter().print(result);
    }

④写用户名验证方法： 
    
    
    public void yanzhenguser(HttpServletRequest request, HttpServletResponse response)
    		throws ServletException, IOException {
    	try{
    		String sqlString = "SELECT username FROM user";
    		resultSet = statement.executeQuery(sqlString);
    		String userName=request.getParameter("username");
    		while(resultSet.next())
    		{
    			String name = resultSet.getString(1);
    			if(name.equals(userName)){
    				userflag = false;
    				break;
    			}
    			else{
    				userflag = true;
    				continue;
    			}
    		}
    		
    	} catch (Exception e) {
    		// TODO: handle exception
    		e.printStackTrace();
    	} 
    	
    }

⑤写密码验证方法： 
    
    
    public void yanzhengpswd(HttpServletRequest request, HttpServletResponse response)
    		throws ServletException, IOException {
    	try{
    		String sqlString = "SELECT password FROM user";
    		resultSet = statement.executeQuery(sqlString);
    		String pswdString=request.getParameter("password");
    		while(resultSet.next())
    		{
    			String pswd = resultSet.getString(1);
    			if(pswd.equals(pswdString)){
    				pawdflag = true;
    				break;
    			}
    			else{
    				pawdflag = false;
    				continue;
    			}
    		}
    		
    	} catch (Exception e) {
    		// TODO: handle exception
    		e.printStackTrace();
    	} 
    }

  
⑥写登陆方法： 
    
    
    public void denglu(HttpServletRequest request, HttpServletResponse response)
    		throws ServletException, IOException {
    	userflag = false;
    	pawdflag = false;
    	yanzhenguser(request, response);
    	yanzhengpswd(request, response);
    }

  
⑦写注册方法： 
    
    
    public void zhuce(HttpServletRequest request, HttpServletResponse response)
    		throws ServletException, IOException {
    	System.out.println("zhuce");
    	userflag = false;
    	yanzhenguser(request, response);
    	try{
    		if(userflag){
    			String sqlString = "INSERT INTO user(username,password) VALUES('"
    					+ request.getParameter("username") +"','"+ request.getParameter("password") +"')";
    			System.out.println(sqlString);
    			if(statement.executeUpdate(sqlString) != 0){
    				result = "注册成功";
    			}else{
    				result = "注册失败";
    			}
    		}else{
    			result = "用户名错误，注册失败";
    		}
    	}catch(Exception e){
    		e.printStackTrace();
    	}
    }

6.OK项目完成，放到tomcat服务器上，进行测试： 

①输入文本框：

![image](https://img-blog.csdn.net/20170724191437029?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
  
正确：

![image](https://img-blog.csdn.net/20170724191511566?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
  
②登陆：

![image](https://img-blog.csdn.net/20170724191553515?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
使用此用户名，密码登陆：

![image](https://img-blog.csdn.net/20170724191638270?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

使用其他用户名：

![image](https://img-blog.csdn.net/20170724191737430?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

③重置：

![image](https://img-blog.csdn.net/20170724191835381?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

④注册：

![image](https://img-blog.csdn.net/20170724191943302?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

![image](https://img-blog.csdn.net/20170724191921024?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

![image](https://img-blog.csdn.net/20170724192012457?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

⑤使用④注册的用户名密码登陆：

![image](https://img-blog.csdn.net/20170724192115384?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE4NzkyNzIxODMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)