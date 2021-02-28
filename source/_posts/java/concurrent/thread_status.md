---
title: 线程状态
tags:
  - java
  - 多线程
categories: java
date: 2021-02-23 16:14:43
---
## 线程状态
### 操作系统中的线程状态
> 现代操作系统线程被认为是轻量级的进程, 所以操作系统中线程与进程状态相同.  
操作系统中的线程主要有以下三个状态:
1. 就绪态(ready): 线程已准备好, 但并未执行
2. 运行态(running): 线程正在cpu上执行
3. 阻塞态(blocked): 一个进程发生了某种操作, 直到发生其他事件才会准备运行. 典型的例子是发起磁盘I/O.

### Java线程状态
```java
public enum State {
    NEW,

    RUNNABLE,

    BLOCKED,

    WAITING,

    TIMED_WAITING,

    TERMINATED;
}
```

#### NEW
> 线程已创建但是尚未启动的状态, 即尚未调用Thread中的start()方法.
```java
Thread thread = new Thread(() -> System.out.println("not yet start"));
// 此时状态为NEW
thread.getState()
```

#### RUNNABLE
> 可运行线程状态. 处于该状态的线程运行于JVM中, 但是可能并未在操作系统中运行. 所以Java的runnable状态线程对应操作系统中的就绪态和运行态.

#### BLOCKED
> 线程阻塞等待监视器锁的状态. 处于该状态的线程等待获取监视器锁进入同步块/方法, 或者在调用Object.wait方法后重入同步块/方法.

### WAITING
> 一个线程在调用一下方法后会进入waiting状态:
- 没有timeout参数的Object.wait, 使当前线程处于等待状态, 除非有其他线程唤醒.
- 没有timeout参数的Thread.join, 等待线程执行完毕, 底层调用的是Object.wait.
- LockSupport.part, 除非获得代用许可, 否则禁止当前线程进行调度.

> 处于waiting状态的线程等待其他线程执行一些特殊的操作. 如: 调用了Object.wait()方法的线程等待其他线程调用Object.notify()或Object.notifyAll()方法.调用Thread.join()等待指定线程执行完毕.

#### TIMED_WAITING
> 调用一下方法将使线程进入timed waiting状态(传入的等待时间为正数):
- Thread.sleep
- Object.wait
- Thread.join
- LockSupport.parkNanos
- LockSupport.parkUntil

#### TERMINATED
> 终止的线程的状态.线程已完成执行.

### Java线程状态转换
![Java线程状态转换](http://concurrent.redspider.group/article/01/imgs/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2%E5%9B%BE.png)_Java线程状态转换_

### 线程中断
> 目前Java中没有安全直接的方法停止线程, 只是提供了线程中断机制来处理需要中断线程的情况.  
> 线程中断是一种协作机制. 通过中断操作不能直接终止一个线程, 而是通知需要被中断的线程自行处理.

Thread类有关线程中断的方法:
- Thread.interrupt(): 中断线程, 设置线程的中断状态为true.
- Thread.interrupted(): 静态方法, 设置连续调用两次会将线程的中断状态设置为true.
- Thread.isInterrupted(): 判断线程是否中断, 不会改变状态.




