---
title: Vue3-slot插槽
published: 2026-04-14
description: ''
image: ''
tags: []
category: 'Vue'
draft: false 
lang: ''
---
# ：Vue3 插槽 Slot 全知识

官方文档：https://cn.vuejs.org/guide/components/slots.html

---

## 一、插槽是什么？
> 插槽（Slot）是 Vue 中**内容分发**的机制，允许父组件将任意模板内容（HTML、组件、文本）插入到子组件的指定位置。
> 子组件用 `<slot>` 作为**占位符**，父组件填充内容。

一句话：
**子挖坑 → 父填坑**

---

# 二、文章知识点 1：匿名插槽（默认插槽）
### 官方名称：**默认插槽 (Default Slot)**
### 子组件定义
```vue
<!-- 子组件 Dialog.vue -->
<template>
  <div class="dialog">
    <!-- 匿名插槽：没有 name，默认名为 default -->
    <slot></slot>
  </div>
</template>
```

### 父组件使用
```vue
<!-- 父组件 -->
<Dialog>
  <!-- 填充到默认插槽 -->
  <div>我是填充的内容</div>
</Dialog>
```

### 官方要点
- 没有 `name` 的 `<slot>` 自动命名为 **default**
- 一个组件**只能有一个匿名插槽**
- 父组件直接写在子组件标签内的内容，自动进入默认插槽

---

# 三、文章知识点 2：具名插槽
### 官方定义
当子组件需要**多个内容出口**时，给插槽命名，父组件按名字填充。

### 子组件（定义多个具名插槽）
```vue
<template>
  <div>
    <slot name="header"></slot> <!-- 头部插槽 -->
    <slot></slot>              <!-- 默认插槽 -->
    <slot name="footer"></slot> <!-- 底部插槽 -->
  </div>
</template>
```

### 父组件（按名称填充）
```vue
<Dialog>
  <template v-slot:header>
    <div>我是头部</div>
  </template>

  <template v-slot:default>
    <div>我是内容</div>
  </template>

  <template v-slot:footer>
    <div>我是底部</div>
  </template>
</Dialog>
```

### 官方规则
- `v-slot:xxx` 绑定到对应 `name="xxx"` 的插槽
- 所有未匹配的内容进入 **default** 插槽
- **`v-slot` 只能用在 `<template>` 上**

---

# 四、插槽简写
### 官方语法
`v-slot:` 可以简写为 **`#`**，是 Vue3 最推荐写法。

上面的例子简写后：
```vue
<Dialog>
  <template #header>
    <div>头部</div>
  </template>

  <template #default>
    <div>内容</div>
  </template>

  <template #footer>
    <div>底部</div>
  </template>
</Dialog>
```

### 官方要点
- `#` 等价于 `v-slot:`
- 必须写插槽名：`#header`、`#default`
- 简洁、通用、行业标准写法

---

# 五、作用域插槽
## 官方核心概念
普通插槽：**父组件只能用自己的数据**
作用域插槽：**子组件可以把数据传给父组件的插槽使用**

### 子组件：向插槽绑定数据
```vue
<template>
  <div>
    <div v-for="item in 10">
      <!-- 把 item 绑定给插槽，父组件可以接收 -->
      <slot :data="item"></slot>
    </div>
  </div>
</template>
```

### 父组件：接收插槽数据
```vue
<Dialog>
  <!-- 解构接收子组件传来的 data -->
  <template #default="{ data }">
    <div>{{ data }}</div>
  </template>
</Dialog>
```

## 官方原理
1. 子组件在 `<slot>` 上通过**v-bind 绑定属性**
2. 这些属性会被打包成一个**插槽 props 对象**
3. 父组件通过 `#default="props"` 接收
4. 支持**解构赋值**，最常用

## 作用域插槽用途
- 列表组件（子遍历，父定义每一项样式）
- 表格组件
- 树形组件
- 高度复用的业务组件

---

# 六、动态插槽
### 官方定义
插槽名可以是**变量**，通过动态指令绑定。

语法：
```vue
<template #[slotName]>
  内容
</template>
```

### 示例
```vue
<template>
  <Dialog>
    <template #[activeSlot]>
      <div>动态插入到 {{ activeSlot }} 插槽</div>
    </template>
  </Dialog>
</template>

<script setup>
import { ref } from 'vue'
// 插槽名是变量
const activeSlot = ref('header')
</script>
```

### 官方要点
- `#[变量]` 动态绑定插槽名
- 可用于：根据状态切换插槽内容、权限控制

---

# 七、插槽默认内容
文章没提，但非常常用：
```vue
<!-- 子组件 -->
<slot>
  我是默认内容，父组件没填时显示
</slot>
```

---
