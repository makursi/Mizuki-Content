---
title: HonoJS开始
published: 2026-04-20
description: ''
image: ''
tags: []
category: 'Honojs'
draft: false 
lang: ''
---

# Hono
## Hono—— *日语意为火焰🔥* ——是一个基于 Web 标准构建的小型、简单且超高速的网页框架。
# 但它支持任何 JavaScript 运行时：Cloudflare Workers、Fastly Compute、Deno、Bun、Vercel、Netlify、AWS Lambda、Lambda@Edge 和 Node.js。(最强的优势)

## 快速入门

    pnpm create hono@latest

    bun create hono@latest

    deno init --npm hono@latest


## 为什么选择hono而不是express? 
- 超高速 🚀——路由器 **RegExpRouter** 非常快。不用线性循环。快点。

- 轻量 🪶级——hono/tiny 预设不到 **14kB**。**Hono 没有任何依赖，只使用 Web 标准。**

- 多运行时 🌍——支持 **Cloudflare Workers、Fastly Compute、Deno、Bun、AWS Lambda 或 Node.js。** 所有平台上运行的代码都是一样的。

- 内置 🔋电池——Hono 内置中间件、定制中间件、第三方中间件和辅助工具。包括电池。

- 令人愉快的 DX 😃——超级干净的 API。一流的 TypeScript 支持。现在，我们有了“类型”。


## Hono的应用场景

Hono 是一个简单的网页应用框架，类似于 Express，但没有前端。但它运行在 CDN Edge 上，结合中间件时可以构建更大的应用。以下是一些使用场景的例子。

- Building Web APIs-构建 Web API
- Proxy of backend servers-后端服务器代理
- Front of CDN-前端CDN资源
- Edge application-边缘应用
- Base server for a library-库的基服务器
- Full-stack application-全栈应用

### 那些企业在使用它?

官方详细介绍: https://hono.dev/docs/


## Hono一分钟快速使用

    import { Hono } from 'hono'
    const app = new Hono()
    
    app.get('/', (c) => c.text('Hono!'))
    
    export default app

这honojs代码中体现出来的东西

### 1. 极快

    Hono x 402,820 ops/sec ±4.78% (80 runs sampled)
    itty-router x 212,598 ops/sec ±3.11% (87 runs sampled)
    sunder x 297,036 ops/sec ±4.76% (77 runs sampled)
    worktop x 197,345 ops/sec ±2.40% (88 runs sampled)
    Fastest is Hono
    ✨  Done in 28.06s.

### 2. 极轻

它使用 hono/tiny 预设后，压缩后的大小不到 **14KB** 。它包含许多中间件和适配器，但只有在需要时才会打包。作为参考，**Express 的大小为 572KB**。


### 3. 路由器极好

Hono 拥有多个路由器 。

### RegExpRouter 是 JavaScript 世界中最快的路由工具。它使用在分发前创建的单个大型正则表达式来匹配路由。借助 SmartRouter ，它支持所有路由模式。

LinearRouter 注册路由的速度非常快，因此适用于每次启动应用程序都需要初始化的环境。PatternRouter 只是简单地添加和匹配模式，因此体积很小。

- https://hono.dev/docs/concepts/routers 有关hono路由相关知识

### 4. 极标准

Hono采用了 Web Standards, 可以在**多平台运行**
- **Cloudflare Workers**
- Cloudflare Pages  Cloudflare 
- Fastly Compute  
- Deno  
- Bun  
- Vercel  
- AWS Lambda
- Lambda@Edge
- Others  

https://hono.dev/docs/concepts/web-standard 有关hono网络标准相关知识

### 5. 中间件和辅助程序

Basic Authentication 
Bearer Authentication
Body Limit
Cache
Compress
Context Storage
Cookie
CORS
ETag
html
JSX
JWT Authentication
Logger
Language
Pretty JSON
Secure Headers
SSG
Streaming
GraphQL Server
Firebase Authentication
Sentry

使用 Hono 添加 ETag 和请求日志记录只需要几行代码：

    import { Hono } from 'hono'
    import { etag } from 'hono/etag'
    import { logger } from 'hono/logger'
    
    const app = new Hono()
    app.use(etag(), logger())

https://hono.dev/docs/concepts/middleware hono官方中间件相关知识
