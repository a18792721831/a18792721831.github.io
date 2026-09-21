---
layout: post
title: "Java基础--HashMap源码"
date: 2020-05-23 23:40:57 +0800
categories: [HashMap源码, HashMap详解, HashMap的组成, HashMap的存储原理, 全面解读HashMap]
description: "HashMap几乎是我们开发中用到的最多的数据结构之一了，但是，HashMap的源码实现你读过吗？_java hashmap keyset源码"
keywords: HashMap源码, HashMap详解, HashMap的组成, HashMap的存储原理, 全面解读HashMap
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/106265498
> - 发布时间：2020-05-23 23:40:57
> - 阅读量：625
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, Java8
> - 标签：#HashMap源码, #HashMap详解, #HashMap的组成, #HashMap的存储原理, #全面解读HashMap

## 摘要

文章浏览阅读625次。HashMap几乎是我们开发中用到的最多的数据结构之一了，但是，HashMap的源码实现你读过吗？_java hashmap keyset源码

---

#### Java基础--HashMap源码

  * 1 HashMap实现的接口
  * 2 HashMap继承的类
  * 3 HashMap 数据获取
  *     * 3.1 Iterable
    *       * 3.1.1 Iterator
      * 3.1.2 Consumer
      * 3.1.3 Spliterator
    * 3.2 Collection
    * 3.3 AbstractCollection
    * 3.4 Set
    * 3.5 AbstractSet
    * 3.6 KeySet -> HashMap
    * 3.7 ValueSet > HashMap
    * 3.8 EntrySet -> HashMap
  * 4 HashMap 的迭代器
  *     * 4.1 HashIterator
    * 4.2 Iterator
  * 5\. Node
  *     * 5.1 Node(链表)
    * 5.2 TreeNode
  * 6\. Spliterator
  *     * 6.1 Spliterator调用链
    * 6.2 Spliterator属性
    * 6.3 Spliterator 操作
    * 6.4 KeySpliterator,ValueSpliterator,EntrySpliterator
  * 7\. HashMap 其他的常用的操作
  *     * 7.1 初始化
    * 7.2 put
    * 7.3 链表化
    * 7.4 hashCode
  * 8\. 总结

说明，以下java环境都是基于jdk8u111进行。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/282e9ea4bcc69b8b8dc6e5b4da8da711.png)  
这是使用idea的功能，查看类的关系。  
可以看到HashMap继承了AbstractMap，实现了Map接口，Cloneable接口和Serializable接口。  
但是，这个结构依然有些粗糙。  
HashMap的结构，可以说一点都不简单。  
HashMap的类图可以如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/752893197a4cd30f59eb84ed4ddd9d63.png)  
这什么玩意？  
不要着急，慢慢来。

## 1 HashMap实现的接口

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/47fb39f5ec95acd526d377987dd3509b.png)  
可以看到，HashMap的结构主要是：继承AbstractMap实现Map接口。  
其他的Serializable和Cloneable主要是标记性接口，没有方法定义。  
Map主要定义了作为Map的操作：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a82f1b10a2084484c588151f68a3961d.png)  
Map定义了Map的通用操作。包括：size,isEmpty,containsKey,containsValue,get,put,putAll,clear,KetSet,values,entrySet操作。  
对于这些操作，可以分为这些：

  * Map指标：size,isEmpty
  * Map操作：put,get,remove,clear,putAll
  * Map查询：containsKey,containsValue
  * Map遍历：keySet,values,entrySet  
在Map内部，定义了Map实际存储数据的数据结构是Entry  
同样的，Map内部的Entry使用时用Map.Entry 访问。  
可以分为以下操作：
  * Map.Entry操作：getKey,getValue,setValue
  * Map.Entry比较操作：comparingByKey,comparingByValue

## 2 HashMap继承的类

HashMap继承了AbstractMap。而AbstractMap实现了Map的接口，其主要的作用是对Map和Map.Entry定义的接口，提供了实现。  
HashMap继承了AbstactMap，那么， 在HashMap中，只需要实现HashMap需要实现的方法即可。当然，作为抽象类，肯定也指定了一些必须要实现的方法：
    
    
    public abstract Set<Entry<K,V>> entrySet();
    
    

看到这里，你一定比较奇怪，为什么AbstractMap没有强制子类实现keySet和valueSet?  
因为，Entry是包含了key和value的。也就是说，我拿到了entry，那么，我就拿到了key和value.

在AbstractMap中，是这样实现的：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1dd6adb514ed86e76eed86157375fe4b.png)  
可以看到，他使用了AbstractSet，通过调用entrySet的方法获取得到对应的Map.Entry，然后调用Map.Entry来实现keySet和valueSet.  
至于AbstractSet，请继续往下看。

## 3 HashMap 数据获取

在1.2中，我们提到，Map定义的遍历的操作分别是:keySet,values,entrySet.通过这三个方法可以获取到Map的数据。  
为了实现这个目的，HashMap在内部定义了三个类：KeySet,Values,EntrySet。  
因为在定义的时候，定义了KeySet，Values，EntrySet返回的必须是Set.  
所以，具体的实现中，返回的是Set的抽象类AbstractSet。  
其具体的结构图如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ec90db87145771cca5d4f0b4389fdf5.png)  
而这些类是定义在HashMap类下的，所以姑且我们称这些类内实现的类为组合类。  
即，在HashMap内存在三个组合类：EntrySet，KetSet和Values.  
这些组合类的结构如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eca2e2d00dbd8855df4613bf1896462f.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1454a62d4d207a6a967b8491f5549887.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5ab346278857748472513bfaacc31960.png)

从这三个组合类的结构，可以分析出：key，entry不允许重复，而value允许重复。 

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/80f971ce8a7b91d004aa8d031c2ff47e.png)

如果key相同，那么就是覆盖，这个 后面会详细说明。

### 3.1 Iterable

这个接口非常的重要，只有实现了这个接口的数据接口，才能拥有迭代的能力。所以，这个接口，我个人更愿意理解为迭代性接口，或者能否迭代接口。

能否迭代接口共有三个方法：

  1. iterator()：这个接口返回当前数据接口的一个迭代器。
  2. forEach方法：这是jdk8中增加的默认方法，传入一个Consumer，然后在方法内使用增强for循环对每一个数据使用Consumer定义的操作。
  3. spliterator方法：我更愿意称这个操作是分割器，或者说遍历器更加准确。

OK，看到这里，发现更加迷茫了，本来是了解Iterable，怎么Iterable没整明白，反而又多了三个新的玩意：  
Iterator，Consumer，Spliterator

#### 3.1.1 Iterator

抱着打破砂锅问到底的态度，我们在继续看看Iterator是什么玩意。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5927bf04ef138fbb32884e87326fbbe9.png)  
通过查看源码发现，Iterator共有4个方法，其中forEachRemaining是在jdk8中增加的默认方法。  
其他三个操作都是我们在使用迭代器时使用的方法。  
使用迭代器的几个步骤：  
① 获取迭代器对象  
② 是否存在下一个迭代对象  
③ 获取并偏移迭代指针  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/263b9215c3304968b7fb8d6b856bcccf.png)  
当然，还有一个方法是remove方法。对于jdk提供的集合框架里面的数据结构，都不能在遍历的时候进行删除操作。因为这存在一个悖论：  
当迭代最后一个元素的时候，移除了一个元素。那么此时，整个数据结构的长度是小于当前元素正在遍历的索引位置，那么这是数组越界的问题。但是如果认为这个是数组越界了，当前遍历的元素却能获取到。  
为了解决这一问题，在jdk提供的结合框架的数据结构里面，使用一个开始遍历的版本号与当前数据结构的版本号验证的一个过程，当发现当前版本号与开始遍历的版本号不一致，就会抛出ConcurrentModificationException异常。  
既然有这样的验证，那为什么在迭代的时候却能够进行remove操作。  
首先明确一点，遍历和迭代是两个不同的概念。  
举个简单的例子：  
for循环是遍历。  
增强for循环是迭代。  
而上面使用while是迭代。

迭代与遍历最大的区别是迭代在循环过程中，可以动态的修改数据的长度。

这里有一个很大的误区，增强for循环也无法在循环中动态的修改数据的长度，为什么认为增强for循环也是迭代器呢？

我们通过上述的示例代码可以得出，作为迭代器，我们需要hasNext判断能进行下一次迭代，使用next方法进行获取当前元素，并且将迭代指针偏移到下一个迭代的元素。

所以，我们写一个增强for循环，然后，在hasNext的方法打个断点，不要忘记在for循环的地方也打上断点。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb40f942a0f55958c43162f2f854f832.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99a160868705bf55a7930158c6f44d8c.png)  
通过这个验证，可以看到，增强for循环也是迭代器，只是没有提供迭代器对象。而是隐式的调用迭代器的next方法，然后将值赋予增强for循环的定义的变量。  
那么，遍历是如何写的呢？  
在之前我们阅读数组的源码的时候，可以通过下标获取数组元素。但是HashMap与数组的结构完全不同。可以说，对于HashMap只能通过迭代遍历。当然，对于HashMap的key或者value或者entry，获取到都是数组存储的。这时候，就可以使用数组的遍历方式进行遍历了。  
有点扯远了，在次回到Iterator，在Iterator中有4个方法，我们已经看了next和hasNext的方法的作用。接下来看看其他的方法。  
我们知道，在迭代的时候，可以实现动态修改数据长度。通过调用Iterator的remove方法实现的。在数组遍历的时候，如果动态的修改数组的长度，会抛出ConcurrentModificationException异常。  
原因与数组遍历的原因相同。在HashMap中有一个版本号的概念，在遍历的时候，为了避免悖论，保证开始遍历的版本号与当前版本号相同，否则就会抛出ConcurrentModificationException异常。  
而迭代器的remove方法中，会进行同步版本号。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/92d2cee1e1a0edecfeb092a1e82ec565.png)  
不过，也不要想着，使用HashMap的remove方法，用完后使用Iterator的remove方法进行同步版本号。这样也是行不通的，因为在Iterator的remove方法中，会先进行版本号验证。  
最后一个就是forEachRemaining方法了。这个方法是jdk8中增加的，为了支持lambda表达式的forEach方法。是一个默认方法。

#### 3.1.2 Consumer

在Iterable中的第二个方法是支持lambda方法的，其参数是Consumer。  
那么，在这个小节中，我们主要搞明白，Consumer是什么。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e4f0d2c612866d78050e11b3bfad0776.png)  
Consumer是一个接口，有FunctionalInterface注解修饰。也就是说，Consumer是一个函数接口。  
其内部有两个方法：accept和andThen。那么，这两个方法是干什么的呢？  
首先，accept是一个接口方法，我们通过定义lambda函数，说明我们要做的操作。然后传入一个对象，对这个对进行操作。其返回值是空。  
这样说可能不是非常明确，请看例子：
    
    
    import java.util.Arrays;
    import java.util.function.Consumer;
    
    /**
     * @author jiayq
     * @Date 2020-05-23
     */
    public class Main {
    
        public static void main(String[] args) {
    
            Consumer<Integer> consumer = x -> x = x+1;
            Consumer<String> stringConsumer = new Consumer<String>() {
                @Override
                public void accept(String s) {
                    System.out.println(s);
                }
            };
            int x = 88;
            consumer.accept(x);
            System.out.println(x);
            stringConsumer.accept("hello");
            Consumer<People> peopleConsumer = people -> people.name = people.name + people.name;
            People people;
            peopleConsumer.accept(people = new People("hello "));
            System.out.println(people.name);
    
            Consumer<Integer> integerConsumer = y -> System.out.println(y);
            Arrays.asList(1,2,3).forEach(integerConsumer.andThen(integerConsumer));
    
            MyConsumer myConsumer = str -> str = str + str;
            MyInterface myInterface = string -> string = string + string;
        }
    
        @FunctionalInterface
        interface MyConsumer{
            void accept(String string);
        }
    
        interface MyInterface{
            void accept(String string);
        }
    
        static class People{
            String name;
    
            People(String s){
                name = s;
            }
        }
    
    }
    
    

说白了，Consumer就是只有一个方法的接口的实现，在支持lambda函数之前，我们只能通过匿名内部类的方式实现接口方法。但是因为Consumer的接口必须只有一个为实现的接口方法，所以，我们实现的操作一定是这个接口方法。而lambda可以用更加直观，更少的代码实现。  
接下来是andThen方法。因为Consumer的特殊性，必须有且只有一个未实现的接口。如果需要执行的操作比较复杂，那么整个实现的操作就非常的复杂，不容易阅读。所以，为了更好的将操作进行划分，就必须将复杂的操作进行抽取操作。而抽取的操作对应了多个方法，这时候，矛盾就出现了。为了解决这个矛盾，Consumer可以进行链化。可以将多个操作进行组合。这里是类似装饰器实现的链式操作。  
操作的先后顺序是调用者先执行，传入的Consumer后执行。  
还有一个点，函数接口和普通的接口是一样的，加不加注解，程序都能实现想要的目的。只不过，加了注解后，更加的清晰。而且对于某些特殊的框架，可以进行额外的处理。

#### 3.1.3 Spliterator

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f19eaef3ee4e42976cd6b1cca4c5ace6.png)  
Spliterator我将其理解为分割迭代器。  
这个方法我们几乎不会用到，所以，我不会深入讨论这个方法的实现。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/49afd68e7e2c31fcb9d24da5fda2147c.png)  
在Iterable中的方法是这样实现的，通过简单查看其注释，大概是一个迭代其实现的方法，通过快速失败进行迭代。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/50dfffa1a904727e35f289d575323e87.png)  
在HashMap中存在4个Spliterator，后面会详细说明这些的操作。

### 3.2 Collection

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9a972354c3bf639a228677a180749e37.png)  
Collection和Iteraotor的关系是这样的：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c32cb3a272da8113758ce2e965a47057.png)  
Collection定义了集合数据结构的通用操作，比如描述集合的size，是否为空；以及集合查询的是否包含；流相关的操作，流化，并行流化；集合操作，增加，删除，清空，更新等操作。

### 3.3 AbstractCollection

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fa2916ededc8f13d499be81d5df0cddc.png)  
AbstractCollection要求子类必须实现iterator和size方法，其余是给Collection接口提供了默认实现。我个人认为，这样的是因为，继承了Collection的数据结构，计算总数量各不相同，而且其迭代的方式也各不相同，所以这部分要求子类必须实现。而其他的方法则都是相同的逻辑。  
Collection获取数据结构的元素的时候，使用的是迭代器的next获取元素并偏移，调用hasNext判断是否可以进行下一次的迭代。这样，抽象了数组，map，set等数据结构的迭代或者说遍历的操作。  
所以，迭代器在Collection中是非常重要的一个基础。

### 3.4 Set

在HashMap中一共有三个数据遍历器，分别是KeySet,ValueSet,EntrySet.  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7491a5ffc82922658b4ffbe05e39f678.png)  
通过查看各个迭代器的继承关系，一个重要的区别是valueSet没有实现Set的接口，也没有继承AbstrctSet抽象类。  
这小节主要是阅读Set接口。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/991b39ea6c37e0f9b42149cd8716efb2.png)  
这是Set的继承关系图。  
看下Set主要定义了些什么操作：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c4a54b7b9f0904735157dfc3ab3abc37.png)  
可以很清楚的看出，Set并没有增加自己独特的接口方法，都是实现器继承类的方法：Collection和Iterable.  
那么，Set不可重复，是谁来保证呢？我们继续往下看。

### 3.5 AbstractSet

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/decdf702fda013065afb1281dd322b93.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1f984e7d29dbc3dc95e8ea7a8d1a5d4.png)  
AbstractSet继承了AbstractCollection的同时，实现了Set接口。但是Set接口并没有定义新的操作，所以Abstract只是实现了自己的hashCode和equals和removeAll操作。并没有定义新的操作，也没有做很大的改动。

### 3.6 KeySet -> HashMap

终于到第一个正主了，KeySet应该是我们使用的最多的数据集了，一般都使用增强for循环遍历hashMap的key，然后使用map.get方法获取key对应的value,然后在对value进行操作。  
那么，HashMap中的keySet是如何实现的呢？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a5244d829c368b6a5e915c4c9ef0a663.png)  
通过查看其代码实现，keySet更像是一个代理或者说访问者，我们通过ketSet方法，获得了一个set，然后使用增强for循环的语法糖，进行遍历HashMap的key.  
而增强for循环用到的迭代器是HashMap中的keyIterator。

### 3.7 ValueSet > HashMap

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f29b5372c013534fd8ad877a90692ecb.png)  
说实话，工作学习中，我自己很少用到ValueSet这个数据集，甚至，在阅读源码之前都不知道有ValueSet这个数据集。但是有时候，我们确实需要遍历全体HashMap，然后根据HashMap的值进行一些操作或者运算，那么，我认为此时使用valueSet数据集，比使用KeySet数据集可能更好一些。

### 3.8 EntrySet -> HashMap

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c26ff8d6f294e37e80d1fa8aed1b95bf.png)  
EntrySet数据集也是比较少用的一个数据集，使用EntrySet可以获取HashMap中的一个存储节点。这个数据集与其他两个数据集最大的区别就是，通过一次迭代，可以既获取key，又获取value.对于需要根据key和value同时进行运算或者操作的时候，可以减少一次迭代的次数。在一定程度上可以增加代码密度。

## 4 HashMap 的迭代器

在1.3中我们讨论了HashMap的数据遍历，但是通过代码查看，数据遍历的数据集都是一个个的代理，实际指向了迭代器。接下来我们看看HashMap的迭代器。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eafb5079717aa6dc160c2b841fd85014.png)  
在HashMap中实现了4个迭代器，其中一个是抽象的迭代器HashIterator,其余的迭代器都是继承了这个抽象的迭代器。

### 4.1 HashIterator

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/108e3be8e30ba42779b05f1da62568b7.png)  
HashIterator没有继承任何借口和抽象类，这个抽象类只是把HashMap的一些基本操作进行封装，同时定义了，一个HashMap的迭代器进行迭代需要的一些属性。

  * expectedModCount是开始迭代的版本号
  * index是当前迭代的哈希槽索引
  * current是当前迭代的节点
  * next是下一个迭代的节点

请注意这个抽象迭代器的构造方法，里面设置了开始迭代的版本号为创建迭代器的HashMap的版本号。  
同时将第一个迭代的节点移动到哈希桶或者说哈希槽的第一个节点。  
这个do-while循环将index移动到了哈希槽的第一个槽位。  
current和next是为了实现remove方法。一般情况下，我们调用next方法获取节点元素，然后通过判断节点元素的值是否符合条件，然后根据判断结果进行remove。  
但是，请不要忘记，在我们调用next的时候，已经将迭代指针移动到了下一个节点了即next，然后当前节点保存在current。我们调用了remove操作，会重新定位迭代指针，也会重新构建节点存储。说简单点，就是节点的指针需要进行更新，需要做一个删除节点的操作。此时current的节点的指针就会被用于获取当前元素的下一个元素，然后将当前元素的上一个元素的指针指向下一个元素。  
打个比方，你从自行车链条中取下一个链条节点，需要将这个链条节点的前一个和后一个连起来。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d7b8c6a362269e37f9dc9a0ad936d0cf.png)  
在remove方法里面，remove之前或进行版本验证，remove之后，会同步版本号。  
关于Iterator的4个方法：hasNext,next,remove，HashIterator已经实现了2个，只剩下next方法未实现。不过它实现了nextNode方法。这个方法就是获取并偏移节点的方法。

### 4.2 Iterator

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dcb0b8ce34fb99ae4f2ac9841fb26298.png)  
所以，剩下的对应的Iterator只需要获取对应的节点的属性即可。  
这个抽象的很厉害，很巧妙。  
代码密度相当高。

## 5\. Node

在HashMap中，一个个的哈希槽中存储的是一个个的Node。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/92e2ffec4cde9154891467c2a1a2acbe.png)  
那么，这个Node是什么？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2b126a77bcd9aa54e4508b21ede7e001.png)

### 5.1 Node(链表)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/386ae8308e08d0fa465fe80914c68c7d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ed1c7fd61d3594dc65fe0c902e118f4.png)  
众所周知，HashMap在存储的时候，有两种存储状态：链表，红黑树。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/400b126041e97487c1c15b0969ceb1bc.png)  
链表实现继承了Map.Entry接口。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/63e55fdf4335c7bb70f0a64fd5cdaa7f.png)  
链表的节点实现重写了hashCode，对key和value进行hash异或。

### 5.2 TreeNode

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c8fd84454e599c531761a15aecf74e5.png)  
这是一个红黑树。红黑树相当复杂，我自己都没有理解透彻。  
这里放两篇文章：  
<https://3g.163.com/news/article/FDA40U250511FQO9.html?from=history-back-list>  
<https://www.jianshu.com/p/e136ec79235c>  
提供一个动态演示的网站：  
<https://rbtree.phpisfuture.com/>

## 6\. Spliterator

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0f3d5b49acf5c9368fd8fcd133050dc.png)  
其实吧，我开始阅读源码的时候，也在疑惑，我们有了迭代器，那么还要什么分割迭代器呢？  
这个分割迭代器在数组中也存在。  
深入的看到哪里用了分割迭代器，才开始理解，分割迭代器的意义所在了。

### 6.1 Spliterator调用链

我们以KeySpliterator为例  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/27a5541989f63b2b0c5a4f009f622229.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/29e5f557d26444dad83399741ff6dce7.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44eb1c15c290ab706809c73442d6a1df.png)  
在流的操作的时候，我们可能对元素比较多的HashMap或者数据结构，进行多线程处理。但是多线程处理存在并发问题，此时，就需要对数据结构中的元素进行分割，类似于数据分片，一个线程处理一片数据，这样避免并发问题。  
这就是分割迭代器存在的意义。

### 6.2 Spliterator属性

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e28db2c75715433018048ce692459866.png)

  * map:迭代对象
  * current:当前节点
  * index:该分割迭代器的开始哈希槽索引
  * fence:该分割迭代器的结束哈希槽索引
  * est:迭代节点的个数
  * expectedModCount:开始迭代的版本号

### 6.3 Spliterator 操作

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6fc6f96cefee62f4c6f54773947f085e.png)  
getFence初始化分割迭代器  
estimateSize初始化并获取分割迭代器的大小

### 6.4 KeySpliterator,ValueSpliterator,EntrySpliterator

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c61f222d321ad4bc4f9eca0a6e5d0d59.png)  
trySplit尝试分割。  
forEachRemaining是forEach的函数操作。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/518470b00aba9098425e0b13be2e1aa3.png)  
tryAdvance和forEachmaining最大的区别是：**tryAdvance只会执行一次或者0次，而forEachMaining会对数据结构中全部的元素都执行。**  
characteristics暂时还不知道是干什么的。

## 7\. HashMap 其他的常用的操作

### 7.1 初始化

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0eb64251356ef912297735e45a6c5432.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7461c3f4cdb468d2265bc03deebbe10d.png)  
默认的扩容因子是0.75，即，75%扩容。  
HashMap在new的时候不会立即分配内存。  
这里需要注意，**扩容扩的是哈希槽数组的容量，不是扩容整个HashMap的容量。**  
因为HashMap内部使用链表或者红黑树进行存储数据，不存在容量限制。  
只有哈希槽是使用数组存储，有扩容的问题。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/68cb1b53b3fc334e6a8f0bcf8ccc15a1.png)  
如果在初始化的时候指定了大小，那么会设置大小，扩容因子以及扩容阈值。扩容的阈值是指定大小+1.  
当然，指定大小加1是一般情况下。  
实际计算比这复杂：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bde931b793810932641207e12f1b42e3.png)

### 7.2 put

在7.1中知道，new 了HashMap并不会立即分配内存，而是第一次put操作的时候，会分配内存。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e748cf8c2075e704513517a3c7e1500.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/11c7a599ffd9cf857512aae15be764ca.png)  
HashMap通过key的hashCode计算哈希槽，使用的hashCode是获取key的Object的hashCode,即内存地址相关的hashCode，然后将key的hashCode的高位和低位进行异或运算。异或运算后，hashCode在一定程度上具有了内存相关的hashCode的高位和地位特征。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dab1172064311905a48eecbf675f9c7c.png)  
如果哈希槽还未分配内存，那么会调用resize进行分配内存。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/140b04aec62617c5968220ec240dd381.png)  
使用无参构造，此时旧的大小和阈值都是0.  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/552e18ad452ebccc0c7e3a9d4b65d126.png)  
默认大小是16，默认扩容阈值是16*0.75=12  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ac0e2cacd5cb7249b3dbee4867a885b.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/64d5e527e844e3ae1c545591670262da.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/04a8078df2ad1539950c621c7abd3074.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6e228c90abf6857e38e9f1c877129de4.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/63b380fda22f5fb79f3cb0ab2728b673.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1607e22e5fd6b4ed92c1b1d604f54861.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c6721eac2de03887b7805af47fb13130.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/35a3c311ddc1463377cb90b0d7cb07ea.png)  
所以说，当单个哈希槽下的节点个数大于等于8的时候，就会进行树化，从链表转为红黑树。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/45006c0ddf46f98127532e4c9b1770f8.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fdafb20ff95b66de97ab6298976a195d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6cb5ef7db0cf3a1b50377eeb045ccc79.png)  
++modCount版本号增加  
resize当数量大于哈希槽库容阈值，进行扩容。  
**HashMap的key和value可空。**这和数组不同。

### 7.3 链表化

我们从7.2知道了，当哈希槽节点数量大于等于8的时候，会进行树化，如果红黑树的节点数量小于等于6的时候，会进行链表化。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/47f40ca3bb2b9ae924855328fcab67b7.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6c8346f14065b56e3212985b15e2a7fb.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/07107b42163440791614de3cc29d2b1f.png)

### 7.4 hashCode

HashMap的hashCode是将所有的节点的hashCode累加后的hashCode  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3de0b9276254ef963ebdc0c7da3ed0c2.png)

## 8\. 总结

阅读HashMap的源码，对于我来说，是一个比较大的挑战。阅读源码是一件非常枯燥，非常费神的工作。但是阅读源码也是一件有意义的工作，阅读源码，对HashMap的理解更加深刻，对以后写出高质量，搞密度的代码，打下了基础。  
在阅读源码的过程中，有一些困难，比如理解HashMap内部类的继承关系，其实现的接口的操作以及HashMap的存储结构。这些比较难以理解，不过多阅读几遍源码，基本上连看代码，带猜测，都能理解其目的。  
最大的困难是理解红黑树。不过，说实话，我看了许多其他大神写的红黑树的讲解博客，看动态演示，依然无法直接从HashMap的源码理解其每一行的含义。  
以后还需要继续努力。加油！
