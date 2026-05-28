---
title: Open Graph（OG）
published: 2026-05-28
description: ''
image: ''
tags: []
category: 'SEO'
draft: false 
lang: ''
---

相关文档:https://nextjs-docs-henna-six.vercel.app/seo/openGraph

根据Facebook、微信等多个平台的官方文档及行业最佳实践，Open Graph的使用可总结如下：

### 什么是Open Graph (OG)？
Open Graph协议是由Facebook在2010年提出的一套元数据规范，它通过在网页的`<head>`中添加一系列特殊的`<meta>`标签，向社交平台提供页面的标准化信息。

### 它的主要作用是什么？
在信息呈现碎片化的社交环境中，OG协议主要扮演了“信息管家”的核心角色。

*   **掌控分享内容呈现**：它能确保你的链接在社交媒体（如Facebook、LinkedIn、微信）上被分享时，以你预设的标题、图片和描述来展示，从而**提升点击率（CTR）**。
*   **统一跨平台体验**：它提供了一套被广泛支持的标准，让链接预览在所有平台上保持一致，确保品牌形象的统一，展现专业度。
*   **间接利好SEO**：虽然OG标签不是直接排名因素，但它们能通过增加社交流量、吸引高质量外链等方式间接促进SEO。同时，其对`og:type`等属性的定义也能辅助搜索引擎理解页面内容。

### 它的核心结构和属性有哪些？
所有Open Graph标签都使用`<meta property="og:..." content="..." />`的格式，放在`<head>`标签内。其中，有四个必须包含的核心标签，通常还被建议加上`og:description`来补全预览信息。

*   **`og:title`**：分享卡片的标题。应简洁有力，一般建议控制在**60字符以内**。
*   **`og:type`**：指明内容的类型，如`article`（文章）、`website`（网站）、`product`（产品）等。
*   **`og:image`**：预览时显示的大图URL，也是提高点击率的关键。推荐尺寸**1200 x 630像素**。
*   **`og:url`**：分享内容的唯一网址（规范链接），用于防止重复内容的混乱。
*   **`og:description`**：对内容的简短描述，通常1-2句话，**155-160字符**为佳，作为卡片标题下方的补充说明。

除了这些基础属性，还有`og:site_name`（网站名称，显示在卡片底部）、`og:audio`与`og:video`（音频或视频文件的URL）、`og:locale`（语言和区域设置，如`zh_CN`表示简体中文）等可选属性来补充更多信息。

### 如何正确使用OG标签？
下面是基础实现的**完整代码示例**：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>我的博客文章</title>
    <!-- 以下是OG标签 -->
    <meta property="og:title" content="Open Graph协议完全指南：提升社交媒体点击率" />
    <meta property="og:type" content="article" />
    <meta property="og:image" content="https://www.yourdomain.com/images/og-image.jpg" />
    <meta property="og:url" content="https://www.yourdomain.com/blog/open-graph-guide" />
    <meta property="og:description" content="详细讲解Open Graph协议的定义、结构、使用方法和常见问题。" />
    <meta property="og:site_name" content="你的网站名称" />
    <!-- 可选：指定语言 -->
    <meta property="og:locale" content="zh_CN" />
</head>
<body>
    <!-- 页面内容 -->
</body>
</html>
```
所有标签都必须写在`<head>`标签内，以便社交平台的爬虫（如Facebook的`facebookexternalhit`）能直接读取。

### 常见注意事项
*   **图片**：必须使用**HTTPS绝对路径**，否则可能无法加载。避免使用SVG、WebP（部分平台兼容性不佳），优先选择**JPG或PNG格式**。设置`og:image:width`和`og:image:height`属性，以确保图片比例正确。
*   **URL**：`og:url`也必须是**绝对路径**，并与`<link rel="canonical">`保持一致，防止SEO混乱。
*   **缓存**：修改标签后，务必使用官方提供的调试工具（如**Facebook Sharing Debugger**、**LinkedIn Post Inspector**或**LINE Page Poker**）强制刷新各平台的缓存来查看最新效果。
*   **SSR**：不要在客户端使用**JavaScript动态生成**OG标签，因为大多数社交爬虫不会执行JS，导致标签缺失。务必在服务端直接渲染好（即SSR）或提供静态HTML。
*   **内容一致性**：确保`og:title/description`与页面实际内容高度相关，不可虚假夸大。但内容可不必与`<title>`和`<meta name="description">`完全相同，应根据社交传播场景进行独立优化。
*   **平台差异**：在不同平台上分享时，最终的显示细节可能有细微差异。例如，微信对图片尺寸有特定要求，而微博可能使用自己的标签体系。建议在不同平台实际测试，并研究其开发者文档以进行更精细的优化。
*   **调试**：**千万不要依赖肉眼观察**。发布前应始终通过官方调试工具和模拟爬虫命令（如使用`curl -A "facebookexternalhit/1.1"`测试）进行验证。
*   **多个标签**：确保页面中没有重复定义同一个`property`的`<meta>`标签，这会导致社交平台解析混乱，可能显示错误的链接信息。
