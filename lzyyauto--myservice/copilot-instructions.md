## myservice

> 这个文件为 Claude Code (claude.ai/code) 在处理此代码库时提供指导。

# CLAUDE.md

这个文件为 Claude Code (claude.ai/code) 在处理此代码库时提供指导。

## 🎯 项目概述

**Z收集系统 (Z Collection System)** - 基于 FastAPI 的 RESTful API 系统，用于收集和管理用户数据。

**主要功能：**
- **休息健康管理**：睡眠/起床时间跟踪，支持与 Notion 同步
- **GTD 任务管理**：任务创建，包含待办/进行中/已完成/已取消状态
- **视频处理**：抖音视频解析、下载、音频提取、语音转文字和 AI 摘要
- **通知**：Bark 推送通知提醒

## 🏗 架构设计

### 核心技术栈
- **后端**：FastAPI 0.104.1，支持异步
- **数据库**：PostgreSQL + SQLAlchemy 2.0 + Alembic 迁移
- **认证**：JWT (python-jose) + bcrypt
- **验证**：Pydantic 2.5.2

### 目录结构
```
app/
├── api/v1/endpoints/      # API 路由
│   ├── rest_records.py    # 睡眠/起床记录
│   ├── gtd.py            # GTD 任务
│   └── video_process.py  # 视频处理（3个端点）
├── core/                 # 核心模块
│   ├── config.py         # 设置（从 .env 加载）
│   ├── security.py       # JWT 认证
│   └── services/         # 外部集成
│       ├── bark_service.py     # Bark 通知
│       ├── notion_service.py   # Notion API 同步
│       └── video_processor_service.py  # 视频处理与解析日志
├── db/                   # 数据库
│   ├── session.py        # 数据库会话管理
│   └── init_db.py        # 数据库初始化
├── models/               # SQLAlchemy 模型
│   ├── user.py          # 用户模型
│   ├── rest_record.py   # 睡眠/起床记录
│   ├── gtd_task.py      # GTD 任务
│   └── video_process_task.py  # 视频任务（包含 task_type 字段）
├── schemas/              # Pydantic 验证模式
└── utils/                # 工具
    └── ai_client.py     # AI 服务客户端（SiliconFlow/OpenAI）

alembic/
└── versions/             # 数据库迁移
```

## 🚀 常用命令

### 开发
```bash
# 安装依赖
pip install -r requirements.txt

# 运行开发服务器
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 使用 Docker 运行
docker-compose up -d

# 查看 API 文档
open http://localhost:8000/docs
```

### 数据库
```bash
# 运行迁移
alembic upgrade head

# 创建新迁移
alembic revision --autogenerate -m "描述"

# 查看迁移历史
alembic history
```

### 配置
```bash
# 复制环境模板
cp .env.example .env

# 编辑配置
# - 数据库：POSTGRES_HOST, USER, PASSWORD, DB
# - Notion：NOTION_TOKEN, *_DATABASE_ID
# - Bark：BARK_DEFAULT_DEVICE_KEY
# - AI：AI_PROVIDER (siliconflow/openai), SILICONFLOW_API_KEY
# - 视频：THIRD_PARTY_DOUYIN_API_URL, FFMPEG_PATH
```

## 🔌 API 端点

### 认证
所有端点都需要在请求头中携带 JWT 令牌：
```
Authorization: Bearer <token>
```

### 核心模块
**1. 休息记录** (`/api/v1/rest-records`)
- `POST /` - 创建睡眠/起床记录
- `GET /` - 列出记录

**2. GTD 任务** (`/api/v1/gtd-tasks`)
- `POST /` - 创建任务
- `GET /` - 列出任务

**3. 视频处理** (`/api/v1/video-process`)
- `POST /` - 提交完整视频处理任务（下载 → 音频 → ASR → AI 摘要）
- `GET /{task_id}` - 查询任务状态/结果
- `POST /parse-url` - 仅解析 URL（返回下载链接，支持视频/图片/实况照片）

### 视频处理详情
- **完整处理**：异步任务包含 4 个步骤（视频下载、音频提取、语音转文字、AI 摘要）
- **仅解析 URL**：快速解析获取下载链接（不做处理）
- **任务类型**：通过 `task_type` 字段区分"处理"和"解析"任务
- **存储**：结果保存在数据库，文件存储在 `temp/video/` 目录
- **依赖**：需要 ffmpeg 和第三方抖音 API 服务

## ⚙️ 配置

### 环境变量 (.env)
```env
# 核心
APP_NAME=rest-data-collector
DEBUG=True/False
ENVIRONMENT=development/production

# 数据库
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=myservice
POSTGRES_HOST=localhost  # Docker 环境下用 'db'
POSTGRES_PORT=5432

# 外部服务
NOTION_TOKEN=secret_xxx
NOTION_SLEEP_DATABASE_ID=xxx
NOTION_WAKE_DATABASE_ID=xxx
NOTION_GTD_DATABASE_ID=xxx

BARK_BASE_URL=https://api.day.app
BARK_DEFAULT_DEVICE_KEY=xxx

# AI 服务
AI_PROVIDER=siliconflow  # 或 openai
SILICONFLOW_API_KEY=sk-xxx
AI_VOICE_MODEL=FunAudioLLM/SenseVoiceSmall
AI_SUMMARY_MODEL=Qwen/QwQ-32B

# 视频处理
THIRD_PARTY_DOUYIN_API_URL=http://localhost:8088/api/hybrid/video_data
FFMPEG_PATH=/opt/homebrew/bin/ffmpeg
VIDEO_PROCESSING_TEMP_DIR=temp/video/
```

### 第三方依赖
- **抖音 API**：在 `THIRD_PARTY_DOUYIN_API_URL` 的外部服务，用于视频解析
- **ffmpeg**：系统必须安装，用于音频提取
- **Notion API**：可选的数据同步
- **Bark**：推送通知
- **AI 服务**：SiliconFlow（推荐）或 OpenAI，用于语音转文字和摘要

## 🗄 数据库模式

### 关键模型
- **User**：JWT 认证用户
- **RestRecord**：睡眠/起床时间记录（Notion 同步）
- **GtdTask**：GTD 任务管理
- **VideoProcessTask**：视频处理任务，包含字段：
  - `task_type`："process"（完整处理）或 "parse"（仅解析 URL）
  - `media_type`："video"、"image" 或 "live_photo"
  - `aweme_id`、`desc`、`author`：解析元数据
  - `download_urls`：下载链接列表
  - `video_path`、`audio_path`：文件路径
  - `subtitle_text`、`ai_summary`：处理结果

### 迁移
位于 `alembic/versions/`。最近的迁移添加了 task_type 和解析字段，用于记录解析 URL 访问。

## 🔐 安全

- 所有端点需要 JWT 令牌认证
- 用户级数据访问（用户只能访问自己的记录/任务）
- 密码使用 bcrypt 哈希
- 基于环境的密钥配置
- 开发环境开启所有源的 CORS

## 📝 开发说明

- **项目最近清理**：文档、测试文件和临时目录已移除
- **视频处理服务**：实现完整处理和仅解析 URL 功能
- **后台任务**：使用 FastAPI BackgroundTasks 进行异步视频处理
- **日志**：在 `app/main.py` (app/main.py:12-21) 配置
- **解析 URL 日志**：在数据库中记录解析访问 (app/core/services/video_processor_service.py)

## 🐛 常见问题

1. **数据库连接**：确保 PostgreSQL 运行且 .env 配置正确
2. **找不到 ffmpeg**：安装 ffmpeg 并在 .env 中设置 FFMPEG_PATH
3. **第三方 API 宕机**：视频处理需要外部抖音 API 服务
4. **AI API 错误**：检查 SILICONFLOW_API_KEY 或 OPENAI_API_KEY 配置
5. **迁移冲突**：用 `alembic downgrade base && alembic upgrade head` 重置

## 📦 依赖

`requirements.txt` 中的关键包：
- fastapi, uvicorn - Web 框架
- sqlalchemy, psycopg2-binary - ORM 和 PostgreSQL 驱动
- alembic - 数据库迁移
- pydantic, pydantic-settings - 验证和配置
- python-jose, passlib - JWT 和密码哈希
- notion-client - Notion API 集成
- aiohttp, requests - HTTP 客户端
- tenacity - 重试机制
- pytest* - 测试（可选，测试已移除）


## 规则
1. 全程使用中文与我沟通
2. 生成的相关文档都存放与docs/文件夹下,非特定名称框架单词之外,文档全部以中文.
3. 针对新的需求,在docs/下创建对应的需求文件夹,相关文档存储在此.
4. 需求设计流程参考 docs/SETP.md 原则
5. 当需求完成之后,更新此文档(CLAUDE.md),保证其信息准确性

---
> Source: [lzyyauto/MyService](https://github.com/lzyyauto/MyService) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
