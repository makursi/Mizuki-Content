---
title: Nuxt4的渲染模式
published: 2026-03-27
description: ''
image: ''
tags: ['Nuxt','render']
category: 'Nuxt4'
draft: false 
lang: ''
---

# 渲染模式(Rendering Modes)

Nuxt 支持不同的渲染模式，包括**通用渲染、客户端渲染**，同时也提供**混合渲染**以及**将应用程序渲染在 CDN 边缘服务器。**

渲染为**浏览器和服务器解析 JavaScript 代码，将 Vue.js 组件转换为 HTML 元素**
 
#  Nuxt的通用渲染模式(universal rendering)?

Nuxt通用渲染模式与传统由 PHP 或 Ruby 应用程序执行的伺服器端渲染类似,渲染流程如下:

1.  当浏览器**请求启用了通用渲染的 URL** 时，**Nuxt** 会在(Nodejs)**运行时环境中运行 JavaScript（Vue.js）代码**，并**将完全渲染的 HTML 页面返回给浏览器。**

2.  如果页面已预先生成，Nuxt 也可能从**缓存**中返回完全渲染的 HTML 页面。与客户端渲染不同，用户立即获得应用程序的初始全部内容。

3.  一旦 HTML文档被下载，**浏览器会解析该文档**，然后 Vue.js 接管该文档。

先前在运行时上运行的相同 JavaScript 代码现在在客户端（浏览器）中初始化应用，通过**将其监听器绑定到 HTML，实现应用的交互性**。这被称为**水合**。

4.  当水合完成后,页面可以享受**动态界面**和**页面过渡**等好处。


## 通用渲染下的运行流程

通用模式中, 在服务器和客户端会运行同一个Vue文件内容中的不同部分:

    <script setup lang="ts">
    const counter = ref(0) // executes in server and client environments
    
    const handleClick = () => {
    counter.value++ // executes only in a client environment
    }
    </script>
    
    <template>
    <div>
        <p>Count: {{ counter }}</p>
        <button @click="handleClick">
        Increment
        </button>
    </div>
    </template>

在这个文件的初始请求中, 由于counter变量是在标签内渲染的,但countr,ref又在服务器nodejs环境下被初始化的.

类似 handleClick 里的内容在服务端里永远不会执行,就像客户端常用的DOM, BOM概念一样.

在浏览器执行水合,也就是将事件监听器绑定到HTML页面的过程中,
counter ref 会被重新初始化, handleClick最终绑定到了按钮上面.
handleClick 的主体始终在浏览器环境中运行。



**不同渲染模式下,Nuxt目录文件内容的渲染规则不同:**

- 在 水合(hydration) 过程中，中间件(**Middlewares**)和页面(**pages**)在服务器和客户端上运行。

- 插件(**plugins**)可以在服务器上、客户端或两者上渲染。

- 组件(**Components**)也可以被强制仅在客户端上运行(使用`<clientOnly></clientOnly>`组件进行包裹)。

- 可组合项(**Composable**)和工具(**utilities**)会根据其使用上下文进行渲染。

##  通用渲染,SSR渲染模式下的优势:

性能:

用户可以立即访问页面的内容，因为浏览器显示静态内容比显示 JavaScript 生成的内容快得多。同时，**Nuxt 在 hydration 过程中保留了 Web 应用的交互性。**

搜索引擎优化：

通用渲染将页面的整个 HTML 内容作为经典的服务器应用程序发送到浏览器。**网络爬虫可以直接索引页面的内容**，这使得通用渲染成为任何希望快速索引的内容的理想选择。

## 通用渲染,SSR渲染模式下的缺点:

开发限制：

服务器和浏览器环境提供的 API 不同，难以编写可在两边无缝运行的代码。

成本：

为了动态渲染页面，服务器需要持续运行。这像任何传统服务器一样增加了每月成本,但是
- 由于使用浏览器进行客户端导航，通用渲染大大减少了服务器调用。
- 通过利用边缘端(vercel,cloudflare)渲染，可以实现成本降低。

# 客户端渲染(Client-Side Rendering)

**客户端渲染（CSR）是指在浏览器中使用 JavaScript 生成 HTML 内容。**


## 关于Nuxt客户端渲染的流程:

原生 Vue SPA 默认是客户端渲染：浏览器必须先下载并跑完所有 JS，才能生成页面 HTML 展示内容。


## 客户端渲染的优势:

开发速度：

当完全在客户端工作时，我们不必担心**代码的服务器兼容性问题**，例如，通过使用仅限浏览器的 API，如 window 对象。


更经济：

**运行服务器会增加基础设施成本**，因为您需要在一个支持 JavaScript 的平台运行。
我们可以将纯客户端应用程序托管在任何静态服务器上，只需 HTML、CSS 和 JavaScript 文件即可。

离线：

由于代码完全在浏览器中运行，因此即使互联网不可用，它也能很好地继续工作。


## 客户端渲染的缺点:

性能：

用户必须等待**浏览器下载、解析和运行所有需要的 JavaScript 文件**。根据下载部分的网络状况以及解析和执行部分的用户设备，这可能需要一些时间并影响用户体验。


搜索引擎优化(SEO)：

**通过客户端渲染交付的内容进行索引和更新比服务器渲染的 HTML 文档需要更多时间**。这与我们讨论的性能缺陷有关，因为搜索引擎爬虫不会等待界面在第一次尝试索引页面时完全渲染。使用纯客户端渲染，您的内容在搜索引擎结果页面中显示和更新的时间会更长。

**客户端渲染适用于不需要索引或用户频繁访问的高度交互式 Web 应用程序**

# Nuxt的混合渲染模式(Hybrid Rendering)

混合渲染支持通过路由规则（Route Rules） 为不同路由单独配置缓存策略，同时规定服务器针对特定 URL 的新请求该如何响应。

在旧版 Nuxt 中，整个应用的所有路由、页面必须统一渲染模式：要么全量 SSR 同构渲染，要么全量客户端渲染。

例如内容类网站：内容页面适合静态化生成，后台管理模块则需要权限登录、更偏向动态 SPA 交互。

Nuxt 现已内置路由规则与混合渲染能力。借助路由规则，可批量匹配路由，单独修改对应页面的渲染模式、配置专属缓存策略。

Nuxt 服务会自动注册对应中间件，依托 Nitro 服务缓存层，为匹配路由包装缓存处理逻辑。

## Nuxt 包括路由规则和混合渲染支持。
使用路由规则，您可以为一组 nuxt 路由定义规则，更改渲染模式或根据路由分配缓存策略！

在 nuxt.config.ts中进行具体的配置:

    export default defineNuxtConfig({
      routeRules: {
        // 首页：在项目构建时提前预渲染（静态生成）
        '/': { prerender: true },
      
        // 产品列表页：按需生成，后台自动刷新，API 数据变化前一直缓存
        '/products': { swr: true },
        
        // 产品详情页：按需生成，后台自动刷新，缓存 1 小时（3600秒）
        '/products/**': { swr: 3600 },
        
        // 博客列表页：按需生成，后台自动刷新，在 CDN 上缓存 1 小时
        '/blog': { isr: 3600 },
        
        // 博客文章页：构建/部署时生成一次，下次部署前一直 CDN 缓存
        '/blog/**': { isr: true },
        
        // 管理后台：仅在客户端渲染（关闭 SSR，纯 SPA 模式）
        '/admin/**': { ssr: false },
        
        // API 路由：自动添加跨域 CORS 响应头
        '/api/**': { cors: true },
        
        // 旧页面：自动重定向到新页面
        '/old-page': { redirect: '/new-page' },
      },
    })

## 对于不同路由规则的渲染:

- redirect: string定义服务端重定向，将请求路由跳转到指定地址。

- ssr: boolean禁用应用部分页面的 HTML 服务端渲染；设置 ssr: false 后，这些页面仅在浏览器中进行客户端渲染。

- cors: boolean设置 cors: true 会自动添加跨域资源共享（CORS）响应头；可通过 headers 配置自定义覆盖跨域参数。

-  headers: object为网站指定页面 / 资源添加自定义 HTTP 响应头（例如静态资源、接口）。

- swr: number | boolean为服务端响应添加缓存头，在服务端 / 反向代理中缓存完整响应，支持配置缓存有效期（TTL）。Nitro 的 node-server 预设可缓存完整响应；缓存过期后，会先返回旧缓存，同时在后台重新生成页面。设置 true 时，仅添加 stale-while-revalidate 缓存头，不设置缓存过期时间。

- isr: number | boolean作用与 swr 完全一致，区别：会将响应缓存到支持的平台（Netlify/Vercel）的 CDN 上。设置 true 时，内容会一直缓存在 CDN，直到下次项目部署才更新。

- prerender: boolean在项目构建阶段提前渲染路由，并将生成的页面作为静态资源打包进构建产物。

- noScripts: boolean禁用网站指定页面的 Nuxt 脚本、JS 资源预加载提示渲染，页面仅保留静态 HTML。

- appMiddleware: string | string[] | Record<string, boolean>为 Vue 应用内的页面路径（非 Nitro 服务端路由）定义需要 / 不需要执行的应用中间件。


# 边缘渲染(Edge-Side Rendering (ESR) )

边缘渲染（ESR）,通过内容分发网络（CDN）的边缘服务器，将Nuxt 应用程序更靠近用户进行渲染。通过利用 ESR，确保性能提升和延迟减少，从而提供更优的用户体验。

使用 ESR，渲染过程被推到网络的"边缘"——CDN 的边缘服务器,ESR 更是一个部署目标，而非实际的渲染模式。

当用户请求页面时，请求不会直接发送到源服务器，而是由最近的边缘服务器拦截处理。边缘服务器会直接生成页面 HTML 并返回给用户。这一过程最大限度缩短了数据传输的物理距离，降低延迟，让页面加载更快。

边缘渲染之所以能够实现，得益于驱动 Nuxt 的底层服务引擎 Nitro。它提供了跨平台支持，可运行在 Node.js、Deno、Cloudflare Workers 等多种环境中。

目前支持使用 ESR 的平台包括：
- Cloudflare Pages：通过 Git 集成与 nuxt build 命令，零配置直接启用
- Vercel：使用 nuxt build 命令，并配置环境变量 NITRO_PRESET=vercel-edge
- Netlify：使用 nuxt build 命令，并配置环境变量 NITRO_PRESET=netlify-edge

注意：在使用边缘渲染时，**路由规则（route rules）** 依然可以正常使用，以实现**混合渲染能力。**
