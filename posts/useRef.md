---
title: useRef
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
# useRef

useRef 是 React 提供的一个 Hook，它能在组件的整个生命周期内保存一个可变的值，这个值的更新不会触发组件的重新渲染；同时它也可以用来获取 DOM 元素或 React 组件实例。

从官方定义来看，useRef 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（initialValue），该 ref 对象在组件的整个生命周期内保持不变。

# useRef 的核心作用

1. **保存可变值且不触发重渲染**：与 useState 不同，修改 useRef 创建的 ref 对象的 `.current` 属性时，不会引发组件重新渲染，适合存储那些不需要参与视图更新的临时状态、计时器 ID、请求实例等。
2. **访问 DOM 元素/组件实例**：通过将 ref 对象绑定到 DOM 元素的 `ref` 属性上，能直接获取到该 DOM 元素的真实节点，进而操作 DOM（如聚焦输入框、获取元素尺寸）；也可用于获取类组件实例（函数组件无实例，需配合 forwardRef + useImperativeHandle）。
3. **跨渲染周期保存数据**：组件每次重新渲染时，useState 创建的状态会重新计算（除非依赖不变），而 useRef 的 `.current` 会保留上一次渲染周期的值，可用于记录上一次的状态、props 等。

# useRef 的使用方法

### 1. 基本语法
```jsx
import { useRef } from 'react';

function MyComponent() {
  // 初始化 ref 对象，initialValue 可以是任意类型（null、数字、对象、DOM 节点等）
  const refContainer = useRef(initialValue);

  // 读取值
  const currentValue = refContainer.current;

  // 修改值（不会触发组件重渲染）
  refContainer.current = newValue;

  return <div ref={refContainer}>示例</div>;
}
```

### 2. 核心使用场景拆解
#### 场景1：保存不触发重渲染的可变值
```jsx
import { useRef, useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const countRef = useRef(0); // 保存计数，修改不触发重渲染

  const handleIncrement = () => {
    // 状态更新触发重渲染
    setCount(prev => prev + 1);
    // ref 更新不触发重渲染
    countRef.current += 1;
    console.log('ref 计数：', countRef.current); // 实时更新
  };

  return (
    <div>
      <p>状态计数：{count}</p>
      <button onClick={handleIncrement}>+1</button>
    </div>
  );
}
```

#### 场景2：访问 DOM 元素
```jsx
import { useRef, useEffect } from 'react';

function InputFocus() {
  // 初始化 ref 为 null，绑定到 input 元素后，current 指向 input DOM 节点
  const inputRef = useRef(null);

  useEffect(() => {
    // 组件挂载后，聚焦输入框
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} type="text" placeholder="自动聚焦" />;
}
```

#### 场景3：跨渲染周期记录数据
```jsx
import { useRef, useState, useEffect } from 'react';

function PreviousValue() {
  const [value, setValue] = useState('');
  // 记录上一次的 value
  const prevValueRef = useRef('');

  useEffect(() => {
    // 每次 value 更新后，更新 ref 的值
    prevValueRef.current = value;
  }, [value]);

  return (
    <div>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="输入内容"
      />
      <p>当前值：{value}</p>
      <p>上一次值：{prevValueRef.current}</p>
    </div>
  );
}
```

# useRef 的使用案例

### 案例1：实现计时器（保存计时器 ID，避免重复创建）
```jsx
import { useRef, useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);
  // 保存计时器 ID，用于清除计时器
  const timerRef = useRef(null);

  const startTimer = () => {
    // 避免重复点击创建多个计时器
    if (!timerRef.current) {
      timerRef.current = setInterval(() => {
        setSeconds(prev => prev + 1);
      }, 1000);
    }
  };

  const stopTimer = () => {
    // 清除计时器，并重置 ref
    clearInterval(timerRef.current);
    timerRef.current = null;
  };

  // 组件卸载时清除计时器，防止内存泄漏
  useEffect(() => {
    return () => clearInterval(timerRef.current);
  }, []);

  return (
    <div>
      <p>计时：{seconds} 秒</p>
      <button onClick={startTimer}>开始</button>
      <button onClick={stopTimer}>停止</button>
    </div>
  );
}
```

### 案例2：获取 DOM 元素尺寸（动态获取宽高）
```jsx
import { useRef, useEffect, useState } from 'react';

function ElementSize() {
  const boxRef = useRef(null);
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    const updateSize = () => {
      if (boxRef.current) {
        const { clientWidth, clientHeight } = boxRef.current;
        setSize({ width: clientWidth, height: clientHeight });
      }
    };

    // 初始获取尺寸
    updateSize();
    // 监听窗口大小变化，更新尺寸
    window.addEventListener('resize', updateSize);

    // 组件卸载时移除监听
    return () => window.removeEventListener('resize', updateSize);
  }, []);

  return (
    <div ref={boxRef} style={{ width: '50%', height: 200, border: '1px solid #000' }}>
      <p>宽度：{size.width}px</p>
      <p>高度：{size.height}px</p>
    </div>
  );
}
```

# useRef 的使用注意事项

1. **修改 ref 不触发重渲染**：useRef 的核心特性是修改 `.current` 不会引发组件重新渲染，若依赖该值更新视图，需配合 useState 或 useReducer 一起使用。
2. **ref 对象的不变性**：useRef 返回的 ref 对象在组件整个生命周期内是同一个引用（不会重新创建），仅 `.current` 属性可变；若每次渲染都需要新的 ref，需用 `useMemo(() => ({ current: initialValue }), [deps])` 替代。
3. **避免在渲染阶段修改 ref**：React 渲染阶段（函数组件执行过程中）应保持纯函数特性，修改 ref.current 可能导致不可预测的行为，建议在事件处理函数、useEffect/useLayoutEffect 中修改。
4. **函数组件无法直接通过 ref 获取实例**：函数组件没有实例，若需向父组件暴露函数组件的内部方法/属性，需配合 `React.forwardRef` 和 `useImperativeHandle`：
   ```jsx
   import { forwardRef, useImperativeHandle, useRef } from 'react';

   // 子组件
   const ChildInput = forwardRef((props, ref) => {
     const inputRef = useRef(null);
     
     // 向父组件暴露指定方法
     useImperativeHandle(ref, () => ({
       focus: () => {
         inputRef.current.focus();
       }
     }));

     return <input ref={inputRef} type="text" />;
   });

   // 父组件
   function Parent() {
     const childRef = useRef(null);

     const focusChildInput = () => {
       childRef.current.focus();
     };

     return (
       <div>
         <ChildInput ref={childRef} />
         <button onClick={focusChildInput}>聚焦子组件输入框</button>
       </div>
     );
   }
   ```
5. **避免过度使用 ref 操作 DOM**：React 提倡声明式编程，优先通过状态驱动视图，仅在无法通过状态实现时（如聚焦、滚动、测量尺寸）使用 ref 操作 DOM。
6. **初始化值的时机**：useRef 的 initialValue 仅在组件首次渲染时生效，后续渲染会忽略该值；若需根据 props/state 动态更新 ref 初始值，需在 useEffect 中手动修改 `.current`。
