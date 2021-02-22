---
title: 多线程基础
tags:
  - java
  - 多线程
categories: java
date: 2021-02-22 23:23:54
---
## 基本概念
程序: 能够完成一定任务或者功能的代码集合, 时指令和数据的有序集合, 是一段静态代码.  
进程: 就是应用程序在内存中分配的空间, 即正在运行的程序. 进程是操作系统分配资源的基本单位.  
线程: 线程是操作系统调度的基本单位.  

### 进程和线程区别
进程是一个独立运行的环境, 线程是进程中执行的一个任务. 本质区别是是否单独战友内存地址空间以及其他系统资源.  
- 进程单独战友内存地址空间, 进程间存在内存隔离, 数据共享复杂, 同步简单, 各个进程间互不干扰; 线程共享所属进程的内存地址空间和资源, 数据共享简单, 但是同步复杂.
- 一个进程出现问题不会影响其他进程; 一个线程崩溃可能导致整个进程奔溃.
- 进程的创建销毁不仅需要保存寄存器和栈信息, 还需要资源分配回收级页调度, 开销较大; 线程只需要保存寄存器和栈信息, 开销较小.
- 进程是操作系统进行资源分配的基本单位, 线程是操作系统进行调度的基本单位.

## java多线程类与接口
### Thread及Runnable
#### Thread类
继承Thread类: 
```java
public class Demo1 {
    public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("Run my thread.");
        }
    }

    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
    }
}
```
NB: 
> 调用run()方法不会创建新的线程, 仅是直接执行run()方法. 调用start()方法创建一个线程.  
多次调用start()方法会抛出IllegalThreadStateException异常.

#### Thread类的构造方法
`Thread`类实现了`Runnable`接口, 查看其构造方法, 发现其核心调用了init方法.
```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals)

```
init 参数:
- g: 线程组, 指定该线程所属线程组.
- target: 需要执行的任务.
- name: 线程的名称
- stackSize: 期望的线程栈大小, 未指定默认0, 但是具体大小还是由jvm实现来决定, 有些jvm的实现会忽略该参数.
- acc: 用于初始化inheritedAccessControlContext变量.
- inheritThreadLocals: 可继承的ThreadLocal.如果为true, 从构造线程中继承初始化参数到可继承的thread-locals中.

#### Thread类常用方法
- currentThread(): 返回当前正在执行的线程对象的引用.
- start(): 开始执行线程的方法, java虚拟机会调用线程内的run()方法.
- yield(): 提示调度器当前线程想要放弃当前处理器的占用(调度器可能会忽略该提示). 即使调用该方法, 也有可能继续运行该线程.
- sleep(): 使当前线程睡眠一段时间.
- join(): 使当前线程等待另一个线程执行完毕后再继续执行, 内部调用了Object类的wait方法.

#### Runnable接口
> 由于Runnable是函数式接口, 可以直接使用lambda表达式进行实现.

Runnable定义: 
```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```
实现Runnable接口: 
```java
public class RunnableImpl {
    public static class MyThread implements Runnable {
        @Override
        public void run() {
            System.out.println("ClassImpl Running...");
        }
    }

    public static void main(String[] args) {
        new Thread(new MyThread()).start();
        new Thread(() -> System.out.println("Lambda running...")).start();
    }
}
```

### Callable, Future及FutureTask
>Runnable与Thread创建线程无返回值; 使用Callable与Future可以创建有返回的线程.

#### Callable接口
> Callable也是一个函数式接口. 有返回值, 且支持泛型.  

Callable定义:
```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```
Callable实现:
```java
public class CallableImpl {
    private static class MyThread implements Callable<String> {
        @Override
        public String call() throws Exception {
            return "callable result.";
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executor = Executors.newCachedThreadPool();
        MyThread thread = new MyThread();
        //调用的是 <T> Future<T> submit(Callable<T> task):
        Future<String> res = executor.submit(thread);
        System.out.println(res.get());
    }
}
```
NB:  
res.get()会阻塞当前线程, 直到得到结果. 可以使用其有超时时间的get()方法.

#### Future接口
Future定义: 
```java
public interface Future<V> {
    // 尝试取消当前任务的执行. 可能会因任务已完成, 已取消或者其他原因导致取消失败. mayInterruptIfRunning表示是否以中断方式取消线程执行. 执行完该方法后, isDone()方法将会永远返回true. 如果该方法返回true, 那么isCancelled()方法总是返回true.
    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    // 等待线程返回计算结果, 阻塞.
    V get() throws InterruptedException, ExecutionException;

    // 在指定时间内等待计算结果. timeout: 最大等待时间, unit: 时间单位
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

```

#### FutureTask类
> FutrueTask类实现了RunnableFuture接口, RunnableFuture接口同时继承了Runnable接口和Future接口.  

RunnableFuture接口定义:
```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```
FutureTask是JDK提供的一个Future接口实现, 方便我们调用.

FutureTask例子:
```java
public class FutureTaskDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        FutureTask<Integer> futureTask = new FutureTask<>(() -> 7);
        // 调用的是 Future<?> submit(Runnable task);
        Future<?> submit = executorService.submit(futureTask);
        // 不会得到返回值
        System.out.println(submit.get());
        System.out.println(futureTask.get());
    }
}
```



