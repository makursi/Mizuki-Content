---
title: useTransition
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
# useTransition

`useTransition` 是 React 18 中引入的一个内置 Hook，它允许你标记某些状态更新为“非紧急更新”（过渡更新），让 React 优先处理紧急任务（如输入、点击、滚动等），待主线程空闲后再执行这些非紧急更新，从而避免因耗时的状态更新阻塞页面交互，提升应用的响应性。其核心设计目标是区分“紧急更新”和“非紧急更新”，优化用户交互体验。

# useTransition 的核心作用

1. **优先级调度状态更新**：将非紧急的状态更新标记为过渡任务，React 会低优先级处理这类更新，确保高优先级的用户交互（如输入框打字、按钮点击）不被阻塞，页面始终保持可响应。
2. **避免不必要的加载态/卡顿**：在处理大量数据渲染、复杂计算后的状态更新时，使用 `useTransition` 可以让页面不出现“冻结”，同时可通过 `isPending` 状态优雅展示加载提示，而非直接阻塞交互。
3. **协调并发更新**：作为 React 并发特性的核心 Hook 之一，它支持在并发渲染模式下，让多个更新任务协调执行，避免单一更新占用全部主线程资源。

# useTransition 的使用方法

### 基本语法
```jsx
import { useTransition } from 'react';

function Component() {
  // startTransition：用于包裹非紧急状态更新的函数
  // isPending：布尔值，标记过渡更新是否正在进行中
  const [isPending, startTransition] = useTransition({
    // 可选：设置超时时间（毫秒），若超时则强制执行更新
    timeoutMs: 1000 
  });

  // 触发非紧急更新
  const handleNonUrgentUpdate = () => {
    startTransition(() => {
      // 这里的状态更新会被标记为非紧急更新
      setSomeState(newValue);
    });
  };

  // 渲染逻辑
  return (
    <div>
      {isPending && <div>加载中...</div>}
      <button onClick={handleNonUrgentUpdate}>触发非紧急更新</button>
    </div>
  );
}
```

### 关键参数与返回值
| 项                | 类型                | 说明                                                                 |
|-------------------|---------------------|----------------------------------------------------------------------|
| `timeoutMs`（可选） | number              | 过渡更新的超时时间，默认 1000ms；若到时间未执行，React 会强制执行该更新 |
| `isPending`       | boolean             | 过渡更新的状态：`true` 表示更新正在处理，`false` 表示更新完成/未开始    |
| `startTransition` | (callback) => void  | 包裹非紧急状态更新的回调函数，回调内的 `setState` 会被标记为低优先级   |

### 使用核心规则
1. `startTransition` 回调内的代码**不会被立即执行**，React 会在主线程空闲时调度执行；
2. 回调内仅应包含**状态更新逻辑**，不应执行副作用（如网络请求、DOM 操作），副作用需放在 `useEffect` 等 Hook 中；
3. `useTransition` 仅影响其包裹的状态更新的优先级，不影响组件渲染本身的优先级；
4. 紧急更新（如输入框的 `value` 绑定）不应放在 `startTransition` 内，否则会导致输入延迟。

# useTransition 的使用案例

### 案例1：大数据列表筛选（避免筛选时页面卡顿）
场景：用户输入关键词筛选包含 1 万条数据的列表，筛选后渲染结果，若直接更新状态会导致输入卡顿。
```jsx
import { useState, useTransition } from 'react';

function BigListFilter() {
  // 紧急状态：输入框的值（必须实时响应）
  const [inputValue, setInputValue] = useState('');
  // 非紧急状态：筛选后的列表数据
  const [filteredList, setFilteredList] = useState([]);
  const [isPending, startTransition] = useTransition();

  // 模拟 1 万条原始数据
  const originalList = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`);

  // 处理输入变化（紧急更新）
  const handleInputChange = (e) => {
    const value = e.target.value;
    setInputValue(value); // 紧急更新，实时响应输入

    // 非紧急更新：筛选列表（耗时操作，标记为过渡更新）
    startTransition(() => {
      const filtered = originalList.filter(item => 
        item.toLowerCase().includes(value.toLowerCase())
      );
      setFilteredList(filtered);
    });
  };

  return (
    <div>
      <input
        type="text"
        value={inputValue}
        onChange={handleInputChange}
        placeholder="筛选列表..."
      />
      {isPending && <div style={{ color: 'blue' }}>筛选中，请稍候...</div>}
      <ul>
        {filteredList.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 案例2：表单提交后跳转（避免提交后页面冻结）
场景：表单提交后需要更新多个状态并跳转页面，标记状态更新为过渡更新，避免阻塞提交按钮的交互反馈。
```jsx
import { useState, useTransition } from 'react';

function FormWithTransition() {
  const [formData, setFormData] = useState({ name: '', age: '' });
  const [isSubmitted, setIsSubmitted] = useState(false);
  const [isPending, startTransition] = useTransition({ timeoutMs: 1500 });

  const handleSubmit = (e) => {
    e.preventDefault();
    // 紧急：先标记提交中，给用户反馈
    setIsSubmitted(true);

    // 非紧急：处理提交后的状态更新（如清理表单、跳转前准备）
    startTransition(() => {
      // 模拟耗时的状态处理
      setFormData({ name: '', age: '' });
      // 可结合 react-router 实现跳转：navigate('/success')
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>姓名：</label>
        <input
          type="text"
          value={formData.name}
          onChange={(e) => setFormData({ ...formData, name: e.target.value })}
        />
      </div>
      <div>
        <label>年龄：</label>
        <input
          type="number"
          value={formData.age}
          onChange={(e) => setFormData({ ...formData, age: e.target.value })}
        />
      </div>
      <button type="submit" disabled={isPending}>
        {isPending ? '提交中...' : '提交'}
      </button>
      {isSubmitted && !isPending && <div>提交成功！</div>}
    </form>
  );
}
```

# useTransition 的使用注意事项

1. **区分紧急/非紧急更新**：
   - 紧急更新（输入框、下拉选择、按钮点击的即时反馈）不应放入 `startTransition`，否则会导致交互延迟；
   - 非紧急更新（大数据渲染、复杂计算后的状态更新、页面跳转前的准备）适合用 `startTransition` 包裹。

2. **不要依赖 `startTransition` 的执行时机**：
   - 回调内的代码执行时机由 React 调度，无法保证同步执行，因此不能在回调内写“依赖执行顺序”的逻辑（如先更新 A 再读取 A 的值）。

3. **与 `useDeferredValue` 的区别**：
   - `useTransition` 标记“状态更新”的优先级，适用于主动触发的状态变更；
   - `useDeferredValue` 衍生一个低优先级的“延迟值”，适用于被动响应其他状态变化的场景；
   - 两者可配合使用，但不要重复包裹同一更新（避免过度调度）。

4. **并发模式相关限制**：
   - `useTransition` 仅在 React 18+ 的并发渲染模式下生效（默认开启），低版本 React 中使用会报错；
   - 服务端渲染（SSR）中 `useTransition` 不生效，需在客户端 hydration 完成后使用。

5. **避免滥用**：
   - 对于简单、快速的状态更新（如切换弹窗显示/隐藏），无需使用 `useTransition`，反而会增加调度开销；
   - 仅在“更新会导致页面卡顿/阻塞交互”时使用。

6. **错误处理**：
   - `startTransition` 回调内的错误不会被 React 捕获，需自行添加 `try/catch` 处理异常；
   - 若超时时间过短（如 `timeoutMs: 100`），可能导致过渡更新被强制执行，仍出现短暂卡顿，需根据业务场景合理设置。

7. **与 Suspense 配合**：
   - `useTransition` 可与 Suspense 结合使用，在过渡更新期间展示 fallback 界面（如加载骨架屏），提升用户体验；
   - 示例：`{isPending ? <Skeleton /> : <Content />}`。
