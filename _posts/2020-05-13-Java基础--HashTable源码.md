---
layout: post
title: "Java基础--HashTable源码"
date: 2020-05-13 21:10:03 +0800
categories: [HashTable, hashTable源码, HashTable原理, HashTable面试知识, 因HashTable引发的悲剧]
description: "本文详细剖析了Java中HashTable的工作原理，包括其继承关系、内部数据结构、关键方法的实现及性能考量。探讨了HashTable如何确保线程安全，以及其在不同场景下的表现。"
keywords: HashTable, hashTable源码, HashTable原理, HashTable面试知识, 因HashTable引发的悲剧
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/106060603
> - 发布时间：2020-05-13 21:10:03
> - 阅读量：653
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, Java8
> - 标签：#HashTable, #hashTable源码, #HashTable原理, #HashTable面试知识, #因HashTable引发的悲剧

## 摘要

文章浏览阅读653次。本文详细剖析了Java中HashTable的工作原理，包括其继承关系、内部数据结构、关键方法的实现及性能考量。探讨了HashTable如何确保线程安全，以及其在不同场景下的表现。

---

#### Java基础--HashTable源码

  * [1.Map接口](<#1Map_7>)
  * [2.Dictionary](<#2Dictionary_33>)
  * [3.HashTable](<#3HashTable_36>)
  *     * [3.1 全局属性](<#31__39>)
    * [3.2 辅助类](<#32__54>)
    *       * [3.2.1 Entry](<#321_Entry_56>)
      * [3.2.2 Enumerator](<#322_Enumerator_71>)
      * [3.2.3 KeySet](<#323_KeySet_91>)
      * [3.2.4 ValueCollection](<#324_ValueCollection_96>)
      * [3.2.5 EntrySet](<#325_EntrySet_101>)
      * [3.2.6 关系](<#326__105>)
    * [3.3 HashTable方法](<#33_HashTable_107>)
    *       * [3.3.1 HashTable构造](<#331_HashTable_108>)
      * [3.3.2 HashTable的普通方法](<#332_HashTable_127>)
      * [3.3.3 HashTable的迭代方法](<#333_HashTable_130>)
      * [3.3.4 contains方法](<#334_contains_133>)
      * [3.3.5 containsKey方法](<#335_containsKey_138>)
      * [3.3.6 get方法](<#336_get_141>)
      * [3.3.7 put方法](<#337_put_144>)
      * [3.3.8 addEntry](<#338_addEntry_148>)
      * [3.3.9 rehash](<#339_rehash_155>)
      * [3.3.10 clear](<#3310_clear_162>)
      * [3.3.11 clone](<#3311_clone_165>)
      * [3.3.12 迭代访问](<#3312__168>)
      * [3.3.13 hashCode](<#3313_hashCode_173>)
      * [3.3.14 getOrDefault](<#3314_getOrDefault_178>)
      * [3.3.15 forEach](<#3315_forEach_181>)
      * [3.3.16 remove](<#3316_remove_186>)
      * [3.3.17 replace](<#3317_replace_189>)
      * [3.3.18 writeObject](<#3318_writeObject_192>)
      * [3.3.19 readObject](<#3319_readObject_195>)
  * [4\. 总结](<#4__197>)

  
HashTable的类图   
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f349849ed5b01ab228b2ecab1be6d0f2.png)   
可以看到HashTable继承于Dictionary实现了Map、Cloneable、Serializable接口。   
其中Cloneable和Serializable是标记性接口。标记性接口，就是说接口里面没有定义任何的方法，一个类实现接口，也不需要实现任何的方法。这些接口存在意义只是标识，这些类可以做什么操作。   
这里讨论的主要是HashTable、Dictionary和Map接口。   
所以，暂时略过Cloneable和Serializable接口。 

## 1.Map接口

Map接口实现了泛型，接受两个类型。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dc01c137165d4a5f40b58b71c983737b.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0f92bd37fb51ad6a943b945477d2b64.png)  
这是Map接口定义的方法：（我们这里只关注下jdk8新增的方法，其他的方法暂不关注。特别是默认实现的方法）

  * getOrDefault 这是默认方法，如果存在指定键的值，那么就返回值，否则返回默认值。
  * forEach 默认方法，传入一个BiConsumer，内部实现是增强for循环。
  * replaceAll 默认方法，传入BiFunction.增强for循环实现。
  * putIfAbsent 默认方法，传入key,value。如果不存在指定的键值对，那么就将键值对放入map.  
会先调用get方法，如果get结果为空，然后在put.
  * remove默认方法，如果根据键得到的值为空或者与指定的值不同，或者不存在键，那么会移除失败。  
移除成功返回true。
  * replace 更新指定键的值，即使指定的键与预期的旧值不同，也会更新为新值。指定键如果不存在，覆盖失败。
  * replace 指定的键不存在或者指定的键对应的值不存在，会replace失败。
  * computeIfAbsent 如果指定的键对应的值不存在，会将指定的键进行运算后的值，作为指定键的值。如果指定的键对应的值存在，那么就返回已有的值。
  * computeIfPresent 指定的键值对存在，那么对指定的键值对进行运算。如果运算的结果为空，那么移除指定的键值对；如果运算的结果不为空，那么将运算得到的新值加入到map中。
  * compute 根据指定的键值对进行运算，如果运算结果为空，且map中存在指定的键对应的键值对，那么就移除此键值对。如果指定的键对应的旧值为空，那么什么也不做。如果运算的新值不为空，那么将将新值作为此键的值。不存在则添加，存在则覆盖。
  * merge 如果指定的键对应的值为空，那么将传入的值作为最终值，如果最终值为空，那么移除键，如果最终结果不为空，那么覆盖原值。如果指定的键对应的值不为空，那么将旧值和传入的值，代入式子进行计算，计算结果作为最终值，如果最终值为空，那么移除键，否则增加或覆盖原值。

Entry作为Map里面元素的键值对数据结构，接口定义了基本的方法。  
Entry的访问控制是默认权限，也就是包内。  
这里看下jdk8增加的默认方法：

  * comparingByKey返回一个比较器。比较器是将键值对的键按照自然顺序进行排序。
  * comparingByValue返回一个比较器。比较器是将键值对的值按照自然顺序进行排序的。
  * comparingByKey根据传入的比较，返回一个比较器。默认是自然顺序，这个是指定排序规则。
  * comparingByValue根据传入的比较，返回一个比较器。默认是自然顺序，这个是指定排序规则。

## 2.Dictionary

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0ccb25053ff7dd3fd0a6cea3839420f.png)  
Dictionary是一个抽象类，除了默认的无参构造，其余方法都是抽象的。(为什么不做成接口呢？)

## 3.HashTable

HashTable集成了Dictonary，实现了Map接口。  
所以，在1,2中的抽象方法和接口中的方法，都需要实现。

### 3.1 全局属性

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5827b1a1412ff3b3dff53f2cd869bf6d.png)  
数据核心的存储结构是一个不序列化的Entry数组
    
    
    private transient Entry<?,?>[] table;
    

  * table是数据存储的数组；
  * count是记录数组中元素的个数
  * threshold是表示需要扩容的阈值。阈值 = 容器容量*扩容因子。
  * loadFactor是扩容因子。一般默认0.75
  * modCount是修改记录版本。
  * keySet内存可见，用volatitle修饰
  * entrySet内存可见
  * values也是内存可见

### 3.2 辅助类

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/47a41955ea00c12bafb7c7a455d53eff.png)

#### 3.2.1 Entry

内部私有静态类，实现了Map中的Entry接口。  
全局属性：  
final int hash;  
final K key;  
V value;  
Entry<K,V> next;  
看到这里，可以看出，HashTable的存储结构是链表。  
数据结构类似下图  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ba561d5626b8ff4507d407ecd9bfa15c.png)  
Entry的hashCode方法是：key的hashCode和value的hashCode进行按位与。  
这个Entry的hashCode方法主要涉及计算整个HashTable的hashCode。  
HashTable的hashCode方法是将存储结构内所有的键值对的hashCode进行累加，累加结果作为HashTable的hashCode返回。  
也就是  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c3274b5b50be64a9c4e5a501193552c.png)

#### 3.2.2 Enumerator

这是HashTable的核心遍历器。  
其全局属性：

  * table hashTable的数组(哈希桶数组)。
  * index hashTable的当前操作的数组元素(当前的哈希桶，默认为数组结尾)
  * entry hashTable的元素(哈希桶元素)。
  * lastReturned 当前的hashTable的元素
  * type 遍历模式
  * iterator 是否可遍历
  * expectedModCount 开始遍历的版本  
遍历器有一个两参数的构造方法：模式，是否可遍历。  
模式共有3个模式：key,values,entries  
分别遍历键、值、元素  
方法：
  * hasMoreElements是否存在前一个hashTable数组元素(是否存在前一个哈希桶)(从后向前)
  * nextElement 根据遍历模式，返回模式对应的元素的值。
  * hasNext 代理指向hasMoreElements
  * next 判断当前的版本与开始遍历的版本是否相同，不同则抛出ConcurrentModificationException异常，相同则代理指向nextElement
  * remove 验证可遍历，当前遍历元素不为空，当前版本与开始遍历版本相同。进入synchronized代码块：当前遍历元素的hashCode计算出当前元素的table位置:hash & 0x7fffffff % table.length;然后在哈希桶中找到此元素，然后从链表结构中取出当前遍历的元素，然后将当前遍历的元素置空。最后会将开始遍历的版本和当前版本加1。

#### 3.2.3 KeySet

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4811cc0b253ac754e350e78e8123efd.png)  
KeySet继承抽象类AbstractSet  
实现了一些抽象方法。  
这些抽象方法都是代理，指向HashTable的方法或者是迭代器的方法。

#### 3.2.4 ValueCollection

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f781716e5798cf28ebc82002a8e6d115.png)  
ValueCollection继承抽象类AbstractCollection  
实现了一些抽象方法。  
这些抽象方法都是代理，指向HashTable的方法或者是迭代器的方法。

#### 3.2.5 EntrySet

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8cf40c3173bbba61e655c3ff4f48cb9e.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c0ec49e79915a5cb908171b7d4bc86d.png)  
EntrySet也是在Enumerator的基础上封装形成的，一些其他的方法还是调用HashTable自己实现的方法。

#### 3.2.6 关系

将辅助类进行分类，可以根据其功能，分为：存储节点Entry，Enumartor迭代器，KeySet,EntrySet,ValueCollection是迭代器代理。

### 3.3 HashTable方法

#### 3.3.1 HashTable构造

HashTable的构造方法可以指定初始大小，也能指定扩容因子，但是不能指定初始扩容阈值。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a492d8488ff9eff1363866de0c923eb7.png)  
从构造方法可以看出，不能创建空的HashTable，即使你指定初始默认大小是0，也会修改为1.  
而且，HashTable在创建的时候，就会将数组空间分配。

注意，这里的初始大小，不是元素的个数的大小，而是初始的哈希桶或者说哈希槽的个数。扩容因子是针对哈希桶进行扩容的乘数因子。但是扩容阈值，却与HashTable中元素的个数有关。  
初始的阈值是哈希槽的数量的扩容因子倍。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4353f7c91f817e496d1cdd48b910b57a.png)  
扩容因子默认是0.75.  
也就是说，HashTable的扩容是75%扩容的。(哈希槽扩容，注意和元素可存储的个数的扩容不同)  
ArrayList是100%扩容。（移位实现）  
Vector也是100%扩容。(Vector还可以定义不可扩容)  
LinkedList不存在扩容问题。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16e3c106d532710c01518ccadec251ac.png)  
HashTable如果不指定初始哈希槽的大小，那么可以存放11*0.75=8.25即8个元素。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/139edee92ec5266ed91cd48cf0084e45.png)  
根据已有集合创建hashTable的时候，如果集合的元素个数的2倍大于11个。那么将哈希槽设置为集合元素数量个，否则就设置为11个。  
所以，HashTable的默认初始哈希槽的大小就是11.

#### 3.3.2 HashTable的普通方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8a65e7cc925e586e83a4c00df6f5dbfa.png)  
HashTable的方法是线程安全的。不过这个线程安全是通过synchronized的关键字实现的。在以前的jdk版本，synchronized的性能比较差，不过随着jdk对synchronized的性能的优化，使用到synchronized的性能影响越来越小了。

#### 3.3.3 HashTable的迭代方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/62f27e3be0b3a853bbaeead9c459871e.png)  
可以发现，通过创建迭代器的模式不同，实现不同属性的遍历。

#### 3.3.4 contains方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e10adc116b3a93ef860d5fd3e724d07.png)  
在内部，contains方法是通过双重for循环实现的。  
所以，这个方法的性能也好不到哪去。  
不过，用起来很方便。

#### 3.3.5 containsKey方法

通过HashTable的结构，因为所有元素都是在一个个的哈希槽下面的，而哈希槽的哈希值是通过key的hashCode计算的。所以，判断一个key是否存在，我们只需要重新计算key的hashCode即可。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/235118416b0c6ad84e8dd1cefd752da9.png)

#### 3.3.6 get方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c3d7074d975d93b7ba91e8f8f3cdd702.png)  
get方法是通过计算key的hash值，找到key对应的哈希槽，然后遍历哈希槽内的元素链表。如果key的hashCode和key都相等，就返回这个key对应的value.

#### 3.3.7 put方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48bcfe5a24732adc623153f6d5412a18.png)  
其put方法首先会判断value是否为空，结合hashTable每次操作都是根据key,hashCode进行的。可以推断出，hashTable不管是键还是值，都不允许为空。  
在加入hashTable前，先根据key的哈希值，找到对应的哈希槽，然后遍历哈希槽中链表的数据，看看我们要加入的数据是否已经存在。也就是说，hashTtable不允许键值同时存在重复。而且，根据其put的逻辑，如果键相同，那么会将新值覆盖旧值，然后返回旧值。

#### 3.3.8 addEntry

如果键值都不相同，那么就需要增加元素：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0261948188d5e2f6fa8371e087b70763.png)  
在增加元素的时候，首先会将版本号加1.  
然后判断现有数量是否达到扩容阈值。如果到达扩容阈值，就行重新哈希，进行扩容。然后将待加入的元素加入。  
如果没到达阈值，如果已有哈希槽，那么就使用链表的头插法，将待加入元素加入此哈希槽下的链表头部，然后将新加入的链表头放到哈希槽。  
如果没有哈希槽，那么新创建的哈希槽只有一个元素，这个元素就是链表的头，而且链表头在哈希槽。

#### 3.3.9 rehash

重新哈希：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0e5cf2967bb26d18406c81a871442e0.png)  
在重新哈希的时候，会将现有的哈希槽的数量扩大一倍+1.  
然后更新版本号，更新扩容的阈值。  
然后将旧的hastTable中的哈希槽进行重新哈希，然后将旧哈希槽对应的链表的链表头赋值给新哈希槽。  
实现hashTable的扩容。

#### 3.3.10 clear

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4528713b04d454f25da97d64025bb7f0.png)  
clear方法是将每个哈希槽对应的链表清空。

#### 3.3.11 clone

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/93a44c26aa447f664c8a069d735c38cf.png)  
克隆方法会循环克隆当前的hashTable的属性，不过会清空版本号。

#### 3.3.12 迭代访问

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a9a70649e2e4c4ee6df69df08f687e7.png)因为内部的迭代器都是私有的，为了实现外界使用迭代器，需要暴露方法，这个暴露方法就是获取内部私有的迭代器。  
外界持有的是接口的实例。  
hashTable内部确实是私有类的迭代器，但是其实现的接口是共有的。  
外界需要共有接口的实例，此时内部通过上转型，将私有内部类的实例，突破权限控制，暴露给外界，然后进行遍历。

#### 3.3.13 hashCode

hashTable内部，几乎所有的操作都需要用到key的hashCode.那么hashTable本身的hashCode是如何计算的呢？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bd472ab1e597fc8fdf29213f3c46c2ff.png)  
是对hashTable内每一个元素都进行hashCode的计算，然后得出的hashCode.  
从其实现的过程，我们可以发现对于hashTable的嵌套使用，性能比较差，特别是将hashTable作为HashTable的key的时候，其性能会差到极点。

#### 3.3.14 getOrDefault

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a7d6dbe07a48ed10202c3e04a687dc4f.png)  
获取指定键的值，如果不存在，返回默认值。

#### 3.3.15 forEach

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ef09264e17cbed9281b0e55b47b5a6fc.png)  
发现forEach中会对开始遍历的版本和当前的版本进行比较，只要不同就会通过抛出异常快速失败。  
所以，在forEach中不能进行put操作，也不能进行remove操作。  
但是可以进行更新操作，因为更新操作不会修改版本号。

#### 3.3.16 remove

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/372b578b76155fbf3ba732f8bb50d73b.png)  
在remove时，会将版本号加1，所以，remove前后，版本号变了。

#### 3.3.17 replace

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/08d55602982e376cedf3dbec0da61c89.png)  
replace前后，版本号也是不变的。

#### 3.3.18 writeObject

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/50593624be344c28e2fdc33619e03b48.png)  
这个方法是在序列化写的时候用到，对应的还有序列化读的方法。

#### 3.3.19 readObject

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/abc92613fac8434db11a3785a5e4d573.png)

## 4\. 总结

看了这么多的for循环，发现在hashTable中，好多的for循环都是从后向前的。  
如果这个for循环给我来写，可能我写的是这样的：
    
    
    for(int i = table.length - 1 ; i >= 0; i--){
    //......
    }
    

但是hashTable里面的写法，却比我写的少一些：
    
    
    for(int i = table.length;--i>=0;){
    //......
    }
    

从这里来看，至少少写了 i–, - 1  
对于我们一般的遍历，这可能没有什么操作，但是对于集合框架来说，少写2个代码单词，可能对于所有使用集合的人来说，节省了时间。  
有时候，我也在想，我们看jdk的源码的目的是什么？  
了解一些非常cool的api,然后在工作中cool的使用出来？  
为了让其他人在问你时，你可以避免被刨根问底？  
为了面试？  
可能都有吧。  
但是我觉得至少还有一点，作为java开发人员，或者说现在大多数的系统都是基于java语言实现，那么，jdk相当于是我的核心武器。作为一个士兵，我应该对自己的武器非常熟悉才对。  
作为武器，我应该比较了解，不仅仅是了解其使用，也是了解这个武器的一些原理。  
摸清这个武器的实现过程，尝试理解这个武器的优劣。  
最终的目的是根据对核心武器的使用，能够创造出更加适合自己的武器。  
从而让自己的能力不断提高。  
当然，这只是一个java码农对自己的一个小小的认知与规划吧。