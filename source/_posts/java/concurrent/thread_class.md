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
#### nextThreadNum
在调用Thread
```java
// 自动编号匿名线程
private static synchronized int nextThreadNum() {
	return threadInitNumber++;
}
```

#### 




