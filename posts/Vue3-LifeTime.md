---
title: Vue3-组件生命周期
published: 2026-04-08
description: ''
image: ''
tags: []
category: 'Vue'
draft: false 
lang: ''
---

# 一、组件基础
## 1. 组件是什么？

**一个 .vue 文件就是一个组件，可复用、可嵌套。**

✅ 官方完全一致：
- 组件 = 独立的 UI 模块
- 可复用、可嵌套、有独立生命周期
- 整个应用就是一棵**组件树**

## 2. 组件怎么用（script setup）
文章示例：
```vue
<!-- 父组件 App.vue -->
<template>
  <HelloWorldVue></HelloWorldVue>
</template>

<script setup>
import HelloWorldVue from './components/HelloWorld.vue'
</script>
```

✅ 官方规范：
- `<script setup>` 中**import 后可直接用**，不用注册
- 组件名**不能和 HTML 内置标签重名**（div、p、button 等）

---

# 二、Vue3 生命周期
## 一句话官方定义
**生命周期 = 组件从 创建 → 挂载 → 更新 → 销毁 的全过程。**

---

# 三、文章最重要一句话（必背）
> **Vue3 组合式 API（script setup）没有 beforeCreate 和 created！**

## 为什么？
官方解释：
- `setup` 执行时机 **就在 beforeCreate 和 created 之间**
- 你在 `setup` 里写的代码**本身就等于 created**
- 所以这两个钩子**不需要了**

---

# 四、Vue3 组合式 API 生命周期全讲解
## 1. onBeforeMount
**挂载前**
- DOM 还没渲染到页面
- 拿不到 DOM 元素
- 即将渲染

## 2. onMounted
**挂载完成**
- **DOM 已经渲染完毕**
- 可以获取、操作 DOM
- 可以发请求、初始化定时器、绑定事件

> 官方：**请求放这最稳**

## 3. onBeforeUpdate
**更新前**
- 数据变了
- 视图还没重新渲染
- 虚拟DOM 还没更新

## 4. onUpdated
**更新完成**
- 数据变了
- **视图已经更新完毕**
- 可以获取最新 DOM

## 5. onBeforeUnmount
**卸载前**
- 组件即将销毁
- 实例还在
- 适合：清除定时器、取消事件监听、取消请求

## 6. onUnmounted
**卸载完成**
- 组件销毁
- 指令解绑、事件监听全部移除

---

# 五、官方对照表（文章里的表，完全正确）
| 选项式 API | 组合式 API（script setup） |
|----------|-------------------|
| beforeCreate | 不需要 |
| created | 不需要 |
| beforeMount | onBeforeMount |
| mounted | onMounted |
| beforeUpdate | onBeforeUpdate |
| updated | onUpdated |
| beforeUnmount | onBeforeUnmount |
| unmounted | onUnmounted |

---

# 六、其他生命周期（官方补充）
这些是进阶钩子，日常很少用：
- **onErrorCaptured**：捕获组件错误
- **onRenderTracked**：调试用，追踪依赖
- **onRenderTriggered**：调试用，看谁触发更新
- **onActivated**：keep-alive 组件激活
- **onDeactivated**：keep-alive 组件停用

---
