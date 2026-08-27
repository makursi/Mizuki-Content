---

## title: HTTP 协议进化史：从 HTTP/0.9 到 HTTP/3
published: 2026-04-05
description: 一文看懂 HTTP 协议的发展历程：HTTP/0.9 单行协议、HTTP/1.x 标准化、SSL/TLS 与 HTTPS、REST 架构、HTTP/2 多路复用，以及基于 QUIC 的 HTTP/3。
image: ''
tags: [HTTP, 网络协议, 前端基础]
category: 'HTTP协议'
draft: false
lang: zh-cn
toc: true

> 本文整理自 MDN 官方文档《Evolution of HTTP》，梳理 HTTP 协议从诞生到 HTTP/3 的完整进化脉络。

## 一、万维网的发明

**1989 年**，在 CERN 工作的 **Tim Berners-Lee** 博士撰写了一份关于"通过网络传输超文本系统"的报告，由此开启了万维网（World Wide Web）时代。

万维网建立在现有的 TCP 和 IP 协议之上，由**四个组成部分**构成：

1. **HTML**——用来表示超文本文档的文本格式（超文本标记语言）
2. **HTTP**——用来交换超文本文档的简单协议（超文本传输协议）
3. **浏览器**——用来显示（以及编辑）超文本文档的客户端，第一个网络浏览器被称为 **WorldWideWeb**
4. **服务器**——用于提供可访问的文档，即 httpd 的前身

:::tip
HTTP 在应用早期非常简单，后来被称为 **HTTP/0.9**，也叫做**单行（one-line）协议**。
:::

## 二、HTTP/0.9——单行协议

最初版本的 HTTP 协议并没有版本号，后来为了区分后续版本，才被定位为 **0.9**。HTTP/0.9 **极其简单**：

- **请求**由单行指令构成，以唯一可用的方法 **GET** 开头，后跟目标资源的路径（连接建立后，协议、服务器、端口号都不是必需的）
- **响应**只包含文档本身，**没有任何 HTTP 头**

```http
GET /mypage.html
```

```html
<html>
这是一个非常简单的 HTML 页面
</html>
```

:::warning
HTTP/0.9 的局限：没有 HTTP 头、没有状态码、没有错误码，**只能传输 HTML 文件**；一旦出现问题，只能发回一个包含问题描述的 HTML 文件供人查看。
:::

## 三、HTTP/1.0——构建可扩展性

由于 HTTP/0.9 功能十分有限，浏览器和服务器迅速扩展协议使其用途更广，主要改进如下：

1. **协议版本号**——版本信息随每个请求发送（HTTP/1.0 被追加到 GET 行）
2. **状态码**——在响应开始时发送，浏览器据此了解请求成功或失败，并调整行为（如更新或使用本地缓存）
3. **HTTP 标头（header）**——请求和响应均可携带元数据，使协议变得灵活、更具扩展性
4. **Content-Type 标头**——支持传输除纯文本 HTML 以外的其他类型文档

一个典型的请求流程：先请求页面，再请求图片。

```http
GET /mypage.html HTTP/1.0
User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)

200 OK
Date: Tue, 15 Nov 1994 08:12:31 GMT
Server: CERN/3.0 libwww/2.17
Content-Type: text/html

<HTML>
一个包含图片的页面
  <IMG SRC="/myimage.gif">
</HTML>
```

第二次连接，请求获取图片（响应类似）：

```http
GET /myimage.gif HTTP/1.0
User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)

200 OK
Date: Tue, 15 Nov 1994 08:12:32 GMT
Server: CERN/3.0 libwww/2.17
Content-Type: text/gif

(这里是图片内容)
```

:::warning
在 **1991—1995 年**，这些新扩展并没有被引入标准以促进协作，只停留在尝试阶段，服务器和浏览器各自实现，产生了大量的**互操作性问题**。
:::

## 四、HTTP/1.1——标准化的协议

HTTP/1.0 的多种实现方式在实际应用中显得混乱。**1997 年初，HTTP/1.1 标准发布**（距 HTTP/1.0 发布仅几个月），消除了大量歧义内容，并引入多项改进：

1. **连接复用（Keep-Alive）**——连接可以复用，节省了多次打开 TCP 连接加载网页资源的时间
2. **管线化（Pipelining）**——允许在第一个响应被完全发送之前就发送第二个请求，以降低通信延迟
3. **响应分块**——支持分块传输（chunked）
4. **缓存控制**——引入额外的缓存控制机制
5. **内容协商**——支持语言、编码、类型等协商，允许客户端和服务器约定以最合适的内容进行交换
6. **Host 标头**——使不同域名可以配置在**同一个 IP 地址**的服务器上（虚拟主机）

一个典型的请求流程——页面和图片的所有请求都通过**同一个连接**完成：

```http
GET /zh-CN/docs/Glossary/CORS-safelisted_request_header HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/zh-CN/docs/Glossary/CORS-safelisted_request_header

200 OK
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Wed, 20 Jul 2016 10:55:30 GMT
Etag: "547fa7e369ef56031dd3bff2ace9fc0832eb251a"
Keep-Alive: timeout=5, max=1000
Last-Modified: Tue, 19 Jul 2016 00:59:33 GMT
Server: Apache
Transfer-Encoding: chunked
Vary: Cookie, Accept-Encoding

(content)
```

```http
GET /static/img/header-background.png HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/zh-CN/docs/Glossary/CORS-safelisted_request_header

200 OK
Age: 9578461
Cache-Control: public, max-age=315360000
Connection: keep-alive
Content-Length: 3077
Content-Type: image/png
Date: Thu, 31 Mar 2016 13:34:46 GMT
Last-Modified: Wed, 21 Oct 2015 18:27:50 GMT
Server: Apache

(image content of 3077 bytes)
```



## 五、超过 15 年的扩展

HTTP 的可扩展性使创建新的头部和方法变得很容易，这 15 年间诞生了多项重要扩展。

### 1. HTTP 用于安全传输（HTTPS）

HTTP 在基本的 TCP/IP 协议栈上发送信息，网景公司（Netscape）在此基础上创建了额外的加密传输层 **SSL**。

SSL 通过加密保证服务器和客户端之间交换消息的**真实性**，支撑了电子商务网站的创建。SSL 在标准化道路上最终成为了 **TLS**。

:::warning
加密传输层的需求愈发高涨：Web 最早几乎是一个学术网络，相对信任度很高；但如今不得不面对一个"险恶的丛林"——广告客户、随机的个人或犯罪分子争相劫取个人信息，甚至改动将要被传输的数据。
:::

随着通过 HTTP 构建的应用程序越来越强大，可以访问越来越多的私人信息（地址簿、电子邮件、地理位置），即使不在电子商务场景下，**对 TLS 的需求也变得普遍**。

### 2. HTTP 用于复杂应用（REST）

**2000 年**，一种新的 HTTP 使用模式被设计出来：**具象状态传输（Representational State Transfer，REST）**。

- API 操作不再通过新的 HTTP 方法传达，而只能通过使用基本的 HTTP/1.1 方法访问**特定 URI**
- 任何 Web 应用程序都可以通过提供 API 来允许查看和修改其数据，而无需更新浏览器或服务器
- 所有需要的内容都被嵌入到网站通过标准 HTTP/1.1 提供的文件中

:::tip
REST 的缺点在于：每个网站都定义了自己的**非标准 RESTful API**，并对其拥有全面的控制权。
:::

自 **2005 年**以来，可用于 Web 页面的 API 大大增加，其中几个 API 为特定目的扩展了 HTTP 协议，大部分是新的特定 HTTP 头：

- **Server-Sent Events（SSE）**——服务器可以偶尔向浏览器推送消息
- **WebSocket**——一个新协议，可以通过升级现有 HTTP 协议来建立



### 3. 放松安全措施——基于当前的 Web 模型

HTTP 和 Web 安全模型——**同源策略**是互不相关的。事实上，当前的 Web 安全模型是在 HTTP 被创造出来**之后**才发展的！

:::tip
这些年来，人们发现如果在特定约束下移除同源策略的部分限制，Web 会更有用。这催生了 **CORS（跨源资源共享）** 和 **CSP（内容安全策略）** 规范——大量成本和时间被花费在通过服务端添加新的 HTTP 头来发送这些策略。
:::

## 六、HTTP/2——为了更优异的表现

**背景**：网页愈渐复杂，甚至演变成了独立的应用程序，媒体播放量、增强交互的脚本体积都大幅增加，更多数据通过 HTTP 请求被传输。而 HTTP/1.1 连接要求请求按正确顺序发送，虽然理论上可以使用一些并行连接（尤其是 5 到 8 个），但带来的成本和复杂性堪忧。

HTTP/2 与 HTTP/1.1 有几处**基本的不同**：

1. **二进制协议**——HTTP/2 是二进制协议而不是文本协议，不再可读、无法方便地手动创建，但可以实施更优化的技术
2. **多路复用（Multiplexing）**——并行的请求能在同一个连接中处理，移除了 HTTP/1.x 中顺序和阻塞的约束
3. **头部压缩**——因为标头在一系列请求中常常相似，HTTP/2 移除了重复和传输重复数据的成本
4. **服务器推送（Server Push）**——允许服务器在客户端缓存中填充数据，通过推送机制提前于请求发送资源



## 七、后 HTTP/2 进化

HTTP 没有停止进化，其扩展性依然被用来添加新功能。**2016 年**的新扩展包括：

- **Alt-Svc**——允许指定给定资源的位置和身份鉴定，支持更智能的 CDN 缓存机制
- **客户端提示（Client Hints）**——允许浏览器或客户端主动向服务端交流其需求或硬件约束信息
- **Cookie 安全前缀**——在 Cookie 标头中引入安全相关的前缀，帮助保证安全的 Cookie 没有被更改



## 八、HTTP/3——基于 QUIC 的 HTTP

HTTP 的下一个主要版本 **HTTP/3** 与早期版本具有**相同的语义**，但在传输层使用 **QUIC** 而非 TCP。

- **更低延迟**——QUIC 旨在为 HTTP 连接设计更低的延迟
- **多路复用 + 每流独立重传**——类似于 HTTP/2，HTTP/3 也是多路复用协议；但 HTTP/2 运行在单个 TCP 连接上，TCP 层的数据包丢失检测和重传会阻塞所有流。而 QUIC 通过 UDP 运行多个流，并为**每个流独立实现数据包丢失检测和重传**，因此发生错误时，只有该数据包中包含数据的流才会被阻塞

:::tip
核心区别一句话：**HTTP/2 的多路复用受制于 TCP 的"队头阻塞"（丢包时全部流被阻塞）；HTTP/3 基于 UDP 的 QUIC 实现了每流独立的丢包重传，从根本上解决了队头阻塞问题。**
:::

## 总结：HTTP 版本进化一览


| 版本       | 发布时间 | 核心特性                         | 关键局限               |
| -------- | ---- | ---------------------------- | ------------------ |
| HTTP/0.9 | 1991 | 单行协议，仅 GET                   | 无标头、无状态码、仅能传输 HTML |
| HTTP/1.0 | 1996 | 版本号、状态码、HTTP 标头、Content-Type | 每次请求新建连接，互操作性问题    |
| HTTP/1.1 | 1997 | 连接复用、管线化、分块、缓存控制、Host 标头     | HTTP 层队头阻塞         |
| HTTP/2   | 2015 | 二进制协议、多路复用、头部压缩、服务器推送        | TCP 层队头阻塞          |
| HTTP/3   | 2022 | 基于 QUIC（UDP）、每流独立丢包重传        | 生态仍在完善中            |


> 资料来源：[MDN — Evolution of HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Evolution_of_HTTP)

