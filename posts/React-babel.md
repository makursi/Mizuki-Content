---
title: Babel,SWC工具
published: 2026-04-30
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](./react/reactcute.webp)

# Babel,SWC工具介绍

对于React来说,**Babel 是 React 的 “标准编译器”，SWC 是 React 的 “高性能替代编译器”，两者核心都是把 JSX/TSX 转成浏览器能跑的 JS。**

# 什么是Babel?

Babel 是一个工具链,**主要用于将 ECMAScript 2015+ 代码转换为当前及较旧浏览器或环境中向下兼容的 JavaScript 版本.** 以下是 Babel 能为你做的主要工作：

- 语法转换：将新版本的 JavaScript 语法转换为旧版本的语法
- Polyfill：通过引入额外的代码，使新功能在旧浏览器中可用
- 源代码转换:将JSX语法转换成普通的JavaScript语法
- 插件: 为Babel提供自定义功能



*严格意义上来说,Babel,SWC应该被叫做转译器,因为它们未改变语言层级将JS高级源码->低级代码/机器码/字节码；
而转译器的工作正是将高级源码->另一种同级别高级源码，语言层级不变，只是语法降级、语法翻译，不做机器指令生成。*

    // Babel 输入: ES2015 箭头函数
    [1, 2, 3].map(n => n + 1);
    
    // Babel Output: ES5等价浏览器兼容代码
    [1, 2, 3].map(function(n) {
      return n + 1;
    });

# 功能案例

## 1. JSX语法转换React

    pnpm add --save-dev @babel/preset-react

测试代码:

    export default function DiceRoll(){
      const getRandomNumber = () => {
        return Math.ceil(Math.random() * 6);
      };
    
      const [num, setNum] = useState(getRandomNumber());
    
      const handleClick = () => {
        const newNum = getRandomNumber();
        setNum(newNum);
      };
    
      return (
        <div>
          Your dice roll: {num}.
          <button onClick={handleClick}>Click to get a new number</button>
        </div>
      );
    };

Babel编译后

    import { useState } from "react";
    import _jsx from "react/jsx-runtime";
    
    export default function DiceRoll() {
      const getRandomNumber = () => {
        return Math.ceil(Math.random() * 6);
      };
    
      const [num, setNum] = useState(getRandomNumber());
    
      const handleClick = () => {
        const newNum = getRandomNumber();
        setNum(newNum);
      };
    
      return _jsx("div", {
        children: [
          "Your dice roll: ",
          num,
          ".",
          _jsx("button", {
            onClick: handleClick,
            children: "Click to get a new number"
          })
        ]
      });
    }


# 什么是SWC?

## SWC 是一个可扩展的基于 Rust 的平台，用于下一代快速开发工具。

SWC 既可用于编译，也可用于打包。编译时，它采用现代 JavaScript 特性的 JavaScript/TypeScript 文件，输出所有主流浏览器都支持的有效代码。

## 核心优势:
*它都使用Rust了, 那就是快呗*

**SWC 在单线程上比 Babel 快 20 倍 ，四核上快 70 倍 。**

为什么这么快?

- 编译型 Rust 是一种编译型语言，在编译时将代码转化为机器码（底层的 CPU 指令）。这种机器码在执行时非常高效，几乎不需要额外的开销。

- 解释型 JavaScript 是一种解释型语言，通常在浏览器或 Node.js 环境中通过解释器运行。尽管现代的 JavaScript 引擎（如 V8 引擎）使用了 JIT（即时编译）技术来提高性能，但解释型语言本质上还是需要更多的运行时开销。

## 核心功能
1.  JavaScript/TypeScript 转换 可以将现代 JavaScript（ES6+）和 TypeScript 代码转换为兼容旧版 JavaScript 环境的代码。这包括语法转换（如箭头函数、解构赋值等）以及一些 polyfill 的处理

2.  模块打包 SWC 提供了基础的打包功能，可以将多个模块捆绑成一个单独的文件

3.  SWC 支持代码压缩和优化功能，类似于 Terser。它可以对 JavaScript 代码进行压缩，去除不必要的空白、注释，并对代码进行优化以减小文件大小，提高加载速度

4.  SWC 原生支持 TypeScript，可以将 TypeScript 编译为 JavaScript

5.  SWC 支持 React 和 JSX 语法，可以将 JSX 转换为标准的 JavaScript 代码。它还支持一些现代的 React 特性

# swc转换react jsx语法

test.jsx

    import react from 'react'
    import { createRoot } from 'react-dom/client'
    
    const App = () => {
        return <div>我喜欢开心元元</div>
    }
    
    createRoot(document.getElementById('root')).render(<App />)

转换代码

    import swc from '@swc/core'
    console.time()
    const result = swc.transformFileSync('./test.jsx', {
        jsc: {
            target: "es5", //代码转换es5
            parser: {
                syntax: 'ecmascript',
                jsx: true
            },
            transform:{
                react: {
                    runtime: 'automatic'
                }
            }
        }
    })
    console.log(result.code)
    console.timeEnd()

结果

    import { jsx as _jsx } from "react/jsx-runtime";
    import react from 'react';
    import { createRoot } from 'react-dom/client';
    var App = function() {
        return /*#__PURE__*/ _jsx("div", {
            children: "我喜欢开心元元"
        });
    };
    createRoot(document.getElementById('root')).render(/*#__PURE__*/ _jsx(App, {}));

## swc转换react,jsx语法的底层原理

1.  **Parse（解析）**
把源码字符串 → 生成 SWC 内部 AST（抽象语法树）
JSX 标签会被识别成特殊节点（不是普通字符串）

2.  **Transform（转换，核心）**
遍历 AST，遇到 JSX 节点就替换成函数调用节点
两种模式（官方配置 jsc.transform.react.runtime):

- classic：<div> → React.createElement("div", ...)
- automatic（React 17+ 默认）：<div> → _jsx("div", ...)，并自动插入 import { jsx as _jsx } from "react/jsx-runtime"

3.  **Generate（生成代码）**
把处理完的 AST → 输出普通 JS 字符串
