---
title: JWT-Json Web Token
tags: 
	- spring security 
	- jwt
categories: spring
date: 2021-04-24 23:08:00
---

## **JWT: Json Web Token**

> JWT 定义了紧凑且自包含的方式在实体间进行安全信息传输. JWT使用数字签名来保证传输的信息可以被验证并且保证其可信. JWT可以使用私钥(HMAC算法)进行签名, 或者使用RSA/ECDSA的公私钥进行签名.

### **JWT使用场景**

- 授权(Authorization): 这是JWT最常用的场景. 使用JWT可以替代常用的cookie-session模式.
- 信息交换: 在实体间进行安全的传输信息

### **JWT结构**

在JWT的紧凑形式中, JWT由三部分组成, 且每个部分之间使用`.`分隔. 以下是三个部分:

- Header
- Payload
- Signature

JWT通常是以下的形式: `xxxxx.yyyyy.zzzzz`

#### **Header**

JWT第一部分是header, header通常由两部分组成:

- token的类型, 即JWT
- 使用的签名算法, 例如: HMAC

以下是header的一个例子:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

接着使用Base64Url编码上述的json作为JWT的第一部分.

#### **Payload**

JWT的第二部分是payload, payload中包含了claims. claims通常是一些实体(通常是用户)和其他数据的声明. claims有三种类型: registered, public, private.

- Registered claims: 一些预定义的claims. 常用的有以下: iss(issuer), exp(expiration time), sub(subject), aud(audience)
- Public claims: 可以由使用JWT的人任意定义. 但是要避免冲突.
- Private claims: 自定义的claims. 由信息传输实体间协商好的的数据. 通俗来讲就是前后端约定的一些属性, 比如使用`uname`来存储用户名.

payload的一个例子:

```json
{
  "sub": "1234",
  "name": "hh",
  "admin": true
}
```

使用Base64Url编码payload的json并作为JWT的第二部分.

**NB: 尽管JWT可以通过签名防止篡改, 但是payload对任何人来说都是可读的(因为只是使用了Base64Url进行编码). 如果需要存放敏感信息, 则可以先对信息加密, 再放入payload中.**

#### **Signature**

JWT的第三部分是signature. 在创建signature之前, 必须确保存在编码后的header, 编码后的payload, secret(加密的key), 然后使用在header中指定的加密算法进行签名. 

例如是哟HMAC SHA256加密算法, 将会以以下方式创建signature: 

```text
HMACSHA256(
	base64UrlEncode(header) + '.' + 
	base64UrlEncode(payload),
	secret
)
```

signature用于保证消息在传输过程中没有被更改.



### **Java JJwt使用**



## 参考资料

- JWT官网: https://jwt.io/introduction

- JJwt: https://github.com/jwtk/jjwt

