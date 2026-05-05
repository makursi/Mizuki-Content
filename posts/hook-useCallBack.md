---
title: useCallBack
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
# useCallback

useCallback 是 React 中的一个内置 Hook，它的核心设计目的是**缓存函数定义**，避免在组件重新渲染时不必要地创建新的函数实例。

从官方定义来看，useCallback 会返回一个记忆化（memoized）的回调函数，该函数仅在其依赖项数组发生变化时才会更新。

# useCallback 的核心作用

1. **优化组件性能**：当把函数作为 props 传递给子组件（尤其是使用 React.memo 包裹的纯组件）时，若每次父组件渲染都创建新的函数实例，会导致子组件因 props 变化而不必要地重新渲染。useCallback 缓存函数实例，确保只有依赖项变化时函数才更新，从而避免子组件无意义的重渲染。
2. **稳定函数引用**：在依赖于函数引用的场景（如 useEffect、useMemo 的依赖项数组，或自定义 Hook 中），稳定的函数引用能防止这些 Hook 因函数实例变化而触发不必要的执行，保证逻辑执行的准确性和性能。
3. **减少内存开销**：避免组件每次渲染都生成新的函数对象，减少内存分配和垃圾回收的开销，尤其在频繁渲染的组件中效果更明显。

# useCallback 的使用方法

## 基本语法
```javascript
const memoizedCallback = useCallback(
  () => {
    // 回调函数的具体逻辑
    doSomething(a, b);
  },
  [a, b], // 依赖项数组：只有当a或b变化时，才会重新创建回调函数
);
```

### 参数说明
- 第一个参数：需要被缓存的回调函数（可以是任意逻辑的函数，如事件处理、数据处理函数等）。
- 第二个参数：依赖项数组（必传，类似 useEffect 的依赖项）：
  - 数组为空 `[]`：回调函数仅在组件首次渲染时创建，后续渲染始终复用该实例；
  - 数组包含变量/状态：只有当依赖项的值发生变化时，才会重新生成新的函数实例；
  - 不传递依赖项数组：React 不会缓存函数，每次渲染都会创建新的函数（等同于直接定义函数）。

## 典型使用场景示例
### 场景1：传递函数给 memo 包裹的子组件
```jsx
import React, { useState, useCallback, memo } from 'react';

// 子组件：使用memo包裹，仅当props变化时重渲染
const ChildComponent = memo(({ onClick, data }) => {
  console.log('子组件渲染');
  return <button onClick={onClick}>{data}</button>;
});

const ParentComponent = () => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('React');

  // 使用useCallback缓存点击事件函数，依赖项为name
  const handleClick = useCallback(() => {
    console.log(`点击了：${name}`);
  }, [name]);

  return (
    <div>
      <p>计数：{count}</p>
      <button onClick={() => setCount(count + 1)}>增加计数</button>
      <button onClick={() => setName(name + ' Hook')}>修改名称</button>
      {/* 传递缓存后的函数给子组件 */}
      <ChildComponent onClick={handleClick} data={name} />
    </div>
  );
};
```
**说明**：点击“增加计数”时，count 变化但 handleClick 的依赖项 name 未变，因此 handleClick 引用不变，子组件不会重渲染；点击“修改名称”时，name 变化，handleClick 重新创建，子组件因 props 变化重渲染。

### 场景2：作为 useEffect 的依赖项
```jsx
import React, { useState, useCallback, useEffect } from 'react';

const Example = () => {
  const [id, setId] = useState(1);
  const [data, setData] = useState(null);

  // 缓存请求函数，依赖项为id
  const fetchData = useCallback(async () => {
    const res = await fetch(`https://api.example.com/data/${id}`);
    const result = await res.json();
    setData(result);
  }, [id]);

  // 仅当fetchData引用变化时（即id变化），触发请求
  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return (
    <div>
      <p>当前ID：{id}</p>
      <button onClick={() => setId(id + 1)}>切换ID</button>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
};
```

# useCallback 的使用案例

## 案例1：列表项点击事件优化
适用于长列表场景，避免列表项因父组件渲染重复创建点击函数：
```jsx
import React, { useState, useCallback, memo } from 'react';

// 列表项组件
const ListItem = memo(({ item, onItemClick }) => {
  console.log(`渲染列表项：${item.id}`);
  return <li onClick={() => onItemClick(item.id)}>{item.name}</li>;
});

const LongList = () => {
  const [list, setList] = useState([
    { id: 1, name: 'useState' },
    { id: 2, name: 'useEffect' },
    { id: 3, name: 'useCallback' },
  ]);
  const [filter, setFilter] = useState('');

  // 缓存列表项点击函数，无依赖项（始终复用）
  const handleItemClick = useCallback((itemId) => {
    console.log(`选中了列表项：${itemId}`);
  }, []);

  // 过滤列表（模拟父组件因filter变化重渲染）
  const filteredList = list.filter(item => item.name.includes(filter));

  return (
    <div>
      <input 
        type="text" 
        value={filter} 
        onChange={(e) => setFilter(e.target.value)} 
        placeholder="过滤列表"
      />
      <ul>
        {filteredList.map(item => (
          <ListItem key={item.id} item={item} onItemClick={handleItemClick} />
        ))}
      </ul>
    </div>
  );
};
```

## 案例2：自定义Hook中稳定函数引用
```jsx
// 自定义Hook：处理表单提交
const useFormSubmit = (onSubmit) => {
  const [loading, setLoading] = useState(false);

  // 缓存提交函数，依赖于外部传入的onSubmit
  const handleSubmit = useCallback(async (values) => {
    setLoading(true);
    try {
      await onSubmit(values);
    } catch (err) {
      console.error('提交失败：', err);
    } finally {
      setLoading(false);
    }
  }, [onSubmit]);

  return { handleSubmit, loading };
};

// 业务组件使用
const FormComponent = () => {
  const [name, setName] = useState('');

  // 缓存提交逻辑，依赖项为name
  const onFormSubmit = useCallback(async (values) => {
    console.log('提交数据：', { ...values, name });
    // 模拟接口请求
    await new Promise(resolve => setTimeout(resolve, 1000));
  }, [name]);

  const { handleSubmit, loading } = useFormSubmit(onFormSubmit);

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      handleSubmit({ name });
    }}>
      <input 
        type="text" 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      <button type="submit" disabled={loading}>
        {loading ? '提交中...' : '提交'}
      </button>
    </form>
  );
};
```

# useCallback 的使用注意事项

1. **不要滥用**：
   - 仅在需要稳定函数引用的场景（如传递给 memo 子组件、作为 useEffect/useMemo 依赖、自定义 Hook 传递函数）使用 useCallback；
   - 若函数仅在当前组件内部使用，且不依赖于外部状态/属性，直接定义函数即可，使用 useCallback 反而会增加额外的缓存开销。

2. **依赖项数组必须准确**：
   - 遗漏依赖项：会导致函数内部使用的变量还是旧值，引发逻辑错误（如闭包陷阱）；
   - 多余依赖项：会导致函数不必要地重新创建，失去缓存意义；
   - 可配合 ESLint 规则 `react-hooks/exhaustive-deps` 检查依赖项是否完整。

3. **与 React.memo 配合才有意义**：
   - 若子组件未使用 memo/shouldComponentUpdate 优化，即使使用 useCallback 缓存函数，子组件仍会因父组件渲染而重渲染，此时 useCallback 无优化效果。

4. **缓存的是函数引用，而非函数执行结果**：
   - useCallback 仅保证函数实例不变，函数内部逻辑的执行结果仍由其依赖项决定；若需缓存函数执行结果，应使用 useMemo（useMemo 可缓存任意值，包括函数执行结果）。

5. **浅比较规则**：
   - React 对 useCallback 的依赖项数组采用**浅比较**（===），若依赖项是对象/数组，即使内容未变但引用变化，仍会触发函数重新创建；此时需配合 useMemo 缓存对象/数组，或使用深比较（谨慎，深比较有性能开销）。

6. **与 useMemo 的关系**：
   - useCallback(fn, deps) 等价于 useMemo(() => fn, deps)；
   - 缓存函数用 useCallback，缓存计算结果用 useMemo，不要混淆两者的使用场景。

7. **严格模式下的行为**：
   - React 严格模式下，组件首次渲染时 useCallback 包裹的函数会被执行两次（模拟卸载重挂载），但最终返回的仍是同一个缓存实例，不影响业务逻辑。
---
