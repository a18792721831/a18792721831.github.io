---
layout: post
title: "Java基础--为什么ArrayList,Vector等都不支持循环中remove?"
date: 2020-04-19 20:52:17 +0800
categories: [ArrayList深度解析, ArrayList源码分析, List为什么用迭代器, 不使用迭代器删除, JDK8流删除操作]
description: "本文深入探讨了Java中Vector的多种遍历方法，包括for循环、迭代器、流操作及任意方向遍历，分析了在遍历过程中进行元素删除的正确实践，以及Vector与ArrayList在操作上的异同。"
keywords: ArrayList深度解析, ArrayList源码分析, List为什么用迭代器, 不使用迭代器删除, JDK8流删除操作
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/105617016
> - 发布时间：2020-04-19 20:52:17
> - 阅读量：2
> - 分类：java核心专栏收录该内容, 订阅专栏
> - 标签：#ArrayList深度解析, #ArrayList源码分析, #List为什么用迭代器, #不使用迭代器删除, #JDK8流删除操作

## 摘要

文章浏览阅读2.4k次，点赞4次，收藏25次。本文深入探讨了Java中Vector的多种遍历方法，包括for循环、迭代器、流操作及任意方向遍历，分析了在遍历过程中进行元素删除的正确实践，以及Vector与ArrayList在操作上的异同。

---

#### 为什么ArrayList,Vector等都不支持循环中remove

  * [1 Vector 直接删除](<#1_Vector__6>)
  * [2 Vector 遍历元素](<#2_Vector__13>)
  *     * [2.1 for循环遍历](<#21_for_17>)
    * [2.2 迭代器循环](<#22__58>)
    * [2.3 任意方向遍历](<#23__88>)
    * [2.4 Vector的foreach](<#24_Vectorforeach_122>)
  * [3\. Vector迭代器删除](<#3_Vector_131>)
  * [4\. Vector不使用迭代器删除元素](<#4_Vector_152>)
  * [5\. Vector流删除元素](<#5_Vector_177>)

  
JDK中有很多的数据结构，可以让我们操作数据。   
操作数据一般都是增删改查，排序等操作。   
其实，在Vector,ArrayList,LinkedList中，删除有两种方式进行删除：   
1.循环中删除   
2.直接删除 

## 1 Vector 直接删除

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d479b52a1e9c020b616ddb5d7b3a0398.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/904385b6ebdff2047efdefc267831485.png)  
直接删除首先调用indexOf方法，得到目标元素的第一个序列，然后调用删除指定序列元素的方法进行删除。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9079ae8f6f8d4e63e4f2efe31267cad1.png)  
在删除指定序列元素的方法中，实际也是使用了System.arraycopy方法，将指定元素后面的所有元素，前移。  
这样就实现了指定序列元素删除的目的。

## 2 Vector 遍历元素

提到循环删除，就不得不提到Vector的遍历，毕竟循环删除的基础是循环。  
那么，Vector进行遍历，是如何循环呢？  
Vector有大概3种常见的遍历方式：

### 2.1 for循环遍历

for循环遍历有三种写法：普通for循环和增强for循环以及流的foreach循环。  
普通for循环：
    
    
            for(int i = 0;i < vector.size();i++){
                People people = vector.get(i);
                people.setAge(people.getAge() + 1);
            }
    

增强for循环:
    
    
    for (People people : vector) {
                people.setAge(people.getAge() + 1);
            }
    

在jdk8中，加入了lambda表达式，于是有了流的foreach循环
    
    
    vector.stream().forEach(x -> x.setAge(x.getAge() + 1));
    

对于普通for循环，涉及到的方法非常简单，我们给定序列值，要求数组返回指定序列的元素，因为Vector是Object的数组，所以，返回的都是引用，我们基于引用进行修改，实际上也就修改了其存储于对内存中对象的属性值。  
而对于基本类型的包装类以及字符串，因为其内部维护有常量表，通过get方法返回的可能是其具体的值，而不是引用，所以，使用普通for循环处理基本包装类以及字符串的Vector可能存在修改失败的问题。不过这个不在我们思考的范围内.

对于增强for循环，其实现比普通for循环要复杂。Vector在其类内有一个私有内部类：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/da8e6ee5a582993f5c50c306d6f15c84.png)  
它实现了Iterator接口，实现了这个接口中的方法，就能实现增强for循环以及迭代器循环。  
所以，严格意义来讲，增强for循环是使用迭代器实现的。  
流的for循环，是使用了jdk8的默认方法进行调度，然后调用的还是迭代器里面的真正执行的方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5b2ff0f86082ca6bba5fe1deda83e1a5.png)  
首先获取Collection中默认的stream方法，得到非并行的spliterator(分割器)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e1d3006451f1a799c407187b77d19e3.png)  
然后调用分割器的forearch方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0cfdee5fa2df6e9930b8306328b1d7d.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4520c4d5208a6ddd1672e4e48f1646b5.png)  
真正调用的是分割器的forEachRemaining方法。  
在Vector中，也有一个私有类，实现了分割器接口：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8830504644cea98c9b506f460c61c5dc.png)  
分割器的的方法中就有foreach调用的实现。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff6889b3439667c1f82b61d8c284788c.png)  
我们可以在里面打断点验证。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18fb2d273b463293208ec096e197062e.png)  
从其调用堆栈，我们可以和清楚的验证我们的分析是正确的。

### 2.2 迭代器循环

为什么我没有在上面讲增强for循环，是因为，增强for循环使用迭代器实现的，所以，将增强for循环放在这里更好一些。  
首先，我们既然用过Vector,ArrayList或者LinkedList中的任意一种，那么就肯定知道如何使用迭代器进行遍历集合数组了：
    
    
    Iterator<People> iterator = vector.iterator();
            while (iterator.hasNext()){
                People people = iterator.next();
                people.setAge(people.getAge() + 1);
            }
    

这基本上是固定模式的迭代器写法。  
所以，迭代器遍历的顺序就是：

  * 得到迭代器对象
  * 存在下一个对象
  * 获取并偏移

而增强for循环，也是使用迭代器实现的，只不过，将上述模板代码进行封装语法糖，显得更加简单易懂、易用。  
那么，增强for循环是怎么实现的呢？  
我们知道，迭代器肯定需要调用hasNext方法确定是否进行下一次遍历，以及使用next方法进行获取遍历元素以及偏移迭代器。  
所以，我们在Vector的迭代器实现的hasNext方法与next方法打上断点，查看其调用堆栈：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/65f866071973fe05f2888002eb77a91f.png)  
发现其toString方法也调用了hasNext  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3e337b9cdd2d6387d2051006be0c1a0c.png)  
那么，我们去掉toString方法的调用。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/acb5e452510c705f5c7a48fdc42628f3.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a582bc13a3d5ee539ef938f7ea8215c.png)  
与我们猜想的一样，在增强for循环中调用了hasNext方法确定是否可以进行下一次循环。  
如果hasNext返回true，那么调用next方法，获取到元素  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bda4daf4bc33ba155e394dbb4fac9f6d.png)  
他这个应该是语法糖封装，所以没有显示调用。

### 2.3 任意方向遍历

我们遍历Vector有两种方向，从前往后，从后往前，前面所有的实现都是从前往后。  
使用迭代器遍历，只能从前往后。  
使用普通for循环可以从后往前，而且可以实现任意连续数组遍历。  
比如10个元素，我需要遍历后面的7个，而且是从后往前。  
这样使用普通for循环也能实现。  
但是这样存在一个问题，遍历时，只能读取，增加，却不能删除。  
如果：  
我需要遍历后面7个元素，从后往前，删除值等于8的元素，怎么实现呢？  
使用迭代器，增强for循环，普通for循环都无法实现。  
使用JDK8的流操作倒是可以实现，但是其过程也是非常的繁琐，性能比较慢。  
所以，JDK提供了任意方向遍历的方法：
    
    
            ListIterator<People> listIterator = vector.listIterator(7);
            while (listIterator.hasPrevious()){
                People people = listIterator.previous();
                people.setAge(people.getAge() + 2);
                if(people.getAge() == 7){
                    listIterator.remove();
                }
            }
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3bdd8ce5cb14c0834a9e15c80d238c70.png)  
我们可以看到，它遍历了前面7个元素，而且将操作后值等于7的元素进行删除。  
而且是从后往前进行遍历的。  
其从前往后遍历：
    
    
    ListIterator<People> listIterator1 = vector.listIterator();
            while (listIterator1.hasNext()){
                People people = listIterator1.next();
                people.setAge(people.getAge() + 3);
            }
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a14e8841ca1e750a380f6c69f7789f9e.png)

### 2.4 Vector的foreach

当然，如果你仅仅想遍历元素，那么Vector也提供了foreachar方法  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4fb5ea91b80c57ba5206793ad6d08996.png)
    
    
    vector.forEach(x -> x.setAge(x.getAge() + 5));
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/65fe8cde86dae2e88a05714e15854176.png)  
所以，总体来说，想要遍历元素，并进行修改，选择还是很多的。  
但是如果你要涉及到元素数量的改变，那么，能使用迭代器或者说流操作，还是尽可能使用这些安全的操作，避免出现ConcurrentModificationException。

## 3\. Vector迭代器删除

我们前面讲了，迭代器遍历，使用到了next方法获取元素以及偏移。  
但是在next方法中会进行一个检测：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e99a9e06019d237188abd334097fc62.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/46ed0543212c62ae5635c33ad13b3273.png)  
这个方法会判断modCount和expectedModCount是否一致，只有一致的条件下，集合数组才会进行循环，否则因其fast-fail机制，会通过抛出异常，进行快速失败。  
expectedModCount是在创建迭代器对象时进行初始化的，值等于modCount  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d6ddfce80811c0a20f7e4d493f53f20c.png)  
而我们的add，set，remove等方法，都会修改modCount的值。  
请注意，Vector自己的方法时不会进行expectedModCount的修改，只有迭代器才会维护这个expectedModCount的值。  
那么，在循环中，进行add，可以吗？  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/11bd36f463a0f9e2db48396421f95e48.png)  
答案也是不行的，在循环遍历中，无法进行造成数组元素数量变化的操作，迭代器提供的删除方法除外。  
为什么迭代器提供的删除方法可以实现删除呢？  
其核心原因是，迭代器没有提供add方法，所以Vector进行add只能调用自己实现的add方法，而自己实现的add方法又不会去维护expectedModCount的值。  
在循环中每次都会调用next方法进行获取本次遍历的元素，以及偏移到下一次遍历的元素的位置，但是在next方法中会调用check方法，如果modCount与expectedModCount不相等，就会进行快速失败。这就是为什么在循环时，不能进行增加的原因，删除也只能调用迭代器实现的删除方法。  
因为expectedModCount就是迭代器自己维护的变量。为了保证迭代器自己的删除操作成功，且能够进行下一次循环，每次删除都会强制将expectedModCount的值设置为modCount的值：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/748c4905eb69d53c8f7992d916ec1b76.png)  
而且迭代器调用的是Vector自己实现的remove方法：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/132c8b5bf721e76fff721a5463773955.png)  
这样也维护了数组有效长度的可靠。

## 4\. Vector不使用迭代器删除元素

看了这些，那么，我使用普通for循环能不能实现删除与增加呢？  
毕竟普通for循环没有check方法：
    
    
            for(int i = 0;i < vector.size();i++){
                People people = vector.get(i);
                if (people.getAge() == 20){
                    vector.add(new People(30));
                }
            }
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9a84e0edea306bef7df39a52d05d5872.png)  
答案是可以的。  
那么删除呢？
    
    
            for(int i = 0;i < vector.size();i++){
                People people = vector.get(i);
                if (people.getAge() == 20 || people.getAge() == 30){
                    vector.remove(people);
                    i--;
                }
            }
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16ed36dcd1ffa482353735207244a2df.png)  
答案是可以的，但是请注意，在删除掉元素后，需要将我们的序列值缩小。

## 5\. Vector流删除元素

除了使用4中删除，我们还可以使用JDK8中的流操作，删除元素。
    
    
            vector = vector.stream().filter(people -> people.getAge() != 21).
                    collect(Vector::new,Vector::add,(left,right)->left.addAll(right));
    

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/87d01921208fc754a9a4f35e58a5ce46.png)  
但是，通过这种流操作，涉及到重新构建，收集的问题，在不考虑多线程流操作的情况下，性能应该是比4中的方法的性能要差。

ArrayList与Vector大同小异。  
而LinkedList其实现是双链表实现，在元素数量的操作上，优于数组的。但是随机访问性能较差。