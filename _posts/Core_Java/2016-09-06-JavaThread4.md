---
layout: post
title:  "Java多线程（四）--线程池"
date:   2016-09-06 23:28:54
categories: Core_Java
tags: Java 多线程
---

* content
{:toc}

我们知道，在Java中启动一个线程是需要一定的资源成本的，使用线程池可以有效的控制系统中线程并发的数量，降低资源的消耗，提高线程的响应速度。

线程池的设计属于享元模式的一种实现。

创建一个线程池我们主要用到java.util.concurrent包下的ThreadPoolExecutor类和它的子类ScheduledThreadPoolExecutor。




根据继承关系，绘制类图如下
 
![ThreadPoolClassDiagram.jpg](/images/Core_Java/JavaThread4/ThreadPoolClassDiagram.jpg)

ThreadPoolExecutor类有四个构造方法，分别是：

**用给定的初始参数和默认的线程工厂及被拒绝的执行处理程序创建新的 ThreadPoolExecutor**

```java
ThreadPoolExecutor(int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime, 
TimeUnit unit, 
BlockingQueue<Runnable> workQueue)
```

**用给定的初始参数和默认的线程工厂创建新的 ThreadPoolExecutor**

```java
ThreadPoolExecutor(int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime, 
TimeUnit unit, 
BlockingQueue<Runnable> workQueue, 
RejectedExecutionHandler handler)
```

用给定的初始参数和默认被拒绝的执行处理程序创建新的 ThreadPoolExecutor。

```java
ThreadPoolExecutor(int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime, 
TimeUnit unit, 
BlockingQueue<Runnable> workQueue,
ThreadFactory threadFactory)
```

用给定的初始参数创建新的 ThreadPoolExecutor。

```java
ThreadPoolExecutor(int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime,
TimeUnit unit, 
BlockingQueue<Runnable> workQueue, 
ThreadFactory threadFactory, 
RejectedExecutionHandler handler)
```

这四个构造方法咋一看，前面三个都是调用第四个构造方法实现的，每一个构造方法都特别复杂，参数很多，使用起来比较麻烦。这里用复杂的一个构造方法说明如何手动创建一个线程池。

JDK6文档上是这么写的：

>public ThreadPoolExecutor(int corePoolSize,  
                          int maximumPoolSize,  
                          long keepAliveTime,  
                          TimeUnit unit,  
                          BlockingQueue<Runnable> workQueue,  
                          ThreadFactory threadFactory,  
                          RejectedExecutionHandler handler)  
用给定的初始参数和默认的线程工厂及被拒绝的执行处理程序创建新的 ThreadPoolExecutor。使用 Executors 工厂方法之一比使用此通用构造方法方便得多。   
参数：  
corePoolSize - 池中所保存的线程数，包括空闲线程。   
maximumPoolSize - 池中允许的最大线程数。   
keepAliveTime - 当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间。   
unit - keepAliveTime 参数的时间单位。   
workQueue - 执行前用于保持任务的队列。此队列仅保持由 execute 方法提交的 Runnable 任务。  
threadFactory - 执行程序创建新线程时使用的工厂。   
handler - 由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序。
	
示例代码如下： 

```java
public class ThreadPoolExecutorTest extends Thread {
	@Override
	public void run() {
		System.out.println("线程执行中");
	}
	public static void main(String[] args) {
		ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 10, 2, TimeUnit.SECONDS, 
				new ArrayBlockingQueue<Runnable>(MIN_PRIORITY), 
				Executors.defaultThreadFactory(), 
				new ThreadPoolExecutor.AbortPolicy());
		for (int i = 0; i < 10; i++) {
			executor.execute(new ThreadPoolExecutorTest());
			System.out.println("线程池中线程数目："+executor.getPoolSize()+"，队列中等待执行的任务数目："+
		             executor.getQueue().size()+"，已执行完的任务数目："+executor.getCompletedTaskCount());
		}
		executor.shutdown();
	}
```

代码执行结果为

![ThreadPool.jpg](/images/Core_Java/JavaThread4/ThreadPool.jpg)

这里在构造ThreadPoolExecutor对象时，使用了参数最多的构造方法，后面两个参数使用的是其它构造方法采取的默认值，平时使用时，我们使用最简单的构造方法构造一个ThreadPoolExecutor线程池对象即可。若是有更复杂的需求，则可以自己实现ThreadFactory接口。在Java6提供的接口里，只有一个Executors的内部类DefaultThreadExecutor实现了ThreadFactory接口。

而ThreadPoolExecutor的子类ScheduledThreadPoolExecutor则与定期执行命令相关的线程池。当需要多个辅助线程时，或者要求 ThreadPoolExecutor 具有额外的灵活性或功能时，此类要优于 Timer，因为Timer是单线程的。

一旦启用已延迟的任务就执行它，但是有关何时启用，启用后何时执行则没有任何实时保证。按照提交的先进先出 (FIFO) 顺序来启用那些被安排在同一执行时间的任务。 

虽然此类继承自 ThreadPoolExecutor，但是几个继承的调整方法对此类并无作用。特别是，因为它作为一个使用 corePoolSize 线程和一个无界队列的固定大小的池，所以调整 maximumPoolSize 没有什么效果。

所以ScheduledThreadPoolExecutor类的构造方法的参数少了很多。有四个构造方法如下。

```java
// 使用给定核心池大小创建一个新 ScheduledThreadPoolExecutor。
ScheduledThreadPoolExecutor(int corePoolSize) 
// 使用给定初始参数创建一个新 ScheduledThreadPoolExecutor。
ScheduledThreadPoolExecutor(int corePoolSize, RejectedExecutionHandler handler) 
// 使用给定的初始参数创建一个新 ScheduledThreadPoolExecutor。
ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory) 
// 使用给定初始参数创建一个新 ScheduledThreadPoolExecutor。
ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory, RejectedExecutionHandler handler) 
```

参数与其父类ThreadPoolExecutor表达的意思是一样的，就不重复说明了。这里也以参数最少的构造方法示例ScheduledThreadPoolExecutor类的用法。

前面的例子使用的是继承Thread类的线程使用线程池，方法是调用线程池的execute(Runnable command)方法来提交任务给线程池的。下面的例子使用线程池的submit(Callable<T> task)方法来提交实现Callable接口的线程任务给线程池，该方法有多种重载形式，这里使用其中一种作为示例。

代码示例如下：

```java
public class ScheduledThreadPoolExecutorTest<V> implements Callable<V>{
	private V v;
	public ScheduledThreadPoolExecutorTest(V v){
		this.v = v;
	}
	@Override
	public V call() throws Exception {
		System.out.println("线程执行中," + v);
		return v;
	}
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(5,
				Executors.defaultThreadFactory(), 
				new ThreadPoolExecutor.AbortPolicy());
		for (int i = 0; i < 10; i++) {
			Future<Integer> result = executor.submit(new ScheduledThreadPoolExecutorTest<Integer>(i));
			System.out.println("获取Callable线程返回结果：" + result.get());
		}
		executor.shutdown();
	}
}
```

代码执行结果为

![ScheduledThreadPoolExecutor.jpg](/images/Core_Java/JavaThread4/ScheduledThreadPoolExecutor.jpg)

上面的代码其实只是用到了从ThreadPoolExecutor类继承来的方法，并没有使用ScheduledThreadPoolExecutor类我称之为“多线程定时任务线程池”的功能。要使ScheduledThreadPoolExecutor线程池能够定时的执行任务，需要使用ScheduledThreadPoolExecutor类的schedule()方法或scheduleAtFixedRate()方法。它们的方法签名如下：
<V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit) 
创建并执行在给定延迟后启用的 ScheduledFuture。
 ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) 
创建并执行在给定延迟后启用的一次性操作。
ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)
创建并执行一个在给定初始延迟后首次启用的定期操作，后续操作具有给定的周期；也就是将在 initialDelay 后开始执行，然后在 initialDelay+period 后执行，接着在 initialDelay + 2 * period 后执行，依此类推。
ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay,TimeUnit unit) 
创建并执行一个在给定初始延迟后首次启用的定期操作，随后，在每一次执行终止和下一次执行开始之间都存在给定的延迟。

这里简单举个定时任务的例子，大部分代码如上个例子没变，只需要改几行代码即可。

代码示例如下

```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
		ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(5,
				Executors.defaultThreadFactory(), 
				new ThreadPoolExecutor.AbortPolicy());
		for (int i = 0; i < 10; i++) {
			ScheduledFuture<Integer> result = executor.schedule(new ScheduledThreadPoolExecutorTest<Integer>(i), 3, TimeUnit.SECONDS);
			System.out.println("获取Callable线程返回结果：" + result.get());
		}
		executor.shutdown();
	}
	
```

而代码的执行结果也没有变化，唯一不同的是，每个线程执行前都会等待3秒钟。也就是相当于把线程定时在3秒后执行而不是立刻执行。

对于类图中的ForkJoinPool这个类，这是Java7新增的类，Java8对这个类又增加了几个方法。ForkJoinPool用于将一个大任务拆分成多个小任务并行处理，处理完后再将结果合并得到最终结果。目前而言，这个类几乎不会用到，只有当对性能要求很高的程序才会使用它进行优化。

以上便是手动构造线程池的一般方法，从以上手动构造一个线程池的代码中可以看出，参数多且复杂，

**幸运的是，从Java5开始，提供了Executor并发框架来简化线程池的创建**
	
Executor框架能够创建两类的线程池，分别对应前面介绍的ThreadPoolExecutor类和它的子类ScheduledThreadPoolExecutor，Java提供了Executors工厂类，提供了一些静态工厂方法来构造常用的线程池，并且参数少了，代码意图更加清晰。

1、三种ThreadPoolExecutor类型的线程池	
（1）可重用固定线程数的线程池FixedThreadPool
newFixedThreadPool(int nThreads) 
创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。
newFixedThreadPool(int nThreads, ThreadFactory threadFactory)
创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程，在需要时使用提供的 ThreadFactory 创建新线程。
（2）只有单线程的线程池SingleThreadExecutor
newSingleThreadExecutor() 
创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。
newSingleThreadExecutor(ThreadFactory threadFactory) 
创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程，并在需要时使用提供的 ThreadFactory 创建新线程。
（3）具有缓存功能的线程池CachedThreadPool
newCachedThreadPool() 
创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。
newCachedThreadPool(ThreadFactory threadFactory) 
创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们，并在需要时使用提供的 ThreadFactory 创建新线程。

2、两种ScheduledThreadPoolExecutor类型的线程池
（1）可延迟执行线程任务的线程池ScheduledThreadPoolExecutor
newScheduledThreadPool(int corePoolSize) 
创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。
newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory) 
创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。
（2）单线程可延迟执行线程任务的线程池SingleThreadScheduledExecutor
newSingleThreadScheduledExecutor() 
创建一个单线程执行程序，它可安排在给定延迟后运行命令或者定期地执行。
newSingleThreadScheduledExecutor(ThreadFactory threadFactory) 
创建一个单线程执行程序，它可安排在给定延迟后运行命令或者定期地执行。
在使用Executors的静态工厂方法得到线程池后，就可以使用之前使用线程池的方法来使用线程池了。
