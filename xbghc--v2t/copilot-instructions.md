## v2t

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Gist 是一个 AI 驱动的视频预审 Web 应用：刷到一个视频但拿不准要不要看完，先用 Gist 抓个大意——下载、转录、生成要点摘要，让你在投入时间前就看清这视频值不值得。支持 B 站、抖音、小红书等多平台。（仓库目录与包名仍沿用历史标识 `v2t`。）

## 项目结构 (pnpm Monorepo + Python backend)

```
v2t/
├── backend/                    # Python 后端（不在 pnpm workspace 里）
│   ├── app/
│   │   ├── main.py            # 应用入口 (FastAPI)
│   │   ├── worker.py          # arq worker 配置
│   │   ├── config.py          # 配置管理
│   │   ├── models/            # 数据模型
│   │   ├── storage/           # 存储层 (Redis + 本地文件)
│   │   ├── tasks/             # 后台任务
│   │   ├── routers/           # API 路由
│   │   ├── utils/             # 工具函数
│   │   └── services/          # 业务服务层
│   ├── tests/
│   └── pyproject.toml
├── apps/
│   └── web/                    # Vue 前端（@v2t/web）
│       ├── src/
│       │   ├── router/        # Vue Router 路由
│       │   ├── stores/        # Pinia 状态管理
│       │   ├── machines/      # 状态机
│       │   └── pages/         # 页面组件
│       ├── package.json
│       └── vite.config.ts
├── packages/
│   └── shared/                 # 共享 TS 类型（@v2t/shared）
│       └── src/api-types.ts   # 后端通信 schema
├── pnpm-workspace.yaml         # workspace 配置
├── package.json                # workspace 根
├── Dockerfile                  # 多阶段构建
├── docker-compose.yml          # 服务编排
├── Makefile                    # 构建脚本
└── deploy/
    ├── v2t.service            # systemd 服务 (web)
    ├── v2t-worker.service     # systemd 服务 (worker)
    ├── nginx-local.conf       # 本地 nginx 配置
    └── nginx-docker.conf      # Docker nginx 配置
```

## 常用命令

### Makefile

```bash
make install              # 安装所有依赖（pnpm install + uv sync）
make backend              # 启动后端 (端口 8103)
make worker               # 启动 arq worker
make frontend             # 启动前端开发服务器
make build                # 构建前端到 apps/web/dist/
make test                 # 运行测试
make lint                 # 运行 lint 检查
make lint-fix             # 自动修复 lint 问题
```

### Docker

```bash
make docker-build         # 构建镜像
make docker-up            # 启动所有服务
make docker-down          # 停止所有服务
make docker-logs          # 查看日志
```

### 后端 (backend/)

```bash
cd backend
uv sync                          # 安装依赖
uv run v2t-web                   # 启动 Web 服务 (端口 8103)
uv run arq app.worker.WorkerSettings  # 启动 arq worker
uv run pytest                    # 运行测试
```

### 前端 (apps/web/)

```bash
# 所有 pnpm 命令都在仓库根目录执行（workspace 模式）
pnpm install                              # 安装所有 workspace 依赖
pnpm --filter @v2t/web dev                # 开发服务器 (代理 /api 到 8103)
pnpm --filter @v2t/web build              # 构建到 apps/web/dist/
pnpm --filter @v2t/web type-check         # 类型检查
```

### 共享类型 (packages/shared/)

- 后端响应 schema（`WorkspaceResponse`、`WorkspaceStatus` 等）统一定义在 `packages/shared/src/api-types.ts`
- 通过 `import { ... } from '@v2t/shared'` 使用
- 新增后端接口时改这里，web 会自动同步

## 架构概览

### 部署架构 (Docker)

```
docker compose 编排 4 个服务：

nginx (8100:80) ──→ web:8101 (FastAPI)  ──→ Redis
                    worker (arq)        ──→ Redis

共享 volume: v2t-data (/data/v2t), v2t-tmp (/tmp/v2t)
```

### 后端分层

```
backend/app/
├── main.py           # FastAPI 入口，注册路由，启动检查
├── worker.py         # arq worker：process_workspace 任务 + 文件清理 cron
├── config.py         # 配置管理 (环境变量 > JSON文件 > 默认值)
├── models/
│   ├── enums.py      # WorkspaceStatus 枚举
│   ├── entities.py   # Workspace, WorkspaceResource 数据类
│   └── schemas.py    # Pydantic 请求/响应模型
├── storage/
│   ├── __init__.py   # 全局实例管理 (get_redis, get_metadata_store, get_workspace 等)
│   ├── redis_store.py    # Redis Hash + List 元数据存储 (24h TTL)
│   ├── local_file.py     # 本地文件存储
│   └── metadata_store.py # MetadataStore Protocol
├── tasks/
│   └── workspace_task.py # 视频处理流水线 (下载→提取→转录)，通过 Redis Pub/Sub 推送状态
├── routers/
│   ├── workspace.py  # POST /api/workspaces, GET /{id}, GET /{id}/status-stream (SSE)
│   ├── stream.py     # 流式内容生成端点
│   └── prompts.py    # GET /api/prompts
├── utils/
│   ├── response.py   # 共享 WorkspaceResponse 构建器
│   └── sse.py        # SSE 辅助函数
└── services/
    ├── xiazaitool.py       # 视频链接解析 (外部API)
    ├── video_downloader.py # 视频下载 (aria2c)
    ├── transcribe.py       # 音频转录 (Whisper 兼容 API)
    ├── llm.py              # AI 生成 (OpenAI 兼容 API)
    └── podcast_tts.py      # 播客音频合成 (阿里云百炼 TTS)
```

**全异步架构**：所有 I/O 使用 `async/await`，外部命令通过 `asyncio.create_subprocess_exec`。

**工作区状态流转**（后端）：
```
pending → downloading → transcribing → ready / failed
```

**任务执行**：arq worker 独立进程，通过 Redis Pub/Sub 向 web 进程推送状态，web 通过 SSE 推送到前端。

### 前端架构 (Vue 3 + Vite + Vue Router)

```
apps/web/src/
├── router/index.ts         # 路由: / (首页) 和 /w/:id (工作区)
├── stores/task.ts          # Pinia 状态管理
├── machines/
│   ├── workspaceMachine.ts # 工作区状态机 (1:1 镜像后端状态)
│   └── contentMachine.ts   # 内容生成状态机 (idle→streaming→done)
├── api/                    # API 调用
├── pages/                  # 页面组件
├── components/             # UI 组件
└── composables/            # Vue Composables
```

**状态同步**：SSE 监听后端状态变化，到达 `ready` 后前端编排内容生成。

### 关键依赖

- **系统**：ffmpeg (音频提取)、aria2c (多线程下载)、Redis
- **外部 API**：Whisper 兼容 API (转录)、OpenAI 兼容 API (内容生成)、阿里云百炼 (TTS)、Xiazaitool (链接解析)

## 环境变量

```bash
# OpenAI 兼容 API (AI 内容生成)
OPENAI_API_KEY       # API 密钥
OPENAI_BASE_URL      # API 端点 (默认: https://api.deepseek.com)
OPENAI_MODEL         # 模型名称 (默认: deepseek-reasoner)

# Whisper 兼容 API (语音转录)
WHISPER_API_KEY      # API 密钥
WHISPER_BASE_URL     # API 端点 (必填，无默认值)
WHISPER_MODEL        # 模型名称 (Groq 推荐 whisper-large-v3-turbo)

# Redis
REDIS_URL            # Redis 连接 (默认: redis://localhost:6379)

# 其他服务
DASHSCOPE_API_KEY    # 阿里云百炼 API (TTS 语音合成，可选)
XIAZAITOOL_TOKEN     # 视频链接解析 API
```

配置文件路径：`~/.config/v2t/config.json`

## API 端点

```
POST /api/workspaces                      # 创建工作区 {url}
GET  /api/workspaces/{id}                 # 获取工作区信息
GET  /api/workspaces/{id}/status-stream   # SSE 状态推送
GET  /api/workspaces/{id}/resources/{rid} # 下载资源文件
GET  /api/stream/{id}/{type}              # 流式内容生成
GET  /api/prompts                         # 获取提示词列表
```

## 错误处理

每个服务模块定义自己的异常类：`DownloadError`、`TranscribeError`、`LLMError`、`XiazaitoolError`。

---
> Source: [xbghc/v2t](https://github.com/xbghc/v2t) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
