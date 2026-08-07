## ai-testing-evaluation

> 本项目是单用户、单服务器部署的「AI 测评自迭代工作台」。

# AGENTS.md

## 项目说明

本项目是单用户、单服务器部署的「AI 测评自迭代工作台」。

核心闭环固定为：

1. 导入客服对话和知识资料。
2. 按固定规则分层抽样。
3. 使用 OpenAI API 逐条评测。
4. 隔离技术异常并统计质量指标。
5. 通过飞书发送质量预警。
6. 对知识缺失案例生成 QA 草稿。
7. 人工审核后导出，不自动发布知识库。

本文约定适用于整个仓库。任何代码生成、重构和功能实现都必须遵守。

## 技术栈锁定

未经用户明确批准，不得擅自替换、升级主版本或并行引入替代方案。

### 后端

- Python 3.12
- Django 5.2 LTS
- Celery 5
- Redis 7
- PostgreSQL 16
- pgvector
- OpenAI Python SDK
- Gunicorn

### 前端

- Django Templates
- HTMX
- Alpine.js
- ECharts
- 原生 CSS

禁止擅自引入 React、Vue、Next.js、Nuxt、独立 SPA 或第二套前端构建体系。

### 测试与质量

- pytest
- pytest-django
- ruff
- mypy
- factory_boy

### 部署

- Docker
- Docker Compose
- Nginx
- 单台 Linux 服务器

禁止拆成微服务、引入 Kubernetes、Kafka、RabbitMQ、Elasticsearch 或独立向量数据库。

## 目录结构

```text
.
├── AGENTS.md
├── manage.py
├── pyproject.toml
├── docker-compose.yml
├── config/
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── test.py
│   │   └── production.py
│   ├── urls.py
│   ├── celery.py
│   ├── asgi.py
│   └── wsgi.py
├── apps/
│   ├── core/
│   │   ├── enums.py
│   │   ├── security.py
│   │   ├── redaction.py
│   │   └── exceptions.py
│   ├── accounts/
│   ├── ingestion/
│   │   └── services/
│   │       ├── parser.py
│   │       ├── mapping.py
│   │       └── importer.py
│   ├── knowledge/
│   │   └── services/
│   │       ├── parsers.py
│   │       ├── indexer.py
│   │       └── retriever.py
│   ├── evaluations/
│   │   ├── schemas.py
│   │   ├── tasks.py
│   │   └── services/
│   │       ├── sampling.py
│   │       ├── batches.py
│   │       ├── evaluator.py
│   │       ├── validation.py
│   │       ├── openai_client.py
│   │       └── orchestrator.py
│   ├── improvements/
│   │   └── services/
│   │       ├── diagnosis.py
│   │       ├── qa_generator.py
│   │       ├── workflow.py
│   │       └── exporter.py
│   ├── alerts/
│   │   └── services/
│   │       ├── rules.py
│   │       └── feishu.py
│   └── dashboard/
│       └── services/
│           └── metrics.py
├── templates/
│   ├── accounts/
│   ├── ingestion/
│   ├── knowledge/
│   ├── evaluations/
│   ├── improvements/
│   └── dashboard/
├── static/
│   ├── css/
│   └── js/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   ├── golden/
│   └── factories/
├── deploy/
│   ├── nginx.conf
│   ├── docker-compose.prod.yml
│   ├── backup.sh
│   └── restore.sh
└── docs/
    ├── architecture.md
    └── operations.md
```

目录职责不得混用：

- View 只处理 HTTP 输入输出，不承载业务规则。
- Model 只保存数据结构和局部数据约束。
- `services/` 承载可复用业务逻辑。
- Celery Task 只负责调度、状态更新和调用 Service。
- OpenAI 调用只能经过 `evaluations.services.openai_client`。
- 枚举只能定义在 `apps/core/enums.py`。

## 硬性规则

1. **不得擅自更换技术栈。** 不得引入第二套 Web 框架、任务队列、数据库、向量库或前端框架；确需更换时必须先更新设计文档并获得用户确认。

2. **大模型输出永远是不可信输入。** 所有返回必须经过严格 JSON Schema、枚举、分数范围、引用 ID 和业务规则校验；总分由系统重新计算，一票否决由代码重新执行。

3. **技术失败不得污染质量指标。** 网络错误、限流、模型拒答、格式异常和 `insufficient_input` 均不进入合格率分母，必须单独统计和展示。

4. **不得静默截断、覆盖或纠正业务数据。** 超长对话应标记失败；重复 ID 内容冲突必须人工处理；历史评测、引用和已导出 QA 不得被新数据覆盖。

5. **敏感数据必须先脱敏再离开服务器。** 原始姓名、手机号、邮箱、地址、订单号、密钥和 Webhook 不得进入模型请求日志、异常日志、任务参数或前端响应。

6. **所有异步任务必须幂等。** Worker 重启、任务重投和人工续跑不得重复保存结果、重复生成 QA、重复发送预警或重复计算已完成项目。

7. **不得扩大自动化权限。** 系统不得自动发布知识库、联系客户、修改线上客服模型、调整提示词、改变评分标准或删除业务数据。

8. **跨阶段契约不得就地修改。** 字段名、枚举、状态机、端口和模块名只能通过显式迁移与版本升级变更；不得为了局部实现方便重命名或增加同义值。

## 不可变约定

以下约定跨越数据导入、评测、展示、预警和改进产物阶段，视为系统契约。

### 服务与端口

| 服务 | 容器名 | 内部端口 | 对外暴露 |
|---|---|---:|---|
| Nginx | `nginx` | 80/443 | 80/443 |
| Django | `web` | 8000 | 仅开发环境映射 8000 |
| Celery Worker | `worker` | 无 | 不暴露 |
| PostgreSQL | `postgres` | 5432 | 仅本机开发环境 |
| Redis | `redis` | 6379 | 仅本机开发环境 |

生产环境只允许 Nginx 的 80/443 端口公开访问。

### Django 应用命名

以下模块名固定：

```text
core
accounts
ingestion
knowledge
evaluations
improvements
alerts
dashboard
```

不得使用以下同义模块名另建重复模块：

```text
users
imports
rag
evals
qa
notifications
analytics
```

### 对话标准字段

```text
conversation_id
started_at
channel
business_type
messages
satisfaction
transferred_to_human
```

其中必填字段固定为：

```text
conversation_id
started_at
channel
business_type
messages
```

单条消息字段固定为：

```text
role
content
timestamp
```

消息角色枚举固定为：

```text
customer
assistant
human_agent
```

### 导入限制

```text
单次最大对话数：10000
单条消息最大字符数：8000
单段对话最大字符数：40000
单个知识文件最大体积：20MB
单个知识文件最大解析字符数：1000000
```

不得静默截断超限内容。

### 批次与任务状态

```text
pending
running
retrying
paused
completed
partial_failed
failed
cancelled
```

不得使用 `processing`、`done`、`error` 等同义状态。

### 评测维度

维度字段、权重和顺序固定为：

```text
factual_accuracy        35
intent_understanding    20
solution_completeness   20
communication_quality   15
compliance_risk         10
```

五个维度均输出 `0～100` 分，允许一位小数。系统按上述权重加权重算总分并保留一位小数。

合格线固定为：

```text
80
```

### 评测输出结构

模型输出固定包含 12 个顶层字段：

```text
verdict
total_score
dimension_scores
hard_fail
hard_fail_reasons
primary_root_cause
secondary_tags
overall_reason
dimension_reasons
evidence_citations
knowledge_coverage
confidence
```

`verdict` 枚举：

```text
qualified
unqualified
needs_review
```

`hard_fail_reasons` 枚举：

```text
factual_error
unsupported_claim
severe_compliance_risk
```

`knowledge_coverage` 枚举：

```text
covered
partial
missing
uncertain
```

`confidence` 枚举：

```text
high
medium
low
```

### 知识引用结构

每个 `evidence_citations` 项固定包含：

```text
chunk_id
document_id
source_name
locator
quote
```

引用的 `chunk_id` 和 `document_id` 必须真实存在，不接受模型自行生成的引用 ID。

### 主根因枚举

```text
knowledge_missing
retrieval_failure
intent_misunderstanding
answer_execution_error
insufficient_input
```

合格案例的 `primary_root_cause` 必须为 `null`。

### 次级标签枚举

```text
hallucination
wrong_fact
incomplete_answer
wrong_step
irrelevant_answer
poor_tone
repetitive_answer
policy_violation
privacy_risk
missing_context
```

不得由模型生成枚举外的自由标签。

### QA 状态

```text
generation_failed
draft
approved
rejected
exported
```

合法状态流转固定为：

```text
generation_failed → draft
draft → approved
draft → rejected
rejected → draft
approved → exported
approved → draft
```

`approved → draft` 只在已通过内容再次被编辑时发生。

### QA 导出字段

CSV 字段名和顺序固定为：

```text
question
suggested_answer
source_conversation_id
evidence_sources
review_status
reviewed_at
```

### 模型与知识配置

默认配置固定为：

```text
评测模型：gpt-5.6-terra
推理强度：low
嵌入模型：text-embedding-3-small
初次检索数量：5
知识诊断检索数量：20
Worker 默认并发：4
Worker 可配置范围：1-16
单请求超时：120 秒
最大尝试次数：4
```

每条评测结果必须保存：

```text
model_name
prompt_version
schema_version
knowledge_version
input_tokens
output_tokens
estimated_cost
```

### 质量指标口径

合格率固定使用：

```text
qualified / (qualified + unqualified)
```

以下记录不进入合格率分母：

```text
needs_review
insufficient_input
technical_failure
model_output_invalid
cancelled
```

技术失败率固定使用：

```text
技术失败终态数量 / 批次总抽样数量
```

`technical_failure` 和 `model_output_invalid` 都属于技术失败终态。

### 预警约定

只有 `qualified + unqualified` 数量不少于 50 条时才判断预警。

触发条件固定为：

```text
合格率 < 80%
或
相比最近 7 个有效批次平均值下降 >= 10 个百分点
```

触达渠道固定为飞书群机器人 Webhook。

同一批次的多个触发条件必须合并为一条消息，同一批次不得重复发送相同预警。

最近 7 个有效批次必须与当前批次使用相同筛选范围、模型、提示词和 Schema 版本，且每批 `qualified + unqualified` 不少于 50 条。不足 7 个历史有效批次时不判断下降规则。

### 补充状态与版本规则

- `paused`、`partial_failed` 只用于批次，`retrying` 主要用于单评测任务。
- 暂停只阻止派发新任务，不强制中断运行中的模型请求。
- 批次正式启动时冻结启用知识版本快照。
- 相同 `conversation_id` 内容冲突经人工确认后创建新版本，不原地覆盖历史对话。

---
> Source: [BROccoLi-921/AI-testing-evaluation](https://github.com/BROccoLi-921/AI-testing-evaluation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
