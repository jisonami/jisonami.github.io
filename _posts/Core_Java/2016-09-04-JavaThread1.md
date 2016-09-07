---
layout: post
title:  "Java多线程（一）--线程是怎么创建的"
date:   2016-09-04 22:28:54
categories: Core_Java
tags: Java 多线程
---

* content
{:toc}

操作系统同时运行着多个任务，每一个任务就是一段内存中运行的程序，也叫进程。而这个运行的程序又可能包含多个同时执行的程序片段，这每一个执行程序片段的过程就叫线程。

多线程就是程序利用进程空闲时间不停地在各个线程之间进行切换，已达到多个程序片段同时进行的效果的过程，也就是多个线程同时执行。

其实线程的概念跟进程很类似，据不可靠消息来源，因为当年苹果计算机的操作系统是单进程操作系统，Java需要实现跨平台的并发程序，所以抽象出了线程的概念。

Java多线程一共五篇，包括线程创建、线程生命周期、线程锁、线程池和线程安全，仅以此文纪念那篇的很水毕业论文。




### 创建和启动一个Java线程有三种方式

#### 以继承Thread类的方式创建线程

Thread类里面有个run()方法，run()方法包装线程执行代码。然而启动线程并不是调用run()方法启动的，而是调用Thread里的start()方法。直接调用run()方法的话并不会启动一个线程，而是直接执行了run()方法体里面的代码片段，达不到创建线程的方法，

**因此通过继承Thread类的方式创建线程的步骤如下**

* 创建Thread类的子类并重写run()方法
* 实例化第(1)步中Thread类的子类为线程对象
* 调用线程对象的start()方法

下面是代码示例：

```java
public class ThreadTest extends Thread{
	@Override
	public void run() {
		System.out.println("线程执行中");
	}
	public static void main(String[] args) {	
		Thread thread = new ThreadTest();
		System.out.println("start()方法调用前");
		thread.start();
		System.out.println("start()方法调用后");
	}
}
```

代码执行结果为：

![ExtendsThreadClass.png](/images/Core_Java/JavaThread1/ExtendsThreadClass.png)

从执行结果中可以看出，线程的执行是异步的，并不是顺序执行的。这就是以继承Thread类的方式创建线程。


#### 以实现Runnable接口的方式创建线程
	
Runnable接口只有一个run()方法，Thread类其实也实现了Runnable接口。同样的，也是这个run()方法包装线程执行代码。

“从Java8开始，Runnable接口使用了@FunctionalInterface修饰，也就是说，Runnable接口是函数式接口，可使用lambda表达式来创建Runnable对象。”
	 
我们可以看到Java8里面Runnable接口的声明如下：

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
	
**实现Runnable接口创建线程的步骤如下：**

* 实现Runnable接口并重写run()方法
* 实例化第(1)步中实现Runnable实现类为线程对象
* 使用Thread类的构造方法来包装Runnable实现类对象

下面是代码示例：

```java
public class RunnableTest implements Runnable{
	@Override
	public void run() {
		System.out.println("线程执行中");
	}
	public static void main(String[] args) {
		Thread thread = new Thread(new RunnableTest());
		System.out.println("start()方法调用前");
		thread.start();
		System.out.println("start()方法调用后");
	}
}
```

代码执行结果为：

![ImplementsRunnableInterface.png](/images/Core_Java/JavaThread1/ImplementsRunnableInterface.png)
 
前面说到Runnable是函数式接口，可以使用lambda表达式简化代码，以下代码执行结果同上。

```java
public class RunnableTest1 {
	public static void main(String[] args) {
		System.out.println("start()方法调用前");
		new Thread( () -> System.out.println("线程执行中") ).start();
		System.out.println("start()方法调用后");
	}
}
```

**需要注意的是，实现Runnable接口方式创建的线程无法获取线程何时执行完成，也无法捕获到线程执行过程中抛出的异常**


#### 以实现Callable接口结合Future接口的方式创建线程
	
前面两种创建线程的方式都是无法从线程中获取返回值的，而接下来要说的这种方式则可以获得返回值，就是实现Callable接口的方式。

**我们先来看看Java8里面Callable接口的声明**

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

我们可以看到，Callable接口跟Runnable接口一样被@FunctionalInterface修饰，是一个函数式接口。

Callable接口只有一个方法call()，有一个返回值V，这是一个泛型的返回值，由实现该接口的类实例化时指定返回值的类型。

**再来看看Future接口的声明**

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

看到Future接口里面声明的方法我们就可以知道，这个类是用来取消线程和判断线程是否完成以及获取线程返回结果的。

Future是接口，不能直接使用，而整个JDk中只有FutureTask跟Future有间接联系。

FutureTask先是实现了RunnableFuture接口，而RunnableFuture接口扩展了Future接口，同时行业扩展了Runnable接口，所以FutureTask也就实现了Future接口。

这两个接口在Java8中的声明分别如下：

```java
public class FutureTask<V> implements RunnableFuture<V> 
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask有个构造方法为

```java
public FutureTask(Callable<V> callable)，
```

**所以就可以得出以下创建线程的步骤**

* 实现Callable接口并重写call()方法
* 实例化第(1)步中实现Callable实现类为线程对象
* 使用FutureTask类的构造方法来包装Runnable实现类对象
* 使用Thread类包装FutureTask的实例对象

下面是代码示例：

```java
public class CallableTest implements Callable<Integer>{
	@Override
	public Integer call() throws Exception {
		System.out.println("线程执行中");
		return 0;
	}
	public static void main(String[] args) throws Exception {
		Callable<Integer> callable = new CallableTest();
		FutureTask<Integer> task = new FutureTask<>(callable);
		Thread thread = new Thread(task);
		System.out.println("start()方法调用前");
		thread.start();
		System.out.println("start()方法调用后");
		System.out.println("获取call方法的返回值：" + task.get());
	}
}
```

代码执行结果为：

 ![ImplementsCallableInterface.png](/images/Core_Java/JavaThread1/ImplementsCallableInterface.png)
 
根据上面的分析可以看出，Callable接口与Runnable接口差别并不大，区别是Callable接口可以有返回值，并且可以抛出异常罢了。

值得注意的是，FutureTask类有个空实现的done()的protected方法，实现该方法即可实现线程回调功能。

代码示例如下

```java
public class CallableTest implements Callable<Integer>{
	@Override
	public Integer call() throws Exception {
		System.out.println("线程执行中");
		Thread.sleep(1000);
		return 0;
	}
	public static void main(String[] args) throws Exception {
		Callable<Integer> callable = new CallableTest();
		FutureTask<Integer> task = new FutureTask<Integer>(callable){
			@Override
			protected void done() {
				System.out.println("done()回调方法执行了");
			}
		};
		
		Thread thread = new Thread(task);
		System.out.println("start()方法调用前");
		thread.start();
		System.out.println("start()方法调用后");
		// FutureTask的get()方法会一致阻塞到回调方法执行完
		System.out.println("获取call方法的返回值：" + task.get());
	}
}
```

代码执行结果为


![CallableCallback.png](/images/Core_Java/JavaThread1/CallableCallback.png)