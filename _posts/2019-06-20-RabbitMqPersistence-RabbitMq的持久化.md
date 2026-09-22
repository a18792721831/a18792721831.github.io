---
layout: post
title: "RabbitMqPersistence:RabbitMq的持久化"
date: 2019-06-20 20:52:13 +0800
categories: [RabbitMQ持久化, RabbitMQ消息持久化, RabbitMQ交换机持久化, RabbitMQ队列持久化, Rabbit持久化原理]
description: "本文介绍了RabbitMQ持久化相关知识。RabbitMQ对队列消息有disk和RAM两种保存方式，disk将消息写入磁盘，RAM不保存数据。还阐述了Queue、Message、Exchange持久化的实现方法，指出即便都持久化也不能保证消息不丢失，并给出解决思路，最后通过gradle项目实例展示持久化操作。"
keywords: RabbitMQ持久化, RabbitMQ消息持久化, RabbitMQ交换机持久化, RabbitMQ队列持久化, Rabbit持久化原理
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/93129821
> - 发布时间：2019-06-20 20:52:13
> - 阅读量：1
> - 分类：RabbitMQ专栏收录该内容, 订阅专栏
> - 标签：#RabbitMQ持久化, #RabbitMQ消息持久化, #RabbitMQ交换机持久化, #RabbitMQ队列持久化, #Rabbit持久化原理

## 摘要

文章浏览阅读1.4k次。本文介绍了RabbitMQ持久化相关知识。RabbitMQ对队列消息有disk和RAM两种保存方式，disk将消息写入磁盘，RAM不保存数据。还阐述了Queue、Message、Exchange持久化的实现方法，指出即便都持久化也不能保证消息不丢失，并给出解决思路，最后通过gradle项目实例展示持久化操作。

---

#### RabbitMqPersistence:RabbitMq的持久化

  * 1.Queue持久化
  * 2.Message持久化
  * 3.Exchange持久化
  * 4.说明
  * 5.实例
  *     * 5.1新建一个gradle项目
    * 5.2创建配置类
    * 5.3创建消费者
    * 5.4创建生产者
    * 5.5生产
    * 5.6消费
    * 5.7总结

RabbitMq对于Queue中消息的保存方式有disk和RAM两种。  
  
disk就是把消息写入磁盘。  
  
RAM不会保存数据。  
  
disk方式有两种方式触发：  

  1. 发布消息时指明需要写入磁盘；  

  2. 当消息服务器中内存紧张时，会将部分内存中的消息转移到磁盘。  
  
disk的实现：  
  
消息数据会被保存在以.rdq后缀命名的文件中，当文件达到一定的大小  
（默认是16777216字节，即16MB）时会生成一个新的文件，当文件  
中的已经被删除的消息比例大于阈值时会触发文件合并操作，以提高  
磁盘利用率。  
  
RAM方式：  
  
只是在RAM中保存内部数据库表数据，而不会保存消息、消息存储索引、  
队列索引和其他节点状态等数据。

## 1.Queue持久化

Queue持久化通过设置durable为true来实现的。  
  
Queue持久化只是持久化了队列，队列里面的消息并不会进行持久化,重启  
MQ服务器后，队列因为持久化不会丢失，但是消息却会丢失。

## 2.Message持久化

Message持久化是通过发布消息时的BasicProperties设置的。  
  
basicPublish方法的第三个参数BasicProperties参数，设置为PERSISTENT_TEXT_PLAIN  
就表示需要进行持久化。

## 3.Exchange持久化

Exchange持久化类似Queue的持久化，都是使用durable为true来实现的。

## 4.说明

即使对上述3部分都做出了持久化，也不能保证消息在使用过程中完全不会丢失。  
比如，如果消息消费者在接受到消息时，autoAck为true,但是消费者在处理消息中发生异常，  
那么此消息消费失败，但是队列中没有此消息，造成消息丢失。

上述例子可以设置autoAck为false然后在消费者完全消费完成后手动确认。  
当然消息发布也存在同样的事情。

考虑消息确认模式解决。

## 5.实例

### 5.1新建一个gradle项目

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d4e3a8c901da0379207e5143ab03f35c.png)  
创建java包  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/b770e8c53ced772396af1fd41d85c026.png)

### 5.2创建配置类
    
    
    package com.study.config;
    
    /**
     * @author jiayq
     */
    public class RabbitMqConfig {
    
        /**
         * 队列名称
         */
        public static final String QUEUE_NAME = "HelloQueue";
    
        /**
         * 主机名称
         */
        public static final String HOST_NAME = "localhost";
    
        /**
         * 路由键
         */
        public static final String ROUNT_KEY = "HelloRounts";
    
        /**
         * 交换机
         */
        public static final String EXCHANGE_NAME = "HelloExchange";
    
    }
    
    

### 5.3创建消费者
    
    
    package com.study.consume;
    
    import com.rabbitmq.client.*;
    import com.study.config.RabbitMqConfig;
    
    /**
     * @author jiayq
     */
    public class Consume {
    
        public static void main(String[] args) throws Exception {
            ConnectionFactory connectionFactory = new ConnectionFactory();
            connectionFactory.setHost(RabbitMqConfig.HOST_NAME);
            Connection connection = connectionFactory.newConnection();
            Channel channel = connection.createChannel();
            channel.queueDeclare(RabbitMqConfig.QUEUE_NAME, true, false, false, null);
            channel.exchangeDeclare(RabbitMqConfig.EXCHANGE_NAME, "direct", true, false, null);
            channel.queueBind(RabbitMqConfig.QUEUE_NAME, RabbitMqConfig.EXCHANGE_NAME, RabbitMqConfig.ROUNT_KEY);
            DeliverCallback deliverCallback = (tag, deliver) -> {
                System.out.println("Consumer get message:\t" + new String(deliver.getBody(),"utf-8"));
                System.out.println("appId:\t" + deliver.getProperties().getAppId());
            };
            CancelCallback cancelCallback = (tag) -> {
                System.out.println("Consumer get message error,tags:\t" + tag);
            };
            channel.basicConsume(RabbitMqConfig.QUEUE_NAME, true, deliverCallback, cancelCallback);
        }
    
    }
    
    

### 5.4创建生产者
    
    
    package com.study.produce;
    
    import com.rabbitmq.client.*;
    import com.study.config.RabbitMqConfig;
    
    import java.io.IOException;
    import java.util.Scanner;
    import java.util.concurrent.TimeoutException;
    
    /**
     * @author jiayq
     */
    public class Produce {
    
        public static void main(String[] args) {
            try (Scanner scanner = new Scanner(System.in)) {
                while (true){
                    System.out.println("Please input message(one row one message,quit to exits):");
                    String message = scanner.nextLine();
                    if ("quit".equals(message)) {
                        System.out.println("Will be quit!!!");
                        break;
                    }
                    sendMessage(message);
                }
            }
        }
    
        private static void sendMessage(String message) {
            ConnectionFactory connectionFactory = new ConnectionFactory();
            connectionFactory.setHost(RabbitMqConfig.HOST_NAME);
            try (
                    Connection connection = connectionFactory.newConnection();
                    Channel channel = connection.createChannel()
                    ) {
                channel.queueDeclare(RabbitMqConfig.QUEUE_NAME, true, false, false, null);
                channel.exchangeDeclare(RabbitMqConfig.EXCHANGE_NAME, "direct", true, false,null);
                channel.queueBind(RabbitMqConfig.QUEUE_NAME, RabbitMqConfig.EXCHANGE_NAME, RabbitMqConfig.ROUNT_KEY);
                channel.basicPublish(RabbitMqConfig.EXCHANGE_NAME, RabbitMqConfig.ROUNT_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
                System.out.println("The message send over:\t" + message);
            } catch (TimeoutException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    
    }
    
    

### 5.5生产

启动MQ服务器，打开管理界面  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/289cefb60534fb26938b6f1483bc8b96.png)  
生产消息  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4d4c39e8da31596b46a515754c91e513.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/27d6d10aba8dfea6b002a10fac95e009.png)  
然后关闭生产者，关闭MQ服务器  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/5cf35e3dab31a5dae4527296ea583811.png)  
管理界面无法访问：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/38d94cf411374814e0a25d319d69cbec.png)

### 5.6消费

启动MQ服务器  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/15294cb0da5763455e0ca1146157a4f6.png)  
启动消费者：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/850c1473f4a80d9012903a6d0c3f2643.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f7341eb403386de1eda08594265d27b3.png)

### 5.7总结

成功的持久化了队列、消息、交换机等等。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/93d9ccbb85f8b5ed79fe53927f7a59ab.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ccc7d4345920cbfc207c6dcce8f46aea.png)

git仓库地址：  
<https://github.com/a18792721831/MQ.git>
