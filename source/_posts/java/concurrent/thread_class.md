---
title: Thread类详解
tags:
  - java
  - 多线程
categories: java
date: 2021-03-04 15:01:08
---

## Thread
一个`Thread`就是一个在程序中执行的线程. JVM允许一个应用有多个并发执行的线程.  
每个线程都有一个优先级, 优先级高的线程优先于优先级低的线程执行(并不是绝对的).
一个线程可能是守护线程. 当新建一个`Thread`对象时, 该对象的优先级继承自创建该对象的线程的优先级. 如果创建线程是守护线程, 那么该新线程也是守护线程.  
JVM启动时, 通常只有一个非守护线程(调用指定类的main方法). 发生以下情况时, JVM停止执行线程:

- 调用了`Runtime`类的exit方法, 并且安全管理器允许执行退出操作.
- 所有的非守护线程全部结束, 或者从`run`方法调用中返回, 或者从`run`方法中抛出了异常.

### **Thread初始化逻辑**
分析源码之前, 先看一下总结:
- 构造新线程时可以传入需要执行的Runnable实例, 线程所属线程组, 线程期望的线程栈大小和线程名称.
- 如果传入的线程名称为null将会抛出异常.
- 若未传入所属线程组, 默认会使用父线程的线程组(安全管理器为空或未指定线程组的情况下)
- 设置新线程的优先级为创建线程的优先级
- 若创建程是守护线程设置新线程为守护线程, 否则设置为非守护线程
- 若未传入线程栈大小, 默认设置为0, 表示JVM可以忽略该参数
- 给新线程设置一个唯一的线程id


#### **构造方法**
下面看一下Thread的构造方法, 可以看到构造函数中可以有以下参数:  
- ThreadGroup group: 指定线程所属线程组
- Runnable target: 需要运行的线程方法
- String name: 线程的名称
- long stackSize: 线程栈期望的大小(实际大小由JVM决定), 如果是0表示忽略该参数

所有构造方法都是调用了类中的init方法. 
```java
    public Thread() {
		// 如果没有传入name(线程名称), 自动生成一个线程名称, 以Thread-开头
		// 如果没有传入stackSize, 忽略该参数
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }

    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }

	// 不对外开放, 只提供给同包下类使用
    Thread(Runnable target, AccessControlContext acc) {
        init(null, target, "Thread-" + nextThreadNum(), 0, acc, false);
    }

    public Thread(ThreadGroup group, Runnable target) {
        init(group, target, "Thread-" + nextThreadNum(), 0);
    }

    public Thread(String name) {
        init(null, null, name, 0);
    }

    public Thread(ThreadGroup group, String name) {
        init(group, null, name, 0);
    }

    public Thread(Runnable target, String name) {
        init(null, target, name, 0);
    }

    public Thread(ThreadGroup group, Runnable target, String name) {
        init(group, target, name, 0);
    }

    public Thread(ThreadGroup group, Runnable target, String name,
                  long stackSize) {
        init(group, target, name, stackSize);
    }
```

#### **init方法**
接下来看一下具体执行初始化逻辑的`init(ThreadGroup g, Runnable target, String name, long stackSize, AccessControlContext acc, boolean inheritThreadLocals)`方法, `init`方法除了上述构造方法的参数还有以下额外参数:
- AccessControlContext acc: 访问控制上下文
- boolean inheriThreadLocals: 如果为true, 从构造线程的可继承`thread-locals`中继承初始化值.


```java
private void init(ThreadGroup g, Runnable target, String name,
				  long stackSize) {
	init(g, target, name, stackSize, null, true);
}

// 执行具体的初始化逻辑
private void init(ThreadGroup g, Runnable target, String name,
				  long stackSize, AccessControlContext acc,
				  boolean inheritThreadLocals) {
    // 线程名称不可为空
	if (name == null) {
		throw new NullPointerException("name cannot be null");
	}

	this.name = name;

	// 设置父线程
	Thread parent = currentThread();
	SecurityManager security = System.getSecurityManager();
	if (g == null) {
        // 存在安全管理器, 使用安全管理器中的getThreadGroup方法
		if (security != null) {
			g = security.getThreadGroup();
		}

        // 不存在安全管理器或安全管理器中的ThreadGroup为空
		if (g == null) {
			g = parent.getThreadGroup();
		}
	}

    // 检查是否有访问权限
	g.checkAccess();

	if (security != null) {
		if (isCCLOverridden(getClass())) {
			security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
		}
	}

    // 调用线程组的addUnstarted, 将线程组中的未启动线程数量加1
	g.addUnstarted();

    // 将新建的线程所属线程组设置为上述线程组
	this.group = g;
    // 复制创建线程的守护状态
	this.daemon = parent.isDaemon();
    // 复制创建线程的优先级
	this.priority = parent.getPriority();
    // 设置类加载器, 默认使用父线程的上下文类加载器
	if (security == null || isCCLOverridden(parent.getClass()))
		this.contextClassLoader = parent.getContextClassLoader();
	else
		this.contextClassLoader = parent.contextClassLoader;
	this.inheritedAccessControlContext =
			acc != null ? acc : AccessController.getContext();
    // 保存需要执行的target
	this.target = target;
    // 设置优先级
	setPriority(priority);
	if (inheritThreadLocals && parent.inheritableThreadLocals != null)
		this.inheritableThreadLocals =
			ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
	// 设置线程栈大小
	this.stackSize = stackSize;

	/* Set thread ID */
	// 设置线程id
	tid = nextThreadID();
}
```
看一下获取父线程的方法:
- `currentThread`方法用于获取当前正在执行的thread对象, 是一个静态本地方法:

```java
public static native Thread currentThread();
```

设置优先级方法:
```java
public final void setPriority(int newPriority) {
	ThreadGroup g;
	checkAccess();
	// 如果优先级大于最大优先级或者小于最小优先级抛出异常.
	if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
		throw new IllegalArgumentException();
	}
	// 存在线程组才设置优先级
	if((g = getThreadGroup()) != null) {
		// 保证优先级不大于线程组的最大优先级
		if (newPriority > g.getMaxPriority()) {
			newPriority = g.getMaxPriority();
		}
		// 调用本地方法设置优先级
		setPriority0(priority = newPriority);
	}
}
```
init方法中获取线程id方法:
```java
private static synchronized long nextThreadID() {
	return ++threadSeqNumber;
}
```

### **Thread字段解析**
接下来看一下Thread中常用的字段.
```java
// 线程名称
private volatile String name;
// 优先级
private int priority;
// 
private Thread threadQ;
// 是否是守护线程
private boolean daemon = false;
// 需要执行的任务
private Runnable target;
// 线程所属线程组
private ThreadGroup group;
// 用于自动编号匿名线程
private static int threadInitNumber;
// 线程期望栈的大小
private long stackSize;
// 线程id
private long tid;
// 生成thread id的seq
private static long threadSeqNumber;
// java线程状态
private volatile int threadStatus = 0;
// 优先级最小值
public final static int MIN_PRIORITY = 1;
// 优先级默认值
public final static int NORM_PRIORITY = 5;
// 优先级最大值
public final static int MAX_PRIORITY = 10;
```

### **Thread方法解析**
#### nextThreadNum(): 生成匿名线程号
创建新线程而没有传入线程名称时, 将调用该方法生成一个匿名线程号, 线程名称为`Thread-`加上该方法生成的number.
```java
// 自动编号匿名线程
private static synchronized int nextThreadNum() {
	return threadInitNumber++;
}
```

#### nextThreadID(): 生成线程id
创建线程时获取线程的唯一id
```java
private static synchronized long nextThreadID() {
	return ++threadSeqNumber;
}
```

#### currentThread(): 获取当前线程
返回当前正在执行的线程
```java
public static native Thread currentThread();
```

#### **yield(): 线程释放cpu**
提示调度器当前线程想要释放cpu, 调度器可以忽略该提示, 所以调用该方法不一定释放cpu.
```java
public static native void yield();
```

#### **sleep: 睡眠进程**
使当前进程进入睡眠状态
**该方法不会释放线程拥有的锁.**
如果在线程睡眠时, 其他线程中断该线程, 那么将会重置该线程状态, 同时抛出异常.  
sleep方法有以下几个重载: 
```java
public static native void sleep(long millis) throws InterruptedException;
```

```java
public static void sleep(long millis, int nanos)
throws InterruptedException {
	if (millis < 0) {
		throw new IllegalArgumentException("timeout value is negative");
	}

	if (nanos < 0 || nanos > 999999) {
		throw new IllegalArgumentException(
							"nanosecond timeout value out of range");
	}
	//还是使用毫秒级
	if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
		millis++;
	}

	sleep(millis);
}
```

#### **start(): 启动线程**
调用该方法进行启动线程; JVM会调用线程的`run`方法.  
**该方法只能调用一次**
```java
public synchronized void start() {
 	// 如果线程不是NEW状态, 抛出异常
	if (threadStatus != 0)
		throw new IllegalThreadStateException();

	// 线程组中添加该线程, 并将线程组的未启动线程数-1, 启动线程数+1
	group.add(this);

	boolean started = false;
	try {
		// 本地方法
		start0();
		started = true;
	} finally {
		try {
			if (!started) {
				// 启动失败, 从线程组的启动集合中删除线程, 未启动线程数+1, 已启动线程数-1
				group.threadStartFailed(this);
			}
		} catch (Throwable ignore) {
			/* do nothing. If start0 threw a Throwable then
			  it will be passed up the call stack */
		}
	}
}
```

#### **run(): 线程需要执行的方法**
如果使用构造函数传入Runnable对象, 那么调用该对象的run()方法, 否则什么都不做.子类应该重写这个方法.
```java
public void run() {
	if (target != null) {
		target.run();
	}
}
```

#### exit(): 清理线程
在线程完全退出之前, 给线程机会进行清理
```java
private void exit() {
	if (group != null) {
		group.threadTerminated(this);
		group = null;
	}
	target = null;
	threadLocals = null;
	inheritableThreadLocals = null;
	inheritedAccessControlContext = null;
	blocker = null;
	uncaughtExceptionHandler = null;
}
```

#### 线程中断相关
##### interrupt()
如果线程因调用该类的wait, join, sleep方法阻塞, 调用`interrupt`方法将会重置其中断状, 同时抛出InterruptedException.  
如果线程因基于InterruptibleChannel的I/O操作阻塞, 那么该channel将会关闭,线程中断状态会被设置, 并抛出java.nio.channels.ClosedByInterruptException.  
如果线程因 java.nio.channels.Selector阻塞, 线程中断状态将被设置, 并立即从selection操作中返回(可能返回非零值), 就像调用改了selector的wakeup方法.  
如果没有上述情况发生, 那么线程的中断状态会被设置.  
中断一个非存活的线程不会有任何影响.  
```java
public void interrupt() {
	if (this != Thread.currentThread())
		checkAccess();

	synchronized (blockerLock) {
		Interruptible b = blocker;
		if (b != null) {
			// 仅设置中断的状态
			interrupt0();
			b.interrupt(this);
			return;
		}
	}
	interrupt0();
}
```
##### interrupted()
**测试当前线程是否中断, 调用一次设置中断状态为true, 连续调用两次以上会使线程中断状态转为false.调用了本地方法`isInterrupted(boolean ClearInterrupted)`**
```java
public static boolean interrupted() {
	return currentThread().isInterrupted(true);
}
```
##### isInterrupted()
测试当前线程是否中断. 该方法不会影响线程的状态.
```java
public boolean isInterrupted() {
	return isInterrupted(false);
}
```
基于ClearInterrupted清空线程中断状态:  
false不清空; true清空, 即将中断状态设置为false
##### isInterrupted(boolean ClearInterrupted)
```java
private native boolean isInterrupted(boolean ClearInterrupted);
```

#### isAlive(): 判断线程是否存活
```java
public final native boolean isAlive();
```

#### setName(String name): 设置线程名称
及时调用了线程的start方法后, 也可以设置线程名称. 线程名称不能为空
```java
public final synchronized void setName(String name) {
	checkAccess();
	if (name == null) {
		throw new NullPointerException("name cannot be null");
	}

	this.name = name;
	if (threadStatus != 0) {
		setNativeName(name);
	}
}
```

#### activeCount(): 获取当前线程所属线程组活动线程数量
```java
public static int activeCount() {
	return currentThread().getThreadGroup().activeCount();
}
```

#### enumerate(Thread tarray[]): 获取当前线程所有线程
```java
public static int enumerate(Thread tarray[]) {
	return currentThread().getThreadGroup().enumerate(tarray);
}
```

#### join及其重载: 等待线程终止
join方实际上是调用join(0).  
join(long millis, int nanos)与sleep(long millis, int nanos)相似
- 如果nanos < 0 或 nanos > 999999抛出异常
- 如果nanos >= 500000 或 nanos != 0 && millis == 0, millis加一
- 最后调用join(millis).  

所以下面具体看一下join(long millis)方法.  
**join实际上是调用了Object的wait方法**
```java
public final synchronized void join(long millis)
throws InterruptedException {
	long base = System.currentTimeMillis();
	long now = 0;

	if (millis < 0) {
		throw new IllegalArgumentException("timeout value is negative");
	}

	if (millis == 0) {
		while (isAlive()) {
			// 实际上是调用了wait方法
			wait(0);
		}
	} else {
		while (isAlive()) {
			long delay = millis - now;
			if (delay <= 0) {
				break;
			}
			wait(delay);
			now = System.currentTimeMillis() - base;
		}
	}
}
```

#### isDaemon(): 测试线程是否是守护线程
```java
public final boolean isDaemon() {
	return daemon;
}
```


