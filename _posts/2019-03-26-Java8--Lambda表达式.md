---
layout: post
title: "Java8--Lambda表达式"
date: 2019-03-26 21:04:19 +0800
categories: [Java新特性, Lambda表达式, 函数接口与Lambda表达式, Java8中新增的函数接口如何使用, 函数式编程jdk源码解读]
description: "Lambda表达式1.什么是Lambda表达式2.Lambda的参数列表3.Lambda的返回值4.Lambda表达式的类型推断5.Lambda表达式引用值6.函数接口7.jdk已定义的函数接口7.1BinaryOperator7.2Predicate7.3Consumer7.4Function7.5Supplier7.6UnaryOperator7.7@FunctionalInterface1..._jdk8 lmab"
keywords: Java新特性, Lambda表达式, 函数接口与Lambda表达式, Java8中新增的函数接口如何使用, 函数式编程jdk源码解读
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/88830188
> - 发布时间：2019-03-26 21:04:19
> - 阅读量：456
> - 分类：java同时被 2 个专栏收录, 订阅专栏, Java8
> - 标签：#Java新特性, #Lambda表达式, #函数接口与Lambda表达式, #Java8中新增的函数接口如何使用, #函数式编程jdk源码解读

## 摘要

文章浏览阅读456次。Lambda表达式1.什么是Lambda表达式2.Lambda的参数列表3.Lambda的返回值4.Lambda表达式的类型推断5.Lambda表达式引用值6.函数接口7.jdk已定义的函数接口7.1BinaryOperator7.2Predicate7.3Consumer7.4Function7.5Supplier7.6UnaryOperator7.7@FunctionalInterface1..._jdk8 lmab

---

#### Lambda表达式

  * 1.什么是Lambda表达式
  * 2.Lambda的参数列表
  * 3.Lambda的返回值
  * 4.Lambda表达式的类型推断
  * 5.Lambda表达式引用值
  * 6.函数接口
  * 7.jdk已定义的函数接口
  *     * 7.1BinaryOperator
    * 7.2Predicate
    * 7.3Consumer
    * 7.4Function
    * 7.5Supplier
    * 7.6UnaryOperator
    * 7.7@FunctionalInterface

## 1.什么是Lambda表达式

Lambda表达式是一种进奏的，传递行为的方式。  
Lambda表达式的构成：  
参数列表 -> 行为操作  
如何理解Lambda表达式：  
前提条件：一个接口中只定义了一个方法。
    
    
    package intf;
    
    public interface Operator {
    
    	String getMessage();
    }
    
    

不使用Lambda表达式。
    
    
    @Test
    	public void testGetMessage(){
    		System.out.println(new Operator(){
    
    			@Override
    			public String getMessage() {
    				return "内部类";
    			}
    			
    		}.getMessage());
    	}
    

使用内部类的方式。  
执行结果
    
    
    内部类
    
    

可以发现为了返回一个字符串，我们多写的许多的代码。
    
    
    @Test
    	public void testGetMessageLmbda(){
    		Operator operator = () -> {return "Lmbda";};
    		System.out.println(operator.getMessage());
    	}
    

Lambda只需要1行代码。
    
    
    Lmbda
    
    

## 2.Lambda的参数列表

根据Lambda表达式的定义 -> 前面的部分是定义的行为的参数列表。  
前提条件：一个接口只有一个方法，方法有一个参数:
    
    
    package intf;
    
    public interface OperatorOne {
    
    	String getMessage(String string);
    	
    }
    
    

内部类调用
    
    
    @Test
    	public void testGetMessageOne(){
    		System.out.println(new OperatorOne() {
    			
    			@Override
    			public String getMessage(String string) {
    				return string+string;
    			}
    		}.getMessage("hello"));
    	}
    

执行结果
    
    
    hellohello
    
    

Lambda表达式调用
    
    
    @Test
    	public void testGetMessageOneLmbda(){
    		OperatorOne operatorOne = string -> {return string+string;};
    		System.out.println(operatorOne.getMessage("nihao"));
    	}
    

执行结果
    
    
    nihaonihao
    
    

当然，我们只测试了一个简单类型，实际使用中不受限制的，接口的方法中定义了多少的参数，在 ->前面就可以写多少的参数  
比如：
    
    
    (a,b) -> a+b;//a+b的Lambda的表达式
    

## 3.Lambda的返回值

Lambda表达式定义的是行为，行为就有可能需要返回操作结果，而Lambda表达式也可以支持有返回结果的行为。  
比如在2中我们尝试的就是有返回值的行为。  
但是存在某些行为是没有返回值的，当然，Lambda同样支持。
    
    
    package intf;
    
    public interface OperatorNoResturn {
    
    	void showMessage();
    	
    }
    
    
    
    @Test
    	public void testShowNoResult(){
    		OperatorNoResturn operatorNoResturn = () -> System.out.println("no return");
    		operatorNoResturn.showMessage();
    	}
    
    
    
    no return
    
    

也有可能是有参数但是没有返回值的行为
    
    
    package intf;
    
    public interface OperatorNoResturnOnePam {
    
    	void showMessage(String string);
    	
    }
    
    
    
    
    @Test
    	public void testShowMessageNoResyltOnePam(){
    		OperatorNoResturnOnePam operatorNoResturnOnePam = string -> System.out.println(string+string);
    		operatorNoResturnOnePam.showMessage("hello");
    	}
    
    
    
    hellohello
    
    

## 4.Lambda表达式的类型推断

在上述中很多的Lambda表达式是这样的：
    
    
    package intf;
    
    public interface OperatorOne {
    
    	String getMessage(String string);
    	
    }
    
    

但是在使用Lambda表达式定义行为的时候：
    
    
    @Test
    	public void testGetMessageOneLmbda(){
    		OperatorOne operatorOne = string -> {return string+string;};
    		System.out.println(operatorOne.getMessage("nihao"));
    	}
    

可以看到等于号后面的string实际上是一个局部变量，表示的是接口方法的形参，并没有指定类型。  
为什么可以通过编译，也可以使用？  
因为Java8支持了上下文类型推断。  
简单的说，就是在OperatorOne的interface中定义的方法中已经规定了getMessage的方法的形参的类型必须是String。  
这也是为什么Lambda表达式实现的接口必须有一个且只能有一个公共的未实现的方法。  
上述Lambda表达式调用:  
OperatorOne operatorOne = string -> {return string+string};  
解析一下这句的含义。  
首先OperatorOne表示接下来的Lambda表达式将要定义的是OperatorOne里面定义的行为接口。  
因为OperatorOne里面只有一个行为接口，所以就能确定一定是未实现的接口。  
那么未实现的接口的参数列表以及返回值就全部能够得到了，此时我只需要定义局部的形参变量就行了，而无需显示的指定类型。  
这就是Java8 关于上下文的类型推断。

## 5.Lambda表达式引用值

Lambda表达式定义行为的实现时，会遇到和内部类相同的问题：  
在内部类访问外部的局部或者全局变量。  
如果在内部类中访问外部的变量，那么就必须把外部的变量显示的定义为final类型。

在Java8 的Lambda中不需要显示的定义变量为final 类型，但是所引用的变量一定符合final类型变量的条件（值不能再次改变）  
如下正确的：
    
    
    	@Test
    	public void testFinal(){
    		String string = "nihao";
    		OperatorNoResturn operatorNoResturn = () -> System.out.println(string);
    		operatorNoResturn.showMessage();
    	}
    

如下异常的：
    
    
    	@Test
    	public void testNoFinal(){
    		String string = "nihao";
    		string += "!";
    		OperatorNoResturn operatorNoResturn = () -> System.out.println(string);
    		operatorNoResturn.showMessage();
    	}
    

异常信息
    
    
    Description	Resource	Path	Location	Type
    Local variable string defined in an enclosing scope must be final or effectively final	Main.java	/study_java8Lambda/src/client	line 100	Java Problem
    
    

因为Lambda必须使用满足final类型的变量也就是值不可更改的变量，所以就会在Lambda中把外部变量(不满足final类型)与值(满足final类型)进行隔离。

同样，Lambda表达式是闭包的也是因为这个原因。

在很多书中：满足final的变量，但是此变量没有用final定义的变量称为引用值或者既成事实的变量。

我更乐意理解为隐式的final变量。

## 6.函数接口

Java8的特性之一就是函数式编程。  
首先函数接口的定义是什么？  
在上述我们测试Lambda表达式中，表达式实现的接口就是函数式接口。  
所以：  
函数式接口简单定义就是：只有一个抽象方法的接口。  
函数式接口一般与Lambda表达式使用。

但是如果一个接口内只能有一个抽象方法，那么需要使用接口就必须实现接口中的抽象方法后才能使用，而有许多的操作在不同的抽象方法的实现中是一样的，那么我在每一次实现抽象方法都写一次吗？  
Java8 在接口的方法类型中增加了一个类型  
原来：  
protected 保护方法  
public 公共方法  
默认的包内方法  
之前只能使用上述3中类型  
Java8 中增加了  
default类型：默认方法。  
对于default的定义是这样的：  
继承接口或者实现接口但是未实现默认方法的类，自动获得默认方法。  
举个例子
    
    
    public interface A{
    	default void show(){
    	System.out.print("A");
    }
    }
    
    
    
    public interface B extends A{
    
    }
    
    
    
    public class C implements B{
    }
    

在上述的3个定义中，A,B,C都具有show这个方法，都可以使用show方法。  
如果在C类中重写了show方法，那么就会调用重写的方法。  
当然在B中也可以重写show方法。

## 7.jdk已定义的函数接口

首先需要明白一个事实：使用Lambda表达式是定义操作不是定义值。  
Lambda表达式描述的是操作过程不是值。

### 7.1BinaryOperator

源码：
    
    
    package java.util.function;
    
    import java.util.Objects;
    import java.util.Comparator;
    @FunctionalInterface
    public interface BinaryOperator<T> extends BiFunction<T,T,T> {
        public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
            Objects.requireNonNull(comparator);
            return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
        }
        public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
            Objects.requireNonNull(comparator);
            return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
        }
    }
    
    
    
    
    package java.util.function;
    import java.util.Objects;
    @FunctionalInterface
    public interface BiFunction<T, U, R> {
        R apply(T t, U u);
        default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
            Objects.requireNonNull(after);
            return (T t, U u) -> after.apply(apply(t, u));
        }
    }
    
    

很明显，使用BinaryOperator函数接口，需要具体使用Lambda定义的操作就是apply抽象方法。  
apply方法有两个参数，一个返回值。  
当然TUR是同一种类型，否则会报异常，编译无法通过。
    
    
    	@Test
    	public void testBinaryOperator1(){
    		BinaryOperator<Integer> addOperator = (x,y) -> x+y;
    		System.out.println(addOperator.apply(32, 28));
    	}
    
    
    
    60
    
    

复杂一点的
    
    
    @Test
    	public void testBinaryOperator2(){
    		BinaryOperator<Long> ffOperator = (x,y) -> {
    			long resut = 1L;
    			for(int i = 0;i < y;i++){
    				resut *= x;
    			}
    			return resut;
    		};
    		System.out.println(ffOperator.apply(5L, 4L));
    	}
    
    
    
    625
    
    

### 7.2Predicate

源码
    
    
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
    
    

Predicate函数接口需要实现的是boolean test(T t);方法  
有and、negate、or、isEqual默认方法  
这里有这样几个概念  
上界通配符  
下界通配符  
上转型  
下转型  
上界通配符<? extend T>目标类是T的子类，T是类集合的上界，故，上界通配符  
下界通配符<? super T>目标类是T的超类，T是类集合的下界，故，下界通配符  
上转型  
下转型  
什么意思呢:
    
    
    		//上界通配符<? extend T>目标类是T的子类，T是类集合的上界，故，上界通配符
    		//下界通配符<? super T>目标类是T的超类，T是类集合的下界，故，下界通配符
    		//上转型
    		//下转型
    		User fUser = new VipUser();//上转型father f = new son();
    		VipUser sUser = (VipUser) fUser;//下转型son s = (son) father;
    		//上转型实际上缩小了子类的范围，上转型中无法使用任何子类扩展父类之后的操作或者属性
    		//（共有或者重写的以子类为准）
    		//下转型的目标必须是上转型的对象，就是son s = (son) father中的father实际上
    		//是一个son类型，只是下转型之前顶着一个father的壳子
    		//上界通配符中所定的范围是T的子类，所以一般可以取值，使用下转型实现。
    		//但是无法存值，因为不能确定具体的子类类型，无法满足最大范围原则。
    		//下界通配符所指定的范围是T的超类也叫父类，可以存值，也可以取值，也是使用上转型实现。
    		//存值可以使用下转型实现，满足最大范围原则，取值使用上转型
    		//因为任何类的最终父类都是Object类的子类，下界通配符默认的上界就是Object类
    		//所以下界通配符可以取值，取出的值只能用Object去存。
    		//实际上还是上转型与下转型。
    		//最大范围原则：尽最大程度的保证对象的操作范围。
    
    
    
    @Test
    	public void testPredicate(){
    		Predicate<String> testSOperator = string -> string.contains("s");
    		System.out.print("test:");
    		System.out.println(testSOperator.test("sorry"));
    	}
    
    
    
    test:true
    
    
    
    
    package domain;
    
    import java.io.Serializable;
    
    public class User implements Serializable{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = -8794064868071396333L;
    
    	protected Long id;
    	
    	protected String name;
    
    	public Long getId() {
    		return id;
    	}
    
    	public void setId(Long id) {
    		this.id = id;
    	}
    
    	public String getName() {
    		return name;
    	}
    
    	public void setName(String name) {
    		this.name = name;
    	}
    	
    }
    
    
    
    
    package domain;
    
    public class VipUser extends User{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 1241959620505393984L;
    
    }
    
    
    
    
    @Test
    	public void testPredicateAnd(){
    		Predicate<User> andOperator = user -> user.getId().equals(7L);
    		Predicate<VipUser> andVipOperator = vipUser -> vipUser.getName().contains("a");
    		VipUser user = new VipUser();
    		user.setId(7L);
    		user.setName("mm");
    		VipUser vipUser = new VipUser();
    		vipUser.setId(7L);
    		vipUser.setName("sadw");
    		System.out.print("and:");
    		System.out.println(andVipOperator.and(andOperator).test(vipUser));
    		System.out.println(andVipOperator.and(andOperator).test(user));
    	}
    
    
    
    and:true
    false
    
    
    
    
    	@Test
    	public void testPredicateOr(){
    		Predicate<User> orOperator = user -> user.getId().equals(4L);
    		Predicate<User> orOperator1 = user -> user.getName().contains("s");
    		User user = new User();
    		user.setId(4L);
    		user.setName("hh");
    		System.out.println("id:4,name:hh");
    		System.out.println("or:"+orOperator.or(orOperator1).test(user));
    		user.setId(3L);
    		user.setName("sd");
    		System.out.println("id:3,name:sd");
    		System.out.println("or:"+orOperator.or(orOperator1).test(user));
    	}
    
    
    
    id:4,name:hh
    or:true
    id:3,name:sd
    or:true
    
    

### 7.3Consumer

源码
    
    
    package java.util.function;
    import java.util.Objects;
    @FunctionalInterface
    public interface Consumer<T> {
        void accept(T t);
        default Consumer<T> andThen(Consumer<? super T> after) {
            Objects.requireNonNull(after);
            return (T t) -> { accept(t); after.accept(t); };
        }
    }
    
    
    
    
    	@Test
    	public void testConsumer(){
    		Consumer<User> firstConsumer = user -> System.out.println(user);
    		Consumer<User> secondConsumer = user -> {
    			System.out.println(user.getId());
    			System.out.println(user.getName());
    		};
    		User user = new User();
    		user.setId(1L);
    		user.setName("testConsumer");
    		firstConsumer.andThen(secondConsumer).accept(user);
    	}
    
    
    
    domain.User@5e8c92f4
    1
    testConsumer
    
    

### 7.4Function

源码
    
    
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
    	public void testFunction(){
    		Function<Long, String> firstFunction = l -> l+"ww";
    		Function<String, String> beforeFunction = string -> string+"before";
    		Function<String, String> afterFunction = string -> string + "after";
    		System.out.println(firstFunction.apply(3L));
    		System.out.println(firstFunction.andThen(afterFunction).apply(25L));
    		System.out.println(afterFunction.compose(beforeFunction).apply("nihao"));
    	}
    
    
    
    3ww
    25wwafter
    nihaobeforeafter
    
    

### 7.5Supplier

源码
    
    
    package java.util.function;
    @FunctionalInterface
    public interface Supplier<T> {
        T get();
    }
    
    
    
    
    @Test
    	public void testSupplier(){
    		Supplier<String> supplier = () -> "hello";
    		System.out.println(supplier.get());
    	}
    
    
    
    hello
    
    

### 7.6UnaryOperator

源码
    
    
    package java.util.function;
    @FunctionalInterface
    public interface UnaryOperator<T> extends Function<T, T> {
        static <T> UnaryOperator<T> identity() {
            return t -> t;
        }
    }
    
    
    
    
    	@Test
    	public void testUnaryOperator(){
    		UnaryOperator<String> unaryOperator = string -> string + "hello";
    		System.out.println(unaryOperator.apply("hh"));
    	}
    
    
    
    hhhello
    
    
    
    
    	@Test
    	public void testThreadLocal(){
    		Runnable runnable = () -> System.out.println("thread"+this.getClass());
    		runnable.run();
    	}
    
    
    
    threadclass client.Main
    
    

### 7.7@FunctionalInterface

@FunctionalInterface注解用来标注这个接口是一个函数接口。

使用@FunctionalInterface标注一个接口有两层含义：  
1.这个接口接收Lambda表达式定义操作行为；  
2.这个接口只能有一个抽象方法。

对于一个接口中只有一个抽象方法时，用@FunctionalInterface标注和不用标注的结果是一样的。

但是如果一个接口标注了@FunctionalInterface注解，那么在接口中试图增加其他的抽象方法时，编译器会报异常：
    
    
    Description	Resource	Path	Location	Type
    Invalid '@FunctionalInterface' annotation; Operator is not a functional interface	Operator.java	/study_java8Lambda/src/intf	line 4	Java Problem
    
    

总结一下：  
@FunctionalInterface注解的作用就是保证使用此注解标注的接口只有一个抽象方法。
