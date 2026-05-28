---
title: SEO-web vitals
published: 2026-05-28
description: ''
image: ''
tags: []
category: 'SEO'
draft: false 
lang: ''
---

---
# Web Vitals

Web Vitals（网页性能指标）是 Google 提出的一套标准化方案，用于量化并衡量一个网页的真实用户体验。它可以帮助你系统性地评估和优化网站的性能。

### Web Vitals 的三大“体检”指标

Google 将影响用户体验的核心指标归纳为三点：加载、交互和视觉稳定。这套核心指标也被称为 Core Web Vitals。

- **最大内容绘制 (Largest Contentful Paint, LCP)**：测量加载性能。

指页面视口中最大的内容元素（如主角图片、标题等）何时完成渲染。建议值：在页面首次开始加载后的 2.5 秒内发生。


- **首次输入延迟 (First Input Delay, FID)**：测量交互性。

指用户第一次与页面互动（例如点击链接或按钮）时，浏览器能够实际开始处理事件处理程序的时间。建议值：小于 100 毫秒。


- **累积布局偏移 (Cumulative Layout Shift, CLS)**：测量视觉稳定性。

指页面内容在加载过程中发生的意外偏移的程度。例如，当你正要点击一个链接时，它突然被上方加载出来的图片挤开了。建议值：保持在 0.1 或更少。

#### 关于指标演变的小提示

需要注意的是，从 2024 年起，Google 官方宣布 FID 将逐步被一个更全面的指标——交互到下一次绘制 (Interaction to Next Paint, INP) 所取代。INP 会观察页面与用户的所有交互的延迟，而不仅仅是第一次。

### 如何测评 Web Vitals？

测评可以分为两类，各有侧重：

- **实验室数据（模拟工具）**：用于在开发、测试环境中模拟各种设备（如手机）、网络状况（如 4G），帮助你在代码上线前发现并定位性能瓶颈。这类工具包括：Chrome DevTools 开发者工具的 Lighthouse、PageSpeed Insights（提供综合诊断报告）以及 WebPageTest。只需提供一个 URL，Google 的 Lighthouse 工具就能自动运行数十项检查并生成报告，给出详细的改进建议。


- **实际数据（真实监控）**：用于采集真实用户 (Real User Monitoring，RUM) 在各种网络和设备上的访问数据，以了解性能对真实业务的影响。获取方式包括 Google Search Console（“核心网页指标”报告可查看网站所有页面的实际数据）、浏览器插件（如 Web Vitals 扩展）或第三方监控平台（如 DebugBear）。

### Next.js 代码实现 (useReportWebVitals)

Next.js 提供了一个非常方便的内部 Hook——`useReportWebVitals`，可以轻松地将性能数据发送到你的数据分析平台。

以下是如何使用它的基本步骤：

1. **创建一个客户端组件**：在 `app/_components/web-vitals.tsx` 新建文件，用于处理性能数据的上报。
   ```tsx
   // app/_components/web-vitals.tsx
   'use client';
    
   import { useReportWebVitals } from 'next/web-vitals';
    
   // 上报函数：你可以将 metric 对象发送到 Google Analytics、Plausible 等平台
   function sendToAnalytics(metric: any) {
     // 打印到控制台，实际使用时应替换为你的服务商API
     console.log(metric);
   }
    
   export function WebVitals() {
     useReportWebVitals((metric) => {
       // 根据指标名称进行不同处理
       switch (metric.name) {
         case 'FCP':
           // 处理首次内容绘制
           break;
         case 'LCP':
           // 处理最大内容绘制
           break;
         case 'CLS':
           // 处理累计布局偏移
           break;
         case 'INP':
           // 处理交互到下次绘制 (未来核心指标)
           break;
         default:
           break;
       }
       sendToAnalytics(metric);
     });
     return null; // 此组件不渲染任何UI
   }
   ```
   `useReportWebVitals` Hook 必须配合 `'use client'` 指令使用。最佳实践是像上面这样创建一个独立的组件，而不是直接写在布局里。

2. **在根布局中引入**：打开你的根布局文件 `app/layout.tsx`，在文件顶部导入并引入 `<WebVitals />` 组件，确保它能收集到所有页面的数据。
   ```tsx
   // app/layout.tsx
   import { WebVitals } from './_components/web-vitals';
    
   export default function RootLayout({ children }: { children: React.ReactNode }) {
     return (
       <html lang="zh-CN">
         <body>
           {children}
           <WebVitals /> {/* 放置在这里即可 */}
         </body>
       </html>
     );
   }
   ```

这个 `metric` 对象包含了 `name` (指标名称，如 `LCP`， `CLS`)， `value` (指标数值)， `rating` (评级: `'good'`， `'needs-improvement'`， `'poor'`) 等关键信息，你可以根据这些信息进行后续的数据分析或预警。

> 小提示：`useReportWebVitals` 在 Next.js 14.1.0 及更高版本中可用。

### 快速启动建议

优化 Web Vitals 是一个持续的过程。建议可以按以下思路来推进：

1. **测量基准**：用 Lighthouse 跑一下你的核心页面，拿到第一份性能报告。
2. **采集实际数据**：在项目中接入 `useReportWebVitals`（比如发送到 Google Analytics），开始收集真实用户的数据，了解线上体验。
3. **聚焦核心指标**：现在优先关注 LCP 和 CLS。使用 `next/image` 优化图片，避免图片加载后撑开容器导致布局偏移。
4. **迭代优化**：根据 Lighthouse 提供的改进建议和真实用户数据，持续优化。
