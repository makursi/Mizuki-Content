---
title: HonoJS-Web Standards
published: 2026-04-20
description: ''
image: ''
tags: []
category: 'Honojs'
draft: false 
lang: ''
---
# Hono 仅使用 Fetch 等 Web 标准 。

这些标准最初用于 fetch 函数，由处理 HTTP 请求和响应的基本对象组成。除了 **Requests 和 Responses** 之外，还有 **URL 、 URLSearchParam 、Headers**。

**Cloudflare Workers、Deno 和 Bun 也都基于 Web 标准构建。**

例如，一个返回“Hello World”的服务器可以编写如下代码。它可以在 Cloudflare Workers 和 Bun 上运行。

    export default {
      async fetch() {
        return new Response('Hello World')
      },
    }
    
**Hono 仅使用 Web 标准，这意味着 Hono 可以在任何支持这些标准的运行时环境中运行。**

此外，Hono还提供了一个 Node.js 适配器。Hono 可在以下运行时环境中运行：

- **Cloudflare Workers(最适合honojs的运行时)**
- Deno 
- Bun  
- Fastly Compute 
- AWS Lambda
- Node.js
- Vercel (edge-light) 
- WebAssembly
