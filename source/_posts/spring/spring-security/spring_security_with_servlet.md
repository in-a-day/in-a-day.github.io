---
title: Spring Security在Servlet中的使用
tags:
  - java
  - spring
  - spring security
categories: spring
date: 2021-03-24 15:10:00
---
> Spring Security通过使用标准的Servlet`Filter`整合了Servlet容器. 这也就意味着它可以运行在任意的Servlet容器中. 更具体的来说, 不必在基于Servlet的应用中使用Spring就可以使用Spring Security.

### Filter概览
Spring Security基于Servlet的`Filter`, 下面看一下一个典型的单个HTTP请求的处理层级.
![FilterChain](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/filterchain.png)_FilterChain_
客户端发送一个请求到应用, 容器创建一个`FilterChain`, 其中包含了`Filter`和`Servlet`用于处理基于请求URL路径的`HttpServletRequest`. 在Spring MVC中该Servlet是`DispatchServlet`的一个实例. 最多使用一个`Servlet`处理一个`HttpServletRequest`和`HttpServletResponse`. 然而可以有多个`Filter`用于以下:
- 阻止下游(downstream) `Filter`或`Servlet`被调用. 
- 使用下游`Filter`和`Servlet`修改`HttpServletRequest`或`HttpServletResponse`.

FilterChain示例:
```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // do something before the rest of the application
    chain.doFilter(request, response); // invoke the rest of the application
    // do something after the rest of the application
}
```
因为`Filter`会影响下游的`Filter`和`Servlet`, 所以Filter的执行顺序极其重要.

## **DelegatingFilterProxy**
Spring提供了一个`Filter`的实现`DelegatingFilterProxy`, 该实现连接了Servlet容器生命周期和Spring的`ApplicationContext`. Servlet容器允许使用其标准注册`Filter`, 但是无法知晓Spring定义的Bean(即Spring管理的Bean实现了Filter接口, 但是未使用Servlet标准进行注册). `DelegatingFilterProxy`可以通过标准的Servlet容器技术注册`Filter`, 同时将任务委派给实现了`Filter`接口的Spring Bean.

下图展示了`DelegatingFilterProxy`如何适配`Filter`和`FilterChain`:
![DelegatingFilterProxy](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/delegatingfilterproxy.png)_DelegatingFilterProxy_
`DelegatingFilterProxy`从`Application`中查找Bean Filter<sub>0</sub>. 以下伪代码展示了`DelegatingFilterProxy`的使用:
```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // Lazily get Filter that was registered as a Spring Bean
    // For the example in DelegatingFilterProxy delegate is an instance of Bean Filter0
    Filter delegate = getFilterBean(someBeanName);
    // delegate work to the Spring Bean
    delegate.doFilter(request, response);
}
```
使用`DelegatingFilterProxy`的另一个好处是允许`Filter`Bean的延迟查找. 这一点十分重要, 因为容器需要在启动之前对`Filter`实例j进行注册. 然而Spring通常在`Filter`实例注册完成后才会使用`ContextLoaderListener`完成Spring Bean加载.


## **FilterChainProxy**
Spring Security的Servlet支持包含于`FilterChainProxy`. `FilterChainProxy`是Spring Security提供的一个特殊的`Filter`, 允许通过SecurityFilterChain委托给许多`Filter`实例. 因为`FilterChainProxy`是一个Bean, 所以通常使用`DelegatingFilterProxy`装饰.
![FilterChainProxy](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/filterchainproxy.png)_FilterChainProxy_


## **SecurityFilterChain**
`SecurityFilterChain`用于`FilterChainProxy`决定哪些Spring Security的`Filter`需要在此次请求中调用.

![SecurityFilterChain](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/securityfilterchain.png)_SecurityFilterChain_

`SecurityFilterChain`中的`Security Filters`通常都是Bean, 但是他们由`FilterChainProxy`注册而不是`DelegatingFilterChain`. 相比于通过`DelegatingFilterProxy`或者Servlet容器注册来说, 使用`FilterChainProxy`提供了大量的好处:
- 首先, 它为Spring Security的Servlet支持提供了一个开始点. 因此, 如果你想要去排除Spring Security的容器支持故障, 就可以在`FilterChainProxy`中打断点.
- 第二, 因为`FilterChainProxy`是Spring Security执行的核心, 所以它可以执行被视为不可选的任务(即该任务必须执行?). 例如, 它清除`SecurityContext`避免内存泄漏. 它也应用Spring Security的`HttpFirewall`保护应用免遭特定类型的攻击.
- 此外, 它提供了更为灵活的方式决定一个`SecurityFilterChain`是否应该被调用. 在Servlet容器中, `Fitler`是基于URL调用的. 然而`FilterChainProxy`可以通过利用`RequestMathcer`接口, 基于`HttpServletRequest`中任何信息决定调用.

实际上, `FilterChainProxy`可以用于决定执行哪一个`SecurityFilterChain`.
![Multiple SecurityFilterChain](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/securityfilterchain.png)_Multiple SecurityFilterChain_





