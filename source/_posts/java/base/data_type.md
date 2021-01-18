---
title: java基本数据类型
date: 2021-01-18 14:23 
tags: java-base 
categories: java
---

### 基本数据类型

类型 | 大小
--- | ---
boolean | true, false
byte | 8bit, [-128, 127]
char | unsigned 16bit, $ [0, 2^{16}-1] $
short | 16bit, $ [$-$2^{15}, 2^{15}-1] $
int | 32bit, $ [$-$2^{31}, 2^{31}-1] $
long | 64bit, $ [$-$2^{65}, 2^{65}-1] $
float | 32bit
double | 64bit


### 包装类型

基本类型 | 包装类型
--- | ---
boolean | Boolean
byte | Byte
char | Character
short | Short
int | Integer
long | Long
float | Float
double | Double

> Java自动装箱通过包装类的valueOf()方法实现. 自动拆箱通过包装类对象的xxxValue()实现.

**NB: Java三目运算符如果同时存在基本类型和包装类型会进行自动拆箱操作.**
```java
Integer a = null;
int b = 1;
// 自动拆箱抛出null pointer异常, Integer.valueOf(a.intValue())
int c = true ? a : b;
// 自动拆箱抛出null pointer异常, Integer.valueOf(a.intValue())
Integer d = true ? a : b;
```

### 包装类型缓存

Integer 缓存: Integer类使用IntegerCache内部类进行缓存, 最大值可以通过jvm属性调整.(只有Integer支持调整最大值, 其他包装类型不支持)
Double 与 Float包装类不存在缓存.

#### 生成Integer方法

1. valueOf()系列, 使用缓存
```java
// jdk源码
public static Integer valueOf(int i) {
	if (i >= IntegerCache.low && i <= IntegerCache.high)
		return IntegerCache.cache[i + (-IntegerCache.low)];
	return new Integer(i);
}

// 测试
Integer c = Integer.valueOf(2);
Integer d = Integer.valueOf(2);
// true
System.out.println(a == b);
c = Integer.valueOf(222);
d = Integer.valueOf(222);
// false
System.out.println(a == b);
```

2. new Integer(), 未使用缓存
```java
// jdk源码
public Integer(int value) {
	this.value = value;
}

public Integer(String s) throws NumberFormatException {
	this.value = parseInt(s, 10);
}

// 测试
Integer a = new Integer(1);
Integer b = new Integer(1);
// false
System.out.println(a == b);

```

3. 自动装箱(由于自动装箱调用的是Integer.valueOf()方法所以结果与直接调用valueOf()相同)
```java
Integer e = 123;
Integer f = 123;
// true
System.out.println(e == f);
Integer g = 222;
Integer h = 222;
// false (未修改jvm Integer类型缓存上限时)
System.out.println(g == h);
```

#### IntegerCache源码
```java
private static class IntegerCache {
	static final int low = -128;
	static final int high;
	static final Integer cache[];

	static {
		// high value may be configured by property
		int h = 127;
		String integerCacheHighPropValue =
			sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
		if (integerCacheHighPropValue != null) {
			try {
				int i = parseInt(integerCacheHighPropValue);
				i = Math.max(i, 127);
				// Maximum array size is Integer.MAX_VALUE
				h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
			} catch( NumberFormatException nfe) {
				// If the property cannot be parsed into an int, ignore it.
			}
		}
		high = h;

		cache = new Integer[(high - low) + 1];
		int j = low;
		for(int k = 0; k < cache.length; k++)
			cache[k] = new Integer(j++);

		// range [-128, 127] must be interned (JLS7 5.1.7)
		assert IntegerCache.high >= 127;
	}

	private IntegerCache() {}
}

```


### boolean变量getter/setter规范
> 普通参数getter/setter
```java
public <PropertyType> get<PropertyName>();
public void set<PropertyName>(<PropertyType> a);
```

> boolean变量getter/setter
```java
public boolean is<PropertyName>();
public void set<PropertyName>(boolean m);
```
**NB: 在pojo中使用boolean不要使用isXXX形式, 使用XXX**

> 使用isXXX形式的boolean类型在不同序列化工具中可能会产生不同的结果.

### Double, Float
```java
/**
 * A constant holding the positive infinity of type
 * {@code double}. It is equal to the value returned by
 * {@code Double.longBitsToDouble(0x7ff0000000000000L)}.
 */
public static final double POSITIVE_INFINITY = 1.0 / 0.0;

/**
 * A constant holding the negative infinity of type
 * {@code double}. It is equal to the value returned by
 * {@code Double.longBitsToDouble(0xfff0000000000000L)}.
 */
public static final double NEGATIVE_INFINITY = -1.0 / 0.0;

/**
 * A constant holding a Not-a-Number (NaN) value of type
 * {@code double}. It is equivalent to the value returned by
 * {@code Double.longBitsToDouble(0x7ff8000000000000L)}.
 */
public static final double NaN = 0.0d / 0.0;
```

```java
/**
 * A constant holding the positive infinity of type
 * {@code float}. It is equal to the value returned by
 * {@code Float.intBitsToFloat(0x7f800000)}.
 */
public static final float POSITIVE_INFINITY = 1.0f / 0.0f;

/**
 * A constant holding the negative infinity of type
 * {@code float}. It is equal to the value returned by
 * {@code Float.intBitsToFloat(0xff800000)}.
 */
public static final float NEGATIVE_INFINITY = -1.0f / 0.0f;

/**
 * A constant holding a Not-a-Number (NaN) value of type
 * {@code float}.  It is equivalent to the value returned by
 * {@code Float.intBitsToFloat(0x7fc00000)}.
 */
public static final float NaN = 0.0f / 0.0f;
```
