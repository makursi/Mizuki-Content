---
title: robots.txt
published: 2026-05-28
description: ''
image: ''
tags: []
category: 'SEO'
draft: false 
lang: ''
---

官方文档:https://developers.google.com/search/docs/crawling-indexing/robots/intro?hl=zh-cn

# robots.txt

robots.txt 文件规定了搜索引擎抓取工具可以访问您网站上的哪些网址。 此文件主要用于避免您的网站收到过多请求；**它并不是一种阻止 Google 抓取某个网页的机制。**

若想阻止 Google 访问某个网页，请**使用 noindex 禁止将其编入索引，或使用密码保护该网页。**

参数说明:

**User-agent**

user-agent是搜索引擎爬虫的名称，例如`Googlebot`，`Baiduspider`，`Bingbot`，`YandexBot`，`Sogou spider`，`Yahoo! Slurp`，`BingPreview`等，也可以直接使用`*`表示所有搜索引擎爬虫都可以访问。

**Disallow**

disallow是搜索引擎爬虫不能访问的页面，例如/admin/，/api/，/login/，/logout/等。

**Allow**

allow是搜索引擎爬虫可以访问的页面，例如/，/about/，/contact/等。

**Crawl-delay**

crawl-delay是搜索引擎爬虫访问网站的间隔时间，例如10，表示搜索引擎爬虫访问网站的间隔时间为10秒。

> 注意: Google机器人不支持该参数，其他部分爬虫机器人支持该参数

**Sitemap**

sitemap是网站地图的URL，例如`https://www.某某某.com/sitemap.xml。`

**Host**

host是网站的域名，例如`https://www.某某某.com。`

# Nextjs中实现robots.txt
Next.js中实现robots.txt非常简单，我们是`AppRouter`，所以直接在`app`目录下创建一个`robots.[ts | js]`文件即可。

    import type { MetadataRoute } from "next";
    
    export default function robots(): MetadataRoute.Robots {
      return {
      /*  如果是通用规则，可以这样写，就直接是一个对象类似于掘金
               rules: {
                  userAgent: '*',
                  allow: '/',
                  disallow: '/private/',
                },
              自定义爬虫机器人规则可以用数组形式，就是一个数组类似于哔哩哔哩 */
        rules: [
          {
            userAgent: "Googlebot",//搜索引擎爬虫名称
            allow: ["/"],//允许访问界面
            disallow: ["/api/"],//不允许访问界面
            crawlDelay: 10, //访问间隔时间(Google机器人不支持该参数，其他部分爬虫机器人支持该参数)
          },
          {
            userAgent: "Baiduspider",
            allow: ["/"],
            disallow: ["/api/"],
            crawlDelay: 10,
          },
          {
            userAgent: "Bingbot",
            allow: ["/"],
            disallow: ["/api/"],
            crawlDelay: 10,
          },
          {
            userAgent: "YandexBot",
            allow: ["/"],
            disallow: ["/api/"],
            crawlDelay: 10,
          },
          {
            userAgent: "Sogou spider",
            allow: ["/"],
            disallow: ["/api/"],
            crawlDelay: 10,
          },
        ],
        sitemap: "https://choria.example.com/sitemap.xml", //网站地图url,有多个可以写成数组
        host: "https://choria.example.com",
      };
    }

保存,并在开发环境运行后,就会添加robots.txt
