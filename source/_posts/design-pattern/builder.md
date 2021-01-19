---
title: 构造器模式
date: 2021-01-19 11:42:09
tags:
    - 设计模式
    - gof
    - 创建型
categories: 设计模式
---

### 目的

> 将复杂对象的表示与其构造分离, 以便同样的构造过程可以创建不同的表示.
通常构造函数有大量参数时, 使用Builder模式进行构造该对象.

### 示例

构造一个卡通人物.

- Cartoon类及Builder
```java
public class Cartoon {
    private String name;
    private int age;
    private double height;
    private double weigth;
    private int sex;

    private Cartoon(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.height = builder.height;
        this.weigth = builder.weigth;
        this.sex = builder.sex;
    }

    public static class Builder() {
        private String name;
        private int age;
        private double height;
        private double weigth;
        private int sex;
        public Builder(String name) {
            this.name = name;
            retrun this;
        }
        
        public Builder age(int age) {
            this.age = age;
            retrun this;
        }

        public Builder height(double height) {
            this.height = height;
            retrun this;
        }

        public Builder weight(double weight) {
            this.weight = weight;
            retrun this;
        }

        public Builder sex(int sex) {
            this.sex = sex;
            retrun this;
        }

        public Cartoon build() {
            retrun new Cartoon(this);
        }
    }
}
```

- 客户端代码
```java
var c = new Cartoon.Builder("z").age(10).height(120.1).build();
```


### Java示例
[java.long.StringBuilder](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html)


### 参考

- [java-design-patterns](https://java-design-patterns.com/patterns/builder/)

