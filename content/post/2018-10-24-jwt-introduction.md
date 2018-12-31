---
title: JWT简介
date: 2018-10-24T16:03:48+08:00
slug: jwt-introduction
tags: ["JWT", "技术分享"]
categories: ["技术分享"]
draft: false
---

# JWT是什么

> JWT is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object.

JWT是JSON Web Token的缩写，从官方文档直译：一个安全的传输信息的开放标准，使用JSON格式携带信息。

可能看完这个定义还是没能理解，简单的说，首先它是一个Token，一串字符，用来验证你可以访问某些资源的凭证。其次它使用轻量级的JSON格式自携带一些数据，以减少服务端的数据库查询，比如用户的ID，权限等。然后通过某种数字签名的算法，保证自携带的信息不会被篡改。

我们再解释一下官方文档中的两个概念：

- Compact（紧凑）：说它紧凑是因为它的尺寸或长度，可以通过URL、POST参数或者放到HTTP的Header来传递，因为它体积小，所以在传输相对更快。当然是相对SAML(Security Assertion Markup Language Tokens)来说的，因为SAML使用的是XML格式，还带了很多其他信息。
- Self-contained（自携带）：它本身带了一些关于用户的必要信息，这样可以减少服务端数据库的查询。

# 为什么使用JWT

## 普通Token与JSON Web Token

我们先描述一下普通Token的实现，当用户通过用户名密码登录认证后，服务器端会生成一个字符串作为用户之后请求的凭证，并且保存在服务端数据库中。这样用户随后的请求就不需要每次都带有用户名与密码信息了，但是要带上Token。服务器端通过查询数据库验证用户的Token是否有效，然后返回数据给用户。

JWT相对上面的流程，服务器端不需要把Token保存在数据库中，因为可以通过签名算法验证Token是否有效，并且自携带了用户的一些必要信息，这样就不用每个请求都去查询数据库，从而减少请求的响应时间。

可能上面说到的优势并不能说服你使用JWT，那么我们再设想一下现在比较流行的微服务的场景。

假设有两个微服务A与B，如果使用普通Token来实现，一般有两种架构方式：

- 集中认证方式，用户无论访问服务A还是B的保护资源，都要去认证服务器去验证Token是否有效，然后再返回数据。这样就会造成认证服务器的访问量巨大，形成瓶颈。
- 分布式认证，用户的Token分别保存在服务A与B的数据库中，每次用户登录成功生成Token都需要同步到所有的微服务数据库中。当Token失效后，还需要再次同步到所有服务。当有新的微服务时，还需要记得将Token同步到新服务。

使用JWT时就比较简单了，首先有一个认证中心，用户登录后获取到JWT后，之后请求都带着Token访问服务A或B，因为每个微服务都可以通过签名验证Token有效性，所以不用查询数据库，也不需要将Token同步到所有微服务。

## SAML vs JWT

Security Assertion Markup Language Tokens (SAML)，SAML是一个标准的协议用在Web浏览器的SSO中。

因为JSON比XML更少的冗余，所以编码后的字符长度也会更短，JWT也就比SAML更短、更紧凑了。这也成为JWT更适合被在HTML中与HTTP环境了。

使用XML数字签名为XML签名要比为签名JSON更困难与易出错。

JSON在大部分的编程语言中都更容易解析映射到对象，XML并没有天然的文档到对象的映射，所以JWT相比SAML更易用。

![jwt-vs-saml](https://magnet-file.qn.cichang.net/yuanping/blog/comparing-jwt-vs-saml2.png)

# 什么时候用JWT

因为它有很少的网路开销，可以跨不同的域名认证，当今JWT广泛的使用在SSO（Single Sign On）单点登录中。

比如A 网站和 B 网站是同一家公司的关联服务。如果要求用户只要在其中一个网站登录，再访问另一个网站就会自动登录。

其实在前面章节也提到过，如果你正在构建一个由多个微服务组成的应用，从服务器到服务器或客户端到服务器（如：移动应用 APP 或SPA单页面应用）的 API 服务，那么使用 JWT 是非常合适的。

假设有三个独立的服务A,B,C，都通过API提供服务。用户通过App登录后，可以通过JWT直接访问服务A,B,C的API。

服务之间内部调用也可以用JWT，但是要与用户的JWT使用不同的Secret（密钥）。

# 怎样使用JWT

客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息`Authorization`字段里面。

    Authorization: Bearer <token>

需要注意的是，JWT中的信息并不是加密的，只是用Base64编码了一下，别人如果可以获得Token，是可以看到你放在Token中的所有信息的，所有Token中不可以放密码等敏感信息。

为了减少盗用和窃取，JWT不建议使用HTTP协议来传输代码，而是使用加密的HTTPS协议进行传输。

假设有三个独立的服务A,B,C，都通过API提供服务。用户在App中使用用户名、密码登录后获得JWT，保存到本地，之后的请求都会将JWT放到请求的Header中，访问各服务的API。因为A,B,C都使用同一个密钥，所以都可以验证JWT是否有效。这样用户就可以访问服务A,B,C的API，获取受保护的资源，而不用再次登录认证了。

# JWT的缺点

通过上面的介绍，你可能认为JWT太棒了，想立即再项目中使用它，但是什么东西都有两面性，你一定要知道他的所有缺点之后再权衡JWT是否真的适合你的项目。

JWT的最大缺点是服务器不保存会话状态，所以在使用期间不可能取消Token或更改Token的有效期。也就是说，一旦JWT签发，在有效期内将会一直有效。

比如用户修改了密码，这时需要把此用户之前登录的所有的Client端都踢出，JWT直接是没发做到的。还有如果需要实现，不允许同一个用户在同一类型设备上同时登录，比如用户在A手机上登录了，当在B手机登录时，需要把A手机上用户踢出登录，JWT是也做不到的。

当然总是有办法的，可以通过BlackList的方式来实现，后续我会在写一篇详细的方案。之后也会再写一篇关于JWT的实现方式与原理的文章。

# 参考链接

[JWT.IO - JSON Web Tokens Introduction](https://jwt.io/introduction/)

[Get Started with JSON Web Tokens - Auth0](https://auth0.com/learn/json-web-tokens/)