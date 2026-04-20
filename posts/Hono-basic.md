---
title: HonoJS的启动手册
published: 2026-04-20
description: ''
image: ''
tags: []
category: 'Honojs'
draft: false 
lang: ''
---

# 使用bun, pnpm , deno 启动

    bun create hono@latest my-app
    
    pnpm create hono@latest my-app
    
    deno init --npm hono@latest my-app

# 返回JSON数据

以下示例展示了如何处理对 /api/hello GET 请求并返回 application/json 响应。
    
    app.get('/api/hello', (c) => {
      return c.json({
        ok: true,
        message: 'Hello Hono!',
      })
    })

# 请求与响应

获取路径参数、URL 查询值并附加响应标头的方法如下。

    app.get('/posts/:id', (c) => {
      const page = c.req.query('page')//获取查询参数
      const id = c.req.param('id')//获取路径参数
      c.header('X-Message', 'Hi!')
      return c.text(`You want to see ${page} of ${id}`)
    })

也可以处理 POST、PUT 和 DELETE 请求。

    app.post('/posts', (c) => c.text('Created!', 201))
    app.delete('/posts/:id', (c) =>
      c.text(`${c.req.param('id')} is deleted!`)
    )

# 返回HTML
可以使用 HTML 助手或 JSX 语法编写 HTML 代码。将文件重命名为 src/index.tsx 并进行配置

    const View = () => {
      return (
        <html>
          <body>
            <h1>Hello Hono!</h1>
          </body>
        </html>
      )
    }
    
    app.get('/page', (c) => {
      return c.html(<View />)
    })

# 返回原始响应

    app.get('/', () => {
      return new Response('Good morning!')
    })

Response 为 Web 标准Fetch API 内容,详见:https://developer.mozilla.org/en-US/docs/Web/API/Response

# 使用中间件
使用 basicAuth 中间件完成简单用户验证
import { basicAuth } from 'hono/basic-auth'

// ...

    app.use(
      '/admin/*',
      basicAuth({
        username: 'admin',
        password: 'secret',
      })
    )
    
    app.get('/admin', (c) => {
      return c.text('You are authorized!')
    })
    
    
Hono 内置了许多实用的中间件，包括 Bearer 协议以及使用 JWT、CORS 和 ETag 的身份验证。
此外，Hono 还支持使用 GraphQL Server 和 Firebase Auth 等外部库的第三方中间件。当您也可以创建自己的中间件。

# 适配器

有一些适配器用于处理平台相关的功能，例如处理静态文件或 WebSocket。例如，要在 Cloudflare Workers 中处理 WebSocket，请导入 hono/cloudflare-workers 。

    import { upgradeWebSocket } from 'hono/cloudflare-workers'
    
    app.get(
      '/ws',
      upgradeWebSocket((c) => {
        // ...
      })
    )
