---
layout: post
title:  "Java多线程（三）--线程锁"
date:   2016-09-05 22:28:54
categories: Core_Java
tags: Java 多线程
---

* content
{:toc}

在并发的情况下，多个线程一起访问同一个资源时会出现线程同步的问题。

很多程序的bug都是由于线程不同步而造成的，解决线程不同步的方法最常用的是使用Java的锁机制对代码进行加锁，而加锁的房补不同和加锁的范围不同又分几种情况。除此之外还可以使用特殊域变量(volatile)实现线程同步。

那么在什么时候需要用到线程同步呢？前面说了，多个线程访问同一个资源的时候就需要对代码进行同步操作了。

比如对文件的读写操作，多个线程同时对某个变量进行修改，对数据库的增删改查的访问等等。




#### 使用synchronized关键字进行加锁

在Java里面，每个类实例对应一把锁，在使用synchronized对同代码块或者方法进行加锁之后，多个线程同时访问加锁代码时，必须要先获取该实例对象的锁才可以执行这一段代码，并且在这段代码执行过程中，其它线程无法获取这个对象的锁，也就无法进入这段同步代码块或者同步方法中了，确保了并发时多线程的同步问题，这就是Java对象锁的排他性。

**使用synchronized关键字对代码块进行加锁**

在并发编程中，使用synchronized关键字对代码加锁是很消耗性能的一件事，而有时候我们只需要对一部分代码进行同步操作就可以了，这样相对来说程序性能会好一点，这样的操作我们称为同步代码块。

同步代码块的语法格式如下：

```java
synchronized (obj) {
	// 在大括号内编写的代码就叫同步代码块
}
```

下面使用对同一个变量同时被访问的场景进行模拟，代码示例如下：

```java
public class SynchronizedBlockTest {
	public Double a;
	public void add(Double b){
		//synchronized (a) {
			a = a + b;
				b = a + b;
			a = a + b;
		//}
	}
}
```

这里使用的是简单的加法操作，但在多线程同时访问这几行加法的代码时，变量a的结果可能就变得不准确了，有可能的情况是，变量a的值被另一个线程修改了。

当把注释的synchronized取消后，就可以保证当一个线程对a的值进行计算时，另一个线程无法进入synchronized括号内的同不代码块，这就保证了这几行加法代码计算的值是正确的。


**使用synchronized关键字对实例方法进行加锁**

使用synchronized关键字对实例方法进行加锁，我们称为同步方法，对实例方法加锁的性能一般比只对一小部分代码块性能稍好大一些。

对实例方法加锁只需要在方法签名上加一个synchronized修饰符即可。当一个类中有多个被synchronized关键字修饰的同步方法时，并且同一时间有一个线程进入其中一个同步方法时，别的线程就只能这个线程将这个同步方法执行完毕，才能进入执行这个同步方法或者该类下别的实例同步方法。

代码示例如下

```java
public class SynchronizedMethodTest {
	public int sum;
	public synchronized void add(int aValue){
		sum = sum + aValue;
	}
	public synchronized void sub(int aValue){
		sum = sum - aValue;
	}
}
```


**使用synchronized关键字对类方法进行加锁**

前面说到每个对象都有一把锁，同一时间只能有一个线程获取这把锁，然后执行被synchronized关键字修饰的一段代码或方法。

而每个类也有这样一把锁，对于每个类里面的静态代码块或者静态方法使用synchronized关键字进行加锁，称之为静态同步代码块或者静态同步方法。

同样的，同一时间内，也只能有一个线程执行一个类里面的静态同步代码块或者静态同步方法，并且当静态同步代码块或静态同步方法被执行时，这个类的同步锁被占用，直到静态同步代码块或静态同步方法执行完毕，同步锁被解除时，其它线程才能获得同步锁，执行静态同步代码或者静态同步方法。其实就是比实例代码块和实例方法前面多了个static关键字。

代码示例如下：（以静态同步方法为例）

```java
public class SynchronizedMethodTest {
	public static int sum;
	public static synchronized void add(int aValue){
		sum = sum + aValue;
	}
	public static synchronized void sub(int aValue){
		sum = sum - aValue;
	}
}
```

#### 使用Lock和ReadWriteLock接口的实现类手动加锁

Lock和ReadWriteLock及其实现类的类图如下
 
![LockAndConditionClassDiagram.jpg](/images/Core_Java/JavaThread3/LockAndConditionClassDiagram.jpg)

除了使用synchronized加锁的方式外，JDK还提供了Lock和ReadWriteLock两个接口来实现手动加锁。

实际编码中我们使用Lock的实现类ReentrantLock（可重入锁）和ReadWriteLock的实现类ReentrantReadWriteLock（可重入读写锁）来进行编码操作。

手动加锁的方式更加灵活，提供了很多对锁进行操作的方法，比如锁对象的各种属性，是否被某个线程持有该锁等等。

ReentrantLock和ReentrantReadWriteLock两个类唯一的区别是当读取某个资源时，ReentrantLock不允许多个线程同时读取，即使是需要多个线程同时读取的资源也不能被其它线程读取，而ReentrantReadWriteLock的ReadLock是允许多线程获取的，WriteLock只能单个线程获取。当然，若对同步锁没有特殊的需求，我们使用synchronized关键字修饰的同步代码块和同步方法就可以了，效果是一样的。

可重入锁ReentrantLock使用的语法格式为：

```java
class X {
   private final ReentrantLock lock = new ReentrantLock();
   public void m() { 
     lock.lock();
     try {
       // 这里是需要加同步锁的代码
     } finally {
       lock.unlock()
     }
   }
}
```

可重入读写锁ReentrantReadWriteLock使用的语法格式为：

```java
public class X {
	private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
	public void readLockTest(){
		rwLock.readLock().lock();
		try {
			// 需要加读取锁的代码
		}
		finally {
			rwLock.readLock().unlock();
		}
	}
	public void writeLockTest(){
		rwLock.writeLock().lock();
		try {
			// 需要加写入锁的代码
		}
		finally {
			rwLock.writeLock().unlock();
		}
	}
}
```


#### 使用特殊域变量（volatile）实现线程同步

volatile关键字为域变量的访问提供了一种免锁机制，使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新，因此每次使用该域就要重新计算，而不是使用寄存器中的值，volatile不会提供任何原子操作，它也不能用来修饰final类型的变量。

还是前面的加法示例，对变量sum加了volatile关键字修饰，同时synchronized就没有必要存在了。

代码示例如下：

```java
public class VolatileTest {
	public volatile int sum;
	public void add(int aValue){
		sum = sum + aValue;
	}
	public void sub(int aValue){
		sum = sum - aValue;
	}
}
```

#### 程序出现死锁

在使用锁机制实现线程同步时，不管是使用synchronized关键还是使用Lock手动加锁的形式，如果出现了两个线程互相等待对方释放同步锁时，我们就说程序出现了死锁。死锁在多线程编程中还是很容易出现的现象。

代码示例如下：

```java
public class DeadLockTest{
	final static ClassA a = new ClassA();
	final static ClassB b = new ClassB();
	public static void main(String[] args) {
		// 此处使用了Java8的lambda表达式简化线程代码
		new Thread( () -> a.firstMethod(b) ).start();
		new Thread( () -> b.firstMethod(a) ).start();
	}
}
class ClassA {
	public synchronized void firstMethod(ClassB b){
		System.out.println("成功调用了ClassA第一个同步方法");
		try {
			Thread.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("即将调用ClassB的第二个方法");
		b.secondMethod();
	}
	public synchronized void secondMethod(){
		System.out.println("成功调用了ClassA第二个同步方法");
	}
}
class ClassB {
	public synchronized void firstMethod(ClassA a){
		System.out.println("成功调用了ClassB第一个同步方法");
		try {
			Thread.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("即将调用ClassA的第二个方法");
		a.secondMethod();
	}
	public synchronized void secondMethod(){
		System.out.println("成功调用了ClassB第二个同步方法");
	}
}
```

代码执行结果为
 
![DeadLock.jpg](/images/Core_Java/JavaThread3/DeadLock.jpg)

从代码执行结果中可以看出，ClassA和ClassB两个类的secondMethod()方法都没有执行，两个线程执行到调用secondMethod()方法时，调用secondMethod()方法的对象的同步锁还被另一个线程占用着，然后两个线程就相互阻塞了，线程没有执行完，也没有什么报错。

在多线程的实际编程中，这样的死锁是要小心避免的。

### 线程通信

当多个线程在系统中运行时，线程之间的切换具有一定的随机性。程序一般无法准确的在多个线程之间进行切换，但是Java也提供了一些机制来使多个线程之间能够协调的运行，这就是线程通信。

根据实现线程同步的方式的同步，线程通信的方式也有所不同。

#### 使用Object类的wait()、notify()和notifyAll()方法

使用synchronized关键字修饰的同步代码块和同步方法的线程之间的通信可以使用Object类的wait()、notify()和notifyAll()方法进行通信。

wait()方法有三个重载的形式，Java6的文档上是这么写的：

wait()方法，在其他线程调用此对象的 notify() 方法或 notifyAll() 方法前，导致当前线程等待。

wait(long timeout)方法，在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量前，导致当前线程等待。

wait(long timeout, int nanos)方法，在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量前，导致当前线程等待。

而notify()方法和notifyAll()方法则是唤醒在此对象监视器上等待的线程，不同的是notify()方法是唤醒单个线程，而notifyAll()方法则是唤醒所有线程。

对于同步代码块而言，this就是同步监视器。

对于同步方法而言，synchronized后括号里面的对象就是同步监视器。

假设有这样一个场景：有一个收纳盒，里面只能收纳一个物品，所有就有存放物品和取出物品两个线程，存放物品之前必须先将之前的物品取出来，取出物品之前收纳盒里面必须存放了物品。

编写成代码则如下所示：

```java
public class ThreadCommunicationTest{
	private boolean depositFlag = false;
	public synchronized void deposit(){
		if(depositFlag){
			try {
				wait();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		} else {
			depositFlag = true;
			notifyAll();
		}
	}
	public synchronized void take(){
		if(!depositFlag){
			try {
				wait();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		} else {
			depositFlag = false;
			notifyAll();
		}
	}
	public static void main(String[] args) {
		ThreadCommunicationTest tct = new ThreadCommunicationTest();
		Thread depositThread = new DepositThread(tct);
		Thread takeThread = new TakeThread(tct);
		depositThread.start();
		takeThread.start();
	}
}
class DepositThread extends Thread {
	private ThreadCommunicationTest tct;
	public DepositThread(ThreadCommunicationTest tct){
		this.tct = tct;
	}
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println("存放物品循环计数："+(i+1)+",");
			tct.deposit();
		}
	}
}
class TakeThread extends Thread {
	private ThreadCommunicationTest tct;
	public TakeThread(ThreadCommunicationTest tct){
		this.tct = tct;
	}
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println("取出物品循环计数："+(i+1)+",");
			tct.take();
		}
	}
}
```

代码执行结果为：（如图2.14所示）
 
图 2.14

#### 使用Condition实现线程通信

使用Lock和ReadWriteLock接口的实现类手动加锁的线程通信不像synchronized关键字同步的线程存在隐式的同步监视器，所以也就不能使用wait()、notify()和notifyAll()方法来进行通信了。

但是，就跟前面介绍的时候说的，Lock手动加锁和synchronized关键字加锁有相似之处。使用Condition实现线程通信和使用wait()、notify()和notifyAll()方法来进行通信也是有相似之处的。

Lock接口声明了一个newCondition()方法，这个方法可以用来获得一个Condition对象，这个对象就相当于使用synchronized加锁时的同步监视器，里面有三个类似wait()、notify()和notifyAll()的方法，分别是await()、signal()和signalAll()方法，功能也是一样的。

从Java6的文档上可以看到这样的声明：

await()方法，造成当前线程在接到信号或被中断之前一直处于等待状态。

signal()方法，唤醒一个等待线程。

signalAll()方法， 唤醒所有等待线程。

还是那个收纳盒的例子，将synchronized关键字修饰的同步方法改成使用Lock手动加锁进行线程同步，相应的wait()、notify()和notifyAll()方法则换成await()、signal()和signalAll()方法，而存放物品和取出物品的线程类不变，其代码运行结果是一样的。

代码示例如下：

```java
public class ThreadCommunicationTest{
	private boolean depositFlag = false;
	private final ReentrantLock lock = new ReentrantLock();
	private final Condition condition = lock.newCondition();
	public void deposit(){
		lock.lock();
		try {
			if(depositFlag){
				try {
					condition.await();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			} else {
				depositFlag = true;
				condition.signalAll();
			}
		}
		finally {
			lock.unlock();
		}
	}
	public void take(){
		lock.lock();
		try {
			if(!depositFlag){
				try {
					condition.await();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			} else {
				depositFlag = false;
				condition.signalAll();
			}
		}
		finally {
			lock.unlock();
		}
	}
	public static void main(String[] args) {
		ThreadCommunicationTest tct = new ThreadCommunicationTest();
		Thread depositThread = new DepositThread(tct);
		Thread takeThread = new TakeThread(tct);
		depositThread.start();
		takeThread.start();
	}
}
```