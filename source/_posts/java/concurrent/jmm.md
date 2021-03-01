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

### volatile变量
#### valatile变量两个特性:
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
cpu在执行一系列指令的时候, 在不影响单线程的执行结果下, 可能会将指令重新排序. Java的运行时编译器(JIT)也会将也会将执行进行重排序.Java中有个as-if-serial术语表示不管如何重排序, 必须保证单线程下重排序的结果与本身应有的结果是一致的.
例如一下代码:
```java
int a = 0;
int b = 1;
int c = a + b;
```
以上代码在执行时大致有一下几个步骤:
```
1. a赋值0
2. b赋值1
3. 取a值
4. 取b的值
5. 将a, b值相加存入c
```

以上动作有很多种重排序的方式(例如2, 1, 3, 4, 5), 但是必须保证在动作5执行之前, a, b的值是正确的值.  
接下来看一个指令重排序的例子:
```java
    static int a = 0, b = 0, c = 0, d = 0;
    public static void main(String[] args) throws InterruptedException {
        int count = 0;
        while (true) {
            a = b = c = d = 0;
            Thread t1 = new Thread(() -> {
                a = 1;
                b = c;
            });

            Thread t2 = new Thread(() -> {
                c = 1;
                d = a;
            });
            t1.start();
            t2.start();
            t1.join();
            t2.join();
            count++;
            if (b == 0 && d == 0) {
                System.out.println("产生重排序: " + count +  ", b: " + b + ", d: " + d);
                break;
            }
        }
    }
```
上述代码如果没有指令重排序, 运行结果可能为b, d的结果可能是以下几种: (1, 0), (0, 1), (1, 1). 然而在实际运行情况下会打印出b, d结果为(0, 0)的情况(测试用了354062次), 说明了存在指令重排序. 可能是线程t1中重排序了`b = c;a = 1`, 或是t2重排序`d = a; c = 1;`, 又或是二者都进行了重排序.

**由于volatile变量只能保证可见性, 在不符合以下两条规则的运算场景中, 仍然需要加锁来保证原子性:**
- 运算结果并不依赖变量当前的值, 或者能保证只有单一的线程修改变量的值.
- 变量不需要与其他的状态变量共同参与不变约束.

### volatile 和 monitor
JMM对于volatile和monitor的指令重排序规则:
![jmm_volatile_monitor_rule](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/concurrent/jmm_volatile_monitor_rule.png)_JMM对于volatile和monitor的指令重排序规则_


### 内存屏障(memory)
内存屏障有以下几种:
#### LoadLoad屏障
语句: Load1; LoadLoad; Load2;  
在Load2及后续读取操作要读取的数据被访问前, 保证Load1要读取的数据被读取完毕.
#### StoreStore屏障
语句: Store1; StoreStore; Store2;
在Store2及后续写入操作执行前, 保证Store1的写入操作对其他处理器可见.
#### LoadStore屏障
语句: Load1; LoadStore; Store2;
在Store2及后续写入操作执行前, 保证Load1要读取的数据被读取完毕.
#### StoreLoad屏障
语句: Store1; StoreLoad; Load2;
在Load2及后续读取操作执行前, 保证Store1的写入对所有的处理器可见. 其开销是四种屏障中最大的. 在大多数处理器实现中, 这个屏障是万能屏障, 兼具其他三种内存屏障的功能.

Java编译器内存屏障使用方式:
![jmm_volatile_monitor_rule](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/concurrent/memory_barrier_rule_new.png)_内存屏障规则_


### Happens-Before(先行发生原则)
> Java happens-before是Java内存模式中定义的两项操作间的偏序关系. 如A操作happens-before与B操作, 则在B操作发生之前, A操作的产生的影响能被B操作观察到. 例如以下代码:
```java
// A操作
i = 1;
// B操作
j = i;
```
在B操作完成之前, A操作一定先完成.

**NB:**
**两个操作间存在happens-before关系, 并不意味着Java具体实现按照指定的关系顺序执行, 如果重排序的结果与按happens-before关系来执行的结果一致, 那么JMM也允许这样的重排序.**
```java
i = 1;
j = 2;
k = i + j;
```
**以上代码可能先执行i = 1, 也可能先执行j = 2, 最终结果不影响k = i + j的执行.**


#### JMM天然支持的Happens-Before
> Java无需任何同步手段就可以保证的先行发生规则: 

1. 程序次序规则(Program Order Rule): 在一个线程内, 按照控制流顺序, 书写在前面的操作先行发生于书写在后面的操作.
2. 管程锁定规则(Monitor Lock Rule): 一个unlock操作先行发生于后面对同一个锁的lock操作. 这里必须是"同一个锁", "后面"指的是时间上的先后.
3. volatile变量规则(Volatile Variable Rule): 对一个volatile变量的写操作先行发生于后面对这个变量的读操作. "后面"指的是时间上的先后.
4. 线程启动规则(Thread Start Rule): Thread对象的start()方法先行发生于此线程的每个动作.
5. 线程终止规则(Thread Termination Rule): 线程中所有操作都先行发生关于对此线程的终止检测. 通过Thread.join()方法是否结束, Thread.isAlive()的返回值等方法检测线程是否已经终止执行. 如果A线程调用了线程B的join()方法并发返回成功, 那么B线程的任意操作都发生于A线程调用B.join()返回成功之前.
6. 线程中断规则(Thread Interruption Rule): 对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生. 通过Thread.interrupted()方法检测是否有中断发生.
7. 对象终结规则(Finalizer Rule): 一个对象的初始化完成(构造函数执行结构)先行发生于他的finalize()方法的开始.
8. 传递性(Transitivity): 如果操作A先行发生于操作B, 操作B先行发生于操作C, 那么就可以得出操作A先行发生于操作C.








