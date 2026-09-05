## miniteamchat

> 该文件为 Claude Code (claude.ai/code) 提供在该仓库中工作的指导。

# CLAUDE.md

该文件为 Claude Code (claude.ai/code) 提供在该仓库中工作的指导。

## 项目概述
Mini Team Chat 是一个实时小组协作聊天室应用，具有以下核心功能：
- 基于 JWT 认证的用户注册/登录
- 房间创建/加入功能
- 通过 WebSocket 实现的实时消息广播
- 消息历史记录持久化与查询
- 基于 gRPC 通信的微服务架构

## 高级架构
仓库采用微服务架构，包含三个主要组件：

1. **用户服务** (`/service/user`)
   - 管理用户认证（注册、登录、JWT 签发/验证）
   - 将用户数据存储在 PostgreSQL 中
   - 暴露 gRPC 端点：GetUser, VerifyToken

2. **房间服务** (`/service/room`)
   - 管理房间和消息持久化
   - 实现 WebSocket 服务器用于实时通信
   - 通过 gRPC 与用户服务通信进行身份验证
   - 使用工作池处理并发消息
   - 将房间和消息数据存储在 PostgreSQL 中

3. **前端 Web** (`/web`)
   - Next.js + React 应用
   - 页面：/login, /rooms, /rooms/[id]
   - 使用 WebSocket 连接到房间服务进行实时聊天

基础设施：
- `/infra` 包含运行 PostgreSQL 的 Docker Compose 配置

## 关键技术
- **后端**: Go, Kratos 框架, GORM, gRPC, Protobuf, JWT, Bcrypt, WebSocket
- **前端**: Next.js, React, JavaScript WebSocket API
- **数据库**: PostgreSQL
- **基础设施**: Docker Compose

## 开发命令
### 数据库
```bash
# 使用 Docker Compose 启动 PostgreSQL
cd infra && docker-compose up
```

### 用户服务
```bash
cd service/user
# 初始化 Go 模块（如果需要）
go mod tidy
# 运行用户服务
go run main.go
```

### 房间服务
```bash
cd service/room
# 初始化 Go 模块（如果需要）
go mod tidy
# 运行房间服务
go run main.go
```

### 前端 Web
```bash
cd web
# 安装依赖
npm install
# 运行开发服务器
npm run dev
```

### Protobuf 代码生成
```bash
# 从 proto 文件生成 Go gRPC 代码
protoc --go_out=. --go-grpc_out=. path/to/proto/file.proto
```

## 关键文件和目录
- `/PRD.md`: 完整的项目需求和按天任务列表
- `/technologies.md`: 完整的技术栈及官方文档链接
- `/service/user/user.proto`: 用户服务 gRPC 接口定义
- `/service/room/room.proto`: 房间服务 gRPC 接口定义
- `/infra/docker-compose.yml`: PostgreSQL Docker 配置

## 开发规则
每次做出较大的代码改动后，自动在当前分支上提交一次。

---
> Source: [MiniTeamChatProject/MiniTeamChat](https://github.com/MiniTeamChatProject/MiniTeamChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
