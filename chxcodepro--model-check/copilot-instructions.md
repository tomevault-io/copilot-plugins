## model-check

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Model Check — AI 模型可用性检测与管理平台。管理多个 API 渠道，自动检测模型可用性，提供统一的代理接口（兼容 OpenAI/Anthropic/Gemini 格式）。

## 常用命令

```bash
# 开发
npm run dev              # 启动开发服务器
npm run build            # Prisma generate + Next.js 构建
npm run lint             # ESLint 检查

# 测试
npm test                 # Vitest (watch 模式)
npm run test:run         # Vitest (单次运行)

# 数据库
npm run db:generate      # 生成 Prisma 客户端 (含 post-generate 脚本)
npm run db:push          # 推送 schema 到数据库
npm run db:seed          # 填充种子数据
npm run db:studio        # Prisma Studio GUI

# 本地开发需先启动依赖服务
docker compose up -d postgres redis
```

修改 `prisma/schema.prisma` 后需执行 `npm run db:generate` 重新生成客户端，生成产物在 `src/generated/prisma/`。

## 技术栈

- **框架**: Next.js 16 (App Router) + React 19 + TypeScript 5
- **数据库**: PostgreSQL (Prisma 7 ORM，使用 `@prisma/adapter-pg` 驱动适配器)
- **缓存/队列**: Redis (ioredis) + BullMQ 任务队列
- **UI**: Tailwind CSS 4 + shadcn/ui (new-york 风格)
- **认证**: JWT + bcryptjs
- **定时任务**: node-cron
- **部署**: Docker 多阶段构建 (node:22-alpine, standalone 模式)

## 架构要点

### 路径别名
`@/*` 映射到 `./src/*`（tsconfig.json 配置）。

### 后端核心模块 (`src/lib/`)

| 模块 | 说明 |
|------|------|
| `prisma.ts` | PrismaClient 全局单例，使用 PG adapter |
| `redis.ts` | Redis 全局单例 + PubSub 管理器（用于 SSE 广播） |
| `auth.ts` | JWT 认证，未配置 JWT_SECRET 时自动生成 |
| `init.ts` | 服务初始化入口，启动 worker + cron |
| `queue/` | BullMQ 检测任务队列：queue.ts(队列管理)、worker.ts(消费者)、service.ts(任务编排) |
| `detection/` | 模型检测引擎：detector.ts(执行)、strategies.ts(按端点类型分策略) |
| `proxy/index.ts` | API 代理层，多格式转发 + 轮询/随机负载均衡 + 多密钥 |
| `scheduler/cron.ts` | 定时任务，支持 cron 表达式和固定间隔，配置持久化到 DB |
| `webdav/sync.ts` | WebDAV 增量同步 |
| `utils/proxy-fetch.ts` | HTTP/SOCKS5 代理请求封装 |
| `middleware/auth.ts` | API 路由认证中间件 |

### 服务自启动机制
`src/instrumentation.ts` 利用 Next.js instrumentation hook，在服务器启动时自动调用 `initializeServices()` 启动 BullMQ worker 和 cron 调度器。

### 前端架构

单页应用模式，主页面 `src/app/page.tsx` 为客户端组件。状态管理直接用 React state（无 Redux/Zustand）。

- `components/dashboard/` — 核心业务组件（ChannelCard、ChannelManager、Summary 等）
- `components/ui/` — 可复用 UI 组件（基于 shadcn/ui）
- `components/providers/` — AuthProvider(JWT localStorage)、ThemeProvider
- `hooks/use-sse.ts` — SSE 实时进度推送，带指数退避重连

### API 路由结构

- `/api/auth/login` — 登录认证
- `/api/channel/` — 渠道 CRUD + 子路由 (key、sync、validate-keys、import/export)
- `/api/dashboard` — 仪表盘数据（分页、筛选）
- `/api/detect` — 触发/查询检测
- `/api/proxy-keys` — 代理密钥管理
- `/api/scheduler/` — 定时任务配置
- `/api/sse/progress` — SSE 实时进度
- `/api/worker` — 队列 worker 状态

### 代理端点（兼容主流 API 格式）

- `/v1/chat/completions` — OpenAI Chat
- `/v1/messages` — Anthropic Messages
- `/v1/models` — 模型列表
- `/v1/responses` — OpenAI Responses
- `/v1beta/models/[...path]` — Gemini

### 数据库模型 (Prisma)

核心模型: Channel(渠道) → ChannelKey(多密钥) → Model(模型) → CheckLog(检测日志)。
辅助模型: SchedulerConfig(调度配置)、ProxyKey(代理密钥)、ModelKeyword(模型关键词过滤)。
枚举: EndpointType(CHAT/CLAUDE/GEMINI/CODEX/IMAGE)、CheckStatus(SUCCESS/FAIL)。

### 检测策略

`detection/strategies.ts` 按 EndpointType 实现不同检测策略：Chat 用 OpenAI 格式、Claude 用 Messages 格式（含 thinking 重试逻辑）、Gemini 用 generateContent、Image 用图片生成接口。

## 环境变量

必选: `ADMIN_PASSWORD`、`JWT_SECRET`、`DATABASE_URL`、`REDIS_URL`。
Docker 环境额外需要: `DOCKER_DATABASE_URL`、`DOCKER_REDIS_URL`。
完整配置见 `.env.example`。

---
> Source: [chxcodepro/model-check](https://github.com/chxcodepro/model-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
