---
title: HonoJS的各运行时本地搭建指南
published: 2026-04-20
description: ''
image: ''
tags: []
category: 'Honojs'
draft: false 
lang: ''
---

# Hono:一次编写,世界各处运行

Hono的核心优势就在于,它既可以部署在传统 Node.js 服务器，也可以下沉到全球边缘节点,以下是各平台配置方案(对齐官方)

---
# Cloudflare Workers(最牛逼的,最推荐的)

https://hono.dev/docs/getting-started/cloudflare-workers
---
# 阿里云函数计算
阿里云函数计算是一项完全托管的事件驱动型计算服务。函数计算让开发者专注于编写和上传代码，而无需管理服务器等基础设施。

https://hono.dev/docs/getting-started/ali-function-compute
---
# Cloudflare Pages
Cloudflare Pages 是一个面向全栈 Web 应用的边缘平台。它提供由 Cloudflare Workers 提供的静态文件和动态内容。

https://hono.dev/docs/getting-started/cloudflare-pages
---
# WebAssembly (w/ WASI) 
WebAssembly 是一个安全、沙盒化、可移植的运行时环境，可以在 Web 浏览器内外运行。

https://hono.dev/docs/getting-started/webassembly-wasi
---
# Nodejs
https://hono.dev/docs/getting-started/nodejs
---
# Deno
https://hono.dev/docs/getting-started/deno
---
# Bun(推荐)
https://hono.dev/docs/getting-started/bun
---
# Fastly Compute
一款先进的边缘计算系统，它允许您使用自己喜欢的编程语言，在 Fastly 的全球边缘网络上运行代码。

https://hono.dev/docs/getting-started/fastly
---
# Vercel 
Vercel 是 AI 云平台，提供开发者工具和云基础设施，用于构建、扩展和保护更快、更个性化的网络。

https://hono.dev/docs/getting-started/vercel
---
# Next.js

https://hono.dev/docs/getting-started/nextjs
---
# Netlify

Netlify 提供静态网站托管和无服务器后端服务。Edge Functions 使我们能够实现网页的动态化。

https://hono.dev/docs/getting-started/netlify
---
# AWS Lambda

AWS Lambda 是亚马逊网络服务 (AWS) 提供的无服务器平台。可以根据事件运行代码，它会自动管理底层计算资源。

https://hono.dev/docs/getting-started/aws-lambda
---
# Lambda@Edge

Lambda@Edge 是亚马逊网络服务 (AWS) 的一个无服务器平台。它允许在亚马逊 CloudFront 的边缘节点运行 Lambda 函数，从而可以自定义 HTTP 请求/响应的行为。

https://hono.dev/docs/getting-started/lambda-edge
---
# Azure Functions

Azure Functions 是 Microsoft Azure 提供的无服务器平台.可以根据事件运行代码，它会自动为您管理底层计算资源。

https://hono.dev/docs/getting-started/azure-functions
---
# Google Cloud Run

Google Cloud Run 是由 Google Cloud 构建的无服务器平台。可以根据事件运行代码，Google 会自动管理底层计算资源。

https://hono.dev/docs/getting-started/google-cloud-run
---
# Supabase Edge Functions
Supabase 是 Firebase 的开源替代方案，提供了一套与 Firebase 功能类似的工具，包括数据库、身份验证、存储，以及现在的无服务器函数。

https://hono.dev/docs/getting-started/supabase-functions
---

# Service Worker
Service Worker 是一个在浏览器后台运行的脚本，用于处理缓存和推送通知等任务。通过 Service Worker 适配器，开发者可以将使用 Hono 开发的应用程序作为 FetchEvent 处理程序在浏览器中运行。

https://hono.dev/docs/getting-started/service-worker
---
