---
title: css-in-js
published: 2026-05-03
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](./react/reactcute.webp)

# React中css-in-js的官方定义
css-in-js是一种将CSS样式直接编写在JavaScript代码中的技术范式，在React生态中，它并非由React官方直接定义的规范，但React官方文档认可并提及这种方案是React应用中管理样式的主流方式之一。从社区与行业权威定义来看，css-in-js本质是通过JavaScript来封装样式逻辑，将样式作为组件的一部分，以JavaScript对象或模板字符串的形式声明样式，再由对应的库（如Styled Components、Emotion等）将其转换为实际的CSS样式并注入到DOM中，实现样式与组件的紧密耦合。

# css-in-js的核心作用
1. **组件样式隔离**：天然实现样式的组件级隔离，避免传统CSS的类名冲突问题，每个组件的样式作用域仅限定于自身，无需手动命名空间或BEM等命名规范来规避冲突。
2. **动态样式便捷性**：可直接利用JavaScript的变量、函数、条件判断等逻辑动态生成样式，轻松实现基于组件状态（如hover、active、自定义props）的样式变化，无需额外操作DOM或拼接类名。
3. **样式复用与组合**：能像复用JavaScript函数/对象一样复用样式片段，支持样式继承、组合，配合React组件的复用特性，提升样式代码的可维护性。
4. **与React组件生命周期适配**：样式的创建、更新、销毁可与React组件的生命周期同步，组件卸载时对应的样式也可自动清理，减少无用样式堆积。
5. **TypeScript友好性**：主流css-in-js库（如Emotion、Styled Components）均提供完善的TypeScript类型定义，可在编写样式时获得类型提示与校验，降低样式编写错误。

# css-in-js的使用方法
## 主流库：Styled Components（最常用）
### 1. 安装
```bash
npm install styled-components
# 若使用TypeScript，安装类型声明
npm install @types/styled-components --save-dev
```

### 2. 基础使用（创建样式化组件）
```tsx
import styled from 'styled-components';
import React from 'react';

// 定义样式化组件
const Button = styled.button`
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  background-color: #1677ff;
  color: white;
  cursor: pointer;

  // 伪类样式
  &:hover {
    background-color: #4096ff;
  }

  // 接收props动态样式
  ${(props) => props.disabled && `
    background-color: #cccccc;
    cursor: not-allowed;
  `}
`;

// 业务组件中使用
const App = () => {
  const [disabled, setDisabled] = React.useState(false);
  return (
    <div>
      <Button disabled={disabled}>普通按钮</Button>
      <Button onClick={() => setDisabled(!disabled)}>切换禁用状态</Button>
    </div>
  );
};

export default App;
```

## 主流库：Emotion（性能更优，React官方示例推荐）
### 1. 安装
```bash
npm install @emotion/react @emotion/styled
# TypeScript无需额外安装类型，内置支持
```

### 2. 基础使用（两种写法）
#### 写法1：styled组件式
```tsx
import styled from '@emotion/styled';
import React from 'react';

const Card = styled.div<{ width: number }>`
  width: ${(props) => props.width}px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  border-radius: 8px;
`;

const App = () => {
  return <Card width={300}>这是一个Emotion样式的卡片</Card>;
};
```

#### 写法2：css函数式（内联样式增强）
```tsx
import { css } from '@emotion/react';
import React from 'react';

const textStyle = css`
  font-size: 14px;
  color: #333;
`;

const dynamicColorStyle = (color: string) => css`
  color: ${color};
`;

const App = () => {
  return (
    <div>
      <p css={textStyle}>基础文本样式</p>
      <p css={dynamicColorStyle('#f50')}>动态颜色文本</p>
    </div>
  );
};
```

# css-in-js的实际业务场景使用案例
## 场景1：表单组件动态样式（根据校验状态变化）
```tsx
import styled from 'styled-components';
import React, { useState } from 'react';

const Input = styled.input<{ isValid: boolean }>`
  width: 300px;
  padding: 8px;
  border: 1px solid ${(props) => (props.isValid ? '#d9d9d9' : '#ff4d4f')};
  border-radius: 4px;
  outline: none;

  &:focus {
    border-color: ${(props) => (props.isValid ? '#1677ff' : '#ff4d4f')};
    box-shadow: 0 0 0 2px ${(props) => (props.isValid ? 'rgba(22, 119, 255, 0.2)' : 'rgba(255, 77, 79, 0.2)')};
  }
`;

const Form = () => {
  const [value, setValue] = useState('');
  const [isValid, setIsValid] = useState(true);

  const validate = (v: string) => {
    // 简单校验：非空且长度≥6
    setIsValid(v.trim() !== '' && v.length >= 6);
  };

  return (
    <div>
      <Input
        value={value}
        onChange={(e) => {
          setValue(e.target.value);
          validate(e.target.value);
        }}
        isValid={isValid}
        placeholder="请输入6位以上字符"
      />
      {!isValid && <p style={{ color: '#ff4d4f', fontSize: 12 }}>输入内容不符合要求</p>}
    </div>
  );
};

export default Form;
```

## 场景2：主题切换（全局样式动态切换）
```tsx
import { ThemeProvider } from 'styled-components';
import styled from 'styled-components';
import React, { useState } from 'react';

// 定义明暗主题
const lightTheme = {
  bg: '#ffffff',
  text: '#333333',
  buttonBg: '#1677ff',
};

const darkTheme = {
  bg: '#1f1f1f',
  text: '#ffffff',
  buttonBg: '#4096ff',
};

// 基于主题的样式组件
const Container = styled.div`
  width: 100vw;
  height: 100vh;
  background-color: ${(props) => props.theme.bg};
  color: ${(props) => props.theme.text};
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
`;

const ThemeButton = styled.button`
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  background-color: ${(props) => props.theme.buttonBg};
  color: white;
  cursor: pointer;
  margin-top: 20px;
`;

const ThemeSwitch = () => {
  const [isDark, setIsDark] = useState(false);

  return (
    <ThemeProvider theme={isDark ? darkTheme : lightTheme}>
      <Container>
        <h1>当前主题：{isDark ? '深色' : '浅色'}</h1>
        <ThemeButton onClick={() => setIsDark(!isDark)}>
          切换{isDark ? '浅色' : '深色'}主题
        </ThemeButton>
      </Container>
    </ThemeProvider>
  );
};

export default ThemeSwitch;
```

# css-in-js的使用注意事项
1. **性能开销**：
   - css-in-js库运行时会解析样式并注入DOM，首次渲染和样式频繁更新时可能产生性能损耗，尤其是大型应用；可通过缓存样式、减少动态样式的频繁变更缓解。
   - 服务端渲染（SSR）场景下，需额外配置（如Styled Components的`ServerStyleSheet`）避免样式闪烁，否则可能出现客户端样式不匹配问题。

2. **调试成本**：
   - 生成的CSS类名是哈希值（如`sc-bdvvtL`），调试时难以直接对应到源码中的样式定义；需开启开发环境的类名映射（如Styled Components的`displayName`配置），提升调试体验。

3. **包体积**：
   - 引入css-in-js库会增加应用包体积（如Styled Components约15KB gzip后），小型应用需权衡是否有必要引入，简单场景可考虑CSS Modules替代。

4. **样式优先级**：
   - css-in-js生成的样式通过`style`标签注入，优先级高于普通CSS文件，可能覆盖全局样式；需避免过度依赖优先级，通过合理的样式结构管理。

5. **TypeScript类型约束**：
   - 自定义props传递给样式组件时，需显式声明类型（如`styled.button<{ disabled: boolean }>`），否则会丢失类型校验；确保安装对应库的类型声明文件。

6. **兼容性**：
   - 部分css-in-js特性（如CSS变量结合动态样式）依赖现代浏览器，如需兼容低版本浏览器（如IE11），需搭配postcss等工具做降级处理。

7. **避免过度封装**：
   - 不要将所有样式都通过css-in-js编写，全局通用样式（如重置样式、基础布局）仍建议用普通CSS/SCSS文件管理，仅对组件级、动态样式使用css-in-js。
