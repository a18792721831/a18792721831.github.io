---
layout: post
title: "ActiveMQ集成到Spring"
date: 2019-07-12 19:37:54 +0800
categories: [spring集成ActiveMQ, 如何简化ActiveMQ的访问流程, ActiveMQ集成到spring中, 将ActiveMQ的管理交给spring, 降低ActiveMQ的连接成本]
description: "ActiveMQ集成到Spring1.ActiveMQ访问流程2.ActiveMQ集成到spring2.1创建一个gradle项目2.2 增加依赖2.3创建spring依赖2.4 创建消费者消息监听2.5 生产者服务2.6 启动ActiveMQ服务器2.7创建启动main方法2.8启动3.总结1.ActiveMQ访问流程生产1.创建连接工厂2.创建连接3.开启连接4.创建会话5.创建..._activemq如何集成到spring项目的?"
keywords: spring集成ActiveMQ, 如何简化ActiveMQ的访问流程, ActiveMQ集成到spring中, 将ActiveMQ的管理交给spring, 降低ActiveMQ的连接成本
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/95647617
> - 发布时间：2019-07-12 19:37:54
> - 阅读量：638
> - 分类：ActiveMQ专栏收录该内容, 订阅专栏
> - 标签：#spring集成ActiveMQ, #如何简化ActiveMQ的访问流程, #ActiveMQ集成到spring中, #将ActiveMQ的管理交给spring, #降低ActiveMQ的连接成本

## 摘要

文章浏览阅读638次。ActiveMQ集成到Spring1.ActiveMQ访问流程2.ActiveMQ集成到spring2.1创建一个gradle项目2.2 增加依赖2.3创建spring依赖2.4 创建消费者消息监听2.5 生产者服务2.6 启动ActiveMQ服务器2.7创建启动main方法2.8启动3.总结1.ActiveMQ访问流程生产1.创建连接工厂2.创建连接3.开启连接4.创建会话5.创建..._activemq如何集成到spring项目的?

---

#### ActiveMQ集成到Spring

  * [1.ActiveMQ访问流程](<#1ActiveMQ_1>)
  * [2.ActiveMQ集成到spring](<#2ActiveMQspring_16>)
  *     * [2.1创建一个gradle项目](<#21gradle_17>)
    * [2.2 增加依赖](<#22__19>)
    * [2.3创建spring依赖](<#23spring_27>)
    * [2.4 创建消费者消息监听](<#24__99>)
    * [2.5 生产者服务](<#25__191>)
    * [2.6 启动ActiveMQ服务器](<#26_ActiveMQ_229>)
    * [2.7创建启动main方法](<#27main_231>)
    * [2.8启动](<#28_259>)
  * [3.总结](<#3_262>)

## 1.ActiveMQ访问流程

生产  
1.创建连接工厂  
2.创建连接  
3.开启连接  
4.创建会话  
5.创建主题  
6.创建生产者  
7.创建消息  
8.发布  
9.关闭资源  
消费  
6.创建消费者  
7.注册监听  
那么有很多的操作其实是重复的，所以用spring去管理这部分相同的处理逻辑，大大的简化ActiveMQ的访问流程。

## 2.ActiveMQ集成到spring

### 2.1创建一个gradle项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf4e2cc13c7ccd7d8c340ea4ec69480b.png)

### 2.2 增加依赖

这里有一个小坑：  
就是使用的什么仓库中心，那么就在这个仓库中心搜索使用的版本号等等。  
比如之前我使用的是阿里云的仓库中心，但是在maven的仓库中搜索的jar，然后增加了依赖，导致在阿里的仓库中无法下载jar.  
为了解决上述问题，最好把两种仓库中心都配置：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/537fda8c0ada2c57f1bb84186481fc0d.png)  
然后增加依赖  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c0763cf3feb69bf85a5bd2881d07f9a9.png)

### 2.3创建spring依赖
    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
          <!--注解扫描的包-->
       <context:component-scan base-package="com.study"/>
       <!-- 创建连接工厂，指定url -->
       <bean id="jmsFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
           <property name="connectionFactory">
               <bean class="org.apache.activemq.ActiveMQConnectionFactory">
                   <property name="brokerURL">
                       <value>tcp://localhost:61616</value>
                   </property>
               </bean>
           </property>
       </bean>
       <!-- 使用缓存连接工厂管理 -->
       <bean id="cachingConnectionFactory" class="org.springframework.jms.connection.CachingConnectionFactory">
           <property name="targetConnectionFactory" ref="jmsFactory"/>
           <!-- 设置最大连接数为1 -->
           <property name="sessionCacheSize" value="1"/>
       </bean>
       <!-- 创建连接模板，指定了消息转换器 -->
       <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
           <property name="connectionFactory" ref="cachingConnectionFactory"/>
           <property name="messageConverter">
               <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
           </property>
       </bean>
       <!-- 声明目标队列 -->
       <bean id="testQueue" class="org.apache.activemq.command.ActiveMQQueue">
           <constructor-arg name="name" value="spring_queue"/>
       </bean>
       <!-- 声明目标主题 -->
       <bean id="testTopic" class="org.apache.activemq.command.ActiveMQTopic">
           <constructor-arg index="0" value="active-test"/>
       </bean>
       <!-- 注册队列监听 -->
       <bean id="queueListener" class="com.study.consume.QueueListener"/>
       <!-- 注册主题监听 -->
       <bean id="topic1Listener" class="com.study.consume.Topic1Listener"/>
       <!--注册主题监听-->
       <bean id="topic2Listener" class="com.study.consume.Topic2Listener"/>
       <!--创建队列转换器-->
       <bean id="queueContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
           <!--使用的连接工程-->
           <property name="connectionFactory" ref="cachingConnectionFactory"/>
           <!--转换的目标队列或主题-->
           <property name="destination" ref="testQueue"/>
           <!--转换器绑定的监听器-->
           <property name="messageListener" ref="queueListener"/>
       </bean>
       <!--主题转换器-->
       <bean id="topic1Container" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
           <property name="connectionFactory" ref="cachingConnectionFactory"/>
           <property name="destination" ref="testTopic"/>
           <property name="messageListener" ref="topic1Listener"/>
       </bean>
       <!--主题转换器-->
       <bean id="topic2Container" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
           <property name="connectionFactory" ref="cachingConnectionFactory"/>
           <property name="destination" ref="testTopic"/>
           <property name="messageListener" ref="topic2Listener"/>
       </bean>
    </beans>
    

### 2.4 创建消费者消息监听

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6845509fdd497a9e1f8d14616c0ba37a.png)
    
    
    package com.study.consume;
    
    import javax.jms.JMSException;
    import javax.jms.Message;
    import javax.jms.MessageListener;
    import javax.jms.TextMessage;
    
    /**
     * @author jiayq
     */
    public class QueueListener implements MessageListener {
        @Override
        public void onMessage(Message message) {
            if (message instanceof TextMessage) {
                TextMessage textMessage = (TextMessage) message;
                try {
                    String messageText = textMessage.getText();
                    System.out.println("message:\t" + messageText);
                    System.out.println("queue Listener");
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            } else {
                throw new IllegalArgumentException("Only Text Message");
            }
        }
    }
    
    
    
    
    package com.study.consume;
    
    import javax.jms.JMSException;
    import javax.jms.Message;
    import javax.jms.MessageListener;
    import javax.jms.TextMessage;
    
    /**
     * @author jiayq
     */
    public class Topic1Listener implements MessageListener {
        @Override
        public void onMessage(Message message) {
            if (message instanceof TextMessage) {
                TextMessage textMessage = (TextMessage) message;
                try {
                    String messageText = textMessage.getText();
                    System.out.println("message:\t" + messageText);
                    System.out.println("topic listener1");
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            } else {
                throw new IllegalArgumentException("Only Text Message");
            }
        }
    }
    
    
    
    
    package com.study.consume;
    
    import javax.jms.JMSException;
    import javax.jms.Message;
    import javax.jms.MessageListener;
    import javax.jms.TextMessage;
    
    /**
     * @author jiayq
     */
    public class Topic2Listener implements MessageListener {
        @Override
        public void onMessage(Message message) {
            if (message instanceof TextMessage) {
                TextMessage textMessage = (TextMessage) message;
                try {
                    String messageText = textMessage.getText();
                    System.out.println("message:\t" + messageText);
                    System.out.println("listener2");
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            } else {
                throw new IllegalArgumentException("Only Text Message");
            }
        }
    }
    
    

### 2.5 生产者服务

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3c5d3790549de9dcaf497e3ae4d8fd48.png)
    
    
    package com.study.product;
    
    import org.springframework.jms.core.JmsTemplate;
    import org.springframework.stereotype.Service;
    
    import javax.annotation.Resource;
    import javax.jms.Destination;
    import java.util.Random;
    
    /**
     * @author jiayq
     */
    @Service("produceService")
    public class ProduceService {
    
        @Resource(name = "jmsTemplate")
        private JmsTemplate jmsTemplate;
    
        @Resource(name = "testQueue")
        private Destination testQueue;
    
        @Resource(name = "testTopic")
        private Destination testTopic;
    
        public void sendMessage(String messageContent) {
            jmsTemplate.send(testQueue, session -> session.createTextMessage(new Random().nextDouble() + "queue produce"));
        }
    
        public void sendTopicMessage(String messageContent) {
            jmsTemplate.send(testTopic, session -> session.createTextMessage(new Random().nextDouble() + "topic produce"));
        }
    
    }
    
    

### 2.6 启动ActiveMQ服务器

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e643e83cb89ed2a171c074c540308d91.png)

### 2.7创建启动main方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/67276ea8dcf244b93dbcdeb14849b7ed.png)
    
    
    package com.study.client;
    
    import com.study.product.ProduceService;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import java.util.Random;
    
    /**
     * @author jiayq
     */
    public class StartClient {
        public static void main(String[] args) throws InterruptedException {
            ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
            ProduceService produceService = applicationContext.getBean(ProduceService.class);
            while (true) {
                produceService.sendMessage("queueTestMessage:\t" + new Random().nextDouble());
                produceService.sendTopicMessage("topicTestMessage:\t" + new Random().nextDouble());
                Thread.sleep(3 * 1000);
            }
    
        }
    }
    
    

### 2.8启动

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eca2ceb58735db8ae8375fd84e9112f3.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/51b4b48bba3e2f2d096fc46a99b94aff.png)

## 3.总结

在实例项目中，采用了注解的方式，所以在spring的配置文件中需要指定注解扫描的包的目录。  
接着定义了一个jms工厂bean，采用的是池化连接工厂类PooledConnectionFactory.实际上就是对内部的ActiveMQ连接工厂增加了连接池的功能，从其内部配置可以看出，它就是对ActiveMQConnectionFactory的封装。  
接下来使用的cachingConnectionFactory是实际项目中常用的，是对连接工厂的又一层增强，使用连接的缓存功能来提升效率。  
jmsTemplate就是Spring用来解决JMS访问时代码冗长和重复的方案，需要配置connectionFactory和messageConverter,通过connectionFactory获取连接、会话等对象，messageConverter则用于配置消息转换器，因为通常消息在发送前和接收后都需要进行一个前置和后置处理，转换器就是做这个工作的。  
这样实际代码直接通过jmsTemplate来发送和接收消息，而每次发送和接收消息时创建连接工厂、创建连接、创建会话等工作都由spring框架完成。  
有了jms模板，还需要知道队列或者主题的作为实际发送和接收消息的目的地，所以接下来定义了testQueue和test-Topic作为两种模式的实例。  
异步接收消息需要提供MessageListener的实现类，所以定义了queueListener作为队列模式下异步接收消息的监听器；主题模式用两个监听器是为了测试当有多个消费者时，都能收到消息。  
最后queueContainer、topic1Container和topic2Container用于将消息监听器绑定到具体的消息目的地。  
@Service将该类声明为一个服务，在实际项目中很多服务代码也是类似的。通过@Resource注解直接将上面配置文件中定义的jmsTemplate引入MessageService类中就可以直接使用了，testQueue和testTopic也是类似的。  
在服务勒种直接引入在配置文件中定义好的队列和主题。sendQueueMessage向队列发送消息，sendTopicMessage向主题发送消息，这两种模式都使用了jmdTemplate的send方法，该方法的第一个参数是Destination类型，表示消息目的地。第二个参数是MessageCreator，这里使用Lambda表达式。