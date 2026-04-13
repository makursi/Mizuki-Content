---
title: Vue-watchEffect
published: 2026-04-08
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---

# 定位
**watchEffect = 自动追踪依赖的高级侦听器  
不用指定监听谁，用到谁就监听谁。**

---

# 一、watchEffect 官方核心定义（文章完全正确）
> 立即执行一次函数  
> 内部**自动追踪响应式依赖**  
> 依赖变化时**重新执行**

特点：
- **非惰性**（一开始就执行）
- **自动收集依赖**（不用写监听源）
- **代码更简洁**

---

# 二、文章第一段：最基础用法
```js
let message = ref('')
let message2 = ref('')

watchEffect(() => {
  console.log(message2.value)
})
```

### 发生了什么？
1. 立即执行一次
2. 内部用到了 **message2**
3. 自动只监听 **message2**
4. message 变了**不触发**

### 大白话
你在回调里用了谁，Vue 就自动监听谁，**不用手动声明**。

---

# 三、文章第二段：清除副作用（onInvalidate）
```js
watchEffect((onInvalidate) => {
  console.log(message2.value)

  onInvalidate(() => {
    // 在下一次执行之前 先跑这里
  })
})
```
onInvalidate 是 “上一次操作的清理函数”。

## onInvalidate 的执行时机
- 下一次 watchEffect 重新执行前
- 组件卸载时
- 手动调用 stop () 停止监听时

总结:当旧副作用消失前,先清理

# 经典场景：定时器 / 防抖
> 输入框一变，就发请求，但为了不频繁发，要加防抖。

```js
watchEffect(() => {
  // 每次 keyword 变，都会进来
  const timer = setTimeout(() => {
    console.log('发送请求')
  }, 500)
})
```

会出大问题：
**keyword 连续变 → 一堆定时器同时执行 → 同时发 N 个请求**

这时候就需要 **onInvalidate** 来清理上一次的定时器：

```js
watchEffect((onInvalidate) => {
  const timer = setTimeout(() => {
    console.log('发送请求')
  }, 500)

  // 关键点：
  // 下次执行前，先把上一次的定时器清掉
  onInvalidate(() => {
    clearTimeout(timer)
  })
})
```

效果：
- 你连续快速输入
- 每次新的执行都会**先清掉上一次的定时器**
- 最后停下 500ms 才发一次请求

这就是它最核心的用途。

---

# 第二个经典场景：取消请求
```js
watchEffect(async (onInvalidate) => {
  const controller = new AbortController()
  const signal = controller.signal

  onInvalidate(() => {
    // 下次重新执行前，取消上一次的请求
    controller.abort()
  })

  await fetch('/api/search', { signal })
})
```

作用：
**上一个请求还没回来，新的又来了 → 自动取消旧请求**

---

# 第三个场景：清除事件监听
```js
watchEffect((onInvalidate) => {
  function handler() {}
  window.addEventListener('scroll', handler)

  onInvalidate(() => {
    window.removeEventListener('scroll', handler)
  })
})
```

---

# 核心逻辑
**onInvalidate 不是“第一次执行”用的，
它是“下一次执行”和“销毁”用的。**

流程永远是：

1. 进入 watchEffect
2. 做一些操作（定时器、请求、监听…）
3. 注册 onInvalidate 清理函数
4. 下次依赖变化 → **先执行清理 → 再重新执行**

---

### 官方解释
- **每次重新执行前，会先调用清理函数**
- 用于：取消请求、清除定时器、取消事件监听
- 组件卸载时也会自动执行一次

### 最常用场景：防抖、取消请求
```js
watchEffect((onInvalidate) => {
  const timer = setTimeout(() => {
    console.log('请求')
  }, 500)

  // 下次执行前先清掉定时器
  onInvalidate(() => clearTimeout(timer))
})
```

---

# 四、停止监听（stop 方法）
```js
const stop = watchEffect(() => {
  console.log(message.value)
})

// 调用后：停止监听
stop()
```

### 官方作用
- 手动**停止追踪依赖**
- 适合：组件销毁、条件关闭监听

---

# 五、配置项 flush（执行时机）
文章给的表格非常标准：

| flush | 执行时机 |
|-------|----------|
| **pre** | 默认，组件更新**前**执行 |
| **sync** | 同步触发（少用） |
| **post** | 组件更新**后**执行（DOM 已更新） |

### 最常用：post
访问 **DOM** 必须用 `post`：
```js
watchEffect(() => {
  // 这里可以安全获取 DOM
}, { flush: 'post' })
```

---

# 六、onTrigger 调试
```js
watchEffect(() => {
  console.log(message.value)
}, {
  onTrigger() {
    // 调试：知道是谁触发了更新
    console.log('被触发了')
  }
})
```

官方作用：
**调试用，看依赖何时触发。**

---

# 七、watchEffect 与 watch 的核心区别（官方重点）
|  | watch | watchEffect |
|--|-------|-------------|
| 依赖 | **手动指定** | **自动收集** |
| 立即执行 | 默认不执行 | **默认立即执行** |
| 新值旧值 | 能获取 | **不能获取** |
| 精准度 | 精准 | 自动 |
| 场景 | 具体值监听 | 逻辑复杂、依赖多 |

---
