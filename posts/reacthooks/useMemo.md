---
title: useMemo
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
# useMemo

useMemo 是 React 提供的一个内置 Hook，它的核心设计目标是**缓存计算昂贵的函数结果**，避免在每次组件重新渲染时都重复执行高开销的计算逻辑。

从官方定义来看，useMemo 接收一个“创建函数”和一个依赖数组，仅当依赖数组中的某个依赖项发生变化时，才会重新执行创建函数并返回新的计算结果；若依赖未变化，则直接返回上一次缓存的结果。

# useMemo 的核心作用

1. **性能优化**：针对计算成本高的操作（例如复杂的数组遍历、数据转换、大量的数学运算等），通过缓存结果减少不必要的重复计算，降低组件渲染的性能开销。
2. **稳定引用**：当把计算结果传递给依赖引用相等性的组件（如 PureComponent、memo 包装的组件）或 Hook（如 useEffect、useCallback）时，useMemo 能保证未触发重计算时结果的引用不变，避免因引用变化导致的不必要重渲染或副作用执行。
3. **与 useCallback 互补**：useCallback 缓存函数引用，useMemo 缓存函数执行的结果，二者共同解决 React 组件重渲染时的性能浪费问题。

# useMemo 的使用方法

### 基本语法
```javascript
const memoizedValue = useMemo(() => {
  // 昂贵的计算逻辑
  return computeExpensiveValue(dep1, dep2);
}, [dep1, dep2]); // 依赖数组
```

### 关键说明
- **创建函数**：第一个参数是一个函数，该函数内部包含需要缓存的昂贵计算逻辑，且必须返回一个值（这是 useMemo 与 useCallback 的核心区别，useCallback 返回的是函数本身）。
- **依赖数组**：第二个参数是可选的依赖数组，React 会浅比较数组中的每个依赖项：
  - 若依赖数组为空 `[]`，则仅在组件首次渲染时执行创建函数，后续重渲染始终返回首次计算的结果；
  - 若不传递依赖数组，useMemo 会在每次组件重渲染时都执行创建函数（等同于无缓存，失去优化意义）；
  - 若依赖项发生变化，React 会重新执行创建函数，计算并缓存新的结果。
- **返回值**：useMemo 返回的是创建函数执行后的计算结果，该结果会被缓存直到依赖项变化。

### 基础使用示例
```javascript
import { useMemo, useState } from 'react';

function ExpensiveCalculationComponent() {
  const [count, setCount] = useState(0);
  const [input, setInput] = useState('');

  // 模拟昂贵的计算：计算从 0 到 count 的累加和
  const expensiveValue = useMemo(() => {
    console.log('执行昂贵计算');
    let sum = 0;
    for (let i = 0; i <= count; i++) {
      sum += i;
    }
    return sum;
  }, [count]); // 仅当 count 变化时重新计算

  return (
    <div>
      <p>昂贵计算结果: {expensiveValue}</p>
      <button onClick={() => setCount(prev => prev + 1)}>增加 count</button>
      <input 
        value={input} 
        onChange={(e) => setInput(e.target.value)} 
        placeholder="输入内容不会触发计算"
      />
    </div>
  );
}
```
上述示例中，修改 `input` 不会触发 `expensiveValue` 的重新计算（控制台不会打印“执行昂贵计算”），只有点击按钮修改 `count` 时，才会重新执行计算逻辑。

# useMemo 的使用案例

### 案例 1：优化列表渲染
当组件需要渲染大量数据的列表，且列表需要经过复杂筛选/转换时，使用 useMemo 缓存处理后的列表，避免每次渲染都重新处理数据。
```javascript
import { useMemo, useState } from 'react';

function BigListComponent({ users }) {
  const [searchText, setSearchText] = useState('');

  // 缓存筛选后的用户列表，仅当 users 或 searchText 变化时重新筛选
  const filteredUsers = useMemo(() => {
    console.log('筛选用户列表');
    return users.filter(user => 
      user.name.toLowerCase().includes(searchText.toLowerCase())
    );
  }, [users, searchText]);

  return (
    <div>
      <input 
        value={searchText} 
        onChange={(e) => setSearchText(e.target.value)} 
        placeholder="搜索用户名"
      />
      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 案例 2：避免子组件不必要的重渲染
结合 `React.memo` 使用 useMemo，保证传递给子组件的 props 引用稳定，避免子组件无意义的重渲染。
```javascript
import { useMemo, useState, memo } from 'react';

// 子组件：仅当 props 变化时重渲染
const ChildComponent = memo(({ userInfo }) => {
  console.log('子组件渲染');
  return <div>用户名: {userInfo.name}, 年龄: {userInfo.age}</div>;
});

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('张三');

  // 缓存 userInfo 对象，仅当 name 变化时重新创建对象
  const userInfo = useMemo(() => ({
    name,
    age: 25
  }), [name]);

  return (
    <div>
      <ChildComponent userInfo={userInfo} />
      <button onClick={() => setCount(prev => prev + 1)}>增加 count</button>
    </div>
  );
}
```
若不使用 useMemo 缓存 `userInfo`，每次点击按钮修改 `count` 时，`userInfo` 会创建新对象，导致子组件触发重渲染；使用 useMemo 后，仅当 `name` 变化时，`userInfo` 引用才会改变，子组件才会重渲染。

# useMemo 的使用注意事项

1. **不要过度使用**：useMemo 本身也有性能开销（需要跟踪依赖、缓存结果），对于简单的计算（如单个数字运算、简单字符串拼接），使用 useMemo 反而会增加额外开销，得不偿失。仅在计算确实“昂贵”（如大循环、复杂逻辑）或需要稳定引用时使用。
   
2. **依赖数组必须完整**：确保依赖数组包含所有在创建函数中使用的变量（组件内的 state、props、局部变量等），否则会导致缓存结果与实际数据不一致，引发难以排查的 bug。React 官方的 ESLint 规则 `react-hooks/exhaustive-deps` 可以帮助检查依赖是否完整。

3. **useMemo 不保证绝对不重计算**：React 可能在某些情况下（如内存不足）清除缓存，重新执行创建函数。因此，不要依赖 useMemo 执行具有副作用的逻辑（副作用应放在 useEffect 中），useMemo 仅用于缓存计算结果。

4. **与 useCallback 的区别**：
   - useMemo：缓存**函数执行的结果**，适用于缓存计算值；
   - useCallback：缓存**函数本身的引用**，适用于传递给子组件的回调函数。
   错误示例：用 useMemo 缓存函数（虽然可行，但语义上应使用 useCallback）：
   ```javascript
   // 不推荐（语义不符）
   const handleClick = useMemo(() => () => {
     console.log('点击事件');
   }, []);

   // 推荐
   const handleClick = useCallback(() => {
     console.log('点击事件');
   }, []);
   ```

5. **浅比较依赖项**：React 对依赖数组的比较是“浅比较”，若依赖项是对象/数组，即使内部属性变化，只要引用不变，useMemo 也不会重新计算。此时需要确保对象/数组的引用能正确反映内部数据变化，或使用深比较（需谨慎，深比较本身也有性能开销）。

6. **不要在 useMemo 中执行非计算逻辑**：useMemo 的创建函数应仅包含纯计算逻辑，不要在其中执行 DOM 操作、网络请求、修改 state 等副作用，这些操作应放在 useEffect 或事件处理函数中。
