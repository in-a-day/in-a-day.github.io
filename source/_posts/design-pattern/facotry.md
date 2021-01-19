---
title: 工厂模式
date: 2021-01-18 16:37:12
tags: [设计模式, gof, 创建型]
categories: 设计模式
---
> 又称简单工厂, 静态工厂方法.

## 目的
> 提供封装在工厂类中的静态方法, 隐藏创建对象的实现逻辑, 使用者仅需关心对象的使用信息而不是对象的实例化.

## 示例
> 通过music工厂创建music

+ music接口及其实现
```java
public interface Music {
    String getType();
}

public class Jazz implements Music {
    private static final String TYPE = "JAZZ MUSIC";
    @Override
    public String getType() {
        return TYPE;
    }
}

public class Rock implements Music {
    private static final String TYPE = "ROCK MUSIC"
    @Override
    public String getType() {
        return TYPE;
    }
}
```

+ 枚举Music的类型
```java
public enum MusicType {
    JAZZ(Jazz::new),
    ROCK(Rock::new);

    private final Supplier<Music> constructor;

    MusicType(Supplier<Music> constructor) {
        this.constructor = constructor;
    }

    public Supplier<Music> getConstructor() {
        return this.constructor;
    }
}
```

+ 创建Music的工厂MusicFactory
```java
public class MusicFactory {
    public static Music getMusic(MusicType type) {
        return type.getConstructor().get();
    }
}
```

+ 客户端使用代码
```java
var jazz = MusicFactory.getMusic(MusicType.JAZZ);
var rock = MusicFactory.getMusic(MusicType.ROCK);
log.info(jazz.getType());
log.info(rock.getType());
```
+ 输出
```java
JAZZ TYPE
ROCK TYPE
```

