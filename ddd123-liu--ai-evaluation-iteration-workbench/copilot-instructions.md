## ai-evaluation-iteration-workbench

> 本文件是本仓库后续所有开发者和 AI Agent 的强制执行规则。任何实现、测试、文档和重构都必须遵守。需求已对齐；未获得项目负责人明确确认，不得偏离本文件。

# AI 评测自迭代工作台：强制项目规则

本文件是本仓库后续所有开发者和 AI Agent 的强制执行规则。任何实现、测试、文档和重构都必须遵守。需求已对齐；未获得项目负责人明确确认，不得偏离本文件。

## 1. 技术栈锁定

| 层级 | 唯一技术 |
| --- | --- |
| 前端 | Next.js App Router + TypeScript + shadcn/ui + ECharts |
| 后端 | FastAPI + Python 业务服务 |
| 数据访问与迁移 | SQLAlchemy + Alembic |
| 数据库 | SQLite |
| 定时调度 | APScheduler |
| 模型调用 | LiteLLM，且全部调用只经 `backend/app/services/llm_client.py` |
| 后端测试 | pytest |
| 端到端测试 | Playwright |
| 正式质量告警 | 飞书 Webhook |

后端仅监听 `127.0.0.1:8000`，固定使用一个 Uvicorn worker。前端仅运行于 `127.0.0.1:3000`。模拟客服 API 仅运行于 `127.0.0.1:8001`。

## 2. 禁止擅自引入的同类技术

禁止引入 Django、Flask、Starlette 独立应用、Node.js 后端、NestJS、Express、Spring、Go 服务、Celery、Redis、RabbitMQ、Kafka、PostgreSQL、MySQL、MongoDB、ORM 替代品、Prisma、Drizzle、SQLModel、其他任务调度器、其他图表库、其他前端组件库、其他端到端测试框架、Docker 编排、多进程部署、Gunicorn、多 Uvicorn worker。

禁止在 `llm_client.py` 之外导入 LiteLLM 或任何模型供应商 SDK。禁止提供第二套模型调用路径、第二个数据库、第二种 JSON 字段命名或第二套状态命名。

## 3. 项目目录结构

```text
backend/
  app/
    api/                 # FastAPI 路由、请求和响应序列化
    db/                  # SQLAlchemy engine、session、模型
    domain/              # 枚举、Pydantic 契约、常量
    repositories/        # 数据读写和短事务
    services/            # 业务服务
    jobs/                # APScheduler 作业注册与调度入口
    main.py              # FastAPI 应用和启动恢复
  alembic/               # 迁移脚本
  tests/                 # pytest
frontend/
  app/                   # Next.js App Router 页面
  components/            # shadcn/ui 组件和 ECharts 图表
  lib/                   # API client 和前端工具
  tests/e2e/             # Playwright
simulator-api/
  app/                   # 模拟客服 API
  data/                  # 100 条固定演示会话
docs/
  requirements/          # 权威 PRD
  superpowers/           # 设计与实施计划
  acceptance/            # 三层验收证据
```

## 4. 模块与文件职责

| 模块 | 唯一文件或目录职责 |
| --- | --- |
| API | `backend/app/api/` 只处理 REST 通信、输入校验、响应序列化和可读错误；不得承载业务编排。 |
| Service | `backend/app/services/` 只承载领域业务；一个服务文件只负责一个领域。 |
| Client | `backend/app/services/source_client.py` 只调用客服数据 API；`backend/app/services/llm_client.py` 是全部模型能力的唯一网关。 |
| Job | `backend/app/jobs/collection_job.py` 只注册和执行定时采集；调度器初始化只位于 `backend/app/jobs/scheduler.py`。 |
| Repository | `backend/app/repositories/` 只执行查询、写入和短事务；不得调用 HTTP、模型或调度器。 |
| 数据库 | `backend/app/db/` 只定义 engine、session 和 SQLAlchemy 表模型；迁移只能位于 `backend/alembic/`。 |
| 前端 API | `frontend/lib/api.ts` 是前端访问 FastAPI 的唯一入口。 |
| 模拟 API | `simulator-api/app/main.py` 只模拟外部客服会话 API；不得访问工作台 SQLite。 |

## 5. 硬性规则（8 条）

1. 所有模型调用必须且只能由 `llm_client.py` 发起；评测、评分标准草稿、归因、QA 草稿和模拟模型均无例外。
2. 采集顺序固定为：过滤 → 按稳定会话 ID 比例抽样 → 按发生时间排序 → 截取单次上限；不得调整。
3. 单条模型调用最多尝试 10 次；单条失败、解析异常或结果不完整不得中断整批。
4. 仅评测结果 `status="success"` 且 `pass=false` 可成为 Badcase；链路异常只进入异常率。
5. 所有指标复用同一公式：通过率仅以成功评测为分母；平均分仅统计成功评测；异常率以三类链路异常除以评测总量。
6. QA 产物状态只能按 `draft -> approved -> exported` 流转；未批准不可导出，批准后不可编辑，导出是终态。
7. 后端、模拟 API、前端仅监听本机回环地址；禁止公网或局域网监听，禁止多 worker、多进程和横向扩容。
8. 密钥、Token、飞书 Webhook 和模型原始敏感配置不得写入 SQLite、日志、接口响应、前端代码或导出文件。

## 6. 不可变约定

### 6.1 数据库表名

以下表名固定，迁移、模型、查询和外键不得改名：

| 表名 | 主键 | 用途 |
| --- | --- | --- |
| `data_source_configs` | `id` | 数据源配置 |
| `collection_runs` | `id` | 每次采集运行 |
| `collection_record_errors` | `id` | 采集单条映射失败 |
| `conversations` | `id` | 当前原始会话；`source_conversation_id` 全局唯一 |
| `evaluation_tasks` | `id` | 评测任务与当前标准 |
| `evaluation_runs` | `id` | 批量评测运行 |
| `evaluation_results` | `id` | 单条评测结果及快照 |
| `alert_rules` | `id` | 质量告警规则 |
| `alert_records` | `id` | 飞书或本地模拟通知记录 |
| `badcase_analyses` | `id` | Badcase AI 归因和人工复核 |
| `improvement_artifacts` | `id` | QA 草稿、审核和导出状态 |
| `export_batches` | `id` | 每次 CSV/JSONL 导出 |
| `export_batch_items` | `id` | 导出批次与产物关联 |
| `pipeline_runs` | `id` | 一键流水线运行 |
| `pipeline_stage_runs` | `id` | 流水线阶段状态和摘要 |
| `app_settings` | `key` | 单用户操作人和模型模式 |

固定表名数量：16。

### 6.2 跨阶段主键和关联字段

| 上游 | 下游 | 固定关联字段 |
| --- | --- | --- |
| `data_source_configs` | `collection_runs` | `collection_runs.data_source_config_id` |
| `collection_runs` | `collection_record_errors` | `collection_record_errors.collection_run_id` |
| `data_source_configs` | `conversations` | `conversations.data_source_config_id` |
| `evaluation_tasks` | `evaluation_runs` | `evaluation_runs.evaluation_task_id` |
| `evaluation_runs` | `evaluation_results` | `evaluation_results.evaluation_run_id` |
| `conversations` | `evaluation_results` | `evaluation_results.conversation_id` |
| `evaluation_results` | `badcase_analyses` | `badcase_analyses.evaluation_result_id`，唯一 |
| `badcase_analyses` | `improvement_artifacts` | `improvement_artifacts.badcase_analysis_id`，唯一 |
| `export_batches` | `export_batch_items` | `export_batch_items.export_batch_id` |
| `improvement_artifacts` | `export_batch_items` | `export_batch_items.improvement_artifact_id` |
| `data_source_configs` | `pipeline_runs` | `pipeline_runs.data_source_config_id` |
| `evaluation_tasks` | `pipeline_runs` | `pipeline_runs.evaluation_task_id` |
| `pipeline_runs` | `pipeline_stage_runs` | `pipeline_stage_runs.pipeline_run_id` |

### 6.3 API 与前端 JSON 命名

后端 API、前端 `frontend/lib/api.ts`、Pydantic 模型和导出字段之外的所有 JSON 一律使用 `snake_case`。数据库列名与 JSON 字段名一致。禁止 camelCase、PascalCase、中文键名和同义字段。

五个固定采集映射键只能是：

```text
conversation_id
occurred_at
user_query
ai_response
context
```

`channel`、`transferred_to_human`、`order_info` 是固定可选扩展字段，不属于五个映射键，且不得替代任一映射键。

`context` 中每条消息的角色枚举固定为：

```text
user
assistant
```

禁止接受或保存 `system`、`tool` 或任何其他角色值。`user_query` 和 `ai_response` 是独立标准字段，不得从 `context` 临时推导。

### 6.4 采集计数字段与公式

`collection_runs` 的八个计数字段固定为：

| 字段 | 固定含义 |
| --- | --- |
| `source_record_count` | 源 API 本次所有分页返回的记录总数 |
| `mapping_failed_count` | 本次映射失败且写入 `collection_record_errors` 的记录数 |
| `matched_record_count` | 映射成功且符合过滤条件、抽样前的记录数 |
| `sampled_record_count` | 固定 seed 比例抽样后、上限截取前的记录数 |
| `selected_record_count` | 按单次上限截取后应执行入库的记录数 |
| `inserted_record_count` | 原库不存在稳定会话 ID 而新增的记录数 |
| `updated_record_count` | 原库已存在稳定会话 ID 而更新的记录数 |
| `stored_record_count` | 实际成功新增或更新的记录数 |

以下公式必须同时成立：

```text
source_record_count >= mapping_failed_count + matched_record_count
sampled_record_count <= matched_record_count
selected_record_count <= sampled_record_count
stored_record_count = inserted_record_count + updated_record_count
stored_record_count <= selected_record_count
```

### 6.5 数据源类型与模型配置分组

数据源类型唯一值为 `http_api`。模拟客服 API 也是 `http_api`，通过 `is_simulated=true` 区分；不得创建 CSV、文件导入、数据库直连或消息队列数据源类型。

模型配置仅分为以下两组：

| 分组 | `app_settings.key` | 用途 |
| --- | --- | --- |
| 真实模型 | `model_mode=real` | `llm_client.py` 经 LiteLLM 调用真实模型；凭证只来自环境变量。 |
| 演示模型 | `model_mode=simulated` | `llm_client.py` 使用确定性本地模拟模型；仅用于本地演示和测试。 |

### 6.6 裁判输出、pass 和归因

裁判 JSON 输出只能为：

```json
{"score": 1, "pass": false, "reason": "具体理由"}
```

- `score` 是 1 至 5 的整数。
- `pass` 是布尔值，由评分标准决定；内置电商客服标准固定为 `score >= 4` 时 `pass=true`，否则 `pass=false`。
- `reason` 是非空字符串。
- 模型输出缺字段、类型不合法或分值不合法时，评测结果状态是 `incomplete_result`。
- 仅 `success` 且 `pass=false` 生成 Badcase。

归因枚举固定为：

```text
missing_data
reply_style_issue
fabricated_fact
violation
undetermined
```

前端展示名称依次固定为：知识缺失、回复方式问题、编造事实、违规、无法判断。

### 6.7 所有状态枚举和流水线阶段

| 对象 | 固定枚举 |
| --- | --- |
| 数据源配置 `status` | `enabled`, `disabled` |
| 采集运行 `status` | `running`, `completed`, `failed` |
| 评测运行 `status` | `running`, `completed`, `failed` |
| 评测结果 `status` | `success`, `call_failed`, `parse_error`, `incomplete_result` |
| 告警规则 `status` | `enabled`, `disabled` |
| 告警记录 `delivery_status` | `sent`, `failed`, `simulated` |
| 归因 `status` | `ai_assessed`, `human_reviewed` |
| QA 产物 `status` | `draft`, `approved`, `exported` |
| 流水线运行 `status` | `running`, `completed`, `failed` |
| 流水线阶段 `stage` | `collection`, `evaluation`, `alert_check`, `attribution`, `artifact_generation` |
| 流水线阶段 `status` | `pending`, `running`, `completed`, `failed` |

流水线显示顺序固定为：`collection -> evaluation -> alert_check -> attribution -> artifact_generation -> completed`。`failed` 是流水线终态，不是阶段。

### 6.8 Service、client 和 job 模块名称

以下模块名固定：

```text
services/collection_service.py
services/evaluation_service.py
services/evaluation_parser.py
services/llm_client.py
services/metrics_service.py
services/alert_service.py
services/attribution_service.py
services/artifact_service.py
services/export_service.py
services/pipeline_service.py
services/recovery_service.py
services/demo_seed_service.py
services/source_client.py
jobs/scheduler.py
jobs/collection_job.py
```

### 6.9 前端固定路由

```text
/
/dashboard
/pipelines
/sources
/evaluation-tasks
/evaluation-results/[id]
/alerts
/badcases
/improvement-artifacts
/settings
```

不得新增第二套同义页面路由或以旧路由兼容新路由。

### 6.10 QA 导出字段及顺序

CSV 和 JSONL 导出的字段及顺序固定为：

```text
standard_question
reference_answer
source_case_id
category
```

## 7. 需求变更处理

任何需求变更先检查是否触及第 6 节不可变约定。

- 未触及不可变约定：先更新 `docs/requirements/ai-evaluation-iteration-workbench-prd.md`，再更新实施计划和测试。
- 触及不可变约定：先获得项目负责人明确确认；随后在同一变更中先更新 PRD，再更新本 `AGENTS.md`，最后更新设计、计划、迁移、后端、前端和测试。
- 未完成上述文档更新前，不得实施变更。
- 不得以兼容层、别名、双写、双状态或并行路由规避不可变约定变更。

## 8. 模块完成与验收标准

| 模块 | 完成条件 | 验收证据 |
| --- | --- | --- |
| 模拟 API 与采集 | Bearer 鉴权、时间分页、五键映射、失败记录、固定采样顺序、会话更新和采集历史完整。 | pytest 结果、`collection_runs`/`collection_record_errors`/`conversations` 行、页面采集历史。 |
| 调度 | 同一数据源仅一个有效作业，新规则替换旧规则。 | pytest 和 APScheduler job ID `collection:{config_id}`。 |
| 评测 | 所有调用经过 `llm_client.py`，10 次重试、Markdown JSON 容错、四类结果状态、输入快照和进度落库。 | pytest、`evaluation_runs`/`evaluation_results` 行、评测结果页面。 |
| 看板与告警 | 三项指标公式唯一，趋势正确，飞书告警或本地模拟记录可追溯，冷却和连续命中生效。 | pytest、`alert_records` 行、看板和告警页面。 |
| 归因 | 仅 Badcase 归因，五枚举固定，未知值归 `undetermined`，只允许人工复核修正。 | pytest、`badcase_analyses` 行、Badcase 页面。 |
| QA 产物 | 仅 `missing_data` 生成一份草稿，审核前不可导出，审核后锁定，重复导出留痕。 | pytest、`improvement_artifacts`/`export_batches` 行、CSV/JSONL 和页面。 |
| 流水线 | 固定五阶段，空结果为完成，后续失败不回滚，服务中断后未完成运行标记失败。 | pytest、`pipeline_runs`/`pipeline_stage_runs` 行、流水线页面。 |
| 前端 | 统一导航、固定路由、全部表单永久标签、错误可读、运行状态可查看。 | Playwright、实际页面核对。 |
| 整体 | 100 条确定性演示数据覆盖成功与异常路径，三层验收完成。 | pytest、Playwright、SQLite 查询、页面截图/观察、导出文件、告警记录。 |

## 9. AGENTS.md 自检

- 硬性规则数量：8 条。
- 固定表名数量：16 个。
- Prompt 必需插槽完整：`user_query`、`ai_response`、`context`、评分标准文本；评分标准草稿额外使用 `evaluation_goal`、`reference_examples`；归因额外使用评测 `score`、`pass`、`reason`；QA 草稿额外使用归因 `category`、`description`、`recommendation`。
- 占位符：无 TODO、TBD、待确认或不可执行占位说明。
- 前后矛盾：无；模型调用、数据源、状态、端口、字段和外推边界均只有一套定义。

---
> Source: [DDD123-LIU/ai-evaluation-iteration-workbench](https://github.com/DDD123-LIU/ai-evaluation-iteration-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
