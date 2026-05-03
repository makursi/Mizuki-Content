---
title: 自定义hook
published: 2026-05-02
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](./react/reactcute.webp)

# React 自定义 Hook 全解析
## 自定义hook的官方定义
React 自定义 Hook 是一个**以 `use` 开头的 JavaScript 函数**，它可以调用其他 Hook（如 `useState`、`useEffect`、`useContext` 等），目的是将组件中可复用的逻辑抽取出来，形成独立的、可复用的函数。

从 React 官方定义来看，自定义 Hook 并非 React 提供的内置 API，而是开发者基于 React 内置 Hook 遵循“Hook 规则”创造的复用逻辑的方式，它本质上是一种代码复用的约定，而非 React 语法层面的新特性。

## 自定义hook的核心作用
1. **逻辑复用**：将多个组件中重复的业务逻辑（如数据请求、表单处理、状态管理、事件监听等）抽取到自定义 Hook 中，实现一次编写、多处复用，减少代码冗余。
2. **组件解耦**：让组件只专注于 UI 渲染，将复杂的逻辑（如数据处理、副作用管理）剥离到自定义 Hook 中，提升组件的可读性和可维护性。
3. **逻辑隔离**：每个自定义 Hook 聚焦单一功能，符合“单一职责原则”，便于测试、调试和迭代。
4. **遵循 Hook 规则**：自定义 Hook 内部可以正常使用所有内置 Hook，同时保证 Hook 的调用规则（只在函数组件/自定义 Hook 顶层调用、不嵌套在条件/循环中），让逻辑复用更规范。

## 自定义hook的使用方法
### 1. 定义规则
- 函数名**必须以 `use` 开头**（如 `useFetch`、`useForm`），这是 React 识别自定义 Hook 的约定，也能让编辑器/IDE 检测 Hook 规则的违规使用。
- 内部可以调用任意 React 内置 Hook（`useState`、`useEffect` 等），但需遵循 Hook 调用规则：
  - 只在函数组件、自定义 Hook 的**顶层**调用（不能嵌套在 `if`、`for`、回调函数中）；
  - 只从 React 函数组件或自定义 Hook 中调用（不能从普通 JavaScript 函数中调用）。
- 可以接收参数，也可以返回任意类型的值（状态、方法、对象、数组等），灵活适配不同场景。

### 2. 基本步骤
#### 步骤1：抽取复用逻辑，定义自定义 Hook
```javascript
// 示例：封装数据请求的自定义 Hook
import { useState, useEffect } from 'react';

function useFetch(url) {
  // 定义状态：数据、加载状态、错误信息
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // 副作用：发送请求
  useEffect(() => {
    // 取消请求的控制器
    const controller = new AbortController();
    const signal = controller.signal;

    // 异步请求函数
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url, { signal });
        if (!response.ok) throw new Error('请求失败');
        const result = await response.json();
        setData(result);
        setError(null);
      } catch (err) {
        if (err.name !== 'AbortError') { // 排除取消请求的错误
          setError(err.message);
          setData(null);
        }
      } finally {
        if (!signal.aborted) setLoading(false);
      }
    };

    fetchData();

    // 清理函数：组件卸载/依赖变化时取消请求
    return () => controller.abort();
  }, [url]); // 依赖 url，url 变化时重新请求

  // 返回状态和数据，供组件使用
  return { data, loading, error };
}
```

#### 步骤2：在组件中使用自定义 Hook
```javascript
// 示例：使用 useFetch 的组件
function UserList() {
  // 调用自定义 Hook，传入请求地址
  const { data: users, loading, error } = useFetch('https://api.example.com/users');

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误：{error}</div>;

  return (
    <ul>
      {users?.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### 3. 进阶用法
- **自定义 Hook 组合**：一个自定义 Hook 可以调用另一个自定义 Hook，实现逻辑的分层复用（如 `useForm` 内部调用 `useState`、`useEffect`，还可以调用 `useValidate` 校验 Hook）；
- **返回回调函数**：自定义 Hook 可返回方法，让组件控制逻辑（如 `useModal` 返回 `openModal`、`closeModal` 方法）；
- **默认参数/可选参数**：提升 Hook 的灵活性（如 `useFetch(url, { method: 'GET' })`）。

## 自定义hook的使用案例
### 案例1：表单处理 Hook
```javascript
// 自定义 Hook：处理表单状态和提交
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);

  // 处理输入变化
  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };

  // 处理表单提交
  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(values);
  };

  // 重置表单
  const resetForm = () => setValues(initialValues);

  return { values, handleChange, handleSubmit, resetForm };
}

// 组件中使用
function LoginForm() {
  const { values, handleChange, handleSubmit, resetForm } = useForm(
    { username: '', password: '' },
    (values) => {
      console.log('提交表单：', values);
      // 实际登录逻辑
    }
  );

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>用户名：</label>
        <input
          type="text"
          name="username"
          value={values.username}
          onChange={handleChange}
        />
      </div>
      <div>
        <label>密码：</label>
        <input
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange}
        />
      </div>
      <button type="submit">登录</button>
      <button type="button" onClick={resetForm}>重置</button>
    </form>
  );
}
```

### 案例2：监听窗口尺寸 Hook
```javascript
// 自定义 Hook：监听窗口尺寸
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    // 监听窗口大小变化
    window.addEventListener('resize', handleResize);
    // 清理函数：移除监听
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// 组件中使用
function ResponsiveComponent() {
  const { width, height } = useWindowSize();

  return (
    <div>
      <p>窗口宽度：{width}px</p>
      <p>窗口高度：{height}px</p>
      <p>{width < 768 ? '移动端' : '桌面端'}</p>
    </div>
  );
}
```

## 自定义hook的使用注意事项
1. **严格遵循 Hook 调用规则**：
   - 不要在条件语句、循环、嵌套函数（如 `if` 块、`for` 循环、组件的回调函数）中调用自定义 Hook，必须在组件/自定义 Hook 的顶层调用；
   - 只在 React 函数组件或自定义 Hook 中调用，禁止在普通 JS 函数中调用（否则会破坏 React 的 Hook 状态管理机制）。

2. **命名必须以 `use` 开头**：
   这是 React 的强制约定，不仅能让开发者快速识别 Hook，还能让 ESLint 插件（`eslint-plugin-react-hooks`）检测出 Hook 规则的违规使用，避免隐藏 bug。

3. **自定义 Hook 是独立的状态隔离**：
   每次调用自定义 Hook，其内部的 `useState`、`useEffect` 等都会创建独立的状态和副作用，不同组件使用同一个自定义 Hook 时，状态互不干扰（这是自定义 Hook 与普通工具函数的核心区别）。

4. **合理处理清理逻辑**：
   自定义 Hook 中的 `useEffect`、`useLayoutEffect` 等副作用，必须处理清理逻辑（如取消请求、移除事件监听、清除定时器），避免内存泄漏。

5. **避免过度抽象**：
   只抽取真正复用的逻辑，不要为了抽象而抽象。如果某个逻辑仅在单个组件中使用，无需封装为自定义 Hook，否则会增加代码复杂度。

6. **明确依赖项**：
   自定义 Hook 中的 `useEffect`、`useCallback`、`useMemo` 等，要确保依赖项数组准确（可通过 `eslint-plugin-react-hooks` 的 `exhaustive-deps` 规则校验），避免因依赖缺失导致的闭包陷阱或无效渲染。

7. **返回值设计要清晰**：
   自定义 Hook 的返回值建议使用对象（而非数组），便于后续扩展（如新增返回值时，组件无需调整解构顺序）；如果返回值较少，也可使用数组（类似 `useState`），但需保持一致性。

8. **类型提示（TypeScript）**：
   使用 TypeScript 开发时，为自定义 Hook 的参数、返回值添加明确的类型注解，提升代码可读性和类型安全性。

# 社区中一些优质的React hooks库

[hooks库](reacthooklibraries.md)
