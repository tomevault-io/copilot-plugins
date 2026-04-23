## slimebot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SlimeBot 是一个个人 AI Agent Demo 项目，实现可扩展的 AI 会话应用雏形。后端为 Go，前端为 Vue 3，生产环境通过嵌入前端静态资源构建单一二进制文件。

## Build & Run Commands

```bash
# 安装依赖
make deps

# 开发模式（同时启动 Go 后端 + Vite 前端，前端代理 /api /ws 到 8080）
npm run dev

# 生产构建（生成嵌入前端的 slimebot 可执行文件）
make build

# 运行测试
make test

# 运行单个包的测试
go test ./internal/services/chat/...

# 带 race 检测的测试
go test -race ./...

# 测试覆盖率
go test -cover ./...

# Docker
make docker-build
make docker-run
```

## Architecture

**分层架构**：`cmd/` → `internal/app/` → `internal/server/` → `internal/services/` → `internal/repositories/` → `internal/domain/`

- **`cmd/server/`** - 服务端入口，初始化日志、加载环境变量、启动应用
- **`internal/app/`** - 应用引导层，手动依赖注入组装所有服务和基础设施
- **`internal/domain/`** - 领域模型与接口定义
- **`internal/services/`** - 业务逻辑层，包含 chat、session、memory、embedding、skill、openai、auth 等服务
- **`internal/repositories/`** - 数据访问层（SQLite + GORM）
- **`internal/server/`** - HTTP 层（Chi 路由、控制器、WebSocket、中间件）
- **`internal/platforms/telegram/`** - 消息平台集成
- **`internal/mcp/`** - Model Context Protocol 集成
- **`frontend/`** - Vue 3 前端，生产构建后嵌入 `web/dist`
- **`prompts/`** - AI 提示词模板

**关键设计决策**：
- 接口在消费方定义，不在实现方
- 手动依赖注入（`app.New()` 中组装）
- 内存检索采用混合策略：优先向量检索（ONNX Runtime + BGE-M3 + Chroma），失败时回退关键词检索
- WebSocket 用于实时流式聊天回复（start/chunk/done 协议）
- Agent 支持多轮 tool call 链路，高风险工具需前端审批

## Configuration

运行时配置通过 `~/.slimebot/.env` 管理，首次启动自动生成。关键必填项：`JWT_SECRET`。默认端口：后端 `8080`，Vite `5173`。

## Code Conventions

- 不可变模式：始终创建新对象，不修改已有对象
- 错误处理：使用 `fmt.Errorf("context: %w", err)` 包装错误上下文
- 文件组织：小文件优先（200-400 行典型，800 行上限），按功能/领域组织
- Go 接口：保持小接口（1-3 个方法），接受接口返回结构体
- 表驱动测试
- Commit 格式：`<type>: <description>`（feat, fix, refactor, docs, test, chore, perf, ci）

## Tech Stack

- **后端**：Go 1.26, Chi router, GORM, SQLite, Gorilla WebSocket, OpenAI Go SDK, ONNX Runtime, Chroma
- **前端**：Vue 3, Vite, TypeScript
- **日志**：zap
- **认证**：JWT

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/natsuz0ra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
