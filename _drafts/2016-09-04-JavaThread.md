---
layout: post
title:  "Java多线程（一）--线程是怎么创建的"
date:   2016-09-04 22:28:54
categories: Core_Java
tags: Java 多线程
---

* content
{:toc}

一、多线程是什么及其意义

（一）什么是多线程
操作系统同时运行着多个任务，每一个任务就是一段内存中运行的程序，也叫进程。而这个运行的程序又可能包含多个同时执行的程序片段，这每一个执行程序片段的过程就叫线程。
多线程就是程序利用进程空闲时间不停地在各个线程之间进行切换，已达到多个程序片段同时进行的效果的过程，也就是多个线程同时执行。
（二）多线程有什么意义
线程比进程的隔离程度较小，多个线程之间共享内存空间和资源。多线程共享资源，线程间的切换花费的时间代价小，在并发时的性能要比多进程高。利用多个线程之间共享资源的特性，很容易实现线程之间的通信。
（三）Java多线程有什么优势
Java内置的多线程支持封装了底层操作系统的调度细节，简化了多线程的编程。
由于Java编程语言是跨平台的，所以在Java中使用多线程最大的优势就是“一次编写，随处运行”。
同时，Java提供锁机制，保证了Java多线程在并发下处理资源同步的问题。

 
二、Java多线程的内容

（一）创建和启动一个Java线程的三种方式
1、以继承Thread类的方式创建线程
Thread类里面有个run()方法，run()方法包装线程执行代码。然而启动线程并不是调用run()方法启动的，而是调用Thread里的start()方法。直接调用run()方法的话并不会启动一个线程，而是直接执行了run()方法体里面的代码片段，达不到创建线程的方法，因此通过继承Thread类的方式创建线程的步骤如下：
（1）创建Thread类的子类并重写run()方法
（2）实例化第(1)步中Thread类的子类为线程对象
（3）调用线程对象的start()方法
下面是代码示例：
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
代码执行结果为：（如图2.1所示）
 
图 2.1
从执行结果中可以看出，线程的执行是异步的，并不是顺序执行的。这就是以继承Thread类的方式创建线程。
2、以实现Runnable接口的方式创建线程
	Runnable接口只有一个run()方法，Thread类其实也实现了Runnable接口。同样的，也是这个run()方法包装线程执行代码。“从Java8开始，Runnable接口使用了@FunctionalInterface修饰，也就是说，Runnable接口是函数式接口，可使用lambda表达式来创建Runnable对象。”○1 我们可以看到Java8里面Runnable接口的声明如下：
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
	实现Runnable接口创建线程的步骤如下：
（1）实现Runnable接口并重写run()方法
（2）实例化第(1)步中实现Runnable实现类为线程对象
（3）使用Thread类的构造方法来包装Runnable实现类对象
下面是代码示例：
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
代码执行结果为：（如图2.2所示）
 
图 2.2
	前面说到Runnable是函数式接口，可以使用lambda表达式简化代码，以下代码执行结果同上。
public class RunnableTest1 {
	public static void main(String[] args) {
		System.out.println("start()方法调用前");
		new Thread( () -> System.out.println("线程执行中") ).start();
		System.out.println("start()方法调用后");
	}
}
3、以实现Callable接口结合Future接口的方式创建线程
	前面两种创建线程的方式都是无法从线程中获取返回值的，而接下来要说的这种方式则可以获得返回值，就是实现Callable接口的方式。我们先来看看Java8里面Callable接口的声明。
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
我们可以看到，Callable接口跟Runnable接口一样被@FunctionalInterface修饰，是一个函数式接口。Callable接口只有一个方法call()，有一个返回值V，这是一个泛型的返回值，由实现该接口的类实例化时指定返回值的类型。
再来看看Future接口的声明
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
看到Future接口里面声明的方法我们就可以知道，这个类是用来取消线程和判断线程是否完成以及获取线程返回结果的。
Future是接口，不能直接使用，而整个JDk中只有FutureTask跟Future有间接联系。FutureTask先是实现了RunnableFuture接口，而RunnableFuture接口扩展了Future接口，同时行业扩展了Runnable接口，所以FutureTask也就实现了Future接口。这两个接口在Java8中的声明分别如下：
public class FutureTask<V> implements RunnableFuture<V> 
public interface RunnableFuture<V> extends Runnable, Future<V>
FutureTask有个构造方法为public FutureTask(Callable<V> callable)，所以就可以得出以下创建线程的步骤：
（1）实现Callable接口并重写call()方法
（2）实例化第(1)步中实现Callable实现类为线程对象
（3）使用FutureTask类的构造方法来包装Runnable实现类对象
（4）使用Thread类包装FutureTask的实例对象
下面是代码示例：
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
代码执行结果为：（如图2.3所示）
 
图 2.3
根据上面的分析可以看出，Callable接口与Runnable接口差别并不大，区别是Callable接口可以有返回值，并且可以抛出异常罢了。
（二）Java线程的生命周期
1、新建状态
使用前面介绍的三种创建线程的方式创建出线程对象后，该线程对象就处于新生状态。
2、就绪状态
通过调用start()方法进入就绪状态（runnable）。处于就绪状态的线程已经具备了运行条件，一旦线程分配给了CPU，线程就进入运行状态并自动调用自己的run()方法。
3、运行状态
当线程开始执行线程代码块run()时我们认为线程进入了运行状态。
运行状态的线程很是复杂，它可以在不同条件下转换为阻塞状态、就绪状态和死亡状态。
4、阻塞状态
运行状态的线程如果执行了sleep()方法，或等待I/O设备等资源，将会暂时停止运行，进入阻塞状态。 
“处于阻塞状态的线程无法进入就绪状态队列中。只有当引起阻塞的原因消除时，如睡眠时间已到，或等待的I/O设备空闲下来，线程便转入就绪状态，重新到就绪队列中排队等待，被系统选中后从原来停止的位置开始继续运行。”
5、死亡状态
所谓死亡状态就是当一个线程的run()方法体执行完，或者被强制性地终止。线程一旦进入死亡状态，就说明该线程的生命周期已经结束了。
（三）控制Java线程的状态
Java线程的状态控制其实主要是对运行状态的控制，通过不同的方法进入不同的状态，而其它状态进入运行状态是有CPU来控制的。下面的线程状态转换图（图2.4○2）可以很清晰地说明这一点。
 
图 2.4
1、调用sleep()方法使线程睡眠
由上图中可以看出，运行状态的线程转换成阻塞状态的线程，其中一种方式就是调用sleep()方法使线程睡眠。当线程睡眠的时间到了之后，线程又进入了就绪状态。
sleep()方法是Thread的静态方法，作用是让当前线程睡眠指定的时间。在Java8中sleep()的方法声明有以下两种。
public static native void sleep(long millis) throws InterruptedException
public static void sleep(long millis, int nanos) throws InterruptedException
	当调用sleep方法时，当前线程会进入睡眠，系统会执行其它线程。这里使用继承Thread类创建线程的例子更改一下作为示例，下面是代码示例：
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
代码执行结果为：（如图2.5所示）
 
图 2.5
从运行结果中可以看出，在线程对象调用start()方法进入就绪状态之后立刻调用sleep()方法使主线程沉睡了1毫秒，系统将当前执行线程切换为ThreadTest类的实例，原本“start()方法调用后”在“线程执行中”之后打印的行为变成了“线程执行中”先于“start()方法调用后”。这就是sleep()方法的作用，是当前执行线程睡眠一段时间。
2、线程的优先级
多个线程的执行是有优先级的，优先级高的线程一般比优先级低的线程先执行。在Thread类中，有三个线程优先级的常量，分别是
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
而main线程的优先级是NORM_PRIORITY。从这三个常量中可以看出，优先级的范围是否1~10之间，然而优先级并不是绝对的，只是优先级越高的线程先执行的可能性越高，也有可能是优先级低的线程比优先级高的线程先执行。所以线程的优先级其实是不靠谱的，只是程序建议CPU先执行优先级高的线程而已，这可能跟当前CPU是多核的有点关系。若是单个单核的CPU，则线程的切换时按照其优先级进行的，只是目前我们使用的计算机基本都是多核CPU为主了。以下代码示例可以证明这一点， 相同的代码多执行几次结果是不同的。
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
代码第一种执行结果为：（如图2.6所示）
 
图 2.6
代码第二种执行结果为：（如图2.7所示）
 
图 2.7
3、调用yield()方法使线程让步
yield()方法是Thread类提供的一个让当前线程让步的方法。与sleep()方法相反的是，yield()方法调用时当前线程并不会阻塞。当调用yield()方法时，当前线程重新进入就绪状态，系统可能切换到别的线程，也可能继续执行当前线程，切换到哪个线程跟线程的优先级有关，跟线程的优先级一样，并不是优先级越高的就一定会先执行，存在一定的偶然性，这个计算机的CPU有关。yield()方法是Thread的无参静态方法，以下是yield()方法使用的示例代码，其运行结果也存在两种可能性：
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
代码第一种执行结果为：（如图2.8所示）
 
图 2.8
代码第二种执行结果为：（如图2.9所示）
 
图 2.9
4、调用join()方法使线程合并
join()方法是Thread类提供的一个让当前线程等待另一个线程执行完成的方法。和sleep()方法一样，join()方法调用时当前线程都将被阻塞。当调用join()方法的线程对象的线程执行体run()方法执行完毕时，之前被阻塞的线程又将重新被调用。多线程的执行是异步的，线程的执行顺序是无法保证的，通过join方法，则可以保证当前线程在另一个线程执行之后才继续执行。join()方法有几种重载形式，一种是无参的join()方法，使join()方法加进来的线程执行完毕之后才回到原来的线程。另一种带参数的join()方法则是在等待指定的时间之后，原来的线程不再等待join()方法加入进来的线程执行完毕则开始执行。
下面是代码示例：
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
代码执行结果为：（如图2.10所示）
 
图 2.10
从结果中可以看出，无参的join()方法确实使原来的线程等待join()线程执行完毕之后才继续执行。join()方法就是起到使当前线程等待的效果。
5、让线程运行在后台
后台线程又叫“守护线程”，是专门为其他线程提供服务的，通过Thread类的setDaemon(true)方法可以将指定的线程运行在后台。后台线程是为其它线程服务的，因此，当所有线程运行完毕时，后台线程也会死亡。
下面是代码示例：
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
代码执行结果为：（如图2.11所示）
 
图 2.11
	从运行结果中可以看到，后台线程的打印语句没有执行，因为普通线程和main线程已经执行完毕了。线程执行体run()方法中通过isDaemon()方法做了一个判断，若是后台线程这调用sleep()方法沉睡1毫秒，将线程执行的机会让给其它线程。若是没有调用sleep()方法，则有可能看到后台线程打印语句的输出结果，因为有可能后台线程先于普通线程执行。
6、结束一个线程
线程是有生命周期的，当一个线程方法体执行完毕时，该线程就死亡了，也就意味着这一个线程结束了。然而也有例外的情况，一种是线程方法体执行时抛出了未捕获的异常或错误，另一种则是调用Thread类的stop()方法主动结束一个线程。当然这两种结束线程的方法都是有问题的，前者是代码有bug或者硬件或操作系统出现了故障，后者则容易导致死锁的问题。
（四）线程同步问题
在并发的情况下，多个线程一起访问同一个资源时会出现线程同步的问题。很多程序的bug都是由于线程不同步而造成的，解决线程不同步的方法最常用的是使用Java的锁机制对代码进行加锁，而加锁的房补不同和加锁的范围不同又分几种情况。除此之外还可以使用特殊域变量(volatile)实现线程同步。
那么在什么时候需要用到线程同步呢？前面说了，多个线程访问同一个资源的时候就需要对代码进行同步操作了。比如对文件的读写操作，多个线程同时对某个变量进行修改，对数据库的增删改查的访问等等。
1、使用synchronized关键字进行加锁
在Java里面，每个类实例对应一把锁，在使用synchronized对同代码块或者方法进行加锁之后，多个线程同时访问加锁代码时，必须要先获取该实例对象的锁才可以执行这一段代码，并且在这段代码执行过程中，其它线程无法获取这个对象的锁，也就无法进入这段同步代码块或者同步方法中了，确保了并发时多线程的同步问题，这就是Java对象锁的排他性。
（1）使用synchronized关键字对代码块进行加锁
在并发编程中，使用synchronized关键字对代码加锁是很消耗性能的一件事，而有时候我们只需要对一部分代码进行同步操作就可以了，这样相对来说程序性能会好一点，这样的操作我们称为同步代码块。
同步代码块的语法格式如下：
synchronized (obj) {
	// 在大括号内编写的代码就叫同步代码块
}
下面使用对同一个变量同时被访问的场景进行模拟，代码示例如下：
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
这里使用的是简单的加法操作，但在多线程同时访问这几行加法的代码时，变量a的结果可能就变得不准确了，有可能的情况是，变量a的值被另一个线程修改了。当把注释的synchronized取消后，就可以保证当一个线程对a的值进行计算时，另一个线程无法进入synchronized括号内的同不代码块，这就保证了这几行加法代码计算的值是正确的。
（2）使用synchronized关键字对实例方法进行加锁
使用synchronized关键字对实例方法进行加锁，我们称为同步方法，对实例方法加锁的性能一般比只对一小部分代码块性能稍好大一些。对实例方法加锁只需要在方法签名上加一个synchronized修饰符即可。当一个类中有多个被synchronized关键字修饰的同步方法时，并且同一时间有一个线程进入其中一个同步方法时，别的线程就只能这个线程将这个同步方法执行完毕，才能进入执行这个同步方法或者该类下别的实例同步方法。
代码示例如下:
public class SynchronizedMethodTest {
	public int sum;
	public synchronized void add(int aValue){
		sum = sum + aValue;
	}
	public synchronized void sub(int aValue){
		sum = sum - aValue;
	}
}
（3）使用synchronized关键字对类方法进行加锁
前面说到每个对象都有一把锁，同一时间只能有一个线程获取这把锁，然后执行被synchronized关键字修饰的一段代码或方法。而每个类也有这样一把锁，对于每个类里面的静态代码块或者静态方法使用synchronized关键字进行加锁，称之为静态同步代码块或者静态同步方法。同样的，同一时间内，也只能有一个线程执行一个类里面的静态同步代码块或者静态同步方法，并且当静态同步代码块或静态同步方法被执行时，这个类的同步锁被占用，直到静态同步代码块或静态同步方法执行完毕，同步锁被解除时，其它线程才能获得同步锁，执行静态同步代码或者静态同步方法。其实就是比实例代码块和实例方法前面多了个static关键字。
代码示例如下：（以静态同步方法为例）
public class SynchronizedMethodTest {
	public static int sum;
	public static synchronized void add(int aValue){
		sum = sum + aValue;
	}
	public static synchronized void sub(int aValue){
		sum = sum - aValue;
	}
}
2、使用Lock和ReadWriteLock接口的实现类手动加锁
Lock和ReadWriteLock及其实现类的类图如下：（如图2.12所示）
 
图 2.12
除了使用synchronized加锁的方式外，JDK还提供了Lock和ReadWriteLock两个接口来实现手动加锁。实际编码中我们使用Lock的实现类ReentrantLock（可重入锁）和ReadWriteLock的实现类ReentrantReadWriteLock（可重入读写锁）来进行编码操作。手动加锁的方式更加灵活，提供了很多对锁进行操作的方法，比如锁对象的各种属性，是否被某个线程持有该锁等等。ReentrantLock和ReentrantReadWriteLock两个类唯一的区别是当读取某个资源时，ReentrantLock不允许多个线程同时读取，即使是需要多个线程同时读取的资源也不能被其它线程读取，而ReentrantReadWriteLock的ReadLock是允许多线程获取的，WriteLock只能单个线程获取。当然，若对同步锁没有特殊的需求，我们使用synchronized关键字修饰的同步代码块和同步方法就可以了，效果是一样的。
可重入锁ReentrantLock使用的语法格式为：
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
可重入读写锁ReentrantReadWriteLock使用的语法格式为：
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
3、使用特殊域变量（volatile）实现线程同步
volatile关键字为域变量的访问提供了一种免锁机制，使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新，因此每次使用该域就要重新计算，而不是使用寄存器中的值，volatile不会提供任何原子操作，它也不能用来修饰final类型的变量。
还是前面的加法示例，对变量sum加了volatile关键字修饰，同时synchronized就没有必要存在了。
代码示例如下：
public class VolatileTest {
	public volatile int sum;
	public void add(int aValue){
		sum = sum + aValue;
	}
	public void sub(int aValue){
		sum = sum - aValue;
	}
}
4、程序出现死锁
在使用锁机制实现线程同步时，不管是使用synchronized关键还是使用Lock手动加锁的形式，如果出现了两个线程互相等待对方释放同步锁时，我们就说程序出现了死锁。死锁在多线程编程中还是很容易出现的现象。
代码示例如下：
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
代码执行结果为：（如图2.13所示）
 
图 2.13
从代码执行结果中可以看出，ClassA和ClassB两个类的secondMethod()方法都没有执行，两个线程执行到调用secondMethod()方法时，调用secondMethod()方法的对象的同步锁还被另一个线程占用着，然后两个线程就相互阻塞了，线程没有执行完，也没有什么报错。在多线程的实际编程中，这样的死锁是要小心避免的。
（五）线程通信
当多个线程在系统中运行时，线程之间的切换具有一定的随机性。程序一般无法准确的在多个线程之间进行切换，但是Java也提供了一些机制来使多个线程之间能够协调的运行，这就是线程通信。
根据实现线程同步的方式的同步，线程通信的方式也有所不同。
1、使用Object类的wait()、notify()和notifyAll()方法
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
代码执行结果为：（如图2.14所示）
 
图 2.14
2、使用Condition实现线程通信
使用Lock和ReadWriteLock接口的实现类手动加锁的线程通信不像synchronized关键字同步的线程存在隐式的同步监视器，所以也就不能使用wait()、notify()和notifyAll()方法来进行通信了。
但是，就跟前面介绍的时候说的，Lock手动加锁和synchronized关键字加锁有相似之处。使用Condition实现线程通信和使用wait()、notify()和notifyAll()方法来进行通信也是有相似之处的。
Lock接口声明了一个newCondition()方法，这个方法可以用来获得一个Condition对象，这个对象就相当于使用synchronized加锁时的同步监视器，里面有三个类似wait()、notify()和notifyAll()的方法，分别是await()、signal()和signalAll()方法，功能也是一样的。
从Java6的文档上可以看到这样的声明：
await()方法，造成当前线程在接到信号或被中断之前一直处于等待状态。
signal()方法，唤醒一个等待线程。
signalAll()方法， 唤醒所有等待线程。
还是那个收纳盒的例子，将synchronized关键字修饰的同步方法改成使用Lock手动加锁进行线程同步，相应的wait()、notify()和notifyAll()方法则换成await()、signal()和signalAll()方法，而存放物品和取出物品的线程类不变，其代码运行结果是一样的。
代码示例如下：
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
（六）线程池○3	
我们知道，在Java中启动一个线程是需要一定的资源成本的，使用线程池可以有效的控制系统中线程并发的数量，降低资源的消耗，提高线程的响应速度。创建一个线程池我们主要用到java.util.concurrent包下的ThreadPoolExecutor类和它的子类ScheduledThreadPoolExecutor。
根据继承关系，绘制类图如下：（如图2.15所示）
 
图 2.15
ThreadPoolExecutor类有四个构造方法，分别是：
ThreadPoolExecutor(int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime, 
TimeUnit unit, 
BlockingQueue<Runnable> workQueue)
用给定的初始参数和默认的线程工厂及被拒绝的执行处理程序创建新的 ThreadPoolExecutor。
ThreadPoolExecutor(int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime, 
TimeUnit unit, 
BlockingQueue<Runnable> workQueue, 
RejectedExecutionHandler handler)
用给定的初始参数和默认的线程工厂创建新的 ThreadPoolExecutor。
ThreadPoolExecutor(int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime, 
TimeUnit unit, 
BlockingQueue<Runnable> workQueue,
ThreadFactory threadFactory)
用给定的初始参数和默认被拒绝的执行处理程序创建新的 ThreadPoolExecutor。
ThreadPoolExecutor(int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime,
TimeUnit unit, 
BlockingQueue<Runnable> workQueue, 
ThreadFactory threadFactory, 
RejectedExecutionHandler handler)
用给定的初始参数创建新的 ThreadPoolExecutor。
这四个构造方法咋一看，前面三个都是调用第四个构造方法实现的，每一个构造方法都特别复杂，参数很多，使用起来比较麻烦。这里用复杂的一个构造方法说明如何手动创建一个线程池。
JDK6文档○4上是这么写的：
public ThreadPoolExecutor(int corePoolSize,
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
代码执行结果为：（如图2.16所示）
 
图 2.16
这里在构造ThreadPoolExecutor对象时，使用了参数最多的构造方法，后面两个参数使用的是其它构造方法采取的默认值，平时使用时，我们使用最简单的构造方法构造一个ThreadPoolExecutor线程池对象即可。若是有更复杂的需求，则可以自己实现ThreadFactory接口。在Java6提供的接口里，只有一个Executors的内部类DefaultThreadExecutor实现了ThreadFactory接口。
而ThreadPoolExecutor的子类ScheduledThreadPoolExecutor则与定期执行命令相关的线程池。当需要多个辅助线程时，或者要求 ThreadPoolExecutor 具有额外的灵活性或功能时，此类要优于 Timer，因为Timer是单线程的。
一旦启用已延迟的任务就执行它，但是有关何时启用，启用后何时执行则没有任何实时保证。按照提交的先进先出 (FIFO) 顺序来启用那些被安排在同一执行时间的任务。 
虽然此类继承自 ThreadPoolExecutor，但是几个继承的调整方法对此类并无作用。特别是，因为它作为一个使用 corePoolSize 线程和一个无界队列的固定大小的池，所以调整 maximumPoolSize 没有什么效果。
	所以ScheduledThreadPoolExecutor类的构造方法的参数少了很多。有四个构造方法如下。
ScheduledThreadPoolExecutor(int corePoolSize) 
使用给定核心池大小创建一个新 ScheduledThreadPoolExecutor。
ScheduledThreadPoolExecutor(int corePoolSize, RejectedExecutionHandler handler) 
使用给定初始参数创建一个新 ScheduledThreadPoolExecutor。
ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory) 
使用给定的初始参数创建一个新 ScheduledThreadPoolExecutor。
ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory, RejectedExecutionHandler handler) 
使用给定初始参数创建一个新 ScheduledThreadPoolExecutor。
	参数与其父类ThreadPoolExecutor表达的意思是一样的，就不重复说明了。这里也以参数最少的构造方法示例ScheduledThreadPoolExecutor类的用法。
前面的例子使用的是继承Thread类的线程使用线程池，方法是调用线程池的execute(Runnable command)方法来提交任务给线程池的。下面的例子使用线程池的submit(Callable<T> task)方法来提交实现Callable接口的线程任务给线程池，该方法有多种重载形式，这里使用其中一种作为示例。
	代码示例如下：
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
代码执行结果为：（如图2.17所示）
 
图 2.17
上面的代码其实只是用到了从ThreadPoolExecutor类继承来的方法，并没有使用ScheduledThreadPoolExecutor类我称之为“多线程定时任务线程池”的功能。要使ScheduledThreadPoolExecutor线程池能够定时的执行任务，需要使用ScheduledThreadPoolExecutor类的schedule()方法或scheduleAtFixedRate()方法。它们的方法签名如下：
<V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit) 
创建并执行在给定延迟后启用的 ScheduledFuture。
 ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) 
创建并执行在给定延迟后启用的一次性操作。
ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)
创建并执行一个在给定初始延迟后首次启用的定期操作，后续操作具有给定的周期；也就是将在 initialDelay 后开始执行，然后在 initialDelay+period 后执行，接着在 initialDelay + 2 * period 后执行，依此类推。
ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay,TimeUnit unit) 
创建并执行一个在给定初始延迟后首次启用的定期操作，随后，在每一次执行终止和下一次执行开始之间都存在给定的延迟。
这里简单举个定时任务的例子，大部分代码如上个例子没变，只需要改几行代码即可。代码示例如下：
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
而代码的执行结果也没有变化，唯一不同的是，每个线程执行前都会等待3秒钟。也就是相当于把线程定时在3秒后执行而不是立刻执行。
对于类图中的ForkJoinPool这个类，这是Java7新增的类，Java8对这个类又增加了几个方法。ForkJoinPool用于将一个大任务拆分成多个小任务并行处理，处理完后再将结果合并得到最终结果。目前而言，这个类几乎不会用到，只有当对性能要求很高的程序才会使用它进行优化。
以上便是手动构造线程池的一般方法，从以上手动构造一个线程池的代码中可以看出，参数多且复杂，幸运的是，从Java5开始，提供了Executor并发框架来简化线程池的创建。
	Executor框架能够创建两类的线程池，分别对应前面介绍的ThreadPoolExecutor类和它的子类ScheduledThreadPoolExecutor，Java提供了Executors工厂类，提供了一些静态工厂方法来构造常用的线程池，并且参数少了，代码意图更加清晰。
1、三种ThreadPoolExecutor类型的线程池○5	
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
2、两种ScheduledThreadPoolExecutor类型的线程池○6
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
 
三、Java多线程下的并发编程技巧
（一）包装线程不安全的集合○7
使用Java编程时经常使用的集合类ArrayList、HashSet和HashMap等都是线程不安全的集合。当多个线程同时访问同一个这种线程不安全的集合类时，就会出现问题。因此Java提供了一个Collections工具类中的多个静态方法来将线程不安全的集合类包装成线程安全的集合类。这些静态方法都是以synchronized开头的方法。方法签名如下：
<T> Collection<T> synchronizedCollection(Collection<T> c) 
返回指定 collection 支持的同步（线程安全的）collection。
<T> List<T> synchronizedList(List<T> list) 
返回指定列表支持的同步（线程安全的）列表。
<K,V> Map<K,V> synchronizedMap(Map<K,V> m) 
返回由指定映射支持的同步（线程安全的）映射。
<T> Set<T> synchronizedSet(Set<T> s) 
返回指定 set 支持的同步（线程安全的）set。
<K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m) 
返回指定有序映射支持的同步（线程安全的）有序映射。
<T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s) 
返回指定有序 set 支持的同步（线程安全的）有序 set。
一般直接以实例化的对象作为这些方法的参数，避免包装之前集合就被线程修改了。这里以ArrayList为例，代码示例如下：
public class SynchronizedListTest {
	public static void main(String[] args) {
		List<String> list = Collections.synchronizedList(new ArrayList<String>());
		System.out.println(list.getClass());
	}
}
代码执行结果为：（如图3.1所示）
 
图 3.1
	从反射得到的信息可以知道包装后的集合是Collections的内部类SynchronizedRandomAccessList。而通过源码可以看出其它的集合被响应的方法包装后也是Collections对应的内部类，分别对应SynchronizedCollection、SynchronizedSet、SynchronizedSortedSet、SynchronizedMap、SynchronizedSortedMap等Collections的内部类。
（二）使用线程安全的集合类
前面说到一个技巧是包装线程不安全的集合类，可以得到一个线程安全的集合类。这种方法一般是一开始没有考虑到多线程环境或者为了效率。若是一开始就知道程序会运行在高并发情景下，并且数据的正确性比较重要但是程序效率又没那么重要时，可以考虑直接使用java.util.concurrent包下的线程安全的集合类。
Java提供的线程安全的集合有两类。一是类名以Concurrent开头的类，如ConcurrentHashMap、ConcurrentLinkedQueue、ConcurrentSkipListMap、ConcurrentSkipListSet和ConcurrentLinkedDeque。这一类集合采用了非常复杂的算法保证永远不会锁住集合，任何时候读操作都是允许的，而写操作也有比较好的性能。
另一种是类名以CopyOnWrite开头的集合，如CopyOnWriteArrayList和CopyOnWriteArraySet。这一类集合采用的是写操作先复制一份，写操作在副本中进行，因此不会影响读操作。
根据Java提供的文档，对线程安全的集合绘制了一个类图，如下所示：（如图3.2所示）
 
图 3.2
（三）使用ThreadLocal类
ThreadLocal类提供的是线程本地值，对于每一个访问该类实例的线程，都有一个副本值，线程所有的取值设值操作都在该线程的本地值中操作，互不相干扰。
ThreadLocal有以下三个方法：
T get() 返回此线程局部变量的当前线程副本中的值。
void remove() 移除此线程局部变量当前线程的值。
void set(T value) 将此线程局部变量的当前线程副本中的值设置为指定值。
代码示例如下：
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
代码执行结果为：（如图3.3所示）
 
图 3.3
ThreadLocal还有一个子类InheritableThreadLocal，该类扩展了 ThreadLocal，为子线程提供从父线程那里继承的值：在创建子线程时，子线程会接收所有可继承的线程局部变量的初始值，以获得父线程所具有的值。通常，子线程的值与父线程的值是一致的；但是，通过重写这个类中的 childValue 方法，子线程的值可以作为父线程值的一个任意函数。

 
四、总  结
	本文详细介绍了Java多线程编程的技术，包括多线程的概念和意义，创建多线程的三种方式，Java线程的生命周期，Java的线程锁机制，还包括Java多线程的核心应用线程池的管理。除此之外，还介绍了一些在多线程编程中的一些技巧。
	本文通过记录Java多线程方方面面的基础知识，对每一个基础知识点都是用Java代码实现了一遍，并将那些代码的执行结果分别截图记录了下来，加深了对Java多线程技术的理解。并且，通过对Java多线程里jdk提供的API做了详细的描述，通过对API绘制类图的方式，理清楚了Java多线程里的各个类的继承关系。
Java多线程编程是属于易学难精的知识，没有实际项目的实战是不可能精通的。本文对于Java多线程编程的基础知识点进行了详细的整理，相信通过掌握这些Java多线程编程的基础知识点能在将来实际的商业应用项目中发挥重大作用。
最后，对于Java多线程的探讨如果更为深入的研究分析，那么足够写一本厚厚的书了，就到此为止吧。掌握这些已经足够面对常用的Java多线程应用领域了。
