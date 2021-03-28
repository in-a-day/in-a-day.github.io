---
title: Spring Security Authentication(认证)(5.4.5)
tags:
  - java
  - spring
  - spring security
categories: spring
date: 2021-03-24 23:08:00
---
Spring Security为认证提供了全面的支持. 接下来从这两个方面进行介绍:
## **架构组件(Architecture Components)**
主要描述Spring Security用于Servlet认证中的架构组件. 主要有以下组件:
- `SecurityContextHolder`: 存储身份验证详细信息的地方.
- `SecurityContext`: 由`SecurityContextHolder`获得,包含了当前认证客户的`Authentication`.
- `Authentication`: 可以是`AuthenticationManager`的输入, 以提供用户提供的用于身份验证的凭据或来自SecurityContext的当前用户.
- `GrantedAuthority`: 授予身份验证主体的权限(即角色, 范围等).
- `AuthenticationManager`: 定义了Spring Security的Filters如何进行认证的API.
- `ProviderManager`: `AuthenticationManager`的一个通用实现.
- `Request Credentails with AuthenticationEntryPoint`: 用于从客户端请求凭证(即重定向到登录页面, 发送WWW-Authenticate响应等).
- `AbstractAuthenticationProcessingFilter`: 用于认证的基础Filter. 这样为高级别的身份认证流程和各个部件如何协同工作提供了一个好的概念.

## **认证机制(Authentication Mechanisms)**
- Username and Password - 如何使用用户名密码认证
- OAuth 2.0 Login - OAhth 2.0 使用OpenID Connect登录和非标准的OAuth 2.0 登录(即GitHub)
- SAML 2.0 Login - SAML 2.0 登录
- Central Authentication Server (CAS) - CAS支持
- Remember Me - 如何记住用户
- JAAS Authentication - JAAS认证
- OpenID - OpenID 认证 (不要与OpenID Connect弄混)
- Pre-Authentication Scenarios - 使用外部机制进行认证(e.g. SiteMider或J2EE security), 但仍使用Spring Security授权和防范常见漏洞攻击.
- X509 Authentication - X509 认证

## **SecurityContextHolder**
`SecurityContextHolder`是Spring Security认证模型的核心. 它包含了`SecurityContext`.
![SecurityContextHolder](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/securitycontextholder.png)_SecurityContextHolder_
上图可以看出, `SecurityContextHolder`中包含了`SecurityContext`, `SecurityContext`中包含了`Authentication`, `Authentication`中包含了`Principal, Credentials, Authorities`.

`SecurityContextHolder`存储了身份认证的详细信息. Spring Security不关心`SecurityContextHolder`是如何填充的. 如果他包含了值, 就将它当做当前认证的用户.

表示客户已经通过认证的最简单的方式是直接设置`SecurityContextHolder`. 下面展示了如何使用:
```java
SecurityContext context = SecurityContextHolder.createEmptyContext();  // 1
Authentication authentication = new TestingAuthenticationToken("name", "pwd", "ROLE_USER");  // 2
context.setAuthentication(authentication);

SecurityContextHolder.setContext(context);  // 3
```
- 1. 此处创建一个新的`SecurityContext`而不是使用`SecurityContextHolder.getContext().setAuthentication(authentication)`, 避免了多线程的条件竞争.
- 2. 接着创建一个新的`Authentication`对象. Spring Security不关心在`SecurityContext`中设置什么类型的`Authentication`实现. 此处使用测试用的`TestingAuthenticationToken`(实现简单). 通常情况下生产环境使用`UsernamePasswordAuthenticationToken(userDetails, password, authorities)`.
- 3. 最后在`SecurityContextHolder`中设置`SecurityContext`.

通过`SecurityContextHolder`获取已验证的principal.
```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```

默认情况下`SecurityContextHolder`使用`ThreadLocal`存储这些详细信息, 这意味着`SecurityContext`对同一线程中的方法都是可用的, 即使`SecurityContext`没有显示地作为参数传递给这些方法. Spring Security的`FilterChainProxy`确保`SecurityContext`在线程请求后正确的被清理, 所以使用以这种方式使用`ThreadLocal`是安全的.

有些应用不适合使用`ThreadLocal`(例如Swing), 所以`SecurityContextHolder`可以在启动时配置一个策略以执行上下文如何存储. 包含以下策略:
- MODE_THREADLOCAL
- MODE_INHERITABLETHREADLOCAL
- MODE_GLOBAL

`SecurityContextHolder`源码如下:
```java
public class SecurityContextHolder {

	public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";

	public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";

	public static final String MODE_GLOBAL = "MODE_GLOBAL";

	public static final String SYSTEM_PROPERTY = "spring.security.strategy";

	private static String strategyName = System.getProperty(SYSTEM_PROPERTY);

	private static SecurityContextHolderStrategy strategy;

	private static int initializeCount = 0;

	static {
		initialize();
	}

	// 初始化strategy
	private static void initialize() {
		if (!StringUtils.hasText(strategyName)) {
			// Set default
			strategyName = MODE_THREADLOCAL;
		}
		if (strategyName.equals(MODE_THREADLOCAL)) {
			strategy = new ThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
			strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_GLOBAL)) {
			strategy = new GlobalSecurityContextHolderStrategy();
		}
		else {
			// Try to load a custom strategy
			try {
				Class<?> clazz = Class.forName(strategyName);
				Constructor<?> customStrategy = clazz.getConstructor();
				strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
			}
			catch (Exception ex) {
				ReflectionUtils.handleReflectionException(ex);
			}
		}
		initializeCount++;
	}

	public static void clearContext() {
		strategy.clearContext();
	}

	public static SecurityContext getContext() {
		return strategy.getContext();
	}

	public static int getInitializeCount() {
		return initializeCount;
	}

	public static void setContext(SecurityContext context) {
		strategy.setContext(context);
	}

	public static void setStrategyName(String strategyName) {
		SecurityContextHolder.strategyName = strategyName;
		initialize();
	}

	public static SecurityContextHolderStrategy getContextHolderStrategy() {
		return strategy;
	}

	public static SecurityContext createEmptyContext() {
		return strategy.createEmptyContext();
	}

	@Override
	public String toString() {
		return "SecurityContextHolder[strategy='" + strategyName + "'; initializeCount=" + initializeCount + "]";
	}

}
```
从源码中可以看出, `SercurityContextHolder`内部使用`SecurityContextHolderStrategy`执行具体的逻辑. 接下来看一下`SecurityContextHolderStrategy`源码:
```java
public interface SecurityContextHolderStrategy {
	// 清空当前context
	void clearContext();

	// 获取当前context
	SecurityContext getContext();
	
	// 设置当前context
	void setContext(SecurityContext context);
		
	// 创建新的空的context实现
	SecurityContext createEmptyContext();
}
```
接下来看一下常用的一个实现`ThreadLocalSecurityContextHolderStrategy`:
```java
final class ThreadLocalSecurityContextHolderStrategy implements
		SecurityContextHolderStrategy {

	// 使用ThreadLocal保存SecurityContext
	private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();

	// ~ Methods
	// ========================================================================================================

	public void clearContext() {
		contextHolder.remove();
	}

	public SecurityContext getContext() {
		SecurityContext ctx = contextHolder.get();

		if (ctx == null) {
			ctx = createEmptyContext();
			contextHolder.set(ctx);
		}

		return ctx;
	}

	public void setContext(SecurityContext context) {
		Assert.notNull(context, "Only non-null SecurityContext instances are permitted");
		contextHolder.set(context);
	}

	public SecurityContext createEmptyContext() {
		return new SecurityContextImpl();
	}
}
```


## **SecurityContext**
> `SecurityContext`可以从`SecurityContextHolder`中获得. 其包含了一个`Authenticaiton`对象.

源码如下:
```java
public interface SecurityContext extends Serializable {

	Authentication getAuthentication();

	void setAuthentication(Authentication authentication);

}
```

## **Authentication**
`Authentication`在Spring Security中主要有以下两个目的:
- 作为`AuthenticationManager`的输入, 提供用户提供的身份认证凭据. 这种情况下, `isAuthenticated()`将返回`false`.
- 表示当前认证的用户. 可以从`SecurityContext`中获取当前的`Authentication`.

`Authentication`主要包含以下属性:
- `principal` - 标识用户. 当使用用户名/密码进行身份认证时, 通常是一个`UserDetails`的一个实例.
- `credentials` - 通常是密码. 多数情况下, 在用户完成身份认证后会被清除, 以保证不会泄露.
- `authorities` - `GrantedAuthority`是用户被授予的高级权限. 例如角色或领域(roles or scops).

`Authentication`源码:
```java
public interface Authentication extends Principal, Serializable {

	Collection<? extends GrantedAuthority> getAuthorities();
	// 获取凭证
	Object getCredentials();
	// 获取认证请求详细信息, 可能是ip地址, 证书编号等
	Object getDetails();
	// 获取主体信息, 通常会是UserDetails
	Object getPrincipal();
	// 是否验证成功
	boolean isAuthenticated();
	
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;

}
```

## **GrantedAuthority**
`GrantedAuthority`是用户被授予的高级权限. 例如roles或scopes.

通过`Authentication.getAuthorities()`可以获得`GrantedAuthority`集合. `GrantedAuthority`就是授予用户的权限. 通常称这些权限为角色, 例如`ROLE_ADMINISTRATOR, ROLE_HR_SUPERVISOR`. 这些角色可以用于配置web授权, 方法授权和domain对象授权. 使用用户名/密码方式认证的`GrantedAuthority`通常由`UserDetailService`加载. 

源码如下:
```java
public interface GrantedAuthority extends Serializable {

	String getAuthority();

}
```

## **AuthenticationManager**
`AuthenticationManager`是定义Spring Security的Filter如何执行认证的API. 在控制器中(即Spring Security的Filter)中调用`AuthenticationManager`将返回的`Authentication`设置到`SecurityContextHolder`中. 如果没有集成Spring Security’s Filters, 也可以直接设置`SecurityContextHolder`而不适用`AuthenticationManager`.

尽管`AuthenticationManager`的实现可以是任意的, 但是最通用的实现是`ProviderManager`.

源码如下:
```java
public interface AuthenticationManager {

	Authentication authenticate(Authentication authentication) throws AuthenticationException;

}
```

## **ProviderManager**
`ProviderManager`是`AuthenticationManager`的一个最常用的实现. `ProviderManager`委托给`AuthenticationProvider`列表进行认证. 每个AuthenticationProvider都有机会指示身份验证应该是成功的、失败的，或者指示它不能做出决定，并允许下游的AuthenticationProvider做出决定.如果配置的authenticationprovider中没有一个可以进行身份验证，那么身份验证将抛出ProviderNotFoundException异常，这是一个特殊的AuthenticationException，它指示ProviderManager不支持传入的`Authentication`类型.

![ProviderManager](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/providermanager.png)_ProviderManager_

实际上，每个AuthenticationProvider都知道如何执行特定类型的身份验证. 例如，一个`AuthenticationProvider`可能能够验证用户名/密码，而另一个可能能够验证SAML断言。 这允许每个AuthenticationProvider执行非常特定类型的身份验证，同时支持多种身份验证类型，并且只暴露出单个AuthenticationManager bean。

ProviderManager还允许配置一个可选的父AuthenticationManager，当AuthenticationProvider无法执行身份验证时，该父AuthenticationManager会被使用.父类可以是任何类型的AuthenticationManager，但它通常是ProviderManager的一个实例.

![ParentProviderManager](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/providermanager-parent.png)_ParentProviderManager_

事实上，多个ProviderManager实例可能共享相同的父AuthenticationManager。这在多个SecurityFilterChain实例中比较常见，这些实例有一些共同的身份验证(共享的父AuthenticationManager)，但也有不同的身份验证机制(不同的ProviderManager实例)。

![ShareParentProviderManager](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/providermanagers-parent.png)_ShareParentProviderManager_

默认情况下，ProviderManager将尝试从成功的身份验证请求返回的身份验证对象中清除任何敏感的凭据信息。这可以防止诸如密码之类的信息在HttpSession中被保留的时间超过必要的时间。

下面看一下ProviderManager的构造方法:

```java
	public ProviderManager(AuthenticationProvider... providers) {
		this(Arrays.asList(providers), null);
	}

	/**
	 * Construct a {@link ProviderManager} using the given {@link AuthenticationProvider}s
	 * @param providers the {@link AuthenticationProvider}s to use
	 */
	public ProviderManager(List<AuthenticationProvider> providers) {
		this(providers, null);
	}

	/**
	 * Construct a {@link ProviderManager} using the provided parameters
	 * @param providers the {@link AuthenticationProvider}s to use
	 * @param parent a parent {@link AuthenticationManager} to fall back to
	 */
	public ProviderManager(List<AuthenticationProvider> providers, AuthenticationManager parent) {
		Assert.notNull(providers, "providers list cannot be null");
		this.providers = providers;
		this.parent = parent;
		checkState();
	}
```

不难看出, ProviderManager可以接受一系列的AuthenticationProvider.

接着看一下ProviderManager是如何具体实现Authentication的authenticate方法(只截取部分):

```java
// 循环每个provider	
for (AuthenticationProvider provider : getProviders()) {
  // 先调用support方法测试该provider是否支持验证toTest
  if (!provider.supports(toTest)) {
    continue;
  }

  // ... 省略一些内容

  try {
    // 调用provider执行验证, 只要有一个验证通过了就返回
    result = provider.authenticate(authentication);
    if (result != null) {
      copyDetails(authentication, result);
      break;
    }
  }
  catch (AccountStatusException | InternalAuthenticationServiceException ex) {
    prepareException(ex, authentication);
    throw ex;
  }
  catch (AuthenticationException ex) {
    lastException = ex;
  }
}
// 如果结果为空, 且存在父AuthenticationManager, 使用父AuthenticationManager执行验证
if (result == null && this.parent != null) {
  try {
    parentResult = this.parent.authenticate(authentication);
    result = parentResult;
  }
  catch (ProviderNotFoundException ex) {
  }
  catch (AuthenticationException ex) {
    parentException = ex;
    lastException = ex;
  }
}
 
// 验证成功, 清除敏感信息, 例如密码等
if (result != null) {
  if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) {
    ((CredentialsContainer) result).eraseCredentials();
  }
	// ...省略一些代码
  return result;
}

// Parent was null, or didn't authenticate (or throw an exception).
if (lastException == null) {
  lastException = new ProviderNotFoundException(this.messages.getMessage("ProviderManager.providerNotFound",
                                                                         new Object[] { toTest.getName() }, "No AuthenticationProvider found for {0}"));
}
// ... 省略一些代码
// 如果验证未通过, 最后抛出异常
throw lastException;

```



## **AuthenticationProvider**
`ProviderManager`可以注入多个`AuthenticationProvider`. 每个`AuthenticationProvider`执行特定类型的认证. 例如`DaoAuthenticationProvider`支持基于用户名/密码的认证, `JwtAuthenticationProvider`支持使用JWT token认证.

`AuthenticationProvider`接口定义了两个方法:

```java
public interface AuthenticationProvider {
  // 验证身份, 如果不支持该authentication类型验证, 则可以返回nullm, 如果验证失败抛出异常
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
	// 测试该provider是否支持验证给定的authentication
	boolean supports(Class<?> authentication);
}

```

下面看一下常用的`AbstractUserDetailsAuthenticationProvider`(`DaoAuthenticationProvider`就继承自该provider)的authenticate流程:

```java

```





## **使用AuthenticationEntryPoint请求凭证**
`AuthenticationEntryPoint`用于发送从客户端请求凭证的HTTP响应.

有时客户端会主动携带凭证(例如用户名/密码)去请求资源. 在这种情况下, Spring Security不需要再向客户端发送请求凭证的HTTP响应.

其他情况下, 客户端可能会发送一个未经认证的获取资源请求. 在这种情况下, 使用`AuthenticationEntryPoint`向客户端请求凭证. `AuthenticationEntryPoint`可能会重定向到登录页面, 使用`WWW-Authenticate`头部响应.

## **AbstractAuthenticationProcessingFilter**
`AbstractAuthenticationProcessingFilter`作为基础的Filter对用户的凭证进行验证. 在验证凭证之前, Spring Security通常会使用`AuthenticationEntryPoint`请求凭证.

然后, `AbstractAuthenticationProcessingFilter`可以验证任何提交给他的身份认证请求.

![AbstractAuthenticationProcessingFilter](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/abstractauthenticationprocessingfilter.png)_AbstractAuthenticationProcessingFilter_
- 1-当用户提交他们的凭证后, `AbstractAuthenticationProcessingFilter`从`HttpServletRequest`中创建一个待验证的`Authentication`. 创建的`Authentication`的类型取决于`AbstractAuthenticationProcessingFilter`子类. 例如`UsernamePasswordAuthenticationFitler`根据`HttpServletRequest`中提交的username和password创建一个`UsernamePasswordAuthenticationToken`.
- 2-将创建的`Authentication`传递给`AuthenticationManager`进行验证. 
- 3-如果验证失败, 执行以下操作:
	- 清空`SecurityContextHolder`
	- 调用`RememberMeService.loginFail`. 如果未配置remember me, 不执行任何操作.
	- 调用`AuthenticationFailureHandler`.
- 4-如果验证成功, 执行以下操作:
	- `SessionAuthenticationStrategy`收到新的登录通知(用于缓存?).
	- 在`SecurityContextHolder`中设置`Authentication`. 之后使用`SecurityContextPersistenceFilter`将`SecurityContex`保存到HttpSession中.
	- 调用`RememberMeService.loginSuccess`. 如果未配置remember me, 不执行任何操作.
	- 通过`ApplicationEventPublisher`发布一个`InteractiveAuthenticationSuccessEvent`.
	- 调用`AuthenticationSuccessHandler`.



