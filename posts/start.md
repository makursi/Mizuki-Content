---
title: React组件学习
published: 2026-05-03
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](./react/reactcute.webp)

# 组件的定义和认识

## 组件的官方定义
在 React 中，组件是可复用的、独立的代码单元，用于封装 UI 中的一部分，可将 UI 拆分为独立且可复用的片段，并对每个片段进行独立思考。

从技术角度来讲，React 组件本质上是接受入参（props）并返回 React 元素（描述屏幕上应显示内容）的 JavaScript 函数（基于 TypeScript 开发时则是类型化的函数/类）。

React 官方将组件分为两大类：
1. **函数组件**：以函数形式定义的纯组件，是 React 16.8 后推荐的主要组件形式，借助 Hooks 可以实现所有类组件的能力，语法简洁且无实例化开销。
2. **类组件**：基于 ES6 Class 定义的组件，需继承 `React.Component` 或 `React.PureComponent`，通过 `render()` 方法返回 UI 结构，在 Hooks 普及前是复杂逻辑组件的主要实现方式。

## 组件的核心作用
1. **复用性**：将通用的 UI 逻辑和界面封装为组件，可在应用内多处复用，减少重复代码，提升开发效率（如通用的按钮、表单输入框、弹窗组件等）。
2. **模块化**：将复杂的应用 UI 拆分为粒度更小的组件，使代码结构清晰，符合“单一职责原则”，便于维护和迭代（如将电商页面拆分为头部、商品列表、购物车、底部组件）。
3. **可组合性**：组件可以嵌套、组合使用，简单组件可组合成复杂组件，灵活适配不同业务场景（如表单组件由输入框、按钮、校验提示等子组件组合而成）。
4. **可维护性**：组件的独立封装使问题定位、Bug 修复、功能升级更聚焦，修改一个组件不会影响其他无关组件，降低维护成本。
5. **声明式编程**：组件通过描述“应该呈现的样子”而非“如何实现”，让开发者专注于 UI 逻辑，React 负责处理 DOM 渲染和更新，提升开发体验。

## 组件的使用方法
### 1. 函数组件（React + TypeScript）
```tsx
import React from 'react';

// 定义组件属性类型
interface ButtonProps {
  text: string;
  onClick?: () => void;
  type?: 'primary' | 'secondary';
}

// 函数组件定义
const Button: React.FC<ButtonProps> = ({ text, onClick, type = 'primary' }) => {
  return (
    <button 
      className={`btn btn-${type}`}
      onClick={onClick}
    >
      {text}
    </button>
  );
};

// 使用组件
const App: React.FC = () => {
  const handleClick = () => console.log('按钮点击');
  
  return (
    <div>
      <Button text="主要按钮" type="primary" onClick={handleClick} />
      <Button text="次要按钮" type="secondary" />
    </div>
  );
};

export default App;
```

### 2. 类组件（React + TypeScript）
```tsx
import React, { Component } from 'react';

interface CounterProps {
  initialCount: number;
}

interface CounterState {
  count: number;
}

// 类组件定义
class Counter extends Component<CounterProps, CounterState> {
  // 初始化状态
  constructor(props: CounterProps) {
    super(props);
    this.state = { count: props.initialCount };
  }

  // 定义方法
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  // 渲染UI
  render() {
    return (
      <div>
        <p>计数：{this.state.count}</p>
        <button onClick={this.increment}>加1</button>
      </div>
    );
  }
}

// 使用组件
const App: React.FC = () => {
  return <Counter initialCount={0} />;
};

export default App;
```

## 组件的使用案例
### 业务场景：电商商品卡片组件
```tsx
import React from 'react';
import { Image, Tag } from 'antd'; // 假设使用Ant Design组件库

// 定义商品卡片属性类型
interface ProductCardProps {
  id: string;
  name: string;
  price: number;
  imgUrl: string;
  isDiscount: boolean;
  discountRate?: number;
  onAddCart: (id: string) => void;
}

// 商品卡片组件
const ProductCard: React.FC<ProductCardProps> = ({
  id,
  name,
  price,
  imgUrl,
  isDiscount,
  discountRate,
  onAddCart,
}) => {
  // 计算折扣后价格
  const finalPrice = isDiscount && discountRate ? price * (1 - discountRate / 100) : price;

  return (
    <div className="product-card" style={{ border: '1px solid #eee', padding: 16, width: 240 }}>
      <Image src={imgUrl} alt={name} width={240} height={180} />
      <h3 style={{ fontSize: 16, margin: '8px 0' }}>{name}</h3>
      <div style={{ display: 'flex', alignItems: 'center', margin: '8px 0' }}>
        <span style={{ color: 'red', fontSize: 18 }}>¥{finalPrice.toFixed(2)}</span>
        {isDiscount && (
          <Tag color="red" style={{ marginLeft: 8 }}>
            {discountRate}折
          </Tag>
        )}
        {isDiscount && (
          <span style={{ marginLeft: 8, color: '#999', textDecoration: 'line-through' }}>
            ¥{price.toFixed(2)}
          </span>
        )}
      </div>
      <button 
        className="add-cart-btn"
        style={{ width: '100%', padding: 8, background: '#0088ff', color: '#fff', border: 'none', borderRadius: 4 }}
        onClick={() => onAddCart(id)}
      >
        加入购物车
      </button>
    </div>
  );
};

// 批量渲染商品列表
const ProductList: React.FC = () => {
  // 模拟商品数据
  const products = [
    { id: '1', name: 'iPhone 15', price: 5999, imgUrl: '/images/iphone15.jpg', isDiscount: true, discountRate: 10 },
    { id: '2', name: '华为Mate 60', price: 4999, imgUrl: '/images/mate60.jpg', isDiscount: false },
  ];

  // 加入购物车逻辑
  const handleAddCart = (id: string) => {
    console.log(`商品${id}加入购物车`);
    // 实际业务中可调用接口、更新redux状态等
  };

  return (
    <div style={{ display: 'flex', gap: 16, padding: 20 }}>
      {products.map(product => (
        <ProductCard key={product.id} {...product} onAddCart={handleAddCart} />
      ))}
    </div>
  );
};

export default ProductList;
```

## 组件的使用注意事项
1. **组件命名规范**：
   - 组件名称必须以大写字母开头（React 区分组件和普通 DOM 元素的依据），如 `Button` 而非 `button`。
   - 文件名建议与组件名一致，如 `Button.tsx`，便于查找和维护。

2. **属性（Props）相关**：
   - Props 是只读的，组件内部不能修改传入的 Props，遵循“单向数据流”原则，若需修改需通过父组件传递回调函数实现。
   - 使用 TypeScript 时必须显式定义 Props 类型，避免隐式 any 类型，提升代码健壮性。
   - 为 Props 设置合理的默认值，避免因未传参导致的渲染错误。

3. **状态（State）相关**：
   - 状态仅用于组件内部可变数据，通用数据建议提升至父组件或使用状态管理库（如 Redux、Zustand）。
   - 类组件中 setState 是异步的，更新状态时若依赖当前状态，需使用函数形式（如 `this.setState(prev => ({ count: prev.count + 1 }))`）。
   - 函数组件中使用 useState 时，状态更新是替换而非合并，需注意对象/数组类型的状态更新。

4. **组件复用与拆分**：
   - 避免过度拆分组件，粒度过细会增加组件通信成本，建议以“可复用、单一职责”为拆分原则。
   - 通用逻辑优先通过自定义 Hooks 抽离，而非重复编写（如表单校验、数据请求 Hooks）。

5. **性能优化**：
   - 避免在渲染过程中创建新函数、新对象（如 `onClick={() => {}}`），可通过 useCallback、useMemo 缓存。
   - 类组件可使用 `React.PureComponent` 或手动实现 `shouldComponentUpdate` 避免不必要的重渲染；函数组件可使用 `React.memo` 包装。
   - 大列表渲染需使用虚拟列表（如 react-window），减少 DOM 节点数量。

6. **生命周期/副作用管理**：
   - 类组件中避免在 `componentWillMount` 中发起异步请求，建议在 `componentDidMount` 中执行；函数组件中使用 useEffect 管理副作用，注意清理副作用（如取消请求、移除事件监听）。
   - useEffect 需合理设置依赖数组，避免无限循环或遗漏依赖。

7. **错误边界**：
   - 为关键组件添加错误边界（Error Boundary），捕获子组件渲染错误，避免整个应用崩溃，提升用户体验。
