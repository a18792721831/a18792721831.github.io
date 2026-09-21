---
layout: post
title: "java使用jdbc连接MySQL数据库"
date: 2017-07-16 17:02:08 +0800
categories: [mysql, java, jdbc, jar, 数据库]
description: "本文介绍如何在Java项目中使用JDBC进行数据库连接及基本操作，包括导入MySQL驱动、建立连接、执行SQL语句及关闭资源等关键步骤。"
keywords: mysql, java, jdbc, jar, 数据库
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/75208101
> - 发布时间：2017-07-16 17:02:08
> - 阅读量：711
> - 分类：java专栏收录该内容, 订阅专栏
> - 标签：#mysql, #java, #jdbc, #jar, #数据库

## 摘要

文章浏览阅读711次。本文介绍如何在Java项目中使用JDBC进行数据库连接及基本操作，包括导入MySQL驱动、建立连接、执行SQL语句及关闭资源等关键步骤。

---

在Java项目中使用jdbc连接数据库需要以下几个步骤：

前期准备：需要把MySQL的jar包导入到项目：

1.在项目上点击右键，new一个folder

2.在新建的folder上右键import选择file System找到jar包导入项目

3.在新导入的jar包上右键Bulid path，选择Add to Build Path

4.当出现Referenced Libraries表示添加jar包成功；

一、通过反射加载驱动，然后连接到数据库

1.确定数据库驱动

String string = "com.mysql.jdbc.Driver";//固定写法

2.加载驱动

try{

//① 加载驱动

Class.forName(string);

//②确定要连接的数据库信息

String url="jdbc:mysql://127.0.0.1:3306/xkd";//127.0.0.1表示数据库所在的PC的IP地址，3306是MySQL数据库的端口号，xkd是需要连接的数据库名称

//有些情况可能出现SSL错误，需要在xkd后面加入:?useSSL = true  
String user="root";//连接数据库的用户名  
String password="mysql";//连接数据库的用户名对应的密码

//③建立连接

Connection connection= DriverManager.getConnection(url,user,password);

二、操作数据库

//① 获取statement  
Statement statement=connection.createStatement();  
//②确定SQL语句  
String sqlString="insert into student(id,name,pass) values(2,'张同学','password')";  
//③执行操作  
statement.executeUpdate(aqlString);  
三、关闭资源  
statement.close();  
connection.close();  
} catch (ClassNotFoundException e) {  
e.printStackTrace();  
System.out.println("找不到类");  
} catch (SQLException e) {  
e.printStackTrace();  
}  

    
    
    package testio;
    
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.SQLException;
    import java.sql.Statement;
    
    public class Data {
       /**
        *   JDBC
        * 
        * */
    	public static void main(String[] args) {
    		//1、连接数据库   反射
    		    //① 确定数据库驱动 MySql
    		     String  string="com.mysql.jdbc.Driver";
    		    try {
    		    //② 加载驱动	
    				Class.forName(string);			
    			//③确定要连接的数据库、用户名、密码
    				String url="jdbc:mysql://127.0.0.1:3306/xkd";
    				String user="root";
    				String password="mysql";
    			//④建立与数据库的连接
    				Connection connection=  DriverManager.getConnection(url,user,password);
    		//2、操作数据库
    			//① 获取statement
    				Statement statement=connection.createStatement();
    			//②确定SQL语句
    				String  aqlString="insert into student(id,name,pass) values(2,'张同学','password')";
    			//③执行操作
    			        statement.executeUpdate(aqlString);
    		//3、关闭资源
    				         statement.close();
    				         connection.close();
    		    } catch (ClassNotFoundException e) {
    				// TODO Auto-generated catch block
    				e.printStackTrace();
    				System.out.println("找不到类");
    			} catch (SQLException e) {
    				// TODO Auto-generated catch block
    				e.printStackTrace();
    			}
    	}
    
    }