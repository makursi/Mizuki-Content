---
title: React传送组件
published: 2026-05-03
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](../react/reactcute.webp)

# React传送组件（Portal）的官方定义
React Portal（传送组件）是React官方提供的一种高级特性，其核心定义为：**Portal 提供了一种将子节点渲染到存在于父组件 DOM 层次结构之外的 DOM 节点的方式**。从官方文档的表述来看，Portal 并不打破 React 组件的父子组件关系和上下文传递机制，仅仅是改变了渲染的 DOM 挂载目标，组件的事件冒泡等行为依然遵循 React 的组件树结构，而非 DOM 树结构。

# 传送组件的核心作用
1. **突破父组件DOM约束**：解决父组件存在 `overflow: hidden`、`z-index` 层级限制、`position` 定位约束等场景下，子组件（如弹窗、模态框、下拉菜单、提示框等）无法正常展示的问题。例如模态框需要脱离当前组件的 DOM 层级，挂载到 `body` 下以避免样式隔离或层级覆盖问题。
2. **保持React组件模型完整性**：尽管渲染到外部 DOM 节点，但 Portal 内的组件依然能继承父组件的 props、context、状态（state）和事件处理逻辑，不会因为 DOM 挂载位置的改变而割裂 React 组件的数据流和交互逻辑。
3. **统一事件处理逻辑**：Portal 内触发的事件会按照 React 组件树向上冒泡，而非 DOM 树结构，开发者无需为外部 DOM 节点单独绑定事件，保持事件处理的一致性。

# 传送组件的使用方法
### 1. 基础使用步骤（基于React + TypeScript）
#### 步骤1：创建挂载的目标DOM节点
通常在 `public/index.html` 中提前定义，或在组件内动态创建：
```html
<!-- public/index.html -->
<body>
  <div id="root"></div>
  <!-- Portal挂载目标节点 -->
  <div id="portal-root"></div>
</body>
```

#### 步骤2：使用 ReactDOM.createPortal 创建Portal
React 18 中需结合 `createRoot`，React 17 及以下使用 `ReactDOM.createPortal` 直接挂载：

```tsx
import React, { ReactNode } from 'react';
import { createPortal } from 'react-dom';

// 定义Portal组件（TypeScript类型约束）
interface PortalProps {
  children: ReactNode;
  container?: HTMLElement; // 可选：自定义挂载容器
}

const Portal: React.FC<PortalProps> = ({
  children,
  container = document.getElementById('portal-root')!,
}) => {
  // 校验容器是否存在，避免报错
  if (!container) return null;
  return createPortal(children, container);
};

// 业务组件中使用Portal
const Modal: React.FC = () => {
  return (
    <Portal>
      <div className="modal">
        <h3>这是一个Portal渲染的模态框</h3>
        <button>关闭</button>
      </div>
    </Portal>
  );
};

export default Modal;
```

### 2. 动态创建挂载容器（进阶用法）
若不想提前在 HTML 中定义节点，可在组件挂载时动态创建：
```tsx
import React, { ReactNode, useEffect, useRef, useState } from 'react';
import { createPortal } from 'react-dom';

interface DynamicPortalProps {
  children: ReactNode;
}

const DynamicPortal: React.FC<DynamicPortalProps> = ({ children }) => {
  const [container, setContainer] = useState<HTMLElement | null>(null);
  const containerRef = useRef<HTMLElement | null>(null);

  useEffect(() => {
    // 动态创建DOM节点
    const newContainer = document.createElement('div');
    newContainer.id = 'dynamic-portal-root';
    document.body.appendChild(newContainer);
    containerRef.current = newContainer;
    setContainer(newContainer);

    // 组件卸载时清理节点
    return () => {
      if (newContainer) {
        document.body.removeChild(newContainer);
      }
    };
  }, []);

  if (!container) return null;
  return createPortal(children, container);
};

export default DynamicPortal;
```

### 3. 结合状态管理使用
Portal 内的组件可正常使用父组件的状态和方法：
```tsx
import React, { useState } from 'react';
import Portal from './Portal';

const ParentComponent: React.FC = () => {
  const [isModalOpen, setIsModalOpen] = useState(false);

  const handleOpenModal = () => setIsModalOpen(true);
  const handleCloseModal = () => setIsModalOpen(false);

  return (
    <div className="parent">
      <button onClick={handleOpenModal}>打开模态框</button>
      {isModalOpen && (
        <Portal>
          <div className="modal-overlay" onClick={handleCloseModal}>
            <div className="modal-content" onClick={(e) => e.stopPropagation()}>
              <h3>Portal模态框</h3>
              <button onClick={handleCloseModal}>关闭</button>
            </div>
          </div>
        </Portal>
      )}
    </div>
  );
};

export default ParentComponent;
```

# 传送组件的使用案例
### 案例1：全局提示框（Toast）
适用于需要在页面任意位置触发、渲染到 `body` 下的轻量级提示：
```tsx
import React, { useState, useEffect } from 'react';
import { createPortal } from 'react-dom';
import './Toast.css';

interface ToastProps {
  message: string;
  duration?: number; // 自动关闭时长
  onClose: () => void;
}

const Toast: React.FC<ToastProps> = ({ message, duration = 2000, onClose }) => {
  const container = document.getElementById('portal-root') || document.body;

  useEffect(() => {
    const timer = setTimeout(() => {
      onClose();
    }, duration);
    return () => clearTimeout(timer);
  }, [duration, onClose]);

  return createPortal(
    <div className="toast">
      {message}
    </div>,
    container
  );
};

// 父组件使用
const ToastDemo: React.FC = () => {
  const [showToast, setShowToast] = useState(false);
  const [toastMessage, setToastMessage] = useState('');

  const show = (msg: string) => {
    setToastMessage(msg);
    setShowToast(true);
  };

  return (
    <div>
      <button onClick={() => show('操作成功！')}>显示提示</button>
      {showToast && <Toast message={toastMessage} onClose={() => setShowToast(false)} />}
    </div>
  );
};

export default ToastDemo;
```

### 案例2：下拉菜单（Dropdown）
解决下拉菜单被父组件 `overflow: hidden` 截断的问题：
```tsx
import React, { useState } from 'react';
import { createPortal } from 'react-dom';
import './Dropdown.css';

const Dropdown: React.FC = () => {
  const [isOpen, setIsOpen] = useState(false);
  const container = document.getElementById('portal-root') || document.body;

  const options = ['选项1', '选项2', '选项3'];

  return (
    <div className="dropdown-trigger" onClick={() => setIsOpen(!isOpen)}>
      点击展开下拉菜单
      {isOpen &&
        createPortal(
          <div className="dropdown-menu">
            {options.map((item, index) => (
              <div key={index} className="dropdown-item">
                {item}
              </div>
            ))}
          </div>,
          container
        )}
    </div>
  );
};

export default Dropdown;
```

# 传送组件的使用注意事项
1. **DOM 节点存在性校验**：使用 `createPortal` 时必须确保目标容器已存在，否则会抛出错误。建议通过非空断言（`!`）或条件判断避免空值，如 `document.getElementById('portal-root')!` 或 `if (!container) return null`。
2. **组件卸载时清理 DOM 节点**：若动态创建了挂载容器（如 `document.createElement`），必须在 `useEffect` 的清理函数中移除节点，否则会导致 DOM 冗余，引发内存泄漏。
3. **事件冒泡规则**：Portal 内的事件会沿 React 组件树冒泡，而非 DOM 树。例如，Portal 挂载到 `body` 下，但点击 Portal 内的按钮，父组件的点击事件依然会触发，若需阻止可使用 `e.stopPropagation()`。
4. **样式隔离与层级**：Portal 渲染的元素脱离了原组件的 DOM 层级，需注意 `z-index` 层级管理，避免被其他元素覆盖；同时，原组件的 CSS 模块化样式可能无法生效，需使用全局样式、CSS-in-JS 或 `:global` 修饰符。
5. **服务端渲染（SSR）注意事项**：在 Next.js 等 SSR 框架中，`document`/`window` 对象在服务端不存在，需通过 `typeof window !== 'undefined'` 判断环境，避免服务端报错：
   ```tsx
   useEffect(() => {
     if (typeof window !== 'undefined') {
       const container = document.getElementById('portal-root');
       setContainer(container);
     }
   }, []);
   ```
6. **避免过度使用**：Portal 仅用于解决 DOM 层级约束问题，若无需脱离父组件 DOM 层级，应优先使用普通组件渲染，避免增加复杂度。
7. **TypeScript 类型安全**：定义 Portal 组件时，需为 `container` 增加 `HTMLElement` 类型约束，避免传入非 DOM 节点类型；同时约束 `children` 为 `ReactNode`，确保传入合法的 React 子元素。
