---
layout: post
title:  "Java多线程（二）--线程的生命周期"
date:   2016-09-04 22:28:54
categories: Core_Java
tags: Java 多线程
---

* content
{:toc}

本文是Java多线程第二篇，关于线程声明周期的几个状态和状态之间的转换




### Java线程的生命周期

#### 新建状态

使用前面介绍的三种创建线程的方式创建出线程对象后，该线程对象就处于新生状态。

#### 就绪状态

通过调用start()方法进入就绪状态（runnable）。处于就绪状态的线程已经具备了运行条件，一旦线程分配给了CPU，线程就进入运行状态并自动调用自己的run()方法。

#### 运行状态

当线程开始执行线程代码块run()时我们认为线程进入了运行状态。

运行状态的线程很是复杂，它可以在不同条件下转换为阻塞状态、就绪状态和死亡状态。

#### 阻塞状态

运行状态的线程如果执行了sleep()方法，或等待I/O设备等资源，将会暂时停止运行，进入阻塞状态。 

“处于阻塞状态的线程无法进入就绪状态队列中。只有当引起阻塞的原因消除时，如睡眠时间已到，或等待的I/O设备空闲下来，线程便转入就绪状态，重新到就绪队列中排队等待，被系统选中后从原来停止的位置开始继续运行。”

#### 死亡状态

所谓死亡状态就是当一个线程的run()方法体执行完，或者被强制性地终止。线程一旦进入死亡状态，就说明该线程的生命周期已经结束了。

### 控制Java线程的状态

Java线程的状态控制其实主要是对运行状态的控制，通过不同的方法进入不同的状态，而其它状态进入运行状态是有CPU来控制的。

下面的线程状态转换图可以很清晰地说明这一点。（图片来源于网络）
 
![ThreadStatus.png](/images/Core_Java/JavaThread2/ThreadStatus.png)
 
#### 调用sleep()方法使线程睡眠

由上图中可以看出，运行状态的线程转换成阻塞状态的线程，其中一种方式就是调用sleep()方法使线程睡眠。当线程睡眠的时间到了之后，线程又进入了就绪状态。

sleep()方法是Thread的静态方法，作用是让当前线程睡眠指定的时间。

在Java8中sleep()的方法声明有以下两种。

```java
public static native void sleep(long millis) throws InterruptedException
public static void sleep(long millis, int nanos) throws InterruptedException
```
	
当调用sleep方法时，当前线程会进入睡眠，系统会执行其它线程。这里使用继承Thread类创建线程的例子更改一下作为示例.

下面是代码示例：

```java
public class ThreadTest extends Thread{
	@Override
	public void run() {
		System.out.println("线程执行中");
	}
	public static void main(String[] args) throws InterruptedException {
		Thread thread = new ThreadTest();
		System.out.println("start()方法调用前");
		thread.start();
		Thread.sleep(1);
		System.out.println("start()方法调用后");
	}
}
```

代码执行结果为
 
![ThreadStatus.png](/images/Core_Java/JavaThread2/ThreadSleep.jpg)

从运行结果中可以看出，在线程对象调用start()方法进入就绪状态之后立刻调用sleep()方法使主线程沉睡了1毫秒，系统将当前执行线程切换为ThreadTest类的实例，原本“start()方法调用后”在“线程执行中”之后打印的行为变成了“线程执行中”先于“start()方法调用后”。这就是sleep()方法的作用，是当前执行线程睡眠一段时间。

#### 线程的优先级

多个线程的执行是有优先级的，优先级高的线程一般比优先级低的线程先执行。在Thread类中，有三个线程优先级的常量，分别是

```java
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
```

而main线程的优先级是NORM_PRIORITY。从这三个常量中可以看出，优先级的范围是否1~10之间，然而优先级并不是绝对的，只是优先级越高的线程先执行的可能性越高，也有可能是优先级低的线程比优先级高的线程先执行。所以线程的优先级其实是不靠谱的，只是程序建议CPU先执行优先级高的线程而已，这可能跟当前CPU是多核的有点关系。若是单个单核的CPU，则线程的切换时按照其优先级进行的，只是目前我们使用的计算机基本都是多核CPU为主了。以下代码示例可以证明这一点， 相同的代码多执行几次结果是不同的。

```java
public class PriorityTest extends Thread{
	@Override
	public void run() {
		for (int i = 0; i < 100; i++) {
			System.out.println(this.getName()+"的优先级为："+this.getPriority()+", 当前执行次数为："+(i+1));
		}
	}
	public static void main(String[] args) throws InterruptedException {
		Thread highLevelThread = new PriorityTest();
		highLevelThread.setName("高优先级线程");
		highLevelThread.setPriority(MAX_PRIORITY);
		Thread lowerLevelThread = new PriorityTest();
		lowerLevelThread.setName("低优先级线程");
		lowerLevelThread.setPriority(MIN_PRIORITY);
		System.out.println("main线程的优先级为："+Thread.currentThread().getPriority());
		highLevelThread.start();
		lowerLevelThread.start();
		Thread.sleep(1);
		for (int i = 0; i < 100; i++) {
			System.out.println("main线程的优先级为："+Thread.currentThread().getPriority()+", 当前执行次数为："+(i+1));
		}
	}
}
```

代码第一种执行结果为：
 
![ThreadPriority1.jpg](/images/Core_Java/JavaThread2/ThreadPriority1.jpg)

代码第二种执行结果为
 
![ThreadPriority2.jpg](/images/Core_Java/JavaThread2/ThreadPriority2.jpg)

#### 调用yield()方法使线程让步

yield()方法是Thread类提供的一个让当前线程让步的方法。与sleep()方法相反的是，yield()方法调用时当前线程并不会阻塞。当调用yield()方法时，当前线程重新进入就绪状态，系统可能切换到别的线程，也可能继续执行当前线程，切换到哪个线程跟线程的优先级有关，跟线程的优先级一样，并不是优先级越高的就一定会先执行，存在一定的偶然性，这个计算机的CPU有关。yield()方法是Thread的无参静态方法，以下是yield()方法使用的示例代码，其运行结果也存在两种可能性：

```java
public class YieldMethodTest extends Thread{
	@Override
	public void run() {
		System.out.println("线程执行中");
	}
	public static void main(String[] args) {
		Thread thread = new YieldMethodTest();
		thread.setPriority(MAX_PRIORITY);
		thread.start();
		System.out.println("yield()方法调用前");
		Thread.yield();
		System.out.println("yield()方法调用后");
	}
}
```

代码第一种执行结果为

![YeildMethod1.jpg](/images/Core_Java/JavaThread2/YeildMethod1.jpg)

代码第二种执行结果为
 
![YeildMethod1.jpg](/images/Core_Java/JavaThread2/YeildMethod1.jpg)

#### 调用join()方法使线程合并

join()方法是Thread类提供的一个让当前线程等待另一个线程执行完成的方法。和sleep()方法一样，join()方法调用时当前线程都将被阻塞。

当调用join()方法的线程对象的线程执行体run()方法执行完毕时，之前被阻塞的线程又将重新被调用。

多线程的执行是异步的，线程的执行顺序是无法保证的，通过join方法，则可以保证当前线程在另一个线程执行之后才继续执行。

join()方法有几种重载形式，一种是无参的join()方法，使join()方法加进来的线程执行完毕之后才回到原来的线程。另一种带参数的join()方法则是在等待指定的时间之后，原来的线程不再等待join()方法加入进来的线程执行完毕则开始执行。

下面是代码示例：

```java
public class JoinMethodTest extends Thread{
	@Override
	public void run() {
		for (int i = 0; i < 5; i++) {
			System.out.println("线程执行中，循环计数："+i);
		}
	}
	public static void main(String[] args) throws InterruptedException {
		Thread thread = new JoinMethodTest();
		System.out.println("start()方法调用前");
		thread.start();
		System.out.println("start()方法调用后");
		System.out.println("join()方法调用前");
		thread.join();
		System.out.println("join()方法调用后");
	}
}
```

代码执行结果为
 
![JoinMethod.jpg](/images/Core_Java/JavaThread2/JoinMethod.jpg)

从结果中可以看出，无参的join()方法确实使原来的线程等待join()线程执行完毕之后才继续执行。join()方法就是起到使当前线程等待的效果。

#### 让线程运行在后台

后台线程又叫“守护线程”，是专门为其他线程提供服务的，通过Thread类的setDaemon(true)方法可以将指定的线程运行在后台。

后台线程是为其它线程服务的，因此，当所有线程运行完毕时，后台线程也会死亡。

下面是代码示例：

```java
public class DaemonTest extends Thread{
	@Override
	public void run() {
		try {
			if(Thread.currentThread().isDaemon()){
				Thread.sleep(1);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("线程执行中，这是"+this.getName());
	}
	public static void main(String[] args) {
		Thread normalThread = new DaemonTest();
		normalThread.setName("普通线程");
		Thread daemonThread = new DaemonTest();
		daemonThread.setName("后台线程");
		daemonThread.setDaemon(true);
		normalThread.start();
		daemonThread.start();
		System.out.println("这是main线程");
	}
}
```

代码执行结果为
 
![DaemonThread.jpg](/images/Core_Java/JavaThread2/DaemonThread.jpg)

从运行结果中可以看到，后台线程的打印语句没有执行，因为普通线程和main线程已经执行完毕了。线程执行体run()方法中通过isDaemon()方法做了一个判断，若是后台线程这调用sleep()方法沉睡1毫秒，将线程执行的机会让给其它线程。若是没有调用sleep()方法，则有可能看到后台线程打印语句的输出结果，因为有可能后台线程先于普通线程执行。

#### 结束一个线程

线程是有生命周期的，当一个线程方法体执行完毕时，该线程就死亡了，也就意味着这一个线程结束了。

然而也有例外的情况，一种是线程方法体执行时抛出了未捕获的异常或错误，另一种则是调用Thread类的stop()方法主动结束一个线程。

当然这两种结束线程的方法都是有问题的，前者是代码有bug或者硬件或操作系统出现了故障，后者则容易导致死锁的问题。

