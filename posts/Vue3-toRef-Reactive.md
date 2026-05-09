---
title: Vue3-toRef-Reactive
published: 2026-04-08
description: ''
image: ''
tags: []
category: 'Vue'
draft: false 
lang: ''
---



# 一、toRef（官方定义：为对象属性创建响应式引用）
## 官方一句话
`toRef` 可以为**响应式对象**的某个属性创建一个 **ref**，并且**和原属性双向绑定**。

## 结论
- 源是**普通对象** → 修改 ref 会改原数据，但**不更新视图**（没有依赖收集）
- 源是**reactive 响应式对象** → 修改 ref **会同步原数据 + 更新视图**

## 核心原理
```ts
class ObjectRefImpl {
  get value() { return this._object[this._key] }
  set value(newVal) { this._object[this._key] = newVal }
}
```
- 它**不自己存值**，只是**指向原对象的属性**
- 只有源是响应式时，才会走 track/trigger → 视图才更新

## 最标准用法（官方推荐）
```ts
const state = reactive({ name: '小明' })
const nameRef = toRef(state, 'name')

nameRef.value = '小红'
// state.name 同步变，视图也更新
```

## 作用
**把一个属性变成 ref，传递出去不丢失响应式连接。**



# 二、toRefs（官方：解构 reactive 必备）
## 官方一句话
把 reactive 对象**所有属性批量转成 ref**，返回一个**普通对象**，每个属性都是 ref。

## 结论
- 解决：**直接解构 reactive 会丢失响应式**
- 解构后依然和原对象**双向绑定**
- 方便在 `<script setup>` 里直接用变量

## 为什么要用？（官方重点）
```ts
const state = reactive({ name: 'a', age: 18 })

// ❌ 直接解构 → 丢失响应式
let { name, age } = state

// ✅ toRefs 解构 → 保留响应式
let { name, age } = toRefs(state)
```

## 源码逻辑（文章讲得对）
```ts
function toRefs(obj) {
  const ret = {}
  for (const key in obj) {
    ret[key] = toRef(obj, key) // 循环调用 toRef
  }
  return ret
}
```
**批量 toRef**。



# 三、toRaw（官方：取响应式对象的原始对象）
## 官方一句话
`toRaw` 返回 `reactive`/`readonly` 代理对象的**原始对象**。

## 结论
- 把**代理对象 → 变回普通对象**
- 修改原始对象**不会触发视图更新**
- 用于：不想被监听、想操作源数据、调试、传给第三方库等

## 源码逻辑
```ts
function toRaw(observed) {
  // 从代理身上取原始对象 __v_raw
  const raw = observed[ReactiveFlags.RAW]
  return raw ? toRaw(raw) : observed
}
```
