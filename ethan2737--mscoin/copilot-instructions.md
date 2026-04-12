## mscoin

> 本文档为 Claude Code (claude.ai/code) 在此代码仓库中工作提供指导。

# CLAUDE.md

本文档为 Claude Code (claude.ai/code) 在此代码仓库中工作提供指导。

## 项目概述

MSCOIN 是一个加密货币交易平台，采用 Go 微服务后端 + Vue.js 前端的架构。

**注意**: 本项目原为前后端分离的两个独立仓库，后合并为统一的 `mscoin` 仓库以便管理。
- 后端原始仓库：`mscoin-backend/`
- 前端原始仓库：`mscoin-frontend/`

## 架构

### 后端 (mscoin-backend/)

基于 go-zero 框架的 Go 微服务，使用 gRPC 进行服务间通信。服务模块：

- **ucenter** - 用户中心（注册、登录、会员、资产、提现）
- **ucenter-api** - 用户服务 HTTP API 网关
- **market** - 行情数据和汇率
- **market-api** - 行情服务 HTTP API（含 WebSocket 支持）
- **exchange** - 订单撮合引擎
- **exchange-api** - 交易所服务 HTTP API
- **jobcenter** - 后台任务（K 线、比特币交易、汇率）
- **mscoin-common** - 公共工具（加密、JWT、工具类、模型）
- **grpc-common** - 生成的 gRPC 客户端存根

每个服务遵循 go-zero 标准布局：
- `internal/config/` - 配置结构体
- `internal/handler/` - HTTP 处理器（API 服务）
- `internal/logic/` - 业务逻辑
- `internal/dao/` - 数据访问层
- `internal/repo/` - 仓库层
- `internal/model/` - 数据模型
- `internal/server/` - gRPC 服务器
- `internal/svc/` - 服务上下文
- `etc/conf.yaml` - 配置文件

### 前端 (mscoin-frontend/)

Vue 2.5 + Vue Router + Vuex + iView UI 框架，Webpack 3 打包。

核心目录：
- `src/pages/` - 页面组件
- `src/components/` - 可复用组件
- `src/config/` - 路由、状态管理、API 配置
- `src/assets/` - 静态资源、JS 工具、语言文件
- `build/` - Webpack 配置

## 命令

### 后端

```bash
# 进入后端目录
cd mscoin-backend

# 同步工作区
go work sync

# 运行单个服务
go run ucenter/main.go -f ucenter/etc/conf.yaml
go run market/main.go -f market/etc/conf.yaml
go run exchange/main.go -f exchange/etc/conf.yaml
go run jobcenter/main.go -f jobcenter/etc/conf.yaml
# API 服务
go run ucenter-api/main.go -f ucenter-api/etc/conf.yaml
go run market-api/main.go -f market-api/etc/conf.yaml
go run exchange-api/main.go -f exchange-api/etc/conf.yaml

# 构建单个服务
go build -o <输出文件> <服务>/main.go
```

### 前端

```bash
cd mscoin-frontend

# 安装依赖
npm install

# 开发服务器（端口 8080）
npm run dev

# 生产构建
npm run build

# 运行测试
npm run test        # 全部测试
npm run unit        # 单元测试 (Jest)
npm run e2e         # 端到端测试 (Nightwatch)

# 代码检查
npm run lint
```

### 基础设施

```bash
# 使用 Docker Compose 启动所有服务
cd mscoin-backend
docker-compose up -d

# 服务：MySQL (3309), Redis (6379), Etcd (2379), MongoDB (27018), Kafka (9092), Kafdrop (9000)
```

## 技术栈

**后端：**
- Go 1.19+ 使用 go-zero 框架
- gRPC 用于服务间通信
- GORM 作为 MySQL ORM
- MongoDB 存储时序数据（K 线、行情）
- Kafka 消息队列
- Redis 缓存
- Etcd 服务发现

**前端：**
- Vue 2.5 + Vue Router + Vuex
- iView 3.x UI 组件库
- Webpack 3 打包
- Socket.io/StompJS WebSocket 通信
- vue-i18n 国际化（中/英）
- vue-resource HTTP 请求

## 配置

- 后端服务使用 `etc/` 目录下的 YAML 配置文件
- 前端配置在 `config/dev.env.js` 和 `config/prod.env.js`
- Docker 环境变量在 `.env`

## 测试

- 前端使用 Jest 进行单元测试，Nightwatch 进行 E2E 测试
- 测试命令：`npm run unit` 单元测试，`npm run e2e` 端到端测试

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ethan2737)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ethan2737)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
