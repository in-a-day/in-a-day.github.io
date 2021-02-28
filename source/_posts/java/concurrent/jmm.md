---
title: Java内存模型
tags:
  - java
  - 多线程
categories: java
date: 2021-02-25 11:33:00
---
## Java内存模型抽象
### 运行时数据区
![java运行时数据区](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/concurrent/Java运行时数据区.png)_java运行时数据区_

对于每个线程来说, 栈是私有的, 堆是共有的. 栈中的变量不会共享, 不会受到内存模型的影响. 堆中的变量是共享的, 称之为共享变量.

### 缓存一致性
尽管堆中的变量是共享的, 仍然会有内存不可见的问题. 现代计算机通常会在高速缓存区中缓存共享变量.
> 线程间共享变量存在于主存中, 每个线程都会有一个私有的工作内存, 存储了共享变量的副本. 工作内存是Java内存的一个抽象模式, 实际并不存在. 其涵盖了缓存, 写缓冲区, 寄存器等.

Java线程间通信由Java内存模型控制(JMM). JMM抽象示意图如下:
![jmm抽象示意图](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/concurrent/JMM抽象示意图.jpg)

从图中可以看出:
1. 所有的共享变量都存在于主内存中.
2. 每个线程都维护一个共享变量的副本.
3. 如果A, B线程需要通信, 则必须进行以下两个步骤:
    1. A将共享变量更新到主存.
    2. B从主存读取A更新的共享变量到工作内存.

JMM规定: 线程对共享变量的所有操作必须现在工作内存进行, 不能直接在主存中操作. 所以B线程首先在工作内存中查找共享变量, 发现该变量更新了, 再从主存拷贝该变量的值到工作内存中.

线程如何知道共享变量更新?
> JMM通过控制主存与每个工作内存之间的交互, 来提供内存的可见性. 

Java中的volatile关键字保证共享变量的可见性并禁止指令重排序. synchronized关键字保证了可见性及原子性. JMM底层使用内存屏障来实现内存的可见性及禁止重排序.

Java内存模型与硬件关系图:
![java内存模型与硬件关系](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/concurrent/java内存模型与硬件关系.jpg)


![java内存模型与硬件关系](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/concurrent/java内存模型2.png)

### 内存交互操作
Java内存模型定义了8个操作完成主内存与工作内存的交互操作, 且保证每种操作都是原子的(double, long类型变量的load, store, read, write操作在某些平台上允许例外):
- lock: 作用于主内存的变量, 把变量标识为一条线程独占的状态.
- unlock: 作用于主内存变量, 将处于锁定状态的变量释放出来, 释放后的变量可以被其他线程锁定.
- read: 作用于主内存变量, 把一个变量值从主内存传输到线程的工作内存中, 以便load动作的使用.
- load: 作用于工作内存变量, 将read操作从主存中得到的变量放入工作内存中.
- use: 作用于工作内存变量, 将工作内存中一个变量的值传递给执行引擎, 每当虚拟机遇到一个需要使用变量的值的字节码指令时执行这个操作.
- assign: 作用于工作内存变量, 将从执行引擎接收的值赋值给工作内存的变量, 每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作.
- store: 作用于工作内存变量, 将工作内存中一个变量的值传递到主内存中, 以便随后的write操作使用.
- write: 作用于主内存变量, 将store操作从工作内存中得到的变量值放入主内存的变量中.

Java内存模式同时规定了在执行以上8中操作的同时必须同时满足一下规则:
- 不允许read, load, store, write操作之一单独出现, 即不允许变量从主内存中读取了但工作内存不接受, 或工作内存发起回写但是主内存不接受的情况.
- 不允许一个线程丢弃其最近的assign操作, 即变量在工作内存中改变了之后, 必须将该改变同步回主内存.
- 不允许一个线程无原因(没有发生过任何assign操作)把数据从线程的工作内存同步回主内存.
- 一个新的变量只能在主内存中诞生, 不允许在工作内存中直接使用一个未被初始化(load或assign)的变量. 即对一个变量实施use, store操作之前, 必须执行assgin和load操作.
- 一个变量在同一时刻只允许一条线程对其进行lock, 但lock可以被同一线程重复执行多次, 多次执行lock后, 只有执行相同次数的unlock, 变量才会解锁.
- 如果对一个变量执行lock, 将会清空工作内存中此变量的值, 在执行引擎使用这个变量前, 需要重新执行load或assign操作初始化变量的值.
- 如果一个变量没有被lock锁定, 那么不允许对其执行unlock, 也不允许区unlock被其他线程锁定的变量.
- 对一个变量执行unlock之前, 必须把此变量同步回主内存中(执行store, write操作).

### volatile变量特殊规则
valatile变量两个特性:
1. 保证变量对所有线程可见性, 一个线程修改了变量的值, 其他线程可以立即得知.但是并不保证变量操作的原子性.
```java
    public static volatile int a = 0;
    public static void main(String[] args) throws InterruptedException {
		// 执行a++ 10w次
        Thread t1 = new Thread(() -> IntStream.range(1, 100000).forEach(it -> a++));
		// 执行a++ 10w次
        Thread t2 = new Thread(() -> IntStream.range(1, 100000).forEach(it -> a++));
        t1.start();
        t2.start();
        t1.join();
        t2.join();
		// 通常a的值 < 200000
        System.out.println(a);
    }

```
上述代码每次执行的结果都会小于200000, 说明volatile并不会保证变量操作的原子性.
2. 禁止指令重排序优化



