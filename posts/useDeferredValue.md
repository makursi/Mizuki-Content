---
title: useDeferredValue
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
# useDeferredValue

useDeferredValue 是 React 提供的一个内置 Hook，用于延迟更新非紧急的状态值。它允许你在渲染优先级更高的内容时，暂时使用旧的、延迟的值，待主线程空闲后再更新为最新值，从而避免高优先级渲染被低优先级的状态更新阻塞，提升应用的交互响应性。

# 核心作用

1. **优先级调度**：将非关键状态的更新延迟到高优先级渲染（如用户输入、动画）完成后执行，保证核心交互的流畅性。
2. **避免不必要的重渲染阻塞**：针对计算成本高、但更新不紧急的状态（如大数据列表过滤、复杂文本渲染），使用延迟值可以防止这些操作阻塞主线程，导致页面卡顿。
3. **与防抖/节流的区别**：useDeferredValue 由 React 调度机制驱动（基于浏览器空闲时段），而非固定时间延迟，更贴合 React 的渲染优先级模型，无需手动设置延迟时间。
4. **自动回退**：如果主线程长期忙碌，延迟值最终仍会更新为最新值，不会永久停留在旧值。

# 使用方法

### 基本语法
```jsx
import { useDeferredValue } from 'react';

function Component() {
  const [value, setValue] = useState('');
  // 延迟更新 deferredValue，直到主线程空闲
  const deferredValue = useDeferredValue(value);
  
  return (
    // 高优先级渲染：输入框（即时响应）
    <input 
      value={value} 
      onChange={(e) => setValue(e.target.value)} 
      placeholder="输入内容..." 
    />
    // 低优先级渲染：使用延迟值，避免阻塞输入
    <ExpensiveComponent value={deferredValue} />
  );
}
```

### 关键参数与返回值
- **参数**：需要延迟的状态值（可以是任意类型：字符串、对象、数组等）。
- **返回值**：延迟更新的“镜像值”——初始阶段与原状态值一致，当原状态更新时，返回值会暂时保留旧值，待 React 调度到空闲时段后，才更新为最新值。

### 配合 memo 优化重渲染
useDeferredValue 本身会触发重渲染，因此通常需要配合 `React.memo`（或 `useMemo`）避免子组件不必要的重渲染：
```jsx
import { useState, useDeferredValue, memo } from 'react';

// 耗时的子组件：仅在 deferredValue 真正变化时重渲染
const ExpensiveList = memo(({ value }) => {
  // 模拟大数据过滤/渲染的耗时操作
  const items = Array.from({ length: 10000 }).map((_, i) => (
    <div key={i}>{value} - {i}</div>
  ));
  return <div>{items}</div>;
});

function SearchBox() {
  const [search, setSearch] = useState('');
  const deferredSearch = useDeferredValue(search);

  return (
    <>
      <input 
        value={search} 
        onChange={(e) => setSearch(e.target.value)} 
      />
      <ExpensiveList value={deferredSearch} />
    </>
  );
}
```

# 使用案例

### 案例1：大数据列表搜索（核心场景）
场景：用户在输入框搜索时，需要过滤并渲染10万条数据的列表，直接渲染会导致输入框卡顿。
```jsx
import { useState, useDeferredValue, memo } from 'react';

// 模拟10万条数据
const bigDataList = Array.from({ length: 100000 }).map((_, i) => ({
  id: i,
  content: `Item ${i} - ${Math.random().toString(36).slice(2)}`
}));

// 耗时的列表组件（memo 优化）
const BigList = memo(({ filterText }) => {
  console.log('列表重渲染：', filterText);
  // 过滤大数据（模拟耗时操作）
  const filteredList = bigDataList.filter(item => 
    item.content.includes(filterText)
  );

  return (
    <div style={{ maxHeight: '500px', overflow: 'auto' }}>
      {filteredList.map(item => (
        <div key={item.id}>{item.content}</div>
      ))}
    </div>
  );
});

export default function DeferredValueDemo() {
  const [inputValue, setInputValue] = useState('');
  // 延迟搜索关键词，优先保证输入框响应
  const deferredInput = useDeferredValue(inputValue);

  return (
    <div>
      <h3>大数据列表搜索</h3>
      <input
        type="text"
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="搜索内容..."
        style={{ padding: '8px', width: '300px' }}
      />
      <BigList filterText={deferredInput} />
    </div>
  );
}
```

### 案例2：实时输入的富文本预览
场景：用户输入富文本内容时，预览区需要解析 markdown 并渲染，解析过程耗时，导致输入卡顿。
```jsx
import { useState, useDeferredValue, memo } from 'react';
import ReactMarkdown from 'react-markdown'; // 第三方 markdown 解析库

// 预览组件（memo 优化）
const MarkdownPreview = memo(({ content }) => {
  // 解析 markdown 是耗时操作
  return (
    <div style={{ border: '1px solid #eee', padding: '16px', marginTop: '10px' }}>
      <ReactMarkdown>{content}</ReactMarkdown>
    </div>
  );
});

export default function RichTextEditor() {
  const [content, setContent] = useState('# 请输入Markdown内容');
  // 延迟预览内容，优先保证输入框流畅
  const deferredContent = useDeferredValue(content);

  return (
    <div>
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        style={{ width: '100%', height: '200px', padding: '8px' }}
      />
      <MarkdownPreview content={deferredContent} />
    </div>
  );
}
```

# 使用注意事项

1. **仅用于非紧急状态**：useDeferredValue 适用于“可以短暂展示旧值”的场景，**不能用于紧急状态**（如表单提交、弹窗显隐、用户权限校验等），否则会导致交互逻辑异常。
2. **必须配合 memo/useMemo**：useDeferredValue 会让组件多一次重渲染（先渲染旧值，再渲染新值），若不配合 memo/useMemo 优化子组件，会失去性能优化的意义，甚至增加额外开销。
3. **不替代防抖/节流**：
   - 防抖/节流：控制状态更新的**频率**（如 500ms 内只更新一次）；
   - useDeferredValue：控制状态更新的**优先级**（空闲时才更新）；
   复杂场景可结合使用（如先节流减少更新次数，再延迟渲染）。
4. **与 useTransition 的区别**：
   - useDeferredValue：延迟“状态值”的更新，适用于从现有状态派生延迟值；
   - useTransition：延迟“状态更新的操作”，适用于主动触发的状态修改（如点击按钮加载数据）；
   核心目标一致（优先级调度），使用场景互补。
5. **不影响服务端渲染**：在服务端渲染（SSR）中，useDeferredValue 会直接返回最新值，不会延迟，避免服务端与客户端渲染结果不一致。
6. **避免依赖延迟值做逻辑判断**：延迟值可能暂时与原状态不一致，若基于延迟值做条件渲染（如 `if (deferredValue) { ... }`），需确保逻辑容许多次渲染的中间状态。
7. **性能边界**：若延迟值对应的渲染操作本身极耗时（如百万级数据渲染），即使延迟也可能阻塞主线程，此时需结合虚拟列表（如 react-window）等更底层的性能优化方案。
