## automated-resume-submission-agent

> > 本文档是项目的「指挥中心」，所有协作者（包括 Claude）在动手前都应先阅读本文。

# 自动投递简历 Agent 项目 (interviewAccessWebsite)

> 本文档是项目的「指挥中心」，所有协作者（包括 Claude）在动手前都应先阅读本文。
> 任何架构、模块、字段的变更都必须同步更新本文档。

---

## 1. 项目概述 (Project Overview)

构建一个智能投递简历的 Agent 系统，用户上传简历并配置筛选条件后，系统自动从国内主流招聘 / 校招网站（Boss 直聘、实习僧、牛客、应届生求职网、智联招聘、前程无忧、拉勾、猎聘等）抓取符合条件的岗位，并代为投递简历，最终向用户回传投递结果（成功列表、失败列表与失败原因）。

### 核心价值
- 把求职者从「在 N 个 App 里反复搜索 + 点投递」中解放出来。
- 用 LLM + LangGraph 编排决策流程，让岗位匹配比关键词搜索更贴近用户真实意图。
- 给出可审计的投递清单和失败原因，便于复盘。

### 合规与边界 (重要)
- 各招聘平台普遍禁止自动化投递，本项目仅作为**技术研究 / 学习用途**使用。
- 默认不破解登录态、不绕过验证码、不规避平台风控；登录态由用户手动完成 (扫码 / 短信验证码)，Agent 仅在用户已登录的浏览器上下文内执行操作。
- 所有抓取行为应遵循目标站点的 robots.txt 与服务条款；生产部署需用户自担风险。
- 简历是敏感个人数据，必须做到：本地或加密存储、不外发到第三方、可一键删除。

---

## 2. 技术栈 (Tech Stack)

### 后端
- **语言**: Python 3.11+
- **Agent 编排**: LangGraph (核心工作流)
- **LLM 调用**: LangChain (OpenAI / Anthropic / 通义千问，可配置)
- **Web 框架**: FastAPI (对外 REST + WebSocket)
- **异步任务**: asyncio + (可选 Celery / RQ，根据规模决定)
- **浏览器自动化**: Playwright (推荐，支持反检测 + 多上下文)
- **HTML 解析**: BeautifulSoup4 / lxml / parsel
- **简历解析**: pdfplumber + python-docx + LLM 兜底解析
- **数据校验**: Pydantic v2
- **数据库**: SQLite (MVP) → PostgreSQL (生产)
- **ORM**: SQLAlchemy 2.x + Alembic
- **认证**: JWT (python-jose) + passlib bcrypt
- **日志**: loguru
- **测试**: pytest + pytest-asyncio

### 前端
- **框架**: Vue 3 (Composition API + `<script setup>`)
- **UI 库**: Element Plus
- **构建**: Vite
- **语言**: TypeScript
- **状态管理**: Pinia
- **路由**: Vue Router 4
- **HTTP**: Axios (含拦截器：JWT 注入、401 跳登录)
- **实时反馈**: WebSocket 或 SSE，用于推送投递进度

---

## 3. 系统架构 (Architecture)

```
┌────────────┐    HTTP/WS    ┌──────────────────────────────┐
│  Vue 前端   │ ─────────────▶│  FastAPI 网关层              │
│ ElementPlus│ ◀─────────────│  (认证 / 路由 / WS 推送)     │
└────────────┘                └──────────────┬───────────────┘
                                             │
                                             ▼
                              ┌──────────────────────────────┐
                              │   LangGraph Agent 编排层     │
                              │  ┌────────────────────────┐  │
                              │  │ Resume Parser Node     │  │
                              │  │ Intent Extract Node    │  │
                              │  │ Site Selector Node     │  │
                              │  │ Search & Crawl Node    │  │
                              │  │ Fuzzy Match Node (LLM) │  │
                              │  │ Filter Node            │  │
                              │  │ Apply Node             │  │
                              │  │ Aggregator Node        │  │
                              │  └────────────────────────┘  │
                              └──────────────┬───────────────┘
                                             │
                          ┌──────────────────┼──────────────────┐
                          ▼                  ▼                  ▼
                   ┌────────────┐    ┌────────────┐      ┌────────────┐
                   │ Site Adapter│   │ Site Adapter│      │ Site Adapter│
                   │  Boss 直聘  │    │   实习僧    │  ...│   牛客      │
                   └─────┬──────┘    └─────┬──────┘      └─────┬──────┘
                         └──────────────────┴──────────────────┘
                                            │
                                            ▼
                                   ┌─────────────────┐
                                   │  Playwright     │
                                   │  浏览器池        │
                                   └─────────────────┘

                              ┌──────────────────────────────┐
                              │     存储层 (SQLite/PG)        │
                              │  用户 / 简历 / 任务 / 投递记录│
                              └──────────────────────────────┘
```

---

## 4. LangGraph Agent 工作流 (核心)

State (TypedDict) 至少包含：
```python
class AgentState(TypedDict):
    user_id: str
    raw_resume: bytes | str         # 原始简历内容
    parsed_resume: dict             # 结构化简历
    user_filters: dict              # 用户筛选条件
    target_sites: list[str]         # 选定的目标站点
    candidate_jobs: list[Job]       # 抓取到的候选岗位
    matched_jobs: list[Job]         # 过滤后匹配的岗位
    apply_results: list[ApplyResult]# 投递结果
    errors: list[ErrorRecord]
    progress: ProgressEvent         # 用于 WS 推送
```

### 节点 (Nodes)
1. **resume_parser**: PDF/DOCX → 结构化 JSON (姓名、学历、技能、经历、期望岗位)。
2. **intent_extractor**: 结合用户输入的「岗位名称」+ 简历，用 LLM 提炼搜索关键词与岗位画像。
3. **site_selector**: 根据用户选择 / 默认策略决定要抓哪些站点。
4. **search_and_crawl** (并行 fan-out): 调用各站点 Adapter 的 `search()`，返回岗位列表。
5. **fuzzy_matcher**: 用 LLM + 规则给每个候选岗位打分 (RapidFuzz + 嵌入相似度兜底)，过滤明显不符。
6. **filter_node**: 按硬性条件 (学历、薪资、规模、融资、地点、到岗天数、岗位类型) 过滤。
7. **apply_node** (并行 + 速率限制): 调用各 Adapter 的 `apply(job, resume)`，记录成功/失败原因。
8. **aggregator**: 汇总结果，写 DB，推 WS。

### 控制流
- 节点之间用 `add_conditional_edges` 处理：搜不到岗位 → 终止 + 提示；匹配数为 0 → 反问用户是否放宽条件。
- 失败重试：节点级 try/except + 最多 N 次退避重试；不可恢复错误立即写 `errors` 并继续。

---

## 5. 模块拆解 (Module Breakdown)

### 后端模块
| 模块 | 路径 | 职责 |
|------|------|------|
| API 层 | `backend/app/api/` | FastAPI 路由 (auth / resume / task / result / ws) |
| 认证 | `backend/app/auth/` | JWT 签发、密码哈希、依赖注入 `get_current_user` |
| 简历解析 | `backend/app/services/resume_parser.py` | PDF / DOCX → 结构化字段；LLM 兜底 |
| 任务编排 | `backend/app/agent/graph.py` | LangGraph 图定义 + 状态 |
| Agent 节点 | `backend/app/agent/nodes/` | 每个节点一个文件，独立可测 |
| 站点适配器 | `backend/app/adapters/<site>/` | 每个站点一个目录，实现统一接口 `SiteAdapter` |
| 模糊匹配 | `backend/app/services/matcher.py` | RapidFuzz + Embedding + LLM 评分 |
| 浏览器池 | `backend/app/browser/pool.py` | Playwright 上下文复用、Cookie 持久化 |
| 数据模型 | `backend/app/models/` | SQLAlchemy ORM |
| Schemas | `backend/app/schemas/` | Pydantic 入参 / 出参 |
| 配置 | `backend/app/core/config.py` | Settings (BaseSettings)，读取 .env |
| 日志 | `backend/app/core/logging.py` | loguru 统一配置 |
| 任务持久化 | `backend/app/services/task_store.py` | 任务 / 进度 / 结果落库 |
| WebSocket | `backend/app/api/ws.py` | 推送 progress 事件 |

#### 站点适配器统一接口
```python
class SiteAdapter(Protocol):
    site_name: str
    async def login(self, ctx: BrowserContext) -> bool: ...
    async def search(self, query: SearchQuery) -> list[Job]: ...
    async def detail(self, job: Job) -> JobDetail: ...
    async def apply(self, job: Job, resume: Resume) -> ApplyResult: ...
```

#### 目标站点 (MVP 先做 2 个，逐步扩展)
- Boss 直聘 (`zhipin.com`)
- 实习僧 (`shixiseng.com`)
- 牛客网校招 (`nowcoder.com`)
- 应届生求职网 (`yingjiesheng.com`)
- 智联招聘 (`zhaopin.com`)
- 前程无忧 (`51job.com`)
- 拉勾 (`lagou.com`)
- 猎聘 (`liepin.com`)

### 前端模块
| 模块 | 路径 | 职责 |
|------|------|------|
| 登录 / 注册 | `frontend/src/views/auth/` | 表单 + JWT 保存 |
| 简历上传 | `frontend/src/views/resume/UploadView.vue` | 拖拽上传 + 解析结果预览 / 修正 |
| 筛选配置 | `frontend/src/views/task/FilterView.vue` | Element Plus Form，校验必填 |
| 任务进度 | `frontend/src/views/task/ProgressView.vue` | WS 订阅 progress，时间线展示 |
| 投递结果 | `frontend/src/views/result/ResultView.vue` | 成功 / 失败两个 Tab，可导出 CSV |
| 历史任务 | `frontend/src/views/history/HistoryView.vue` | 历史投递记录、再次发起 |
| 公共组件 | `frontend/src/components/` | JobCard、StatusTag、EmptyState 等 |
| API SDK | `frontend/src/api/` | axios 封装，类型与后端 schemas 对齐 |
| Store | `frontend/src/stores/` | user / task / result Pinia stores |

---

## 6. 用户输入字段 (筛选条件清单)

> 这是和用户的契约，所有字段都要在前端表单、Pydantic schema、DB 模型三处保持一致。

### 简历相关
- `resume_file`: 上传的 PDF / DOCX (必填)
- `resume_parsed`: 解析后的结构化字段 (姓名 / 学历 / 学校 / 专业 / 毕业年份 / 工作经历 / 项目 / 技能 / 期望薪资 / 期望城市)，允许用户在前端纠错

### 岗位意图
- `target_position` (必填): 岗位名称，用于模糊匹配 (例如 "前端开发 / 后端 Java / 算法")
- `position_keywords` (选填): 附加关键词数组
- `job_type` (必填): 单选 / 多选 — `全职 (full_time)` / `兼职 (part_time)` / `实习 (intern)` / `校招 (campus)` / `社招 (social)`

### 硬性筛选
- `education_min`: 学历要求下限 — 大专 / 本科 / 硕士 / 博士
- `salary_min`, `salary_max`: 期望月薪 (k)，单位千元
- `salary_period`: 月薪 / 日薪 (实习常用日薪)
- `company_size`: 企业规模多选 — 0-20 / 20-99 / 100-499 / 500-999 / 1000-9999 / 10000+
- `funding_stage`: 融资阶段多选 — 未融资 / 天使 / A / B / C / D 及以上 / 已上市 / 不需要融资
- `industries`: 行业多选 — 互联网 / 金融 / 制造 / 教育 / 医疗 / ...
- `cities`: 城市多选 (含「不限」)
- `work_mode`: 现场 / 远程 / 混合
- `days_per_week`: 一周到岗天数 (实习专用，1-7)
- `duration_months`: 实习时长下限 (实习专用)
- `company_blacklist`: 排除的公司名 (字符串数组)
- `company_whitelist`: 优先投递的公司

### 投递控制
- `target_sites`: 目标站点多选 (默认全选)
- `max_apply_per_site`: 每个站点最多投递数 (默认 20，防风控)
- `dry_run`: 仅匹配不投递 (用于调试)
- `apply_cover_letter`: 是否生成 LLM 个性化打招呼语 / 求职信

### 账号 (登录态)
- 每个站点的登录方式: `manual_scan` (默认，弹出浏览器扫码) / `cookie_import` (高级用户)

---

## 7. API 接口设计 (REST + WS)

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/auth/register` | 注册 |
| POST | `/api/auth/login` | 登录，返回 JWT |
| GET  | `/api/auth/me` | 当前用户信息 |
| POST | `/api/resume/upload` | 上传并解析简历，返回结构化字段 |
| PUT  | `/api/resume/{id}` | 用户修正解析结果 |
| GET  | `/api/resume/{id}` | 查询简历 |
| POST | `/api/tasks` | 创建投递任务 (body: filters + resume_id) |
| GET  | `/api/tasks/{id}` | 查询任务状态与汇总结果 |
| GET  | `/api/tasks/{id}/results` | 投递明细 (支持分页 + 成功/失败筛选) |
| POST | `/api/tasks/{id}/cancel` | 取消任务 |
| GET  | `/api/tasks` | 历史任务列表 |
| WS   | `/ws/tasks/{id}` | 实时进度推送：`{stage, current, total, last_job, last_status}` |

### 投递结果返回示例
```json
{
  "task_id": "xxx",
  "summary": {
    "total_matched": 87,
    "applied_success": 62,
    "applied_failed": 25,
    "by_site": {"zhipin": 30, "shixiseng": 20, "nowcoder": 12}
  },
  "items": [
    {
      "site": "zhipin",
      "company": "字节跳动",
      "company_size": "10000+",
      "funding": "已上市",
      "city": "北京",
      "position": "前端开发工程师",
      "salary": "20-35k·15薪",
      "status": "success",
      "applied_at": "2026-05-23T10:21:00+08:00",
      "url": "https://..."
    },
    {
      "site": "lagou",
      "company": "XX 科技",
      "status": "failed",
      "fail_reason": "HR 已下线，无法沟通",
      "url": "https://..."
    }
  ]
}
```

---

## 8. 数据模型 (DB Schema 草稿)

- `users` (id, email, password_hash, created_at)
- `resumes` (id, user_id, file_path, parsed_json, created_at)
- `site_credentials` (id, user_id, site, cookie_blob_encrypted, expires_at)
- `tasks` (id, user_id, resume_id, filters_json, status, created_at, finished_at)
- `task_progress` (id, task_id, stage, payload_json, created_at) — 可选，用于回放
- `apply_records` (id, task_id, site, company, position, status, fail_reason, raw_json, url, applied_at)

---

## 9. 项目目录结构

```
interviewProject/
├── CLAUDE.md                    ← 本文档
├── README.md
├── .env.example
├── docker-compose.yml           (后期)
├── backend/
│   ├── pyproject.toml
│   ├── alembic/
│   └── app/
│       ├── main.py
│       ├── core/
│       ├── api/
│       ├── auth/
│       ├── agent/
│       │   ├── graph.py
│       │   ├── state.py
│       │   └── nodes/
│       ├── adapters/
│       │   ├── base.py
│       │   ├── zhipin/
│       │   ├── shixiseng/
│       │   └── ...
│       ├── browser/
│       ├── services/
│       ├── models/
│       ├── schemas/
│       └── tests/
└── frontend/
    ├── package.json
    ├── vite.config.ts
    ├── tsconfig.json
    └── src/
        ├── main.ts
        ├── App.vue
        ├── router/
        ├── stores/
        ├── api/
        ├── components/
        ├── views/
        │   ├── auth/
        │   ├── resume/
        │   ├── task/
        │   ├── result/
        │   └── history/
        └── styles/
```

---

## 10. 开发任务拆解 (按里程碑)

### M0 - 项目脚手架 (1-2 天)
- [ ] 后端：FastAPI + Pydantic + SQLAlchemy + Alembic 初始化
- [ ] 前端：Vite + Vue3 + TS + Element Plus + Pinia 初始化
- [ ] 统一 .env、日志、错误处理中间件
- [ ] CI lint (ruff / eslint)

### M1 - 用户与简历 (2-3 天)
- [ ] 注册 / 登录 / JWT
- [ ] 简历上传 + PDF/DOCX 解析 + 字段纠错 UI
- [ ] 简历持久化与查询

### M2 - 筛选表单与任务创建 (2 天)
- [ ] 前端筛选表单 (按 §6 字段)
- [ ] 后端 task 创建接口 + 落库
- [ ] WebSocket 通道连通

### M3 - LangGraph Agent 骨架 (3-4 天)
- [ ] 定义 State + 各节点空实现
- [ ] Mock Adapter (返回假数据) 跑通端到端流程
- [ ] 模糊匹配节点 (RapidFuzz + LLM)

### M4 - 站点适配器 (按站点逐个迭代)
- [~] Boss 直聘 (优先) — search + detail 骨架，apply 待联调
- [~] 实习僧 — 骨架完成，薪资字体解密待处理 (TODO(font))
- [~] 牛客网 — 骨架完成，选择器待联调 (TODO(selector))
- [ ] 应届生求职网 / 智联 / 前程 / 拉勾 / 猎聘

> 当前默认 `ADAPTER_MODE=mock`；要切真站点，需先 `pip install -e ".[browser]"`
> → `python -m playwright install chromium` → `.env` 改 `ADAPTER_MODE=playwright`，
> 并由用户在 headed 浏览器手动扫码。

### M5 - 浏览器池 & 反风控 (贯穿) ✅
- [x] Playwright 上下文池 (`app/browser/pool.py` — per (user,site) BrowserContext)
- [x] Cookie 加密持久化 (`app/services/credential_store.py`，Fernet 对称加密)
- [x] 速率限制 / 随机延时 / UA 轮换 (`app/browser/{rate_limit,ua}.py`)
- [x] 失败重试 + 死信队列 (`app/browser/{retry,deadletter}.py`)

### M6 - 结果展示与导出 ✅
- [x] 成功 / 失败 / 跳过 Tab + 按站点 multi-select 过滤 (`ProgressView.vue`)
- [x] CSV 导出 (跟随当前过滤)
- [x] 历史任务页 (`HistoryView.vue`)

### M7 - 加固与上线 (持续)
- [ ] 单测 / E2E (Playwright codegen)
- [ ] Docker 化
- [ ] 简单可观测性 (loguru → 文件 / Sentry 可选)

---

## 11. 风险与挑战

| 风险 | 缓解策略 |
|------|----------|
| 平台反爬 / 风控封号 | 用户手动登录、低频率、随机间隔、单账号单任务、可配置上限 |
| 验证码 | 不主动绕过；遇到验证码暂停任务并通知用户在浏览器内手动完成 |
| HTML 结构频繁变化 | 每个 Adapter 配 e2e 冒烟测试；解析失败自动降级 LLM 兜底 |
| LLM 成本 / 延迟 | 关键路径用规则 + RapidFuzz；LLM 仅在排序与兜底解析时调用，结果缓存 |
| 简历隐私 | 本地 SQLite + 文件加密；提供「彻底删除」接口；不发送整份简历给第三方 LLM，必要时脱敏 |
| 并发与资源 | Playwright 实例数受限；任务级排队 + 单用户并发上限 |
| 合规 | UI 显著位置加免责声明；尊重 robots.txt；仅供学习研究 |

---

## 12. 编码与协作规范

- Python: PEP8 + ruff，类型注解必填，函数 ≤ 50 行，模块内单一职责。
- 提交信息: Conventional Commits (`feat:` / `fix:` / `chore:` ...)。
- 任何新增 Adapter 必须：实现 `SiteAdapter` 协议 + 至少 1 个 mock 集成测试 + 在 §9 与站点表中登记。
- 任何修改用户输入字段：同步更新 §6、Pydantic schema、前端表单、DB 迁移。
- 任何节点的输入输出必须是 `AgentState` 的子集，禁止隐式跨节点共享。
- 前端组件：单文件 ≤ 300 行；超过则拆子组件；样式使用 `<style scoped>`。
- 任务执行类操作必须可中断、可重试、可观测 (写 `task_progress`)。

---

## 13. 给 Claude 的工作指引

执行本项目相关任务时：
1. 先读本文档，定位目标模块；不在本文档范围内的需求，先与用户对齐再动手。
2. 修改架构 / 字段 / 接口前，先更新本文档对应小节，再写代码。
3. 站点 Adapter 开发遵循「先 mock，后真站点」的顺序，保证主流程可单测。
4. 涉及反爬或登录态的代码，必须在 PR 描述里说明是否绕过平台保护机制；如有，立即停止并询问用户。
5. 用户简历是敏感数据：禁止打印 / 上传到第三方服务、禁止写入日志、本地存储须加密。
6. 任何「投递」动作在未通过用户确认前，默认 `dry_run=True`。

---

_最后更新: 2026-05-23_

---
> Source: [gototrip1/Automated-resume-submission-Agent](https://github.com/gototrip1/Automated-resume-submission-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
