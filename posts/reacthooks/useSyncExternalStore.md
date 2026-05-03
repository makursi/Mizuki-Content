---
title: useSyncExternalStore
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
# useSyncExternalStore

`useSyncExternalStore` 是 React 官方提供的一个内置 Hook，专门用于**安全地读取和订阅外部存储（External Store）** 的值，确保组件在外部存储更新时能够同步重新渲染，同时兼容 React 的并发渲染特性（如 Suspense、Transitions 等）。它是 React 18 新增的 API，解决了传统自定义订阅 Hook 在并发模式下可能出现的渲染不一致、撕裂（tearing）等问题。

# 核心作用

1. **统一外部存储订阅范式**：为读取外部状态（如 Redux 存储、全局事件总线、浏览器 API 状态、第三方状态管理库等）提供标准化的订阅/取消订阅机制，替代自定义的 `useEffect` + 订阅逻辑，避免手动管理订阅的漏写/错写。
2. **兼容并发渲染**：保证在 React 并发更新（如 `startTransition`、Suspense 加载）过程中，组件读取的外部状态是一致的，不会出现“中间状态撕裂”的问题。
3. **支持选择性更新**：通过返回的快照（snapshot）控制组件是否重新渲染，仅当快照值变化时触发更新，优化性能。
4. **兜底服务端渲染**：内置对服务端渲染（SSR）的支持，在服务端执行时会调用 `getServerSnapshot` 方法获取初始快照，避免客户端/服务端渲染不匹配。

# 使用方法

## 基本语法
```javascript
const snapshot = useSyncExternalStore(
  subscribe,    // 订阅外部存储的函数
  getSnapshot,  // 获取外部存储当前值的函数
  getServerSnapshot? // 可选，服务端渲染时获取初始快照的函数
);
```

### 参数详解
| 参数                | 类型                | 说明                                                                 |
|---------------------|---------------------|----------------------------------------------------------------------|
| `subscribe`         | `(callback) => () => void` | 订阅函数：接收一个“更新回调”，返回“取消订阅”函数。当外部存储变化时，调用回调触发组件重新渲染。 |
| `getSnapshot`       | `() => T`           | 快照函数：返回外部存储的当前值（快照）。React 会对比两次快照的浅值，不同则触发重新渲染。 |
| `getServerSnapshot` | `() => T`           | 可选，服务端渲染专用：返回服务端环境下的初始快照，避免客户端水合（hydration）不匹配。 |

## 核心使用原则
1. `subscribe` 必须是稳定的（如通过 `useCallback` 包裹），避免每次渲染都创建新函数导致重复订阅/取消订阅；
2. `getSnapshot` 返回的快照必须是**不可变值**（如原始类型、冻结的对象），否则 React 无法正确判断是否需要更新；
3. 外部存储更新时，必须调用 `subscribe` 传入的回调函数，才能触发组件重新读取快照并渲染。

# 使用案例

## 案例1：订阅浏览器 `window.size` （基础场景）
```javascript
import { useSyncExternalStore } from 'react';

// 封装获取窗口尺寸的自定义Hook
function useWindowSize() {
  // 1. 定义订阅函数：监听 resize 事件
  const subscribe = useCallback((callback) => {
    window.addEventListener('resize', callback);
    // 返回取消订阅函数
    return () => window.removeEventListener('resize', callback);
  }, []);

  // 2. 定义获取快照函数：返回当前窗口尺寸（原始类型，不可变）
  const getSnapshot = () => {
    return {
      width: window.innerWidth,
      height: window.innerHeight,
    };
  };

  // 3. 服务端快照（可选）：服务端无 window，返回默认值
  const getServerSnapshot = () => {
    return { width: 0, height: 0 };
  };

  // 4. 调用 useSyncExternalStore
  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

// 组件中使用
function WindowSizeDisplay() {
  const size = useWindowSize();
  return (
    <div>
      窗口宽度：{size.width}px<br />
      窗口高度：{size.height}px
    </div>
  );
}
```

## 案例2：订阅外部 Redux 存储（复杂状态管理）
```javascript
import { useSyncExternalStore } from 'react';
import { store } from './redux/store'; // 已创建的 Redux store

// 封装订阅 Redux store 的自定义Hook
function useReduxStore(selector) {
  // 1. 订阅 Redux store 的变化
  const subscribe = useCallback((callback) => {
    return store.subscribe(callback); // Redux store 内置 subscribe 方法
  }, []);

  // 2. 获取快照：通过 selector 筛选需要的状态（返回不可变值）
  const getSnapshot = useCallback(() => {
    return selector(store.getState());
  }, [selector]);

  // 3. 服务端快照（假设服务端有初始 state）
  const getServerSnapshot = useCallback(() => {
    return selector(store.getInitialState());
  }, [selector]);

  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

// 组件中使用：订阅 Redux 中的 user 状态
function UserProfile() {
  const user = useReduxStore((state) => state.user);
  if (!user) return <div>加载中...</div>;
  return (
    <div>
      用户名：{user.name}<br />
      邮箱：{user.email}
    </div>
  );
}
```

# 使用注意事项

## 1. 避免快照值可变导致的更新异常
`getSnapshot` 返回的快照如果是可变对象（如普通对象/数组），React 会通过**浅比较**判断是否更新，若对象引用未变（即使内容变了），组件不会重新渲染。  
正确做法：返回不可变值（原始类型、`Object.freeze` 冻结对象、解构新对象）：
```javascript
// 错误：返回同一个可变对象，内容变化但引用不变，组件不更新
const getSnapshot = () => windowSizeObj; 

// 正确：返回新对象，引用变化触发更新
const getSnapshot = () => ({ ...windowSizeObj }); 
```

## 2. 避免订阅函数不稳定
`subscribe` 若每次渲染都创建新函数，会导致 React 频繁取消旧订阅、创建新订阅，引发性能问题或状态丢失。  
正确做法：用 `useCallback` 包裹 `subscribe`：
```javascript
// 错误：每次渲染都创建新的 subscribe 函数
const subscribe = (callback) => { /* ... */ }; 

// 正确：稳定的订阅函数
const subscribe = useCallback((callback) => { /* ... */ }, []); 
```

## 3. 服务端渲染必须提供 `getServerSnapshot`
若组件在服务端渲染（如 Next.js、Remix），未提供 `getServerSnapshot` 会导致 React 警告，且客户端水合时可能出现内容不匹配。  
正确做法：根据服务端环境返回合理的初始值：
```javascript
const getServerSnapshot = () => {
  // 服务端无 window，返回默认尺寸
  return typeof window === 'undefined' ? { width: 1200, height: 800 } : getSnapshot();
};
```

## 4. 避免在 `getSnapshot` 中执行副作用
`getSnapshot` 仅用于**读取**外部状态，不能包含异步操作、修改状态等副作用，否则会导致渲染不一致。  
❌ 错误：
```javascript
const getSnapshot = () => {
  fetch('/api/data'); // 副作用：异步请求
  return window.innerWidth;
};
```

## 5. 并发模式下的更新优先级
`useSyncExternalStore` 触发的更新默认是**紧急更新**（高优先级），若需低优先级更新（如非关键的UI变化），可结合 `startTransition`：
```javascript
const subscribe = useCallback((callback) => {
  const handleChange = () => {
    startTransition(callback); // 低优先级更新
  };
  window.addEventListener('resize', handleChange);
  return () => window.removeEventListener('resize', handleChange);
}, []);
```

## 6. 替代传统的 `useEffect + useState` 订阅
对于简单的外部状态订阅，传统写法在并发模式下可能出现撕裂，建议优先使用 `useSyncExternalStore`：
```javascript
// 传统写法（并发模式有风险）
const [width, setWidth] = useState(window.innerWidth);
useEffect(() => {
  const handleResize = () => setWidth(window.innerWidth);
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);

// 推荐写法（兼容并发模式）
const width = useSyncExternalStore(
  useCallback((cb) => {
    window.addEventListener('resize', cb);
    return () => window.removeEventListener('resize', cb);
  }, []),
  () => window.innerWidth,
  () => 1200
);
```

---
