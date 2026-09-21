---
layout: post
title: "RabbitMQ-Hello_World"
date: 2019-06-17 21:05:03 +0800
categories: [Hello World RabbitMQ, RabbitMQ, 入门RabbitMQ, RabbitMQ的三种交换机制, RabbitMQ原理详解]
description: "本文介绍了RabbitMQ的特点，如保证可靠性、支持消息集群等。阐述了其基本概念，包括消息、生产者、交换器等。重点讲解了AMQP中的消息路由及三种交换器类型。最后给出了hello world实例，包含创建model、下载jar包、实现生产者和消费者等内容。"
keywords: Hello World RabbitMQ, RabbitMQ, 入门RabbitMQ, RabbitMQ的三种交换机制, RabbitMQ原理详解
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/92710054
> - 发布时间：2019-06-17 21:05:03
> - 阅读量：414
> - 分类：RabbitMQ专栏收录该内容, 订阅专栏
> - 标签：#Hello World RabbitMQ, #RabbitMQ, #入门RabbitMQ, #RabbitMQ的三种交换机制, #RabbitMQ原理详解

## 摘要

文章浏览阅读414次。本文介绍了RabbitMQ的特点，如保证可靠性、支持消息集群等。阐述了其基本概念，包括消息、生产者、交换器等。重点讲解了AMQP中的消息路由及三种交换器类型。最后给出了hello world实例，包含创建model、下载jar包、实现生产者和消费者等内容。

---

#### RabbitMQ-Hello_World

  * 1.RabbitMQ的特点
  * 2.RabbitMQ的基本概念
  * 3.重点核心
  * 4.实例-hello world
  *     * 4.1创建一个model
    * 4.2jar包下载
    * 4.3生产者
    * 4.4消费者
    * 4.5生产一个消息
    * 4.6消费一个消息
    * 4.7消费者一直消费

## 1.RabbitMQ的特点

  1. 保证可靠性(Reliability)
  2. 灵活的路由功能(Flexible Routing)
  3. 支持消息集群(Clustering)
  4. 具有高可用性(Highly Available)
  5. 支持多种协议(Multi-protocol)
  6. 支持多语言客户端(Many Client)
  7. 提供管理界面(Management UI)
  8. 提供跟踪机制(Tracing)
  9. 提供插件机制(Plugin System)

## 2.RabbitMQ的基本概念

RabbitMQ是AMQP协议的一个开源实现。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e6b8552f5695f9aa2b156b8ee04b599.png)

  * Message:消息
  * Publisher:消息生产者
  * Exchange:交换器
  * Binding:绑定
  * Queue:消息队列
  * Connection:网络连接
  * Channel:信道
  * Consumer:消息消费者
  * Virtual Host:虚拟主机，在RabbitMQ中叫做vhost
  * Broker:消息服务器

## 3.重点核心

  1. AMQP中的消息路由  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d8c4c40cea91b29a127cf4153eaf8ff1.png)  
生产者需要把消息发布到Exchange上，消息最终到达队列并被消费者接收，而Binding决定交换器上的消息应该被送到哪个队列中。
  2. 交换器类型

  * Direct交换器  
如果消息中的路由键(routing key)和Binding中的绑定键binding key一致，交换器就将消息发送到对应的队列中。路由键与队列名称要完全匹配。Direct交换器是完全匹配、单播的模式。
  * Fanout交换器  
Fanout交换器不处理路由键，只是简单地将队列绑定到交换机，发送到交换器的每条消息都会被转发到与该交换器绑定的所有队列中。类似广播，通过Fanout交换器发消息是最快的。
  * Topic交换器  
Topic交换器通过模式匹配分配消息的路由键属性，将路由键和某种模式进行匹配，此时队列需要绑定一种模式。Topic交换将路由键和绑定键的字符串用.分割成单词，然后用#或者*进行匹配：#匹配0个或者多个单词，`*`匹配1个单词。

## 4.实例-hello world

### 4.1创建一个model

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b16b68a464ecb9911c16873f41257a96.png)  
并创建目录。

### 4.2jar包下载

下载RabbitMQ的客户端的jar包：<https://repo1.maven.org/maven2/com/rabbitmq/amqp-client/5.7.1/amqp-client-5.7.1.jar>  
下载客户端jar包依赖包：  
<https://repo1.maven.org/maven2/org/slf4j/slf4j-api/1.7.26/slf4j-api-1.7.26.jar>  
<https://repo1.maven.org/maven2/org/slf4j/slf4j-simple/1.7.26/slf4j-simple-1.7.26.jar>  
导入到工程  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eec46f7b8fbb72b5f327c749c6f5b1ce.png)

### 4.3生产者
    
    
    package com.study.produce;
    
    import com.rabbitmq.client.Channel;
    import com.rabbitmq.client.Connection;
    import com.rabbitmq.client.ConnectionFactory;
    
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    
    /**
     * @author jiayq
     */
    public class Produce {
    
        private final static String QUEUE_NAME = "hello_rabbit";
    
        public static void main(String[] args) {
            //创建连接工厂
            ConnectionFactory connectionFactory = new ConnectionFactory();
            //设置目标主机
            connectionFactory.setHost("localhost");
            try (
                    //创建连接
                    Connection connection = connectionFactory.newConnection();
                    //创建信道
                    Channel channel = connection.createChannel()
                    ) {
                //声明队列
                channel.queueDeclare(QUEUE_NAME,false,false,false,null);
                //创建消息
                String message = "Hello World";
                //生产消息
                channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
                System.out.println("publish");
            } catch (TimeoutException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    
    }
    
    

### 4.4消费者
    
    
    package com.study.consume;
    
    import com.rabbitmq.client.Channel;
    import com.rabbitmq.client.Connection;
    import com.rabbitmq.client.ConnectionFactory;
    import com.rabbitmq.client.DeliverCallback;
    
    /**
     * @author jiayq
     */
    public class Consume {
    
        private final static String QUEUE_NAME = "hello_rabbit";
    
        public static void main(String[] args) throws Exception{
            //创建连接工厂
            ConnectionFactory connectionFactory = new ConnectionFactory();
            //设置目标主机
            connectionFactory.setHost("localhost");
            //创建连接
            Connection connection = connectionFactory.newConnection();
            //创建信道
            Channel channel = connection.createChannel();
            //声明队列
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            //创建回调(ps:如果回调方法一直得不到执行，那么是否会发生阻塞？)
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                String message = new String(delivery.getBody(), "utf-8");
                System.out.println(message);
                System.exit(0);
            };
            //消费目标队列消息
            channel.basicConsume(QUEUE_NAME,true,deliverCallback,consumerTage -> {});
        }
    
    }
    
    

### 4.5生产一个消息

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7163b546bfdf8f48df7139102a07eded.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ed4a79a52f26c10f011daa4fe449421f.png)

### 4.6消费一个消息

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/62230155699530eb45d4b2688e010c31.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7addabe44628ef4a3462de0f4332a3f4.png)

### 4.7消费者一直消费

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f4d80bb612538052543c1ae183ce727e.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1ecb70a540afbc35df4d93f62c698df5.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/273c7e2abc47605670a8597b869afaaa.png)  
生产一个消息  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/89f10815a4b31c6b9dc55a68d63e78d8.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a93aa1df412eb0c3fde8ff7ff3a8c8e.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2779986ee9c521d26a67a94d2031c141.png)  
多生产几个消息  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0c3a4e3b6169db48fad55d647b33a3d5.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/db8b5737a7f77a1c9c92b6a3958eeb65.png)

git仓库地址  
<https://github.com/a18792721831/MQ.git>
