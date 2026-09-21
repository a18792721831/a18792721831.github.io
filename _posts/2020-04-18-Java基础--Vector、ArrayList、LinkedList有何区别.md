---
layout: post
title: "Java基础--Vector、ArrayList、LinkedList有何区别"
date: 2020-04-18 20:02:22 +0800
categories: [Java数组, Vector, ArrayList, LinkedList, 数组线程安全]
description: "本文详细对比了Vector、ArrayList和LinkedList三种数据结构的优缺点，包括它们的底层实现、读写机制、效率分析及线程安全性。同时介绍了集合框架的基本概念，如集合的长度、存储特性及集合框架的分类。"
keywords: Java数组, Vector, ArrayList, LinkedList, 数组线程安全
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/105602837
> - 发布时间：2020-04-18 20:02:22
> - 阅读量：712
> - 分类：java核心专栏收录该内容, 订阅专栏
> - 标签：#Java数组, #Vector, #ArrayList, #LinkedList, #数组线程安全

## 摘要

文章浏览阅读712次。本文详细对比了Vector、ArrayList和LinkedList三种数据结构的优缺点，包括它们的底层实现、读写机制、效率分析及线程安全性。同时介绍了集合框架的基本概念，如集合的长度、存储特性及集合框架的分类。

---

#### Vector、ArrayList、LinkedList有何区别

  * 1\. 定义
  * 2.Vector
  *     * 2.1 使用new创建，默认大小是10
    * 2.2 扩容
    * 2.3 删除
  * 3\. LinkedList
  *     * 3.1 初始化
    * 3.2 add方法
    * 3.3 扩容
    * 3.4 LinkedList查找元素：
    * 3.5 LinkedList的pop&push
  * 4\. ArrayList
  *     * 4.1 初始化
    * 4.2 增加元素
    * 4.3 扩容
    * 4.4 查找元素
    * 4.5 删除元素
  * 5\. 线程安全
  * 6\. 排序
  * 7\. 静态工厂方法

## 1\. 定义

Vector 是 Java 早期提供的线程安全的动态数组，如果不需要线程安全，并不建议选择，毕竟同步是有额外开销的。Vector 内部是使用对象数组来保存数据，可以根据需要自动的增加容量，当数组已满时，会创建新的数组，并拷贝原有数组数据。

ArrayList 是应用更加广泛的动态数组实现，它本身不是线程安全的，所以性能要好很多。与 Vector 近似，ArrayList 也是可以根据需要调整容量，不过两者的调整逻辑有所区别，Vector 在扩容时会提高 1 倍，而 ArrayList 则是增加 50%。

LinkedList 顾名思义是 Java 提供的双向链表，所以它不需要像上面两种那样调整容量，它也不是线程安全的。  
Vector、ArrayList、LinkedList均为线型的数据结构，但是从实现方式与应用场景中又存在差别。

1 底层实现方式  
ArrayList内部用数组来实现；LinkedList内部采用双向链表实现；Vector内部用数组实现。

2 读写机制  
ArrayList在执行插入元素是超过当前数组预定义的最大值时，数组需要扩容，扩容过程需要调用底层System.arraycopy()方法进行大量的数组复制操作；在删除元素时并不会减少数组的容量（如果需要缩小数组容量，可以调用trimToSize()方法）；在查找元素时要遍历数组，对于非null的元素采取equals的方式寻找。

LinkedList在插入元素时，须创建一个新的Entry对象，并更新相应元素的前后元素的引用；在查找元素时，需遍历链表；在删除元素时，要遍历链表，找到要删除的元素，然后从链表上将此元素删除即可。  
Vector与ArrayList仅在插入元素时容量扩充机制不一致。对于Vector，默认创建一个大小为10的Object数组，并将capacityIncrement设置为0；当插入元素数组大小不够时，如果capacityIncrement大于0，则将Object数组的大小扩大为现有size+capacityIncrement；如果capacityIncrement<=0,则将Object数组的大小扩大为现有大小的2倍。

3 读写效率

ArrayList对元素的增加和删除都会引起数组的内存分配空间动态发生变化。因此，对其进行插入和删除速度较慢，但检索速度很快。

LinkedList由于基于链表方式存放数据，增加和删除元素的速度较快，但是检索速度较慢。

4 线程安全性

ArrayList、LinkedList为非线程安全；Vector是基于synchronized实现的线程安全的ArrayList。

需要注意的是：单线程应尽量使用ArrayList，Vector因为同步会有性能损耗；即使在多线程环境下，我们可以利用Collections这个类中为我们提供的synchronizedList(List list)方法返回一个线程安全的同步列表对象。

集合：就像是一种容器。用于存储、获取、操作对象的容器。

  1. 数组的弊端  
①数组的长度不可变 ②数组没有提供可以查看有效元素个数的方法

  2. 集合的特点  
①集合的长度是可变的  
②集合可以存储任意类型的对象  
③集合只能存储对象

  3. 集合框架  
java.util.Collection : 集合层次的根接口

    
    
        |--- java.util.List: 有序的，可以重复的。
            |--- ArrayList: 采用数组结构存储元素。 查询操作多时选择
            |--- LinkedList: 采用链表结构存储元素。 增删操作多时选择
            |--- Vector:
        |--- java.util.Set: 无序的，不允许重复。
            |--- HashSet : 是 Set 接口的典型实现类。
                判断元素是否存在的依据是：先比较 hashCode 值，若 hashCode 存在，再通过 equals() 比较内容,若 hashCode 值不存在，则直接存储
                注意：重写 hashCode 和 equals 二者需要保持一致！
                |--- LinkedHashSet: 相较于 HashSet 多了链表维护元素的顺序。遍历效率高于 HashSet ， 增删效率低于 HashSet
           |--- TreeSet : 拥有自己排序方式
                |-- 自然排序（Comparable）：
                    ①需要添加 TreeSet 集合中对象的类实现 Comparable 接口
                    ②实现 compareTo(Object o) 方法
                |-- 定制排序（Comparator）
                    ①创建一个类实现 Comparator 接口
                    ②实现 compare(Object o1, Object o2) 方法
                    ③将该实现类的实例作为参数传递给 TreeSet 的构造器
    

## 2.Vector

Vector使用了泛型。

### 2.1 使用new创建，默认大小是10

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d5c60410005cdc5587762cce29a694d0.png)  
Vector是对象数组，所以，里面只能存放Object及其子类  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82581f0bb9c43e2f40875e0bc5a41f8e.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8e3b7429ad439e097765db3fe1f16a3c.png)  
Vector里面所有的方法都是用了synchronized关键字，所以是线程安全的。(jdk8)  
因为其有modCount，所以也不能直接remove.  
需要使用迭代器进行删除。  
当然，在jdk8中，使用流操作更加方便。

### 2.2 扩容

查看其addAll方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2024daa5314640be4d171f841c149bb2.png)  
ensureCapacityHelper是扩容的关键;  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8b4053b8cc49316ade2ba630ddd98df5.png)  
这里注意这一行：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a7e677ae3f0577bdd4aaa3254d87b025.png)  
这个newCapacity就是决定到底扩容到多大的值。  
我们在搜索的时候发现，这个变量在初始化的时候会被初始化为0：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a44f85f541cbefefa04e9353d35cea2.png)  
然后在没有任何地方会去修改这个值。  
那么：
    
    
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                             capacityIncrement : oldCapacity);
    

其实就等于
    
    
    int newCapacity = oldCapacity + oldCapacity;
    

即，2倍扩容。  
但是也存在一些类似数组的操作，比如删除某一个元素  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8587f554fece86ab37c016b50d611995.png)

### 2.3 删除

我们 看下 其删除的源码：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f491d61d14d4a8c783fcb1d546f074f0.png)  
remove有两个方法，一个是序列，一个是对象。  
那么，如果我要删除对象，但是这个数组又存储的是Integer的包装类的时候，因为没有进行自动装箱，导致，你传入的Integer对象 被认为是index，存在ArrayIndexOutOfBoundsException异常  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/963a57d4656e88c46b57b87d609ddcd3.png)  
那么，如果手动装箱呢？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3bebd63cf1d0c85899893b9d2d19bd46.png)  
可以看到其只调用了一次，但是我实际上可能是想清楚掉2个10。  
而在编程的时候，其实并不知道需要调用几次，所以，有了一个类似流的过滤的方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f93e26468c382a0bd4f691d584d3551e.png)

## 3\. LinkedList

链表，其插入、删除 性能好，但是随即访问性能差。  
其数据结构是链表，而Vector和ArrayList是数组。  
数组的随即访问性能好，插入、删除性能差。  
链表的的插入、删除性能好，随即访问性能差。

### 3.1 初始化

链表在初始化的长度是0，即空链表  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9be9a03e08869846b85e4cb7be5aa11f.png)

### 3.2 add方法

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f7fd2857803ebe5433c2afab6ec87644.png)  
JDK8中，LinkedList默认是尾插法  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b49f3b85d5aae189e9e56f59c4f91215.png)  
如果链表为空，就讲这个元素放到头结点，否则放到尾节点。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cbb9a20da66672efb598220d7a1a60d7.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/830465af5b30d1cbdf56f3713a1e3708.png)  
这实现的还是双向链表。

### 3.3 扩容

双向链表，那么就可以两端扩容。  
所以，不仅仅有尾插法，还有头插法。  
可以选择将元素放到链表的头或者尾。  
LinkedList有一个set方法，我们知道，链表随即方法性能比较差劲，那么其set方法可以将指定元素放到指定位置，怎么实现性能最大化呢？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/482c7c5776b07f679d8ca3dd2759d3c7.png)  
这个方法实现了，将原来index位置上的元素取出来，然后将新元素放进去，最后返回原来index位置上的元素。  
那么，这就存在一个问题，怎么找index位置。  
作为链表，它是无法随即访问的，只能从头部或者尾部(需要保存头结点和尾节点，有些链表可能只保存了头结点或者尾节点)进行寻找。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f570d25309a62477c510f0e153c93dfe.png)  
它是将index与size/2进行比较，如果在前半段，就从头结点开始查找，否则从尾节点开始查找。

### 3.4 LinkedList查找元素：

ListedList有一个查找指定元素位置的方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dc8ecb7bd7bfa64733bdd940fa74880b.png)  
发现LinkedList可以存放null元素。  
如果找不到这个元素，就返回-1

### 3.5 LinkedList的pop&push

LinkedList也有pop与push，也是，双向链表，且保存了头结点与尾节点，实现pop与push非常正常  
不过，LinkedList的pop与push是对头结点进行操作的：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fb11da52228b72c8f72654a7e22b41a8.png)

## 4\. ArrayList

ArrayList应该是我们经常使用的数组的一个结构了，虽然他不是多线程安全的，但是因为其性能等原因综合方面比较平衡，所以在平时使用较多。

### 4.1 初始化

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/86c2ddeede73e4572684a6c18aecc427.png)  
ArrayList在初始化时，会创建一个空数组。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1bf8794c3ef5b0bc58653c50761df62.png)  
这个数组也是存储Object的数组，意味着，我们在存储基本类型时，会涉及到自动拆箱和自动装箱。  
如果你仔细看这个空数组，一定看到了它是使用final修饰的，final修饰，意味着不可变，那么，它是如何增加元素的呢？

### 4.2 增加元素

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e71f9b13896d60dca1c57ef04ee1067.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1376a28b4c245c2316d7269b30fbaeb7.png)  
发现它在add的时候，会将当前size+1进行计算，在计算方法里面会进行判断，如果是空数组，那么取默认大小和size+1的数值大的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e761cd539236d5e445e4cd81bfe90e7b.png)  
默认大小是10；  
也就是说，ArrayList在使用new创建之后，并不会立即开辟数组空间，而是使用空数组代替。只有当第一个元素被加入的时候，才会真正的开辟数组空间，空间大小是10.也就是说，ArrayList是一个懒开辟。

### 4.3 扩容

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/13b5eea4895da5ba9feb9540d18c65a2.png)  
在加入元素时，如果size+1大于数组长度，就需要进行扩容了，扩容的方法如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c348604a03ca3c0bba7c149d1a4e994d.png)  
其newCapacity是扩容之后数组的真实长度，扩容的机制是  
newCapacity=oldCapacity + oldCapacity/2  
即50%扩容。

### 4.4 查找元素

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/34a272094bfb51e1aa172ea989628990.png)  
查找元素使用indexOf方法进行调用。  
从查找的方法可以得到，ArrayList支持null。  
而且，其查找策略是从前到后正向查找。  
利用equals方法进行判断是否相等。

### 4.5 删除元素

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/864f3d03e6becd082088d2412557adc6.png)  
删除元素方法，调用一次，只会删除1个目标元素，不会删除所有的目标元素。  
这个删除是指直接删除，不是循环删除，这里非常注意。

## 5\. 线程安全

在 Collections 工具类中，提供了一系列的 synchronized 方法，比如
    
    
    static <T> List<T> synchronizedList(List<T> list)
    

我们完全可以利用类似方法来实现基本的线程安全集合：
    
    
    List list = Collections.synchronizedList(new ArrayList());
    

它的实现，基本就是将每个基本方法，比如 get、set、add 之类，都通过 synchronizd 添加基本的同步支持，非常简单粗暴，但也非常实用。注意这些方法创建的线程安全集合，都符合迭代时 fail-fast 行为，当发生意外的并发修改时，尽早抛出 ConcurrentModificationException 异常，以避免不可预计的行为。

## 6\. 排序

Java 提供的默认排序算法，具体是什么排序方式以及设计思路等。  
需要区分是 Arrays.sort() 还是 Collections.sort() （底层是调用 Arrays.sort()）；什么数据类型；多大的数据集（太小的数据集，复杂排序是没必要的，Java 会直接进行二分插入排序）等。

  * 对于原始数据类型，目前使用的是所谓双轴快速排序（Dual-Pivot QuickSort），是一种改进的快速排序算法，早期版本是相对传统的快速排序
  * 而对于对象数据类型，目前则是使用TimSort，思想上也是一种归并和二分插入排序（binarySort）结合的优化排序算法。TimSort 并不是 Java 的独创，简单说它的思路是查找数据集中已经排好序的分区（这里叫 run），然后合并这些分区来达到排序的目的。

另外，Java 8 引入了并行排序算法（直接使用 parallelSort 方法），这是为了充分利用现代多核处理器的计算能力，底层实现基于 fork-join 框架（专栏后面会对 fork-join 进行相对详细的介绍），当处理的数据集比较小的时候，差距不明显，甚至还表现差一点；但是，当数据集增长到数万或百万以上时，提高就非常大了，具体还是取决于处理器和系统环境。

在 Java 8 之中，Java 平台支持了 Lambda 和 Stream，相应的 Java 集合框架也进行了大范围的增强，以支持类似为集合创建相应 stream 或者 parallelStream 的方法实现，我们可以非常方便的实现函数式代码。阅读 Java 源代码，你会发现，这些 API 的设计和实现比较独特，它们并不是实现在抽象类里面，而是  
以默认方法的形式实现在 Collection 这样的接口里！这是 Java 8 在语言层面的新特性，允许接口实现默认方法，理论上来说，我们原来实现在类似 Collections 这种工具类中的方法，大多可以转换到相应的接口上。

## 7\. 静态工厂方法

在 Java 9 中，Java 标准类库提供了一系列的静态工厂方法，比如，List.of()、Set.of()，大大简化了构建小的容器实例的代码量。根据业界实践经验，我们发现相当一部分集合实例都是容量非常有限的，而且在生命周期中并不会进行修改。但是，在原有的 Java 类库中，我们可能不得不写成：
    
    
    ArrayList<String>  list = new ArrayList<>();
    list.add("Hello");
    list.add("World");
    

而利用新的容器静态工厂方法，一句代码就够了，并且保证了不可变性。
    
    
    List<String> simpleList = List.of("Hello","world");
    

更进一步，通过各种 of 静态工厂方法创建的实例，还应用了一些我们所谓的最佳实践，比如，它是不可变的，符合我们对线程安全的需求；它因为不需要考虑扩容，所以空间上更加紧凑等。

如果我们去看 of 方法的源码，你还会发现一个特别有意思的地方：我们知道 Java 已经支持所谓的可变参数（varargs），但是官方类库还是提供了一系列特定参数长度的方法，看起来似乎非常不优雅，为什么呢？这其实是为了最优的性能，JVM 在处理变长参数的时候会有明显的额外开销，如果你需要实现性能敏感的 API，也可以进行参考。
