---
layout: post
title:  "Java多线程（五）--其它线程安全问题"
date:   2016-09-08 23:14:54
categories: Core_Java
tags: Java 多线程
---

* content
{:toc}

前面四篇文章已经把基本的Java多线程编程的知识介绍完了，下面介绍下其余的一些线程编程技巧，包括

1. 如何包装线程不安全的集合类

2. 使用ThreadLocal线程本地变量

3. 使用concurrent包下线程安全的集合类

4. 使用原子类编程处理并发问题

5. 使用final关键字处理并发问题




### 包装线程不安全的集合

\n换行符问题

使用Java编程时经常使用的集合类ArrayList、HashSet和HashMap等都是线程不安全的集合。

当多个线程同时访问同一个这种线程不安全的集合类时，就会出现问题。

因此Java提供了一个Collections工具类中的多个静态方法来将线程不安全的集合类包装成线程安全的集合类。


这些静态方法都是以synchronized开头的方法。方法签名如下：

```java
// 返回指定 collection 支持的同步（线程安全的）collection。
<T> Collection<T> synchronizedCollection(Collection<T> c) 
// 返回指定列表支持的同步（线程安全的）列表。
<T> List<T> synchronizedList(List<T> list) 
// 返回由指定映射支持的同步（线程安全的）映射。
<K,V> Map<K,V> synchronizedMap(Map<K,V> m) 
// 返回指定 set 支持的同步（线程安全的）set。
<T> Set<T> synchronizedSet(Set<T> s) 
// 返回指定有序映射支持的同步（线程安全的）有序映射。
<K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m) 
// 返回指定有序 set 支持的同步（线程安全的）有序 set。
<T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s) 
```

一般直接以实例化的对象作为这些方法的参数，避免包装之前集合就被线程修改了。

这里以ArrayList为例，代码示例如下：

```java
public class SynchronizedListTest {
	public static void main(String[] args) {
		List<String> list = Collections.synchronizedList(new ArrayList<String>());
		System.out.println(list.getClass());
	}
}
```

代码执行结果为

![Collections.jpg](/images/Core_Java/JavaThread5/Collections.jpg)


从反射得到的信息可以知道包装后的集合是Collections的内部类SynchronizedRandomAccessList。而通过源码可以看出其它的集合被响应的方法包装后也是Collections对应的内部类，分别对应SynchronizedCollection、SynchronizedSet、SynchronizedSortedSet、SynchronizedMap、SynchronizedSortedMap等Collections的内部类。

### 使用线程安全的集合类

前面说到一个技巧是包装线程不安全的集合类，可以得到一个线程安全的集合类。这种方法一般是一开始没有考虑到多线程环境或者为了效率。若是一开始就知道程序会运行在高并发情景下，并且数据的正确性比较重要但是程序效率又没那么重要时，可以考虑直接使用java.util.concurrent包下的线程安全的集合类。

Java提供的线程安全的集合有两类。一是类名以Concurrent开头的类，如ConcurrentHashMap、ConcurrentLinkedQueue、ConcurrentSkipListMap、ConcurrentSkipListSet和ConcurrentLinkedDeque。这一类集合采用了非常复杂的算法保证永远不会锁住集合，任何时候读操作都是允许的，而写操作也有比较好的性能。

另一种是类名以CopyOnWrite开头的集合，如CopyOnWriteArrayList和CopyOnWriteArraySet。这一类集合采用的是写操作先复制一份，写操作在副本中进行，因此不会影响读操作。

根据Java提供的文档，对线程安全的集合绘制了一个类图，如下所示

![ConcurrentCollection.jpg](/images/Core_Java/JavaThread5/ConcurrentCollection.jpg)

### 使用ThreadLocal类

ThreadLocal类提供的是线程本地值，对于每一个访问该类实例的线程，都有一个副本值，线程所有的取值设值操作都在该线程的本地值中操作，互不相干扰。

ThreadLocal有以下三个方法：

T get() 返回此线程局部变量的当前线程副本中的值。

void remove() 移除此线程局部变量当前线程的值。

void set(T value) 将此线程局部变量的当前线程副本中的值设置为指定值。

代码示例如下：

```java
class Bean{
	private ThreadLocal<String> threadLocalValue = new ThreadLocal<String>();
	public Bean(String value){
		this.threadLocalValue.set(value);
	}
	public ThreadLocal<String> getThreadLocalValue() {
		return threadLocalValue;
	}
	public void setThreadLocalValue(ThreadLocal<String> threadLocalValue) {
		this.threadLocalValue = threadLocalValue;
	}
}
public class ThreadLocalTest extends Thread{
	private Bean bean;
	public ThreadLocalTest(Bean bean, String name){
		this.bean = bean;
		this.setName(name);
	}
	@Override
	public void run() {
		ThreadLocal<String> threadLocalValue = bean.getThreadLocalValue();
		threadLocalValue.set(this.getName());
		System.out.println(this.getName() + "的线程本地变量值为：" + threadLocalValue.get());
	}
	public static void main(String[] args) {
		Bean bean = new Bean("初始值");
		System.out.println(Thread.currentThread().getName() + "的线程本地变量值为：" + bean.getThreadLocalValue().get());
		bean.getThreadLocalValue().set("修改值");
		System.out.println(Thread.currentThread().getName() + "的线程本地变量值为：" + bean.getThreadLocalValue().get());
		new ThreadLocalTest(bean, "线程1").start();
		new ThreadLocalTest(bean, "线程2").start();
	}
}
```

代码执行结果为

![ThreadLocal.jpg](/images/Core_Java/JavaThread5/ThreadLocal.jpg)

ThreadLocal还有一个子类InheritableThreadLocal，该类扩展了 ThreadLocal，为子线程提供从父线程那里继承的值：在创建子线程时，子线程会接收所有可继承的线程局部变量的初始值，以获得父线程所具有的值。通常，子线程的值与父线程的值是一致的；但是，通过重写这个类中的 childValue 方法，子线程的值可以作为父线程值的一个任意函数。

### 使用原子类

Java的concurrent包下除了线程安全的集合类和线程池框架外，还有一个atomic的包，里面提供了很多Atomic开头的原子类。

比如AtomicInteger、AtomicLong、AtomicBoolean等，这些原子类是jdk内部基于CAS无等待乐观锁的方式实现的，这些类里面提供的public方法都可以被认为是线程安全的。

当然，这些类的声明需要跟volatile关键字结合使用。

volatile关键字的使用在[Java多线程（三）--线程锁](/2016/09/06/JavaThread3/)中已经介绍过了，使用了原子类AtomicInteger举例，这里不重复介绍了。

### 使用final关键字

Java的final关键字的语义是不可变，可以用来修饰字段、方法和类。

当final关键字修饰字段时，该字段在被赋予初始值之后就不可变了，即不可被修改了，修改final关键字修饰的字段是编译不通过的。

当final关键字修饰方法时，该方法可以被子类继承但是不能被重写，一般与设计模式中的模板方法模式一起使用。

当final关键字修饰类时，该类是不可变得，即不可被子类继承的。Java里最典型的不可变类是String类。

即**字段不可修改、方法不可重写、类不可继承**

在并发环境下，我们可以使用final来修饰字段，既然字段是不可被修改的，自然就不存在数据不同步的问题了。这也是函数式编程语言比较适合并发编程的原因，因为数据是不可变得。

### 总结

关于多线程的话题到这里就结束了。

本文详细介绍了Java多线程编程的技术，包括多线程的概念和意义，创建多线程的三种方式，Java线程的生命周期，Java的线程锁机制，还包括Java多线程的核心应用线程池的管理。除此之外，还介绍了一些在多线程编程中的一些技巧。

本文通过记录Java多线程方方面面的基础知识，对每一个基础知识点都是用Java代码实现了一遍，并将那些代码的执行结果分别截图记录了下来，加深了对Java多线程技术的理解。并且，通过对Java多线程里jdk提供的API做了详细的描述，通过对API绘制类图的方式，理清楚了Java多线程里的各个类的继承关系。

Java多线程编程是属于易学难精的知识，没有实际项目的实战是不可能精通的。本文对于Java多线程编程的基础知识点进行了详细的整理，相信通过掌握这些Java多线程编程的基础知识点能在将来实际的项目中发挥重大作用。
