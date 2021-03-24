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
## 架构组件(Architecture Components)
主要描述Spring Security用于Servlet认证中的架构组件. 主要有以下组件:
- `SecurityContextHolder`: 存储身份验证详细信息的地方.
- `SecurityContext`: 由`SecurityContextHolder`获得,包含了当前认证客户的`Authentication`.
- `Authentication`: 可以是`AuthenticationManager`的输入, 以提供用户提供的用于身份验证的凭据或来自SecurityContext的当前用户.
- `GrantedAuthority`: 授予身份验证主体的权限(即角色, 范围等).
- `AuthenticationManager`: 定义了Spring Security的Filters如何进行认证的API.
- `ProviderManager`: `AuthenticationManager`的一个通用实现.
- `Request Credentails with AuthenticationEntryPoint`: 用于从客户端请求凭证(即重定向到登录页面, 发送WWW-Authenticate响应等).
- `AbstractAuthenticationProcessingFilter`: 用于认证的基础Filter. 这样为高级别的身份认证流程和各个部件如何协同工作提供了一个好的概念.

## 认证机制(Authentication Mechanisms)
- Username and Password - 如何使用用户名密码认证
- OAuth 2.0 Login - OAhth 2.0 使用OpenID Connect登录和非标准的OAuth 2.0 登录(即GitHub)
- SAML 2.0 Login - SAML 2.0 登录
- Central Authentication Server (CAS) - CAS支持
- Remember Me - 如何记住用户
- JAAS Authentication - JAAS认证
- OpenID - OpenID 认证 (不要与OpenID Connect弄混)
- Pre-Authentication Scenarios - 使用外部机制进行认证(e.g. SiteMider或J2EE security), 但仍使用Spring Security授权和防范常见漏洞攻击.
- X509 Authentication - X509 认证


