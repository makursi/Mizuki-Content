---
title: Vue3-Watch
published: 2026-04-08
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---

官方定义：
**watch 用于监听响应式数据，数据变化时自动执行回调。**

---

# 一、 watch 基础结构
```js
watch(
  监听源,        // 要监听谁
  回调函数,      // 变了做什么 (newVal, oldVal)
  配置项         // { immediate: true, deep: true }
)
```

### 三个参数官方解释
1. **监听源**：可以是 ref、reactive、函数返回值、数组
2. **回调**：新值、旧值
3. **配置**
   - `immediate: true` → **初始化立刻执行一次**
   - `deep: true` → **深度监听对象内部变化**

---

# 二、4 大使用场景（全部对齐官方）

## 1. 监听单个 ref（含深层对象）
文章代码：
```js
let message = ref({ nav: { bar: { name: "" } } })

watch(message, (newVal, oldVal) => {
  console.log(newVal, oldVal)
}, {
  immediate: true,
  deep: true
})
```

### 官方解释
- ref 包裹**对象**时，默认**不深度监听**
- 必须加 **deep: true** 才能监听内部属性变化

---

## 2. 监听多个 ref（数组写法）
代码：
```js
let a = ref('')
let b = ref('')

watch([a, b], ([newA, newB], [oldA, oldB]) => {
  // 任意一个变了都会触发
})
```

### 官方标准用法
- 多个数据源用**数组**包裹
- 回调里接收的也是**数组**

---

## 3. 监听整个 reactive 对象
代码：
```js
let state = reactive({ nav: { bar: { name: "" } } })

watch(state, () => {
  // 自动深度监听！不需要 deep: true
})
```

### 官方重点
- **reactive 自带深度监听**
- **加不加 deep 都一样生效**
- 无法获取正确的 oldVal（因为是同一个对象）

---

## 4. 监听 reactive 的**单个属性**
文章代码：
```js
watch(
  () => state.name,
  (newVal, oldVal) => {}
)
```

### 官方推荐写法
- 监听 reactive 具体属性**必须用 getter 函数**
- 好处：精准监听、性能高、能拿到正确 oldVal

> 直接写 `state.name` 不行，因为传的是**字符串/值**，不是响应式引用。
