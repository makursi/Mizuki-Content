---
title: Fetch API
published: 2026-04-22
description:
image: ''
tags: []
category: 'Web API'
draft: false 
lang: 'zh'
---

# Fetch API

Fetch API 提供了一个用于获取资源（包括跨网络通信）的接口。它是 **XMLHttpRequest** 的一个更强大、更灵活的替代。

## 简介

- Fetch API 使用 `Request` 和 `Response` 对象（以及网络请求涉及的其他内容），还涉及 CORS 和 HTTP Origin 标头语义等相关概念。

- 要发起请求并获取资源，请使用 `fetch()` 方法。该方法在 `Window` 和 `Worker` 上下文中均为全局方法。这意味着在几乎任何需要获取资源的上下文中，都可以使用它。

- `fetch()` 方法有一个必需参数，即要获取资源的路径。它返回一个 `Promise`，该 Promise 会在服务器返回头部信息后，立即解析兑现为该请求的 `Response` 对象——**即使服务器的响应是 HTTP 错误状态**。你也可以传入一个可选的 `init` 选项对象作为第二个参数（参见 `Request`）。

- 一旦获取到 `Response` 对象后，就有多种方法可用于定义主体内容以及如何处理该内容。虽然可以直接使用 `Request()` 和 `Response()` 构造函数创建请求和响应对象.通常，它们会作为其他 API 操作的结果被创建（例如，来自 service worker 的 `FetchEvent.respondWith()`）。

# Fetch API的使用

Fetch API提供了一个全局 fetch() 方法

Fetch 还提供了专门的逻辑空间来定义其他与 HTTP 相关的概念，例如 CORS 和 HTTP 的扩展。

`fetch` 规范与 `jQuery.ajax()` 主要有以下的不同：

-   当接收到一个代表错误的 HTTP 状态码时，从 `fetch()` 返回的 Promise **不会被标记为 reject**，即使响应的 HTTP 状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve（如果响应的 HTTP 状态码不在 200 - 299 的范围内，则设置 resolve 返回值的 `ok` 属性为 false），**仅当网络故障时或请求被阻止时，才会标记为 reject。**

-   `fetch` **不会发送跨域 cookie**，除非你使用了 _credentials_ 的初始化选项。

## 一个基本的 fetch 请求:

    fetch("http://example.com/movies.json")
      .then((response) => response.json())
      .then((data) => console.log(data));

想要通过网络获取一个 JSON 文件并将其打印到控制台。

最简单的操作方法就是:
- 只提供一个参数用来指明想 fetch() 到的资源路径
- 然后返回一个包含响应结果的 promise（一个 Response 对象）。

它是一个 HTTP 响应，而不是真的 JSON。为了获取 JSON 的内容，我们需要使用 json()方法（该方法返回一个将响应 body 解析成 JSON 的 promise对象）。

## fetch()方法的参数

fetch(resource, options)

参数一可选择为：
 1. 字符串 URL 
 2. Request 实例对象
 
`fetch()` 接受第二个可选参数，一个可以控制不同配置的 `init` 对象：

    async function postData(url = "", data = {}) {
      const response = await fetch(url, {
        method: "POST", 
        mode: "cors", 
        cache: "no-cache", 
        credentials: "same-origin", 
        headers: {
          "Content-Type": "application/json",
       
        },
        redirect: "follow", 
        referrerPolicy: "no-referrer", 
        body: JSON.stringify(data), 
      });
      return response.json(); 
    }
    
    postData("https://example.com/answer", { answer: 42 }).then((data) => {
      console.log(data); 
    });

注意：`mode: "no-cors"` 仅允许使用一组有限的 HTTP 请求头：

-   `Accept`
-   `Accept-Language`
-   `Content-Language`
-   `Content-Type` 允许使用的值为：`application/x-www-form-urlencoded`、`multipart/form-data` 或 `text/plain`

## 发送带凭据的请求

浏览器环境中，带凭据的请求（Credentialed Request），就是在 HTTP 请求里带上用户的身份凭据，让服务器能识别出 “你是谁”，从而返回与你身份相关的内容（比如登录态、个人数据）。

浏览器中常见的凭据:

1.  Cookie（最常见）
服务器种在你浏览器里的 Session ID、登录态 Cookie 等
2.  HTTP Basic / Digest 认证
浏览器自动附加的 Authorization 请求头，包含用户名密码的 Base64 编码或摘要信息。
3.  TLS 客户端证书
少数金融 / 企业级场景会用到的客户端证书，用于双向身份认证。

为了让浏览器发送包含凭据的请求（即使是跨域源），要将 credentials: 'include' 添加到传递给 fetch() 方法的 init 对象。

    fetch("https://example.com", {
      credentials: "include",
    });

如果只想在请求 URL 与调用脚本位于同一源时发送凭据，添加 credentials: 'same-origin'。

要改为确保浏览器不在请求中包含凭据，使用 credentials: 'omit'。
    
    fetch("https://example.com", {
      credentials: "omit",
    });


## 上传 JSON 数据
使用 fetch() POST JSON 数据

    const data = { username: "example" };
    
    fetch("https://example.com/profile", {
      method: "POST", // or 'PUT'
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(data),
    })
      .then((response) => response.json())
      .then((data) => {
        console.log("Success:", data);
      })
      .catch((error) => {
        console.error("Error:", error);
      });

这段代码创建用户信息数据，以 JSON 格式通过 POST 请求发送到指定接口，然后解析服务器返回的 JSON 响应并打印结果，同时捕获请求或解析过程中出现的错误。

## 上传文件
通过 HTML `<input type="file" />` 元素，`FormData()`和 `fetch()` 上传文件。
    
    const formData = new FormData();
    const fileField = document.querySelector('input[type="file"]');
    
    formData.append("username", "abc123");
    formData.append("avatar", fileField.files[0]);
    
    fetch("https://example.com/profile/avatar", {
      method: "PUT",
      body: formData,
    })
      .then((response) => response.json())
      .then((result) => {
        console.log("Success:", result);
      })
      .catch((error) => {
        console.error("Error:", error);
      });

## 上传多个文件

通过 HTML `<input type="file" multiple />` 元素，`FormData()` 和 `fetch()` 上传文件。


    const formData = new FormData();
    const photos = document.querySelector('input[type="file"][multiple]');
    
    formData.append("title", "My Vegas Vacation");
    for (let i = 0; i < photos.files.length; i++) {
      formData.append(`photos_${i}`, photos.files[i]);
    }
    
    fetch("https://example.com/posts", {
      method: "POST",
      body: formData,
    })
      .then((response) => response.json())
      .then((result) => {
        console.log("Success:", result);
      })
      .catch((error) => {
        console.error("Error:", error);
      });

## 检测请求是否成功

如果遇到网络故障或服务端的 CORS 配置错误时，`fetch()` promise 将会 reject，带上一个 `TypeError` 对象。

虽然这个情况经常是遇到了权限问题或类似问题——比如 404 不是一个网络故障。想要精确的判断 `fetch()` 是否成功，需要包含 promise resolved 的情况，此时再判断 `Response.ok` 是否为 true。类似以下代码：

    fetch("flowers.jpg")
      .then((response) => {
        if (!response.ok) {
          throw new Error("Network response was not OK");
        }
        return response.blob();
      })
      .then((myBlob) => {
        myImage.src = URL.createObjectURL(myBlob);
      })
      .catch((error) => {
        console.error("There has been a problem with your fetch operation:", error);
      });

blob():把响应体解析成 Blob 对象(二进制大对象)
- 用于：图片、音频、视频、文件
- 返回：Promise，解析完给一个 文件数据
- 用 URL.createObjectURL(blob) 变成图片地址显示


## 自定义请求对象

通过使用 `Request()`构造函数来创建一个 request 对象，然后再作为参数传给 `fetch()`：

    const myHeaders = new Headers();
    
    const myRequest = new Request("flowers.jpg", {
      method: "GET",
      headers: myHeaders,
      mode: "cors",
      cache: "default",
    });
    
    fetch(myRequest)
      .then((response) => response.blob())
      .then((myBlob) => {
        myImage.src = URL.createObjectURL(myBlob);
      });

Request() 和 fetch() 接受同样的参数,可以传入一个已存在的 request 对象来创造一个拷贝：

    const anotherRequest = new Request(myRequest, myInit);

# Headers

使用 Headers 的接口通过 Headers() 构造函数来创建一个自定义的 headers 对象。

一个 headers 对象是一个简单的多键值对：

    const content = "Hello World";
    const myHeaders = new Headers();
    myHeaders.append("Content-Type", "text/plain");
    myHeaders.append("Content-Length", content.length.toString());
    myHeaders.append("X-Custom-Header", "ProcessThisImmediately");

也可以传入一个多维数组或者对象字面量：

    const myHeaders = new Headers({
      "Content-Type": "text/plain",
      "Content-Length": content.length.toString(),
      "X-Custom-Header": "ProcessThisImmediately",
    });
    
获取它的内容：

    console.log(myHeaders.has("Content-Type")); // true
    console.log(myHeaders.has("Set-Cookie")); // false
    myHeaders.set("Content-Type", "text/html");
    myHeaders.append("X-Custom-Header", "AnotherValue");
    
    console.log(myHeaders.get("Content-Length")); // 11
    console.log(myHeaders.get("X-Custom-Header")); // ['ProcessThisImmediately', 'AnotherValue']
    
    myHeaders.delete("X-Custom-Header");
    console.log(myHeaders.get("X-Custom-Header")); // null

抛出 TypeError 异常的情况:

- 如果使用了一个不合法的 HTTP Header 属性名，那么 Headers 的方法通常都抛出 TypeError 异常
- 如果不小心写入了一个不可写的属性 

除此以外的情况，失败了并不抛出异常。例如：

    const myResponse = Response.error();
    try {
      myResponse.headers.set("Origin", "http://mybank.com");
    } catch (e) {
      console.log("Cannot pretend to be a bank!");
    }

应该在使用之前检查内容类型 content-type 是否正确，比如：

    fetch(myRequest)
      .then((response) => {
        const contentType = response.headers.get("content-type");
        if (!contentType || !contentType.includes("application/json")) {
          throw new TypeError("Oops, we haven't got JSON!");
        }
        return response.json();
      })
      .then((data) => {
        /* 拿到数据后, 随便处理 */
      })
      .catch((error) => console.error(error));

# Response 对象

Response 实例是在 fetch() 处理完 promise 之后返回的。

Response() 构造方法接受两个可选参数:
- response 的 body 
- 一个初始化对象（与Request() 所接受的 init 参数类似）。

最常见的 response 属性有：

-   `Response.status` — 整数（默认值为 200）为 response 的状态码。
-   `Response.statusText` — 字符串（默认值为 ""），该值与 HTTP 状态码消息对应。注意：HTTP/2 不支持状态消息
-   `Response.ok` — 如上所示，该属性是来检查 response 的状态是否在 200 - 299（包括 200 和 299）这个范围内。该属性返回一个布尔值。

它的实例也可用通过 JavaScript 来创建，但只有在 `ServiceWorkers` 中使用 `respondWith()`方法并提供了一个自定义的 response 来接受 request 时才真正有用：

    const myBody = new Blob();
    
    addEventListener("fetch", (event) => {
      // ServiceWorker intercepting a fetch
      event.respondWith(
        new Response(myBody, {
          headers: { "Content-Type": "text/plain" },
        }),
      );
    });

# Body

不管是请求还是响应都能够包含 body 对象。body 也可以是以下任意类型的实例。

-   `ArrayBuffer`
-   `ArrayBufferView` (Uint8Array 等)
-   `Blob`/File
-   string
-   `URLSearchParams`
-   `FormData`

Body 类定义了以下方法（这些方法都被 `Request` 和 `Response`所实现）以获取 body 内容。这些方法都会返回一个被解析后的 Promise 对象和数据。

-   `Request.arrayBuffer()` / `Response.arrayBuffer()`
-   `Request.blob()` / `Response.blob()`
-   `Request.formData()` / `Response.formData()`
-   `Request.json()` / `Response.json()`
-   `Request.text()` / `Response.text()`

请求体可以由传入 body 参数来进行设置：

    const form = new FormData(document.getElementById("login-form"));
    fetch("/login", {
      method: "POST",
      body: form,
    });

request 和 response（包括 fetch() 方法）都会试着自动设置 Content-Type。如果没有设置 Content-Type 值，发送的请求也会自动设值。
