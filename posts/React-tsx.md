---
title: React开发环境搭建
published: 2026-04-30
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](./react/reactcute.webp)

# TSX语法介绍

## 定义:TSX是 JSX + TypeScript 的语法扩展，
JSX 是 React 声明式 UI 语法，TSX 在此基础上增加类型约束

## 核心概念

1.  语法本质：类 HTML 标签语法，会被编译为React.createElement；

2.  单一根节点：组件 TSX 必须包裹在一个根标签 / 空标签 <></> 内。

3.  JS 嵌入：通过 { } 嵌入变量、表达式、逻辑；(别和Vue的模板渲染搞混了 {{}} )

4.  类型约束：可给组件 props、状态、元素属性标注类型，杜绝类型错误；

*Vue: 你(TSX)讲话挺有趣的,混哪里的啊*

## TSX的核心作用如下:
1.  视图与逻辑一体化
2.  原生支持 TS 静态类型检查，提前拦截报错；

## 快速上手使用

    // 1. 定义props类型
    type MsgProps = {
      text: string
    }
    
    // 2. TSX组件+类型约束
    const Hello = (props: MsgProps) => {
      const count: number = 10
      // 3. {} 嵌入JS变量/表达式
      return (
        <div>
          <h1>{props.text}</h1>
          <p>数字：{count + 1}</p>
        </div>
      )
    }
    
    // 使用组件
    export default function App() {
      return <Hello text="Hello TSX" />
    }

语法编写细节:https://message163.github.io/react-docs/react/basic/tsx.html
