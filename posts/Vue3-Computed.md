---
title: Vue3-Computed
published: 2026-04-08
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---


这篇文章讲的是 **Vue3 Composition API 中 computed 计算属性**，我结合 **Vue 官方最新文档** 帮你梳理、校正、深化，让你既看懂原文，又符合官方规范。

---

# 一、computed 官方核心定义（Vue3）
**computed 计算属性**：
- 基于**响应式依赖**自动计算的**派生值**
- 自带**缓存**：依赖不变时，多次访问直接返回缓存，不重复执行
- 惰性求值：只有被读取时才计算
- 返回一个**只读 Ref**（可写需用 get/set 对象形式）

> 原文对核心特性描述基本正确，但缺少**官方规范约束**与**边界说明**。

---

# computed计算属性写法格式
## 1. 函数式（只读，最常用）
```ts
import { computed, ref } from 'vue'
let price = ref(0)
let m = computed<string>(() => `$` + price.value)
```

**官方规范**：
- 必须依赖 **ref/reactive** 等响应式数据
- getter **必须是纯函数**：无副作用、不异步、不修改外部状态
- 返回值是 **ComputedRef**，模板自动解包，JS 中用 `.value`

## 2. 对象式（可写计算属性）
原文写法：
```ts
let mul = computed({
  get: () => price.value,
  set: (value) => { price.value = 'set' + value }
})
```

**官方校正**：
- 可写 computed 本质是**封装依赖的修改逻辑**，不是真的“存值”
- 官方强烈建议：**只在必须双向绑定时使用**，避免滥用

---

# 三、购物车案例

    <template>
        <div>
            <input placeholder="请输入名称" v-model="keyWord" type="text">
            <table style="margin-top:10px;" width="500" cellspacing="0" cellpadding="0" border>
                <thead>
                    <tr>
                        <th>物品</th>
                        <th>单价</th>
                        <th>数量</th>
                        <th>总价</th>
                        <th>操作</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in searchData">
                        <td align="center">{{ item.name }}</td>
                        <td align="center">{{ item.price }}</td>
                        <td align="center">
                            <button @click="item.num > 1 ? item.num-- : null">-</button>
                            <input v-model="item.num" type="number">
                            <button @click="item.num < 99 ? item.num++ : null">+</button>
                        </td>
                        <td align="center">{{ item.price * item.num }}</td>
                        <td align="center">
                            <button @click="del(index)">删除</button>
                        </td>
                    </tr>
                </tbody>
                <tfoot>
                    <tr>
                        <td colspan="5" align="right">
                            <span>总价：{{ total }}</span>
                        </td>
                    </tr>
                </tfoot>
    
            </table>
        </div>
    </template>
    
    <script setup lang='ts'>
    import { reactive, ref,computed } from 'vue'
    let keyWord = ref<string>('')
    interface Data {
        name: string,
        price: number,
        num: number
    }
    const data = reactive<Data[]>([
        {
            name: "小满的绿帽子",
            price: 100,
            num: 1,
        },
        {
            name: "小满的红衣服",
            price: 200,
            num: 1,
        },
        {
            name: "小满的黑袜子",
            price: 300,
            num: 1,
        }
    ])
    
    let searchData = computed(()=>{
        return data.filter(item => item.name.includes(keyWord.value))
    })
    
    let total = computed(() => {
        return data.reduce((prev: number, next: Data) => {
            return prev + next.num * next.price
        }, 0)
    })
    
    const del = (index: number) => {
        data.splice(index, 1)
    }
    
    </script>
    
    <style scoped lang='less'></style>

## 使用 computed 进行：
1. **搜索过滤**：`searchData = computed(() => data.filter(...))`
2. **总价汇总**：`total = computed(() => data.reduce(...))`

### ✅ 优点
- 用 computed 做**数据派生/过滤/聚合**，比 methods 性能高
- 依赖变化自动更新，代码更简洁
- 模板直接使用，无冗余逻辑

### ⚠️ 官方建议优化
- 计算属性**不要做异步**、不要做 DOM 操作、不要修改依赖
- 复杂列表计算建议**拆分**，避免单个 getter 过于臃肿

---

# 四、手写源码



---

# 五、重点
## 1. computed vs methods（官方核心区别）
| 特性 | computed | methods |
|---|---|---|
| 缓存 | 有，依赖不变不重算 | 无，每次调用都执行 |
| 执行时机 | 惰性，读取才算 | 主动调用才执行 |
| 用途 | 派生值、格式化、过滤 | 事件处理、异步、副作用 |

## 2. 官方禁忌
- ❌ 不要在 computed 里发请求、定时器、修改 DOM
- ❌ 不要依赖非响应式变量（普通 let/const）
- ❌ 不要在 getter 里修改依赖（造成死循环）
- ✅ 必须 return 一个值

## 3. Vue3.4+ 新特性（原文无）
- 支持 getter 接收 **previous** 参数，可基于旧值计算
- 性能进一步优化，依赖追踪更精准

---
