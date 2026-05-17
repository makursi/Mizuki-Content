---
title: nginx入门学习
published: 2026-05-17
description: ''
image: ''
tags: ['']
category: 'Nginx'
draft: false 
lang: ''
---

# 使用将react+ts项目部署到nginx服务器,


## 使用vite创建react应用

```bash
npm create vite@latest
```

## 使用构建命令进行打包

```bash
npm run build
```


## 进行nginx配置

```nginx
#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}
  
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       8080;
        server_name  localhost; #如果有真实域名,就填写真实域名

        # 真实的 dist 目录位置
        root (你的SPA应用项目dist目录);
        index index.html;

        # 设置条件, 如果
        location / {
            try_files $uri $uri/ /index.html;
        }

        # 作缓存,提示资源访问性能
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2?|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
}

```

## 部署

首先在nginx目录中,使用nginx相关命令: 

1. 测试配置

```bash
nginx -t
```

2.  重新加载配置

```bash
nginx -s reload 
```

3.  启动nginx服务:

```bash
nginx
```

4.  停止nginx服务:

```bash
nginx -s stop
```

## 访问

在web中通过域名进行访问,一般是localhost:8080
