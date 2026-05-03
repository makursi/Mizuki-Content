---
title: useReducer
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
# useReducer


`useReducer` 是 React 提供的一个用于状态管理的 Hook，它是 `useState` 的替代方案，适用于处理复杂的状态逻辑。

其核心遵循 Redux 风格的 reducer 模式，接收一个形如 `(state, action) => newState` 的 reducer 函数和初始状态，返回当前状态以及一个派发（dispatch）动作的方法。从官方定义来看，它让组件的状态更新逻辑可以被抽离和复用，尤其适合多个状态操作之间存在关联、状态逻辑较复杂的场景。

# useReducer的核心作用


1. **复杂状态管理**：当组件的状态包含多个子值（如对象、数组），或状态更新逻辑需要根据不同条件执行复杂计算时，`useReducer` 能将状态更新逻辑集中到 reducer 函数中，让组件逻辑更清晰，避免在组件内写大量分散的 `setState` 逻辑。
2. **状态逻辑复用**：可以将 reducer 函数抽离到组件外部，多个组件可复用相同的状态更新逻辑，提升代码复用性。
3. **可预测的状态更新**：基于“action 驱动”的模式，状态更新的触发和逻辑分离，每一次状态变更都对应明确的 action，便于调试和追踪状态变化（结合 React DevTools 可清晰看到 action 派发记录）。
4. **替代嵌套的 useState**：当多个 `useState` 之间存在依赖关系（比如修改 A 状态需要同时调整 B 状态），`useReducer` 能更优雅地处理这种关联更新。

# useReducer的使用方法


### 基本语法
```javascript
const [state, dispatch] = useReducer(reducer, initialArg, init?);
```
- `reducer`：必选，纯函数，接收两个参数：当前 `state` 和 `action`，返回新的状态。
- `initialArg`：必选，初始状态的参数，可直接传初始值，或配合 `init` 函数生成初始状态。
- `init`：可选，初始化函数，用于惰性创建初始状态，接收 `initialArg` 作为参数，返回真正的初始状态。

### 基础使用步骤
1. **定义 reducer 函数**：根据不同的 action type 处理状态更新逻辑。
2. **初始化状态**：直接传入初始值，或通过 `init` 函数惰性初始化。
3. **在组件中调用 useReducer**：获取当前 `state` 和 `dispatch` 方法。
4. **派发 action**：通过 `dispatch` 传入 action 对象（通常包含 `type` 标识和 `payload` 数据），触发状态更新。

### 惰性初始化示例（可选）
当初始状态需要复杂计算时，可通过 `init` 函数惰性创建，避免每次渲染都执行计算：
```javascript
function init(initialCount) {
  return { count: initialCount, step: 1 };
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'setStep':
      return { ...state, step: action.payload };
    default:
      throw new Error('未知的 action type');
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, 0, init);
  // ...
}
```

# useReducer的使用案例


### 案例1：基础计数器（对比 useState 更易扩展）
```javascript
import { useReducer } from 'react';

// 定义 reducer 函数
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: action.payload }; // 重置为指定值
    default:
      throw new Error(`未处理的 action type: ${action.type}`);
  }
}

function Counter() {
  // 初始化状态，调用 useReducer
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <p>当前计数: {state.count}</p>
      {/* 派发 action 触发状态更新 */}
      <button onClick={() => dispatch({ type: 'increment' })}>+1</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-1</button>
      <button onClick={() => dispatch({ type: 'reset', payload: 0 })}>重置</button>
    </div>
  );
}

export default Counter;
```

### 案例2：复杂表单状态管理
当表单包含多个字段（用户名、密码、记住密码），用 `useReducer` 统一管理更清晰：
```javascript
import { useReducer } from 'react';

const initialState = {
  username: '',
  password: '',
  rememberMe: false
};

function formReducer(state, action) {
  switch (action.type) {
    case 'UPDATE_FIELD':
      return {
        ...state,
        [action.field]: action.value
      };
    case 'RESET_FORM':
      return initialState;
    default:
      return state;
  }
}

function LoginForm() {
  const [formState, dispatch] = useReducer(formReducer, initialState);

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    // 适配 checkbox 的 checked 属性
    const fieldValue = type === 'checkbox' ? checked : value;
    dispatch({
      type: 'UPDATE_FIELD',
      field: name,
      value: fieldValue
    });
  };

  const handleReset = () => {
    dispatch({ type: 'RESET_FORM' });
  };

  return (
    <form>
      <div>
        <label>用户名:</label>
        <input
          type="text"
          name="username"
          value={formState.username}
          onChange={handleChange}
        />
      </div>
      <div>
        <label>密码:</label>
        <input
          type="password"
          name="password"
          value={formState.password}
          onChange={handleChange}
        />
      </div>
      <div>
        <label>
          <input
            type="checkbox"
            name="rememberMe"
            checked={formState.rememberMe}
            onChange={handleChange}
          />
          记住我
        </label>
      </div>
      <button type="button" onClick={handleReset}>重置</button>
    </form>
  );
}

export default LoginForm;
```

# useReducer的使用注意事项


1. **reducer 必须是纯函数**：
   - 不能在 reducer 中执行副作用（如请求数据、修改 DOM、调用非纯函数），仅能根据 `state` 和 `action` 返回新状态。
   - 不能直接修改原状态对象/数组（React 依赖不可变数据判断更新），需返回新的对象/数组（如用扩展运算符 `...`、`Array.map`/`Array.filter` 等）。

2. **action 建议规范格式**：
   - 推荐使用 `type` 字段标识动作类型（字符串/Symbol），`payload` 携带数据，便于维护和调试。
   - 避免在 action 中传入函数或可变数据，保证 action 可序列化，便于追踪状态变化。

3. **初始状态的注意点**：
   - 直接传入的初始状态仅在组件首次渲染时生效，后续渲染会忽略；若需动态修改初始状态，需配合 key 重置组件或使用惰性初始化。
   - 惰性初始化（`init` 函数）仅在组件首次渲染时执行一次，适合初始状态需要复杂计算的场景。

4. **dispatch 方法的稳定性**：
   - `dispatch` 方法的引用在组件生命周期中保持不变（除非组件卸载重建），因此可以安全地传递给子组件、作为 useEffect 的依赖（无需加入依赖数组）。

5. **与 useState 的选择原则**：
   - 简单状态（单个值、无关联更新）：优先用 `useState`，更简洁。
   - 复杂状态（多字段关联、更新逻辑复杂、需要复用更新逻辑）：优先用 `useReducer`。

6. **错误处理**：
   - 建议在 reducer 中对未知的 `action.type` 抛出错误，避免静默失败，便于定位问题。

7. **性能优化**：
   - 若 reducer 返回的新状态与原状态引用相同（如 `return state`），React 会跳过组件重渲染。
   - 当状态是大型对象/数组时，避免不必要的拷贝，可结合 Immer 库简化不可变数据操作（非官方内置，需额外引入）。
---
