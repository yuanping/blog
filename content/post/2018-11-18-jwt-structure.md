---
title: JWT的数据结构
date: 2018-11-18T16:03:48+08:00
slug: jwt-structure
tags: ["JWT", "技术分享"]
categories: ["技术分享"]
draft: false
---

# JWT 的原理
服务器认证以后，生成一个 JSON 对象，发回给用户，就像下面这样。

    {
      "姓名": "张三",
      "角色": "管理员"
    }

以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名（详见后文）。

服务器就不保存任何 session 数据了，也就是说，服务器变成无状态了，从而比较容易实现扩展。

# JWT 的数据结构

实际的 JWT 大概就像下面这样。

![](https://www.wangbase.com/blogimg/asset/201807/bg2018072304.jpg)

它是一个很长的字符串，中间用点（`.`）分隔成三个部分。注意，JWT 内部是没有换行的，这里只是为了便于展示，将它写成了几行。

JWT 的三个部分依次如下。

    Header（头部）Payload（负载）Signature（签名）

写成一行，就是下面的样子。

    Header.Payload.Signature

![](https://www.wangbase.com/blogimg/asset/201807/bg2018072303.jpg)

下面依次介绍这三个部分。

## Header

Header 部分是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子。

    {
      "alg": "HS256",
      "typ": "JWT"
    }

上面代码中，`alg`属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；`typ`属性表示这个令牌（token）的类型（type），JWT 令牌统一写为`JWT`。

最后，将上面的 JSON 对象使用 Base64URL 算法（详见后文）转成字符串。

## Payload

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。

- iss (issuer)：签发者
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众，接收jwt的一方
- nbf (Not Before)：生效时间，定义在什么时间之前，该jwt都是不可用的。
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

除了官方字段，你还可以在这个部分定义私有字段，下面就是一个例子。

    {
      "sub": "1234567890",
      "name": "John Doe",
      "admin": true
    }

注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。

这个 JSON 对象也要使用 Base64URL 算法转成字符串。

## Signature

Signature 部分是对前两部分的签名，防止数据篡改。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

> HMACSHA256(base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户。

## Base64URL

前面提到，Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符`+`、`/`和`=`，在 URL 里面有特殊含义，所以要被替换掉：`=`被省略、`+`替换成`-`，`/`替换成`_` 。这就是 Base64URL 算法。

## 完整的JWT

JWT格式的输出是以`.`分隔的三段Base64编码，与SAML等基于XML的标准相比，JWT在HTTP和HTML环境中更容易传递。

下列的JWT展示了一个完整的JWT格式，它拼接了之前的Header， Payload以及秘钥签名：

![encoded-jwt](https://magnet-file.qn.cichang.net/yuanping/blog/encoded-jwt3.png)

如果你想把刚才我们说的概念做一个实践，你可以使用官方网站提供的小工具去调试

![debug-jwt](https://magnet-file.qn.cichang.net/yuanping/blog/legacy-app-auth-5.png)