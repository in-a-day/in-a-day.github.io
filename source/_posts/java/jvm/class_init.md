---
title: Java内存区域
tags:
  - java
  - jvm
categories: java
date: 2021-03-22 10:31:20
---

## 类加载时机
一个类型从被加载到虚拟机内存开始, 到卸载除内存为止, 整个生命周期将会经历`加载(Loading), 验证(Verificatoin), 准备(Preparation), 解析(Resolution), 初始化(Initialization), 使用(Using), 和卸载(Unloading)`七个阶段. 其中验证, 准备, 解析三个部分统称为连接(Linking). 七个阶段发生顺序如下:
![Java类生命周期](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/jvm/jvm_class_lifecycle.png)_Java类生命周期_

加载, 验证, 准备, 初始化和卸载五个阶段的顺序是确定的, 类加载过程必须按照这种顺序**开始(不是进行和完成, 这些阶段通常都是交叉进行的, 会在一个阶段执行的过程中调用, 激活另一个阶段)**. 解析阶段不确定: 某些情况下可以在初始化阶段开始之后再开始, 这是为了支持Java的运行时绑定的特性(动态绑定).

### 初始化阶段
Java虚拟机规范规定了**有且只有**六种情况必须立即对类型进行初始化, 也称这些情况为**主动引用**:
1. 遇到new, getstatic, putstatic, invokestatic这四条字节码指令时, 如果类型没有进行过初始化, 则需要触发其初始化. 能够生成这些指令的典型Java代码场景有:
- 使用new关键字实例化对象
- 读取或设置一个类型的静态字段的时候(被final修饰或已在编译期将结果放入常量池的静态字段除外, 不会触发类型初始化).
- 调用类型的静态方法的时候
2. 使用java.lang.reflect包的方法对类型进行反射调用时, 如果类型尚未初始化, 需要触发其初始化.
3. 当初始化类时, 发现父类尚未初始化, 先进行父类初始化.
4. 当虚拟机启动时, 用户需指定一个要执行的主类(包含main()方法的类), 虚拟机会先初始化这个主类.
5. 
6. 一个接口中定义了默认方法(default关键字修饰), 如果有这个接口的实现类发生了初始化, 那么先初始化该接口(如果不存在default关键字, 将不会初始化该接口).

除以上主动引用之外, 所有引用都不会触发初始化, 称为**被动引用**. 下面看一下被动引用的例子:
1. **通过子类引用父类的静态字段**, 只触发父类初始化, 不触发子类初始化.
```java
// 父类
public class SuperClass{
    static {
        System.out.println("init super");
    }
  	public static int desc = "super class";
}

// 子类
public class SubClass extends SuperClass{
    static {
        System.out.println("init sub");
    }
}

// 测试
public class Test {
    public static void main(String[] args) {
		// 只会打印出init super
        String desc = SubClass.desc;
    }
}
```

2. **通过数组定义来引用类**, 不会触发类的初始化
```java
// 父类还是使用1中代码
public class Test {
    public static void main(String[] args) {
		SuperClass[] s = new SuperClass[10];

    }
}
```

3. **编译期存入调用类的常量池**, 本质上没有引用到定义常量的类, 不会触发初始化
```java
public class ConstClass {
    static {
        System.out.println("const class");
    }

	// 定义常量
    public static final String desc =  "const";
}

// 不会有任何输出
public class Test {
    public static void main(String[] args) {
        String desc = ConstClass.desc;
    }
}
```


