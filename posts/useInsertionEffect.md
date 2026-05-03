---
title: useInsertionEffect
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
# useInsertionEffect

useInsertionEffect 是 React 官方提供的一个专门为 CSS-in-JS 库设计的 Hook，它在 DOM 变更之后、布局副作用（如 useLayoutEffect）触发之前同步执行，其核心设计目标是解决 CSS-in-JS 库在注入样式时可能出现的布局抖动（layout thrashing）问题，是 React 18 中新增的 Hook，属于 useEffect 家族的特殊变体。

# useInsertionEffect 的核心作用

1. **样式注入的时机优化**：在 React 执行 DOM 变更后、计算布局和绘制之前插入样式，避免因样式注入时机过晚导致浏览器重复计算布局（布局抖动），保证样式在布局生效前已存在，确保视觉一致性。
2. **专为 CSS-in-JS 场景设计**：解决传统 useEffect/useLayoutEffect 注入样式时，因执行时机问题导致的闪屏、样式延迟应用等问题，是 CSS-in-JS 库的专属优化 Hook。
3. **同步执行特性**：与 useLayoutEffect 类似同步执行，但执行时机更早（DOM 变更后 → useInsertionEffect → 布局计算 → useLayoutEffect → 浏览器绘制 → useEffect），避免样式注入干扰布局计算。
4. **限制与边界**：该 Hook 内部**无法访问 DOM 节点的布局信息**（如 offsetWidth、scrollTop 等），也不能触发浏览器重绘/重排，仅用于样式注入，若尝试访问布局信息会抛出警告。

# useInsertionEffect 的使用方法

### 基本语法
```jsx
import { useInsertionEffect } from 'react';

function MyComponent() {
  useInsertionEffect(() => {
    // 执行样式注入逻辑（仅同步操作，无布局读取）
    // 副作用清理函数（可选）
    return () => {
      // 清理注入的样式（如移除 style 标签）
    };
  }, [依赖项数组]); // 依赖项变化时重新执行

  return <div>组件内容</div>;
}
```

### 核心使用规则
1. **依赖项数组**：与 useEffect/useLayoutEffect 一致，为空数组时仅在组件挂载/卸载时执行，依赖项变化时重新执行。
2. **无布局读取**：禁止在回调中访问 DOM 节点的布局属性（如 clientHeight），否则会触发 React 警告。
3. **清理逻辑**：若注入了 style 标签等 DOM 元素，需在清理函数中移除，避免内存泄漏。
4. **使用场景限制**：仅用于 CSS-in-JS 库的样式注入，普通业务逻辑优先使用 useEffect/useLayoutEffect。

# useInsertionEffect 的使用案例

### 案例1：极简 CSS-in-JS 样式注入
实现一个简单的 CSS-in-JS 函数，通过 useInsertionEffect 注入样式，避免布局抖动：
```jsx
import { useInsertionEffect } from 'react';

// 简易 CSS-in-JS 样式注入函数
function useInjectStyles(styles) {
  useInsertionEffect(() => {
    // 创建 style 标签
    const styleTag = document.createElement('style');
    styleTag.innerHTML = styles;
    // 插入到 head 中（DOM 变更后、布局计算前）
    document.head.appendChild(styleTag);

    // 清理函数：卸载时移除 style 标签
    return () => {
      document.head.removeChild(styleTag);
    };
  }, [styles]); // 样式变化时重新注入
}

// 业务组件
function Button() {
  // 注入按钮样式
  useInjectStyles(`
    .custom-button {
      padding: 8px 16px;
      background: #165DFF;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    .custom-button:hover {
      background: #0040C9;
    }
  `);

  return <button className="custom-button">点击按钮</button>;
}

export default Button;
```

### 案例2：结合组件状态动态注入样式
根据组件状态动态生成样式，通过 useInsertionEffect 实时注入：
```jsx
import { useState, useInsertionEffect } from 'react';

function DynamicStyleComponent() {
  const [isDark, setIsDark] = useState(false);

  // 动态生成样式
  const dynamicStyles = `
    .theme-container {
      width: 300px;
      height: 200px;
      padding: 20px;
      background: ${isDark ? '#1f1f1f' : '#ffffff'};
      color: ${isDark ? '#ffffff' : '#1f1f1f'};
      border: 1px solid #e0e0e0;
    }
  `;

  // 注入动态样式
  useInsertionEffect(() => {
    const styleTag = document.createElement('style');
    styleTag.id = 'dynamic-theme-style';
    styleTag.innerHTML = dynamicStyles;
    
    // 先移除旧样式（避免重复），再插入新样式
    const oldStyleTag = document.getElementById('dynamic-theme-style');
    if (oldStyleTag) {
      document.head.removeChild(oldStyleTag);
    }
    document.head.appendChild(styleTag);

    return () => {
      const styleTag = document.getElementById('dynamic-theme-style');
      if (styleTag) {
        document.head.removeChild(styleTag);
      }
    };
  }, [dynamicStyles]); // 依赖动态样式，状态变化时重新注入

  return (
    <div>
      <button onClick={() => setIsDark(!isDark)}>
        切换{isDark ? '浅色' : '深色'}模式
      </button>
      <div className="theme-container">
        动态主题容器：{isDark ? '深色模式' : '浅色模式'}
      </div>
    </div>
  );
}

export default DynamicStyleComponent;
```

# useInsertionEffect 的使用注意事项

1. **使用场景严格限制**：仅用于 CSS-in-JS 库的样式注入，普通业务逻辑（如数据请求、DOM 操作、布局计算）禁止使用，应优先选择 useEffect（异步，不阻塞绘制）或 useLayoutEffect（同步，布局计算后）。
2. **禁止访问布局信息**：回调函数中不能读取 DOM 节点的布局属性（如 offsetTop、getBoundingClientRect() 等），否则会触发 React 警告，且可能导致布局抖动。
3. **执行时机优先级**：执行顺序为：
   - 组件 render → DOM 变更 → useInsertionEffect 执行 → 布局计算 → useLayoutEffect 执行 → 浏览器绘制 → useEffect 执行。
   需明确该时机与其他 Effect 的差异，避免逻辑顺序错误。
4. **服务端渲染 (SSR) 注意事项**：在 SSR 环境中，useInsertionEffect 不会在服务端执行（仅在客户端 hydration 阶段执行），因此 CSS-in-JS 库需额外处理服务端样式注入，避免客户端 hydration 时样式闪烁。
5. **清理函数必须完整**：若注入了 DOM 元素（如 style 标签），必须在清理函数中移除，否则会导致重复注入、内存泄漏或样式冲突。
6. **依赖项需精准**：依赖项数组需包含所有影响样式生成的变量，避免因依赖缺失导致样式不更新，或因依赖冗余导致不必要的重新注入。
7. **React 版本要求**：useInsertionEffect 是 React 18 新增 Hook，使用前需确保项目 React 版本 ≥ 18.0.0，否则会报错。
8. **避免复杂逻辑**：回调函数应仅包含样式注入的极简逻辑，禁止执行复杂计算、异步操作等，否则会阻塞布局计算，影响页面性能。
