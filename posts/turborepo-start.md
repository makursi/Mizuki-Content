---
title: Turborepo学习
published: 2026-04-01
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---

# Turborepo的定义

Turborepo 是一个用于 JavaScript 和 TypeScript 代码库的高性能构建系统。它设计用于扩展单存储库，同时也加快了单一包工作区的工作流程。

# Turborepo的优势

## Monorepo的核心问题。

Monorepo模式下两个核心问题: 
- 重复劳动
- 任务调度复杂

Monorepo模式下,每个工作区都有自己的测试套件、自己的 linting 和独特的构建流程。Monorepo中可能有成千上万个任务要执行。在大规模开发时,这些任务执行缓慢会极大影响团队构建软件的方式。

## Turborepo的解决方案

通过「**远程缓存**」彻底杜绝重复的构建、测试和代码检查工作，同时通过「**智能并行调度**」来最大化利用硬件资源，从而系统性地解决了大型 Monorepo 的执行效率问题。

- 远程缓存: 将任何任务（如构建）的结果存储在云端。当在另一台机器（如 CI 服务器）或新环境中执行相同任务时，直接下载缓存结果，而不是重新执行。
- 智能并行调度: 自动分析任务间的依赖关系（例如，build 必须先于 test 执行），并最大化利用多核 CPU 来并行执行那些没有相互依赖的任务。

# Turborepo的使用方法

## 安装

### 全局安装turbo工具

```bash
npm install turbo --global

yarn global add turbo

pnpm add turbo --global
```

使用如下命令创建Turborepo项目

```bash
npx create-turbo@latest

yarn dlx create-turbo@latest

pnpm dlx create-turbo@latest

bunx create-turbo@latest
```
启动项目包含:
- 两个可部署应用
- 三个共享库，用于 monorepo 的其他部分

### 项目内安装

将 turbo 作为 devDependency 固定在项目根目录，确保团队所有开发者及 CI 环境使用完全相同的版本。

```bash
npm install turbo --save-dev

yarn add turbo --dev --ignore-workspace-root-check

pnpm add turbo --save-dev --ignore-workspace-root-check

bun install turbo --dev
```

# 注意事项

如果之前全局安装过 `turbo`，务必使用和现有安装相同的包管理器，以避免出现意外行为。通过`turbo bin`命令,快速查看之前用过哪个包管理器搭配 。
