---
title: JSON-LD(JSON for Linked Data)
published: 2026-05-28
description: ''
image: ''
tags: []
category: 'SEO'
draft: false 
lang: ''
---

# JSON-LD

JSON-LD（JSON for Linked Data）是一种用于表达结构化数据的 JSON 格式。
它能帮助搜索引擎和 AI 更准确理解页面内容（例如商品、文章、组织、人物、活动等实体），从而提升页面在检索系统中的可理解性。

在 Next.js（App Router）里，推荐在 `layout.tsx` 或 `page.tsx` 中，直接输出一个原生 <script type="application/ld+json"> 标签来注入 JSON-LD。

## JSON-LD的基础结构

一个最简的 JSON‑LD 块长这样：

    <script type="application/ld+json">
    {
    "@context": "https://schema.org",
    "@type": "Person",
    "name": "张三",
    "url": "https://example.com/zhangsan",
    "birthDate": "1990-01-01"
    }
    </script>

它的核心结构就是 一个 JSON 对象，包含三个层次的字段：

1.  全局上下文（`@context`）

2.  类型声明（`@type`）

3.  该类型的其他属性（如 `name`、`url`、`birthDate` 等）

---

## 三、每个字段的详细说明

| 字段 | 是否必须 | 作用 | 示例值 |
| --- | --- | --- | --- |
| **`@context`** | **是** | 指定数据所用的“词汇表”的网址。绝大多数情况下固定为 `"https://schema.org"`，即使用 Schema.org 定义的术语。 | `"https://schema.org"` |
| **`@type`** | **是** | 声明当前描述的是“什么类型”的东西。类型也来自 Schema.org，比如 `Person`、`Article`、`Product`、`Organization` 等。 | `"Person"` |
| **`@id`** | 推荐 | 给这个实体一个**全局唯一的标识符**（通常是 URI），方便其他数据引用它。 | `"https://example.com/people/123"` |
| 其他属性 | 视类型而定 | 根据 `@type` 的不同，可以填写对应的属性，例如 `name`、`description`、`image`、`author`、`price` 等。这些属性名也来自 Schema.org 词汇表。 | `"name": "张三"` |

### 补充几个常见但容易混淆的点：

-   **`@context` 可以嵌套或使用数组**，但 99% 的场景只用 `"https://schema.org"`。

-   **属性和值可以是字符串、数字、布尔值、另一个 JSON‑LD 对象（表示嵌套实体）或数组**。

-   **自定义属性**：除非你使用 `@vocab` 或 `@context` 扩展，否则不建议随意造属性名，因为搜索引擎不认识。

---

---

## 四、一个稍微完整的例子（产品）

<script type="application/ld+json"\>
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "无线机械键盘",
  "image": "https://example.com/keyboard.jpg",
  "description": "一款支持三模连接的紧凑型机械键盘",
  "sku": "KBD-123",
  "brand": {
    "@type": "Brand",
    "name": "KeyMaster"
  },
  "offers": {
    "@type": "Offer",
    "price": "399.00",
    "priceCurrency": "CNY",
    "availability": "https://schema.org/InStock"
  }
}
</script>

字段解释（除了上面已有的）：

-   `sku` – 商品库存单位

-   `brand` – 嵌套一个 `Brand` 类型的对象

-   `offers` – 嵌套一个 `Offer` 类型的对象，包含价格、币种、库存状态

---

### 1. 必须放在 `<script type="application/ld+json">` 里  
- 不要直接写在 HTML 属性里，也不要自己改 `type` 值。

### 2. `@context` 和 `@type` 不能少  
- `@context: "https://schema.org"` 固定写法。  
- `@type` 必须是 Schema.org 里已定义的类型（如 `Product`、`Article`），不要自己造。

### 3. 属性值要匹配 Schema 定义的类型  
- 比如 `price` 应该是数字或字符串，`availability` 要用官方指定的 URL（如 `https://schema.org/InStock`），不要随意写 “有货”。  
- 日期格式一般用 ISO 8601：`2025-01-01` 或 `2025-01-01T10:00:00+08:00`。

### 4. 内容必须与页面上**真实可见**的内容一致  
- 不要虚构价格、评分或不存在的信息，这违反 Google 指南，可能被判定为欺骗并导致降权或手动处罚。

### 5. 一个页面可以有多个 JSON‑LD 块，但不推荐重复描述同一实体  
- 你可以同时描述 `BreadcrumbList`、`Article` 和 `Person`，没问题。  
- 但不要用两个块分别描述同一篇文章（除非用 `@id` 明确指向同一事物）。

### 6. 注意嵌套与空值  
- 嵌套对象也要带 `@type`。  
- 不要出现 `null` 或空字符串的属性，Google 可能忽略或报错；没有值就不写该字段。

### 7. 别忘了用 Rich Results Test 或 Schema.org 验证工具测试  
- 手写很容易出错（比如末尾多一个逗号）。  
- 测试通过 ≠ 一定能出富结果，但语法错误一定不能出。

### 8. 不要放在被动态加载后又删除的 DOM 中  
- 如果用 JavaScript 动态插入 JSON‑LD，确保它最终留在 DOM 里，且渲染后依然存在。  
- 最稳妥：直接在服务器端输出的 HTML 的 `<head>` 或 `<body>` 末尾静态生成。

### 9. 移动端与桌面端应保持一致  
- 如果响应式站点用不同 HTML 结构，两端的 JSON‑LD 内容应相同或等价，不要只在桌面版放结构化数据。

---
