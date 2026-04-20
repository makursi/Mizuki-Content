---
title: HonoJS-中间件
published: 2026-04-20
description: ''
image: ''
tags: []
category: 'Honojs'
draft: false 
lang: ''
---

# 洋葱模型的概念

## 我们将返回 Response 基本函数称为“Handler”。“Middleware”在 Handler 执行前后执行，负责处理 Request 和 Response 。它的结构类似于洋葱。
---

![Alt text](./hono/onion.png)
### 1. Handler（处理器）
- **定义**：返回 `Response` 对象的**最底层函数**，是请求的**最终业务逻辑**。
- **作用**：接收请求 → 执行业务 → 返回响应。
- **特点**：每个路由只会执行**一个 Handler**。

### 2. Middleware（中间件）
- **定义**：在 **Handler 之前、之后** 都能执行的函数。
- **能力**：
  - 拿到并修改 `Request`
  - 拿到并修改 `Response`
  - 决定是否继续往下执行
- **定位**：通用逻辑（日志、鉴权、跨域、响应头、错误处理）的**复用容器**。

### 请求像从洋葱外皮穿到中心，响应再从中心穿回外皮。
**Hono 中间件 = 洋葱模型**


---

## 二、洋葱模型（Onion Structure）到底是什么？
### 总结:
**请求先进外层中间件 → 一层层往里走到 Handler → 响应再一层层往外返回，反向执行中间件后半段。**

### 2. 洋葱模型的可视化流程
```
请求 →
  中间件1 前半段
    中间件2 前半段
      中间件3 前半段
        Handler（业务核心）
      中间件3 后半段
    中间件2 后半段
  中间件1 后半段
→ 响应返回
```

### 3. 执行顺序铁律
1. **按注册顺序进入**：先注册的先执行前半段
2. **靠 await next() 穿透**：调用 next 才会进入下一层
3. **响应时逆序退出**：后进入的先执行后半段

---

## 三、HonoJS官方示例代码解析

https://hono.dev/docs/concepts/middleware
文档给出的**计算响应时间**是中间件：
```typescript
app.use(async (c, next) => {
  // 1. 请求阶段：进入中间件（洋葱外皮）
  const start = performance.now()

  // 2. 穿透到下一个中间件 / Handler（洋葱中心）
  await next()

  // 3. 响应阶段：回来执行（洋葱外皮，逆序）
  const end = performance.now()
  c.res.headers.set('X-Response-Time', `${end - start}`)
})
```

### 执行步骤拆解
1. **请求进来**
   - 记录开始时间 `start`
   - 执行 `await next()` → 控制权交给下一层
2. **穿透到最内层**
   - 执行路由 Handler（业务逻辑）
   - 生成 Response
3. **响应返回**
   - 从 `await next()` 恢复执行
   - 记录结束时间 `end`
   - 计算耗时，写入响应头

这就是**一次完整的洋葱穿透**：先进后出。

---

## 四、洋葱模型多中间件完整演示
```typescript
// 中间件 A
app.use(async (c, next) => {
  console.log('A 进入')
  await next()
  console.log('A 退出')
})

// 中间件 B
app.use(async (c, next) => {
  console.log('B 进入')
  await next()
  console.log('B 退出')
})

// 路由 Handler
app.get('/', async (c) => {
  console.log('Handler 执行')
  return c.text('Hello Hono')
})
```

### 控制台输出（严格洋葱顺序）
```
A 进入
B 进入
Handler 执行
B 退出
A 退出
```

---

## 五、洋葱模型带来的核心价值
1. **请求/响应都能管**
   - next 之前：改请求、鉴权、日志
   - next 之后：改响应、加头、封装格式、统计耗时
2. **逻辑解耦**
   - 日志、CORS、JWT、缓存各自独立
3. **执行顺序可控**
   - 先注册先入，后注册后入
   - 响应逆序返回，完美包裹业务逻辑

---
用这种**极简写法**就能写自定义中间件,也能直接用官方/社区中间件：`logger`、`cors`、`jwt`、`basicAuth` 等

---
