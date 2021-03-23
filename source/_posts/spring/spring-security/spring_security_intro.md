---
title: 常用注解
tags:
  - java
  - spring
categories: spring
date: 2021-03-23 23:40:00
---

## Spring Security 简介
> Spring Security 提供了认证(authentication)和授权(authorzation)功能, 并且可以防护常见的攻击.

## Spring Security特性
> Spring Security为认证, 授权和方法常见攻击提供了广泛的支持. 同时也提供了其他库的整合简化使用.

### 认证(Authentication)
认证是我们验证试图访问特定资源的人的身份的方式. 通常的认证一个用户的方式是使用用户名和密码. 一旦完成了认证, 我们就知道身份并执行授权.

#### 认证支持
Spring Security为认证客户提供了内置的支持. 
TODO authentication for Servelt

#### 密码存储(Password Storage)
##### PasswordEncoder
Spring Security的`PasswordEncoder`接口用于执行密码的单向转换(one way transformation), 使得密码可以安全存储. 给定的`PasswordEncoder`是进行单项转换, 当密码需要双向转换的时候(例如存储于向数据库进行身份验证的凭证, 当我们连接数据库是, 需要使用真实的密码传到数据库, 所以需要双向转换)就不需要使用它. 通常`PasswordEncoder`用于存储需要在身份验证时就与用户提供的密码进行比较的密码(可以在数据库中存储使用PasswordEncoder转换后的密码, 当用户需要验证密码时, 使用PasswordEncoder将用户提供的密码进行转换并与数据库中的密码进行比较, 这样数据库中就不会显示保存用户的密码).

##### 密码存储历史
1. 直接存储明文密码, 使用SQL注入很容易导致密码泄露
2. SHA-256, Rainbow Tables
3. 加盐的SHA-256, 计算机计算效率大幅提升, 也不安全
4. 现在推荐使用自适应单项函数来存储密码. 自适应单项函数有以下几个例子: bcrypt, PBKDF2, scrypt, and argon2.

在Spring Security 5.0 之前`PasswordEncoder`默认使用`NoOpPasswordEncoder`(需要简单的文本密码).由于密码存储历史, 你可能希望当前的默认`PasswordEncoder`是`BCryptPasswordEncoder`, 但是这样可能导致以下问题:
- 有许多应用程序使用无法轻易迁移的旧密码编码
- 密码存储的最佳实践会再次改变
- 作为一个框架, Spring Security不能频繁进行破坏性的更改

Spring Security引进了`DelegatingPasswordEncode`解决以下问题:
- 确保使用当前推荐的密码编码方式对密码进行编码
- 允许以现代和传统格式验证密码
- 允许在未来升级密码的编码方式

##### DelegatingPasswordEncoder
创建`DelegatingPasswordEncoder`可以通过`PasswordEncoderFactories`或者自定义的方式.
- `PasswordEncoderFactories`创建
```java
PasswordEncoder pe = PasswordEncoderFactories.createDelegatingPasswordEncoder();
```

- 自定义创建:
```java
String idForEncode = "bcrypt";
Map encoders = new HashMap<>();
encoders.put(idForEncode, new BCryptPasswordEncoder());
encoders.put("noop", NoOpPasswordEncoder.getInstance());
encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
encoders.put("scrypt", new SCryptPasswordEncoder());
encoders.put("sha256", new StandardPasswordEncoder());

PasswordEncoder passwordEncoder =
    new DelegatingPasswordEncoder(idForEncode, encoders);
```

##### 密码存储格式(Password Storage Format)
通用的密码格式如下:
```text
{id}encodePassword
```
`id`用于标识PasswordEncoder的编码方式. `id`必须在编码密码的开始, 并且以`{`开始, 以`}`结尾. 下面看一下使用不同id编码后的密码格式:
```text
{bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG 
{noop}password 
{pbkdf2}5d923b44a6d129f3ddf3e3c8d29412723dcbde72445e8ef6bf3b508fbf17fa4ed4d6b99ca763d8dc 
{scrypt}$e0801$8bWJaSu2IKSn9Z9kM+TPXfOc/9bdYSrN1oD9qfVThWEwdRTnO7re7Ei+fUZRJ68k9lTyuTeUp4of4g24hHnazw==$OAOec05+bXxvuu/1qZ6NUR+xQYvYv7BeL1QxwRpY5Pc=  
{sha256}97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0
```



DelegatingPasswordEncoder
### 授权(Authoriation)




