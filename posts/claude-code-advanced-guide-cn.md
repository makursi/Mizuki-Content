---
title: Claude Code指南
published: 2026-05-18
description: ''
image: ''
tags: []
category: 'Claude code'
draft: false 
lang: ''
---

博客总结自b站up主[Yin_Code](https://space.bilibili.com/3706940936948128?spm_id_from=333.788.upinfo.head.click)

相关视频:

<iframe src="//player.bilibili.com/player.html?isOutside=true&aid=116533082852451&bvid=BV1SddcBFESs&cid=38162532217&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="468"></iframe>

---

# 什么是 Claude Code

Claude Code 一个终端 AI 编程助手。

它本质上是：

- 一个运行在 Terminal / Shell 中的 AI Agent
- 能读取项目代码
- 能修改文件
- 能执行命令
- 能分析项目结构
- 能进行重构
- 能调用工具
- 能通过上下文持续协作开发

它与传统 Chat AI 最大的区别：

| 普通 AI Chat | Claude Code |
|---|---|
| 只能聊天 | 能直接操作项目 |
| 用户复制代码 | AI 直接读取代码库 |
| 无法执行命令 | 能执行 shell 命令 |
| 纯问答 | Agent 工作流 |
| 短期上下文 | 长时间工程上下文 |


---

# Claude Code 的基本启动方式

通常通过：

```bash
claude
```

启动。

进入项目目录后：

```bash
cd my-project
claude
```

Claude 会自动读取：

- 当前目录
- Git 状态
- 文件结构
- package.json
- 配置文件
- 代码内容

然后开始进入工程上下文。

---

# 1. 开启 Plan Mode（计划模式）

## 什么是 Plan Mode

Plan Mode 是 Claude Code 中非常重要的工作模式。

它的核心思想是：

> 先规划，再执行。

而不是：

> 用户一句话 → AI 直接开始乱改代码。

这是 Claude Code 与很多 AI 编程工具最大的差异之一。

---

## 为什么需要 Plan Mode

大型项目开发中：

如果 AI 不先规划：

可能会出现：

- 修改错误文件
- 架构混乱
- 重复逻辑
- 状态管理冲突
- 类型定义错乱
- API 不一致
- 组件职责污染

所以 Claude 官方非常强调：

> 复杂任务必须先做 planning。

---

## Plan Mode 适合什么场景

特别适合：

### 1. 大型重构

例如：

- React → Next.js
- Redux → Zustand
- JS → TS
- REST → GraphQL

---

### 2. 新增大型功能

例如：

- 登录系统
- 支付系统
- AI Chat 模块
- 权限系统
- RAG 系统

---

### 3. 多文件修改

例如：

- 修改 API Schema
- 修改数据库结构
- 修改组件通信

---

## 推荐 Prompt

```text
进入 plan mode，先分析整个项目架构。
不要立即修改代码。
先给出：
1. 当前问题
2. 影响范围
3. 修改计划
4. 风险点
5. 推荐实现方案
```

---

## 高级技巧

你可以要求 Claude：

```text
请像 Tech Lead 一样先做架构设计。
```

或者：

```text
先不要写代码，只做技术方案。
```

这会显著提高代码质量。

---

## 注意事项

### 不要一上来就让 AI 自动改大型项目

很多人错误用法：

```text
帮我重构整个项目
```

然后 Claude 开始疯狂改文件。

这是非常危险的。

正确做法：

- 先 plan
- 再确认
- 再执行

---

# 2. 使用 /rewind 回滚操作

## /rewind 是什么

Claude Code 会记录会话中的操作历史。

/rewind 可以：

- 回退 AI 做过的修改
- 回到之前状态
- 撤销错误操作

类似：

- Git Reset
- 时间回溯
- 会话 Undo

---

## 基本使用

```bash
/rewind
```

Claude 会显示：

- 可回退节点
- 历史修改记录
- 修改时间

你可以选择回退。

---

## 推荐场景

### AI 改坏项目时

例如：

- 类型炸了
- 项目跑不起来
- 样式崩溃
- AI 大面积误修改

---

### Prompt 写错时

例如：

```text
帮我删除所有没用的代码
```

结果把核心逻辑删了。

此时 rewind 非常重要。

---

## 官方推荐

Claude 官方非常建议：

> Claude Code 必须结合 Git 使用。

因为：

/rewind 不是完整版本控制系统。

最安全组合：

```bash
git commit
+ Claude Code
+ /rewind
```

---

## 注意事项

### rewind命令只能回滚claude code使用内置工具创建的文件和文件夹,不能回滚使用shell命令的执行结果



rewind不能替代git,不能使用rewind命令去执行以下操作：

- Git Branch
- Commit History
- PR 审查
- Merge 管理

生产项目必须使用 Git。

---

# 3. 使用 ! 开启 Shell Mode

## 什么是 Shell Mode

Claude Code 可以直接执行 shell 命令。

例如：

```bash
!npm run dev
```

或者：

```bash
!pnpm build
```

---

## 本质

! 相当于：

> 告诉 Claude：
> “去终端执行命令。”

---

## 常见使用方式

### 安装依赖

```bash
!pnpm add zustand
```

---

### 启动项目

```bash
!npm run dev
```

---

### 查看目录

```bash
!ls
```

---

### Git 操作

```bash
!git status
```

---

### 执行测试

```bash
!pnpm test
```

---

## 高级玩法

Claude 可以：

- 分析命令输出
- 自动修复报错
- 自动安装缺失依赖
- 自动定位错误文件

例如：

```text
运行 build 并修复所有 TS 报错
```

Claude 会：

- 执行 build
- 读取错误
- 修改代码
- 再次 build
- 循环修复

这就是 Agent Workflow。

---

## 注意事项

### 不要随便执行危险命令

例如：

```bash
!rm -rf /
```

或者：

```bash
!sudo ...
```

Claude 虽然会请求权限。

但：

你仍然需要自己负责。

---

# 4. 使用 /effort 深度计划模式

## /effort 是什么

这是 Claude Code 的深度思考模式。

它会：

- 使用更多上下文
- 更长时间分析
- 更复杂规划
- 更深入推理

适合：

- 大型重构
- 架构设计
- 性能优化
- 多系统协作

---

## 推荐场景

### Monorepo 改造

### 微前端

### AI Agent 系统

### 数据库迁移

### Next.js App Router 重构

---

## 示例 Prompt

```text
/effort
请深度分析当前项目架构问题。
给出企业级重构方案。
```

---

## ctrl+j

用于换行。

因为很多复杂 Prompt：

- 很长
- 多段
- 需要结构化描述

例如：

```text
我要你：
1. 分析目录结构
2. 分析状态管理
3. 分析 API 层
4. 给出优化方案
```

---

## ctrl+g

通常用于：

- 编辑 Prompt
- 修改输入内容
- 进入文本编辑状态

适合写大型 Prompt。

---

## 注意事项

/effort 会消耗更多 token。

意味着：

- 成本更高
- 响应更慢
- 上下文占用更大

不要小任务也一直用。

---

# 5. claude --dangerously-skip-permission

## 作用

```bash
claude --dangerously-skip-permission
```

会跳过权限确认。

Claude 执行：

- 文件修改
- Shell 命令
- 删除文件
- 安装依赖

时不再询问。

---

## 为什么危险

因为 Claude 本质是 Agent。

它会：

- 自动执行操作
- 自动修改项目
- 自动运行命令

如果 Prompt 出错：

可能：

- 删除文件
- 覆盖代码
- 安装恶意依赖
- 修改环境配置

---

## 官方态度

Anthropic 官方也明确：

> 不推荐默认开启。

尤其：

- 公司项目
- 生产环境
- 重要代码库

---

## 推荐使用场景

仅适合：

- 沙盒环境
- 个人实验项目
- 临时 Demo
- 完全可回滚项目

---

## 最佳实践

不要长期使用：

```bash
--dangerously-skip-permission
```

推荐：

- Git commit
- 分支开发
- 小步修改
- 人工 Review

---

# 6. /exit 与 /resume

## /exit

退出 Claude Code。

```bash
/exit
```

---

## resume

恢复之前会话：

```bash
/resume
```

Claude 会恢复：

- 上下文
- 会话历史
- 当前开发任务

---

## 为什么重要

大型开发往往：

- 持续几天
- 多轮协作
- 多次中断

resume 可以保持上下文连续性。

---

## 注意事项

### 上下文不是无限的

Claude 依然有 Context Window 限制。

超长会话：

- 会丢失早期上下文
- 推理质量下降
- 出现幻觉

所以：

长项目建议：

- 定期总结
- 使用 CLAUDE.md
- 使用 Git Commit

---

# 7. /init 初始化 CLAUDE.md

## 什么是 CLAUDE.md

这是 Claude Code 的项目记忆文件。

类似：

- Cursor Rules
- Copilot Instructions
- AI Project Memory

Claude 会长期读取它。

---

## 初始化方式

```bash
/init
```

Claude 会生成：

```md
# Project Overview
# Coding Standards
# Architecture
# Commands
# Rules
```

---

## CLAUDE.md 的核心作用

它是：

> 给 AI 的项目说明书。

---

## 推荐内容

### 技术栈

```md
- Next.js 16
- TypeScript
- Tailwind
- Prisma
- PostgreSQL
```

---

### 代码规范

```md
- 使用函数式组件
- 禁止 any
- 使用 Server Components 优先
```

---

### 架构规范

```md
- app/api 只处理 HTTP
- service 层处理业务逻辑
- db 层只负责数据库
```

---

### UI 规范

```md
- 使用 shadcn/ui
- 使用 Tailwind
- 响应式优先
```

---

### 命令规范

```md
pnpm dev
pnpm build
pnpm lint
```

---

## 为什么非常重要

因为 Claude：

- 没有真正长期记忆
- 每次会话都可能变化

CLAUDE.md 可以稳定 AI 行为。

---

## 使用 @ 更新文件

你可以：

```text
@CLAUDE.md
请补充当前项目的数据库规范
```

Claude 会基于文件内容进行修改。

---

## 最佳实践

大型项目建议：

- 持续维护 CLAUDE.md
- 把架构约束写进去
- 把禁止事项写进去
- 把目录职责写进去

这样 AI 更稳定。

---

# 8. /context 查看上下文使用情况

## 作用

```bash
/context
```

查看：

- 当前上下文占用
- token 使用量
- 剩余上下文

---

## 为什么重要

AI Agent 最大限制之一：

就是 Context Window。

上下文越大：

- 成本越高
- 推理越慢
- 幻觉越多

---

## 推荐做法

上下文过大时：

- 新开会话
- 总结当前任务
- 更新 CLAUDE.md
- 减少无关文件

---

# 9. Skills 使用

*Skills的官方文档:https://agentskills.io/home*

## 什么是 Skills

Skills 本质上是：

> 可复用 AI 工作流。

类似：

- Prompt 模板
- AI 工作技能
- 自动化流程

---

## 查看 Skills

```bash
/skills
```

---

## Skills 能做什么

例如：

### 自动代码审查

### 自动生成 API

### 自动生成测试

### 自动生成文档

### 自动性能分析

---

## 创建 Skill 的思路

本质：

你在沉淀：

> AI 的最佳实践。

例如：

```text
每次新增 API：
1. 创建 schema
2. 创建 validator
3. 创建 service
4. 创建 route
5. 创建 test
```

你可以封装成 Skill。

---

## 高级理解

Skills 很像：

- DevOps Pipeline
- Prompt Engineering Workflow
- AI 自动化 SOP

---

## 注意事项

不要把 Skill 写得太模糊。

错误：

```text
帮我优化项目
```

正确：

```text
检查 React 组件：
1. 是否存在重复渲染
2. 是否存在 useEffect 滥用
3. 是否缺少 memo
```

---

# 10. Agents 使用

## 什么是 Agent

Agent 是 Claude Code 最核心概念之一。

传统 AI：

> 回答问题。

Agent：

> 主动完成任务。

---

## Claude Agent 的能力

Agent 可以：

- 读取代码
- 修改代码
- 执行命令
- 分析错误
- 自主规划
- 多轮执行
- 自动修复

---

## /agents

```bash
/agents
```

进入 Agent 功能。

---

## Agent 工作流示例

例如：

```text
修复当前项目所有 ESLint 错误
```

Agent 会：

1. 扫描项目
2. 运行 lint
3. 定位错误
4. 修改文件
5. 再次 lint
6. 循环直到通过

---

## 多 Agent 思维

未来 AI 开发趋势：

- 一个 Agent 写代码
- 一个 Agent Review
- 一个 Agent 测试
- 一个 Agent 部署

Claude Code 已经在向这个方向发展。

---

## 注意事项

### 不要完全信任 Agent

因为：

AI 会：

- 幻觉
- 误删代码
- 误解业务逻辑
- 引入安全问题

必须：

- 人工 Review
- Git 管理
- 小步提交

---

# 11. Plugins 使用

## plugins 是什么

plugins 本质是 skills + agents + mcp + hooks

Claude Code 的插件生态本质是：

>扩展 Claude 的能力边界。

---

## MCP 是什么

*MCP的官方相关文档:https://modelcontextprotocol.io/docs/getting-started/intro*

MCP（Model Context Protocol）是 Anthropic 推出的协议。

它允许 AI：

- 连接外部工具
- 调用数据库
- 调用浏览器
- 调用 API
- 调用本地服务

---

## MCP 类似什么

类似：

- AI 的 USB 接口
- AI Tool Calling Protocol
- AI 插件协议

---

## Hooks 是什么

Hooks 本质是：

> 在某个阶段自动触发动作。

例如：

- 修改代码后自动 lint
- 提交前自动 test
- 保存后自动格式化

---

# 案例：使用plugins创建摄影师个人网站

## 下载plugins

使用/plugins下载关于 frontend-design 插件

## Prompt 示例

```text
使用 Next.js + Tailwind + shadcn/ui
创建一个高端摄影师个人网站。

要求：
1. 极简黑白风格
2. 首页 Hero
3. 作品展示 Gallery
4. About 页面
5. Contact 页面
6. 响应式布局
7. Framer Motion 动画
8. SEO 优化
```

---

## Claude 推荐工作流

### 第一步

Plan Mode。

先规划：

- 页面结构
- 组件结构
- 数据结构
- 动画方案

---

### 第二步

生成 UI。

---

### 第三步

执行：

```bash
!pnpm dev
```

查看运行效果。

---

### 第四步

Agent 自动修复。

例如：

```text
修复所有 hydration error
```

---

# 总结 Claude Code 的最佳实践

# 1. 永远使用 Git

这是最重要原则。

---

# 2. 大任务先 Plan

不要直接生成。

---

# 3. 小步提交

不要一次改几十个文件。

---

# 4. 长项目维护 CLAUDE.md

稳定 AI 行为。

---

# 5. 重要项目不要关闭权限确认

尤其公司项目。

---
