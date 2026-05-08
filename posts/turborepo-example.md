---
title: Turborepo的示例
published: 2026-04-01
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---

# 使用 `create-turbo` 命令

## 使用 `create-turbo` 命令，基于官方或社区维护的现成示例来快速启动一个 Turborepo 项目。

```bash
# Use an example listed below
pnpm dlx create-turbo@latest --example [example-name]
# Use a GitHub repository from the community
pnpm dlx create-turbo@latest --example [github-url]

# Use an example listed below
yarn dlx create-turbo@latest --example [example-name]
# Use a GitHub repository from the community
yarn dlx create-turbo@latest --example [github-url]

# Use an example listed below
npx create-turbo@latest --example [example-name]
# Use a GitHub repository from the community
npx create-turbo@latest --example [github-url]

# Use an example listed below
bunx create-turbo@latest --example [example-name]
# Use a GitHub repository from the community
bunx create-turbo@latest --example [github-url]
```

### 核心维护示例
| 名称 | 描述 |
| :--- | :--- |
| **[Basic](https://github.com/vercel/turbo/tree/main/examples/basic)** | 包含两个 Next.js 应用的基础 monorepo。 |
| **[Kitchen sink](https://github.com/vercel/turbo/tree/main/examples/kitchen-sink)** | 包含前后端多种框架的示例。 |
| **[Non-monorepo](https://github.com/vercel/turbo/tree/main/examples/non-monorepo)** | 展示了在单包中如何使用 Turborepo。 |
| **[OpenTelemetry](https://github.com/vercel/turbo/tree/main/examples/opentelemetry)** | 集成了 OpenTelemetry、Prometheus 和 Grafana 用于指标可视化。 |
| **[SvelteKit](https://github.com/vercel/turbo/tree/main/examples/sveltekit)** | 包含多个共享 UI 库的 SvelteKit 应用。 |
| **[TailwindCSS](https://github.com/vercel/turbo/tree/main/examples/tailwindcss)** | 包含多个共享 TailwindCSS UI 库的 Next.js 应用。 |

## 社区维护示例

以下是 Turborepo 官方文档中列出的**社区维护示例**完整列表。这些示例主要用于展示如何将 Turborepo 与各种流行的工具或框架集成使用。

### 社区维护示例列表

| 示例名称 | 描述说明 |
| :--- | :--- |
| **[Design System](https://github.com/vercel/turbo/tree/main/examples/design-system)** | 通过在多个应用间共享设计系统，统一网站的外观和感觉。 |
| **[Angular](https://github.com/vercel/turborepo/tree/main/examples/with-angular)** | 用于学习 Turborepo 基础知识的极简 Angular 示例。 |
| **[Yarn Berry](https://github.com/vercel/turborepo/tree/main/examples/with-berry)** | 使用 Yarn Berry (Yarn 3) 的 Monorepo 示例。 |
| **[Biome](https://github.com/vercel/turborepo/tree/main/examples/with-biome)** | 包含两个 Next.js 应用并集成了 Biome 配置的基础 Monorepo 示例。 |
| **[Changesets](https://github.com/vercel/turborepo/tree/main/examples/with-changesets)** | 配置为通过 Changesets 发布包的 Monorepo。 |
| **[Docker](https://github.com/vercel/turborepo/tree/main/examples/with-docker)** | 包含 Express API 和 Next.js 应用的 Monorepo，使用 Docker 部署并利用 `turbo prune`。 |
| **[Gatsby](https://github.com/vercel/turborepo/tree/main/examples/with-gatsby)** | 包含一个 Gatsby.js 和一个 Next.js 应用，两者共享一个 UI 库的 Monorepo。 |
| **[Nest.js](https://github.com/vercel/turborepo/tree/main/examples/with-nestjs)** | 使用 Nest.js 的 Monorepo 示例。 |
| **[Next.js + Elysia](https://github.com/vercel/turborepo/tree/main/examples/with-nextjs-elysia)** | 包含 Next.js 前端和 Elysia 后端的 Monorepo。 |
| **[npm workspaces](https://github.com/vercel/turborepo/tree/main/examples/with-npm)** | 使用 NPM workspaces 的 Monorepo 示例。 |
| **[Prisma](https://github.com/vercel/turborepo/tree/main/examples/with-prisma)** | 包含一个完整配置了 Prisma 的 Next.js 应用的 Monorepo。 |
| **[React Native](https://github.com/vercel/turborepo/tree/main/examples/with-react-native-web)** | 包含一个简单的 React Native 和 Next.js 应用，并共享一个 UI 库的 Monorepo。 |
| **[Rollup](https://github.com/vercel/turborepo/tree/main/examples/with-rollup)** | 包含一个 Next.js 应用，并共享一个使用 Rollup 打包的 UI 库的 Monorepo。 |
| **[Solid.js](https://github.com/vercel/turborepo/tree/main/examples/with-solid)** | 包含 SolidJS 应用的 Monorepo 示例。 |
| **[typeorm](https://github.com/vercel/turborepo/tree/main/examples/with-typeorm)** | 包含一个完整配置了 typeorm 的 Next.js 应用的 Monorepo。 |
| **[Ultracite](https://github.com/vercel/turborepo/tree/main/examples/with-ultracite)** | 包含两个 Next.js 应用并集成了 Ultracite 配置的基础 Monorepo 示例。 |
| **[Vite](https://github.com/vercel/turborepo/tree/main/examples/with-vite)** | 包含多个使用 Vite 打包的 Vanilla JS 应用，并共享一个 UI 库的 Monorepo。 |
| **[Vite + React](https://github.com/vercel/turborepo/tree/main/examples/with-vite-react)** | 使用 Vite 和 React 的 Monorepo 示例。 |
| **[Vitest](https://github.com/vercel/turborepo/tree/main/examples/with-vitest)** | 使用 Vitest 进行测试的 Monorepo 示例。 |
| **[Vue/Nuxt](https://github.com/vercel/turborepo/tree/main/examples/with-vue-nuxt)** | 包含 Vue 和 Nuxt 应用，并共享一个 UI 库的 Monorepo。 |
| **[Yarn](https://github.com/vercel/turborepo/tree/main/examples/with-yarn)** | 使用 Yarn workspaces 的 Monorepo 示例。 |
