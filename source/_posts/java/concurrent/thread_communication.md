---
title: 线程通信
tags:
  - java
  - 多线程
categories: java
date: 2021-02-25 00:25:34
---
## 线程通信
线程有自己私有的上下文, 互不干扰. 但多个线程需要协作时, 就需要使用Java的线程通信方式.

### 锁与同步
Java中, 所有锁都是基于对象的, 所以又称对象锁.(内置锁, 监视器锁) Java的内置锁是一种互斥锁, 一个锁同一时间只能被一个线程持有. 其他线程想要得到该锁, 必须要等持有该锁的线程释放才行.

Java内置锁是可重入锁, 如果一个线程试图获得一个已被它自己持有的锁, 那么这个请求就会成功.

使用锁可以进行线程间的同步.下面来看一个用锁进行同步的例子:
```java
    public static void main(String[] args) {
        Object lock = new Object();
        Thread a = new Thread(() -> {
            synchronized (lock) {
                IntStream.range(0, 100).forEach(it -> System.out.println(Thread.currentThread().getName() + it));
            }
        }, "Thread a");
        Thread b = new Thread(() -> {
            synchronized (lock) {
                IntStream.range(0, 100).forEach(it -> System.out.println(Thread.currentThread().getName() + it));
            }
        }, "Thread b");
        // lock保证了a与b是同步执行的, 即若a先获得锁, 待a执行完毕, b才能进入同步方法执行, 反之亦然.
        a.start();
        b.start();
    }

```

### 等待/通知机制
基于锁实现同步, 线程需要不断获取锁, 如果失败了, 还有再次获取, 会消耗服务器资源.  
Java的等待/通知机制基于Object类的wait()方法和notify()方法.
> wait()方法使进程进入waiting状态, 等待其他线程(拥有同一个锁)唤醒.
> notify方法随机唤醒一个进程(唤醒了也不一定立即在cpu上执行), notifyAll唤醒所有等待同一个锁的进程.

NB:
> 有时候线程调用了Object.wait(), 尽管没有其他线程调用响应的notify方法, 该线程也会自动唤醒.
> As in the one argument version, interrupts and spurious wakeups are possible, and this method should always be used in a loop:

```java
     synchronized (obj) {
         while (;condition does not hold;)
             obj.wait();
         ... // Perform action appropriate to condition
     }
```

下面来看一下利用等待/通知机制的例子:
```java
	// 保证a线程wait后不会自行启动
    public static volatile boolean condition = false;
    public static void main(String[] args) {
        Object lock = new Object();
        Thread a = new Thread(() -> {
            synchronized (lock) {
                while (!condition) {
                    System.out.println("Thread a is waiting");
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("Thread a is done");
            }
        });

        Thread b = new Thread(() -> {
            synchronized (lock) {
                condition = true;
                System.out.println("Thread b is Running" + condition);
				lock.notify();
            }
        });

        a.start();
        b.start();
    }
```

以上例子中若a线程先获得锁, 将依次输出以下: 
```
Thread a is waiting
Thread b is running
Thread a is done.
```
若b线程先获得锁, 依次输出以下:
```
Thread b is running
Thread a is done.
```

### 信号量 TODO
信号量是一个有整数值的对象. 通常有增加和减少两个操作, 且是原子操作.

JDK提供了一个类似于信号量功能的类Semahore.

### 管道
管道是基于管道流的通信方式. JDK提供了PipedWriter, PipedReader, PipedOutputStream, PipedInputeStream

### 其他通信相关
#### join方法

#### sleep方法

#### ThreadLocal类

#### InheritableThreadLocal

