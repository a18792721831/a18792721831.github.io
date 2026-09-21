---
layout: post
title: "redis--在spring中使用"
date: 2020-08-01 19:01:35 +0800
categories: [Spring集成Redis, Spring中使用Redis, Spring对Redis的封装, Spring-Redis的转换, 将Redis集成到项目中]
description: "本文深入讲解Spring框架如何整合Redis缓存工具，包括依赖配置、连接池管理、序列化策略及数据类型操作。通过实例演示了如何使用RedisTemplate进行键值存储、批量操作及序列化控制。"
keywords: Spring集成Redis, Spring中使用Redis, Spring对Redis的封装, Spring-Redis的转换, 将Redis集成到项目中
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107735240
> - 发布时间：2020-08-01 19:01:35
> - 阅读量：1
> - 分类：Redis专栏收录该内容, 订阅专栏
> - 标签：#Spring集成Redis, #Spring中使用Redis, #Spring对Redis的封装, #Spring-Redis的转换, #将Redis集成到项目中

## 摘要

文章浏览阅读1.3k次。本文深入讲解Spring框架如何整合Redis缓存工具，包括依赖配置、连接池管理、序列化策略及数据类型操作。通过实例演示了如何使用RedisTemplate进行键值存储、批量操作及序列化控制。

---

#### 在spring中使用redis

  * [redis依赖](<#redis_1>)
  * [spring-data-redis](<#springdataredis_13>)
  * [RedisConfig](<#RedisConfig_27>)
  * [RedisTemplate](<#RedisTemplate_105>)
  * [Redis序列化](<#Redis_140>)
  * [字符串序列化器](<#_165>)
  * [Spring 对Redis数据类型操作的封装](<#Spring_Redis_183>)
  * [Spring 对Redis批量操作的封装](<#Spring_Redis_197>)
  * [SessionCallback和RedisCallback](<#SessionCallbackRedisCallback_215>)

## redis依赖

redis是一个很强大的缓存工具，那么，在Java项目中如何使用呢？

redis连接驱动，使用的最多的目前应该是Jedis。

在maven中央仓库搜索[jedis](<https://search.maven.org/artifact/redis.clients/jedis/3.3.0/jar>)

![image-20200801153453181](https://i-blog.csdnimg.cn/blog_migrate/00832406b4a318ee4b935bac4d6dc729.png)

最新是3.3.0。

## spring-data-redis

spring提供了一个RedisConnectionFactory的接口，通过它可以生成一个RedisConnection接口对象，而RedisConnection接口对象是对Rediis底层接口的封装。

这是RedisConnection相关的结构图

![RedisConnectionFactory](https://i-blog.csdnimg.cn/blog_migrate/450028f0eec5c565e3ab93787def9a35.png)

简化下：

![img](https://i-blog.csdnimg.cn/blog_migrate/18e2dd0771b3035f7e3b1056158ce647.png)

在spring中是通过RedisConnection接口操作Redis的，而RedisConnection则对原生的Jedis进行封装。要获取RedisConnection接口对象，是通过RedisConnectionFactory接口去生成的，所以第一步需要配置RedisConnectionFactory。

## RedisConfig

首先创建一个配置类：
    
    
    @Configuration
    public class RedisConfig {
    
        @Value("${spring.redis.host}")
        private String hostname;
    
        @Value("${spring.redis.port}")
        private int port;
    
        @Value("${spring.redis.password}")
        private String password;
    
        @Value("${spring.redis.timeout}")
        private int timeout;
    
        @Value("${spring.redis.database}")
        private int database;
    
        private RedisConnectionFactory factory = null;
    
        @Bean("redisConnectionFactory")
        public RedisConnectionFactory initConnectionFactory() {
            if (factory != null) {
                return factory;
            }
            JedisPoolConfig poolConfig = new JedisPoolConfig();
            // 最大空闲数
            poolConfig.setMaxIdle(10);
            // 最大连接数
            poolConfig.setMaxTotal(25);
            // 最大等待毫秒数
            poolConfig.setMaxWaitMillis(timeout);
            // 创建连接工厂
            JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(poolConfig);
            // 获取单机的Redis配置
            RedisStandaloneConfiguration rsConfig = jedisConnectionFactory.getStandaloneConfiguration();
            rsConfig.setHostName(hostname);
            rsConfig.setPort(port);
            rsConfig.setPassword(password);
            rsConfig.setDatabase(database);
            factory = jedisConnectionFactory;
            return factory;
        }
    
    }
    

配置文件：
    
    
    spring:
      redis:
        host: 10.0.228.117
        port: 6379
        password: ""
        timeout: 2000
        database: 0
    

通过一个连接池创建了RedisConnectionFactory，通过factory创建RedisConnection对象。再试用的时候，要先从RedisConnectionFactory工厂获取，然后使用，使用完成后，需要手动关闭。

我们在test方法中尝试用下：

![image-20200801165515369](https://i-blog.csdnimg.cn/blog_migrate/893953fa45221d4ff38c3cbca4ee1936.png)

执行testFactory方法

![image-20200801165538898](https://i-blog.csdnimg.cn/blog_migrate/c2bfe30a6cbd64061b04aa87c51f5f00.png)

然后使用客户端连接查看

![image-20200801165605846](https://i-blog.csdnimg.cn/blog_migrate/ccbe9a7d450814664ff54c3b72943747.png)

## RedisTemplate

单纯的使用RedisConnection,每次使用前需要获取连接，使用完需要释放连接，如果忘记释放，那么很快就会耗尽redis连接池的连接。

所以，spring提供了[RedisTemplate](<https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:template>)

![image-20200801165844883](https://i-blog.csdnimg.cn/blog_migrate/8d86a3c25c135e24ad500ca1b09d927d.png)

ok,我们继续尝试，首先是创建RedisTemplate
    
    
        @Bean("redisTemplate")
        public RedisTemplate initRedisTemplate() {
            RedisTemplate template = new RedisTemplate();
            template.setConnectionFactory(initConnectionFactory());
            return template;
        }
    

![image-20200801171509642](https://i-blog.csdnimg.cn/blog_migrate/be756477a69f87ab2858bc309afd184a.png)

然后在test中尝试使用：

![image-20200801171528957](https://i-blog.csdnimg.cn/blog_migrate/45de01d1adcad73f979157faff2b3ed7.png)

执行结果：

![image-20200801171550565](https://i-blog.csdnimg.cn/blog_migrate/1d7b235c8b2dd5a671fbf25061af4ac0.png)

发现并没有输出，我们使用客户端查看

![image-20200801171618201](https://i-blog.csdnimg.cn/blog_migrate/cac96e5698c6a01f5775a3d9dfa1d191.png)

咦，我们给的key是test，为什么存储后是xxxxtest呢？

## Redis序列化

Redis是给予字符串存储的NoSql，而Java是基于帝乡的语言，对象是无法存储到Redis中的，不过Java提供了序列化机制，只要类实现了Serializable接口，就代表能够进行序列化。通过将类对象进行序列化就能得到二进制字符串，然后Redis就可以将类对象以字符串的方式进行存储。java也可以将二进制字符串，进行反序列化转为对象。Spring提供了序列化器的机制，并实现了几个序列化器。

![xxxx](https://i-blog.csdnimg.cn/blog_migrate/fb02315ae8c8a18cb2faa1dcece6f7ce.png)

这是spring实现的序列化器

![image-20200801173129749](https://i-blog.csdnimg.cn/blog_migrate/a7dc7a5df221cd9005b233a6d6ba4837.png)

![image-20200801173429114](https://i-blog.csdnimg.cn/blog_migrate/add603da2981208e15a2b4c9a9c585ac.png)

RedisTemplate中可以配置的序列化器

| 属性                  | 描述                   | 备注                                        |
|---------------------|----------------------|-------------------------------------------|
| defaultSerializer   | 默认序列化器               | 如果没有设置，则使用JdkSerializationRedisSerializer |
| keySerializer       | Redis键序列化器           | 如果没有设置，则使用默认序列化器                          |
| valueSerializer     | Redis值序列化器           | 如果没有设置，则使用默认序列化器                          |
| hashKeySerializer   | Redis Hash field序列化器 | 如果没有设置，则使用默认序列化器                          |
| hashValueSerializer | Redis Hash value序列化器 | 如果没有设置，则使用默认序列化器                          |
| stringSerializer    | 字符串序列化器              | RedisTemplate自动赋值为StringRedisSerializer   |

因为我们对RedisTemplate什么样的序列化器都没有配置，所以使用默认的JdkSerializationRedisSerializer进行序列化和反序列化，也就是说，将字符串按照对象进行序列化了，所以我们设置的键前面还有一些字符。

## 字符串序列化器

我们将RedisTemplate的序列化器指定为stringSerializer.

![image-20200801181159768](https://i-blog.csdnimg.cn/blog_migrate/5be40b81a2ff6e2ba63177b0861db3ff.png)

然后重新执行

![image-20200801181304624](https://i-blog.csdnimg.cn/blog_migrate/d391c9b5c40bb8aa647ec9d7c17a0c20.png)

然后查看redis：

![image-20200801181322843](https://i-blog.csdnimg.cn/blog_migrate/9530a33055003341471e0a28d2126122.png)

忘记设置valueSerializer了

![image-20200801181421361](https://i-blog.csdnimg.cn/blog_migrate/9817f00ebf7bd4cdfb86fa24976f2db5.png)

## Spring 对Redis数据类型操作的封装

Redis能够支持7种类型的数据结构,常用的5种数据结构：string,hash,list,set,sorted set

redisTemplate获取数据类型操作接口：

| redis数据结构  | 操作接口获取                      |
|------------|-----------------------------|
| string     | redisTemplate.opsForValue() |
| hash       | redisTemplate.opsForHash()  |
| list       | redisTemplate.opsForList()  |
| set        | redisTemplate.opsForSet()   |
| sorted set | redisTemplate.opsForZSet()  |

## Spring 对Redis批量操作的封装

同样的，在spring中，也支持对某一个key进行批量操作：

redisTemplate对批量操作的支持：

| redis数据类型  | spring批量操作的封装                      |
|------------|------------------------------------|
| string     | redisTemplate.boundValueOps(“key”) |
| hash       | redisTemplate.boundHashOps(“key”)  |
| list       | redisTemplate.boundListOps(“key”)  |
| set        | redisTemplate.boundSetOps(“key”)   |
| sorted set | redisTemplate.boundZSetOps(“key”)  |

![image-20200801182856400](https://i-blog.csdnimg.cn/blog_migrate/8af99addfa7f5d8ba64d0f4e91b6a4de.png)

![image-20200801182907972](https://i-blog.csdnimg.cn/blog_migrate/5f9c523dd1dfb45f14f927622ea58d92.png)

## SessionCallback和RedisCallback

RedisTemplate的回调

通过使用回调，可以在同一个连接下执行多个Redis命令。

其中SessionCallback比RedisCallback拥有更多的封装，使用起来更加友好。

SessionCallback
    
    
        @Test
        public void testRedisSessionCallback() {
            redisTemplate.execute((RedisConnection rc) -> {
                rc.set("session1".getBytes(), "session1".getBytes());
                rc.set("session2".getBytes(), "session2".getBytes());
                return null;
            });
        }
    

![image-20200801184913178](https://i-blog.csdnimg.cn/blog_migrate/60c11e554addcd015a0db7dc27bc9b10.png)

RedisCallback
    
    
        @Test
        public void testRedisCallback() {
            redisTemplate.execute((RedisConnection rc) -> {
                rc.set("redis1".getBytes(), "redis1".getBytes());
                rc.set("redis2".getBytes(), "redis2".getBytes());
                return null;
            });
        }
    

![image-20200801185815374](https://i-blog.csdnimg.cn/blog_migrate/5084e750a55d305afe6746db5b8513d1.png)