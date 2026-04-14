---
title: Vue-动态组件
published: 2026-04-14
description: ''
image: ''
tags: []
category: 'Vue'
draft: false 
lang: ''
---

# 补充知识--**可选链操作符**和**双问号表达式**
---

# 一,总结
1. **可选链 `?.`**
   **作用：安全读取深层属性，防止报错“Cannot read property 'xxx' of undefined”**
   遇到 `null/undefined` 就**停止**，返回 `undefined`，不报错。

2. **空值合并运算符 `??`**
   **作用：给“空值”设置默认值，只在值为 `null/undefined` 时生效**
   比 `||` 更精准、更安全。

---

# 二、可选链操作符 `?.` （Optional Chaining）
## 官方定义
> 允许读取位于连接对象链深处的属性的值，**而不必明确验证链中的每个引用是否有效**。
> 遇到 `null` 或 `undefined` 时，表达式会**短路返回 `undefined`**。

---

## 1. 为什么要用它？
你在组件里经常取嵌套数据：
```js
const user = {
  info: {
    address: {
      city: "北京"
    }
  }
}
```

如果某一层没值，传统写法会**直接报错**：
```js
// 报错！Cannot read properties of undefined
const city = user.info.address.city
```

**用可选链就不会报错：**
```js
const city = user?.info?.address?.city
```
规则：
- 前面是 `null/undefined` → **不往后执行，直接返回 undefined**
- 前面有值 → **继续往后取**

---

## 2. 四种常用写法（Vue 全能用）
```js
// 1. 对象深层属性（最常用）
obj?.a?.b?.c

// 2. 数组取值
arr?.[0]

// 3. 函数/方法安全调用
fn?.()

// 4. 结合动态key
obj?.[key]
```

---

## 3. 递归组件
```vue
<TreeItem v-if="item?.children?.length" />
```
等价于安全写法：
```js
if(item && item.children && item.children.length)
```
**用 ?. 一行搞定，不报错、更简洁。**

---

# 三、空值合并运算符 `??` （Nullish Coalescing）
## 官方定义
> 当左侧操作数为 **null 或 undefined** 时，返回右侧操作数；
> 否则返回左侧操作数。

**一句话：只对“真正空”的值生效。**

---

## 1. 为什么要用 `??` 而不是 `||`？
因为 **`||` 会把 0、''、false 都当成“空”**，会出bug！

例子：
```js
const num = 0

console.log(num || 100) // 输出 100（错误！0 被当成空）
console.log(num ?? 100) // 输出 0（正确！只判断 null/undefined）
```

`??` 只认两种空：
- **null**
- **undefined**

`0` `''` `false` 都会被当成**有效值**，这才是业务需要的安全逻辑。

---

## 2. Vue 中最常用场景
### ① Props 默认值
```ts
const props = withDefaults(defineProps<{
  count?: number
}>(), {
  count: 0
})
```
底层等价于：
```js
props.count ?? 0
```

### ② 安全赋值
```js
const name = data?.user?.name ?? "匿名用户"
```

---

# 四、最强组合：`?.` + `??` 一起用（Vue 实战万能公式）
这是你写项目一定会用到的**黄金组合**：

```js
const city = user?.info?.address?.city ?? "未知城市"
```

执行逻辑：
1. `?.` 安全读取，不报错
2. 读到最后是 `undefined`
3. `??` 兜底给默认值

**安全、简洁、无bug、官方推荐。**

---

# 六、代码解释
```vue
<TreeItem v-if="item?.children?.length" />
```

### 翻译：
1. **先看 item 存在吗？**
   不存在 → 返回 `undefined` → `v-if` 为 false → 不渲染
2. **item 存在 → 看 children 存在吗？**
   不存在 → 返回 `undefined` → 不渲染
3. **children 存在 → 取 length**
   length > 0 → 渲染，否则不渲染

### 作用：
**多层嵌套数据，不报错、安全判断是否有子节点。**

---


# 一、Vue3 动态组件
官方定义：
> **动态组件**：让多个组件使用**同一个挂载点**，并根据数据**动态切换渲染哪个组件**。
> 依靠 Vue 内置的 `<component>` 标签 + `is` 属性实现。

官方文档：
https://cn.vuejs.org/guide/essentials/component-basics.html#dynamic-components

---

# 二、基础用法
## 1. 基本写法
```vue
<template>
  <!-- 同一个挂载点，动态渲染不同组件 -->
  <component :is="currentComp"></component>
</template>

<script setup>
import A from './A.vue'
import B from './B.vue'
import { ref } from 'vue'

const currentComp = ref(A)
</script>
```

- `<component>`：Vue 内置标签，专门用于**动态渲染组件**
- `:is`：决定当前渲染**哪个组件**
- 切换 `currentComp` 的值，就会自动切换组件

## 2. 典型场景
- Tab 切换
- 步骤表单
- 低代码平台、动态仪表盘
- 权限控制下的动态视图

---

# 三、Vue2 与 Vue3 动态组件的关键区别
## 文章原话：
> Vue2 的 `is` 是通过**组件名称**切换
> Vue3 setup 下是通过**组件实例/对象**切换

## 官方规范对比
### 1）Vue2
```vue
<!-- Vue2 -->
<component :is="compName"></component>
```
```js
export default {
  components: { A, B },
  data() {
    return { compName: 'A' }
  }
}
```
- `is` 绑定**字符串名称**
- 组件必须先在 `components` 中注册

### 2）Vue3 `<script setup>`
```vue
<!-- Vue3 -->
<component :is="A"></component>
```
```ts
import A from './A.vue'
```
- `is` 直接绑定**组件对象/实例**
- 无需注册，import 后直接用
- 更类型安全，更符合组合式 API 风格

---

# 四、重点警告：不要把组件放进 reactive / ref
## 警告：
```
Vue received a Component which was made a reactive object.
This can lead to unnecessary performance overhead.
```

## 官方原因
- 组件本身是**静态配置对象**，**永远不需要响应式**
- `reactive` / `ref` 会对组件做**深层 Proxy 代理**
- 完全浪费性能，还可能导致异常

所以：
**组件对象绝对不应该被响应式代理！**

---

# 五、官方推荐的两种优化方案

## 1. markRaw
```ts
import { markRaw, reactive } from 'vue'
import A from './A.vue'
import B from './B.vue'

const tabList = reactive([
  { name: 'A组件', com: markRaw(A) },
  { name: 'B组件', com: markRaw(B) }
])
```

### 官方解释 markRaw：
> 将一个对象**标记为永远不会转为响应式**，返回对象本身。
> 适用于：第三方库实例、组件对象、大型不可变数据。

作用：
- 组件被 markRaw → **不会被 Proxy**
- 消除警告，提升性能

---

## 2. shallowRef
```ts
import { shallowRef } from 'vue'
import A from './A.vue'

const currentComp = shallowRef(A)
```

### 官方解释 shallowRef：
> 浅层 ref，**只追踪 .value 的引用变化**，不对内部对象做深层代理。
> 非常适合存放：组件、DOM、大型数据。

作用：
- 修改 `.value` 会触发更新
- 内部对象**不被代理**
- 比普通 ref 性能高很多

---

# 六、markRaw vs shallowRef 怎么选？（官方建议）
| 场景 | 推荐 API |
|------|----------|
| 组件放在数组/对象中 | **markRaw** |
| 单独存储当前激活组件 | **shallowRef** |
| 第三方库实例、不可变数据 | markRaw |
| 只需切换引用，不需内部响应 | shallowRef |

一句话：
**存集合用 markRaw，存单个引用用 shallowRef。**

---

# 七、标准示例
```vue
<template>
  <button @click="currentComp = A">A</button>
  <button @click="currentComp = B">B</button>

  <component :is="currentComp"></component>
</template>

<script setup>
import { shallowRef, markRaw, reactive } from 'vue'
import A from './A.vue'
import B from './B.vue'

// 单个切换：shallowRef 最佳
const currentComp = shallowRef(A)

// 列表存放：markRaw 最佳
const tabList = reactive([
  { label: 'A', comp: markRaw(A) },
  { label: 'B', comp: markRaw(B) },
])
</script>
```

---

# 八、搭配 KeepAlive 缓存状态
动态组件默认**切换会销毁重建**，用 `<KeepAlive>` 缓存：
```vue
<KeepAlive>
  <component :is="currentComp"></component>
</KeepAlive>
```
- 保留表单输入
- 保留滚动位置
- 避免重复请求
官方文档：https://cn.vuejs.org/guide/built-ins/keep-alive.html

---

# 完整实例
**完整可直接运行、Vue3 + script setup + 官方规范** 
包含：
1. 动态组件 `<component :is>`
2. Tab 切换
3. 组件存对象不被响应式代理（`markRaw` / `shallowRef`）
4. `KeepAlive` 缓存组件状态
5. 动态组件传参、事件
6. 完全规避 Vue 警告：`component was made a reactive object`

---

## 目录结构
```
src/
├── components/
│   ├── TabA.vue
│   ├── TabB.vue
└── App.vue
```

---

## 1. 子组件 TabA.vue
```vue
<template>
  <div class="box">
    <h3>组件 A</h3>
    <p>父组件传值：{{ msg }}</p>
    <input v-model="inputVal" placeholder="缓存测试" />
    <button @click="emitData">向父传值</button>
  </div>
</template>

<script setup lang="ts">
const props = defineProps<{
  msg: string
}>()

const emit = defineEmits<{
  (e: 'send', val: string): void
}>()

const inputVal = $ref('')

const emitData = () => {
  emit('send', '我是A组件的数据')
}
</script>

<style scoped>
.box { padding: 20px; border: 1px solid #ccc; }
</style>
```

---

## 2. 子组件 TabB.vue
```vue
<template>
  <div class="box">
    <h3>组件 B</h3>
    <p>父组件传值：{{ msg }}</p>
    <textarea v-model="text" placeholder="切换不丢失内容"></textarea>
  </div>
</template>

<script setup lang="ts">
const props = defineProps<{
  msg: string
}>()

const text = $ref('')
</script>

<style scoped>
.box { padding: 20px; border: 1px solid #ccc; margin-top:10px }
</style>
```

---

## 3. 父组件 App.vue
```vue
<template>
  <!-- Tab切换按钮 -->
  <button 
    v-for="item in tabList" 
    :key="item.name"
    @click="currentComp = item.comp"
    :class="{ active: currentComp === item.comp }"
  >
    {{ item.name }}
  </button>

  <!-- 动态组件 + 缓存 + 传参 + 监听事件 -->
  <KeepAlive>
    <component
      :is="currentComp"
      msg="父组件动态下发文本"
      @send="handleReceive"
    />
  </KeepAlive>
</template>

<script setup lang="ts">
import { reactive, markRaw, shallowRef } from 'vue'
import TabA from './components/TabA.vue'
import TabB from './components/TabB.vue'

// 场景1：组件数组列表 —— 用 markRaw 防止组件被 reactive 代理
const tabList = reactive([
  {
    name: 'A页面',
    comp: markRaw(TabA)
  },
  {
    name: 'B页面',
    comp: markRaw(TabB)
  }
])

// 场景2：单独存放当前组件 —— 用 shallowRef 浅层引用，不做响应式劫持
const currentComp = shallowRef(TabA)

// 接收子组件事件
const handleReceive = (val: string) => {
  console.log('父收到子组件数据：', val)
}
</script>

<style>
button.active { background: #409eff; color: #fff; border:none; padding:4px 8px; margin:0 4px }
</style>
```

---

# 关键知识点

## 1. 为什么要用 `markRaw`
- 组件本身是**静态构造函数/对象**，不需要响应式
- 放入 `reactive` 会被 Vue 深层 Proxy，造成：
  - 控制台警告
  - 不必要性能损耗
  - 潜在渲染异常
- `markRaw`：**永久标记对象不可被响应式处理**（Vue 官方 API）

## 2. 为什么要用 `shallowRef`
- 普通 `ref` 会对内部对象深度监听
- `shallowRef` 仅监听 `.value` 引用变化，**不劫持组件内部**
- 适合：存放组件、DOM、第三方实例、大型类

## 3. `<component :is>` 动态组件
- Vue3 组合式：直接绑定**组件引入对象**
- Vue2 选项式：绑定**组件名字符串**
- 同一挂载点，无缝切换不同视图

## 4. `<KeepAlive>` 作用
- 默认切换组件会 `unmount` 销毁
- 被 KeepAlive 包裹：组件**缓存、不销毁**
- 保留表单输入、变量、定时器、滚动位置

---
