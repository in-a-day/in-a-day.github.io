---
title: 事务管理
date: 2021-03-16 09:14:59
tags: 
    - spring
categories: spring
---
## **数据库事务相关**
介绍Spring事务相关之前, 先介绍一下数据库有关事务内容.
数据库四大特性: ACID
- Atomicity(原子性): 事务要么成功, 要么失败.
- Consistency(一致性): 系统从一个正确的状态迁移到另一个正确的状态.
- Isolation(隔离性): 允许多个事务并发执行, 事务不会被其他事务影响.
- Durability(持久性): 事务处理结束后对数据的修改是永久的, 即使系统故障也不会丢失.

其中Isolation又分为四个等级:
- read uncommited(未提交读)
- read commited(提交读)
- repeatable read(可重复读)
- serilizable(串行化)

由于不同的隔离策略, 并发时可能产生以下问题:
- 脏读: 事务T1读取数据, 此时事务T2修改了T1读取的数据, 但未做提交, 而T1读取到了T2修改的数据, 这种情况就称为脏读.
- 不可重复读: 事务T1在一次事务中根据同一条件多次读取数据, 事务T2在T1读取间隔内修改了T1读取的数据, 导致T1多次读取的数据不一致, 称为不可重复读.
- 幻读: 事务T1在一次事务中根据同一条件多次读取数据, 事务T2在T1读取的间隔内增加或删除了T1读取的数据, 导致T1多次读取的数据变多/变少了, 称为幻读.

**NB:**  
不可重复读和幻读区别在于不可重复读是修改数据, 而幻读是增加/减少了数据.

## **Spring事务管理抽象**
Spring 使用TransationManager进行事务管理抽象, 通常使用PlatformTransactionManager进行命令式的事务管理.
PlatformTransactionManager接口定义了一下方法:
- TransactionStatus getTransaction(TransactionDefinition definition: 获取事务状态.
- void commit(TransactionStatus status): 提交事务
- void rollback(TransactionStatus status): 回滚事务

TransactionDefinition主要定义了以下内容:
- propagation: 事务传播属性
- isolation: 事务隔离性级别
- timeout: 事务运行时间, 超过改时间将由底层的事务管理器回滚事务.
- read-only: 定义事务只读而不会修改数据.

TransactionStatus用于控制事务和查询事务状态. TransactionStatus 继承了TransactionExecution, SavepointManager, Flushable接口.
```java
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable {
	// 是否存在保存点
	boolean hasSavepoint();

	@Override
	void flush();
}

```

TransactionExecution主要定义了一些表示当前事务状态的接口. 
```java
public interface TransactionExecution {
	// 事务是否是新的事务
	boolean isNewTransaction();
	// 将事务属性设置为只能回滚
	void setRollbackOnly();
	// 事务是否只能回滚
	boolean isRollbackOnly();
	// 事务是否已经完成
	boolean isCompleted();
}

```
SavepointerManager定义了一些管理事务保存点的接口, 创建, 回滚事务保存点等.
```java
public interface SavepointManager {
	// 创建保存点
	Object createSavepoint() throws TransactionException;
	// 回滚到保存点
	void rollbackToSavepoint(Object savepoint) throws TransactionException;
	// 释放保存点
	void releaseSavepoint(Object savepoint) throws TransactionException;
}

```

下面具体介绍一下TransactionDefinition中定义的事务传播属性.

## **Spring事务传播属性**
TransactionDefinition中定义了如下传播属性:
- PROPAGATION_REQUIRED
- PROPAGATION_SUPPORTS
- PROPAGATION_MANDATORY
- PROPAGATION_REQUIRES_NEW
- PROPAGATION_NOT_SUPPORTED
- PROPAGATION_NEVER
- PROPAGATION_NESTED

Spring提供了`Propagation`枚举类来映射到`TransactionDefinition`的传播属性:
```java
public enum Propagation {

	REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),

	SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),

	MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),

	REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),

	NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),

	NEVER(TransactionDefinition.PROPAGATION_NEVER),

	NESTED(TransactionDefinition.PROPAGATION_NESTED);

	private final int value;

	Propagation(int value) {
		this.value = value;
	}

	public int value() {
		return this.value;
	}
}
```

### **PROPAGATION_REQUIRED**
> Spring默认的事务传播行为. 支持当前事务, 如果不存在事务就创建一个新的事务.

**NB: Spring通过Aop进行事务控制, 所以只要理解Aop的触发时机就可以容易理解事务的传播机制.**

下面看一下具体的例子: 
```java
class A {
	@Transaction(propagation=Propagation.REQUIRED)
	public void func() {
		// NB: 这里调用的并不是代理类中的doFunc()方法, 而是被代理的普通方法.
		doFunc();
		B.func();
	}

	public void doFunc() {
		// 处理...
	}
}

class B {
	@Transaction(propagation=Propagation.REQUIRED)
	public void func() {
	}
}
```
如果调用A接口的func函数, 由于不存在事务, A会新建一个事务, A中调用B中的事务时, B将会使用A中的事务. 如果B中的事务发生异常, 那么A, B全部回滚. A中事务发生异常, A, B全部回滚.  

看一下B发生异常的情况(默认A不发生异常):
- B发生异常但是不抛出异常, Sprig Aop无法捕获异常, 正常执行, 事务不会回滚.
- B发生异常并抛出, 且A也抛出异常, A, B都会回滚.
- B发生异常且B抛出该异常, A进行捕获, 并且A不抛出异常, A, B都会回滚. 原理: B中发生异常, Spring的Aop捕获到该异常, 并将事务的状态设置为rollback-only(只能回滚), 由于A, B是同一个事务, 但是A中捕获了该异常, 并未抛出, A在提交事务时, 发现只能回滚事务, 所以抛出了`org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only`异常, 并且回滚了事务.

看一下A发生异常的情况(默认B不发生异常):
- A发生异常并抛出, A, B都会回滚
- A发生异常但是不抛出, 正常提交事务.


假设A方法存在事务, A方法中调用B方法, 则B方法也使用A方法的事务.  
Spring官方的解释:
![prop-required](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/tx_prop_required.png)_PROPAGATION_REQUIRED_

### **PROPAGATION_SUPPORTS**
> 支持当前事务, 如果当前域存在事务, 就以当前事务执行, 如果不存在事务, 就以非事务方式执行.

### **PROPAGATION_MANDATORY**
> 支持当前事务, 如果不存在事务, 抛出异常

### **PROPAGATION_REQUIRES_NEW**
> 不管是否存在事务, 都会新建一个事务.

Spring官方解释:
![prop-required](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/tx_prop_requires_new.png)_PROPAGATION_REQUIRES_NEW_

### **PROPAGATION_NOT_SUPPORTED**
> 不支持事务, 不管是否存在事务, 都会以非事务方式执行.

### **PROPAGATION_NEVER**
> 不支持事务, 如果存在事务, 抛出异常. 不存在事务以非事务方式运行.

### **PROPAGATION_NESTED**
> 如果存在事务, 使用内嵌事务, 如果不存在事务, 则新建一个事务.
> 使用内嵌事务: 内嵌事务如果抛出异常, 内嵌事务回滚, 而父事务不会回滚父事务如果抛出异常, 则父事务和内嵌事务都需要回滚.


## Spring隔离级别
`TransactionDefinition`定义了以下几个隔离级别:
- ISOLATION_DEFAULT
- ISOLATION_READ_UNCOMMITTED
- ISOLATION_READ_COMMITTED
- ISOLATION_REPEATABLE_READ
- ISOLATION_SERIALIZABLE

Spring同时定义了`Isolation`枚举类对应以上级别:
```java
public enum Isolation {
	DEFAULT(TransactionDefinition.ISOLATION_DEFAULT),
	READ_UNCOMMITTED(TransactionDefinition.ISOLATION_READ_UNCOMMITTED),
	READ_COMMITTED(TransactionDefinition.ISOLATION_READ_COMMITTED),
	REPEATABLE_READ(TransactionDefinition.ISOLATION_REPEATABLE_READ),
	SERIALIZABLE(TransactionDefinition.ISOLATION_SERIALIZABLE);

	private final int value;

	Isolation(int value) {
		this.value = value;
	}

	public int value() {
		return this.value;
	}
}
```

### ISOLATION_DEFAULT
> 使用数据库默认级别

Oracle: Read committed
Mysql: Repeatable read

Oracle数据库只支持Read committed和Serializable两种隔离等级, 所以Oracle不存在脏读的现象, 但是默认的Read committed 也无法解决不可重复读和幻读的问题.

### ISOLATION_READ_UNCOMMITTED
> 读未提交, 可能会导致脏读

### ISOLATION_READ_COMMITTED
> 读已提交, 不会导致脏读, 但是会存在不可重复读, 和幻读问题

### ISOLATION_REPEATABLE_READ
> 重复读, 解决脏读, 不可重复读, 但还是存在幻读问题

### ISOLATION_SERIALIZABLE
> 串行化, 脏读, 不可重复读, 幻读都可以解决, 通常情况下不会使用该隔离级别


## @Transactional注解
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

	@AliasFor("transactionManager")
	String value() default "";

    // 指定事务管理器
	@AliasFor("value")
	String transactionManager() default "";

    // 定义事务标签, 用于描述事务
	String[] label() default {};

    // 事务传播属性, 默认REQUIRED
	Propagation propagation() default Propagation.REQUIRED;

    // 事务的隔离级别, 默认DEFAULT
	Isolation isolation() default Isolation.DEFAULT;

    // 事务超时时间(秒)
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

	String timeoutString() default "";

    // 如果事务是只读的可以将该标志设置为true, 从而获得运行时优化
    // 并不保证只读事务在进行写事务时会抛出异常.
	boolean readOnly() default false;

    // 定义异常回滚, 表示发生该异常进行回滚
    // 默认情况下发生RuntimeException和Error时会进行回滚, 但是受检的异常不会回滚
	Class<? extends Throwable>[] rollbackFor() default {};

    // 异常回滚全限定类名
	String[] rollbackForClassName() default {};

    // 设置不回滚的异常, 如果异常同时存在于rollbackFor()中, 将会回滚异常.
	Class<? extends Throwable>[] noRollbackFor() default {};

    // 异常不会滚全限定类名
	String[] noRollbackForClassName() default {};
}

```



