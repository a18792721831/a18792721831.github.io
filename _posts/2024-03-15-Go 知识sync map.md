---
layout: post
title: Go-知识sync map
categories: [go]
description: Go-知识sync map
keywords: golang, go, sync map, sync map源码, 并发安全map
---

Go-知识sync map

> 一个小活动： https://developer.aliyun.com//topic/lingma/activities/202403?taskCode=14508&recordId=40dcecb786f9a65c2e83e95306822ce4#/?utm_content=m_fission_1  「通义灵码 · 体验 AI 编码，开 AI 盲盒」

map 是一个并发不安全的key-value存储映射工具，在Go支持高性能并发的语言中，如果多个 goroutine 之间传递map，
在并发的过程中非常容易触发读写冲突，导致程序panic。  
sync map 是并发安全的key-value映射。
# 1. 用法
## 1.1 声明
sync map不需要想map那样，使用make或者使用剪短变量声明赋值初始，可以直接使用,零值为空 sync map ，不是nil.
```Go
    var sm sync.Map
```
## 1.2 增删改查
增删改查比较简单：
```Go
func TestSyncMap(t *testing.T) {
	var sm sync.Map
	// 增加或修改
	sm.Store("hi", "hello")
	// 查询
	// 查询返回 value, bool,必须显示忽略 bool
	v, ok := sm.Load("hi")
	fmt.Printf("time : %s, v = %s, ok = %v", time.Now().Format(T_F), v, ok)
	// 删除
	sm.Delete("hi")
	wd := sync.WaitGroup{}
	wd.Add(2)
	fmt.Printf("time : %s , start\n", time.Now().Format(T_F))
	go func() {
		rand.Seed(time.Now().UnixNano())
		fmt.Printf("time : %s , sleep\n", time.Now().Format(T_F))
		time.Sleep(time.Duration(rand.Intn(10)) * time.Second)
		// 查询并增加，返回 oldV, bool , 如果 key 已经有值，返回 v 和 true ,否则返回 nil 和 false
		oldValue, isExists := sm.LoadOrStore("hi", "world")
		fmt.Printf("time : %s ,old := %s , is Exists : %v\n", time.Now().Format(T_F), oldValue, isExists)
		wd.Done()
	}()
	go func() {
		rand.Seed(time.Now().UnixNano())
		fmt.Printf("time : %s , sleep\n", time.Now().Format(T_F))
		time.Sleep(time.Duration(rand.Intn(5)) * time.Second)
		oldValue, isExists := sm.LoadOrStore("hi", "worldX")
		fmt.Printf("time : %s ,old := %s , is Exists : %v\n", time.Now().Format(T_F), oldValue, isExists)
		fmt.Printf("time : %s ,delete map\n", time.Now().Format(T_F))
		sm.Delete("hi")
		wd.Done()
	}()
	wd.Wait()
}
```
![img.png](/images/posts/2024-03-15-Go%20知识sync%20map/img.png)


与map不同的是，sync map 不能使用`[]`来指定key，因为map是标准库提供的，编译的时候会做链接。  
还需要注意一点，sync map 能存储任何类型的key-value，key不在限制为基本类型。  
但是这也意味着，如果无法保证value的类型，那么在使用的时候，需要使用类型断言。
## 1.3 增强操作
除了map中的简单的增删改查之外，还有一些结合了查询的操作。
- LoadOrStore
```Go
// LoadOrStore returns the existing value for the key if present.
// LoadOrStore 返回旧值，如果不存在，返回给定的值
// Otherwise, it stores and returns the given value.
// The loaded result is true if the value was loaded, false if stored.
// 如果有旧值，返回true，否则返回false
func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool) {}
```
- LoadAndDelete
```Go
// LoadAndDelete deletes the value for a key, returning the previous value if any.
// LoadAndDelete 删除给定的key，如果存在，返回值，如果不存在返回 nil
// The loaded result reports whether the key was present.
// 如果给定的key对应的只存在，返回true，否则返回false
func (m *Map) LoadAndDelete(key interface{}) (value interface{}, loaded bool) {}
```
- Range
```Go
func (m *Map) Range(f func(key, value interface{}) bool) {} 
```
sync map无法像map一样使用range进行遍历，所以提供了`Range`方法实现遍历能力。 `Range`会遍历每一个key-value，并对每一个key-value调用传入的函数，实现遍历。
> 因为sync map支持并发读写，遍历期间可能读取到其他goroutine写入的数据。也就是说，遍历过程中，map是动态变化的。

# 2. sync map 使用注意
sync map是用于解决并发情况下map的读写冲突问题的，sync map不是为了替代map，仅仅是map的一个优化实现。  
sync map提供并发读写的能力也是有代价的，引入了一些限制或者是风险。
## 2.1 多读少写
sync map内部实现采用了两个原生map实现读写分离，数据读取并且能命中才能提升读取的性能，否则因为要遍历两个map，性能不如原生map。  
由于sync map 使用了两个冗余的原生map，就会使用更多的内存存储数据，会对系统的内存有相对高的要求。  
当无法确定是否使用sync map的时候，可以采用benchmark进行性能测试。
## 2.2 类型安全风险
在sync map中，不管是key还是value都是使用interface{}类型存储的，不是像map一样，指定了key和value的类型。  
所以在使用的时候，需要做类型断言。(Go支持了泛型之后会有改善)  
比如：
```Go
func TestSyncMapType(t *testing.T) {
	sm := sync.Map{}
	sm.Store("hi", true)
	sm.Store(10, "haha")
	sm.Range(func(key, value interface{}) bool {
		fmt.Printf("key : %v , type : %s , value : %v , type : %s\n", key, reflect.TypeOf(key).String(), value, reflect.TypeOf(value).String())
		return true
	})
}
```
![img_1.png](/images/posts/2024-03-15-Go%20知识sync%20map/img_1.png)


虽然在存储的时候，任何类型都能成功的存储，但是也带来了读取的类型困扰，这意味这读取到的key-value的类型无法确定，必须先使用类型断言后才能使用。
## 2.3 不能拷贝和传递
因为sync map中使用sync.Mutex实现并发安全，锁是不能拷贝的，否则会导致死锁或者panic，所以在使用sync map的时候，优先使用指针，这样函数参数拷贝时，拷贝的是指针，而不是sync map本身。
> Go编译器无法识别这个风险，特别注意。

# 3. 实现原理
## 3.1 数据结构
在`sync/map.go`中定义了sync map:
```Go
type Map struct {
    // 锁
	mu Mutex
	// 读表，允许并发读
	read atomic.Value // readOnly
	// 写临时表
	dirty map[interface{}]*entry
	// 查找读表丢失次数
	misses int
}
```
sync map 由两个map表组成，read表提供并发读的能力，新数据则写入dirty表。read表的数据类型虽然是原子类型，但是存放的是map。  
dirty表是新数据的临时存放区，数据最终会同步到read表，同步的时机则取决于misses。读取数据时先查询read表，如果未找到，则misses++，在查询dirty表。  
等misses达到一定的数量时，触发数据同步。  
锁主要是保护dirty表，同时在数据同步时，也起到保护作用。
## 3.2 read表数据结构
在sync map中，read表是`atomic.Value`类型，实际的类型是`readOnly`结构体:
```Go
// readOnly is an immutable struct stored atomically in the Map.read field.
type readOnly struct {
	m       map[interface{}]*entry
	amended bool // true if the dirty map contains some key not in m.
	// 标记dirty表中是否存在数据未同步
}
```
`amended`用于指示在查询数据时，是否需要继续查询dirty表，如果`amended=false`，那么当read表中没有找到key，那么就不在查询dirty表，而是直接返回nil.  
这样就节省了一次加锁，遍历dirty表的时间。
## 3.3 entry 的数据结构
不管是dirty表还是read表，类型都是`entry`指针，entry的定义：
```Go
// An entry is a slot in the map corresponding to a particular key.
type entry struct {
	p unsafe.Pointer // *interface{}
}
```
entry是map中存放数据的槽位，使用entry的指针可以让read表和dirty表进行内存共享，在数据同步的时候，避免数据拷贝，只需要操作指针即可。
## 3.4 sync map 的结构图
![img_2.png](/images/posts/2024-03-15-Go%20知识sync%20map/img_2.png)


这是一个空的sync map的结构图。
## 3.5 插入数据
插入数据通过`Store`存储：
```Go
// 插入一个key-value
func (m *Map) Store(key, value interface{}) {
    // 从 atomic.Value中拿出read表，需要做类型转换
	read, _ := m.read.Load().(readOnly)
	// 如果 插入的数据已经存在，ok 为true ，那么检测是否标记为删除，如果标记删除，那么必须先写入dirty表，维护标记，
	// 如果没有被标记删除，那么尝试进行 cas 交换新值，如果cas成功，结束，否则加锁写dirty表
	if e, ok := read.m[key]; ok && e.tryStore(&value) {
		return
	}
	// 加锁
	m.mu.Lock()
	// 重新读取read表，防止之前拿到的read表被更新，但是当前goroutine没有重新读取，导致数据丢失
	read, _ = m.read.Load().(readOnly)
	// 如果 read表中存在数据，那么进行更新
	if e, ok := read.m[key]; ok {
	    // 检查 read表中 key 是否标记删除了，如果删除，那么必须写入dirty表
		if e.unexpungeLocked() {
		    // 使用cas进行写入 key
			m.dirty[key] = e
		}
		// 使用 cas 写入 value
		e.storeLocked(&value)
	// 如果 read 表中没有，那么写入dirty表，如果dirty表中已经存在，也就是还未做数据同步就又修改了
	} else if e, ok := m.dirty[key]; ok {
	    // 直接使用 cas 写入 value
		e.storeLocked(&value)
	} else {
	// dirty表中没有key
	    // amended=true表示未同步，false表示已经同步， 如果之前数据已经同步了，那么本次是写入dirty表的第一个数据，需要初始化dirty表
		if !read.amended {
			// 尝试初始化dirty表，如果dirty表不为空，什么也不做
			m.dirtyLocked()
			// 更新read表的 amended=true 表示有数据未同步，read表的atomic.Value不变
			// 主要修改 amended 标记
			m.read.Store(readOnly{m: read.m, amended: true})
		}
		// dirty写入value
		m.dirty[key] = newEntry(value)
	}
	// 解锁
	m.mu.Unlock()
}
```
插入数据后的结构图：  
![img_3.png](/images/posts/2024-03-15-Go%20知识sync%20map/img_3.png)


## 3.6 查找数据
查找数据是通过`Load`进行查找的：
```Go
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
    // 获取read表
	read, _ := m.read.Load().(readOnly)
	// 查询 read表
	e, ok := read.m[key]
	// 如果 read表没有找到，或者有数据未同步
	if !ok && read.amended {
	    // 加锁
		m.mu.Lock()
		// 重新加载 read 表
		read, _ = m.read.Load().(readOnly)
		// 查询read表
		e, ok = read.m[key]
		// read 表没有找到，或者有数据未同步
		if !ok && read.amended {
		    // 查询dirty表
			e, ok = m.dirty[key]
			// misses++,如果misses>= read表，那么用dirty表替换read表，并重置 misses值
			m.missLocked()
		}
		// 解锁
		m.mu.Unlock()
	}
	// 如果 read表未找到，而且数据全部都同步了，那么说明key不存在，返回nil
	if !ok {
		return nil, false
	}
	// 否则返回 key 对应的value，如果entry被标记删除了，返回nil
	return e.load()
}
```
对于上面的结构图，因为amended=true，那么就会在dirty表中查询，每次查询都会misses++,等遍历完了，或者misses等于dirty表size，那么使用dirty表替换read表。  
![img_4.png](/images/posts/2024-03-15-Go%20知识sync%20map/img_4.png)


如果使用dirty表替换了read表，dirty表会置空nil:
```Go
func (m *Map) missLocked() {
	m.misses++
	if m.misses < len(m.dirty) {
		return
	}
	// read 表使用dirty表替换
	m.read.Store(readOnly{m: m.dirty})
	// dirty表置空
	m.dirty = nil
	m.misses = 0
}
```
## 3.7 再次插入
因为在查询的时候，已经将dirty给了read表，那么在次插入的时候，如果在read表中找到了，那么使用cas修改read表。  
如果没有找到，那么将read表同步到dirty表，然后插入数据，amended=true.
还记的插入数据里面的这个代码吗：  
![img_5.png](/images/posts/2024-03-15-Go%20知识sync%20map/img_5.png)


```Go
func (m *Map) dirtyLocked() {
    // 如果dirty表不为空结束
	if m.dirty != nil {
		return
	}
	// 如果dirty表为空，那么拷贝read表的map到dirty,数据通过指针共享
	read, _ := m.read.Load().(readOnly)
	m.dirty = make(map[interface{}]*entry, len(read.m))
	for k, e := range read.m {
		if !e.tryExpungeLocked() {
			m.dirty[k] = e
		}
	}
}
```
所以再次插入后的结构图：  
![img_6.png](/images/posts/2024-03-15-Go%20知识sync%20map/img_6.png)


> 为什么dirty表需要冗余read表的数据，直接存储增量数据不行吗？

dirty表通过冗余read表中的数据从而维护一个全量数据，read表只是dirty表的一个临时副本。数据同步的时候，直接用dirty表替换read表，避免遍历。  
同时在删除数据的时候，只是标记删除，由dirty表进行执行，避免多个入口进行删除数据，从而导致数据不一致，在dirty表从read表拉取数据的时候，会忽略标记删除的数据。
## 3.8 删除数据
删除操作通过`Delete`实现：
```Go
func (m *Map) Delete(key interface{}) {
    // delete真正由LoadAndDelete实现
	m.LoadAndDelete(key)
}

func (m *Map) LoadAndDelete(key interface{}) (value interface{}, loaded bool) {
    // 获取read表
	read, _ := m.read.Load().(readOnly)
	// 从读表中查询
	e, ok := read.m[key]
	// 没有找到而且存在数据未同步
	if !ok && read.amended {
	    // 加锁
		m.mu.Lock()
		// 重新读取read表
		read, _ = m.read.Load().(readOnly)
		// 从read表中查找
		e, ok = read.m[key]
		// 没有找到而且从在数据未同步
		if !ok && read.amended {
		    // 从dirty表中查找
			e, ok = m.dirty[key]
			// 删除dirty表数据
			delete(m.dirty, key)
			// misses++，如果misses大于等于dirty表数量，将dirty表转移到read表，dirty表置空
			m.missLocked()
		}
		// 解锁
		m.mu.Unlock()
	}
	// 如果找到了，将key的value指针清空(因为read表是原生map实现，需要避免读写冲突)
	if ok {
		return e.delete()
	}
	// 如果没有找到，返回false
	return nil, false
}
```
因为原生map不支持并发读写，为了避免读写冲突，那么从read表中删除数据时，只是将原生map对应的key的value置空，下次dirty表从read表拉取数据的时候，忽略value的entry为空的key.  
当删除数据后的结构图如下：  
![img_7.png](/images/posts/2024-03-15-Go%20知识sync%20map/img_7.png)


dirty表从read表拉取数据，跳过value的entry为空的逻辑如下：  
![img_8.png](/images/posts/2024-03-15-Go%20知识sync%20map/img_8.png)


使用cas，设置旧值为nil，来判断是否为nil，如果是nil ,返回true,跳过key的拷贝。
# 4. 总结
sync map 将互斥锁内置实现并发读写，将互斥锁的范围仅仅限定在dirty表，减少锁等待和锁调用，提升性能。因为dirty表和read表总是在争取保持一致，所以大部分读场景下，read表就能查询到数据，适合读多写少。  
sync map 中read表和dirty表都会持有key，所以内存占用上会比较大，而且在写多读少的场景下，因为既要遍历read表，又要遍历dirty表，性能上会比较慢。  
