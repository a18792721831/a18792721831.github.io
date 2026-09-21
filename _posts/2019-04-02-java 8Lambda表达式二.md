---
layout: post
title: "java 8Lambda表达式二"
date: 2019-04-02 20:56:00 +0800
categories: [java8新特性源码解读, Java8新增Stream的常用方法使用, Stream的源码解读, 如何使用函数式编程进行一系列的处理, Java8使用起来如何的方便]
description: "java 8Lambda表达式二1.Filter2.collect3.Java8之前的替换4.1.Filter源码:Stream<T> filter(Predicate<? super T> predicate);package java.util.function;import java.util.Objects;@FunctionalInterfacep..."
keywords: java8新特性源码解读, Java8新增Stream的常用方法使用, Stream的源码解读, 如何使用函数式编程进行一系列的处理, Java8使用起来如何的方便
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88962415
> - 发布时间：2019-04-02 20:56:00
> - 阅读量：387
> - 分类：java同时被 2 个专栏收录, 订阅专栏, Java8
> - 标签：#java8新特性源码解读, #Java8新增Stream的常用方法使用, #Stream的源码解读, #如何使用函数式编程进行一系列的处理, #Java8使用起来如何的方便

## 摘要

文章浏览阅读387次。java 8Lambda表达式二1.Filter2.collect3.Java8之前的替换4.1.Filter源码:Stream<T> filter(Predicate<? super T> predicate);package java.util.function;import java.util.Objects;@FunctionalInterfacep...

---

#### java 8Lambda表达式二

  * 1.Filter
  * 2.collect
  * 3.Java8之前的替换
  * 4.java8的替换Map
  * 5.stream的Filter
  * 6.stream的flatMap
  * 7.stream的max
  * 8.stream的min
  * 9.使用reduce实现stream的max和min方法
  * 10.Java8新增方法调用操作符：：
  * 11.Java8新增方法调用操作符静态方法调用
  * 12.Java8高阶函数的调用

## 1.Filter

源码:
    
    
    Stream<T> filter(Predicate<? super T> predicate);
    
    
    
    package java.util.function;
    import java.util.Objects;
    @FunctionalInterface
    public interface Predicate<T> {
        boolean test(T t);
        default Predicate<T> and(Predicate<? super T> other) {
            Objects.requireNonNull(other);
            return (t) -> test(t) && other.test(t);
        }
        default Predicate<T> negate() {
            return (t) -> !test(t);
        }
        default Predicate<T> or(Predicate<? super T> other) {
            Objects.requireNonNull(other);
            return (t) -> test(t) || other.test(t);
        }
        static <T> Predicate<T> isEqual(Object targetRef) {
            return (null == targetRef)
                    ? Objects::isNull
                    : object -> targetRef.equals(object);
        }
    }
    
    

创建一个方法，生成List
    
    
    	private List<User> getUsers(){
    		Supplier<User> userSupplier = () -> {
    			User user = new User();
    			user.setId((long)(Math.random()*100));
    			user.setName("name:"+Math.random());
    			return user;
    		};
    		Supplier<List<User>> userListSupplier = () -> {
    			List<User> userList = new ArrayList<>();
    			for(int i = 0;i < 10;i++){
    				userList.add(userSupplier.get());
    			}
    			return userList;
    		};
    		return userListSupplier.get();
    	}
    

测试代码
    
    
    @Test
    	public void testFilter(){
    		List<User> userList = getUsers();
    		userList.stream().forEach(u -> System.out.println(u.getId()+u.getName()));
    		userList.stream().filter(p -> {
    			System.out.println(p.getName());
    			return p.getId().equals(2L);
    		}).count();
    		//及早求值
    		System.out.println("nocount");
    		userList.stream().filter(p -> {
    			System.out.println(p.getName());
    			return p.getId().equals(2L);
    		});
    		//惰性求值
    		//思想：使用一系列的惰性求值定义操作过程，使用及早求值获得结果
    		//判断：返回值是Stream类型就是惰性求值，返回其余类型或者空就是及早求值
    	} 
    
    
    
    74name:0.0585052399557352
    5name:0.5363716565301203
    74name:0.35460050578903934
    36name:0.5314012848311339
    67name:0.8668274048065532
    82name:0.5669007695259906
    45name:0.9243080485802774
    19name:0.8909273339608627
    92name:0.9212101325052954
    47name:0.09911401539015208
    name:0.0585052399557352
    name:0.5363716565301203
    name:0.35460050578903934
    name:0.5314012848311339
    name:0.8668274048065532
    name:0.5669007695259906
    name:0.9243080485802774
    name:0.8909273339608627
    name:0.9212101325052954
    name:0.09911401539015208
    nocount
    
    

## 2.collect
    
    
     <R, A> R collect(Collector<? super T, A, R> collector);
    
    
    
    	@Test
    	public void testCollect(){
    		List<String> oneList = Stream.of("a","b","hello").collect(Collectors.toList());
    		assertEquals(Arrays.asList("a","b","hello"),oneList);
    		oneList.forEach(o -> System.out.println(o));
    	}
    
    
    
    a
    b
    hello
    
    

## 3.Java8之前的替换
    
    
    	@Test
    	public void testMapFor(){
    		List<String> list = new ArrayList<>();
    		for(String string : Arrays.asList("a","b","hello")){
    			String str = string.toUpperCase();
    			list.add(str);
    		}
    		assertEquals(Arrays.asList("A","B","HELLO"), list);
    		list.forEach(l -> System.out.println(l));
    	}
    
    
    
    A
    B
    HELLO
    
    

## 4.java8的替换Map

源码:
    
    
    <R> Stream<R> map(Function<? super T, ? extends R> mapper);
    
    
    
    package java.util.function;
    import java.util.Objects;
    @FunctionalInterface
    public interface Function<T, R> {
        R apply(T t);
        default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
            Objects.requireNonNull(before);
            return (V v) -> apply(before.apply(v));
        }
        default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
            Objects.requireNonNull(after);
            return (T t) -> after.apply(apply(t));
        }
        static <T> Function<T, T> identity() {
            return t -> t;
        }
    }
    
    
    
    
    	@Test
    	public void testMapTo() {
    		assertEquals(Arrays.asList("A", "B", "NIHAO"),
    				Stream.of("a", "b", "nihao").map(str -> {
    					System.out.println(str);
    					return str.toUpperCase();
    				}).collect(Collectors.toList()));
    	}
    
    
    
    a
    b
    nihao
    

## 5.stream的Filter
    
    
        @SafeVarargs
        @SuppressWarnings("varargs") // Creating a stream from an array is safe
        public static<T> Stream<T> of(T... values) {
            return Arrays.stream(values);
        }
    
    
    
    	@Test
    	public void testStreamFilter() {
    		assertEquals(Arrays.asList("a", "aa", "aaa", "aaaa"),
    				Stream.of("a", "aa", "b", "bb", "aaa", "aaaa").filter(str -> {
    					System.out.println(str);
    					return str.startsWith("a");
    				}).collect(Collectors.toList()));
    	}
    
    
    
    a
    aa
    b
    bb
    aaa
    aaaa
    
    

## 6.stream的flatMap
    
    
    	@Test
    	public void testStreamFlatMap() {
    		assertEquals(
    		// [1,2,3,4]
    				Arrays.asList(1, 2, 3, 4),
    				// [[1,2],[3,4]]
    				Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4))
    						.flatMap(numbers -> numbers.stream())
    						.collect(Collectors.toList()));
    	}
    

flatMap是整合多个流为一个流  
结果断言为true

## 7.stream的max

源码:
    
    
        @Override
        public final Optional<P_OUT> max(Comparator<? super P_OUT> comparator) {
            return reduce(BinaryOperator.maxBy(comparator));
        }
    
    
    
        public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
            Objects.requireNonNull(comparator);
            return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
        }
    
    
    
    	@Test
    	public void testStreamMax(){
    		Integer integer = Stream.of(1,2,3,4,5).max(Comparator.comparing(t -> t)).get();
    		System.out.println(integer);
    	}
    

max用deduce实现的
    
    
    5
    

## 8.stream的min

源码:
    
    
        @Override
        public final Optional<P_OUT> min(Comparator<? super P_OUT> comparator) {
            return reduce(BinaryOperator.minBy(comparator));
    
        }
    
    
    
        public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
            Objects.requireNonNull(comparator);
            return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
        }
    

结果  
1

## 9.使用reduce实现stream的max和min方法
    
    
    	@Test
    	public void testMyReduce(){
    		Integer res = Integer.MAX_VALUE;
    		System.out.println(res);
    		System.out.println(Stream.of(1,2,3,4,5).reduce((t1,t2) -> t1 < t2 ? t1 : t2).get());
    	}
    
    
    
    2147483647
    1
    
    
    
    
    	@Test
    	public void testMyReduce(){
    		Integer res = Integer.MIN_VALUE;
    		System.out.println(res);
    		System.out.println(Stream.of(1,2,3,4,5).reduce((t1,t2) -> t1 >= t2 ? t1 : t2).get());
    	}
    
    
    
    -2147483648
    5
    
    

## 10.Java8新增方法调用操作符：：
    
    
    	@Test
    	public void testMethodUse(){
    		Operator operator = () -> "method use";
    		Operator ooOperator = operator::getMessage;
    		System.out.println(ooOperator.getMessage());
    	}
    
    
    
    method use
    

## 11.Java8新增方法调用操作符静态方法调用
    
    
    	@Test
    	public void testStaticMethod(){
    		Operator operator = StaticMethodUse::message;
    		System.out.println(operator.getMessage());
    	}
    

static Message use

## 12.Java8高阶函数的调用

什么是高阶函数？  
函数的参数列表是一个函数，那么这个函数就是一个高阶函数。  
举个例子：  
stream的max,min,map等等方法，查看其方法定义一定是一个函数，或者是一个接口的定义，或者可以接收一个Lambda的表达式作为参数。  
这种方法就是高阶函数。  
高阶函数有什么用？  
高阶函数可以把一系列的操作连接起来，形成一个连贯的处理流程。  
因为高阶函数一般情况下返回的是流的对象，也就是惰性求值。  
所以使用一系列的高阶函数进行惰性求值处理，当处理完成后需要返回结果时，使用及早求值的方式得出结果，结束整个操作。
    
    
    	@Test
    	public void apptly(){
    		List<User> users = getUsers();
    		System.out.println(users);
    		//找出id中有3的user,只取id
    		System.out.println("id have 3");
    		System.out.println(users.stream()
    				.filter(u -> u.getId().toString().contains("3"))
    				.map(u -> u.getId())
    				.collect(Collectors.toList()));
    		//对user的id中含有3的id进行计数
    		System.out.println("id have 3 count");
    		System.out.println(users.stream()
    				.filter(u -> u.getId().toString().contains("3"))
    				.map(u -> u.getId())
    				.count());
    		//输出user的id中含有3的id
    		System.out.println("sum of id have 3 count");
    		System.out.println(users.stream()
    				.filter(u -> u.getId().toString().contains("3"))
    				.map(u -> u.getId())
    				.collect(Collectors.toList()));
    		//对user的id中含有3的id进行求和
    		System.out.println(users.stream()
    				.filter(u -> u.getId().toString().contains("3"))
    				.map(u -> u.getId())
    				.reduce((u1,u2) -> u1 + u2)
    				.get());
    		//去除id中含有3的user
    		System.out.println("id have 3 for user");
    		System.out.println(users.stream()
    				.filter(u -> u.getId().toString().contains("3"))
    //				.map(u -> u.getId())
    				.collect(Collectors.toList()));
    	}
