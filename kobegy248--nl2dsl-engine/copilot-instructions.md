## nl2dsl-engine

> > **定位**: Why / What / Where 的入口。不负责 How。

# NL2DSL — 项目导航中心 (Knowledge Hub)

> **定位**: Why / What / Where 的入口。不负责 How。
> 详细设计请查阅 `docs/` 目录对应文档。

---

## 1. Project Philosophy

NL2DSL 不是 NL2SQL 项目。它是一个 **Governance-Aware Semantic Query Engine**。

- **SQL 是实现细节**，DSL 是语义契约
- **Governance 是数据源真理**，不是事后补丁
- LLM 只负责语义理解，系统负责执行治理

### 五条核心原则

| 原则 | 含义 |
|------|------|
| **Governance First** | 所有查询必须经过权限注入、敏感字段脱敏、审计日志记录。安全不是可选项 |
| **Semantic First** | 指标、维度、术语必须语义层注册。禁止自由文本字段名，禁止发明未定义的指标 |
| **DSL First** | LLM 只生成结构化 DSL，不生成 SQL。DSL 可校验、可修正、可做权限控制 |
| **Explainability First** | 每个查询必须可解释：用了什么指标、什么过滤条件、为什么这样聚合 |
| **Security First** | SQL 执行前必须经过扫描。禁止 DELETE/UPDATE/DROP/UNION/注释注入/多语句 |

---

## 2. Project Architecture Overview

高层架构：

```
Natural Language
       ↓
Intent Recognition（意图识别）
       ↓
Query Planner（查询规划）
       ↓
DSL（领域特定语言）← 语义契约
       ↓
Validation（校验与修正）
       ↓
SQL Generation（SQL 构建）
       ↓
Execution（执行与扫描）
       ↓
Explanation（结果解释）
```

双层解耦：
- **Agent 层**（宏观编排）：意图识别 → 任务分解 → 子查询调度 → 结果聚合 → 自然语言解释
- **Graph 层**（微观执行）：单查询 DSL → 校验 → 权限注入 → SQL 构建 → 扫描 → 执行

详细架构 → [`docs/architecture/02-system-architecture.md`](docs/architecture/02-system-architecture.md)
LangGraph 节点流程 → [`docs/agent/31-langgraph-workflow.md`](docs/agent/31-langgraph-workflow.md)

---

## 3. Documentation Navigation

> 以下按主题组织。每个文档附一句职责说明。
> 接到任务时，先定位主题，再读对应文档，最后读代码。

### Architecture（架构设计）

| 文档 | 职责说明 |
|------|------|
| [`docs/architecture/01-overview.md`](docs/architecture/01-overview.md) | 项目背景、设计目标（可校验/可优化/可治理/可扩展）、技术选型总览 |
| [`docs/architecture/02-system-architecture.md`](docs/architecture/02-system-architecture.md) | 整体架构图、数据流、模块边界 |
| [`docs/architecture/03-sql-engine.md`](docs/architecture/03-sql-engine.md) | SQLAlchemy Core 构建、sqlglot 方言转换、Query Planner |
| [`docs/architecture/04-deployment.md`](docs/architecture/04-deployment.md) | Docker Compose 部署、环境变量、性能调优 |
| [`docs/architecture/sql-execution-design.md`](docs/architecture/sql-execution-design.md) | SQL 执行层：连接池、异步执行、结果返回 |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | 面向开发者的完整架构文档，含与传统 NL2SQL 区别、API 参考 |

### DSL & API（语义契约与接口）

| 文档 | 职责说明 |
|------|------|
| [`docs/api/20-dsl-spec.md`](docs/api/20-dsl-spec.md) | DSL Schema 定义：metrics/dimensions/filters/order_by/limit 字段规范 |
| [`docs/api/21-api-contract.md`](docs/api/21-api-contract.md) | HTTP API 接口定义：/query /query/dsl /query/execute 请求/响应格式 |
| [`docs/api/22-error-handling.md`](docs/api/22-error-handling.md) | 错误分类、HTTP 状态码、歧义响应格式 |
| [`docs/business/11-dsl-validation.md`](docs/business/11-dsl-validation.md) | DSL 校验规则、风险控制层级、SQL 执行前正则扫描 |

### Business Layer（业务语义层）

| 文档 | 职责说明 |
|------|------|
| [`docs/business/10-semantic-layer.md`](docs/business/10-semantic-layer.md) | YAML 指标/维度注册、value_map 枚举映射、数据血缘与 Join 推导 |
| [`docs/business/12-permission.md`](docs/business/12-permission.md) | 行级权限注入规则、列级敏感字段黑名单、数据脱敏策略 |
| [`docs/business/13-business-rules.md`](docs/business/13-business-rules.md) | 术语表映射、Prompt 显式注入、歧义反问机制、时间语义处理 |

### Query Processing（查询处理）

| 文档 | 职责说明 |
|------|------|
| [`docs/query/query-clarification-design.md`](docs/query/query-clarification-design.md) | 歧义检测：时间缺失、指标歧义、维度歧义、比较基准歧义 |
| [`docs/query/query-sandbox-design.md`](docs/query/query-sandbox-design.md) | 查询沙箱：EXPLAIN 预估、LIMIT 预览、执行超时检测 |

### Agent & LangGraph（智能编排与执行管道）

| 文档 | 职责说明 |
|------|------|
| [`docs/agent/30-rag-design.md`](docs/agent/30-rag-design.md) | 向量检索：4 集合设计、混合检索策略、Milvus 集合结构 |
| [`docs/agent/31-langgraph-workflow.md`](docs/agent/31-langgraph-workflow.md) | StateGraph 完整节点流程图、条件分支、自检重试、链路追踪 |
| [`docs/agent/32-metadata-sync.md`](docs/agent/32-metadata-sync.md) | 元数据提取、向量库同步、增量更新策略 |
| [`docs/agent/33-testing.md`](docs/agent/33-testing.md) | 单元/集成/E2E 三层测试策略概览 |
| [`docs/agent/34-llm-risks.md`](docs/agent/34-llm-risks.md) | LLM 成本/延迟/幻觉/安全风险及缓解方案 |

### Planner（查询规划）

| 文档 | 职责说明 |
|------|------|
| [`docs/planner/query-optimization-design.md`](docs/planner/query-optimization-design.md) | 查询优化器：谓词下推、投影下推、Join 重排序（预留架构） |

### Optimizer（语义优化器）

| 文档 | 职责说明 |
|------|------|
| [`docs/specs/semantic-optimizer-architecture-v2.md`](docs/specs/semantic-optimizer-architecture-v2.md) | Optimizer 架构设计：Normalizer → Rule Engine → CanonicalResolver 三层管道 |
| [`docs/specs/semantic-optimizer-error-taxonomy-v2.md`](docs/specs/semantic-optimizer-error-taxonomy-v2.md) | 26 种错误类型分类体系（9 大类），含 Confidence 机制 |
| [`docs/specs/semantic-optimizer-implementation-plan-v2.md`](docs/specs/semantic-optimizer-implementation-plan-v2.md) | 5 Phase 实施计划（P0-P5）、里程碑、测试策略 |
| [`docs/evaluation/optimizer-guide.md`](docs/evaluation/optimizer-guide.md) | Optimizer 使用指南：CLI 参数、规则分类、报告解读 |

### Audit & Feedback（审计与反馈）

| 文档 | 职责说明 |
|------|------|
| [`docs/audit/audit-log-design.md`](docs/audit/audit-log-design.md) | 审计日志：数据模型、存储策略、查询接口、保留策略 |
| [`docs/feedback/feedback-loop-design.md`](docs/feedback/feedback-loop-design.md) | 反馈闭环：收集机制、存储格式、用于模型改进的流程 |

### Configuration（配置系统）

| 文档 | 职责说明 |
|------|------|
| [`docs/configuration/schema-reference.md`](docs/configuration/schema-reference.md) | 配置 Schema 参考：metrics/terms/intents/permissions/history 完整字段 |

### Evaluation（评测框架）

| 文档 | 职责说明 |
|------|------|
| [`docs/specs/evaluation-design.md`](docs/specs/evaluation-design.md) | 4 大类 12 维度量化评估框架设计 |
| [`docs/evaluation/framework-guide.md`](docs/evaluation/framework-guide.md) | 评估框架使用指南：添加用例、运行评估、解读报告、自定义权重 |

### Specs & Reports（专项设计与过程报告）

| 文档 | 职责说明 |
|------|------|
| [`docs/specs/2026-05-19-failover-system-design.md`](docs/specs/2026-05-19-failover-system-design.md) | 生产级兜底：Retry Chain、Query Sandbox、Clarification 机制 |
| [`docs/specs/semantic-optimizer-error-taxonomy-v2.md`](docs/specs/semantic-optimizer-error-taxonomy-v2.md) | Semantic Query Optimizer V1 错误分类体系（9 大类 26 种） |
| [`docs/specs/semantic-optimizer-architecture-v2.md`](docs/specs/semantic-optimizer-architecture-v2.md) | Semantic Query Optimizer V1 架构设计、Rule Engine、Evaluation 集成 |
| [`docs/specs/semantic-optimizer-implementation-plan-v2.md`](docs/specs/semantic-optimizer-implementation-plan-v2.md) | Semantic Query Optimizer V1 实施计划（6 Phase、里程碑、测试策略） |
| [`docs/reports/e2e_report.md`](docs/reports/e2e_report.md) | 253 个 E2E 测试通过报告，含查询链路 Trace 分析 |
| [`docs/reports/complex_nl_query_analysis.md`](docs/reports/complex_nl_query_analysis.md) | 22 个复杂查询语义分析，17 个语义丢失案例根因 |
| [`docs/reports/code_review_report.md`](docs/reports/code_review_report.md) | feat/agent-capabilities 分支代码审查，5 个 P0 Bug |
| [`docs/reports/nl2dsl_design_answers.md`](docs/reports/nl2dsl_design_answers.md) | 6 个核心设计问题 Q&A（意图识别、DSL 校验、权限注入等） |

### History（历史归档）

> 已完成实施的设计文档和实施计划，供追溯参考。
> 详见 [`docs/history/README.md`](docs/history/README.md)。

---

## 4. Task Routing Rules

> 接到任务时，先根据主题定位文档，再读代码。**不要直接读代码猜设计。**

| 如果任务涉及... | 先读这些文档 |
|----------------|------------|
| **DSL 生成 / 校验 / 修正** | `docs/api/20-dsl-spec.md` → `docs/business/11-dsl-validation.md` → `docs/agent/31-langgraph-workflow.md` |
| **RAG 检索 / 向量库 / 元数据同步** | `docs/agent/30-rag-design.md` → `docs/agent/32-metadata-sync.md` |
| **权限 / 安全 / 脱敏** | `docs/business/12-permission.md` → `docs/business/11-dsl-validation.md` → `docs/specs/2026-05-19-failover-system-design.md` |
| **Agent 编排 / 意图识别 / 复杂查询拆解** | `docs/agent/31-langgraph-workflow.md` → `docs/business/13-business-rules.md` → `docs/reports/complex_nl_query_analysis.md` |
| **SQL 构建 / 方言转换 / 执行优化** | `docs/architecture/03-sql-engine.md` → `docs/architecture/sql-execution-design.md` → `docs/business/10-semantic-layer.md` |
| **语义层 / 指标注册 / 枚举映射** | `docs/business/10-semantic-layer.md` → `docs/configuration/schema-reference.md` → `docs/business/13-business-rules.md` |
| **API 接口 / 错误处理 / 流式响应** | `docs/api/21-api-contract.md` → `docs/api/22-error-handling.md` |
| **部署 / 配置 / 性能调优** | `docs/architecture/04-deployment.md` → `docs/architecture/01-overview.md` |
| **测试 / 评估 / 质量报告** | `docs/agent/33-testing.md` → `docs/evaluation/framework-guide.md` → `docs/specs/evaluation-design.md` → `docs/reports/e2e_report.md` |
| **插件扩展 / 自定义节点** | `docs/history/superpowers/specs/2026-05-22-插件框架设计.md` |
| **多领域支持 / 域切换** | `docs/history/superpowers/specs/2026-05-27-multi-domain-design.md` |
| **前端 / Web UI** | `docs/history/superpowers/specs/2026-05-23-web-frontend-design.md` |
| **审计日志 / 链路追踪** | `docs/audit/audit-log-design.md` → `docs/agent/31-langgraph-workflow.md` |
| **用户反馈 / 纠错闭环** | `docs/feedback/feedback-loop-design.md` |
| **查询歧义 / 澄清机制** | `docs/query/query-clarification-design.md` |
| **查询沙箱 / 安全预检** | `docs/query/query-sandbox-design.md` |
| **查询规划 / 语义优化** | `docs/planner/query-optimization-design.md` → `docs/specs/semantic-optimizer-architecture-v2.md` → `docs/specs/semantic-optimizer-error-taxonomy-v2.md` → `docs/specs/semantic-optimizer-implementation-plan-v2.md` |
| **Semantic Query Optimizer / 规则开发** | `docs/specs/semantic-optimizer-architecture-v2.md` → `docs/specs/semantic-optimizer-error-taxonomy-v2.md` → `docs/specs/semantic-optimizer-implementation-plan-v2.md` |

---

## 5. Engineering Rules

不可违反的架构底线：

1. **Do not bypass DSL** — LLM 只生成结构化 DSL，不生成 SQL。DSL 是唯一的语义契约。
2. **Do not generate SQL directly from NL** — SQL 由系统从 DSL 构建，确保可校验、可优化、可审计。
3. **Do not invent metrics** — 所有指标必须在 `configs/metrics.yaml` 中注册，禁止自由文本指标名。
4. **Do not invent dimensions** — 所有维度必须在 `configs/metrics.yaml` 中注册，禁止自由文本字段名。
5. **Governance is authoritative** — 权限策略（`configs/permissions.yaml`）是数据源真理，不是事后补丁。
6. **All new features require evaluation** — 新增功能必须添加评测用例，确保语义准确性不下降。

---

## 6. Development Workflow

```
1. Read AGENTS.md          ← 你在这里。了解项目定位、找到相关文档主题
2. Locate relevant docs    ← 按 Task Routing Rules 定位设计文档
3. Read design documents   ← 理解设计意图和约束
4. Create implementation plan
5. Implement
6. Add tests
7. Update docs             ← 更新 docs/ 中对应文档（不是 AGENTS.md）
```

---

## 7. Anti-Patterns

❌ **NL → SQL Directly** — 跳过 DSL 直接生成 SQL，丧失校验、权限控制、可解释性  
❌ **Hardcoded Metrics** — 在代码里写死指标定义，而非从语义层注册表读取  
❌ **Hardcoded Permissions** — 在代码里写死权限规则，而非从配置文件注入  
❌ **Business Logic In API Layer** — 把业务逻辑写在 API 路由里，而非下沉到 Agent/Graph 层  
❌ **Duplicate Definitions Between Docs** — 同一份设计在多个文档中重复描述，造成维护负担  
❌ **Modifying Architecture Without Updating Docs** — 改了架构但不更新对应的设计文档

---

## 附录 A：Quick Start

```bash
# 安装依赖
pip install -e ".[dev]"
cd web && npm install

# 复制环境变量模板并编辑
cp .env.example .env

# 启动后端（首次自动初始化向量库）
uvicorn nl2dsl.api:app --reload --host 0.0.0.0 --port 8000

# 启动前端
cd web && npm run dev

# 运行测试
pytest                    # 全部测试
pytest tests/unit/        # 只跑单元测试
pytest --cov=nl2dsl --cov-report=html  # 带覆盖率
```

## 附录 B：环境变量

前缀 `NL2DSL_`：

| 变量 | 必填 | 说明 |
|------|------|------|
| `NL2DSL_LLM_API_KEY` | 否 | LLM API 密钥，不配置时用 Mock |
| `NL2DSL_LLM_BASE_URL` | 否 | API 基础 URL |
| `NL2DSL_LLM_MODEL` | 否 | 模型名，默认 `glm-4.5-air` |
| `NL2DSL_MILVUS_URI` | 否 | Milvus Lite 路径 |
| `NL2DSL_DB_URL` | 否 | 数据库连接串 |
| `NL2DSL_MAX_LIMIT` | 否 | 单次最大返回行数，默认 10000 |
| `NL2DSL_QUERY_TIMEOUT` | 否 | 查询超时（秒），默认 30 |

## 附录 C：技术决策要点（10条 ADR）

1. **LLM 只生成 DSL 不生成 SQL** — DSL 可校验、可修正、可做权限控制
2. **RAG 分 4 集合不同策略** — 短词用关键词检索，问句用语义检索
3. **启动自检同步** — 改 YAML 重启即生效，增量同步避免重复加载模型
4. **SQLAlchemy Core 而非 ORM** — 表达式树级别控制，适合动态 SQL 构建
5. **LangGraph StateGraph** — 条件分支、可观测性、检查点恢复、流式输出
6. **SQLite 默认数据库** — 零部署成本，生产环境可切换 MySQL/PostgreSQL
7. **LLM 输出三层兜底** — 正则提取 JSON → 字段补全 → 硬约束修正
8. **双层架构（Agent + Graph）** — Agent 负责宏观编排，Graph 负责微观执行
9. **配置驱动意图** — 新增意图无需改代码，修改 `configs/intents.yaml` 即可
10. **置信度评估不阻断** — syntax + semantic + history 三维度评分，低于阈值路由到澄清

## 附录 D：目录结构速查

```
nl2dsl/
├── api.py / api_factory.py   # FastAPI 入口
├── config.py                 # 配置管理
├── domain_context.py         # 领域上下文（每域独立运行时）
├── engine.py                 # 引擎入口：插件加载、StateGraph 编译
├── plugin.py / protocols.py  # 插件框架 + 组件协议
├── agent/                    # Agent 智能编排层
├── dsl/                      # DSL Schema、校验器、构建工具
├── graph/                    # LangGraph StateGraph 查询管道
├── llm/                      # Prompt 模板 + LLM 客户端
├── optimizer/                # 语义优化器：Normalizer → Rule Engine → CanonicalResolver
│   ├── base.py               #   规则基类 + 接口定义
│   ├── context.py            #   优化上下文（DSL + 语义元数据）
│   ├── engine.py             #   优化引擎入口（三层管道）
│   ├── metadata.py           #   语义元数据提供器
│   ├── normalizer.py         #   DSL 标准化器
│   ├── registry.py           #   规则注册表
│   ├── report.py             #   优化报告生成器
│   └── rules/                #   规则集（9 类 26 种错误类型）
│       ├── ambiguity.py      #     歧义检测规则
│       ├── dimension.py      #     维度修正规则
│       ├── filter.py         #     过滤条件修正规则
│       ├── governance.py     #     治理合规规则
│       ├── intent.py         #     意图修正规则
│       ├── metric.py         #     指标修正规则
│       ├── planning.py       #     规划优化规则
│       ├── structural.py     #     结构修正规则
│       └── time.py           #     时间语义修正规则
├── permission/               # 行级权限 + 列级权限 + 脱敏
├── query/                    # 歧义检测 + 查询改写 + 沙箱
├── semantic/                 # 语义层注册中心 + 解析器
├── sql_engine/               # SQLAlchemy 构建 + 扫描 + 执行
├── audit/                    # 审计日志
├── feedback/                 # 用户纠错反馈
├── planner/                  # 传统查询规划器
└── utils/                    # 日志工具

configs/                      # YAML 配置（指标/维度/意图/术语/权限）
docs/
├── specs/                    # 专项设计文档（优化器架构、评估设计等）
├── superpowers/              # Superpowers 实施计划与过程记录
└── ...                       # 架构/API/业务/Agent/审计等（见文档导航）
web/                          # React + Vite 前端
tests/                        # 单元 / 集成 / E2E
data/                         # SQLite + Milvus + 测试结果
logs/                         # 日志文件
```

---
> Source: [kobegy248/nl2dsl-engine](https://github.com/kobegy248/nl2dsl-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
