---
title: Spring Security Filter使用(5.4.5)
tags:
  - java
  - spring
  - spring security
categories: spring
date: 2021-03-24 15:10:00
---
> Spring Security通过使用标准的Servlet`Filter`整合了Servlet容器. 这也就意味着它可以运行在任意的Servlet容器中. 更具体的来说, 不必在基于Servlet的应用中使用Spring就可以使用Spring Security.

以下将介绍这些内容:
- Filter, Servlet中的Filter及FilterChain
- DelegatingFilterChainProxy
- FilterChainProxy
- SecurityFilterChain
- Security Filters


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
上图中, `FilterChainProxy`决定了哪一个`SecurityFilterChain`该被使用. 只有第一个匹配的`SecurityFilterChain`才会被调用. 如果请求`/api/messages`URL, 那么SecurityFilterChain<sub>0</sub>将会匹配(由于其模式是`/api/**`), 所以只有SecurityFilterChain<sub>0</sub>会被调用. 如果请求的URL是`/message/`, 那么SecurityFilterChain<sub>0</sub>不会匹配, 所以`FilterChainProxy`会继续尝试调用每个`SecurityFilterChain`. 如果没有其他的`SecurityFilterChain`匹配, 最后匹配的SecurityFilterChain<sub>n</sub>将会被调用.

**NB:每个`SecurityFilterChain`都可以是唯一的, 并且可以单配置.** 事实上, 一个`SecurityFilterChain`可能有0个security `Filter`(如果应用希望Spring Security忽略特定的请求).


## **Security Filters**
Security Filters通过SecurityFilterChain API插入FilterChainProxy. Filters的顺序十分重要. 通常没有必要去了解Spring Security Filters的顺序. 然而有时候还是有必要去知道这些顺序的. 以下是完成的Spring Security Filter 排序:
``` text
ChannelProcessingFilter

WebAsyncManagerIntegrationFilter

SecurityContextPersistenceFilter

HeaderWriterFilter

CorsFilter

CsrfFilter

LogoutFilter

OAuth2AuthorizationRequestRedirectFilter

Saml2WebSsoAuthenticationRequestFilter

X509AuthenticationFilter

AbstractPreAuthenticatedProcessingFilter

CasAuthenticationFilter

OAuth2LoginAuthenticationFilter

Saml2WebSsoAuthenticationFilter

UsernamePasswordAuthenticationFilter

OpenIDAuthenticationFilter

DefaultLoginPageGeneratingFilter

DefaultLogoutPageGeneratingFilter

ConcurrentSessionFilter

DigestAuthenticationFilter

BearerTokenAuthenticationFilter

BasicAuthenticationFilter

RequestCacheAwareFilter

SecurityContextHolderAwareRequestFilter

JaasApiIntegrationFilter

RememberMeAuthenticationFilter

AnonymousAuthenticationFilter

OAuth2AuthorizationCodeGrantFilter

SessionManagementFilter

ExceptionTranslationFilter

FilterSecurityInterceptor

SwitchUserFilter
```

## **处理Security异常**
`ExceptionTranslationFilter`允许将`AccessDeniedException`和`AuthenticalException`转换为Http 响应. `ExceptionTranslationFilter`作为`SecurityFilters`之一被插入到`FilterChainProxy`中.

![ExceptionTranslationFilter](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/exceptiontranslationfilter.png)_ExceptionTranslationFilter_
- 首先, `ExceptionTranslationFilter`调用`FilterChain.doFilter(request, response)`去调用应用的剩余部分.
- 如果用户未认证或抛出AuthenticationException异常, 开始进行认证:
    - 清空`SecurityContextHolder`
    - 将`HttpServletRequest`保存到`RequestCache`中. 当用户成功认证后, 将`RequestCache`内容重播原始request.
    - `AuthenticationEntryPoint`用于从客户端请求凭证(credential). 例如, 它可能重定向到登录页面或者发送一个`WWW-Authenticat`头部(header).
- 否则, 如果产生`AccessDeniedException`, 那么拒绝访问. 调用`AccessHandler`处理决绝访问.

**NB: 如果应用不抛出`AccessDeniedException`或`AuthenticationException`异常, 那么`ExceptionTranslationFilter`什么都不会做.** 

上述`ExceptionTranslationFilter`伪代码如下:
```java
try {
    filterChain.doFilter(request, response);  // 1
} catch (AccessDeniedException | AuthenticationException ex) {
    if (!authentication || ex instanceof AuthenticationException) {
        startAuthentication();  // 2
    } else {
        accessDenied();  // 3
    }
}
```
- 1: 意味着如果应用的其他部分抛出了`AccessDeniedException`或`AuthenticationException`异常, 将在此处捕获
- 2: 如果用户尚未认证, 或者抛出的是AuthenticationException, 那么开始认证.
- 3: 其他情况, 拒绝访问.

