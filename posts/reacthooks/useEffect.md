---
title: useEffect
published: 2026-05-02
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](../react/reactcute.webp)

---
# useEffect

`useEffect` 是 React 提供的一个核心 Hook，用于在函数组件中执行**副作用操作**。副作用指那些不直接属于组件渲染流程的操作，比如数据获取、订阅/取消订阅、DOM 操作、手动修改状态等。它整合了类组件中 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 三个生命周期方法的能力，让函数组件能够处理生命周期相关的逻辑。

# 核心作用

1. **执行副作用**：处理组件渲染后需要执行的非渲染逻辑（如请求数据、操作 DOM、绑定事件监听）；
2. **控制执行时机**：通过依赖项数组精准控制副作用的执行时机（首次渲染后、指定状态/属性变化后）；
3. **清理副作用**：支持返回清理函数，处理组件卸载或副作用重新执行前的收尾工作（如取消订阅、清除定时器、取消网络请求）；
4. **替代类组件生命周期**：无需编写类组件，即可实现生命周期相关的逻辑，让代码更简洁、逻辑更内聚。

# 使用方法

## 基本语法
```jsx
import { useEffect } from 'react';

function MyComponent() {
  // 基础用法
  useEffect(() => {
    // 副作用执行的代码（核心逻辑）
    
    // 可选：清理函数（用于卸载/重新执行前清理）
    return () => {
      // 清理副作用的代码
    };
  }, [/* 依赖项数组 */]);

  return <div>组件内容</div>;
}
```

## 关键参数说明
1. **第一个参数（必选）**：一个函数，包含要执行的副作用逻辑，可选返回一个清理函数；
2. **第二个参数（可选）**：依赖项数组，控制 `useEffect` 的执行时机：
   - 不传递：组件**每次渲染后**都会执行副作用；
   - 空数组 `[]`：仅在组件**首次渲染后**执行（对应 `componentDidMount`）；
   - 包含特定值：仅在依赖项中的值**发生变化时**执行（对应 `componentDidUpdate`）。

## 常见使用场景语法示例
### 场景1：仅首次渲染执行（如初始化数据、绑定全局事件）
```jsx
useEffect(() => {
  // 首次渲染后请求数据
  fetch('/api/data').then(res => res.json()).then(data => {
    console.log('数据获取成功', data);
  });

  // 绑定全局事件
  window.addEventListener('resize', handleResize);

  // 清理函数：组件卸载时移除事件监听
  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []); // 空依赖数组 → 仅首次执行
```

### 场景2：依赖项变化时执行（如监听状态/属性变化）
```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  // count 变化时更新文档标题
  document.title = `当前计数：${count}`;
  
  // 清理函数：count 变化前执行（可选）
  return () => {
    console.log('count 将更新，旧值：', count);
  };
}, [count]); // 仅 count 变化时执行
```

### 场景3：每次渲染都执行（不推荐，易引发性能问题）
```jsx
useEffect(() => {
  // 每次渲染后执行（如实时同步某个状态到本地存储）
  localStorage.setItem('tempState', JSON.stringify(someState));
}); // 无依赖数组 → 每次渲染执行
```

# 使用案例

## 案例1：定时器管理（含清理）
```jsx
import { useState, useEffect } from 'react';

function TimerComponent() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    // 启动定时器：每秒更新秒数
    const timer = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // 清理函数：组件卸载时清除定时器
    return () => {
      clearInterval(timer);
      console.log('定时器已清除');
    };
  }, []); // 仅首次渲染启动定时器

  return (
    <div>
      <h1>已计时：{seconds} 秒</h1>
    </div>
  );
}
```

## 案例2：监听 props 变化并请求数据
```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    // 当 userId 变化时，重新请求用户信息
    const fetchUser = async () => {
      setLoading(true);
      try {
        const res = await fetch(`/api/users/${userId}`);
        const data = await res.json();
        setUser(data);
      } catch (err) {
        console.error('请求用户信息失败：', err);
      } finally {
        setLoading(false);
      }
    };

    // 若 userId 存在则执行请求
    if (userId) {
      fetchUser();
    }

    // 清理函数：取消未完成的请求（防止竞态问题）
    return () => {
      const controller = new AbortController();
      controller.abort(); // 中止请求
    };
  }, [userId]); // 依赖 userId，变化时重新执行

  if (loading) return <div>加载中...</div>;
  if (!user) return <div>暂无用户信息</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>邮箱：{user.email}</p>
      <p>年龄：{user.age}</p>
    </div>
  );
}

```
# 使用注意事项

1. **依赖项必须完整**：
   - 副作用中用到的所有组件内变量（状态、props、函数）都必须加入依赖项数组，否则可能导致闭包陷阱（读取到旧值）；
   - 若依赖项是函数，建议使用 `useCallback` 包裹，避免每次渲染创建新函数导致 `useEffect` 频繁执行。

2. **清理函数的执行时机**：
   - 组件卸载时执行；
   - 副作用因依赖项变化重新执行前执行；
   - 清理函数是同步的，不能是异步函数（若需异步清理，需在内部封装异步逻辑）。

3. **避免无限循环**：
   - 若在 `useEffect` 中修改依赖项本身（如 `setCount(count + 1)`），且依赖项包含该值，会导致无限执行；
   - 解决方案：使用函数式更新（`setCount(prev => prev + 1)`），或移除不必要的依赖。

4. **异步操作注意竞态问题**：
   - 当组件卸载或依赖项快速变化时，未完成的异步请求（如接口调用）可能仍会执行，导致更新已卸载组件的状态；
   - 解决方案：使用 `AbortController` 中止请求，或在清理函数中设置标志位阻止状态更新。

5. **不要在 useEffect 中直接修改状态（无依赖）**：
   - 若 `useEffect` 无依赖且内部修改状态，会导致组件无限渲染（渲染 → 执行 useEffect → 修改状态 → 重新渲染）。

6. **严格模式下的执行行为**：
   - React 严格模式下，`useEffect` 的副作用函数（除首次渲染）会**执行两次**（开发环境），用于检测潜在的清理问题；
   - 确保副作用和清理函数是幂等的（多次执行不会产生意外结果）。

7. **useEffect 是同步执行的**：
   - 副作用函数在组件渲染完成后**同步执行**（浏览器绘制前），若需延迟到绘制后执行，可包裹 `setTimeout`。
```
