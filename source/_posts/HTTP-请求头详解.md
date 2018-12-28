---
title: HTTP 请求头详解
date: 2016-08-21 11:02:09
tags: [HTTP, 概念]
categories: tech
---

# 简介

HTTP (Hypertext Transfer Protocol) 是一个为分布式、协同、超媒体信息系统制定的**应用层协议**。通常，HTTP 消息包括客户端向服务器的请求消息和服务器向客户端的响应消息。这两种类型的消息由一个起始行、一个或多个头域、一个头域结束的空行和可选的消息体组成。其中，HTTP 的头域包括`通用头`、`请求头`、`响应头`和`实体头`这四个部分。每个头域由一个头域的域名，冒号和域值组成。
本文主要研究客户端向服务器请求消息的部分:

```
Request       =         Request-Line              ;
                        *(( general-header        ;
                         | request-header         ;
                         | entity-header ) CRLF)  ;
                        CRLF
                        [ message-body ]          ;
```

<!--more-->

---

## 一、 请求消息中 Request-Line 的结构

> A request message from a client to a server includes, within the first line of that message, the method to be applied to the resource, the identifier of the resource, and the protocol version in use.

除了前面提到的，一条从客户端到服务器的请求消息的第一行包括以下这几个内容：请求资源的方法，资源的标识符（URI）和所使用协议的版本号，如下所示：

```
Requset-Line = Method SP Request-URI SP HTTP-Version CRLF
```

我们以请求网易首页 (http://www.163.com/) 的 Requset 消息为例，如下所示：

```
GET / HTTP/1.1
Host: www.163.com
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Referer: https://www.baidu.com/link?url=RhiUC8ctmOtA7LLqJ6SrkHdOkWD_l2xnFR4uJkrxcAi&wd=&eqid=fe2aa5df000998f30000000557b96de6
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
Cookie: _ntes_nnid=...
```

我们可以知道，请求的方法为 `GET`, URI 为 `/`, HTTP 版本号为 `HTTP/1.1`。其中，请求方法除了上述的 GET 外，常用的还有 POST、PUT、DELETE 等。具体可以参照 W3C 标准： [https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)：

```
Method         = "OPTIONS"                ;
               | "GET"                    ;
               | "HEAD"                   ;
               | "POST"                   ;
               | "PUT"                    ;
               | "DELETE"                 ;
               | "TRACE"                  ;
               | "CONNECT"                ;
               | extension-method
extension-method = token
```

## 二、 请求头 Request Header Fields

> The request-header fields allow the client to pass additional information about the request, and about the client itself, to the server. These fields act as request modifiers, with semantics equivalent to the parameters on a programming language method invocation.

请求头域允许客户端向服务器传递附加信息，包括关于请求消息的和关于客户端自己的。这些头域就像是请求消息的修饰器，在语法上等价于编程语言中方法调用的参数。

```
       request-header = Accept                   ;
                      | Accept-Charset           ;
                      | Accept-Encoding          ;
                      | Accept-Language          ;
                      | Authorization            ;
                      | Expect                   ;
                      | From                     ;
                      | Host                     ;
                      | If-Match                 ;
                      | If-Modified-Since        ;
                      | If-None-Match            ;
                      | If-Range                 ;
                      | If-Unmodified-Since      ;
                      | Max-Forwards             ;
                      | Proxy-Authorization      ;
                      | Range                    ;
                      | Referer                  ;
                      | TE                       ;
                      | User-Agent               ;
```

以上是请求头中各种头域的名称，关于每个域名的详细功能和取值范围，可以参考 [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)。

我们还是以上面的网易的首页请求消息分析一下各个头域的作用：

- `Host`: 指定请求的服务器的域名和端口号
- `Connection`: 表示是否需要持久连接。（HTTP 1.1 默认进行持久连接）
- `Cache-Control`: 指定请求和响应遵循的缓存机制
- `Upgrade-Insecure-Requests`: 让浏览器自动升级请求 （由 http 升级成 https）
- `User-Agent`: 包含发出请求的用户信息
- `Accept`: 指定客户端能够接收的内容类型
- `Referer`: 先前网页的地址，当前请求网页紧随其后,即来路
- `Accept-Encoding`: 指定浏览器可以支持的 web 服务器返回内容压缩编码类型
- `Accept-Language`: 浏览器可接受的语言
- `Cookie`: HTTP 请求发送时，会把保存在该请求域名下的所有 cookie 值一起发送给 web 服务器

## 三、 请求正文 message-body

请求头和请求正文之间要有一个 `CRLF`， 这是必须的，表示请求头的结束和请求正文的开始。请求正文中包含用户提交的信息，例如：

```
username=gaopeng&age=25
```

在上面的例子中，由于只是访问了首页而没有进行登录操作，所以请求消息中没有提交数据给服务器。

## 四、 总结

自此，参照了 W3C 标准中的 HTTP 请求消息规范，我们理解了一条客户端发往服务器的请求消息中包含了哪些内容。主要包括：

1. Request-Line；
2. 请求头；
3. 请求正文。

这些是与 request 有关的内容，在后续的部分中，我会继续整理有关 response 有关的内容，即由服务器发往客户端的响应消息。
