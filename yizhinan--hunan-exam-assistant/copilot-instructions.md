## hunan-exam-assistant

> 此文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 项目概述

湖南省公务员考试助手 — AI 驱动的智能备考平台。三大功能模块：AI 申论批改、每日范文推荐、职位匹配与上岸难度分析。通过 ChromaDB 向量知识库（历年真题、湖南政策、时政新闻、高分范文）增强 RAG 检索。

## 常用命令

### 后端（Python/FastAPI）

```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000          # 启动开发服务器
```

尚未配置测试框架和代码检查工具。

### 前端（React/TypeScript）

```bash
cd frontend
npm install
npm run dev        # Vite 开发服务器，端口 5173，/api 请求代理到 localhost:8009
npm run build      # tsc -b && vite build（类型检查后打包）
npm run preview    # 预览生产构建
```

尚未配置测试和 lint 脚本。

### Docker Compose（全栈部署）

```bash
docker-compose up -d    # 启动 postgres、redis、chromadb、backend、frontend、celery_worker、celery_beat
```

## 架构总览

### 后端（`backend/`）

**入口文件：** `app/main.py` — FastAPI 应用，使用 `lifespan` 上下文管理器（启动时通过 `Base.metadata.create_all` 自动建表），注册 CORS 中间件和五个功能路由，挂载在 `/api/*`。

**数据库：** `app/core/database.py` 实现双引擎切换。根据 `DATABASE_URL` 判断：含 `sqlite` 则用同步引擎（避免 Windows 上 `greenlet` 报错），否则走 PostgreSQL 异步引擎（`create_async_engine` + `async_sessionmaker`）。所有 ORM 模型继承 `Base`，采用 SQLAlchemy 2.0 `mapped_column` 风格，主键为 UUID 字符串（`String(36)`），由 `gen_uuid()` 生成。查询统一使用 2.0 风格：`db.execute(select(Model).where(...)).scalars().all()`。

**认证：** JWT 方案，基于 `python-jose`。`decode_token`（在 `app/core/security.py` 中）是一个 FastAPI 依赖项，使用 `HTTPBearer` 提取请求头中的令牌。通过 `user_id: str = Depends(decode_token)` 注入路由。申论路由在模块级别统一要求认证，分析路由按接口单独控制。密码哈希采用 bcrypt，PBKDF2 作为降级方案。

**申论批改流水线（`app/core/grading_engine.py`）：** RAG 检索 → 提示词组装 → DeepSeek LLM 调用 → JSON 解析。五个评分维度：立意观点 (25%)、结构逻辑 (20%)、内容论据 (25%)、语言表达 (20%)、格式规范 (10%)。通过 `get_grading_engine()` 获取单例实例。

**LLM 客户端（`app/core/llm_client.py`）：** 基于 OpenAI SDK，指向 DeepSeek API 地址。`chat_json()` 方法发送系统提示词和用户提示词，返回解析后的 JSON。

**RAG（`app/core/rag.py`）：** 四个 ChromaDB 集合（`exam_questions`、`hunan_policy`、`hunan_news`、`model_essays`）。使用 `BAAI/bge-large-zh-v1.5` 生成 1024 维中文嵌入向量。不同文档类型采用不同的分块策略。

**Celery（`app/tasks/celery_app.py`）：** Redis 作为消息代理，JSON 序列化，时区 `Asia/Shanghai`。定时任务：每日新闻爬取、每周试题爬取。任务模块：`app/tasks/grading.py` 和 `app/tasks/crawl.py`。

**难度评分（`app/services/difficulty.py`）：** 加权公式 — 进面分 (40%) + 竞争比 (35%) + 招录规模 (15%) + 趋势修正 (10%)。输出 0-100 分数及三档分类：保底 / 稳妥 / 冲刺。通过 `app/services/difficulty_cache.py` 预计算缓存。

### 前端（`frontend/`）

**路由：** React Router v6，定义在 `src/App.tsx`。除 `/login` 和 `/register` 外，所有页面渲染在 `AppLayout` 布局组件内。未匹配路由重定向到首页 `/`。

**API 客户端：** `src/services/api.ts` — `ApiClient` 类自动从 `localStorage` 读取 JWT 并附加 `Authorization: Bearer` 请求头。401 响应自动清除令牌并跳转登录页。五个 API 模块：`authApi`、`essayApi`、`knowledgeApi`、`dailyApi`、`analysisApi`。所有请求/响应的 TypeScript 类型定义均在此文件中。

**状态管理：** 认证上下文通过 `src/hooks/useAuth.tsx` 提供（`AuthProvider` 包裹整个应用）。无其他全局状态库。

**样式：** Tailwind CSS 3.4，使用 `@/*` 路径别名导入。工具函数：`clsx` + `tailwind-merge`。

### 爬虫（`crawler/`）

Scrapy 项目。爬虫包括：`offcn_exam`、`huatu_exam`（真题）、`hunan_gov`、`rednet`（政策/新闻）、`people_essay`、`qstheory_essay`（范文）。Pipeline 负责去重和写入 PostgreSQL。

### 知识库素材（`knowledge_materials/`）

RAG 知识库的静态源文件，按目录分类：`exam/`（真题）、`policy/`（政策）、`news/`（时政）、`model/`（范文）。

## 配置

所有后端配置通过环境变量管理，定义在 `app/core/config.py`（pydantic-settings 的 `BaseSettings`）。复制 `backend/.env.example` 为 `backend/.env`，只需配置 `DEEPSEEK_API_KEY` 和 `JWT_SECRET` 即可启动。`.env` 文件通过 `@lru_cache` 缓存，只读取一次。

## 关键约定

- **语言：** 用户界面和对用户展示的文本使用中文。代码标识符和注释使用英文。
- **数据库查询：** 统一使用 SQLAlchemy 2.0 风格 — `select(Model).where(...)` 配合 `db.execute().scalars().all()`。
- **序列化：** 使用 Pydantic v2 的 `.model_dump()`（非 `.dict()`）。
- **可选类型：** 使用 `str | None`（Python 3.10+ 联合语法），而非 `Optional[str]`。
- **UUID：** `String(36)` 列，由 `gen_uuid()` 生成。
- **错误消息：** `HTTPException` 中的 `detail` 使用中文。
- **Vite 代理：** 前端开发服务器将 `/api` 请求代理到 `localhost:8009`，如后端端口不同需调整。

---
> Source: [Yizhinan/hunan-exam-assistant](https://github.com/Yizhinan/hunan-exam-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
