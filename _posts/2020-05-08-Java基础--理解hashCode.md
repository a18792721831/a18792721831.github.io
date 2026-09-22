---
layout: post
title: "Java基础--理解hashCode"
date: 2020-05-08 19:57:56 +0800
categories: [java核心, hashCode, 哈希, 对象比较, Java的哈希]
description: "本文深入探讨了Java中hashCode和equals方法的原理及应用，详细解释了两者的关系，以及如何在自定义类中正确实现它们，以提升哈希表性能。"
keywords: java核心, hashCode, 哈希, 对象比较, Java的哈希
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/106004311
> - 发布时间：2020-05-08 19:57:56
> - 阅读量：1
> - 分类：java同时被 2 个专栏收录, 订阅专栏, java核心
> - 标签：#java核心, #hashCode, #哈希, #对象比较, #Java的哈希

## 摘要

文章浏览阅读1.6k次。本文深入探讨了Java中hashCode和equals方法的原理及应用，详细解释了两者的关系，以及如何在自定义类中正确实现它们，以提升哈希表性能。

---

#### Java基础--理解hashCode

  * 1.hashCode
  * 2.equals
  * 3\. equals & hashCode
  * 4.例子
  * 5\. 总结

## 1.hashCode

什么是hashCode?  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/42aa6cf993a4a0317a477b259cf71026.png)  
根据jdkapi文档，可以很明确的得知hashCode的特点：

  *     1. 在一次运行期间，同一对象的equal比较信息没有被修改hashCode一定相同
  *     2. 如果对象的equals方法相同，那么hashCode一定相同
  *     3. 可以存在equals不等，但是hashCode相等的数据

接下来看看hashCode方法的定义：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/d9bdc2af8aa5f0840248b1fcfdf28c02.png)  
因为Object是Java所有类的父类，所以，Java所有类的默认的hashCode的方法就是Object的hashCode方法。Object的hashCode方法是一个native方法，即用平台相关语言实现的，由Java调用的方法。Object的hashCode方法是返回其内存地址经过计算后的数据。  
hashCode返回的是一个int型的值。  
内存地址计算过程（网上找到的资料，不保证一定正确）：  
int型的数据长度是4个字节即32位，一个内存地址是64位。  
得到内存地址后，将地址的高32位于低32位进行异或，得到的32位数值就是hashCode

## 2.equals

equals方法是Java中判断一个对象是否相等的重要的方法。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/06251cd395ce8aed92ff32d2bfeded76.png)  
equals方法需要满足一下特性：

  * 自反性：非空X，X对自己的equals一定是true
  * 对称性：非空X,Y；如果X.equals(Y)是true，那么，Y.equaks(X)一定是true
  * 传递性：非空X,Y,Z:如果X.equals(Y),Y.equals(Z)，那么X.equals(Z)
  * 一致性：非空X,Y:如果X.equals(Y)，那么在一次运行中，任何时候，任何情况下都满足(equals的属性未修改)
  * null：非空X：X.equals(null)一定false

持此之外，equals方法和`==`通常意义不同。  
equals可以自定义比较的属性  
而`==`如果是基本类型，就是比较其值  
如果`==`不是基本类型，而是引用类型，那么比较的就是引用地址了。

## 3\. equals & hashCode

一般情况下，普遍的通识中，会默认保证：  
如果equals是true，那么hashCode相等。  
这一原则。

为什么？

1.hashCode的设计目的是为了提高哈希表（例如 java.util.Hashtable 提供的哈希表）的性能。  
哈希表存储数据会将hashCode相同的数据放到同一个哈希桶里。  
所以，如果hashCode不相等，但是equals却相等，在使用哈希表等数据结构时，就无法提升其性能了。

2.不仅仅是性能问题，hashCode在许多的数据结构中都有涉及，所以，尽可能保证这一原则非常重要。

## 4.例子
    
    
    import java.util.HashSet;
    import java.util.Set;
    
    /**
     * @author jiayq
     * @Date 2020-04-21
     */
    public class HashMain {
    
        private int myHash;
    
        public HashMain(){
            this.myHash = hashCode();
        }
    
        public static void main(String[] args){
            HashMain hashMain = new HashMain();
            //输出hashCode（hashCode一般以16进制进行展示或者运算）
            System.out.println(Integer.toHexString(hashMain.hashCode()));
            //调用Object的toString方法
            System.out.println(hashMain);
            System.out.println(27 & 0b1110);
            // 27 = 0b 0001 1011 & 0b 0000 1110 => 0b 0000 1010
            System.out.println(27 ^ 0xE);
            // 27 = 0b 0001 1011 ^ 14 => 0b 0000 1110 => 0001 0101 = 21
            HashMain hashMain1 = hashMain;
            //比较引用
            System.out.println(hashMain == hashMain1);
            //比较equals
            System.out.println(hashMain.equals(hashMain1));
            //设置属性
            hashMain.myHash = 1;
            //创建不同的对象
            hashMain1 = new HashMain();
            //设置属性
            hashMain1.myHash = 1;
            //比较equals
            System.out.println(hashMain.equals(hashMain1));
            //输出hashCode
            System.out.println(hashMain.hashCode());
            System.out.println(hashMain1.hashCode());
            //调用Object的toString输出
            System.out.println(hashMain);
            System.out.println(hashMain1);
            //Set不允许重复数据，其重复的标准是equals方法
            //如果hashCode相同，那么会将数据放到同一个哈希桶里
            //如果equals相同，那么不会进行存储，注意，在判断equals之前会判断hashCode相等
            //也就是说，如果两个对象的hashCode不同，其equals相同，那么Set也会将这两个对象当做不同的数据进行存储
            //反之，如果两个对象的hashCode相同，那么会继续判断equals
            //1.hashCode不同，equals相同
            Set<HashMain> set = new HashSet<>();
            set.add(hashMain);
            set.add(hashMain1);
            System.out.println(set);
            set.clear();
            //2.hashCode相同，equals不同
            hashMain1.myHash = 2;
            set.add(hashMain);
            set.add(hashMain1);
            System.out.println(set);
        }
    
    
        /**
         * 重写了子类的hashCode的方法
         * @return
         */
        @Override
        public int hashCode() {
            // 0b1111 = 1+2+4+8 = 15
            // value & 0b1111 -> [0,15]
            // value & (ob1111 - 1) -> [0,14] => 0b 1110 & value => 高 3 位
            //调用Object的hashCode方法，将返回的结果取低4位(0~15),类似将数值 >>> 60
            //同时 value % 16 等价于 value & 0b1111 <=> value &0xF
            //场景1.hashCode不同，equals相同
            return super.hashCode() & 0b1111;
            //2.hashCode相同，equals不同
    //        return this.getClass().getName().hashCode();
        }
    
    
        /**
         * 重写了子类的equals方法
         * @param obj
         * @return
         */
        @Override
        public boolean equals(Object obj) {
            //类型相同
            if (obj instanceof HashMain){
                HashMain main = (HashMain) obj;
                //属性相同
                return this.myHash == main.myHash;
            }
            return false;
        }
    
    }
    
    
    

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/ffef41edf3cab00cbc90a12f5cce3c8e.png)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/dc5e7a59b852d3faea176c87c4b4e68f.png)

## 5\. 总结

hashCode的目的是提升哈希表的性能。  
哈希表在存储的时候会先调用hashCode进行比较，然后使用equals进行比较  
为了更好的使用哈希表，需要保证equals相等，hashCode一定相等  
默认的hashCode是内存地址进行运算  
hashCode将对象的部分差异集中到了int数据中  
hashCode一般可以实现对象比较的先行比较
