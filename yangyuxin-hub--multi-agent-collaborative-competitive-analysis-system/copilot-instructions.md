## multi-agent-collaborative-competitive-analysis-system

> 字节 AI 全栈挑战赛（2026.05.20 - 2026.06.10）

# 主 Agent 协作式竞品分析系统

字节 AI 全栈挑战赛（2026.05.20 - 2026.06.10）

输入产品名 → **Lead Analyst 主 Agent** 自主规划、调用 tools、并发派 **Researcher Subagent** → 带引用的报告。Claude Code 风格 agentic 系统，专项化用于竞品分析。

## 开发原则

1. Demo 主路径优先：输入 → 澄清 → 主 agent 调研 → 报告 PDF
2. V1 禁用 Redis / ChromaDB / MinIO / Celery，本地文件 + PG 足够
3. **sources 由 tool 强约束**：所有引用走 tool 写入 sources_pool；`write_section` schema 强制 `source_ids` 非空且在 pool 中
4. 先 mock 后真实；每个 commit 只动一处；失败返结构化 error 让主 agent 自决
5. 三个 case 始终能跑：Notion AI / Cursor / Linear，合并前必回归
6. 预算自感知：system prompt 明示预算 + `check_budget` tool

**Non-Goals**：账号 / 知识库 / 定时监控 / 任务队列 / 多租户 / 模板编辑器 / 国际化 / 移动端 / 多类 subagent

## 架构

**Lead Analyst**（`claude-sonnet-4-6` + **Claude Agent SDK**）：单循环 think → tool_use → observe，自主完成规划/澄清/派 subagent/综合/写报告。预算 ≤ **$0.05**、≤ **90s**，硬超时 8 分钟。system prompt 启用 `cache_control`。

**Tools**

| Tool | 实现 | 说明 |
|------|------|------|
| `web_search(query)` | Tavily / Serper | 分配 source_id 写入 pool |
| `fetch_page(url)` | BS4 + readability | 写入 pool 返回 id + 摘要 |
| `extract_fields(text, schema, source_ids)` | **Haiku 4.5** | 文本 → 结构化 |
| `ask_user(questions)` | WS interrupt | 多选卡片，30s 兜底 default |
| `write_section(title, content, source_ids)` | 校验入库 | **强制 source_ids 非空且在 pool 中** |
| `check_budget()` | 内部计数 | 剩余 token / 时间 / cost |

**Researcher Subagent**（仅 1 类 `research_competitor(name)`，Sonnet 4.6）：主 agent 确认竞品后并发派 5 个，独立 thread + 上下文隔离，工具子集 `web_search / fetch_page / extract_fields`，返回结构化卡片，单实例 ≤ $0.005 / 30s。**对比 / SWOT / 写报告由主 agent 自己做，不拆 subagent。**

**执行流**

```
Lead → 规划 → ask_user(澄清) → web_search → 并发 5× Researcher
     → 综合(矩阵/SWOT/象限，缺字段再派 subagent) → check_budget
     → write_section × N
```

无质量门，硬重试由 system prompt 引导。

**引用追踪三道防线**：① tool 入口分配 source_id ② `write_section` schema 校验 ③ 前端二次校验。主 agent 写报告只有 `write_section` 一条路径。

**WS 事件**：`text_delta` / `tool_use` / `tool_result` / `task_spawn` / `task_result` / `interrupt` / `resume` / `budget_update` / `done` / `failed`。每事件带 `seq`，断线按 `last_seq` 补推。

## 技术栈

- **前端**：Next.js 14 + TS + Tailwind + shadcn/ui + Recharts；WS 事件流 reducer 渲染（嵌套 subagent 卡片）
- **后端**：FastAPI + **Claude Agent SDK** + Anthropic Claude (Sonnet 4.6 / Haiku 4.5) + Tavily/Serper + BS4 + WeasyPrint
- **存储**：PostgreSQL（任务/事件流/sources_pool/报告）+ 本地文件（PDF/快照）

## API

| | Path | 说明 |
|--|------|------|
| `POST` | `/api/tasks` | `{product, options?}` → `{task_id}` |
| `GET` | `/api/tasks/{id}` | 任务状态 + 事件流 |
| `GET` | `/api/reports/{id}[/pdf]` | 报告 JSON 或 PDF |
| `WS` | `/ws/tasks/{id}` | 订阅事件流 |

## 项目结构

```
backend/app/
├── api/                  # 路由 + WS
├── agent/                # lead.py / researcher.py / prompts/*.md
├── tools/                # search / fetch / extract / ask_user / write_section / budget
├── core/                 # sources_pool / event_bus / budget_tracker / config
├── models/  mocks/

frontend/src/components/
├── EventStream/          # TextDelta / ToolUseCard / SubagentCard / InterruptModal
├── BudgetMeter/  ReportViewer/  ComparisonTable/
```

## 启动

```bash
docker-compose up -d postgres
cd backend && uv sync && uv run fastapi dev app/main.py
cd frontend && pnpm install && pnpm dev

# 检查
cd backend && uv run ruff check . && uv run pytest tests/e2e/ -v
cd frontend && pnpm lint && pnpm type-check
```

## 验收用例

| Case | 产品 | 预期竞品 | 报告维度 |
|------|------|---------|--------|
| 1 | Notion AI | Coda AI / ClickUp AI / Confluence AI / Microsoft Copilot | 功能 / 定价 / 痛点 |
| 2 | Cursor | GitHub Copilot / Windsurf / Cline / Continue | 模型 / 定价 / IDE 集成 |
| 3 | Linear | Jira / Asana / Height / Shortcut | 协作 / 定价 / 团队规模 |

每个 case 存 `data/fixtures/{case}/`，5 文件标准结构：

| 文件 | 用途 |
|------|------|
| `task.json` | 用户输入 + clarifier 答案，离线回放固定输入 |
| `researcher_cards.json` | subagent 结构化产出，e2e 比对 / mock |
| `sources_pool.json` | 引用池（id / url / title / publisher / fetched_at / snippet），**反幻觉核心** |
| `report.md` | 标杆 baseline 报告 |
| `events.jsonl` | 按 `seq` 编号的 WS 事件流，Demo 回放 + 重连回归 |

**fixture 是真值快照**：时效字段标 `as_of`，禁止 `TBD` 占位（违反 `write_section` 校验）。

## 创新点（答辩）

| 维度 | 创新点 |
|------|--------|
| 技术 | Claude Code 范式专项化 — SDK 主循环 + 专用 subagent，比 DAG 灵活、比通用 agent 可控 |
| 工程 | 引用追踪三道防线 — 从根本上消除 URL 幻觉 |
| 人机协同 | 主 agent 自主调用澄清 — 模型判断何时该问 |
| 体验 | 事件流可视化 — 思考链 + 嵌套 subagent + 实时预算条 |

## 兜底

- Tavily 限额 → 切 Serper / DuckDuckGo + 预跑缓存
- Claude 限流 → 自动降级 Haiku
- 反爬 → `fetch_page` 返 error，主 agent 换 URL 或派子 agent
- 主 agent 跑偏 → 预算耗尽强 `interrupt` 出报告
- SDK 不支持某能力 → 自建 fallback（特殊异常 + WS）
- Demo 网络故障 → 离线事件流回放

## 时间节点

| 日期 | 目标 |
|------|------|
| 5/20 | Agent SDK 最小 demo（主 agent + 1 tool + 1 subagent + interrupt）|
| 5/23 | 6 tools + Researcher 就位，端到端粗糙报告 |
| 5/27 | 三道防线闭环，三个 case 全引用 + ≤ $0.05 |
| 5/31 | 前端事件流可视化完成 |
| 6/3 | 报告质量达标 |
| 6/5 | PDF + 离线事件流缓存 |
| 6/7 | 功能冻结，做 PPT + 视频 |
| 6/10 | 提交截止 |

## 环境变量

```env
ANTHROPIC_API_KEY=
TAVILY_API_KEY=
SERPER_API_KEY=
DATABASE_URL=postgresql://localhost:5432/competitor_analysis
DATA_DIR=./data
```

## 规范

Tool 返 Pydantic 校验；LLM tenacity 3 次指数退避；SDK 事件全部入库（支持回放/重连）；Python ruff / TS strict；提交前必跑 e2e。

---
> Source: [yangyuxin-hub/Multi-Agent-Collaborative-Competitive-Analysis-System](https://github.com/yangyuxin-hub/Multi-Agent-Collaborative-Competitive-Analysis-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
