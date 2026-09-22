---
layout: post
title: "Java基础--多线程-ConcurrentHashMap(JDK1.8)--转载"
date: 2020-07-10 19:05:54 +0800
categories: [ConcurrtHashMap, 线程安全Map解析, ConcurrtHhMap源码, jdk7，8中线程安全区别, ConcurrHhMap的优化]
description: "本文详细解析了ConcurrentHashMap在JDK1.7与JDK1.8中的实现原理，包括其数据结构、线程安全机制、put、get、size等核心方法的工作流程，以及在多线程环境下的高效性能。"
keywords: ConcurrtHashMap, 线程安全Map解析, ConcurrtHhMap源码, jdk7，8中线程安全区别, ConcurrHhMap的优化
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/107236797
> - 发布时间：2020-07-10 19:05:54
> - 阅读量：421
> - 分类：java同时被 3 个专栏收录, 订阅专栏, java核心, 多线程
> - 标签：#ConcurrtHashMap, #线程安全Map解析, #ConcurrtHhMap源码, #jdk7，8中线程安全区别, #ConcurrHhMap的优化

## 摘要

文章浏览阅读421次。本文详细解析了ConcurrentHashMap在JDK1.7与JDK1.8中的实现原理，包括其数据结构、线程安全机制、put、get、size等核心方法的工作流程，以及在多线程环境下的高效性能。

---

#### Java基础--多线程-ConcurrentHashMap

  * 前言
  * ConcurrentHashMap（JDK1.7）
  * put
  * get
  * size
  * ConcurrentHashMap（JDK1.8）
  * 类图
  * get
  * put
  * initTable初始化
  * transfer
  * treeifyBin
  * addCount
  * size

## 前言

HashMap非线程安全的，HashTable是线程安全的，所有涉及到多线程操作的都加上了synchronized关键字来锁住整个table，这就意味着所有的线程都在竞争一把锁，在多线程的环境下，它是安全的，但是无疑效率低下的。

## ConcurrentHashMap（JDK1.7）

在JDK1.7中，ConcurrentHashMap的数据结构是由一个Segment数组和多个HashEntry组成的，如图：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/42f29c49d25746faa9260569537e8383.png)  
Segment数组的意义就是将一个大的table分割成多个小的table来进行加锁，也就是锁分离技术，而每一个Segment元素存储的是HashEntry数组+ 链表。分段是一开始就确定的，后期不能再进行扩容（即并发度不能改变），但是单个Segment里面的数组是可以扩容的。

而JDK1.8中，是bin扩容（并发度可变）。

## put

对于ConcurrentHashMap的数据插入，这里要进行两次Hash去定位数据的存储位置。
    
    
    static class Segment<K,V> extends ReentrantLock implements Serializable {
    }
    

从上Segment的继承体系可以看出，Segment实现了ReentrantLock,也就带有锁的功能，当执行put操作时，会进行第一次key的hash来定位Segment的位置，如果该Segment还没有初始化，即通过CAS操作进行赋值，然后进行第二次hash操作，找到相应的HashEntry的位置，这里会利用继承过来的锁的特性，在将数据插入指定的HashEntry位置时（链表的尾端），会通过继承ReentrantLock的tryLock（）方法尝试去获取锁，如果获取成功就直接插入相应的位置，如果已经有线程获取该Segment的锁，那当前线程会以自旋的方式去继续的调用tryLock（）方法去获取锁，超过指定次数就挂起，等待唤醒。

## get

ConcurrentHashMap的get操作跟HashMap类似，只是ConcurrentHashMap第一次需要经过一次hash定位到Segment的位置，然后再hash定位到指定的HashEntry，遍历该HashEntry下的链表进行对比，成功就返回，不成功就返回null。

## size

计算ConcurrentHashMap的元素大小是一个有趣的问题，因为他是并发操作的，就是在你计算size的时候，他还在并发的插入数据，可能会导致你计算出来的size和你实际的size有相差（在你return size的时候，插入了多个数据），要解决这个问题，JDK1.7版本用两种方案:
    
    
    try {
        for (;;) {
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j) ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) { sum += seg.modCount; int c = seg.count; if (c < 0 || (size += c) < 0)
                   overflow = true;
                } }
            if (sum == last) break;
            last = sum; } }
    finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    

  *     1. 第一种方案他会使用不加锁的模式去尝试多次计算ConcurrentHashMap的size，最多三次，比较前后两次计算的结果，结果一致就认为当前没有元素加入，计算的结果是准确的
  *     2. 第二种方案是如果第一种方案不符合，他就会给每个Segment加上锁，然后计算ConcurrentHashMap的size返回

## ConcurrentHashMap（JDK1.8）

JDK1.8的实现已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用Synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap，虽然在JDK1.8中还能看到Segment的数据结构，但是已经简化了属性，只是为了兼容旧版本；loadFactor仅用于构造函数中设定初始容量，已经不能影响扩容阈值，JDK1.8中阈值计算基本恒定为0.75；concurrencyLevel只影响初始容量，后续的并发度大小依赖于table数组的大小。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/6abd793e047ddb6c3c72ed31c0f2f1e9.png)  
先看一些常量设计和数据结构：
    
    
    // node数组最大容量：2^30=1073741824
    private static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认初始值，必须是2的幕数
    private static final int DEFAULT_CAPACITY = 16;
    //数组可能最大值，需要与toArray（）相关方法关联
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    //并发级别，遗留下来的，为兼容以前的版本
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
    // 负载因子
    private static final float LOAD_FACTOR = 0.75f;
    // 链表转红黑树阀值,> 8 链表转换为红黑树
    static final int TREEIFY_THRESHOLD = 8;
    //树转链表阀值，小于等于6（tranfer时，lc、hc=0两个计数器分别++记录原bin、新binTreeNode数量，<=UNTREEIFY_THRESHOLD 则untreeify(lo)）
    static final int UNTREEIFY_THRESHOLD = 6;
    static final int MIN_TREEIFY_CAPACITY = 64;
    private static final int MIN_TRANSFER_STRIDE = 16;
    private static int RESIZE_STAMP_BITS = 16;
    // 2^15-1，help resize的最大线程数
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
    // 32-16=16，sizeCtl中记录size大小的偏移量
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
    // forwarding nodes的hash值
    static final int MOVED     = -1;
    // 树根节点的hash值
    static final int TREEBIN   = -2;
    // ReservationNode的hash值
    static final int RESERVED  = -3;
    // 可用处理器数量
    static final int NCPU = Runtime.getRuntime().availableProcessors();
    //存放node的数组
    transient volatile Node<K,V>[] table;
    /*控制标识符，用来控制table的初始化和扩容的操作，不同的值有不同的含义
     *当为负数时：-1代表正在初始化，-N代表有N-1个线程正在 进行扩容
     *当为0时（默认值）：代表当时的table还没有被初始化
     *当为正数时：表示初始化或者下一次进行扩容的大小<br>*/
    private transient volatile int sizeCtl;　　
    

## 类图

![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/3c225c4077d859f5a6ee654791a3f462.png)

  * **Node** 是ConcurrentHashMap存储结构的基本单元，实现了Map.Entry接口，用于存储数据。它对value和next属性设置了volatile同步锁(与JDK7的Segment相同)，它不允许调用setValue方法直接改变Node的value域，它增加了find方法辅助map.get()方法。

  * **TreeNode** 继承于Node，但是数据结构换成了二叉树结构，它是红黑树的数据的存储结构，用于红黑树中存储数据，当链表的节点数大于8时会转换成红黑树的结构，他就是通过TreeNode作为存储结构代替Node来转换成黑红树。

  * **TreeBin** 从字面含义中可以理解为存储树形结构的容器，而树形结构就是指TreeNode，所以TreeBin就是封装TreeNode的容器，它提供转换黑红树的一些条件和锁的控制。

  * **ForwardingNode** 一个用于连接两个table的节点类。它包含一个nextTable指针，用于指向下一张表。而且这个节点的key value next指针全部为null，它的hash值为-1. 这里面定义的find的方法是从nextTable里进行查询节点，而不是以自身为头节点进行查找。

**Unsafe和CAS**

在ConcurrentHashMap中，随处可以看到U, 大量使用了U.compareAndSwapXXX的方法，这个方法是利用一个CAS算法实现无锁化的修改值的操作，他可以大大降低锁代理的性能消耗。这个算法的基本思想就是不断地去比较当前内存中的变量值与你指定的一个变量值是否相等，如果相等，则接受你指定的修改的值，否则拒绝你的操作。因为当前线程中的值已经不是最新的值，你的修改很可能会覆盖掉其他线程修改的结果。这一点与乐观锁，SVN的思想是比较类似的。
    
    
    private static final sun.misc.Unsafe U;
    private static final long SIZECTL;
    private static final long TRANSFERINDEX;
    private static final long BASECOUNT;
    private static final long CELLSBUSY;
    private static final long CELLVALUE;
    private static final long ABASE;
    private static final int ASHIFT;
     
    static {
        try {
            U = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentHashMap.class;
            SIZECTL = U.objectFieldOffset
                (k.getDeclaredField("sizeCtl"));
            TRANSFERINDEX = U.objectFieldOffset
                (k.getDeclaredField("transferIndex"));
            BASECOUNT = U.objectFieldOffset
                (k.getDeclaredField("baseCount"));
            CELLSBUSY = U.objectFieldOffset
                (k.getDeclaredField("cellsBusy"));
            Class<?> ck = CounterCell.class;
            CELLVALUE = U.objectFieldOffset
                (ck.getDeclaredField("value"));
            Class<?> ak = Node[].class;
            ABASE = U.arrayBaseOffset(ak);
            int scale = U.arrayIndexScale(ak);
            if ((scale & (scale - 1)) != 0)
                throw new Error("data type scale not a power of two");
            ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
        } catch (Exception e) {
            throw new Error(e);
        }
    }
    

ConcurrentHashMap定义了三个原子操作，用于对指定位置的节点进行操作。正是这些原子操作保证了ConcurrentHashMap的线程安全。
    
    
    // 获取tab数组的第i个node<br>    @SuppressWarnings("unchecked")
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }
    // 利用CAS算法设置i位置上的node节点。在CAS中，会比较内存中的值与你指定的这个值是否相等，如果相等才接受<br>    // 你的修改，否则拒绝修改，即这个操作有可能不成功。
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
    // 利用volatile方法设置第i个节点的值，这个操作一定是成功的。
    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }
    

## get

通过key获取value
    
    
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;<br>        // 计算hash值
        int h = spread(key.hashCode());<br>        // 如果tab不空并且bin里面的节点不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) { // 如果bin里面的头节点就是需要查询的value
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0) // eh < 0 只有可能是MOVED(-1)或TREEBIN(-2)
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) { // 链表
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
    

## put

首先看一下put的源码：根据hash值计算这个新插入的点在table中的位置i，如果i位置是空的，直接放进去，否则进行判断，如果i位置是树节点，按照树的方式插入新的节点，否则把i插入到链public V put(K key, V value) { return putVal(key, value, false);
    
    
    final V putVal(K key, V value, boolean onlyIfAbsent) {<br>        // key和value不允许null
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode()); // 计算hash值
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) { // 死循环，何时插入成功，才跳出
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable(); // table是在首次插入元素的时候初始化，lazy
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null, // 如果这个位置没有值，直接放进去，由CAS保证线程安全，不需要加锁
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED) // 参与扩容
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;<br>                // 节点上锁，这里的节点可以理解为hash值相同组成的链表的头节点，锁的粒度为头节点。
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) { // 普通Node的hash值为key的hash值大于零，而ForwardingNode的是-1，TreeBin是-2
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) { // 主要遍历链表到最后，然后增加新节点
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) { // 在链表最后插入新节点
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) { // 如果是数节点，就按照树的方式插入节点。
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)// 如果链表长度大于等于8，则试着把链表转树
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);// 数量 + 1；
        return null;
    }
    

JDK8中的实现也是锁分离思想，只是锁住的是一个node，而不是JDK7中的Segment；锁住Node之前的操作是基于在volatile和CAS之上无锁并且线程安全的。 

put操作的流程图如下：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/14b2fd5adafd1fc4fb2bf76c6b1b6a5a.png)  
从put可以看出有几个操作比较重要，下面我们就重点讲解这几个方法：initTable，helpTransfer，treeifyBin，addCount

## initTable初始化

初始化方法主要应用了关键属性sizeCtl 如果这个值小于0，表示其他线程正在进行初始化，就放弃这个操作。在这也可以看出ConcurrentHashMap的初始化只能由一个线程完成。如果获得了初始化权限，就用CAS方法将sizeCtl置为-1，防止其他线程进入。初始化数组后，将sizeCtl的值改为0.75*n。
    
    
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)// sizeCtl < 0 标示有其他线程正在进行初始化操作，把线程让出cpu，对于table的厨师操作，只能有一个线程在进行
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) { // 利用CAS把sizeCtl设置为-1，标示本线程正在进行初始化，同一个时刻只有一个线程能更新成功，失败的重新循环，发现sizeCtl已经 < 0
                try {<br>                    // 为什么还要判断，因为：如果走到下面的finally改变了sizeCtl值，有可能其他线程是会进入这个逻辑的
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY; // 默认大小是16
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2); // 0.75*n，下一次扩容阈值
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
    

## transfer

当ConcurrentHashMap容量不足的时候，需要对table进行扩容，它支持并发扩容，却没有锁。  
扩容的场景：  
（1）往hashMap中成功插入一个key/value节点时，有可能触发扩容动作：所在链表的元素个数达到了阈值 8，则会调用treeifyBin方法把链表转换成红黑树，不过在结构转换之前，会对数组长度进行判断，如果小于64，则优先扩容，而不是链表转树。  
（2）新增节点之后，会调用addCount方法记录元素个数，并检查是否需要进行扩容，当数组元素个数达到阈值时，会触发transfer方法，重新调整节点的位置。

整个扩容操作分为两个部分

  * 第一部分是构建一个nextTable,它的容量是原来的两倍，这个操作是单线程完成的。这个单线程的保证是通过RESIZE_STAMP_SHIFT这个常量经过一次运算来保证的，这个地方在后面会有提到；

  * 第二个部分就是将原来table中的元素复制到nextTable中，这里允许多线程进行操作。

    
    
    // 扩容操作
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;<br>        // 每核处理的量小于16，则强制赋值16
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        if (nextTab == null) { // 构造一个2倍的Node数组
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        int nextn = nextTab.length;<br>        // 初始化fwd节点，其中保存了新数组nextTable的引用，在处理完每个bin的节点之后<br>        // 当做占位节点，表示该bin已经被处理过了
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab<br>        // 通过for循环处理每个bin中的链表元素<br>        // i:表示当前处理的bin序号，bound：表示需要处理的bin边界，处理是从数组的最后一个bin开始
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;<br>            // 这个while其实是确定下一个需要处理的bin
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) { // 整个扩容结束的标志
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null) // 如果bin为空，则通过CAS设置fwd，表示已经处理过了
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED) // 如果已经处理过了则设置fwd节点，其hash值为MOVED(-1)
                advance = true; // already processed
            else {
                synchronized (f) { // 对bin的头节点（链表的头节点或数的根节点）加锁
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) { // 链表的操作：构建2个链表，一个是原链表，另一个是原链表的反序排列
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln); // 在nextTable的i位置插入链表
                            setTabAt(nextTab, i + n, hn); // 在nextTable的i+n位置插入链表
                            setTabAt(tab, i, fwd); // 在tab的i位置插入fwd节点，标示处理过了
                            advance = true;
                        }
                        else if (f instanceof TreeBin) { // 树的操作
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }<br>                            // 如果树<=6，则树转数组
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
    

状态变化图：  
（1）初始化有一个16大小的数组：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/a4efea7da6ca0dd0200abce0bd111867.png)  
（2）创建一个二倍大小的nextTable，并且new ForwardingNode<K,V>(nextTab)  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/804383e9bf96cf22c0b0d1aea9011517.png)  
（3）从后往前移动tab中元素到nextTable，比如：已经把tab[10-15]移动到nextTable中的状态图为：  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c422233b4441c1a9e275dbc185eaa915.png)

## treeifyBin

在put操作中，如果发现链表结构中的元素超过8个，则会把链表转换为红黑树，便于提高查询效率。
    
    
    private final void treeifyBin(Node<K,V>[] tab, int index) {
          Node<K,V> b; int n, sc;
          if (tab != null) {<br>            // 如果tab长度小于64，则优先扩容(2倍扩展)，而不是链表转树。
              if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                  tryPresize(n << 1);
              else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                  synchronized (b) { // 对bin的头节点加锁，保证整个红黑树的建立是同步的
                      if (tabAt(tab, index) == b) {
                          TreeNode<K,V> hd = null, tl = null; // 创建TreeNode节点
                          for (Node<K,V> e = b; e != null; e = e.next) {
                              TreeNode<K,V> p =
                                  new TreeNode<K,V>(e.hash, e.key, e.val,
                                                    null, null);
                              if ((p.prev = tl) == null)
                                  hd = p;
                              else
                                  tl.next = p;
                              tl = p;
                          }
                          setTabAt(tab, index, new TreeBin<K,V>(hd)); // 最终设置bin头结点的值为TreeBin
                      }
                  }
              }
          }
      }
    

## addCount

把当前ConcurrentHashMap元素个数 + 1，主要有2个步骤：（1）更新baseCount值（2）检测是否进行扩容
    
    
    private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;        // CAS更新baseCount值
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                fullAddCount(x, uncontended);
                return;
            }
            if (check <= 1)
                return;
            s = sumCount();
        }
        if (check >= 0) { // 检测是否需要扩容
            Node<K,V>[] tab, nt; int n, sc;
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
                if (sc < 0) {
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
    

## size

最后，我们看看size方法  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/220fd379241d867dd5ff0007f4d0a7ff.png)  
调用sumCount获取数量。  
![在这里插入图片描述](https://picgo-1302191088.cos.ap-guangzhou.myqcloud.com/csdn/csdnimg/c8996efc9213f8f4be33fe7990a67231.png)
