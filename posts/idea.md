---
title: IntelliJ IDEA 工作区简单介绍
published: 2026-04-02
description: ''
image: ''
tags: []
category: 'JavaScript'
draft: false 
lang: ''
---

## 项目根目录

```
Java-start/
└── C:\Users\29634\Desktop\W\apps\Java-start
```

代码、配置文件、依赖等都放在此目录下。

---

## `.idea` 文件夹

IDEA 自动生成的配置目录，记录项目如何运行。

### 保存的内容

- 项目设置
- JDK 版本
- 模块信息
- 运行配置
- 编译配置
- IDE 工作区信息

### 主要文件

| 文件 | 作用 |
|------|------|
| `misc.xml` | 项目使用的 JDK 版本、语言级别 |
| `workspace.xml` | IDEA 状态（打开的文件、窗口布局、光标位置等） |
| `modules.xml` | 项目包含的模块 |

#### modules.xml 示例（多模块项目）

```
商城系统
├── user-module
├── order-module
└── payment-module
```

---

## 外部库

IDEA 认为该项目可以使用的外部 Java 库，例如 `import java.util.ArrayList`。

---

## 临时文件和控制台

IDEA 临时文件及控制台输出相关。
