---
title: OAuth2介绍
date: 2021-04-11 14:19:19
tags: OAuth2
categories: OAuth2
---
## OAuth2
> OAuth2是一个授权框架, 允许第三方应用受限获取用户数据.

### 角色(Roles)
OAuth定义了4中角色:
- resource owner
能够授权访问受保护资源的实体, 通常是一个用户.
- resource server
存储受保护资源的服务器, 可以使用访问令牌(access token)接收和响应受保护资源的请求.
- client
代表资源所有者和其授权, 对受保护资源发出请求的应用程序. 通常的例子是一个浏览器, 或者是后端服务器.
- authorization server
在成功认证资源所有者并获取授权后, 向客户端发送access token的服务器.

### 协议流(Protocol Flow)
```text
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)-----Access Token  ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+ 
```
以上协议流描述了OAuth2中四种角色的交互:
- A: 客户端向Resource Owner请求授权. 授权请求可以直接向resource owner进行请求, 也可以通过授权服务器作为中介进行请求, 通常使用后者. 
- B: 客户端接收到资源所有者的授权许可, 该授权许可是可以代表资源所有者的授权的凭证, 使用四种授权类型之一或拓展的授权类型表示. 授权类型取决于客户端请求授权使用的方法和授权服务器支持的类型.
- C: 客户端使用B中接受的授权许可向授权服务器请求访问令牌.
- D: 授权服务器验证客户端的授权许可, 如果通过, 返回访问令牌.
- E: 客户端使用访问令牌向资源服务器请求资源.
- F: 资源服务器验证访问令牌, 如果通过, 返回请求资源.

### 授权许可(Authorization Grant)
> 授权许可是代表资源所有者的授权(获取资源所有者受保护的资源)的凭证, 用于客户端获取访问令牌.
Oauth2规范定义了四种授权类型:
- 授权码: authorization code
- 隐式: implicit
- 资源所有者密码凭证: resource owner password credentials
- 客户端凭证: client credentials

除此之外, 定义了一个用于其他类型的拓展机制.

#### 授权码(Authorization Code)
授权码通过使用授权服务器作为客户端与资源所有者之间的中介获取的. 客户端不直接从资源所有者获取授权, 而是将资源所有者定向到授权服务器, 然后授权服务器将携带授权码的资源所有者返回到客户端.

在将携带授权码的资源所有者返回到客户端之前, 资源服务器会资源所有者进行验证并获取授权. 因为资源所有者仅和授权服务器进行交互验证, 所以资源所有者的凭证不会与客户端共享.

例如我们调用github进行第三方登录, 用户只需要在github服务器进行验证授权, 客户端不会得到用户的密码, 仅会得到github返回的访问令牌.

#### 隐式(Implicit)


#### 资源所有者密码凭证(Resource Owner Password Credentials)

#### 客户端凭证(Client Credentials)


