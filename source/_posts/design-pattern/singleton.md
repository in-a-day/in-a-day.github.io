---
title: 单例模式
tags:
  - 设计模式
  - gof
categories: 设计模式
date: 2021-01-19 16:16:58
---

## 目的
> 保证一个类只有一个实例.

## 实现方式

### 枚举
最佳实现方式. 提供了反序列化机制.
```java
public enum Singleton {
    INSTANCE;
}
```

### 静态域
通过反射`AccessibleObject.setAccessible`方法仍可以调用私有构造器.
```java
public class Singleton {
    public static final Singleton instance = new Singleton();
    private Singleton() {}
}
```

### 静态工厂方法
同样可以通过反射调用私有构造器
```java
public class Singleton{
    private static final Singleton instance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() {
        return instance;
    }
}
```

### 内部静态类
实现懒加载
```java
public class Singleton{
    private Singleton() {}
    public static class Inner() {
        private static final Singleton instance = new Singleton();
        public static Singleton getInstance() {
            return instance;
        }
    }
}
```

