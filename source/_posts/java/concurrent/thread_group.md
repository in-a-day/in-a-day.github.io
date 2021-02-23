---
title: 线程组详解
tags:
  - java
  - 多线程
categories: java
date: 2021-02-23 14:44:37
---
## 线程组
Java中使用ThreadGroup表示线程组. 每个Thread都必须存在于ThreadGroup中. 

### 线程组使用
创建线程及线程组demo:
```java
public class ThreadGroupDemo {
    public static void main(String[] args) {
        ThreadGroup threadGroup = new ThreadGroup("MyThreadGroup");
		// 设置最大优先级
		threadGroup.setMaxPriority(5);
        Thread t1 = new Thread(threadGroup, () -> System.out.println("t1: " + Thread.currentThread().getThreadGroup()));
		// 默认会将该线程添加到执行该线程的线程所属线程组中
        Thread t2 = new Thread(() -> System.out.println("t2: " + Thread.currentThread().getThreadGroup()));
        t1.start();
        t2.start();
    }
}
```

### 成员变量
ThreadGroup中的成员变量:
```java
// 当前线程组的父线程组
private final ThreadGroup parent;
// 当前线程组名称
String name;
// 当前线程组的最大优先级
int maxPriority;
// 线程组是否销毁
boolean destroyed;
// 是否守是护线程组, 守护线程组: 在该线程组的最后一个线程停止执行或其最后一个子线程组销毁后, 该线程组自动销毁
boolean daemon;
// 是否允许vm悬停
boolean vmAllowSuspension;

// 记录尚未开始执行的线程数
int nUnstartedThreads = 0;
// 线程组中线程总数
int nthreads;
// 记录所有线程
Thread threads[];

// 子线程组数
int ngroups;
// 所有子线程组
ThreadGroup groups[];
```

### 构造函数
下面来看一下ThreadGroup的构造函数:
```java
/**
 * 创建一个不属于任何线程组的空线程组
 * 该私有构造方法用于创建系统线程组
 */
private ThreadGroup() {     // called from C code
	this.name = "system";
	this.maxPriority = Thread.MAX_PRIORITY;
	this.parent = null;
}

/**
 * 创建一个线程组, 该线程组的父线程组是当前运行线程所属的线程组
 */
public ThreadGroup(String name) {
	this(Thread.currentThread().getThreadGroup(), name);
}

/**
 * 创建一个线程组, 父线程组为指定线程组.
 */
public ThreadGroup(ThreadGroup parent, String name) {
	this(checkParentAccess(parent), parent, name);
}

/**
 * 可以发现, 构造方法最终调用了此构造方法
 */
private ThreadGroup(Void unused, ThreadGroup parent, String name) {
	this.name = name;
    // 拷贝parent线程组参数
	this.maxPriority = parent.maxPriority;
	this.daemon = parent.daemon;
	this.vmAllowSuspension = parent.vmAllowSuspension;
	this.parent = parent;
    // 当前线程组添加到parent的子线程组中
	parent.add(this);
}
```

在第三个构造参数中, 调用了一个checkParentAccess(ThreadGroup parent)函数, 用于检查当前线程是否有权限修改该线程组. 下面来看一下具体代码:
```java
private static Void checkParentAccess(ThreadGroup parent) {
	parent.checkAccess();
	return null;
}

public final void checkAccess() {
	// 如果存在安全管理器, 则调用用安全管理器检查当前线程是否有权限修改该线程组.
	SecurityManager security = System.getSecurityManager();
	if (security != null) {
		security.checkAccess(this);
	}
}
```

第四个构造参数真正进行线程组的初始化, 拷贝了父线程组的信息, 并调用parent.add()方法将当前线程组添加到父线程组的子线程数组中. 下面来看一下parent.add()的具体实现:
```java
private final void add(ThreadGroup g){
	// 加锁
	synchronized (this) {
		// 如果当前线程组已经销毁, 抛出异常
		if (destroyed) {
			throw new IllegalThreadStateException();
		}
		// 如果子线程数组为空, 初始化
		if (groups == null) {
			groups = new ThreadGroup[4];
		} else if (ngroups == groups.length) {
			// 如果子线程组数组已满, 扩容长度*2
			groups = Arrays.copyOf(groups, ngroups * 2);
		}
		// 添加线程组到线程组数组中
		groups[ngroups] = g;

		// This is done last so it doesn't matter in case the
		// thread is killed
		// 线程组数量+1
		ngroups++;
	}
}
```

### 常用方法

#### 设置最大优先级
new一个线程组时, 默认复制父线程组的优先级, 调用setMaxPriority(int pri)可以手动设置线程组的最大优先级, 下面看一下源码:
```java
public final void setMaxPriority(int pri) {
	int ngroupsSnapshot;
	ThreadGroup[] groupsSnapshot;
	synchronized (this) {
		checkAccess();
		// 小于1, 大于10不做修改
		if (pri < Thread.MIN_PRIORITY || pri > Thread.MAX_PRIORITY) {
			return;
		}
		// 保证不大于父线程的最大优先级
		maxPriority = (parent != null) ? Math.min(pri, parent.maxPriority) : pri;
		ngroupsSnapshot = ngroups;
		if (groups != null) {
			groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
		} else {
			groupsSnapshot = null;
		}
	}
    // 为每个子线程组都设置最大优先级
	for (int i = 0 ; i < ngroupsSnapshot ; i++) {
		groupsSnapshot[i].setMaxPriority(pri);
	}
}
```

#### 销毁一个线程组
如果线程组已经销毁或线程组中还存在线程抛出异常.
```java
public final void destroy() {
	int ngroupsSnapshot;
	ThreadGroup[] groupsSnapshot;
	synchronized (this) {
		checkAccess();
		if (destroyed || (nthreads > 0)) {
			throw new IllegalThreadStateException();
		}
		ngroupsSnapshot = ngroups;
		if (groups != null) {
			groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
		} else {
			groupsSnapshot = null;
		}
		// 清空当前线程组状态
		if (parent != null) {
			destroyed = true;
			ngroups = 0;
			groups = null;
			nthreads = 0;
			threads = null;
		}
	}
	// 销毁所有子线程组
	for (int i = 0 ; i < ngroupsSnapshot ; i += 1) {
		groupsSnapshot[i].destroy();
	}
	// 从父线程组中移除
	if (parent != null) {
		parent.remove(this);
	}
}

```

#### 判断是否是指定线程组的子线程组
调用parentOf()方法判断是否是指定线程组的子线程: 
```java
public final boolean parentOf(ThreadGroup g) {
	for (; g != null ; g = g.parent) {
		if (g == this) {
			return true;
		}
	}
	return false;
}
```

#### 获取线程组中激活线程的数量
- int activeCount()  调用activCount()方法可以估计激活线程的数量(包含子线程组中线程), 该值仅是近似值.

#### 获取线程组的激活线程
获取激活线程, 如果给定参数的list长度小于激活线程数, 则其他激活线程将不会复制到list中.
```java
// 该方法会获取当前线程组及其子线程组的所有几号线程
public int enumerate(Thread list[]) {
	checkAccess();
	return enumerate(list, 0, true);
}

// recurse为false时仅列出当前线程组的激活线程, 不包含子线程组
public int enumerate(Thread list[], boolean recurse) {
	checkAccess();
	return enumerate(list, 0, recurse);
}

private int enumerate(Thread list[], int n, boolean recurse) {
	int ngroupsSnapshot = 0;
	ThreadGroup[] groupsSnapshot = null;
	synchronized (this) {
		if (destroyed) {
			return 0;
		}
		int nt = nthreads;
        // 仅复制给定list.length数量的激活线程, 其他将忽略
		if (nt > list.length - n) {
			nt = list.length - n;
		}
		for (int i = 0; i < nt; i++) {
			if (threads[i].isAlive()) {
				list[n++] = threads[i];
			}
		}
		if (recurse) {
			ngroupsSnapshot = ngroups;
			if (groups != null) {
				groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
			} else {
				groupsSnapshot = null;
			}
		}
	}
	if (recurse) {
		for (int i = 0 ; i < ngroupsSnapshot ; i++) {
			n = groupsSnapshot[i].enumerate(list, n, true);
		}
	}
	return n;
}

```



#### 一些属性get方法
- String getName() 获取当前线程组的名称
- ThreadGroup getParent() 获取当前线程组的父线程组
- int getMaxPriority() 获取当前线程组最大优先级
- boolean isDaemon() 当前线程组是否是守护线程组 
- boolean isDestroyed() 当前线程组是否已销毁













