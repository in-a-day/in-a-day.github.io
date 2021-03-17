---
title: 代理模式
date: 2021-03-17 09:14:59
tags: 
    - 设计模式
    - java
categories: java
---
代理模式对被代理对象添加额外功能.

## 静态代理
看一下静态代理的例子:
首先定义一个简单的接口:
```java
public interface SimpleInterface {
    void func1();
    void func2();
}
```
实现该接口, 作为被代理的角色:
```java
public class SimpleImpl implements SimpleInterface {
    @Override
    public void func1() {
        System.out.println("do something...");
    }

    @Override
    public void func2() {
        System.out.println("do other...");
    }
}
```
实现一个代理类, 该代理类用于在调用SimpleImpl方法前进行日志记录. 该代理类实现`SimpleInterface`接口, 并在内部保存一个`SimpleImpl`对象, 实现`SimpleInterface`接口的方法实际上是调用SimpleImpl对象的实现方法, 并提供一些其他功能.
```java
public class SimpleProxy implements SimpleInterface {
    private SimpleImpl simple;

    public SimpleProxy(SimpleImpl simple) {
        this.simple = simple;
    }

    @Override
    public void func1() {
        doLog();
        simple.func1();
    }

    @Override
    public void func2() {
        doLog();
        simple.func2();
    }

    private void doLog() {
        System.out.println("do log...");
    }
}
```
客户端使用代理类进行功能调用:
```java
public class Client {
    public static void main(String[] args) {
        SimpleInterface simpleInterface = new SimpleImpl();
        // 静态代理
        SimpleProxy simpleProxy = new SimpleProxy(simpleInterface);
        simpleProxy.func1();
        simpleProxy.func2();
}
```


## JDK动态代理
JDK动态代理可以通过Java动态实现代理类. 通过实现InvocationHandler接口, 完成对对象方法调用增强. InvocationHandler接口定义如下:
```java
public interface InvocationHandler {
	/**
     * 调用代理对象的方法, 添加自定义实现, 完成代理
     * @param proxy 调用方法的代理对象
     * @param method 代理对象执行的方法
     * @param args 方法的参数
     * @return 返回方法执行结果
     * @throws Throwable
     */
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

### 使用JDK动态代理实现日志功能
首先定义代理对象实现InvocationHandler接口:
```java
public class JdkDynamicProxy implements InvocationHandler {
    // 被代理的对象
    private Object proxied;

    public JdkDynamicProxy(Object proxied) {
        this.proxied = proxied;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 可以通过判断条件, 指定哪些方法需要增强, 例如只增强名称为func1的方法
        if (Objects.equals("func1", method.getName())) {
            doLog();
        }
        return method.invoke(proxied, args);
    }

    private void doLog() {
        System.out.println("Jdk dynamic proxy log.");
    }

    public static <T> T getJdkProxy(T proxied, Class proxiedInterFace) {
        JdkDynamicProxy proxy = new JdkDynamicProxy(proxied);
        return (T) Proxy.newProxyInstance(proxied.getClass().getClassLoader(), new Class[] {proxiedInterFace}, proxy);
    }
}
```
`method.invoke()`方法接收两个参数: 
- proxied: 被代理的对象, 即哪个对象的方法被调用
- args: 方法的参数

在`getJdkProxy`方法中, 使用了JDK提供的`Proxy.newProxyInstance()`进行代理对象的创建, 该方法接受三个参数:
- ClassLoader loader: 类加载器, 直接使用代理对象的类加载器即可
- Class<?>[] interfaces: 被代理的接口, JDK的动态代理只能进行接口代理
- InvocationHandler h: 代理对象, 即实现了InvocationHandler的对象.

通过`newProxyInstance`方法, Java会动态的生成一个代理类供我们使用. 但是JDK的动态代理仅支持实现接口的方式. 如果想要对类进行代理, 那么需要使用cglib.

客户端测试代码:
```java
public class Client {
    public static void main(String[] args) {
        SimpleInterface simpleInterface = new SimpleImpl();
        // jdk动态代理
        SimpleInterface jdkProxy = JdkDynamicProxy.getJdkProxy(simpleInterface, SimpleInterface.class);
        jdkProxy.func1();
        jdkProxy.func2();
    }
}
```

## CGLIB动态代理
cglib可以对普通类实现生成代理, 实现增强.
首先需要创建Enhancer实例, 然后调用`setSuperclass()`方法设置需要代理的父类, 调用`setCallback()`设置回调类, 最后调用`create()`方法创建一个代理.

cglib创建动态代理:
```java
public class CglibProxy implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        doLog();
        return methodProxy.invokeSuper(o, objects);
    }

    private void doLog() {
        System.out.println("cglib log.");
    }

    public static <T> T getCglibProxy(Class<T> clazz) {
        CglibProxy proxy = new CglibProxy();
        Enhancer enhancer = new Enhancer();
        // 设置父类
        enhancer.setSuperclass(clazz);
        // 设置回调类
        enhancer.setCallback(proxy);
        // 创建代对象
        return (T) enhancer.create();
    }
}
```

客户端测试代码:
```java
public class Client {
    public static void main(String[] args) {
        SimpleInterface simpleInterface = new SimpleImpl();
        // cglib动态代理, 这里的getClass将会得到SimpleImpl类
        SimpleInterface cglibProxy = CglibProxy.getCglibProxy(simpleInterface.getClass());
        cglibProxy.func1();
        cglibProxy.func2();
    }
}
```
















