---
title: HOC(Higher Order Component) 高阶组件
published: 2026-05-03
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](../react/reactcute.webp)

# HOC(Higher Order Component)高阶组件的官方定义
高阶组件（HOC）是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。具体来说，**高阶组件是一个接收组件作为参数并返回新组件的函数**。

从类型定义（TypeScript）角度，其核心形态可表示为：
```typescript
type HigherOrderComponent<P, T = {}> = (Component: React.ComponentType<P>) => React.ComponentType<P & T>;
```
需要明确的是，HOC 不是组件，而是一个函数，这是区分 HOC 和普通组件的核心——组件接收 props 并返回 JSX，而 HOC 接收组件并返回新组件。

# HOC(Higher Order Component)高阶组件的核心作用
1. **逻辑复用**：这是 HOC 最核心的价值。在多个组件需要共享相同逻辑（如数据请求、权限校验、状态管理、日志埋点等）时，HOC 可将这些逻辑抽离为独立函数，避免重复编写代码，符合 DRY（Don't Repeat Yourself）原则。
2. **增强组件能力**：为原组件注入额外的 props、状态或生命周期逻辑，无需修改原组件代码，遵循“开闭原则”（对扩展开放，对修改关闭）。
3. **抽象复杂逻辑**：将组件中与 UI 无关的复杂逻辑（如数据订阅、缓存处理）抽离到 HOC 中，让组件专注于 UI 渲染，提升代码可读性和可维护性。
4. **条件渲染控制**：可基于特定条件（如用户权限、设备类型）控制原组件是否渲染，或替换为其他组件（如未登录时展示登录弹窗）。

# HOC(Higher Order Component)高阶组件的使用方法
## 1. 基本实现步骤
### 步骤1：定义 HOC 函数
接收原始组件作为参数，返回一个新的组件，新组件中封装复用逻辑，并渲染原始组件。
### 步骤2：为原始组件注入 props/逻辑
新组件可通过 props 向原始组件传递额外数据，或添加生命周期、事件处理等逻辑。
### 步骤3：使用 HOC 包装目标组件
将需要增强的组件传入 HOC，得到增强后的新组件并使用。

## 2. 常见示例
### 示例1：带数据请求的 HOC
```typescript
import React, { ComponentType, useEffect, useState } from 'react';

// 定义HOC：封装数据请求逻辑
function withDataFetch<T>(url: string) {
  // 返回一个接收组件的函数
  return function (WrappedComponent: ComponentType<{ data: T | null; loading: boolean }>) {
    // 返回新组件
    return function DataFetchingComponent() {
      const [data, setData] = useState<T | null>(null);
      const [loading, setLoading] = useState(true);

      useEffect(() => {
        setLoading(true);
        fetch(url)
          .then(res => res.json())
          .then(data => {
            setData(data);
            setLoading(false);
          })
          .catch(() => {
            setLoading(false);
          });
      }, [url]);

      // 渲染被包装组件，并传入额外props
      return <WrappedComponent data={data} loading={loading} />;
    };
  };
}

// 原始组件：专注UI渲染
interface UserListProps {
  data: { id: number; name: string }[] | null;
  loading: boolean;
}
const UserList: ComponentType<UserListProps> = ({ data, loading }) => {
  if (loading) return <div>加载中...</div>;
  if (!data) return <div>数据加载失败</div>;
  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};

// 使用HOC增强组件
const UserListWithData = withDataFetch<{ id: number; name: string }[]>('/api/users')(UserList);

// 调用增强后的组件
function App() {
  return <UserListWithData />;
}
```

### 示例2：带权限控制的 HOC
```typescript
import React, { ComponentType } from 'react';

// 定义HOC：封装权限校验逻辑
interface WithAuthProps {
  hasPermission: boolean;
}
function withAuth(WrappedComponent: ComponentType<WithAuthProps>) {
  return function AuthComponent(props: { userRole: string }) {
    // 模拟权限判断
    const hasPermission = props.userRole === 'admin';

    // 无权限时展示提示，有权限时展示原组件
    if (!hasPermission) {
      return <div>无访问权限，请联系管理员</div>;
    }
    return <WrappedComponent {...props} hasPermission={hasPermission} />;
  };
}

// 原始组件：需要权限控制的操作面板
const AdminPanel: ComponentType<WithAuthProps> = () => {
  return <div>管理员操作面板：可修改用户权限、查看系统日志</div>;
};

// 使用HOC增强组件
const ProtectedAdminPanel = withAuth(AdminPanel);

// 调用增强后的组件
function App() {
  return (
    <>
      {/* 管理员角色 */}
      <ProtectedAdminPanel userRole="admin" />
      {/* 普通用户角色 */}
      <ProtectedAdminPanel userRole="user" />
    </>
  );
}
```

## 3. HOC 的进阶写法（装饰器模式）
注：装饰器语法（`@`）目前是 Stage 3 提案，需配置 babel 或 TypeScript 支持：
```typescript
import React, { ComponentType } from 'react';

// 定义HOC
function withLog(WrappedComponent: ComponentType) {
  const LoggedComponent = (props: any) => {
    console.log(`组件渲染：${WrappedComponent.displayName || WrappedComponent.name}`);
    console.log('传入props：', props);
    return <WrappedComponent {...props} />;
  };
  // 设置displayName，方便调试
  LoggedComponent.displayName = `withLog(${WrappedComponent.displayName || WrappedComponent.name})`;
  return LoggedComponent;
}

// 使用装饰器语法包装组件
@withLog
const MyComponent = (props: { text: string }) => {
  return <div>{props.text}</div>;
};
```

# HOC(Higher Order Component)高阶组件的实际业务场景使用案例
## 场景1：电商项目 - 商品列表复用筛选逻辑
电商项目中，“商品列表”“秒杀商品列表”“推荐商品列表”都需要筛选（价格区间、分类、销量排序）逻辑，可通过 HOC 抽离筛选逻辑：
```typescript
// 封装筛选逻辑的HOC
function withFilter<T>(WrappedComponent: ComponentType<{ list: T[]; filterParams: any }>) {
  return function FilterComponent(props: { rawList: T[]; filterConfig: any }) {
    // 抽离筛选逻辑：根据filterConfig过滤rawList
    const filterList = () => {
      const { rawList, filterConfig } = props;
      return rawList.filter(item => {
        // 价格区间筛选
        if (filterConfig.minPrice && item.price < filterConfig.minPrice) return false;
        if (filterConfig.maxPrice && item.price > filterConfig.maxPrice) return false;
        // 分类筛选
        if (filterConfig.category && item.category !== filterConfig.category) return false;
        return true;
      });
    };

    const filteredList = filterList();
    return <WrappedComponent list={filteredList} filterParams={props.filterConfig} />;
  };
}

// 商品列表组件
const ProductList = ({ list, filterParams }) => {
  return (
    <div>
      <div>筛选条件：{JSON.stringify(filterParams)}</div>
      <div className="product-list">
        {list.map(product => (
          <div key={product.id} className="product-item">
            <h3>{product.name}</h3>
            <p>价格：{product.price}</p>
          </div>
        ))}
      </div>
    </div>
  );
};

// 增强后的商品列表
const FilteredProductList = withFilter(ProductList);

// 业务中使用
function GoodsPage() {
  const rawProductList = [/* 原始商品数据 */];
  return (
    <FilteredProductList
      rawList={rawProductList}
      filterConfig={{ minPrice: 100, maxPrice: 1000, category: 'electronics' }}
    />
  );
}
```

## 场景2：后台管理系统 - 表单提交防抖 + 加载状态
后台管理系统中，多个表单（用户表单、订单表单、商品表单）提交时需要防抖（防止重复提交）和加载状态展示，可通过 HOC 封装：
```typescript
import React, { ComponentType, useState, useCallback } from 'react';
import { debounce } from 'lodash';

// 封装表单提交逻辑的HOC
function withFormSubmit(WrappedComponent: ComponentType<{ onSubmit: (data: any) => void; loading: boolean }>) {
  return function FormSubmitComponent() {
    const [loading, setLoading] = useState(false);

    // 防抖 + 加载状态的提交函数
    const handleSubmit = useCallback(
      debounce(async (formData) => {
        setLoading(true);
        try {
          // 模拟接口提交
          await new Promise(resolve => setTimeout(resolve, 1000));
          console.log('表单提交成功：', formData);
        } catch (e) {
          console.error('表单提交失败：', e);
        } finally {
          setLoading(false);
        }
      }, 500),
      []
    );

    return <WrappedComponent onSubmit={handleSubmit} loading={loading} />;
  };
}

// 订单表单组件
const OrderForm = ({ onSubmit, loading }) => {
  const handleClick = () => {
    onSubmit({ orderNo: 'ORD123456', amount: 999 });
  };

  return (
    <div>
      <h3>订单提交表单</h3>
      <button onClick={handleClick} disabled={loading}>
        {loading ? '提交中...' : '提交订单'}
      </button>
    </div>
  );
};

// 增强后的订单表单
const OrderFormWithSubmit = withFormSubmit(OrderForm);

// 业务页面使用
function OrderPage() {
  return <OrderFormWithSubmit />;
}
```

# HOC(Higher Order Component)高阶组件的使用注意事项
1. **不要在组件内部定义 HOC**：
   每次组件渲染时，HOC 会创建新的组件，导致之前的组件实例被卸载，触发重新渲染，丢失状态。
   错误示例：
   ```typescript
   function MyComponent() {
     // 每次渲染都会创建新的withDataFetch
     const EnhancedComponent = withDataFetch('/api/data')(SomeComponent);
     return <EnhancedComponent />;
   }
   ```
   正确示例：在组件外部定义 HOC，确保组件引用稳定。

2. **透传不相关的 props**：
   HOC 应将未使用的 props 透传给被包装组件，避免丢失 props。
   ```typescript
   function withHOC(WrappedComponent) {
     return function NewComponent(props) {
       // 提取HOC需要的props，其余透传
       const { hocProp, ...restProps } = props;
       // 处理hocProp逻辑...
       return <WrappedComponent {...restProps} />;
     };
   }
   ```

3. **设置 displayName 便于调试**：
   默认情况下，HOC 返回的组件名称会覆盖原组件，导致 React DevTools 中难以识别。需手动设置 displayName：
   ```typescript
   function withHOC(WrappedComponent) {
     function NewComponent() { /* ... */ }
     NewComponent.displayName = `withHOC(${WrappedComponent.displayName || WrappedComponent.name})`;
     return NewComponent;
   }
   ```

4. **不要修改原组件，而是组合**：
   HOC 的核心是“组合”而非“修改”，避免直接修改被包装组件的原型或状态，保持纯函数特性。
   错误示例：
   ```typescript
   function withBadHOC(WrappedComponent) {
     WrappedComponent.prototype.componentDidMount = function() {
       // 直接修改原组件生命周期，污染原组件
     };
     return WrappedComponent;
   }
   ```
   正确示例：通过新组件封装逻辑，不修改原组件。

5. **避免多层 HOC 嵌套的复杂度**：
   过多 HOC 嵌套会导致组件 props 溯源困难（“嵌套地狱”），可结合 React Hooks（如 useContext、useReducer）替代部分 HOC 场景，或使用组合方式简化。

6. **注意 Refs 透传**：
   HOC 会屏蔽原组件的 ref，需使用 `React.forwardRef` 透传 ref：
   ```typescript
   function withHOC(WrappedComponent) {
     return React.forwardRef((props, ref) => {
       return <WrappedComponent {...props} ref={ref} />;
     });
   }
   ```

7. **与 Hooks 的取舍**：
   React 16.8+ 推出的 Hooks 可更简洁地复用逻辑，对于简单逻辑，Hooks 比 HOC 更轻量；但对于复杂的组件增强（如批量包装多个组件、条件渲染），HOC 仍有优势。避免盲目使用 HOC，根据场景选择合适的方案。
