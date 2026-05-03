---
title: React底层原理-第二章
published: 2026-04-30
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---

![Alt text](./react/reactcute.webp)

# 1. requestIdleCallback

requestIdleCallback 是浏览器原生 API,作用是 **在浏览器主线程「空闲时段」执行低优先级任务**

**参数:**
    requestIdleCallback(callback, options)

**callback:**

空闲时要执行的回调函数，回调会接收 **唯一入参：IdleDeadline 对象**

**options:**

timeout:毫秒
- 含义:超时时间
- 如果超过 timeout 毫秒浏览器一直没空,**强制立即执行回调**

**返回值:**
- 返回一个数字ID(唯一标识符)
- 作用:用于取消任务
- 取消API

    // 取消空闲任务
    cancelIdleCallback(handle)

### 2. IdleDeadline
requestIdleCallback的回调函数唯一参数

    interface IdleDeadline {
      // 剩余空闲时间（只读）
      timeRemaining(): number 
      // 标记：是否因为超时强制执行
      readonly didTimeout: boolean 
    }

**1. timeRemaining()**
- 返回：当前帧剩余空闲时间（单位 ms）
- 规则：最大限制 50ms
- 使用：在任务里循环判断，时间不够就暂停，下次空闲再继续

**2. didTimeout**

boolean:
- true：任务是因为 timeout 超时被强制执行
- false：正常空闲触发

# React不使用原生requestIdleCallback的原因

**1. 兼容性差**
实验性 API，Safari 完全不支持,行为可能随浏览器版本变化，无法跨环境稳定运行。

**2. 调度粒度太粗，帧率不达标**

原生设计为低优先级后台任务（如日志上报），浏览器限制其回调执行间隔约 **50ms（20FPS）**，远低于 **UI渲染所需的16.7ms(60FPS)** 基准，无法支撑流畅的界面更新。

**3. 差异性**

每个浏览器实现该API的方式不同，导致执行时机有差异有的快有的慢

# 替代方案-MessageChannel

MessageChannel 是浏览器原生异步通信 API

异步得是个宏任务,宏任务中会在下次事件循环中执行，不会阻塞当前页面的更新。

创建两个互相连通的端口：port1 / port2
    
    // 1. 创建消息通道
    const channel = new MessageChannel();
    const port2 = channel.port2;
    
    // 2. 监听 port2 收到消息
    channel.port1.onmessage = function () {
      // 这里就是异步回调，React 在这里执行下一段 Fiber 任务
      console.log("执行分片任务");
    };
    
    // 3. 主动发消息，触发异步执行
    port2.postMessage(null);


React通过创建消息通道实例，利用其异步宏任务特性高频、稳定地触发回调，在回调中执行workLoop处理 Fiber 任务，同时主动监控时间片（默认 5ms）

时间耗尽便通过postMessage让出主线程并发起下一次调度，既实现了可中断的任务分片，又能紧跟浏览器 60fps 渲染节奏，保证 UI 更新流畅且不阻塞交互。
