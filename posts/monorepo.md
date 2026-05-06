---
title: Monorepo学习
published: 2026-05-06
description: ''
image: ''
tags: []
category: 'Monorepo'
draft: false 
lang: ''
---

好的，我会严格按照你给的“问题回答模板”来讲清楚 **Monorepo**，然后再给你一个**最小可运行案例（pnpm + workspace）**，尽量做到既系统又能落地。

---

# 官方定义

Monorepo（Monolithic Repository）是一种代码管理策略：
将多个项目（packages / apps / libs）存放在**同一个代码仓库**中进行统一管理。

典型工具官方定义（来自 pnpm / workspace 概念）：

> workspace 允许你在一个仓库中管理多个 package，并通过本地依赖链接它们。

简单理解就是：
一个 repo = 多个项目（前端、后端、组件库等）

---

# 核心作用

### 1. 代码复用能力提升

多个项目可以直接引用：

```bash
packages/ui
packages/utils
apps/web
```

web 可以直接依赖 ui 和 utils（不用发 npm）

---

### 2. 依赖统一管理

所有项目共享：

* node_modules（pnpm 会优化）
* 版本统一（避免版本地狱）

---

### 3. 原子提交（Atomic Commit）

一次提交可以同时修改：

* 后端 API
* 前端调用
* 类型定义

不会出现“版本不匹配”的问题

---

### 4. 提高开发体验

配合工具：

* pnpm workspace
* turborepo / nx

可以实现：

* 增量构建
* 缓存
* 并行执行

---

### 5. 适合中大型项目

比如：

* 微前端
* 组件库 + 业务项目
* AI 应用(AI-SaaS平台)

---

# 使用方法

## 目录结构（推荐）

```bash
my-monorepo/
├── apps/
│   └── web/
├── packages/
│   ├── ui/
│   └── utils/
├── package.json
├── pnpm-workspace.yaml
```

---

## 核心配置

### 1. 初始化项目

```bash
mkdir my-monorepo
cd my-monorepo
pnpm init
```

---

### 2. 创建 workspace 配置

`pnpm-workspace.yaml`

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

---

### 3. 创建子包

```bash
mkdir -p apps/web
mkdir -p packages/utils
```

---

### 4. 初始化子项目

```bash
cd apps/web && pnpm init
cd ../../packages/utils && pnpm init
```

---

### 5. 建立依赖关系（重点）

在 `apps/web/package.json`：

```json
{
  "name": "web",
  "dependencies": {
    "utils": "workspace:*"
  }
}
```

`workspace:*` 是关键！

---

### 6. 写一点代码

`packages/utils/index.js`

```js
export function add(a, b) {
  return a + b;
}
```

`apps/web/index.js`

```js
import { add } from "utils";

console.log(add(1, 2));
```

---

### 7. 安装依赖

在根目录中:

```bash
pnpm install
```

pnpm 会：
- 自动链接 utils
- 不会走 npm registry

---

### 8. 运行

```bash
node apps/web/index.js
```

输出：

```
3
```

---

# 注意事项

### 1. workspace:* 不是随便写的

只能用于：

* monorepo 内部依赖

不能发布到 npm（需要替换版本号）

---

### 2. node_modules 结构不同

pnpm 使用：
硬链接 + content-addressable storage

所以：

* node_modules 看起来“奇怪”是正常的
* 不要手动改

---

### 3. 依赖提升（hoisting）

pnpm 默认：
不完全扁平

可能出现：

```bash
找不到某些包
```

解决：

```bash
pnpm add lodash -w
```

---

### 4. 子包必须有 name

```json
{
  "name": "utils"
}
```

否则 workspace 无法识别

---

### 5. 适合但不简单

Monorepo 很强，但也带来复杂度：

* 构建系统复杂:所有项目一个仓库,构建起来麻烦,存在性能问题
* CI/CD 更复杂:所有项目共用代码库，依赖多、关联杂，CI不好精准判断哪些需要构建、部署，流水线配置和执行都会更麻烦。
* 权限管理困难:权限管理通常依赖代码托管平台(github,gitlab),粒度太粗.
例如: 程序员A->整个仓库,程序员B -> 整个仓库
如果有敏感业务模块,外包人员接受项目,什么都能看到.
