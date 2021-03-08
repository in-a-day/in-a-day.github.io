---
title: 线程池
tags:
  - java
  - 多线程
categories: java
date: 2021-03-08 15:20:30
---

## 线程池使用原因
- 现阶段java线程实现是1:1的, 即与操作系统的轻量级进程(内核线程的一种高级接口)是一一对应的. 创建和销毁线程消耗较多系统资源. 线程池可以复用线程资源.
- 可以控制并发的数量.
- 可以对线程进行统一的管理.

## 线程池原理
### Executor
线程池的顶级接口是`Executor`, 该接口定义了一个方法`execute`. 下面看一下接口的定义:
```java
public interface Executor {
    void execute(Runnable command);
}

```

### ExecutorService
Executor有一个拓展接口ExecutorService, 常用的ThreadPoolExecutor就是实现了该接口. 下面来看一下ExecutorService的定义:
```java
public interface ExecutorService extends Executor {
	// 有序执行已提交任务的shutdown方法
    void shutdown();

	// 尝试停止所有执行的任务, 暂停等待中的任务, 返回等待执行任务的列表
    List<Runnable> shutdownNow();

	// 返回当前执行器是否已停止
    boolean isShutdown();

	// 如果在调用shutdown方法后所有的任务都完成了返回ture. 在调用shutdown或shutdownNow方法前, 该方法永远返回false.
    boolean isTerminated();

	// 阻塞直到所有任务在shutdown请求之后全部完成, 或者timeout到期, 或当前线程中断. 如果当前执行器终止返回true, 如果经过了timeout还未终止返回false.
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

	// 提交一个callable任务, 返回到Future中
    <T> Future<T> submit(Callable<T> task);

	// 提交一个runnable任务, 并给定一个默认返回值result, 如果任务成功执行, Future的get方法将返回该result.
    <T> Future<T> submit(Runnable task, T result);

	// 提交一个runnable任务.
    Future<?> submit(Runnable task);

	// 调用所有callable任务, 任务完成后返回Future列表, 列表中所有的Future.isDone方法将返回true. 返回的Future顺序与tasks的顺序一一对应.
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

	// 调用给定的tasks, 当任务全部完成或经过timeout时间后返回Future列表, 列表中的Future.isDone将返回true. 如果返回的Future如果未完成就被全部取消. 返回的Future顺序与tasks一一对应.
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

		// 执行给定的tasks, 返回一个已经完成的任务的结果.
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

		// 执行给定的tasks, 如果在给定的timeout结束之前完成, 返回一个完成的任务结果.
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

### ThreadPoolExecutor
ThreadPoolExecutor是ExecutorService的一个具体实现, 首先看一下其构造方法:
```java
public ThreadPoolExecutor(int corePoolSize,
						  int maximumPoolSize,
						  long keepAliveTime,
						  TimeUnit unit,
						  BlockingQueue<Runnable> workQueue) {
	this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
		 Executors.defaultThreadFactory(), defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
						  int maximumPoolSize,
						  long keepAliveTime,
						  TimeUnit unit,
						  BlockingQueue<Runnable> workQueue,
						  ThreadFactory threadFactory) {
	this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
		 threadFactory, defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
						  int maximumPoolSize,
						  long keepAliveTime,
						  TimeUnit unit,
						  BlockingQueue<Runnable> workQueue,
						  RejectedExecutionHandler handler) {
	this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
		 Executors.defaultThreadFactory(), handler);
}

public ThreadPoolExecutor(int corePoolSize,
						  int maximumPoolSize,
						  long keepAliveTime,
						  TimeUnit unit,
						  BlockingQueue<Runnable> workQueue,
						  ThreadFactory threadFactory,
						  RejectedExecutionHandler handler) {
	if (corePoolSize < 0 ||
		maximumPoolSize <= 0 ||
		maximumPoolSize < corePoolSize ||
		keepAliveTime < 0)
		throw new IllegalArgumentException();
	if (workQueue == null || threadFactory == null || handler == null)
		throw new NullPointerException();
	this.acc = System.getSecurityManager() == null ?
			null :
			AccessController.getContext();
	this.corePoolSize = corePoolSize;
	this.maximumPoolSize = maximumPoolSize;
	this.workQueue = workQueue;
	this.keepAliveTime = unit.toNanos(keepAliveTime);
	this.threadFactory = threadFactory;
	this.handler = handler;
}

```
可以看出ThreadPoolExecutor最多有7个参数的构造函数, 下面具体看一下这7个参数.
- int corePoolSize
核心线程数量  
线程池中有两类线程: 核心线程和非核心线程. 核心线程默认情况下一直存在于线程池中, 即使这个核心线程是空闲的. 而非核心线程超过存活时间就会被销毁.
- int maximumPoolSize
最大线程数量  
核心线程数量 + 非核心线程数量
- long keepAliveTime
线程存活时间  
非核心线程处于空闲状态超过这个时间就会被销毁. 如果设置了allCoreThreadTimeOut(true), 那么核心线程也会受到影响.
- TimeUnit unit
keepAliveTime存活时间单位
- BlockingQueue<Runnable> workQueue
阻塞队列, 维护等待执行的Runnable对象.  
常用的几个阻塞队列:
	- LinkedBlockingQueue
	链式阻塞队列, 底层是链表, 默认大小是Integer.MAX_VALUE.
	- ArrayBlockingQueue
	数组阻塞队列, 底层是数组, 需要指定队列的大小.
	- SynchronousQueue
	同步队列, 内部容量为0, 每个put操作必须等待take操作, 反之亦然.
	- DelayQueue
	延迟队列, 该队列中的元素只有当指定的延迟时间到了, 才能够从队列中获取到该元素.

前五个是必要参数, 后两个是非必要参数.
- ThreadFactory threadFactory
线程工厂  
用于批量创建线程, 统一在创建线程时设置一些参数, 如是否守护线程, 线程优先级等. 如果不指定, 会创建一个默认的线程工厂, 如下(定义在Executors中):
```java
static class DefaultThreadFactory implements ThreadFactory {
	private static final AtomicInteger poolNumber = new AtomicInteger(1);
	private final ThreadGroup group;
	private final AtomicInteger threadNumber = new AtomicInteger(1);
	private final String namePrefix;

	DefaultThreadFactory() {
		SecurityManager s = System.getSecurityManager();
		group = (s != null) ? s.getThreadGroup() :
							  Thread.currentThread().getThreadGroup();
		namePrefix = "pool-" +
					  poolNumber.getAndIncrement() +
					 "-thread-";
	}

	public Thread newThread(Runnable r) {
		Thread t = new Thread(group, r,
							  namePrefix + threadNumber.getAndIncrement(),
							  0);
		if (t.isDaemon())
			t.setDaemon(false);
		if (t.getPriority() != Thread.NORM_PRIORITY)
			t.setPriority(Thread.NORM_PRIORITY);
		return t;
	}
}
```

- RejectedExecutionHandler handler
拒绝执行策略  
线程数量大于线程数就会采用拒绝处理策略, 四种拒绝处理策略为:
	- ThreadPoolExecutor.AbortPolicy: 默认拒绝处理策略, 丢弃任务并抛出RejectedExecutionException异常.
	- ThreadPoolExecutor.DiscardPolicy: 丢弃新来的任务, 但是不抛出异常.
	- ThreadPoolExecutor.DiscardOldestPolicy: 丢弃队列头部(最旧)的任务, 然后重新尝试执行程序(如果失败, 重复此过程).
	- ThreadPoolExecutor.CallerRunsPolicy: 由调用线程处理该任务.


