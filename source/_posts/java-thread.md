---
title: Java线程与线程池
date: 2015-02-25 14:06:37
categories: java
tags:
- 线程
description: 为什么使用线程？程序的运行模式可以分成串行和并行，传统的dos系统就是典型的串行程序的运行，在同一时间内只能有一个程序运行，而并行就是可以同一时间运行多个程序。多个程序就是多个进程。
---
# 线程
## 为什么使用线程？
程序的运行模式可以分成串行和并行，传统的dos系统就是典型的串行程序的运行，在同一时间内只能有一个程序运行，而并行就是可以同一时间运行多个程序。多个程序就是多个进程。

> 而线程是相对于进程而言的。在操作系统中，可以在同一个时间段上运行多个程序。所有的程序都是并发运行的。但在一个时间点上只能有一个进行执行。
线程实际就是进程基础上的进一步划分，一个进程可以划分多个线程。
就好比word，word运行于windows中就相当于一个进程，而拼写检查就是这个进程中的一个线程，当word运行时，才会可能存在拼写检查的操作。

> 在一个程序中，这些独立运行的程序片段就叫作线程，它能允许程序在同一时间执行多个任务，进而提升整体处理性能。

> 一个采用了多线程技术的应用程序可以更好地利用系统资源。其主要优势在于充分利用了CPU的空闲时间片，可以用尽可能少的时间来对用户的要求做出响应，使得进程的整体运行效率得到较大提高，同时增强了应用程序的灵活性。更为重要的是，由于同一进程的所有线程是共享同一内存，所以不需要特殊的数据传送机制，不需要建立共享存储区或共享文件，从而使得不同任务之间的协调操作与运行、数据的交互、资源的分配等问题更加易于解决。

## Java创建线程的三种方式及对比

### 1.继承Thread 类

重写run方法，方法体代表要处理的任务。因此把`run()`方法称为执行体。
调用`start()`方法来启动线程
在Thread中可以直接使用`getName()`方法返回调用的当前线程的线程名
使用`Thread.currentThread()`方法返回当前正在执行的线程对象

```java
class Thread_Thread extends Thread {
	int i = 0;
	
	public void run() {
		for (; i<100; i++) {
			System.out.println(getName() + "" +i);
		}
	}
	
}
```
### 2.实现Runnable接口

重写`run()`方法
借助`Thread`类来创建`Thread`对象
调用线程对象`start()`方法来启动线程
```java
class Thread_Runnable implements Runnable {
	
	private int i;

	@Override
	public void run() {
		// TODO Auto-generated method stub
		for (; i<100; i++) {
			System.out.println(Thread.currentThread().getName() + "" +i);
		}
	}
}
```
### 3.通过Callable和Future创建线程

#### 1) 实现Callable接口，重写实现call（）方法。call()方法作为线程执行体，并且有返回值
#### 2) 使用FutureTask类来包装Callable对象，该FutureTask对象封装了Callable对象的call()方法的返回值
#### 3) 使用FutureTask对象作为Thread对象的target创建并启动新线程|
#### 4) 调用FutureTask对象的get() 方法来获得子线程执行结束后的返回值

```java
package com.thread;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableThreadTest implements Callable
{

	public static void main(String[] args)
	{
		CallableThreadTest ctt = new CallableThreadTest();
		FutureTask ft = new FutureTask<>(ctt);
		for(int i = 0;i < 100;i++)
		{
			System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);
			if(i==20)
			{
				new Thread(ft,"有返回值的线程").start();
			}
		}
		try
		{
			System.out.println("子线程的返回值："+ft.get());
		} catch (InterruptedException e)
		{
			e.printStackTrace();
		} catch (ExecutionException e)
		{
			e.printStackTrace();
		}

	}

	@Override
	public Integer call() throws Exception
	{
		int i = 0;
		for(;i<100;i++)
		{
			System.out.println(Thread.currentThread().getName()+" "+i);
		}
		return i;
	}

}
```

## 4.创建线程三种方式的对比

- *继承局限：* 采用Runnable， Callable接口的方式创建多线程时，可以避免单继承的局限，而Thread类则有此局限。
- *资源共享：* 采用Runnable, Callable接口的方式，可以共享同一个target对象，所以适合用来多个相同线程来处理同一个资源的情况。从而把CPU，代码和数据分开。较好地体现了面向对象的思想。

	> 为什么能实现资源共享：
	因为Runnable和Callable 接口实现的线程，最终都是通过Thread类来启动，相当于一个任务可以启动多个线程，用多个线程来同时处理这个任务。实际是开辟一个线程将任务传递进去，由此线程执行。可以实例化多个Thread对象 ，将同一个任务传递进去。这些线程执行的是同一个任务。所以他们的资源是共享的。
```java
public class RunnableDemo02  
{  
    public static void main(String args[]){  
        MyThread A = new MyThread("A");  //实例化线程要执行的任务  
        Thread Ta = new Thread(A);    //实例两个线程对象，实际传递的是一个任务  
        Thread Tb = new Thread(A);    //因为两个线程执行的是一个任务，所以资源是共享的  
        Ta.start();  
        Tb.start();  
    }  
}
```
	> 而Thread类实现的线程，线程和线程要执行的任务是捆绑在一起的。也就使得一个任务只能启动一个线程，不同的线程执行的任务是不相同的，所以没有必要，也不能让两个线程共享此任务中的资源。

使用Thread类的好处是：编写简单，如果需要访问当前的线程，则无需使用Thread.currentThread()方法，直接使用this即可获取到当前线程。
Thread类实际上是Runnable的子类，而Thread类也要去接收Runnable其他子类的对象，而Runnable实现的子类都是编写具体的功能，并没有CPU调试，真正意义的CPU调试是由操作系统完成的，而Java的实现中就是通过Thread类的start()方法。在Thread的源码中有一个native 的方法 start0 (JNI实现)

```java
public synchronized void start() {
	if (threadStatus != 0)
		throw new IllegalThreadStateException();
	group.add(this);
	start0();

	if (stopBeforeStart) {
		stop0(throwableFromStop);
	}
}

private native void start0();
```

从Thread类与Runnable接口的关系中可以发现，现在在线程中应用的设计思路就是代理设计模式。

## 线程的休眠

休眠就是减缓程序的运行速度。

sleep(long millis)

 

## 线程同步与锁

当多个线程同时进行一种资源的操作时，为了保证操作的完整性，所以引入了同步处理。

使用Synchronized关键字来加入锁的标记，锁的标记中将判断和修改同时进行，要想完成锁的程序可以通过以下四种方式：
```java
sychronized method(){}
sychronized (objectReference) {/*block*/}
static synchronized method(){}
sychronized(classname.class)
```

多个线程访问同一个资源的时候需要进行同步，但是过多的同步会产生死锁。

在Object类中提供了以下的方法可以实现对线程的等待及唤醒的处理：

- 等待：`public final void wait() throws InterruptedException`
- 唤醒：`public final void notify()`，唤醒第一个等待的线程
- 唤醒：`public final void notifyAll()`，唤醒全部等待的线程

## 线程其他注意点

Java本身属于多线程的处理机制，每次Java运行的时候，实际上都会启动一个JVM的进程
主方法是在一个JVM上产生的一个线程而已，一个JVM启动的时候会产生两个线程：main,GC
关于线程安全与不安全：线程安全的执行效率相对较低，但是安全，能保证数据的一致性以及操作的完整性。而线程非安全的执行效率肯定高，但是不安全。Java中提供了很多线程安全或者不安全的类，外部可以根据具体的需求来选择需要使用的类。比如在集合中，如果对于不需要同时处理某一资源的情况，可选择使用非线程安全的集合（ArrayList)来处理。
线程的优先级：所有线程在启动之后并不是立刻运行的，都需要等待CPU进行调试。调试的时候会本身也存在优先级。优先级高的则有可能最先被执行。

`setPriority(int new Priority)` 可以设置优先级

- 最高：MAX_PRIORITY
- 中等：NORM_PRIORITY
- 低级：MIN_PRIORITY

# 线程池

## 简介

线程池主要还是在Java1.5之后加入了java.util.concurrent包才使用得比较多，在这之前使用是比较简陋的。

## 线程池的作用

线程池作用就是限制系统中执行线程的数量。

根据系统的环境情况，可以自动或手动设置线程数量，达到运行的最佳效果；少了浪费了系统资源，多了造成系统拥挤效率不高。用线程池控制线程数量，其他线程排队等候。一个任务执行完毕，再从队列的中取最前面的任务开始执行。若队列中没有等待进程，线程池的这一资源处于等待。当一个新任务需要运行时，如果线程池中有等待的工作线程，就可以开始运行了；否则进入等待队列。

## 为什么要使用线程池

减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务
可以根据系统的承受能力，调整线程池中工作线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

## 线程池的使用

Java里面线程池的顶级接口是`Executor`，但是严格意义上讲`Executor`并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是`ExecutorService`。

比较重要的几个类：

`ExecutorService` 真正的线程池接口。
`ScheduledExecutorService` 能和`Timer/TimerTask`类似，解决那些需要任务重复执行的问题。
`ThreadPoolExecutor	ExecutorService`的默认实现。
`ScheduledThreadPoolExecutor`	继承`ThreadPoolExecutor的ScheduledExecutorService`接口实现，周期性任务调度的类实现。
要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在`Executors`类里面提供了一些静态工厂，生成一些常用的线程池。

*newSingleThreadExecutor*

创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

*newFixedThreadPool*

创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。

*newCachedThreadPool*
创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。

*newScheduledThreadPool*

创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。

实例

#### newSingleThreadExecutor
```java
// MyThread.java

publicclassMyThread extends Thread {
	@Override
	publicvoid run() {
	System.out.println(Thread.currentThread().getName() + “正在执行。。。”);

	}
}

// TestSingleThreadExecutor.java

public class TestSingleThreadExecutor {
	public static void main(String[] args) {

//创建一个可重用固定线程数的线程池

ExecutorService pool = Executors. newSingleThreadExecutor();

//创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口

Thread t1 = new MyThread();

Thread t2 = new MyThread();

Thread t3 = new MyThread();

Thread t4 = new MyThread();

Thread t5 = new MyThread();

//将线程放入池中进行执行

pool.execute(t1);

pool.execute(t2);

pool.execute(t3);

pool.execute(t4);

pool.execute(t5);

//关闭线程池

pool.shutdown();

}

}
```

输出结果
```
pool-1-thread-1正在执行。。。
pool-1-thread-1正在执行。。。

pool-1-thread-1正在执行。。。

pool-1-thread-1正在执行。。。

pool-1-thread-1正在执行。。。
```

#### newFixedThreadPool

```java
//TestFixedThreadPool.Java

publicclass TestFixedThreadPool {
	publicstaticvoid main(String[] args) {

		//创建一个可重用固定线程数的线程池

		ExecutorService pool = Executors.newFixedThreadPool(2);

		//创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口

		Thread t1 = new MyThread();

		Thread t2 = new MyThread();

		Thread t3 = new MyThread();

		Thread t4 = new MyThread();

		Thread t5 = new MyThread();

		//将线程放入池中进行执行

		pool.execute(t1);

		pool.execute(t2);

		pool.execute(t3);

		pool.execute(t4);

		pool.execute(t5);

		//关闭线程池

		pool.shutdown();

	}

}
```

输出结果
```
pool-1-thread-1正在执行。。。
pool-1-thread-2正在执行。。。

pool-1-thread-1正在执行。。。

pool-1-thread-2正在执行。。。

pool-1-thread-1正在执行。。。
```

#### newCachedThreadPool
```java

//TestCachedThreadPool.java

publicclass TestCachedThreadPool {
publicstaticvoid main(String[] args) {

//创建一个可重用固定线程数的线程池

ExecutorService pool = Executors.newCachedThreadPool();

//创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口

Thread t1 = new MyThread();

Thread t2 = new MyThread();

Thread t3 = new MyThread();

Thread t4 = new MyThread();

Thread t5 = new MyThread();

//将线程放入池中进行执行

pool.execute(t1);

pool.execute(t2);

pool.execute(t3);

pool.execute(t4);

pool.execute(t5);

//关闭线程池

pool.shutdown();

}

}
```

输出结果：

```
pool-1-thread-2正在执行。。。
pool-1-thread-4正在执行。。。

pool-1-thread-3正在执行。。。

pool-1-thread-1正在执行。。。

pool-1-thread-5正在执行。。。
```

#### newScheduledThreadPool

```java
// TestScheduledThreadPoolExecutor.java

publicclass TestScheduledThreadPoolExecutor {
publicstaticvoid main(String[] args) {

ScheduledThreadPoolExecutor exec = new ScheduledThreadPoolExecutor(1);

exec.scheduleAtFixedRate(new Runnable() {//每隔一段时间就触发异常

@Override

publicvoid run() {

//throw new RuntimeException();

System.out.println(“================”);

}

}, 1000, 5000, TimeUnit.MILLISECONDS);

exec.scheduleAtFixedRate(new Runnable() {//每隔一段时间打印系统时间，证明两者是互不影响的

@Override

publicvoid run() {

System.out.println(System.nanoTime());

}

}, 1000, 2000, TimeUnit.MILLISECONDS);

}

}
```

输出结果
```
================
8384644549516

8386643829034

8388643830710

================

8390643851383

8392643879319

8400643939383
```
 

## ThreadPoolExecutor类

ThreadPoolExecutor的完整构造方法的签名是：`ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) `.

- corePoolSize – 池中所保存的线程数，包括空闲线程。

- maximumPoolSize-池中允许的最大线程数。

- keepAliveTime – 当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间。

- unit – keepAliveTime 参数的时间单位。

- workQueue – 执行前用于保持任务的队列。此队列仅保持由 execute方法提交的 Runnable任务。

- threadFactory – 执行程序创建新线程时使用的工厂。

- handler – 由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序。

- ThreadPoolExecutor是Executors类的底层实现。


# 详细参考资料：

(http://www.cnblogs.com/dolphin0520/p/3932921.html)

(http://www.oschina.net/question/565065_86540)
