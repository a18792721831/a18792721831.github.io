---
layout: post
title: "Java访问ActiveMQ"
date: 2019-07-08 20:08:37 +0800
categories: [activeMQ, activeMQHello, java 如何连接activeMQ, activeMQ生产者消费者, activeMQ的两种模式]
description: "Java访问ActiveMQ1.创建gradle项目2.增加依赖3.创建类4.启动服务器5.生产6.消费7.git仓库地址ActiveMQ发布订阅模式1.创建gradle项目2.增加依赖3.创建类package com.study.config;import org.apache.activemq.ActiveMQConnection;/** * @author jia..._java activemq username"
keywords: activeMQ, activeMQHello, java 如何连接activeMQ, activeMQ生产者消费者, activeMQ的两种模式
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/95090695
> - 发布时间：2019-07-08 20:08:37
> - 阅读量：523
> - 分类：ActiveMQ专栏收录该内容, 订阅专栏
> - 标签：#activeMQ, #activeMQHello, #java 如何连接activeMQ, #activeMQ生产者消费者, #activeMQ的两种模式

## 摘要

文章浏览阅读523次。Java访问ActiveMQ1.创建gradle项目2.增加依赖3.创建类4.启动服务器5.生产6.消费7.git仓库地址ActiveMQ发布订阅模式1.创建gradle项目2.增加依赖3.创建类package com.study.config;import org.apache.activemq.ActiveMQConnection;/** * @author jia..._java activemq username

---

#### Java访问ActiveMQ

  * 1.创建gradle项目
  * 2.增加依赖
  * 3.创建类
  * 4.启动服务器
  * 5.生产
  * 6.消费
  * 7.git仓库地址

  
ActiveMQ   
发布订阅模式 

## 1.创建gradle项目

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a49ba00deebe925cbeda6f50c9d69d92.png)

## 2.增加依赖

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/340cf9a3095998b2073314608d98a447.png)

## 3.创建类

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/8cd11226e2b5f4341a3a26d2f2ab00b5.png)
    
    
    package com.study.config;
    
    import org.apache.activemq.ActiveMQConnection;
    
    /**
     * @author jiayq
     */
    
    public enum ActiveMQConfig {
        /**
         * username
         */
        USERNAME(ActiveMQConnection.DEFAULT_USER),
        /**
         * password
         */
        PASSWORD(ActiveMQConnection.DEFAULT_PASSWORD),
        /**
         * url
         */
        URL(ActiveMQConnection.DEFAULT_BROKER_URL);
    
        /**
         * value
         */
        private String value;
    
        private ActiveMQConfig(String value) {
            this.value = value;
        }
    
        public String getValue() {
            return value;
        }
    
        public void setValue(String value) {
            this.value = value;
        }
    }
    
    
    
    
    package com.study.consume;
    
    import com.study.config.ActiveMQConfig;
    import org.apache.activemq.ActiveMQConnectionFactory;
    
    import javax.jms.*;
    
    /**
     * @author jiayq
     */
    public class Consume {
    
        public static void main(String[] args) throws JMSException {
            //创建连接工厂
            ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ActiveMQConfig.USERNAME.getValue(),
                    ActiveMQConfig.PASSWORD.getValue(), ActiveMQConfig.URL.getValue());
            //创建连接
            Connection connection = activeMQConnectionFactory.createConnection();
            //开启连接
            connection.start();
            //创建会话，不需要事务
            Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
            //创建主题
            Topic topic = session.createTopic("active-test");
            //创建消息消费者
            MessageConsumer consumer = session.createConsumer(topic);
            //注册监听
            consumer.setMessageListener(message -> {
                try {
                    System.out.println(((TextMessage)message).getText());
                    Thread.sleep(4 * 1000);
                } catch (JMSException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
    
    }
    
    
    
    
    package com.study.publish;
    
    
    import com.study.config.ActiveMQConfig;
    import org.apache.activemq.ActiveMQConnectionFactory;
    
    import javax.jms.*;
    import java.util.Random;
    
    /**
     * @author jiayq
     */
    public class Publish {
    
        public static void main(String[] args) {
            //创建连接工厂
            ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ActiveMQConfig.USERNAME.getValue(),
                    ActiveMQConfig.PASSWORD.getValue(),ActiveMQConfig.URL.getValue());
            try{
                //创建连接
                Connection connection = activeMQConnectionFactory.createConnection();
                //开启连接
                connection.start();
                //创建会话
                Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
                //创建主题，用于订阅消息
                Topic topic = session.createTopic("active-test");
                //消息生产者
                MessageProducer producer = session.createProducer(topic);
                while (true) {
                    //创建消息
                    TextMessage message = session.createTextMessage(new String(new Random().nextDouble() + ""));
                    System.out.println(message.getText());
                    //发送
                    producer.send(message);
                    //模拟生产消息的耗时
                    Thread.sleep(3 * 1000);
                }
            } catch (JMSException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
    
    }
    
    

## 4.启动服务器

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7ed0adb086d2bbf45e8d39d7dfda0db0.png)

## 5.生产

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/bed85789e7b54e098f18682df3a4735d.png)  
在图形化界面查看  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/4a45f414fd609fe795791404f9eba8f6.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/f96c5f54a3e0a61c2a198c6f4a6fb36d.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/32714489eae5917e6480842abd5ccee4.png)

## 6.消费

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/7b160c536d9f310dfa855f5c9f1fb719.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/0a351560b619a828c7c1e6c465383d49.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/58dd873cd3fd3efba1bda5acb877635c.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/86ad0cbad01b1b81ebdd79a200bac0fb.png)

## 7.git仓库地址

<https://github.com/a18792721831/MQ.git>
