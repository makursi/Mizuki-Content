---
title: SEO-TDK
published: 2026-05-28
description: ''
image: ''
tags: []
category: 'SEO'
draft: false 
lang: ''
---

# TDK + meta

TDK是Title,Description,Keyword的缩写,是 SEO（搜索引擎优化）里的核心元信息，也常统称为页面的**元数据**。

在原生 HTML 里，它们大致对应 `<head>` 中的 `<title>`、`<meta name="description">`、`<meta name="keywords">` 等。

使用 App Router 时，Next.js 通过 `export const metadata` 或 `generateMetadata` 生成上述标签，由框架写入文档头部，无需手写整段 `<head>`。

## 什么是元数据?

元数据就是“关于数据的数据”，用大白话讲就是：给数据本身打标签、做说明的信息. 

在生活中,一本书的内容是数据,那这本书的书名、作者、出版日期、页数、目录就是这本书的元数据。

## 使用有效的 HTML 指定网页元数据

1.  **我们必须按照 HTML 的规则来写元数据的代码。**写对了，Google 才能正确读取并使用这些信息（比如用标题和描述展示搜索结果）。如果写错了，Google 可能看不懂，甚至忽略。

2.  所有元数据都要放在 `<head>` 标签里面（而不是 `<body>` 里）。`<head>` 就是专门放“页面的说明信息”的地方。

3.  如果我们在`<head>`标签中使用了无效元素,Google会忽略该无效元素之后的所有元素

`<head>`元素中的有效元素:
- title
- meta
- link
- script
- style
- base
- noscript
- template

`<head>`元素中的无效元素:
- iframe
- img

# TDK的作用

title

`title` 是页面标题，通常会出现在浏览器标签页和搜索引擎结果页（SERP）上，**对点击率影响最大。** 建议简洁、准确，并体现当前页与站点/栏目的关系（例如与根布局的 `title.template` 搭配使用，见下文）。

description

`description` 是页面摘要,常被用作 SERP 中的描述文案（搜索引擎也可能根据内容自行改写）。应用一两句话概括页面价值，避免堆砌关键词。

keywords
`keywords` 用于概括页面主题。主流搜索引擎对 `<meta name="keywords">` 的排序权重已很低，但*规范填写*仍有利于内部归类、CMS 或后续扩展；不要为 SEO 而重复、堆砌无意义词组。

## 在Nextjs中如何配置

    export const metadata: Metadata = {
      metadataBase: new URL("https://choria.example.com"),
      title: {
        default: "Choria - AI-Powered Assistant Platform",
        template: "%s | Choria",
      },
      description:
        "一站式 AI 智能平台，整合 DeepSeek 驱动的对话助手与宝可梦图鉴等实用工具", //description 要简洁描述
      keywords: [
        "AI",
        "assistant",
        "chat",
        "DeepSeek",
        "pokemon",
        "智能助手",
        "AI对话",
      ], // 关键词添写,但不能恶意堆彻
      authors: [{ name: "Choria" }],
      robots: {
        index: true,
        follow: true,
        googleBot: {
          index: true,
          follow: true,
          "max-video-preview": -1,
          "max-image-preview": "large",
          "max-snippet": -1,
        },
      },
      openGraph: {
        title: "Choria - AI-Powered Assistant Platform",
        description:
          "一站式 AI 智能平台，整合 DeepSeek 驱动的对话助手与宝可梦图鉴等实用工具",
        type: "website",
        siteName: "Choria",
        locale: "zh_CN",
      },
      twitter: {
        card: "summary_large_image",
        title: "Choria - AI-Powered Assistant Platform",
        description:
          "一站式 AI 智能平台，整合 DeepSeek 驱动的对话助手与宝可梦图鉴等实用工具",
      },
      alternates: {
        canonical: "/",
      },
    };
