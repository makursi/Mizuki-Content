---
title: HonoJS-路由器
published: 2026-04-20
description: ''
image: ''
tags: []
category: 'Honojs'
draft: false 
lang: ''
---

# Hono有五台路由器

## RegExpRouter 正则表达式路由器

RegExpRouter 是 JavaScript 世界中最快的路由器。

它并非像 Express 那样使用路径到正则表达式的实现，而是**使用了线性循环。** 因此，所有路由都会进行正则表达式匹配，**路由越多，性能越差。**

Hono 的 RegExpRouter 将路由模式转换成“一个大型正则表达式”。然后，它只需一次匹配即可获得结果。
在大多数情况下，这种方法比使用基于树的算法（例如基数树）的方法速度更快。
但是，RegExpRouter 并不支持所有路由模式，因此它通常与下面支持所有路由模式的其他路由器之一结合使用。

## TrieRouter

TrieRouter 是使用 Trie 树算法的路由器。与 RegExpRouter 类似，它不使用线性循环。
这个路由器的速度不如 RegExpRouter，但比 Express 路由器快得多。TrieRouter 支持所有模式。


## SmartRouter

当您使用多个路由器时， SmartRouter 非常有用。它会根据已注册的路由器推断出最佳路由器。Hono 默认使用 SmartRouter、RegExpRouter 和 TrieRouter：

    // Inside the core of Hono.
    readonly defaultRouter: Router = new SmartRouter({
      routers: [new RegExpRouter(), new TrieRouter()],
    })

应用程序启动时，SmartRouter 会根据路由检测出速度最快的路由器，并继续使用该路由器。


## LinearRouter 线性路由器

RegExpRouter 速度很快，但路由注册阶段可能会稍慢。因此，它不适合每次请求都进行初始化的环境。
LinearRouter 针对“一次性”场景进行了优化。由于它采用线性方法添加路由，无需编译字符串，因此路由注册速度比 RegExpRouter 快得多。

官方测试: 

    • GET /user/lookup/username/hey
    ----------------------------------------------------- -----------------------------
    LinearRouter     1.82 µs/iter      (1.7 µs … 2.04 µs)   1.84 µs   2.04 µs   2.04 µs
    MedleyRouter     4.44 µs/iter     (4.34 µs … 4.54 µs)   4.48 µs   4.54 µs   4.54 µs
    FindMyWay       60.36 µs/iter      (45.5 µs … 1.9 ms)  59.88 µs  78.13 µs  82.92 µs
    KoaTreeRouter    3.81 µs/iter     (3.73 µs … 3.87 µs)   3.84 µs   3.87 µs   3.87 µs
    TrekRouter       5.84 µs/iter     (5.75 µs … 6.04 µs)   5.86 µs   6.04 µs   6.04 µs
    
    summary for GET /user/lookup/username/hey
      LinearRouter
      2.1x faster than KoaTreeRouter
      2.45x faster than MedleyRouter
      3.21x faster than TrekRouter
      33.24x faster than FindMyWay

## PatternRouter   模式路由器
PatternRouter 是 Hono 路由器中最小的路由器。
虽然 Hono 本身已经很小巧，但如果需要在资源有限的环境中进一步缩小其体积，请使用 PatternRouter。
仅使用 PatternRouter 的应用程序大小小于 15KB。

    $ npx wrangler deploy --minify ./src/index.ts
    ⛅️ wrangler 3.20.0
    -------------------
    Total Upload: 14.68 KiB / gzip: 5.38 KiB

# 如何使用这五台路由器?
**90%场景直接用默认 SmartRouter 即可**；只有边缘/极致体积/每次请求初始化等特殊场景，才手动指定路由。
---

## 5种路由核心用法
| 路由 | 速度 | 特点 | 适用场景 | 必选理由 |
| :--- | :--- | :--- | :--- | :--- |
| **RegExpRouter** | ⚡️ 最快 | 编译成单一大正则，一次匹配；不支持所有路由写法 | 标准REST、路径简单、追求极致QPS | 最快匹配速度** |
| **TrieRouter** | ⚡️⚡️ 快 | 前缀树算法，**支持所有路由模式** | 复杂通配符、多层嵌套、混合路由 | 全兼容、不踩坑** |
| **SmartRouter** | ⚡️⚡️⚡️ 自动最优 | 启动时自动选最快路由 | 绝大多数项目、不想折腾 | **默认首选，开箱最优** |
| **LinearRouter** | ⚡️ 一般 | 路由注册极快，不编译字符串 | 每次请求都初始化（如Serverless冷启频繁） | 超快注册、一次性场景** |
| **PatternRouter** | ⚡️ 一般 | 体积最小，gzip≈5KB | 资源极有限环境（小程序/轻量边缘函数） | 最小打包体积** |

---

## 二、怎么设置（直接复制代码）
### 1. 默认用法（最稳）
Hono 内置 `SmartRouter + RegExpRouter + TrieRouter`，**不用写任何配置**：
```ts
import { Hono } from 'hono'
const app = new Hono() // ✅ 自动最优
```

### 2. 手动指定单一路由
```ts
import { Hono } from 'hono'
import { RegExpRouter } from 'hono/router/reg-exp-router'
import { TrieRouter } from 'hono/router/trie-router'
import { LinearRouter } from 'hono/router/linear-router'
import { PatternRouter } from 'hono/router/pattern-router'

// 要最快
const app = new Hono({ router: new RegExpRouter() })

// 要全兼容
const app = new Hono({ router: new TrieRouter() })

// 要注册快（冷启/每次初始化）
const app = new Hono({ router: new LinearRouter() })

// 要体积最小
const app = new Hono({ router: new PatternRouter() })
```

### 3. 自定义组合路由（进阶）
```ts
import { Hono } from 'hono'
import { SmartRouter } from 'hono/router/smart-router'
import { RegExpRouter } from 'hono/router/reg-exp-router'
import { TrieRouter } from 'hono/router/trie-router'

const app = new Hono({
  router: new SmartRouter({
    routers: [new RegExpRouter(), new TrieRouter()]
  })
})
```

---
