---
title: React受控组件和非受控组件
published: 2026-05-03
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](../react/reactcute.webp)

# 受控组件和非受控组件

## 受控组件的官方定义
在 React 中，**受控组件（Controlled Component）** 是指表单元素的状态（value、checked 等）由 React 组件的 state 完全控制，表单元素的值只能通过更新 React 状态来修改。

React 官方文档明确说明：受控组件将表单数据交由 React 状态管理，用户输入会触发状态更新，进而同步更新表单元素的值，形成“数据单向流动”的闭环。

## 非受控组件的官方定义
**非受控组件（Uncontrolled Component）** 则是表单元素的状态由 DOM 自身管理，而非 React state。React 官方将其描述为：非受控组件更接近原生 HTML 表单的工作方式，通过 ref 直接从 DOM 节点中获取表单数据，而非通过状态绑定，数据的获取时机由开发者手动触发（如提交表单时）。

## 核心作用
### 受控组件核心作用
1. **统一状态管理**：将所有表单数据纳入 React 状态体系，实现表单数据与组件状态的实时同步，便于集中处理和校验。
2. **实时数据控制**：支持实时输入校验（如输入字符长度限制、格式验证）、动态禁用/启用表单元素、数据格式化（如输入手机号自动加空格）等场景。
3. **可预测的数据流**：遵循 React “单向数据流” 原则，状态变更可追溯，便于调试和维护，符合 React 核心设计理念。

### 非受控组件核心作用
1. **简化简单场景开发**：对于无需实时校验、仅需在提交时获取数据的简单表单（如登录表单仅需最终提交账号密码），减少状态绑定的冗余代码。
2. **兼容原生 DOM 交互**：适配依赖原生 DOM 行为的场景（如文件上传 `<input type="file" />`，其 value 为只读属性，只能通过 DOM 操作获取）。
3. **迁移/集成旧代码**：在从原生 JS 或 jQuery 项目迁移到 React 时，非受控组件可降低改造成本，逐步过渡到 React 状态管理。

## 使用方法
### 受控组件使用方法
#### 核心步骤：
1. 在组件 state 中定义表单数据的初始值；
2. 给表单元素绑定 `value`（或 `checked`，针对复选框/单选框）属性，值指向 state 中的对应字段；
3. 绑定 `onChange` 事件，在事件处理函数中更新 state，实现“输入 → 事件 → 状态更新 → 表单值更新”的闭环。

#### 基础示例（输入框）：
```tsx
import { useState } from 'react';

const ControlledInput = () => {
  // 1. 初始化状态
  const [inputValue, setInputValue] = useState('');

  // 2. 处理输入变更
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setInputValue(e.target.value);
  };

  // 3. 提交处理
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('提交的值：', inputValue);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* 绑定value和onChange */}
      <input
        type="text"
        value={inputValue}
        onChange={handleChange}
        placeholder="请输入内容"
      />
      <button type="submit">提交</button>
    </form>
  );
};

export default ControlledInput;
```

#### 复选框示例：
```tsx
import { useState } from 'react';

const ControlledCheckbox = () => {
  const [isChecked, setIsChecked] = useState(false);

  const handleCheck = (e: React.ChangeEvent<HTMLInputElement>) => {
    setIsChecked(e.target.checked);
  };

  return (
    <div>
      <input
        type="checkbox"
        checked={isChecked}
        onChange={handleCheck}
      />
      <label>同意协议：{isChecked ? '是' : '否'}</label>
    </div>
  );
};
```

### 非受控组件使用方法
#### 核心步骤：
1. 使用 `useRef` 创建 DOM 引用；
2. 给表单元素绑定 `ref` 属性，将 DOM 节点挂载到 ref 对象上；
3. 在需要获取数据时（如提交表单），通过 `ref.current` 直接访问 DOM 节点的 `value`/`checked` 属性。

#### 基础示例（输入框）：
```tsx
import { useRef } from 'react';

const UncontrolledInput = () => {
  // 1. 创建ref引用DOM
  const inputRef = useRef<HTMLInputElement>(null);

  // 2. 提交时获取值
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (inputRef.current) {
      console.log('提交的值：', inputRef.current.value);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* 绑定ref，无需value和onChange */}
      <input
        type="text"
        ref={inputRef}
        defaultValue="默认值" // 非受控组件用defaultValue设置初始值
        placeholder="请输入内容"
      />
      <button type="submit">提交</button>
    </form>
  );
};

export default UncontrolledInput;
```

#### 文件上传示例（必用非受控）：
```tsx
import { useRef } from 'react';

const FileUpload = () => {
  const fileRef = useRef<HTMLInputElement>(null);

  const handleUpload = () => {
    if (fileRef.current?.files) {
      const file = fileRef.current.files[0];
      console.log('上传的文件：', file);
    }
  };

  return (
    <div>
      <input type="file" ref={fileRef} />
      <button onClick={handleUpload}>上传</button>
    </div>
  );
};
```

## 使用案例
### 受控组件典型案例：实时校验的注册表单
场景：注册表单需实时校验手机号格式、密码长度，动态提示错误信息。
```tsx
import { useState } from 'react';

const RegisterForm = () => {
  const [formData, setFormData] = useState({
    phone: '',
    password: ''
  });
  const [errors, setErrors] = useState({
    phone: '',
    password: ''
  });

  // 通用变更处理
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));

    // 实时校验
    if (name === 'phone') {
      const phoneReg = /^1[3-9]\d{9}$/;
      setErrors(prev => ({
        ...prev,
        phone: phoneReg.test(value) ? '' : '请输入正确的手机号'
      }));
    }
    if (name === 'password') {
      setErrors(prev => ({
        ...prev,
        password: value.length >= 6 ? '' : '密码长度不能少于6位'
      }));
    }
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // 最终校验
    if (!errors.phone && !errors.password && formData.phone && formData.password) {
      console.log('注册提交：', formData);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>手机号：</label>
        <input
          type="tel"
          name="phone"
          value={formData.phone}
          onChange={handleChange}
        />
        {errors.phone && <span style={{ color: 'red' }}>{errors.phone}</span>}
      </div>
      <div>
        <label>密码：</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
        />
        {errors.password && <span style={{ color: 'red' }}>{errors.password}</span>}
      </div>
      <button type="submit" disabled={!!errors.phone || !!errors.password}>
        注册
      </button>
    </form>
  );
};
```

### 非受控组件典型案例：简单留言板提交
场景：仅需在提交时获取留言内容，无需实时校验，简化代码。
```tsx
import { useRef } from 'react';

const MessageBoard = () => {
  const messageRef = useRef<HTMLTextAreaElement>(null);
  const [messages, setMessages] = useState<string[]>([]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (messageRef.current?.value.trim()) {
      setMessages(prev => [...prev, messageRef.current!.value.trim()]);
      // 清空输入框
      messageRef.current.value = '';
    }
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <textarea ref={messageRef} rows={5} cols={30} placeholder="请输入留言"></textarea>
        <button type="submit">提交留言</button>
      </form>
      <div>
        <h3>留言列表：</h3>
        <ul>
          {messages.map((msg, index) => (
            <li key={index}>{msg}</li>
          ))}
        </ul>
      </div>
    </div>
  );
};
```

## 使用注意事项
### 受控组件注意事项
1. **必须绑定 onChange**：若仅绑定 `value` 而不绑定 `onChange`，表单元素会变成“只读”状态（用户输入无法更新状态，进而无法改变 value）；
2. **区分 value/checked**：文本输入框、下拉框用 `value`，复选框/单选框用 `checked`，避免属性不匹配导致的状态不同步；
3. **处理空值与类型**：对于数字类型的输入（如年龄），需在 onChange 中转换类型（`Number(e.target.value)`），否则 state 会存储字符串，可能引发计算错误；
4. **避免过度渲染**：复杂表单若每个输入框都单独绑定 onChange，可能导致频繁渲染，可通过合并状态、使用 useCallback 优化；
5. **下拉框（select）处理**：受控 select 需给 `<select>` 绑定 `value`，而非 `<option>`，`value` 值对应 `<option>` 的 `value` 属性。

### 非受控组件注意事项
1. **初始值用 defaultValue/defaultChecked**：非受控组件不能用 `value`/`checked` 设置初始值（否则会变成受控组件），需用 `defaultValue`（输入框/文本域/下拉框）或 `defaultChecked`（复选框/单选框）；
2. **ref 判空**：访问 `ref.current` 时必须先判断是否为 null，避免 DOM 节点未挂载时出现 TypeError；
3. **避免混合使用受控/非受控模式**：同一表单元素不能同时绑定 `value` 和 `ref`（既受控又非受控），React 会抛出警告，且状态管理会混乱；
4. **文件输入框的特殊性**：`<input type="file" />` 是只读的受控组件（无法通过 value 设置值），必须用非受控方式通过 ref 获取文件；
5. **数据更新时机**：非受控组件的状态存储在 DOM 中，若需重置表单，需手动修改 `ref.current.value`，而非通过 React 状态；
6. **性能考量**：非受控组件减少了状态更新的渲染，但大规模表单中手动管理 ref 可能增加代码复杂度，需权衡。

### 受控 vs 非受控 选型建议（官方推荐）
- **优先使用受控组件**：符合 React 状态管理理念，适合需要实时交互、校验、动态控制的表单场景；
- **使用非受控组件的场景**：
  - 简单表单（仅提交时获取数据）；
  - 依赖原生 DOM 行为的元素（如文件上传）；
  - 集成第三方 UI 库且无法适配受控模式的场景；
  - 快速原型开发，追求极简代码。
