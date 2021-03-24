---
title: Spring Security介绍及特性(5.4.5)
tags:
  - java
  - spring
  - spring security
categories: spring
date: 2021-03-23 23:40:00
---

## **Spring Security 简介**
> Spring Security 提供了认证(authentication)和授权(authorzation)功能, 并且可以防范常见的漏洞攻击.

## **Spring Security特性**
> Spring Security为认证, 授权和防范常见漏洞攻击提供了广泛的支持. 同时也提供了其他库的整合简化使用.

### **认证(Authentication)**
认证是我们验证试图访问特定资源的人的身份的方式. 通常的认证一个用户的方式是使用用户名和密码. 一旦完成了认证, 我们就知道身份并执行授权.

#### **认证支持**
Spring Security为认证客户提供了内置的支持. 
TODO authentication for Servelt

#### **密码存储(Password Storage)**
##### **PasswordEncoder**
Spring Security的`PasswordEncoder`接口用于执行密码的单向转换(one way transformation), 使得密码可以安全存储. 给定的`PasswordEncoder`是进行单项转换, 当密码需要双向转换的时候(例如存储于向数据库进行身份验证的凭证, 当我们连接数据库是, 需要使用真实的密码传到数据库, 所以需要双向转换)就不需要使用它. 通常`PasswordEncoder`用于存储需要在身份验证时就与用户提供的密码进行比较的密码(可以在数据库中存储使用PasswordEncoder转换后的密码, 当用户需要验证密码时, 使用PasswordEncoder将用户提供的密码进行转换并与数据库中的密码进行比较, 这样数据库中就不会显示保存用户的密码).

##### **密码存储历史**
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

##### **DelegatingPasswordEncoder**
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


##### **密码存储格式(Password Storage Format)**
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
上述`{bcrypt}`代表使用BCrypt编码方式进行密码加密, 其他类似.

##### **密码存储配置**
Spring Security默认使用`DelegatingPasswordEncoder`. 可以通过定义`Password`bean注入Spring作为默认`PasswordEncoder`. 例如:
```
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

### **漏洞防护(Protection Against Exploits)**
Spring Security提供了针对常见漏洞的防护. 只要可能, 这些防护防护措施是默认开启的.

#### **CSRF(Cross Site Request Forgery)**
> Spring Security对跨站请求伪造攻击防护提供了全面的支持.

##### **CSRF是什么**
CSRF利用网站对用户网页浏览器的信任进行攻击. 通常用户登录后, 服务器会返回cookie给用户浏览器保存登录信息. 攻击者在自己网站中调用用户授权的网站链接, 浏览器会默认携带cookie信息, 从而欺骗浏览器进行攻击.

##### **防护CSRF攻击**
Spring 提供了两种技术进行防护CSRF攻击:
- Synchronizer Token Pattern(同步器token模式)
- 在cookie中指定SameSite Attribute

**NB:以上两种方式都要求安全方法必须是幂等的(Safe Methods Must be Idempotent)** 
> Safe Methods Must be Idempotent, 为了保证CSRF防护正常工作, 必须保证安全方法的幂等性. 这意味着`GET`, `HEAD`, `OPTIONS`, `TRACE`必须保证不修改应用的状态.

##### **同步器token模式(Synchronizer Token Pattern)**
最主要且广泛使用的防护CSRF攻击的措施是同步器token模式. 该方式确保每个HTTP请求除了有`session cookie`还必须携带一个称为`CSRF token`的安全随机值.

每当一个HTTP请求被提交, 服务器必须验证提交的`CSRF token`是否与后台生成的`CSRF token`一致, 若不一致, 拒绝请求.

使用这种方法的关键点在于`CSRF token`必须是HTTP请求的一部分, 并且不会被浏览器自动提交. 例如: 在HTTP请求的参数中或者header中携带`CSRF token`可以防护攻击. 在cookie中使用则不行, 因为cookie会被浏览器自动提交.


##### **SameSite Attribute**
一个新兴的防护CSRF攻击的方式是为cookie指定SameSite Attribute. 指定SameSite属性可以使用通过外部网站访问时浏览器不会自动携带cookie.

**NB:Spring Security不直接控制`session cookie`的创建, 所以不支持SameSite属性. `Spring Session`提供了对该属性的支持.** 

HTTP响应头带有SameSite属性例子:
```text
Set-Cookie: JSESSIONID=randomid; Domain=bank.example.com; Secure; HttpOnly; SameSite=Lax
```
`SameSite`的合法属性包括以下两种:
- Strict: 相同站点请求时会携带cookie, 否则不携带
- Lax: 相同站点或者该站点的顶级站点(如当前站点是social.example.com, 则email.example.com也会携带)且方法幂等请求时会携带cookie, 否则不携带.

**NB:`SameSite`属性需要浏览器的支持.** 

#### **TODO 安全HTTP响应头(Security HTTP Response Header)**







