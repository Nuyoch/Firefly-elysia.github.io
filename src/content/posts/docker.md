---
title: Docker 核心知识总结
published: 2026-02-11T17:41:26.381Z
tags: [Markdown, docker, 博客, 演示]
pinned: false
image: "./images/a3.jpg"
category: 开发笔记
draft: true
---

# Docker 核心知识总结

## 一、Docker 简介

### 1.1 什么是 Docker

-   **容器化平台**：将应用及其依赖打包成标准化单元
-   **轻量级虚拟化**：与虚拟机相比，共享主机内核，资源消耗少
-   **一次构建，到处运行**：解决“在我机器上能运行”的问题


### 1.2 Docker vs 传统虚拟机

| 特性       | Docker 容器         | 虚拟机             |
|:-----------|:-------------------:|-------------------:|
| 启动速度   | 秒级                | 分钟级             |
| 性能       | 接近原生            | 有损耗             |
| 系统资源   | MB 级别             | GB 级别            |
| 隔离性     | 进程级              | 系统级             |
| 镜像大小   | 通常较小            | 较大               |

---

## 二、Docker 核心概念

### 2.1 三大核心组件

-   **镜像（Image）**：只读模板，包含运行应用所需的所有内容
-   **容器（Container）**：镜像的运行实例，可读可写
-   **仓库（Registry）**：集中存放镜像的地方（Docker Hub、私有仓库）

### 2.2 Docker 架构

```
Client (docker CLI) → REST API → Docker Daemon
                              ↓
           Containers（运行时环境）
                              ↓
           Images（只读模板）
                              ↓
           Registry（镜像仓库）
```

---

## 三、镜像管理

### 3.1 镜像相关命令

```bash
# 搜索镜像
docker search nginx

# 拉取镜像
docker pull nginx:alpine

# 列出本地镜像
docker images
docker image ls

# 删除镜像
docker rmi <image_id>
docker image rm <image_id>

# 导出镜像
docker save -o nginx.tar nginx:alpine

# 导入镜像
docker load -i nginx.tar

# 查看镜像历史
docker history <image_name>
```

### 3.2 Dockerfile 编写

```dockerfile
# 基础镜像
FROM node:14-alpine

# 维护者信息
LABEL maintainer="your-email@example.com"

# 设置工作目录
WORKDIR /app

# 复制文件
COPY package*.json ./
COPY src ./src

# 安装依赖
RUN npm install --production

# 环境变量
ENV NODE_ENV=production \
    PORT=3000

# 暴露端口
EXPOSE 3000

# 启动命令
CMD ["node", "src/index.js"]
```

### 3.3 构建镜像

```bash
# 构建镜像
docker build -t myapp:1.0 .

# 指定 Dockerfile 路径
docker build -f Dockerfile.prod -t myapp:prod .
```

---

## 四、容器管理

### 4.1 容器生命周期命令

```bash
# 创建并启动容器
docker run [options] image [command]
```

**常用选项：**

-   `-d`：后台运行
-   `-p`：端口映射（主机端口:容器端口）
-   `-v`：卷挂载（主机路径:容器路径）
-   `--name`：指定容器名称
-   `-e`：设置环境变量
-   `--network`：指定网络

**示例：**

```bash
docker run -d -p 8080:80 --name my-nginx nginx

# 启动已停止的容器
docker start <container>

# 停止容器
docker stop <container>

# 重启容器
docker restart <container>

# 暂停/恢复容器
docker pause <container>
docker unpause <container>

# 删除容器
docker rm <container>
docker rm -f <container>   # 强制删除运行中的容器
```

### 4.2 容器操作命令

```bash
# 查看容器
docker ps          # 运行中的容器
docker ps -a       # 所有容器
docker ps -q       # 仅显示容器ID

# 查看容器日志
docker logs <container>
docker logs -f <container>   # 实时日志
docker logs --tail 50 <container>

# 进入容器
docker exec -it <container> /bin/bash

# 查看容器详情
docker inspect <container>

# 查看容器内进程
docker top <container>

# 查看资源使用
docker stats <container>

# 复制文件
docker cp <container>:<path> <host_path>   # 容器→主机
docker cp <host_path> <container>:<path>   # 主机→容器
```

---

## 五、Docker 网络

### 5.1 网络模式

-   **bridge（默认）**：通过 docker0 网桥通信
-   **host**：使用主机网络栈
-   **none**：无网络配置
-   **container**：共享另一个容器的网络
-   **overlay**：Swarm 集群容器间通信
-   **macvlan**：为容器分配 MAC 地址

### 5.2 网络管理命令

```bash
# 查看网络
docker network ls

# 创建网络
docker network create my-network

# 查看网络详情
docker network inspect my-network

# 连接容器到网络
docker network connect my-network my-container

# 断开网络连接
docker network disconnect my-network my-container

# 删除网络
docker network rm my-network
```

---

## 六、数据管理

### 6.1 数据卷（Volumes）

```bash
# 创建数据卷
docker volume create my-volume

# 查看数据卷
docker volume ls

# 使用数据卷
docker run -v my-volume:/data nginx

# 匿名卷
docker run -v /data nginx

# 绑定挂载（主机目录）
docker run -v /host/path:/container/path nginx

# 查看卷详情
docker volume inspect my-volume

# 删除卷
docker volume rm my-volume
docker volume prune   # 删除所有未使用的卷
```

### 6.2 数据卷容器

```bash
# 创建数据卷容器
docker create -v /data --name data-container busybox

# 使用数据卷容器
docker run --volumes-from data-container nginx
```

---

## 七、Docker Compose

### 7.1 docker-compose.yml 示例

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      - FLASK_ENV=development
    depends_on:
      - redis
    networks:
      - backend

  redis:
    image: "redis:alpine"
    networks:
      - backend

networks:
  backend:
    driver: bridge

volumes:
  db-data:
```

### 7.2 Compose 常用命令

```bash
# 启动服务
docker-compose up
docker-compose up -d      # 后台运行

# 停止服务
docker-compose down
docker-compose down -v    # 同时删除卷

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs
docker-compose logs -f

# 执行命令
docker-compose exec service_name command

# 构建服务
docker-compose build

# 重启服务
docker-compose restart
```

---

## 八、Docker Registry

### 8.1 使用 Docker Hub

```bash
# 登录 Docker Hub
docker login

# 标记镜像
docker tag myapp:1.0 username/myapp:1.0

# 推送镜像
docker push username/myapp:1.0

# 拉取镜像
docker pull username/myapp:1.0
```

### 8.2 私有仓库

```bash
# 启动私有仓库
docker run -d -p 5000:5000 --name registry registry:2

# 标记并推送到私有仓库
docker tag myapp:1.0 localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0

# 从私有仓库拉取
docker pull localhost:5000/myapp:1.0
```

---

## 九、最佳实践

### 9.1 镜像优化

1.  **使用多阶段构建**减少镜像大小
2.  **使用 Alpine 基础镜像**
3.  **合并 RUN 指令**减少镜像层数
4.  **使用 .dockerignore 文件**排除无关文件
5.  **清理 apt 缓存**等临时文件

### 9.2 安全实践

1.  不要以 root 用户运行容器
2.  定期更新基础镜像
3.  扫描镜像中的漏洞
4.  限制容器资源
5.  使用可信的基础镜像

### 9.3 资源限制

```bash
# 内存限制
docker run -m 512m --memory-swap=1g nginx

# CPU限制
docker run --cpus="1.5" nginx
docker run --cpuset-cpus="0-3" nginx

# 重启策略
docker run --restart=always nginx
docker run --restart=on-failure:5 nginx
```

---

## 十、常用命令速查表

| 功能               | 命令                               |
|:-------------------|:-----------------------------------|
| 查看 Docker 信息   | `docker info` `docker version`     |
| 清理无用资源       | `docker system prune -a`           |
| 批量停止容器       | `docker stop $(docker ps -q)`      |
| 批量删除容器       | `docker rm $(docker ps -aq)`       |
| 批量删除镜像       | `docker rmi $(docker images -q)`   |
| 查看磁盘使用       | `docker system df`                |

---

## 总结

Docker 的核心优势在于其**标准化、轻量级、可移植性**。掌握以下关键点：

1.  **理解核心概念**：镜像、容器、仓库的关系
2.  **熟练使用 Dockerfile**：构建可复用的镜像
3.  **掌握容器生命周期管理**：创建、启动、停止、删除
4.  **理解网络和数据持久化**：容器通信和数据管理
5.  **学会使用 Docker Compose**：多容器应用编排
6.  **遵循最佳实践**：安全、性能、维护性