---
title: useLayoutEffect
published: 2026-05-02
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](./react/reactcute.webp)

---
# useLayoutEffect

useLayoutEffect 是 React 提供的一个生命周期 Hook，它与 useEffect 的执行时机不同——**useLayoutEffect 会在所有 DOM 变更之后、浏览器绘制（paint）之前同步执行**，而 useEffect 是在浏览器完成绘制后异步执行。

从官方定义来看，它的签名与 useEffect 完全一致，仅执行阶段存在差异：`useLayoutEffect(effect, deps?)`，其中 effect 是包含副作用逻辑的函数（可返回清理函数），deps 是依赖项数组。

# useLayoutEffect 的核心作用

1. **同步读取并修改 DOM 布局**：由于执行时机在 DOM 变更后、绘制前，可确保读取到最新的 DOM 布局信息（如元素尺寸、位置），并在浏览器绘制前完成 DOM 调整，避免页面出现视觉闪烁（比如避免因先渲染再调整位置导致的“跳变”）。
2. **替代类组件的 componentDidMount / componentDidUpdate 同步逻辑**：对于需要在 DOM 更新后立即同步执行的操作（而非异步），useLayoutEffect 是更贴合的选择，弥补了 useEffect 异步执行无法同步操作 DOM 布局的不足。
3. **清理同步副作用**：其返回的清理函数会在组件卸载前、或下一次 effect 执行前同步执行，确保布局相关的清理逻辑及时完成。

# useLayoutEffect 的使用方法

### 基本语法
```jsx
import { useLayoutEffect, useState } from 'react';

function MyComponent() {
  const [width, setWidth] = useState(0);

  useLayoutEffect(() => {
    // 1. 读取DOM布局信息（同步，确保拿到最新值）
    const element = document.getElementById('target');
    const currentWidth = element.offsetWidth;
    
    // 2. 修改状态或DOM，此时浏览器尚未绘制，不会出现闪烁
    setWidth(currentWidth);

    // 3. 清理函数（可选）
    return () => {
      // 清理与布局相关的副作用（如事件监听、定时器）
    };
  }, []); // 依赖项数组：空数组仅执行一次（挂载时），非空则依赖变化时执行

  return <div id="target">目标元素</div>;
}
```

### 关键使用规则
1. **依赖项数组**：与 useEffect 一致，若省略则每次渲染都执行；若为空数组则仅挂载时执行；若包含变量，则变量变化时执行。
2. **禁止阻塞渲染**：useLayoutEffect 是同步执行的，若其中包含耗时操作（如复杂计算、网络请求），会阻塞浏览器绘制，导致页面卡顿——**仅用于布局相关的同步操作**，非布局逻辑优先使用 useEffect。
3. **服务端渲染（SSR）注意事项**：在服务端渲染时，useLayoutEffect 不会执行（因为服务端无 DOM 绘制阶段），若代码中依赖其执行的逻辑可能导致报错，可通过 `typeof window !== 'undefined'` 包裹，或优先使用 useEffect。

# useLayoutEffect 的使用案例

### 案例1：避免弹窗/元素位置闪烁
场景：根据按钮位置动态计算弹窗的定位，若用 useEffect 会先渲染弹窗到默认位置，再调整，导致闪烁；用 useLayoutEffect 可在绘制前完成定位。
```jsx
import { useLayoutEffect, useState, useRef } from 'react';

function PopupButton() {
  const [showPopup, setShowPopup] = useState(false);
  const [popupStyle, setPopupStyle] = useState({});
  const buttonRef = useRef(null);

  useLayoutEffect(() => {
    if (showPopup && buttonRef.current) {
      // 读取按钮的DOM位置（同步获取最新布局）
      const buttonRect = buttonRef.current.getBoundingClientRect();
      // 计算弹窗位置，在绘制前设置样式
      setPopupStyle({
        position: 'absolute',
        top: `${buttonRect.bottom + 5}px`,
        left: `${buttonRect.left}px`,
        width: '200px',
        padding: '10px',
        background: '#fff',
        border: '1px solid #ccc'
      });
    }
  }, [showPopup]);

  return (
    <div style={{ position: 'relative', marginTop: '50px' }}>
      <button ref={buttonRef} onClick={() => setShowPopup(!showPopup)}>
        打开弹窗
      </button>
      {showPopup && <div style={popupStyle}>这是无闪烁的弹窗</div>}
    </div>
  );
}
```

### 案例2：同步更新滚动位置
场景：组件渲染后需要立即调整滚动条位置，确保滚动到指定区域，避免先渲染再滚动导致的视觉跳动。
```jsx
import { useLayoutEffect, useRef } from 'react';

function ScrollToTarget() {
  const scrollContainerRef = useRef(null);
  const targetRef = useRef(null);

  useLayoutEffect(() => {
    if (scrollContainerRef.current && targetRef.current) {
      // 同步滚动到目标元素，绘制前完成，无跳动
      targetRef.current.scrollIntoView({ behavior: 'instant' });
    }
  }, []);

  return (
    <div ref={scrollContainerRef} style={{ height: '300px', overflow: 'auto' }}>
      <div style={{ height: '500px' }}>占位内容</div>
      <div ref={targetRef} style={{ color: 'red' }}>需要滚动到的目标</div>
      <div style={{ height: '200px' }}>更多内容</div>
    </div>
  );
}
```

# useLayoutEffect 的使用注意事项

1. **执行时机优先级**：
   - 执行顺序：组件渲染 → DOM 变更 → useLayoutEffect 执行 → 浏览器绘制 → useEffect 执行。
   - 若多个组件都有 useLayoutEffect，会按组件挂载顺序依次执行，全部执行完后才会绘制页面。

2. **性能风险**：
   - 禁止在 useLayoutEffect 中执行耗时操作（如循环计算、大量 DOM 操作），同步执行会阻塞渲染，导致页面响应变慢。
   - 优先使用 useEffect：仅当需要同步读取/修改 DOM 布局、避免视觉闪烁时，才使用 useLayoutEffect，其余场景（如数据请求、事件监听）用 useEffect。

3. **与 useEffect 的选择原则**：
   - 若操作不依赖 DOM 布局信息，或不需要同步执行 → 用 useEffect（异步，不阻塞绘制）。
   - 若操作依赖 DOM 布局且需要在绘制前完成 → 用 useLayoutEffect（同步，避免闪烁）。

4. **清理函数执行时机**：
   - useLayoutEffect 的清理函数会在**组件卸载前**、或**下一次 useLayoutEffect 执行前**同步执行，确保布局相关的副作用及时清理（如移除布局监听的事件）。

5. **严格模式与开发环境**：
   - React 严格模式下，开发环境会触发两次 useLayoutEffect（挂载 → 模拟卸载 → 重新挂载），需确保清理函数能正确处理重复执行的情况，避免内存泄漏或逻辑错误。

6. **服务端渲染（SSR）兼容**：
   - Next.js、Remix 等 SSR 框架中，服务端渲染时 useLayoutEffect 不会执行，若直接访问 window/document 会报错，需做兼容处理：
     ```jsx
     useLayoutEffect(() => {
       if (typeof window === 'undefined') return;
       // 仅在客户端执行的布局逻辑
     }, []);
     ```
   - 若无需同步执行，可直接替换为 useEffect 以适配 SSR。
---
