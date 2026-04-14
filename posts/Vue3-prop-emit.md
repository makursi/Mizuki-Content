---
title: Vue-父子组件通信
published: 2026-04-13
description: ''
image: ''
tags: []
category: 'Vue'
draft: false 
lang: ''
---
# 1.  父组件向子组件传值：defineProps 接收参数（普通写法 + TS 纯类型写法）

**官方结论**：
> `defineProps` 是 `<script setup>` 中**专门用来接收父组件传值**的编译器宏，**不需要 import**，直接用。
> 它有两种写法：
> 1. **普通（运行时）写法**：适合 JS 项目，靠运行时校验
> 2. **TS 纯类型写法**：适合 TS 项目，靠编译时类型校验

---

## 父组件怎么传值？
### 1. 传【静态字符串】：不用 v-bind
```vue
<!-- 父组件 -->
<Child msg="我是字符串" />
```
- 不加 `:` → 当成 **HTML 原生属性** → 值永远是字符串
- 只能传固定字符串

### 2. 传【非字符串 / 变量】：必须用 v-bind / :
```vue
<!-- 父组件 -->
<Child 
  :num="123"         <!-- 数字 -->
  :isShow="true"     <!-- 布尔 -->
  :list="[1,2,3]"    <!-- 数组 -->
  :user="{name:'zs'}"<!-- 对象 -->
  :custom="data"     <!-- 变量 -->
/>
```
- 加 `:` → 当成 **JS 表达式** → 保留真实类型
- 数字、布尔、数组、对象、变量、表达式**必须用 v-bind**

---

## 二、子组件接收：defineProps 两种完整写法
## 写法 1：普通（运行时）写法 —— 适合 JavaScript
### 完整语法（官方标准）
```vue
<script setup>
// 直接使用，无需导入
const props = defineProps({
  // 字段名: 类型 / 配置项
  msg: String,
  num: {
    type: Number,        // 类型
    required: true,      // 是否必填
    default: 0,          // 默认值
    validator: (val) => { // 自定义校验
      return val > 0
    }
  }
})
</script>
```

### 支持的配置项（官方完整）
| 配置 | 作用 |
|---|------|
| `type` | 数据类型（String/Number/Boolean/Object/Array/Function） |
| `required` | 是否为必传参数 |
| `default` | 默认值（没传时用这个） |
| `validator` | 自定义校验函数 |

### 引用类型默认值规则（重要）
对象/数组默认值**必须用函数返回**：
```js
defineProps({
  list: {
    type: Array,
    default: () => [] // 函数返回，避免多实例共享
  },
  user: {
    type: Object,
    default: () => ({})
  }
})
```

### 优点 & 缺点
 **优点**：
- 纯 JS 就能用，不用 TS
- 自带**运行时校验**，控制台会报错提示
- 支持默认值、必填、自定义校验

**缺点**：
- 没有 TS 类型提示
- 大型项目维护困难

---

## 写法 2：TS 纯类型写法 —— 官方推荐（TypeScript）
### 完整语法（官方标准）
```vue
<script setup lang="ts">
// 纯类型写法：只用 TS 类型，不写任何运行时配置
const props = defineProps<{
  msg: string               // 必传字符串
  num?: number              // ? = 可选（非必传）
  list: number[]            // 数字数组
  user: { name: string }    // 对象
  fn: (id: number) => void  // 函数
}>()
</script>
```

### 官方定义：什么是「纯类型语法」？
解答：
> **使用 TypeScript 可以使用传递字面量类型的纯类型语法做为参数**

**直白解释：**
- 只写 **TS 类型**（`string`/`number`/`[]`/`{}`/接口）
- 不写 `type: String`、`required`、`default` 等运行时字段
- 完全靠 **TS 类型系统** 做校验

### 必传 / 可选 怎么写？
- 必传：`msg: string`
- 可选：`msg?: string`（加问号）

### 进阶：用 interface 抽离类型（更优雅）
```ts
// 定义接口
interface User {
  name: string
  age: number
}

// 纯类型声明
defineProps<{
  user: User
  list?: number[]
}>()
```

### 优点 & 缺点
**优点（官方主推）**：
1. **编译期校验**：传错类型直接编辑器标红，不用运行
2. **极强类型提示**：用 `props.xxx` 自动提示类型
3. **代码更简洁**
4. 开发环境自动生成运行时校验

**缺点**：
- 纯类型写法**不能直接写默认值**！
- 必须配合 `withDefaults` 使用

---

# 三、TS 纯类型 + 默认值：withDefaults
```vue
<script setup lang="ts">
interface Props {
  msg?: string
  num?: number
  list?: number[]
}

// withDefaults( defineProps, 默认值对象 )
const props = withDefaults(defineProps<Props>(), {
  msg: "默认消息",
  num: 100,
  list: () => [1,2,3] // 引用类型必须用函数！
})
</script>
```

### 强制规则（官方）
- 基本类型：直接写默认值
- **对象/数组：必须用函数返回**（避免多个组件共用一个引用）

---

# 模板样例
## 1. JS 项目 → 普通写法
```vue
<script setup>
const props = defineProps({
  msg: String,
  num: {
    type: Number,
    default: 0
  },
  list: {
    type: Array,
    default: () => []
  }
})
</script>
```

## 2. TS 项目 → 纯类型 + withDefaults
```vue
<script setup lang="ts">
const props = withDefaults(defineProps<{
  msg?: string
  num?: number
  list?: number[]
}>(), {
  msg: "默认",
  num: 0,
  list: () => []
})
</script>
```


# 2.  子组件向父组件传值：defineEmits 触发自定义事件
---

## 一、核心前置概念
1. **数据流规则**
Vue 默认：
- 父 → 子：单向下行（通过 `props`）
- 子 → 父：**不能直接修改父数据**，必须通过「自定义事件派发」通知父

2. **`defineEmits` 作用**
`<script setup>` 内置编译器宏（无需 import）：
- 用于**声明当前组件可以触发哪些自定义事件**
- 子组件主动调用 `emit()` 触发事件，携带参数
- 父组件通过 `@事件名` 监听，接收子组件传递的数据

---

## 二、完整流程总览
1. 子组件：`const emit = defineEmits(事件列表/类型约束)`
2. 子组件：`emit('自定义事件名', 需要传给父的数据)`
3. 父组件：`<Child @自定义事件名="父函数" />`
4. 父函数：自动接收子组件传来的参数，完成赋值/业务逻辑

---

## 三、写法一：基础数组写法（通用、JS/TS 都能用）
### 1. 子组件（触发事件 + 传值）
```vue
<!-- 子组件 Child.vue -->
<script setup>
// 1. 声明当前组件要抛出的自定义事件（数组形式）
const emit = defineEmits(['sendData', 'changeCount'])

// 测试方法：点击触发事件
const handleClick = () => {
  // 2. 触发自定义事件，第二个参数为【传给父的值】
  emit('sendData', { name: "小明", age: 18 })
  emit('changeCount', 100)
}
</script>

<template>
  <button @click="handleClick">向父组件传值</button>
</template>
```

### 2. 父组件（监听事件 + 接收数据）
```vue
<!-- 父组件 Parent.vue -->
<template>
  <!-- 3. 父组件使用 @事件名 监听 -->
  <Child 
    @sendData="getChildData"
    @changeCount="handleCount"
  />
</template>

<script setup>
// 4. 回调函数自动接收子组件传递的参数
const getChildData = (data) => {
  console.log("子组件传来的数据：", data) 
  // { name: "小明", age: 18 }
}

const handleCount = (num) => {
  console.log("子组件数字：", num) // 100
}
</script>
```

### 特点
- 简单轻量，适合简单业务
- 无类型约束，TS 项目不推荐
- 仅做「事件名声明」，不约束传参格式

---

## 四、写法二：TS 纯类型写法
### 1. 完整标准语法
```vue
<!-- 子组件 -->
<script setup lang="ts">
// 纯类型约束：约束 事件名 + 传参类型 + 返回值
const emit = defineEmits<{
  // 格式：(事件名: 字面量, 参数: 类型): 返回值
  (e: 'sendData', user: { name: string; age: number }): void
  (e: 'changeCount', count: number): void
  (e: 'close'): void // 无参事件
}>()

const handleClick = () => {
  emit('sendData', { name: "小红", age: 20 })
  emit('changeCount', 66)
  emit('close')
}
</script>
```

### 核心优势（对应官方文档）
1. **强类型校验**
子组件 `emit` 传参写错类型，直接编辑器爆红，编译拦截错误；
2. **事件名强约束**
只能抛出定义过的事件名，拼写错误直接报错；
3. **IDE 智能提示**
输入 `emit(` 自动提示合法事件名、对应参数类型；
4. **可维护性极强**
多人协作时，一眼看清每个事件需要传递什么格式的数据。

---

## 五、进阶：拆分类型
复杂项目可以抽离类型，代码更整洁：
```ts
<script setup lang="ts">
// 先定义参数类型
type User = {
  name: string
  age: number
}

// 定义事件类型
type Emits = {
  sendData: [user: User]
  changeCount: [count: number]
  close: []
}

// 使用
const emit = defineEmits<Emits>()
</script>
```

---

## 六、关键细节：emit 可以传递多个参数
子组件支持一次性传**多个参数**：
```ts
// 子组件
emit('info', "标题", 200, [1,2,3])

// 父组件接收
const getInfo = (title: string, num: number, list: number[]) => {
  console.log(title, num, list)
}
```

---

## 七、底层核心原理
1. **组件实例绑定**
每个组件实例内部会维护一个 `$emit` 方法；
`defineEmits` 是语法糖，本质就是**封装组件原生 $emit**。

2. **模板编译机制**
- 父组件 `@sendData`：编译时会绑定一个事件回调，挂载到子组件事件监听器；
- 子组件 `emit('sendData')`：向上派发自定义事件，触发父绑定的回调函数；

3. **单向数据流底层保障**
子永远不会直接修改父的响应式数据；
所有修改逻辑**收敛在父组件内部**，符合 Vue 设计规范，避免数据流混乱。

4. **与原生 DOM 事件区别**
- 原生事件：`click` / `input` 是浏览器原生；
- 自定义事件：`sendData` / `changeCount` 是 Vue 组件层面自定义，**不会冒泡、不会穿透**。

---

## 八、注意事项
1. 禁止在子组件直接修改 props
```ts
// 错误
props.name = "xxx"

// 正确
emit('update:name', "xxx") // 推荐 update 语法
```

2. 事件名规范
Vue 推荐：自定义事件使用 **小驼峰 / 短横线命名**，统一团队规范；

3. `defineEmits` 只能在 `<script setup>` 使用
非 setup 语法写法不同，现在项目基本废弃。

---

## 九、补充：双向绑定简写
实际开发常用 `v-model` 语法糖，底层依旧是 `props + emit`：
- 接收：`modelValue`
- 派发：`update:modelValue`
本质还是 **props 下行 + emit 上行**。


# 3.  父组件获取子组件实例 / 内部属性：defineExpose
---

## 一、官方原文
> 在 `<script setup>` 中，组件是**默认关闭**的——即通过模板引用获取到的子组件实例，**无法访问** `<script setup>` 内声明的任何响应式数据、方法、变量。
> 如果你希望父组件能访问子组件内部的属性/方法，**必须使用 `defineExpose` 编译器宏显式暴露**。

文档地址：
[defineExpose 官方文档](https://cn.vuejs.org/api/sfc-script-setup.html#defineexpose)

---

## 二、核心关键点：为什么要有 `defineExpose`？
### 1. 默认行为（重要）
在 **非 `<script setup>`** 的写法里：
```js
export default {
  data() { return { name: '子组件数据' } },
  methods: { test() {} }
}
```
父组件用 `ref` 可以直接拿到 `name` 和 `test`。

### 2. `<script setup>` 的安全机制（封闭性）
`<script setup>` 为了**组件封装安全**，默认：
- 所有变量、函数、状态 **全部私有**
- 父组件完全访问不到
- 避免外部随意修改内部状态，保证组件健壮性

### 3. 所以需要 `defineExpose`
它的作用只有一句话：
**手动选择哪些内部属性/方法可以暴露给父组件。**

---

## 三、完整使用流程（标准官方用法）
### 步骤 1：子组件内部 —— 使用 `defineExpose` 暴露
```vue
<!-- 子组件 Child.vue -->
<script setup>
import { ref } from 'vue'

// 1. 子组件内部数据（默认父组件访问不到）
const childMsg = ref('我是子组件的消息')
const childMethod = () => {
  console.log('我是子组件方法')
}

// 2. 主动暴露给父组件（核心代码）
defineExpose({
  childMsg,
  childMethod
})
</script>
```

### 步骤 2：父组件 —— 用 `ref` 获取子组件实例
```vue
<!-- 父组件 Parent.vue -->
<template>
  <!-- 给子组件标记 ref -->
  <Child ref="childRef" />
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

// 1. 创建同名 ref，用于接收子组件实例
const childRef = ref(null)

// 2. 在挂载后访问子组件（必须在 onMounted 中）
onMounted(() => {
  console.log(childRef.value.childMsg) // 获取数据
  childRef.value.childMethod() // 调用方法
})
</script>
```

---

## 四、重要规则
### 1. 暴露什么，父组件才能用什么
你不写进 `defineExpose` 的内容，**父组件永远访问不到**。

### 2. 只能在 `onMounted` 之后获取实例
模板渲染完成后，`ref` 才会绑定实例，否则为 `null`。

### 3. 响应式数据会自动保持同步
子组件内部修改 `childMsg`，父组件获取到的值**自动更新**。

### 4. 不推荐过度使用
官方说明：
> **优先使用 props + emit 数据流**
> `defineExpose` 属于**直接操作子实例**，过度使用会破坏单向数据流。

---

## 五、TypeScript 官方规范写法（类型安全版）
如果使用 TS，必须给子组件实例**定义类型**，否则 TS 会报错。

### 子组件（暴露）
```vue
<script setup lang="ts">
import { ref } from 'vue'
const msg = ref('hello')
const fn = () => 123

defineExpose({
  msg,
  fn
})
</script>
```

### 父组件（TS 类型写法）
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

// 1. 获取子组件实例类型
const childRef = ref<InstanceType<typeof Child> | null>(null)

onMounted(() => {
  childRef.value?.msg
  childRef.value?.fn()
})
</script>
```

这是 **Vue 官方推荐的 TS 写法**。

---

## 六、`defineExpose` 核心作用总结（面试/理解必备）
1. **封装性保障**
   `<script setup>` 默认封闭，保护组件内部状态不被外部随意修改。

2. **显式暴露**
   开发者主动控制哪些属性/方法可以被外部访问，更安全。

3. **支持父组件主动调用子组件**
   例如：
   - 父让子表单重置
   - 父调用子组件的滚动方法
   - 父获取子组件内部状态

4. **不破坏单向数据流，但属于“直接通信”**
   它是 Vue 支持的**合法跨组件直接调用方案**。


---

# 4.  官方文档精讲：`withDefaults` —— 给 TS 纯类型 Props 设置默认值

**官方原文**：
> 当使用**基于类型的 Props 声明**时（也就是你学的 `<script setup>` + TS 纯类型写法），无法直接提供默认值。
> 对此，Vue 提供了 `withDefaults` 这个编译器宏，**允许为纯类型 Props 指定默认值**。

官方文档：
[withDefaults - 官方文档](https://cn.vuejs.org/api/sfc-script-setup.html#withdefaults)

---

## 一、先搞懂：为什么必须用 `withDefaults`？
### 1. 痛点：TS 纯类型写法 **不能写默认值**
你之前学的 **纯类型 Props** 写法：
```ts
// 这是纯 TS 类型，只用于编译检查，不参与运行
defineProps<{
  title: string
  count?: number // ? 代表可选
}>()
```
❌ **错误写法：不能直接在类型里写 default**
```ts
// 这是 TS 类型语法，根本不支持 default，会直接报错
defineProps<{
  title: string = "默认标题" // 错误！TS 语法不支持
}>()
```

### 2. 核心原因
- **纯类型写法**：只在**编译时**做检查，运行时会被擦除，Vue 运行时拿不到默认值。
- **运行时需要**：Vue 必须在运行时知道默认值是什么，才能在父组件没传参时生效。

**解决方案**：
Vue 专门提供 **`withDefaults` 宏** —— **给纯类型 Props 补充运行时默认值**。

---

## 二、官方标准
### 完整语法
```ts
withDefaults( defineProps<类型>(), 默认值配置对象 )
```

### 最简可运行 Demo
```vue
<script setup lang="ts">
// 1. 用 withDefaults 包裹
const props = withDefaults(
  // 第一步：纯类型 Props 声明
  defineProps<{
    title: string
    count?: number // 可选
    list?: number[] // 可选数组
  }>(),

  // 第二步：设置默认值（运行时生效）
  {
    title: "默认标题",
    count: 0,
    list: () => [1, 2, 3] // 重点：引用类型必须用函数返回！
  }
)
</script>
```

---

## 三、官方规则
### 规则 1：基本类型（string/number/boolean）直接赋值
```ts
{
  title: "默认标题",
  count: 0,
  disabled: false
}
```

### 规则 2：引用类型（数组/对象）**必须用工厂函数返回**
这是 Vue **官方强制规范**：
```ts
{
  // 正确：函数返回，每个组件实例独立一份，不共享
  list: () => [1, 2, 3],
  user: () => ({ name: "默认名" })

  // ❌ 错误：直接写会造成多组件实例共享同一个对象/数组！
  // list: [1,2,3]
}
```
**原因**：防止多个子组件共用同一个引用，导致改一个影响全部。

### 规则 3：可选属性 `?` + 默认值 = TS 自动推导为非空
```ts
defineProps<{
  count?: number // 原本可能是 undefined
}>()

// 设置默认值后
const props = withDefaults(..., { count: 0 })

props.count // TS 自动推导为 number，永远不会 undefined
```
这就是 **类型收敛**，非常安全。

---

## 四、进阶优雅写法：用 interface 抽离类型
官方推荐复杂 Props 用接口定义，更清晰：
```vue
<script setup lang="ts">
// 1. 定义 Props 接口
interface Props {
  title: string
  count?: number
  list?: number[]
}

// 2. 传入接口 + 默认值
const props = withDefaults(defineProps<Props>(), {
  title: "默认标题",
  count: 0,
  list: () => [1, 2, 3]
})
</script>
```

---

## 五、`withDefaults` 核心作用
1. **专门服务 TS 纯类型 Props**
   只有纯类型写法才需要它，普通运行时 Props 直接写 `default` 即可。

2. **兼顾「编译时类型安全」+「运行时默认值」**
   - TS 检查：编译前拦截类型错误
   - 默认值：运行时父组件不传参也能正常显示

3. **保持代码简洁**
   不写冗余的 `type`、`required`，只用 TS 类型 + 默认值配置。

4. **自动生成运行时 Props 定义**
   Vue 编译器会把 `withDefaults` 自动转为 Vue 运行时识别的默认值配置。
