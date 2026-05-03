---
title: useDebugValue
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
# useDebugValue

useDebugValue 是 React 提供的一个自定义 Hook，专门用于在 React 开发者工具中为自定义 Hook 添加可读的标签（调试值），它**不会影响组件的功能逻辑**，仅作用于调试阶段的体验优化，让开发者能更直观地识别自定义 Hook 的状态和返回值。

# 核心作用

1. **调试友好性**：为自定义 Hook 标注自定义的调试信息，替代 React 开发者工具中默认显示的“Custom Hook”等模糊标识，快速定位自定义 Hook 的运行状态；
2. **无侵入性**：仅在开发者工具中生效，生产环境下不会产生任何性能开销，也无需额外的环境判断；
3. **灵活性**：支持静态值或动态计算值（可结合函数延迟计算，避免不必要的性能消耗）。

# 使用方法

## 基本语法
```javascript
useDebugValue(value, formatFn?)
```
- **参数1（value）**：要在开发者工具中显示的调试值（可以是任意类型：字符串、对象、数组、布尔值等）；
- **参数2（formatFn，可选）**：一个函数，用于对调试值进行格式化处理，仅在开发者工具展开该 Hook 时才会执行（延迟计算，优化性能），函数返回格式化后的显示值。

## 核心使用场景
useDebugValue 仅推荐在**自定义 Hook 内部使用**，普通组件中使用无意义。

### 场景1：基础使用（静态/简单动态值）
自定义一个获取用户信息的 Hook，添加调试值：
```javascript
import { useDebugValue, useState, useEffect } from 'react';

// 自定义 Hook：获取用户信息
function useUserInfo(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/user/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]);

  // 为自定义 Hook 添加调试值：显示加载状态 + 用户名称
  useDebugValue(loading ? 'Loading...' : `User: ${user?.name}`);

  return { user, loading };
}
```
此时在 React 开发者工具中，使用该 Hook 的组件下会显示对应的调试值（如 `User: 张三` 或 `Loading...`）。

### 场景2：延迟格式化（优化性能）
如果调试值的计算逻辑较复杂（比如处理大对象、复杂数组），可通过第二个参数延迟计算：
```javascript
import { useDebugValue, useContext } from 'react';
import UserContext from './UserContext';

// 自定义 Hook：获取用户权限
function useUserPermissions() {
  const user = useContext(UserContext);
  
  // 复杂的格式化函数：仅在开发者工具展开时执行
  const formatPermissions = (perms) => {
    return perms?.length 
      ? `Permissions (${perms.length}): ${perms.join(', ')}` 
      : 'No permissions';
  };

  // 第二个参数传入格式化函数，避免每次渲染都执行复杂计算
  useDebugValue(user?.permissions, formatPermissions);

  return user?.permissions || [];
}
```

# 使用案例

## 案例1：基础调试标识（状态类自定义 Hook）
```javascript
import { useDebugValue, useState } from 'react';

// 自定义 Hook：管理表单输入
function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);
  
  // 标注当前输入值
  useDebugValue(`FormInput: ${value}`);

  const handleChange = (e) => {
    setValue(e.target.value);
  };

  return { value, onChange: handleChange };
}

// 组件中使用
function LoginForm() {
  const username = useFormInput('');
  const password = useFormInput('');

  return (
    <form>
      <input type="text" {...username} placeholder="用户名" />
      <input type="password" {...password} placeholder="密码" />
    </form>
  );
}
```
在开发者工具中，LoginForm 组件下会显示两个调试值：`FormInput: （输入的用户名）` 和 `FormInput: （输入的密码）`，快速区分两个 useFormInput 实例的状态。

## 案例2：复杂状态的调试（异步/上下文类 Hook）
```javascript
import { useDebugValue, useState, useEffect, useContext } from 'react';
import ThemeContext from './ThemeContext';

// 自定义 Hook：结合上下文 + 异步数据
function useThemedData(apiUrl) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    fetch(apiUrl)
      .then(res => res.json())
      .then(setData)
      .catch(setError);
  }, [apiUrl]);

  // 组合多个状态为调试值
  useDebugValue({
    theme: theme.mode,
    dataStatus: data ? 'Loaded' : error ? 'Error' : 'Loading',
    dataCount: data?.length || 0
  });

  return { data, error, theme };
}

// 组件中使用
function ThemedDataList() {
  const { data, error } = useThemedData('/api/list');
  
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return <div>Loading...</div>;
  
  return (
    <ul>
      {data.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```
在开发者工具中，ThemedDataList 组件下的调试值会显示一个对象：`{ theme: 'dark', dataStatus: 'Loaded', dataCount: 5 }`，直观看到上下文状态、异步数据状态和数据量。

# 使用注意事项

1. **仅用于自定义 Hook**：useDebugValue 设计初衷是为自定义 Hook 提供调试标识，普通组件中调用不会报错，但无任何效果；
2. **避免过度使用**：仅在自定义 Hook 逻辑较复杂、需要区分多个实例时使用，简单 Hook（如仅封装 useState）无需添加，避免冗余；
3. **生产环境无影响**：React 会自动忽略生产环境下的 useDebugValue 调用，无需手动移除，也不会增加生产包体积；
4. **格式化函数的参数**：formatFn 的参数是 useDebugValue 的第一个参数（value），而非自定义 Hook 的返回值，需注意参数对应关系；
5. **不要依赖调试值做逻辑判断**：调试值仅用于展示，不能作为业务逻辑的判断依据（比如不能通过 useDebugValue 的值来控制组件渲染）；
6. **兼容性**：useDebugValue 从 React 16.8.0 开始支持（与其他 Hook 同步），需确保项目的 React 版本≥16.8.0；
7. **避免敏感信息暴露**：调试值会显示在开发者工具中，不要将密码、token 等敏感信息作为调试值，防止泄露。
