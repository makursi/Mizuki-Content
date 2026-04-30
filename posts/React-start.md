---
title: React学习之旅
published: 2026-04-30
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](./react/reactcute.webp)

# React定义

官方定义为:**The library for web and native user interfaces--一个为创建网页和原生用户界面的库**

# React的特点

1.  **组件化：**React 通过将 UI 分解为独立的、可重用的组件，使得代码更易于管理和维护。每个组件只关注于自身的逻辑和视图。

2.  **声明式编程：**React 采用声明式的编程风格，开发者只需描述 UI 应该是什么样子的，而不需要手动操作 DOM。React 会根据数据的变化自动更新 UI。

3.  **虚拟 DOM：**React 使用虚拟 DOM（Virtual DOM）来优化 UI 的更新过程。当数据发生变化时，React 会创建一个新的虚拟 DOM，然后将其与之前的虚拟 DOM 进行比较，找出最小的变化，并将这些变化应用到实际的 DOM 中，从而提高性能。

4.  **单向数据流：** React 采用单向数据流（也称为单向数据绑定），这意味着数据在组件之间通过 props 进行传递，使得数据的流动更加清晰和可预测。

# React与Vue的不同点

１.  定位：**React 是UI 库，生态靠社区组合**,Vue 是渐进式框架，官方提供完整工具链（Router/Pinia/Vite）；

２.  **React 用JSX,TSX，逻辑与视图混写。** Vue 用HTML 模板 + 指令（v-if/v-for），分离 concerns；

3.  响应式原理:**React手动 setState/Hooks，需依赖数组。**Vue的响应式原理为Proxy+依赖收集+触发式更新.

4.  数据流方面:React 严格单向数据流。Vue 支持双向绑定（v-model）

# React的核心突出点

1.  JSX,TSX的极致灵活不必多说,将试图和逻辑统一在JS

2.  函数式编程范式的典范,纯函数、不可变数据、单向流，可预测、易测试、可维护。

3.  并发渲染与调度:Fiber架构支持**时间切片、优先级调度**，大型应用更流畅。

React官方文档:https://zh-hans.react.dev/
