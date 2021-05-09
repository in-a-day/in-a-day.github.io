---
title: Spring Security杂记
date: 2021-04-26 14:19:19
tags: OAuth2
categories: OAuth2
---

TODO 异常处理:

1. 



#### 授权管理

授权主要是经由AccessDecisionManager决定. 通过委派给AccessDecisionVoter链实现授权管理. 有点类似与ProviderManager委派AuthenticationProvider链进行认证.

AccessDecisionManager主要有三个实现:

1. AffirmativeBased: 只要存在投票器通过就通过
2. ConsensusBased: 根据投票器通过和拒绝的票数决定是否通过
3.  所有的投票器都通过才通过



1. AccessDecisionManager:

   ```java
   public interface AccessDecisionManager {
   	/**
   	 * authentication是认证后的认证信息, 即SecurityContext中的Authentication
   	 * object是受保护的对象
   	 * configAttributes是与被保护对象相关的一些配置属性
   	 */
   	void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)
   			throws AccessDeniedException, InsufficientAuthenticationException;
    // 是否支持attribute对象
   	boolean supports(ConfigAttribute attribute);
   	// 是否支持给定class类型的受保护对象进行投票
   	boolean supports(Class<?> clazz);
   
   }
   ```

   

1. 投票器: AccessDecisionVoter

   ```java
   public interface AccessDecisionVoter<S> {
   	int ACCESS_GRANTED = 1;
   	int ACCESS_ABSTAIN = 0;
   	int ACCESS_DENIED = -1;
   	boolean supports(ConfigAttribute attribute);
   	boolean supports(Class<?> clazz);
   	int vote(Authentication authentication, S object,
   			Collection<ConfigAttribute> attributes);
   }
   ```

   

