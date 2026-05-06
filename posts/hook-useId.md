---
title: useId
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
# useId

useId 是 React 18 中新增的一个内置 Hook，用于生成跨服务端和客户端的唯一标识符（ID），且不会因为服务端渲染（SSR）的水合（hydration）过程而导致不匹配的问题。

它生成的 ID 是稳定的，在服务端和客户端渲染的同一组件实例中会生成相同的值，解决了传统方式（如随机数、自增ID）在 SSR 场景下易出现的水合不匹配问题。

# useId 的核心作用

1. **SSR 友好的唯一 ID 生成**：解决服务端渲染时，客户端与服务端生成的 ID 不一致导致的水合警告/错误，保证同一份组件在服务端和客户端渲染出的 ID 完全相同。
2. **无障碍（a11y）场景适配**：为表单控件（如 input/label 关联）、ARIA 属性（如 aria-labelledby、aria-describedby）等需要唯一 ID 关联的场景提供可靠标识，提升应用的无障碍性。
3. **无副作用的 ID 生成**：生成过程不依赖组件状态/副作用，属于纯计算型 Hook，不会触发额外的重渲染，且生成的 ID 仅与组件在组件树中的位置相关，而非组件的渲染顺序或其他动态因素。
4. **批量生成关联 ID**：支持通过 `useId()` 生成根 ID，再拼接后缀的方式，为同一组件内的多个元素生成关联且唯一的 ID 集合。

# useId 的使用方法

### 基础使用方式
```jsx
import { useId } from 'react';

function InputWithLabel() {
  // 生成唯一 ID
  const inputId = useId();

  return (
    <div>
      {/* label 与 input 通过 ID 关联，提升无障碍性 */}
      <label htmlFor={inputId}>用户名：</label>
      <input id={inputId} type="text" />
    </div>
  );
}
```

### 批量生成关联 ID
```jsx
import { useId } from 'react';

function FormField() {
  // 生成根 ID
  const rootId = useId();
  // 拼接后缀生成多个关联 ID
  const inputId = `${rootId}-input`;
  const errorId = `${rootId}-error`;
  const helpId = `${rootId}-help`;

  return (
    <div>
      <label htmlFor={inputId}>邮箱：</label>
      <input
        id={inputId}
        type="email"
        aria-describedby={`${helpId} ${errorId}`} // 关联帮助文本和错误文本
      />
      <p id={helpId}>请输入有效的邮箱地址（如 xxx@example.com）</p>
      <p id={errorId} style={{ color: 'red' }}>邮箱格式错误</p>
    </div>
  );
}
```

### 服务端渲染（SSR）场景使用
useId 无需额外配置即可适配 SSR（如 Next.js、Remix 等框架），服务端和客户端会生成相同的 ID：
```jsx
// 服务端/客户端通用组件
import { useId } from 'react';

export default function SSRComponent() {
  const uniqueId = useId();
  return <div id={uniqueId}>SSR 友好的唯一 ID</div>;
}
```

# useId 的使用案例

### 案例1：表单控件与 label 关联（基础无障碍场景）
```jsx
import { useId, useState } from 'react';

function LoginForm() {
  const [password, setPassword] = useState('');
  const passwordId = useId();

  return (
    <form>
      <div>
        <label htmlFor={passwordId}>密码：</label>
        <input
          id={passwordId}
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          placeholder="请输入密码"
        />
      </div>
    </form>
  );
}
```

### 案例2：ARIA 无障碍属性关联（进阶无障碍场景）
```jsx
import { useId, useState } from 'react';

function ToggleSwitch() {
  const switchId = useId();
  const [isChecked, setIsChecked] = useState(false);

  return (
    <div>
      <label htmlFor={switchId}>
        开启通知：
        <input
          id={switchId}
          type="checkbox"
          checked={isChecked}
          onChange={(e) => setIsChecked(e.target.checked)}
          aria-label={isChecked ? "通知已开启" : "通知已关闭"}
        />
      </label>
    </div>
  );
}
```

### 案例3：动态生成多个表单元素（批量 ID 场景）
```jsx
import { useId } from 'react';

function MultiInputForm() {
  const baseId = useId();
  // 模拟动态生成的表单字段
  const fields = ['username', 'phone', 'address'];

  return (
    <form>
      {fields.map((field) => {
        const fieldId = `${baseId}-${field}`;
        return (
          <div key={field}>
            <label htmlFor={fieldId}>{field === 'username' ? '用户名' : field === 'phone' ? '手机号' : '地址'}：</label>
            <input id={fieldId} name={field} type={field === 'phone' ? 'tel' : 'text'} />
          </div>
        );
      })}
    </form>
  );
}
```

# useId 的使用注意事项

1. **不要用于生成列表 key**：useId 生成的 ID 是字符串且较长，而列表的 key 应优先使用数据自身的唯一标识（如 ID、UUID），而非 useId。useId 生成的 ID 仅与组件位置相关，若列表项顺序变化，key 会失效，导致性能问题或渲染错误。
   错误用法：
   ```jsx
   {list.map(item => <li key={useId()}>{item.name}</li>)}
   ```
   正确用法：
   ```jsx
   {list.map(item => <li key={item.id}>{item.name}</li>)}
   ```

2. **ID 仅在组件树位置稳定时唯一**：若组件在组件树中的位置发生变化（如被移动、条件渲染切换），useId 生成的 ID 会改变。若需固定 ID，应手动指定而非依赖 useId。

3. **不支持生成数字 ID**：useId 生成的是字符串（以 `:` 开头，如 `:R0:`），无法直接作为数字使用，若需数字 ID 需自行转换（但不推荐，易丢失唯一性）。

4. **避免频繁拼接/修改 ID**：虽然支持拼接后缀，但应尽量减少不必要的 ID 拼接逻辑，保持 ID 简洁，避免因拼接错误导致无障碍属性关联失效。

5. **SSR 环境下的兼容性**：useId 要求 React 版本 ≥ 18，且 SSR 框架需适配 React 18 的流式渲染/水合逻辑（如 Next.js 12+、Remix 1.0+），低版本框架可能出现 ID 不匹配问题。

6. **不依赖组件状态/道具**：useId 的生成逻辑与组件的 props、state 无关，即使 props/state 变化，同一组件实例的 useId 返回值仍保持稳定。

7. **避免用于敏感数据标识**：useId 生成的 ID 仅用于前端渲染标识，不具备加密/唯一性保障（仅组件树内唯一），不可用于用户 ID、订单 ID 等敏感/全局唯一标识场景。
