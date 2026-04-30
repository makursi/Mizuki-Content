---
title: React开发环境搭建
published: 2026-04-30
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](./react/reactcute.webp)

# 创建React应用

## 1. 使用Next.js (App Router) 
Next.js 的 App Router 是一个 React 框架，充分利用了 React 的架构，支持全栈 React 应用。

    npx create-next-app@latest

## 2. Expo(for native apps)

Expo 是一个 React 框架，让你可以创建支持真正原生 UI 的通用 Android、iOS 和 Web 应用。它为 React Native 提供了一个 SDK，让原生部分更易于使用。要创建一个新的 Expo 项目，运行：

    npx create-expo-app@latest

## 3. vite
Vite 是一款速度极快的前端构建工具,为下一代 Web 应用程序提供支持

    npm create vite@latest my-app -- --template react-ts

Vite 采用约定式设计，开箱即提供合理的默认配置。它拥有丰富的插件生态系统，能够支持快速热更新、JSX、Babel/SWC 等常见功能。

## 4.Rsbuild
Rsbuild 是一个基于 Rspack 的构建工具

    npx create-rsbuild --template react

Rsbuild 内置了对 React 特性的支持，如快速刷新、JSX、TypeScript 和样式。


# 使用Vite构建的React应用目录

    public 公共目录
    src
    assets 静态资源
    App.css 根组件样式
    App.tsx 根组件
    index.css 全局css文件
    main.tsx 全局tsx文件
    vite-env.d.ts 声明文件
    .eslintrc.cjs eslint配置文件
    .gitignore git忽略文件
    index.html 入口文件index.html
    package.json 项目依赖模块文件
    tsconfig.json ts配置文件
    tsconfig.node.json vite-ts配置文件
    vite.config.ts vite配置文件

## 面试可能会问到的一些问题:

- **public公共目录和assets静态资源有什么区别?**

回答: public目录的资源编译之后会存放到根目录，而静态资源assets是会随着项目一起打包的，public则不会被编译。

- **为什么main.tsx的document.getElementById('root')!要加一个!**

回答:因为document.getElementById('root')返回可能为空，这时候就会报错。!是非空断言，告诉编辑器这个表达式不会为空。

# package.json命令

    "dev": "vite",//启动开发模式项目
    "build": "tsc && vite build", //打包构建生产包
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",//代码检查
    "preview": "vite preview" //预览模式
