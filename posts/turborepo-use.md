---
title: Turborepo使用
published: 2026-04-01
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---

# 多包工作区使用

官方文档中，关于 **在多包工作区（monorepo）中添加 Turborepo** 的部分，核心步骤如下：

1. **前提准备**  
   Turborepo 基于包管理器的 Workspaces 功能，因此你需要确保仓库已配置好 workspaces（如 `pnpm-workspace.yaml` 或 `package.json` 中的 `workspaces` 字段）。

2. **安装 turbo**  
   推荐在**仓库根目录**安装 `turbo` 作为开发依赖，同时全局安装以获得更好体验：
   ```bash
   # 以 pnpm 为例
   pnpm add turbo --save-dev --workspace-root
   pnpm add turbo --global
   ```

3. **配置 turbo.json**  
   在根目录创建 `turbo.json`，定义任务（如 `build`、`check-types`）。关键配置包括：
```json
// turbo.json
{
  "$schema": "https://turborepo.dev/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**"]
    },
    "check-types": {
      "dependsOn": ["^check-types"]
    },
    "dev": {
      "persistent": true,
      "cache": false
    }
  }
}
```
   - `dependsOn: ["^build"]`：表示任务的依赖（`^build` 表示依赖上游包的 `build` 任务）。
   - `outputs`：指定缓存输出目录。
   - 对于开发任务设置 `persistent: true` 和 `cache: false`。

4. **调整包脚本**  
   确保各包的 `package.json` 中包含与 `turbo.json` 中任务对应的脚本（如 `check-types`）。建议将组合命令（如 `tsc && vite build`）拆分为独立的 `check-types` 和 `build` 脚本。

```json
// apps/web/package.json 
{
  "scripts": {
    "check-types": "tsc --noEmit",
    "build": "vite build"
  }
}
```


5. **忽略 turbo 缓存目录**  
   在 `.gitignore` 中添加 `.turbo`，避免缓存文件被提交。

6. **声明包管理器**  
   在根 `package.json` 中添加 `packageManager` 字段（如 `"pnpm@10.0.0"`），帮助 Turborepo 优化行为。

7. **配置包管理器 workspaces**  
   在 `pnpm-workspace.yaml` 或 `package.json` 中声明子包目录模式（如 `"apps/*"`、`"packages/*"`），让包管理器识别所有包。

8. **运行任务**  
   使用 `turbo build check-types` 等命令运行任务。Turborepo 会根据依赖图并行执行，并利用缓存加速（第二次运行可能仅需 185ms）。

终端输出结果:
```bash
Tasks:    2 successful, 2 total
Cached:    2 cached, 2 total
Time:    185ms >>> FULL TURBO
```

9. **启动开发环境**  
   运行 `turbo dev` 可同时启动所有包的开发任务，或使用 `--filter` 参数聚焦特定包。


# 在单一包工作区

单一包工作区,指的是运行 `npx create-next-app` 或 `npm create vite` 后得到的。

## 适用性与功能
虽然 Turborepo 在多包工作区 （通常称为 monorepo）中非常有效，
但Turborepo的本地和远程缓存以及任务并行化能够很好的适用于单包工作区.

## 安装

```bash
pnpm add turbo --save-dev

yarn add turbo --dev

npm install turbo --save-dev

bun install turbo --dev
```

## 使用一个命令运行多个脚本

### 关键作用

将多个有关联关系的、原本需要逐个手动运行的脚本（例如启动数据库、推送数据、运行开发服务器），通过 Turborepo 编排成一条命令，并自动处理它们之间的顺序依赖。

### 核心原理

**通过 dependsOn 定义任务依赖树**
Turborepo 并不是简单地同时运行所有脚本，而是通过 turbo.json 中的 dependsOn 字段，构建一个有向无环图来决定执行顺序。

例如官方文档中的场景：运行开发环境前，必须按顺序完成 db:up → db:push → db:seed → dev。

1. 为数据库创建一个 Docker 容器。

2. 把数据库模式推送到数据库。

3. 用数据填充数据库。

4. 启动开发服务器。

### 配置步骤

1. 在package.json中定义基础脚本

```json
// package.json
{
  "scripts": {
    "dev": "next dev",                    // 最终要运行的开发服务器
    "db:up": "docker-compose up -d",     // 启动数据库容器
    "db:push": "your-orm-tool schema-push", // 推送数据库结构
    "db:seed": "node ./db-seed.js"       // 填充初始数据
  }
}
```

2. 在 turbo.json 中通过 dependsOn 串联任务

```json
// turbo.json
{
  "$schema": "https://turborepo.dev/schema.json",
  "tasks": {
    "dev": {
      "dependsOn": ["db:seed"],
      "cache": false,
      "persistent": true
    },
    "db:seed": {
      "dependsOn": ["db:push"],
      "cache": false
    },
    "db:push": {
      "dependsOn": ["db:up"],
      "cache": false
    },
    "db:up": {
      "cache": false
    }
  }
}
```

dependsOn 数组为任务创建顺序。运行 turbo dev 时，先运行 db：up、db：push、db：seed 的脚本。

## 任务并行化

### 关键作用

**将串行等待变为同时执行**

在传统项目中，当你需要运行多个检查任务时（如 lint、类型检查、格式化），通常会这样做:

```bash
# 串行执行方式（慢）
npm run lint && npm run check-types && npm run format
# 总耗时 = lint时间 + check-types时间 + format时间
```

而使用 Turborepo 的任务并行化，这些没有依赖关系的任务会同时执行：

```bash
turbo lint check-types format
# 总耗时 ≈ max(lint时间, check-types时间, format时间)
```

提速效果：如果每个任务需要 10 秒，串行需要 30 秒，并行只需要约 10 秒。

### 基础配置

使用`turbo`并行化任务，可以通过同时运行所有任务来加快速度。例如，你可以同时运行 ESLint、TypeScript 和 Prettier 检查。给定像这样的脚本：
```json
{
  "scripts": {
    "lint": "eslint .",
    "format": "prettier .",
    "check-types": "tsc --noEmit"
  }
}
```

创建类似这样的配置:

```json
// turbo.json
{
  "$schema": "https://turborepo.dev/schema.json",
  "tasks": {
    "lint": {},
    "format": {},
    "check-types": {}
  }
}
```

然后，要同时运行所有任务: 

```bash
turbo run lint check-types format
```
