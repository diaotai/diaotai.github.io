---
title: 'server 端认证:cookie,session and token'
date: 2019-06-09 18:33:01
tags: web
---
# 概述
最初的 Web 时代基本只需要阅读信息，所以无所谓身份认证。但是后续很多应用的出现，让身份认证变成了必须要存在的东西。本文简要介绍三个常见的手段:cookie, session 还有 token

# cookie
cookie 是一个非常具体的东西，指的就是浏览器里面能永久存储的一种数据，仅仅是浏览器实现的一种数据存储功能。由于 cookie 在客户端是完全可见且可修改的，因此不适合放一些机密的东西，也不适合把权限认证完全放在这一层来做。
此外，cookie 存在着 CORS 问题，即当需要进行跨域时，在前端应当设置:
```
xhr.withCredentials= true;
```
而在服务器端应当设置以下内容，借此表示后端愿意获得跨域 cookie 内容。
```
Access-Control-Allow-Origin: a.com //这里需要换成相应的发起请求的域名
Access-Control-Allow-Credentials: true 
```

# session
session 是将必要信息在服务器端存储，以此来辨认客户端身份的做法。
session 可能在服务器的内存来存放，也可能在 NOSQL 里集中存放，当存在负载均衡的时候，如何保证每一次访问都可以到指定 session 是一个很严重的问题。


# token
## 简介
token 是一种无状态的身份认证手段，拥有可扩展、跨设备（更适合移动端）、跨应用、安全的特点。基于 token 的认证流程如下:
1. 用户通过用户名和密码发送请求。
2. 程序验证。
3. 程序返回一个签名的token 给客户端。
4. 客户端储存token,并且每次发送请求都携带token。
5. 服务端验证token并返回数据。
![token 流程图](https://pic1.zhimg.com/80/v2-26d5210a9c95c3a112372a12555118d4_hd.jpg)

token 本质上是用时间换空间，通过每次请求都进行校验，来避免使用空间来存储 session。

## JWT
JWT(JSON Web Token) 是应用度很广的一种 token 标准，该标准的 token 中会存在三个部分——Header,Payload 和 $Signature，三者之间由 . 分隔。
### Header
Header 是一个 JSON 对象，对象里有一个值为 “JWT” 的 typ 属性，以及 alg 属性，值为 HS256，表明最终使用的加密算法是 HS256。该对象被 Base64 编码。
### Payload
Payload 被定义为实体的状态，就像 token 自身附加元数据一样，claim 包含我们想要传输的信息，以及用于服务器验证的信息，一般有 reserved/public/private 三类。
其中 Registered claims 指的是官方推荐但是不强制的一些属性，Public claims 属于公共约定的部分，有特定的地方进行注册，Private claims 就是开发者自行约定的，不需要遵循什么。
### Signature
$Signature 则是算法计算后的结果，该算法使用两个参数，第一个是 Base64 编码之后的 Header 和 Payload，第二个是服务器的密钥。
通过计算 token 里的前两段和 $Signature 是否一致，就可以判断 token 是否被修改，若无修改，则证明着确实是我们下发的 token，应予以通过。

### 注意点
* JWT 只能保证信息不被修改，无法保证不被看到，因此，不应该放一些关键信息在 token 中
* token 被窃取的情况是无法避免的，服务器无法判断这一点。

### 优点：
* 体积小（一串字符串）。因而传输速度快
* 传输方式多样。可以通过 HTTP 头部（推荐）/URL/POST 参数等方式传输
* 严谨的结构化。它自身（在 payload 中）就包含了所有与用户相关的验证消息，如用户可访问路由、访问有效期等信息，服务器无需再去连接数据库验证信息的有效性，并且 payload 支持应用定制
* 支持跨域验证，多应用于单点登录

**单点登录（Single Sign On）：在多个应用系统中，用户只需登陆一次，就可以访问所有相互信任的应用。**

# 参考资料
* [Server端的认证神器——JWT(一)](https://zhuanlan.zhihu.com/p/27370773)
* [彻底理解cookie，session，token](https://zhuanlan.zhihu.com/p/63061864)
* [A guide for adding JWT token-based authentication to your single page Node.js applications](https://medium.com/dev-bits/a-guide-for-adding-jwt-token-based-authentication-to-your-single-page-nodejs-applications-c403f7cf04f4)
* [Introduction to JSON Web Tokens](https://jwt.io/introduction/)