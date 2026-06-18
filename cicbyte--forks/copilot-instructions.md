## forks

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Forks 是一个 Git 仓库管理工具，提供 Web UI 管理本地克隆的 GitHub 仓库。后端 Go + Gin，前端 Vue 3 + Naive UI，前后端分离，前端通过 `//go:embed web/dist` 嵌入到 Go 二进制中。

## 构建与运行

**后端：**
```bash
go build -o forks        # 构建
./forks serve             # 启动服务 (默认 :8080)
./forks serve -p 9090     # 指定端口
./forks serve --token xxx # 启用 Bearer Token 认证
```

**环境变量配置：**
- `FORKS_PORT` — 服务端口（优先级低于 `-p` 参数，默认 8080）
- `FORKS_ADDRESS` — 监听地址（默认 0.0.0.0）
- `FORKS_HOME` — 数据目录（默认 `~/.forks/`）
- `FORKS_REPO_PATH` — 仓库存储路径

首次运行会交互式要求输入仓库存储路径，数据保存在 `~/.cicbyte/forks/`。

**前端：**
```bash
cd web
npm install
npm run dev       # 开发服务器 (localhost:3000)
npm run build     # 构建到 web/dist/
npm run lint      # ESLint 检查
npm run format    # Prettier 格式化
```

前端构建后需要重新 `go build` 才能将新前端嵌入二进制。

**Node 要求：** `^20.19.0 || >=22.12.0`

## 架构

```
main.go              → 入口，embed web/dist 资源
├── cmd/
│   ├── root.go      → Cobra 根命令，DB/日志初始化，建表
│   └── serve.go     → 所有 HTTP 路由和业务逻辑（Gin）
├── common/           → 全局变量（Db, LogFile）
├── models/           → 数据结构（GitRepoInfo, ProxyConfig, GitPlusConfig）
├── utils/            → 配置路径管理、GitHub API、JSON 工具、分页
├── assets/           → 嵌入资源桥接
└── web/              → Vue 3 前端 SPA
    └── src/
        ├── api/         → Axios API 调用
        ├── views/       → 页面组件
        ├── components/  → 通用组件
        ├── stores/      → Pinia 状态管理
        └── router/      → Vue Router
```

### 关键设计决策

- **serve.go 是单体路由文件**：所有 API 路由、SSE 事件流、业务逻辑都在 `cmd/serve.go` 中（约 3700+ 行），没有分层 controller/service。
- **SQLite 数据库**：使用 `go-sqlite3`（需 CGO），表结构在 `cmd/root.go` 中通过 `CREATE TABLE IF NOT EXISTS` 创建，字段变更通过 `ALTER TABLE ADD COLUMN` 兼容旧数据。
- **SSE 实时推送**：长时间操作（clone、pull、reset、scan、batch-clone）通过 SSE 推送进度，使用临时 token 认证。
- **Git Smart HTTP**：通过 `git-upload-pack --stateless-rpc` 实现只读镜像克隆，路径 `/git/{source}/{author}/{repo}.git`。Docker 环境需 `git config --global --add safe.directory '*'`。
- **代理支持**：运行时代理配置存储在内存中（`currentProxyConfig`），支持 HTTP/SOCKS5，Git 命令和 GitHub API 调用都走代理。
- **GitHub API**：通过 `utils/github.go` 获取仓库元信息（stars、forks、topics 等），未使用 SDK，直接 HTTP 请求 GitHub REST API。
- **剪贴板兼容**：`web/src/utils/clipboard.js` 统一封装，优先 `navigator.clipboard`，fallback 到 `execCommand('copy')` 兼容 HTTP 环境。

## Docker 部署

```bash
# 构建
docker build -t forks .

# 运行（默认端口 8080）
docker run -d -p 8080:8080 -v ./data:/data forks

# 自定义端口
docker run -d -p 9090:9090 -e FORKS_PORT=9090 -v ./data:/data forks

# 使用 docker-compose
docker compose up -d
```

环境变量：`FORKS_PORT`、`FORKS_ADDRESS`、`FORKS_HOME`、`FORKS_REPO_PATH`、`TZ`

## CLI 工具

配套 CLI 已独立到 [cicbyte/forks-cli](https://github.com/cicbyte/forks-cli)，提供镜像加速克隆、批量备份等功能。

### API 结构

所有 API 在 `/api` 前缀下，完整文档见 `API.md`。统一响应格式 `{ code, message, data }`。

主要路由组：
- `/api/repos` — 仓库 CRUD + Git 操作（clone/pull/reset）+ 文件浏览
- `/api/authors` — 作者列表
- `/api/stats` — 统计信息
- `/api/activities` — 操作日志（GET 列表 / POST 添加 / DELETE 清空）
- `/api/proxy` — 代理配置
- `/api/info` — 系统信息
- `/git/{source}/{author}/{repo}.git` — Git Smart HTTP 协议（只读镜像克隆）

### 前端结构

- **UI 框架**：Naive UI，暗色主题
- **代码查看器**：CodeMirror（支持多语言语法高亮）
- **状态管理**：Pinia（stores/repos.js, stores/theme.js, stores/view.js）
- **HTTP 客户端**：Axios，封装在 src/api/ 中

---
> Source: [cicbyte/forks](https://github.com/cicbyte/forks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
