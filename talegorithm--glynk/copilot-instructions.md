## glynk

> Agent时代内容平台。吃进长内容（书/论文/播客/文章），做结构化处理，开放给人和Agent来标注和检索。

# Glynk

Agent时代内容平台。吃进长内容（书/论文/播客/文章），做结构化处理，开放给人和Agent来标注和检索。

## Tech Stack

- **Backend**: FastAPI + Python 3.11+
- **Database**: PostgreSQL + pgvector (向量搜索)
- **Embedding**: Azure OpenAI text-embedding-3-large (3072维)
- **Background**: APScheduler (RSS定时拉取)
- **Frontend**: React 19 + TypeScript, Vite + Tailwind CSS v4, Zustand, React Router 7
- **Agent**: cyber-agent 框架 (/Users/sunlit/Code/Agent)

## Data Model (v2)

核心 3 张表：

- **entities** — 参与者（人 / AI），状态 active / dormant / claimed
- **units** — 信息单元（ingested 内容 或 authored 想法），body JSONB + vector 3072维
- **anchors** — 连接关系（标注、回复、like 等），source → target

Sidecar 表：auth, reading_progress, reading_sessions, event_log, rss_sources

详见 `docs/glynk-data-model.md`

## Project Structure

```
glynk/
├── main.py                 # FastAPI入口
├── config.py               # 配置
├── models.py               # Entity / Unit / Anchor dataclass
├── ingestion/              # 核心1：任意格式 → Unit(origin=ingested)
│   ├── pipeline.py         # 摄入流水线（含 author Entity 创建）
│   ├── registry.py         # Handler选择
│   ├── handler/            # 内容类型handler (epub, pdf, html, markdown, ...)
│   ├── format_utils/       # 格式工具（epub, pdf, html）
│   └── processing/         # HTML处理
├── content/                # 核心2：双视图阅读
│   ├── reader.py           # 统一read接口（文件级/分页级）
│   ├── ai_view.py          # AI视图过滤
│   └── locator.py          # Span定位
├── annotation/             # 核心3：Anchor + 检索
│   ├── service.py          # AnchorService: CRUD + embedding
│   ├── search.py           # 语义检索引擎
│   └── vector_store.py     # pgvector on units table
├── storage/postgres.py     # PostgreSQL存储
├── embedding/service.py    # Embedding生成
├── api/                    # REST API
│   ├── auth.py             # Token验证中间件
│   ├── user_router.py      # POST /auth/register, GET /auth/me
│   ├── content_router.py   # /units/* (读取/详情/进度/会话/搜索)
│   ├── annotation_router.py # /anchors/* (CRUD + 检索)
│   ├── ingest_router.py    # /ingest
│   └── ...
├── agent/                  # 官方Agent工具
│   └── tools.py            # list_units/read_unit/create_anchors/search_units/save_thought
└── worker/rss_fetcher.py   # RSS拉取

glynk-web/src/
├── api/                    # HTTP客户端层
├── types/                  # TypeScript类型
├── store/
│   ├── auth.ts             # 认证状态
│   └── reader.ts           # 阅读器状态
├── components/reader/      # 阅读器组件
└── pages/                  # 页面
```

## Running

```bash
# 后端
pip install -r requirements.txt
uvicorn glynk.main:app --reload --port 8000

# 前端
cd glynk-web && npm install && npm run dev
```

本地开发连远程数据、服务器部署、端口转发等详见 `docs/deploy.md`。

## Key Design

- 平台不跑LLM，只做结构化处理和embedding检索
- 每个 Unit 归属真正作者（dormant Entity），导入者记在 metadata.imported_by
- 标注 = Unit(authored) + Anchor(连接到目标内容)
- span_id格式：`{unit_id}-{file_idx}-p{n}-s{m}`
- Unit 身份 vs 内容指纹：`unit_id` 是随机 16 位 hex（创建时固定）；`metadata.content_hash` 是 sha256，用于 ingest 去重和 update 检测
- 更新 publication：`POST /api/publications?update_of={unit_id}` → unit_id 不变，内容替换，span anchor 自动迁移（tier 1 exact / tier 2 fuzzy ≥ 0.85 / tier 3 orphan）

### Anchor role 与 schema

Role 是 Anchor 关系的性质。允许取值与约束见 `models.ROLE_SCHEMAS`：

| role | source | target | body |
|---|---|---|---|
| highlight | unit | span | auto（= target span 副本）|
| hook | unit | span | required |
| note | unit | span \| unit | required |
| summary | unit | unit | required |
| reply | unit | span \| unit | optional（emoji / 图片 / 文字皆可）|
| like | entity | span \| unit | none |
| bookmark | entity | span \| unit | none |
| follow | entity | entity | none |

创建 Anchor 时 `validate_anchor(role, source_type, target_type, has_body)` 强制校验；不合法 → 400。role 只存在 anchor 上，Unit 不复制 —— 搜索用 LEFT JOIN 到 anchors 按 `a.role` 过滤。

### Embedding 规则

一处决策：`embedding.service.maybe_embed(text, config, metadata)` 返回 vector 或 None。条件：

- `metadata.skip_embedding` 为 true → 跳过
- 有效字符（字母 / 数字 / CJK）< 30 → 跳过
- 未配置 Azure OpenAI → 跳过
- 其他情况 → 调 Azure 生成 3072 维向量

Embedding 不看 role —— 任何带 body 的 authored Unit 自然参与检索。Ingested Unit 不设 `vector_text`，自然不 embed。

### Anchor metadata 格式

Anchor 的 `metadata` 是 JSONB，格式由创建者决定：

```jsonc
// 用户高亮（精确选区）
{ "type": "text_selection", "spans": [...], "startSpanId": "...", "endSpanId": "...", "startOffset": 5, "endOffset": 10, "color": "yellow" }

// Agent 标注（span级别）
{ "type": "text", "spans": [...], "color": "ghost" }
```

### Unit 的两种 publishing 形态

平台里所有内容都是 Unit，但有**两种 publishing 行为**，对应不同的创建入口：

- **publication** = 可阅读 / 可精细标注的作品（书、文章、转写稿、用户 md）
  - `shape=structured`，HTML 文件落盘，span 级 ID，TOC
  - 内容寻址（`unit_id = sha256(content)[:16]`），同内容幂等去重
  - 作者可以是第三方（dormant Entity），导入者记在 `metadata.imported_by`
- **thought** = 一段 authored 的想法 / 评论 / 随手记录
  - `shape=flat`，body 是 html 字符串，无 span ID
  - 随机 UUID，两条相同文字的 thought 是独立 Unit
  - 作者永远是发布者本人
  - 标注的 source Unit 也走 thought 路径

读取、查询、搜索、挂 anchor、权限都用同一套（`GET /api/units/*`、`POST /api/anchors/*`）—— Unit 这一层是统一的。

### API 路径

```
POST /api/auth/register                  → Entity + auth
GET  /api/auth/me

# Unit 查询（二合一，publication 和 thought 都在这里）
GET  /api/units                          → 列出（可按 origin / author_id 过滤）
GET  /api/units/{id}                     → 详情
GET  /api/units/{id}/read                → 阅读（按 span 分页）
PUT  /api/units/{id}/outline             → AI 大纲
POST /api/units/search                   → 语义检索

# 创建 publication（发布可阅读内容）
POST /api/publications                   → 从 URL 导入（?update_of={id} 走原地更新）
POST /api/publications/upload            → 从文件（epub/pdf/html/md/zip）导入（同上支持 update_of）
POST /api/publications/media/init        → 音频/视频：拿 OSS presigned URL + 分配 unit_id
POST /api/publications/media/finalize    → 音频/视频：后端转写 + 建 HTML

# 创建 thought（放下想法）
POST /api/thoughts                       → 纯文本想法

# Anchor
POST /api/anchors                        → 创建标注
POST /api/anchors/batch                  → 批量
GET  /api/anchors                        → 查询
GET  /api/anchors/thread                 → 某 span 下的讨论
```


### Markdown 发布

用户自制内容通过 Markdown 上传成 publication。支持 frontmatter（title/author）和本地图片引用。上传脚本 `skills/glk-add/upload_md.py`。

### 图片路径

HTML 中图片 src 为 `/media/{unit_id}/{filename}`，后端提供静态文件。

### 数据库

核心 3 表 + sidecar 表。pgvector 3072维。

---
> Source: [Talegorithm/Glynk](https://github.com/Talegorithm/Glynk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
