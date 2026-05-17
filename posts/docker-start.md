---
title: docker容器话技术学习
published: 2026-05-17
description: ''
image: ''
tags: []
category: 'Docker'
draft: false 
lang: ''
---


好的，我将按照你提供的技能模板，结合 Docker 官方文档和微软官方文档中的内容，为你系统地讲解 Docker 的相关知识。

---

# docker是什么？

Docker 是一个开源的平台，用于开发、交付和运行应用程序。它利用**操作系统级虚拟化**技术（即容器）来打包应用及其所有依赖项（代码、运行时、系统工具、系统库等），确保应用能够在任何支持 Docker 的环境中（包括本地开发机、物理服务器、虚拟机、云平台等）以相同的方式运行。

**核心价值**：
- **环境一致**：解决“在我机器上能运行，到你机器上就不行”的问题。
- **轻量高效**：容器共享宿主机的操作系统内核，无需像虚拟机那样模拟整个 OS，因此启动速度极快（毫秒级），资源占用更低。
- **可移植性**：容器镜像可以轻松地在不同操作系统（如 Ubuntu、CentOS、Windows Server）和云平台之间迁移。

> 引用自 Docker 官方文档：“Docker 是一种容器化平台，它可将您的应用程序及其依赖项打包到一个称为镜像的独立单元中，然后可以在任何运行 Docker 的 Linux 或 Windows 服务器上运行。”

---

# docker的基础概念

Docker 生态系统由以下几个核心概念构成：

### 1. 镜像（Image）
- **定义**：只读的模板，包含创建容器所需的所有指令和文件系统（例如一个完整的 Nginx + Ubuntu 环境）。
- **类比**：相当于面向对象编程中的“类”。
- **获取方式**：从 Docker Hub（公共仓库）拉取，或通过 Dockerfile 构建。

### 2. 容器（Container）
- **定义**：镜像运行时的实例。容器是一个隔离且安全的进程沙箱，可被启动、停止、删除。
- **类比**：相当于“类”的“对象”。
- **特点**：每个容器都有自己独立的文件系统、网络和进程空间。

### 3. 仓库（Repository）与 Docker Hub
- **定义**：集中存储和分发 Docker 镜像的地方。Docker Hub 是官方默认的公共仓库。
- **命名示例**：`nginx:1.25` 表示 nginx 仓库 + 1.25 标签（版本）。

### 4. Dockerfile
- **定义**：一个文本文件，包含一系列构建镜像所需的指令（如 `FROM`、`RUN`、`COPY`、`CMD`）。
- **作用**：实现基础设施即代码（IaC），可重复生成镜像。

### 5. 数据卷（Volume）
- **定义**：用于持久化容器生成的数据，独立于容器生命周期。
- **作用**：防止容器删除后数据丢失，并可在多个容器之间共享数据。

### 6. Docker 网络（Network）
- **定义**：Docker 内置的网络驱动（如 bridge、host、overlay），用于容器间、容器与外部世界之间的通信。

---

# docker的使用方法

以下内容基于官方文档推荐的基础操作流程，适用于 Ubuntu / Windows WSL2 / macOS。

### 1. 安装 Docker
- **Ubuntu**（参照阿里云镜像快速安装）：
  ```bash
  curl -fsSL https://get.docker.com -o get-docker.sh
  sudo sh get-docker.sh --mirror Aliyun
  ```
- **Windows（使用 WSL2 后端）**：
  - 从 [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/) 下载安装包。
  - 确保 BIOS 已开启虚拟化，并启用 WSL2（PowerShell 管理员运行 `wsl --install`）。
  - 安装后启动 Docker Desktop，设置中启用“Use WSL 2 based engine”。

### 2. Docker 常用命令（官方分类）
#### 镜像操作
| 命令 | 说明 |
|------|------|
| `docker pull <镜像名>:<标签>` | 从仓库拉取镜像 |
| `docker images` | 列出本地已有镜像 |
| `docker rmi <镜像ID>` | 删除镜像 |
| `docker build -t <名称> .` | 使用 Dockerfile 构建镜像 |

#### 容器操作
| 命令 | 说明 |
|------|------|
| `docker run -d --name <容器名> <镜像>` | 创建并后台运行容器 |
| `docker ps` | 列出运行中的容器 |
| `docker ps -a` | 列出所有容器（包括停止的） |
| `docker stop <容器名>` | 停止容器 |
| `docker start <容器名>` | 启动已停止的容器 |
| `docker rm <容器名>` | 删除容器 |
| `docker exec -it <容器名> bash` | 进入容器内部执行命令 |

#### 其他常用选项
- `-p 宿主机端口:容器端口`：端口映射
- `-v 宿主机目录:容器目录`：挂载数据卷
- `--network`：指定网络

---

# 如何使用docker实战安装一个nginx

以下步骤可在任何安装好 Docker 的 Linux / Windows WSL2 / macOS 终端中执行。

### 步骤 1：拉取官方 Nginx 镜像
```bash
docker pull nginx:latest
```
*说明：`latest` 标签指向当前最新稳定版。*

### 步骤 2：运行 Nginx 容器
```bash
docker run -d --name my-nginx -p 8080:80 nginx:latest
```
**参数解析**：
- `-d`：后台运行（detach 模式）
- `--name my-nginx`：为容器命名
- `-p 8080:80`：将宿主机的 8080 端口映射到容器的 80 端口（Nginx 默认监听端口）
- `nginx:latest`：使用的镜像

### 步骤 3：验证访问
打开你的浏览器，访问 `http://localhost:8080` 或虚拟机的 IP:8080，你应该看到 Nginx 的欢迎页面。

### 步骤 4：管理容器
- **查看日志**：`docker logs my-nginx`
- **停止容器**：`docker stop my-nginx`
- **重启容器**：`docker restart my-nginx`
- **删除容器**（需先停止）：`docker rm my-nginx`

### 可选进阶：挂载自定义网页目录
```bash
# 创建一个本地目录用于存放网页文件
mkdir -p ~/nginx-data
echo "<h1>Hello from Docker Nginx</h1>" > ~/nginx-data/index.html

# 运行容器并挂载该目录覆盖 Nginx 默认的 /usr/share/nginx/html
docker run -d --name my-nginx -p 8080:80 -v ~/nginx-data:/usr/share/nginx/html nginx:latest
```

### 官方文档参考
- [Docker 官方入门指南](https://docs.docker.com/get-started/)
- [Nginx 官方镜像文档](https://hub.docker.com/_/nginx)
- [Windows 上的 Docker 文档](https://docs.docker.com/desktop/windows/)

---
