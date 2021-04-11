---
title: Spring Security认证流程
tags:
  - java
  - spring
  - spring security
categories: spring
date: 2021-04-11 11:47:00
---
## Spring Security 流程

1. 首先Spring Security是通过一个filter进行拦截的, 起始Spring Security内部也会有FilterChain, 但是只暴露出了DelegatingFilterProxy(代理类)去执行

2. 通过DelegatingFilterProxy代理真正的Filter进行拦截

   ![DelegatingFilterProxy](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/delegatingfilterproxy.png)

3. Spring Security内部使用FilterChainProxy(FilterChain代理类)去执行SecurityFilterChain, FilterChainProxy是一个特殊的Filter, 交由DelegatingFilterProxy进行代理.

   ![FilterChainProxy](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/filterchainproxy.png)

   FilterChainProxy提供了3个构造方法:

   ```java
   	public FilterChainProxy() {
   	}
   
   	public FilterChainProxy(SecurityFilterChain chain) {
   		this(Arrays.asList(chain));
   	}
   
   	public FilterChainProxy(List<SecurityFilterChain> filterChains) {
   		this.filterChains = filterChains;
   	}
   
   ```

   从构造方法中可以看出, FilterChainProxy是支持多个SecurityFilterChain的. 再看一下FilterChainProxy是如何实现默认Filter中的doFilter方法:

   ```java
   	@Override
   	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
   			throws IOException, ServletException {
   		boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;
   		if (!clearContext) {
   			doFilterInternal(request, response, chain);
   			return;
   		}
   		try {
   			request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
   			doFilterInternal(request, response, chain);
   		}
   		catch (RequestRejectedException ex) {
   			this.requestRejectedHandler.handle((HttpServletRequest) request, (HttpServletResponse) response, ex);
   		}
   		finally {
   			SecurityContextHolder.clearContext();
   			request.removeAttribute(FILTER_APPLIED);
   		}
   	}
   ```

   从源码中可以看出其内部是调用了doFilterInternal方法, 如下:

   ```java
   	private void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain)
   			throws IOException, ServletException {
       // 这里首先经过Spring Security的防火墙判断
   		FirewalledRequest firewallRequest = this.firewall.getFirewalledRequest((HttpServletRequest) request);
   		HttpServletResponse firewallResponse = this.firewall.getFirewalledResponse((HttpServletResponse) response);
       // 重点: 获取所有的Filter, 本质上就是获取一个SpringFilterChain中的所有Filter
   		List<Filter> filters = getFilters(firewallRequest);
   		if (filters == null || filters.size() == 0) {
   			if (logger.isTraceEnabled()) {
   				logger.trace(LogMessage.of(() -> "No security for " + requestLine(firewallRequest)));
   			}
   			firewallRequest.reset();
   			chain.doFilter(firewallRequest, firewallResponse);
   			return;
   		}
   		if (logger.isDebugEnabled()) {
   			logger.debug(LogMessage.of(() -> "Securing " + requestLine(firewallRequest)));
   		}
   		VirtualFilterChain virtualFilterChain = new VirtualFilterChain(firewallRequest, chain, filters);
   		virtualFilterChain.doFilter(firewallRequest, firewallResponse);
   	}
   ```

   接下来看一下getFilters如何去获取Filters:

   ```java
   	private List<Filter> getFilters(HttpServletRequest request) {
   		int count = 0;
       // 首先遍历注册的SecurityFilterChain
   		for (SecurityFilterChain chain : this.filterChains) {
   			if (logger.isTraceEnabled()) {
   				logger.trace(LogMessage.format("Trying to match request against %s (%d/%d)", chain, ++count,
   						this.filterChains.size()));
   			}
         // 调用chain的matches方法, 如果匹配了, 就返回, 从这里也可以看出只有一条SecurityFilterChain会执行. 并且这个匹配是按顺序来了, 所以添加SecurityFilterChain的顺序也十分重要.
   			if (chain.matches(request)) {
           // 返回该SecurityFilterChain中所有Filter
   				return chain.getFilters();
   			}
   		}
   		return null;
   	}
   ```

   从getFilters方法中我们很容易可以得到以下结论: 最多只有一个SecurityFilterChain会执行.最后在doFilterInternal方法中调用Filter的doFilter方法.

   总结以下FilterChainProxy的流程:

   1. 通过构造参数传入SecurityFilterChain
   2. 匹配需要执行的SecurityFilterChain, 只可能是没有或是1个
   3. 执行SecurityFilterChain中Filter

   ![Multiple SecurityFilterChain](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/multi-securityfilterchain.png)

   由于FilterChainProxy是Spring Security中真正的执行入口, 所以debug的时候可以在该类中打断点.

4. 接下来看一下SecurityFilterChain(FilterChain接口), 该接口主要定义了以下两个方法:

   ```java
   public interface SecurityFilterChain {
   
   	boolean matches(HttpServletRequest request);
   
   	List<Filter> getFilters();
   
   }
   ```

   matches用于匹配是否满足执行该FilterChain的条件

   getFilters获取该FilterChain中的所有Filter

   在FilterChainProxy中的getFilters方法就是调用了以上两个方法进行判断并获取Filters.

5. 总结一下SS的Filter流程

   1. FilterChainProxy交由DelegatingFilterProxy代理, 最终由DelegatingFilterProxy充当一个普通的Servlet Filter. 
   2. 通过FilterChainProxy去决定执行哪一个SecurityFilterChain
   3. SecurityFilterChain中定义需要执行的Filter

### Spring Security中常见的Filter

#### SecurityContextPersistenceFilter

> 该Filter主要用于设置SecurityContextHoler, 即将SecurityContext属性写入SecurityContextHolder中. 同时也在请求结束后清空SecurityContextHolder.

下面看一下该Filter的doFilter方法:

```java
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);
	}

	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		// ensure that filter is only applied once per request
    // 确保每个请求只应用该filter一次
		if (request.getAttribute(FILTER_APPLIED) != null) {
			chain.doFilter(request, response);
			return;
		}
		request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
		if (this.forceEagerSessionCreation) {
			HttpSession session = request.getSession();
			if (this.logger.isDebugEnabled() && session.isNew()) {
				this.logger.debug(LogMessage.format("Created session %s eagerly", session.getId()));
			}
		}
    // HttpRequestResponseHolder只是简单存储了HttpServletRequest和HttpServletResponse
		HttpRequestResponseHolder holder = new HttpRequestResponseHolder(request, response);
    // 使用SecurityContextRepository从holder中获取SecurityContext
		SecurityContext contextBeforeChainExecution = this.repo.loadContext(holder);
		try {
			// 设置线程私有的SecurityContext. SecurityContextHolder默认使用ThreadLocal对象去存储SecurityContext
      SecurityContextHolder.setContext(contextBeforeChainExecution);
			if (contextBeforeChainExecution.getAuthentication() == null) {
				logger.debug("Set SecurityContextHolder to empty SecurityContext");
			}
			else {
				if (this.logger.isDebugEnabled()) {
					this.logger
							.debug(LogMessage.format("Set SecurityContextHolder to %s", contextBeforeChainExecution));
				}
			}
      // 之后执行下游的filter
			chain.doFilter(holder.getRequest(), holder.getResponse());
		}
		finally {
      // 在每次请求结束之后清空SecurityContext, 防止泄露
			SecurityContext contextAfterChainExecution = SecurityContextHolder.getContext();
			// Crucial removal of SecurityContextHolder contents before anything else.
			SecurityContextHolder.clearContext();
			this.repo.saveContext(contextAfterChainExecution, holder.getRequest(), holder.getResponse());
			request.removeAttribute(FILTER_APPLIED);
			this.logger.debug("Cleared SecurityContextHolder to complete request");
		}
	}
```

从以上doFilter看来, 主要有以下关键点:

1. HttpRequestResponseHolder, 该类主要存储了请求的HttpServletRequest和HttpServletResponse:

   ```java
   public final class HttpRequestResponseHolder {
   
   	private HttpServletRequest request;
   
   	private HttpServletResponse response;
   
   	public HttpRequestResponseHolder(HttpServletRequest request, HttpServletResponse response) {
   		this.request = request;
   		this.response = response;
   	}
   
   	public HttpServletRequest getRequest() {
   		return this.request;
   	}
   
   	public void setRequest(HttpServletRequest request) {
   		this.request = request;
   	}
   
   	public HttpServletResponse getResponse() {
   		return this.response;
   	}
   
   	public void setResponse(HttpServletResponse response) {
   		this.response = response;
   	}
   
   }
   ```

   从源码可以看出只是保存了两个属性: HttpServletRequest和HttpServletResponse.

2. 使用SecurityContextRepository从HttpRequestResponseHolder加载SecurityContext:

   ```java
   public interface SecurityContextRepository {
     // 获取给定请求的SecurityContext. 对于未认证的用户来说, 应该返回一个空的context实现. 这个方法不应该返回null;
   	SecurityContext loadContext(HttpRequestResponseHolder requestResponseHolder);
   
     // 在完成的请求中存储SecurityContext
   	void saveContext(SecurityContext context, HttpServletRequest request, HttpServletResponse response);
   
     // repository查询自身是否包含当前请求的一个SecurityContext
   	boolean containsContext(HttpServletRequest request);
   }
   ```



### UsernamePasswordAuthenticationFilter

> 处理身份验证表单提交. 登录表单中必须存在两个参数: username和password. 默认的参数名称是username和password. 可以通过setUsernameParameter和setPasswordParameter方法修改默认名称.
>
> 该Filter默认会响应/login页面.

看一下该类中的默认属性, 从属性中我们可以看出为什么默认是username和password:

```java
	// 这里就是默认的username和password名称	
	public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";
	public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";
	// 这里设置了默认的响应login页面POST方法.
	private static final AntPathRequestMatcher DEFAULT_ANT_PATH_REQUEST_MATCHER = new AntPathRequestMatcher("/login", "POST");
	
	// 这里就是默认的username和password名称, 所以可以通过这两个属性的set方法修改
	private String usernameParameter = SPRING_SECURITY_FORM_USERNAME_KEY;
	private String passwordParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;
	// 默认只是POST方法
	private boolean postOnly = true;

	// 从request中获取密码
	@Nullable
	protected String obtainPassword(HttpServletRequest request) {
		return request.getParameter(this.passwordParameter);
	}
	// 从request中获取用户名
	@Nullable
	protected String obtainUsername(HttpServletRequest request) {
		return request.getParameter(this.usernameParameter);
	}

```

下面看一下UsernamePasswordAuthenticationFilter的doFilter方法(该类继承自AbstractAuthenticationFilter, 默认使用了父类的doFilter方法):

```java
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);
	}

	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws IOException, ServletException {
    // 如果不需要身份认证, 执行下游Filter
		if (!requiresAuthentication(request, response)) {
			chain.doFilter(request, response);
			return;
		}
		try {
      // 真正执行认证的方法
			Authentication authenticationResult = attemptAuthentication(request, response);
      // 返回null代表需要进一步认证
			if (authenticationResult == null) {
				// return immediately as subclass has indicated that it hasn't completed
				return;
			}
			this.sessionStrategy.onAuthentication(authenticationResult, request, response);
			// 认证成功, 判断是否需要继续执行下游Filter
			if (this.continueChainBeforeSuccessfulAuthentication) {
				chain.doFilter(request, response);
			}
			successfulAuthentication(request, response, chain, authenticationResult);
		}
		catch (InternalAuthenticationServiceException failed) {
			this.logger.error("An internal error occurred while trying to authenticate the user.", failed);
			unsuccessfulAuthentication(request, response, failed);
		}
		catch (AuthenticationException ex) {
			// Authentication failed
			unsuccessfulAuthentication(request, response, ex);
		}
	}
```

下面具体看一下attemptAuthentication方法执行流程:

```java
	@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException {
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
    // 从request中获取username和password
		String username = obtainUsername(request);
		username = (username != null) ? username : "";
		username = username.trim();
		String password = obtainPassword(request);
		password = (password != null) ? password : "";
    // 创建认证令牌, 此时尚未开始认证
		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
		// Allow subclasses to set the "details" property
    // 默认是在authRequest中设置一些details: request中的remoteAddress和sessionId
		setDetails(request, authRequest);
    // 交由AuthenticationManager进行认证, 通常是使用ProviderManager, ProviderManager中通常有一系列的AuthenticationProvider, 最终交由这些Provider进行验证, 只要有一个Provider验证通过就通过. 如果没有通过, 使用ProviderManager的父AuthenticationManager进行验证(如果存在的话).
		return this.getAuthenticationManager().authenticate(authRequest);
	}

```

### ExceptionTranslationFilter

> 用于处理Spring Security中的异常, 将AccessDeniedException和AuthenticalException异常转换为HTTP响应.

![ExceptionTranslationFilter](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/java/spring/spring-security/exceptiontranslationfilter.png)

看一下该Filter的doFilter方法:

```java
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);
	}

	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		try {
      // 调用下游的Filter
			chain.doFilter(request, response);
		}
		catch (IOException ex) {
      // 如果是IO异常直接抛出
			throw ex;
		}
		catch (Exception ex) {
			// Try to extract a SpringSecurityException from the stacktrace
			Throwable[] causeChain = this.throwableAnalyzer.determineCauseChain(ex);
      // 判断异常是否存在AuthenticaitonException或AccessDeniedException
			RuntimeException securityException = (AuthenticationException) this.throwableAnalyzer
					.getFirstThrowableOfType(AuthenticationException.class, causeChain);
			if (securityException == null) {
				securityException = (AccessDeniedException) this.throwableAnalyzer
						.getFirstThrowableOfType(AccessDeniedException.class, causeChain);
			}
      // 如果不是上述两个异常直接抛出
			if (securityException == null) {
				rethrow(ex);
			}
			if (response.isCommitted()) {
				throw new ServletException("Unable to handle the Spring Security Exception "
						+ "because the response is already committed.", ex);
			}
      // 处理上述两个异常
			handleSpringSecurityException(request, response, chain, securityException);
		}
	}
```

可以看到, 该类调用handleSpringSecurityException进行处理这两个异常:

```java
	private void handleSpringSecurityException(HttpServletRequest request, HttpServletResponse response,
			FilterChain chain, RuntimeException exception) throws IOException, ServletException {
		if (exception instanceof AuthenticationException) {
			handleAuthenticationException(request, response, chain, (AuthenticationException) exception);
		}
		else if (exception instanceof AccessDeniedException) {
			handleAccessDeniedException(request, response, chain, (AccessDeniedException) exception);
		}
	}
```

首先看一下处理handleAuthenticationException(认证异常)异常:

```java
	private void handleAuthenticationException(HttpServletRequest request, HttpServletResponse response,
			FilterChain chain, AuthenticationException exception) throws ServletException, IOException {
		this.logger.trace("Sending to authentication entry point since authentication failed", exception);
		sendStartAuthentication(request, response, chain, exception);
	}
```

最终调用了sendStartAuthentication:

```java
protected void sendStartAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
      AuthenticationException reason) throws ServletException, IOException {
   // SEC-112: Clear the SecurityContextHolder's Authentication, as the
   // existing Authentication is no longer considered valid
   SecurityContextHolder.getContext().setAuthentication(null);
   this.requestCache.saveRequest(request, response);
   this.authenticationEntryPoint.commence(request, response, reason);
}
```

首先清空SecurityContextHolder, 然后将`HttpServletRequest`保存到`RequestCache`中. 当用户成功认证后, 将`RequestCache`内容重播原始request. 最后使用`AuthenticationEntryPoint`从客户端请求凭证(credential).可能重定向到登录页面或者发送一个`WWW-Authenticat`头部(header).



最后看一下handleAccessDeniedException(访问异常):

```java
	private void handleAccessDeniedException(HttpServletRequest request, HttpServletResponse response,
			FilterChain chain, AccessDeniedException exception) throws ServletException, IOException {
		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		boolean isAnonymous = this.authenticationTrustResolver.isAnonymous(authentication);
		if (isAnonymous || this.authenticationTrustResolver.isRememberMe(authentication)) {
			if (logger.isTraceEnabled()) {
				logger.trace(LogMessage.format("Sending %s to authentication entry point since access is denied",
						authentication), exception);
			}
			sendStartAuthentication(request, response, chain,
					new InsufficientAuthenticationException(
							this.messages.getMessage("ExceptionTranslationFilter.insufficientAuthentication",
									"Full authentication is required to access this resource")));
		}
		else {
			if (logger.isTraceEnabled()) {
				logger.trace(
						LogMessage.format("Sending %s to access denied handler since access is denied", authentication),
						exception);
			}
			this.accessDeniedHandler.handle(request, response, exception);
		}
	}
```

通常是交由accessDeniedHandler进行处理.
