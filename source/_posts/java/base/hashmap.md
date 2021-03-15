---
title: HashMap详解
date: 2021-03-15 14:26:55
tags:
    - java
categories: java
---

## HashMap
### **HashMap中的hash方法的作用**
HashMap中提供了`hash()`函数在调用`key.hashCode()`得到原始哈希码之后, 再进行一次hash, 称之为扰动函数. 将key的高16位与低16位进行异或, 减少hash冲突几率.  
HashMap中设置key和获得key都是通过`(n - 1) & hash`进行的(相当于hash%n, 但是效率较高), 其中n是HashTable的大小. 这样进行计算, 如果只取后几位, 哈希碰撞的概率比较高. 而使用了扰动函数, 混合了原始哈希码的高位和低位, 加大了低位的随机性, 降低了碰撞几率.

JDK1.8 hash函数源码:

```java
static final int hash(Object key) {
	int h;
	return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

### **HashMap属性及其意义**
- DEFAULT_INITIAL_CAPACIT = 1 <<< 4, 默认的HashMap容量.
- MAXIMUM_CAPACITY = 1 << 30, 最大容量.
- DEFAULT_LOAD_FACTOR = 0.75f, 装载因子
- TREEIFY_THRESHOLD = 8, 链表转红黑树的阈值
- UNTREEIFY_THRESHOLD = 6, 红黑树转链表的阈值
- MIN_TREEIFY_CAPACITY = 64, 链表转红黑树的最小容量值, 当map的容量小于该值时, 即使hash相同的链表长度大于8也不会转换成红黑树, 而是进行扩容.
- loadFactor, 实际的装载因子
- size, map的实际长度, 即当前map中存在size个key-value. 当`size > threshold`时, 将会触发扩容.
- threshold, 扩容的阈值, `threshold = capacity * loadFactor`
- capacity, map的容量

### **为什么HashMap的长度是2^n?**
使用hashcode & (length - 1)取模效率高于 hashcode % length. 同时也解决了hashcode是负数的问题.

### **如何保证长度是2^n?**
```java
static final int tableSizeFor(int cap) {
	int n = cap - 1;
	// n无符号右移一位 | n, 这样如果n存在一个1那么该1的下一个位置也变成了1
	// 假设n二进制是: 0100 0000(省略了前面的24个0), 则该操作后n为: 0110 0000
	n |= n >>> 1;
	// 同样操作, 一共至少四位完成了1的复制
	n |= n >>> 2;
	// 接着是8位
	n |= n >>> 4;
	// 然后是16位
	n |= n >>> 8;
	// 32位全都完成
	n |= n >>> 16;
	return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```


