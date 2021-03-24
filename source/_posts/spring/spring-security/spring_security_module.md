---
title: Spring Security模块介绍(5.4.5)
tags:
  - java
  - spring
  - spring security
categories: spring
date: 2021-03-24 13:54:00
---
## **Spring Security核心模块**
Spring Security分为以下几个核心模块:

### **Core — spring-security-core.jar**
> 该模块包含了核心认证(core authentication), 访问控制(access-control)类和接口, 远程支持(remoting support), 和基础的配置(provisioning)API. 包含以下几个顶级包:
- org.springframework.security.core
- org.springframework.security.access
- org.springframework.security.authentication
- org.springframework.security.provisioning

### **Remoting — spring-security-remoting.jar**
> 该模块提供了与Spring Remoting的整合.

### **Web — spring-security-web.jar**
> 该模块包含了过滤器和相关的web安全(web-security)基础代码. 包含了任何与servlet API相关的依赖. 

### **Config — spring-security-config.jar**
> 该模块包含了安全命令空间解析代码和Java配置代码. 如果配置中使用Spring Security的XML命名空间或Spring Security的Java配置支持, 则需要引入该模块.

### **LDAP — spring-security-ldap.jar**
> 该模块包含LDAP认证和配置代码.

### **OAuth 2.0 Core — spring-security-oauth2-core.jar**
> 该模块提供了支持OAuth2.0认证框架和OpenID Connect Core1.0框架的核心类和接口.

### **OAuth 2.0 Client — spring-security-oauth2-client.jar**
> 该模块包含对OAuth2.0认证框架和OpenID Connect Core1.0的客户端支持.

### **OAuth 2.0 JOSE — spring-security-oauth2-jose.jar**
> 该模块包含了Spring Security对JOSE(Javascript Object Signing and Encryption)的支持. JOSE框架旨在提供一个在各方之间安全转移声明的方法(The JOSE framework is intended to provide a method to securely transfer claims between parties). 它由一组规范构成:
- JSON Web Token (JWT)
- JSON Web Signature(JWS)
- JSON Web Encryption(JWE)
- JSON Web Key(JWE)

包含以下顶级包:
- org.springframework.security.oauth2.jwt
- org.springframework.security.oauth2.jose

### **OAuth 2.0 Resource Server — spring-security-oauth2-resource-server.jar**
> 该模块包含Spring Security对OAuth 2.0资源服务的支持.

### **ACL — spring-security-acl.jar**
> 该模块包含一个特定领域对象ACL实现. 用于将安全性应用到应用中的指定领域对象实例上.(It is used to apply security to specific domain object instances within your application. 太拗口了).

### **CAS — spring-security-cas.jar**
> 该模块包含了Spring Security的CAS(Central Authentication Service)客户端整合. 如果想要使用CAS single sign-on(单点登录)服务可以使用它.

### **OpenID — spring-security-openid.jar**
> 该模块包含OpenID web认证支持. **OpenID 1.0和2.0协议已经过时了, 现在推荐使用OpenID Connect**

### **Test — spring-security-test.jar**
> 该模块包含测试支持.

### **Taglibs — spring-secuity-taglibs.jar**
> 该模块包含Spring Security的JSP标签实现.
