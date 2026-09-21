---
layout: post
title: "RabbitMqAck--消息确认模式"
date: 2019-06-21 21:12:32 +0800
categories: [RabbitMQ的消息确认模式, RabbitMQ是如何保证消息的安全, RabbmitMQ的生产者确认, RabbitMQ的消费者应答, 如何保证RabbitMQ的消息安全性]
description: "本文围绕RabbitMQ消息安全展开，介绍了生产者消息确认的默认、事务和消息确认三种模式，对比其优缺点，还阐述了消费者应答的自动和手动方式，以及消费者拒绝消息、消息预取等内容，并给出了示例代码，帮助理解RabbitMQ消息安全保障机制。"
keywords: RabbitMQ的消息确认模式, RabbitMQ是如何保证消息的安全, RabbmitMQ的生产者确认, RabbitMQ的消费者应答, 如何保证RabbitMQ的消息安全性
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/93228623
> - 发布时间：2019-06-21 21:12:32
> - 阅读量：1
> - 分类：RabbitMQ专栏收录该内容, 订阅专栏
> - 标签：#RabbitMQ的消息确认模式, #RabbitMQ是如何保证消息的安全, #RabbmitMQ的生产者确认, #RabbitMQ的消费者应答, #如何保证RabbitMQ的消息安全性

## 摘要

文章浏览阅读1.5k次。本文围绕RabbitMQ消息安全展开，介绍了生产者消息确认的默认、事务和消息确认三种模式，对比其优缺点，还阐述了消费者应答的自动和手动方式，以及消费者拒绝消息、消息预取等内容，并给出了示例代码，帮助理解RabbitMQ消息安全保障机制。

---

#### 消息确认模式

  * [1.生产者的消息确认](<#1_1>)
  *     * [1.1默认模式](<#11_5>)
    * [1.2事务模式](<#12_7>)
    * [1.3消息确认模式](<#13_9>)
    * [1.4总结](<#14_11>)
    * [1.5消息确认模式的确认方式](<#15_16>)
  * [2.消费者应答](<#2_20>)
  *     * [2.1自动应答](<#21_24>)
    * [2.2手动应答](<#22_27>)
  * [3.消费者拒绝接受消息](<#3_31>)
  * [4.消息预取](<#4_42>)
  * [5.例子](<#5_45>)
  *     * [5.1创建项目](<#51_46>)
    * [5.2添加依赖](<#52_48>)
    * [5.3创建配置类](<#53_60>)
    * [5.4创建模式枚举类](<#54_93>)
    * [5.5创建生产者](<#55_221>)
    * [5.6创建消费者](<#56_315>)
    * [5.7创建消息拒绝的消费者](<#57_369>)
    * [5.8生产消息](<#58_407>)
    * [5.9消息消费](<#59_418>)
    * [5.10拒绝消息的消费者](<#510_423>)

## 1.生产者的消息确认

  * 事务模式
  * 确认模式
  * 默认模式

### 1.1默认模式

默认不进行确认，没有任何操作，当消息生产者发送给MQ就完成了一次消息生产。并不管如果MQ发生异常导致消息丢失的问题。

### 1.2事务模式

事务模式类似数据库中的事务，但是这里的事务是基于信道的，在信道内提交事务，此时才会把消息传送给MQ，如果因为业务上某个消息生产失败，那么可以使用信道的事务的回滚操作，撤回上次提交之前生产的消息。

### 1.3消息确认模式

消息确认模式采用回调实现的。所以比起事务模式，大大的降低了生产消息的消费，而且还能保证消息的安全性。

### 1.4总结

默认模式不安全，存在消息丢失的情况，但是消耗最低，速度最快。  
事务模式比默认模式安全，不存在消息丢失的情况，但是生产消息的消耗较大，且客户端需要阻塞等待MQ的返回，导致消息生产的吞吐量不高，效率较低。  
消息确认模式最为灵活，采用异步回调的方式，不会造成阻塞。而且对于消息来说又是安全的，所以也是使用最多的模式。  
其效率远远大于事务模式。

### 1.5消息确认模式的确认方式

  * 普通确认:waitForConfirms
  * 批量确认:waitForConfirms
  * 异步确认:addConfirmListener

## 2.消费者应答

消费者应答的两种方式：

  * 自动应答
  * 手动应答

### 2.1自动应答

自动应答模式下，MQ发送给消费者，然后就会从队列中删除消息。  
自动应答模式只会保证MQ到消费者，却无法保证消费者消费异常的情况。

### 2.2手动应答

手动应答模式下，MQ发送消息给消费者，然后此消息还会在队列中存储，只有收到消费者发送的应答结果，然后才会从队列中删除消息。  
这种方式能够保证消息在消费者正确的消费，如果消费者消费出现异常，可以不进行应答，然后MQ会将消息发送给其他的消费者。  
注意：如果一个队列只有一个消费者，这个消费者没有进行手动的应答，那么可能会造成死循环导致内存溢出。

## 3.消费者拒绝接受消息

当消费者无法消费消息时，可以拒绝MQ投递的消息。  
basicReject(long deliveryTag, boolean requeue)throws IOException  
拒绝一条消息：  
deliveryTag消息唯一标识；  
requeue是否重传(false为丢弃)  
basicNack(long deliveryTag, boolean multiple, boolean requeue) throws IOException  
拒绝多个消息：  
deliveryTag拒绝消息的边界  
multiple:为true时拒绝所有deliveryTab小于当前消息的所有未确认消息；  
requeue:被拒绝的消息是否重传(false为丢弃)

## 4.消息预取

可以设置信道一次可以传输的消息数量。  
basicQos方法。

## 5.例子

### 5.1创建项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8d886301722e52a84419ededfeb8879a.png)

### 5.2添加依赖

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/570dbc3024f72dbf543dcd05090d4da1.png)
    
    
    dependencies {
        compile group: 'com.rabbitmq', name: 'amqp-client', version: '5.7.1'
        compile group: 'org.slf4j', name: 'slf4j-api', version: '1.7.26'
        compile group: 'org.slf4j', name: 'slf4j-simple', version: '1.7.26'
        testCompile group: 'junit', name: 'junit', version: '4.12'
    }
    
    

### 5.3创建配置类

一个普通的java类，定义了一些配置参数：
    
    
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
    
    

### 5.4创建模式枚举类
    
    
    package com.study.neum;
    
    /**
     * @author jiayq
     */
    
    public enum RabbitMqAckModel {
    
        /**
         * 普通确认模式
         */
        COMMON(0,"普通确认模式"),
    
    
        /**
         * 批量确认模式
         */
        BATCHACK(1,"批量确认模式"),
    
        /**
         * 异步确认模式
         */
        ASYNACK(2,"异步确认模式");
    
        /**
         * id
         */
        private int id;
    
        /**
         * name
         */
        private String name;
    
        private RabbitMqAckModel(int id, String name) {
            this.id = id;
            this.name = name;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public int getId() {
            return id;
        }
    
        public void setId(int id) {
            this.id = id;
        }
    }
    
    
    
    
    package com.study.neum;
    
    /**
     * @author jiayq
     */
    
    public enum RabbitMqModel {
    
        /**
         * 默认模式
         */
        NONE(0,"默认模式"),
    
        /**
         * 事务模式
         */
        TX(1,"事务模式"),
    
        /**
         * 消息确认模式
         */
        ACK(2,"确认模式");
    
        /**
         * id
         */
        private int id;
    
        /**
         * name
         */
        private String name;
    
        private RabbitMqModel(int id, String name){
            this.id = id;
            this.name = name;
        }
    
        /**
         * @return
         */
        public int getId() {
            return id;
        }
    
        /**
         * @param id
         */
        public void setId(int id) {
            this.id = id;
        }
    
        /**
         * @return
         */
        public String getName() {
            return name;
        }
    
        /**
         * @param name
         */
        public void setName(String name) {
            this.name = name;
        }
    }
    
    

### 5.5创建生产者
    
    
    package com.study.produce;
    
    import com.rabbitmq.client.*;
    import com.study.config.RabbitMqConfig;
    import com.study.neum.RabbitMqAckModel;
    import com.study.neum.RabbitMqModel;
    
    import java.util.Scanner;
    
    /**
     * @author jiayq
     */
    public class Produce {
    
        private static RabbitMqModel rabbitMqModel = RabbitMqModel.ACK;
    
        private static RabbitMqAckModel rabbitMqAckModel = RabbitMqAckModel.ASYNACK;
    
        public static void main(String[] args) {
            System.out.println("produce model is " + rabbitMqModel.getName());
            if (RabbitMqModel.ACK.equals(rabbitMqModel)) {
                System.out.println("produce ack model is " + rabbitMqAckModel.getName());
            }
            try (Scanner scanner = new Scanner(System.in)) {
                while (true) {
                    System.out.println("Please input message(one row one message,quit to exits):");
                    String message = scanner.nextLine();
                    if ("quit".equals(message)) {
                        System.out.println("Will be quit!!!");
                        break;
                    }
                    sendMessage(message);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    
        private static void sendMessage(String message) throws Exception{
            ConnectionFactory connectionFactory = new ConnectionFactory();
            connectionFactory.setHost(RabbitMqConfig.HOST_NAME);
            Connection connection = connectionFactory.newConnection();
            Channel channel = connection.createChannel();
            channel.queueDeclare(RabbitMqConfig.QUEUE_NAME, true, false, false, null);
            channel.exchangeDeclare(RabbitMqConfig.EXCHANGE_NAME, "direct", true, false, null);
            channel.queueBind(RabbitMqConfig.QUEUE_NAME, RabbitMqConfig.EXCHANGE_NAME, RabbitMqConfig.ROUNT_KEY);
            switch (rabbitMqModel) {
                case NONE:
                    break;
                case TX:
                    channel.txSelect();
                    break;
                case ACK:
                    channel.confirmSelect();
                    break;
                    default:
                        break;
            }
            channel.basicPublish(RabbitMqConfig.EXCHANGE_NAME, RabbitMqConfig.ROUNT_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
            switch (rabbitMqModel) {
                case NONE:
                    break;
                case TX:
                    channel.txCommit();
                    break;
                case ACK:
                    switch (rabbitMqAckModel) {
                        case COMMON:
                            channel.waitForConfirms();
                            break;
                        case BATCHACK:
                            channel.waitForConfirms();
                            break;
                        case ASYNACK:
                            ConfirmCallback confirmCallback = (deliverTag, multiple) ->{
                                System.out.println("deliverTag:\t" + deliverTag);
                                System.out.println("multiple:\t" + multiple);
                            };
                            channel.addConfirmListener(confirmCallback, confirmCallback);
                            break;
                            default:
                                break;
                    }
                    break;
                    default:
                        break;
            }
        }
    
    }
    
    

### 5.6创建消费者
    
    
    package com.study.consume;
    
    import com.rabbitmq.client.*;
    import com.study.config.RabbitMqConfig;
    
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    
    /**
     * @author jiayq
     */
    public class Consume {
    
        /**
         * 消费者应答模式
         */
        private static boolean consumeAck = false ;
    
        public static void main(String[] args) throws IOException, TimeoutException {
            ConnectionFactory connectionFactory = new ConnectionFactory();
            connectionFactory.setHost(RabbitMqConfig.HOST_NAME);
            Connection connection = connectionFactory.newConnection();
            Channel channel = connection.createChannel();
            channel.queueDeclare(RabbitMqConfig.QUEUE_NAME, true, false, false, null);
            channel.exchangeDeclare(RabbitMqConfig.EXCHANGE_NAME, "direct", true,false,null);
            channel.queueBind(RabbitMqConfig.QUEUE_NAME, RabbitMqConfig.EXCHANGE_NAME, RabbitMqConfig.ROUNT_KEY);
            DeliverCallback deliverCallback;
            CancelCallback cancelCallback;
            if (consumeAck) {
                deliverCallback = (tag,message) -> {
                    System.out.println("consumeAck:\t" + consumeAck);
                    System.out.println("message:\t" + new String(message.getBody(),"utf-8"));
                };
                cancelCallback = (tag) -> {
                    System.out.println("consume error,tag:\t" + tag);
                };
            } else {
                deliverCallback = (tag, message) -> {
                    System.out.println("consumeAck:\t" + consumeAck);
                    System.out.println("message:\t" + new String(message.getBody(),"utf-8"));
                    channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
                };
                cancelCallback = (tag) -> {
                    System.out.println("consumer error,tag:\t" + tag);
                };
            }
            channel.basicConsume(RabbitMqConfig.QUEUE_NAME, consumeAck, deliverCallback, cancelCallback);
        }
    
    }
    
    

### 5.7创建消息拒绝的消费者
    
    
    package com.study.consume;
    
    import com.rabbitmq.client.*;
    import com.study.config.RabbitMqConfig;
    
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    
    /**
     * @author jiayq
     */
    public class LazyConsume {
    
        public static void main(String[] args) throws IOException, TimeoutException {
            ConnectionFactory connectionFactory = new ConnectionFactory();
            connectionFactory.setHost(RabbitMqConfig.HOST_NAME);
            Connection connection = connectionFactory.newConnection();
            Channel channel = connection.createChannel();
            channel.queueDeclare(RabbitMqConfig.QUEUE_NAME, true, false, false, null);
            channel.exchangeDeclare(RabbitMqConfig.EXCHANGE_NAME, "direct", true, false,null);
            channel.queueBind(RabbitMqConfig.QUEUE_NAME, RabbitMqConfig.EXCHANGE_NAME, RabbitMqConfig.ROUNT_KEY);
            channel.basicQos(1);
            DeliverCallback deliverCallback = (tag, message) -> {
                System.out.println("get Message:\t" + new String(message.getBody(),"utf-8"));
                channel.basicReject(message.getEnvelope().getDeliveryTag(), true);
                channel.basicAck(message.getEnvelope().getDeliveryTag(), true);
            };
            CancelCallback cancelCallback = (tag) -> {
                System.out.println("consume error,tag:\t" + tag);
            };
            channel.basicConsume(RabbitMqConfig.QUEUE_NAME, false, deliverCallback, cancelCallback);
        }
    
    }
    
    

### 5.8生产消息

默认模式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d42fab6418b0644d9fab582bacdc5cce.png)  
事务模式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b3fa25bf8b8d64a66ad012718cd844ff.png)  
确认模式–普通确认模式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5f12a7e75eff2ad2b05373c7e828b15e.png)  
确认模式–批量确认模式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6b60034b3ab82cb45e9f21119bcf430f.png)  
确认模式–异步确认模式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dd98768eddfe0cfdd28adfca1561a8fd.png)

### 5.9消息消费

自动应答模式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ed81ae67cca0090b995919ac68ac551f.png)  
手动应答模式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/15117910935e6566eb57f5381dbc988c.png)

### 5.10拒绝消息的消费者

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18a631c96612cb3e99583b0362674c84.png)  
生产一个消息  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7996c37de96e9f3ab5d995e945e9def4.png)  
此时生产者和拒绝消息的消费者同时启动  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f8f317b6550736e97a0fd1fc9dd33976.png)  
生产了一个消息  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/23b16e4d5b03aa29ac219a109fce904c.png)  
拒绝消息的消费者受到了消息，但是拒绝处理，并要求重传。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6bf954e053bfe1587ebd9db9edda1481.png)  
在启动一个消费者  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bced1ebae8e945253d66568ba3a49e6c.png)  
被正常消费  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/14a0db1b2679421fb1adf5369871750e.png)

git仓库：  
<https://github.com/a18792721831/MQ.git>