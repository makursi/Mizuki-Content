---
title: Vue-全局组件,局部组件,递归组件
published: 2026-04-14
description: ''
image: ''
tags: []
category: 'Vue'
draft: false 
lang: ''
---

# 组件概述
**Vue3 + `<script setup>` + TypeScript** 下的三种组件注册/使用方式：
1. **全局组件**：一次注册，全项目可用
2. **局部组件**：当前组件内可用，最常用、最推荐
3. **递归组件**：组件自己调用自己，用于树、菜单、级联选择器

官方文档对应章节：
- 组件注册：https://cn.vuejs.org/guide/components/registration.html
- 组件 name 选项：https://cn.vuejs.org/api/options-misc.html#name

---

# 二、第一部分：全局组件（Global Component）
## 1. 官方定义
> 全局注册的组件在**整个应用的任何组件模板中都能直接使用**，无需再次导入。
> 使用 `app.component(名称, 组件)` 注册。

## 2. 示例 + 官方规范讲解
### ① 封装一个 Card 组件
```vue
<!-- Card.vue -->
<template>
  <div class="card">
    <div class="card-header">
      <div>标题</div>
      <div>副标题</div>
    </div>
    <div v-if="content" class="card-content">
      {{ content }}
    </div>
  </div>
</template>

<script setup lang="ts">
type Props = {
  content: string
}
defineProps<Props>()
</script>

<style scoped lang="less">
/* 样式省略 */
</style>
```

### ② 在 main.ts 全局注册（官方标准写法）
```ts
// main.ts
import { createApp } from 'vue'
import App from './App.vue'
// 导入组件
import Card from './components/Card/index.vue'

// 链式调用：必须在 mount 之前！
const app = createApp(App)
app.component('Card', Card) // 全局注册
app.mount('#app')
```

### ③ 任意页面直接使用，无需 import
```vue
<template>
  <Card content="我是全局卡片内容" />
</template>
```

---

## 3. 官方核心要点
1. **注册顺序绝对不能错**
   - `createApp → component → mount`
   - 不能写在 `mount` 之后
2. **全局组件优点**
   - 高频组件（Button、Input、Icon、Card）不用反复 import
3. **全局组件缺点（官方不推荐滥用）**
   - 不利于 **Tree-shaking**（没用到也会被打包）
   - 组件依赖不清晰，大型项目难维护
   - **官方建议：优先局部注册**

## 4. 批量注册全局组件
文章里注册 Element-Plus 图标：
```ts
import * as ElementPlusIconsVue from '@element-plus/icons-vue'
const app = createApp(App)
// 遍历批量注册
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component)
}
```
官方认可：批量注册高频通用组件很常见。

---

# 三、第二部分：局部组件（Local Component）
## 1. 官方定义
> 局部注册的组件**仅在当前父组件内可用**，子组件的子组件不能直接用。
> `<script setup>` 中：**import 后可直接在模板使用**，无需注册。

## 2. 示例 + 官方规范
```vue
<template>
  <div class="wraps">
    <!-- 直接使用导入的组件 -->
    <layout-menu />
    <layout-header />
    <layout-main />
  </div>
</template>

<script setup lang="ts">
// 1. 导入
import layoutHeader from './Header.vue'
import layoutMenu from './Menu.vue'
import layoutMain from './Content.vue'
</script>
```

## 3. 官方核心规则
1. **`<script setup>` 天然支持局部注册**
   - import → 模板直接用，**不用写 components 选项**
2. **作用域严格隔离**
   - A 组件导入 B → 只有 A 能用 B
   - C 想用 B → C 必须自己 import
3. **优点（官方最推荐）**
   - 依赖清晰
   - 天然支持 Tree-shaking
   - 代码可维护性最高

---

# 四、第三部分：递归组件（Recursive Component）
## 1. 官方定义
> 递归组件 = **组件在自己的模板中调用自己**
> 必须满足两个条件：
> 1. 组件有 **name**（或由文件名自动推导）
> 2. 有 **终止条件**（否则无限递归、栈溢出）

官方文档明确：
> `name` 选项的用途之一：**允许组件在自己的模板中递归引用自己**。

---

## 2. 递归组件三要素
1. **数据结构**：必须是**嵌套型（children）**
2. **组件名称**：必须有 name（用于自调用）
3. **终止条件**：`v-if="item?.children?.length"`

---

## 3. 完整案例
### ① 父组件：定义树形数据
```ts
type TreeList = {
  name: string
  children?: TreeList[]
}

const data = reactive<TreeList[]>([
  {
    name: 'no.1',
    children: [
      { name: 'no.1-1', children: [{ name: 'no.1-1-1' }] }
    ]
  },
  { name: 'no.2', children: [{ name: 'no.2-1' }] },
  { name: 'no.3' }
])
```

```vue
<template>
  <Tree :data="data" />
</template>
```

---

### ② 子组件（递归核心）
#### 第一步：给组件命名（三种官方方式）

##### 方式 1：双 script 标签（兼容所有版本，最稳）
```vue
<!-- 这个 script 只用来定义 name，无业务逻辑 -->
<script lang="ts">
export default {
  name: 'TreeItem' // 递归时用的名字
}
</script>

<!-- 业务逻辑写在 setup 里 -->
<script setup lang="ts">
interface Tree {
  name: string
  children?: Tree[]
}
defineProps<{ data?: Tree[] }>()
</script>
```

##### 方式 2：Vue 3.3+ 官方原生 `defineOptions`（推荐）
```vue
<script setup lang="ts">
defineOptions({ name: 'TreeItem' }) // 官方原生
</script>
```

##### 方式 3：插件 unplugin-vue-macros（旧兼容方案）
现在 Vue 3.3+ 已不需要。

---

#### 第二步：模板中自己调用自己（递归）
```vue
<template>
  <div class="tree" style="margin-left:10px;">
    <div v-for="(item, index) in data" :key="index">
      <div @click="clickItem(item)">{{ item.name }}</div>

      <!-- 递归：调用自己 TreeItem + 终止条件 -->
      <TreeItem
        v-if="item?.children?.length"
        :data="item.children"
      />
    </div>
  </div>
</template>
```

---

## 4. 递归组件
1. **必须有 name**
   - 用于模板内自调用
   - 不写 name → 报错：组件未找到
2. **必须有终止条件**
   ```vue
   v-if="item?.children?.length"
   ```
   没有子节点时停止渲染
3. **数据必须是递归类型**
   ```ts
   type Tree = {
     name: string
     children?: Tree[]
   }
   ```
4. **每次递归传入 children**
   ```vue
   <TreeItem :data="item.children" />
   ```

---
