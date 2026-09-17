## factory-copilot

> 本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 语言规范（强制）

- **一律用中文**：包括回答、步骤说明、内部思考/推理过程（thinking）、工具调用描述、todo 列表、目标/子任务描述，不得切换到英文（代码、技术术语如 Agent/SSE/LLM/Neo4j/Cypher/API 保留原名除外）。
- **思考过程（thinking）也必须用中文**，这是强制项，不是可选项；哪怕只是内部推理，也禁止整段英文。
- 生成/修改的代码注释、docstring、日志文案、前端提示文案全部使用中文。

## 项目概述

**Factory Copilot**（制造业 MES AI 智能体）是基于 FastAPI + React 构建的制造业 AI 助手。本体（Ontology）由 **OntoStudio**（另一仓库 `Ontology-Graph/`）建模并推送到 Neo4j，FC 动态编译本体生成 Agent 和工具，支持多业务域协作、长期记忆向量检索、SSE 流式响应。

**数据流**：OntoStudio（本体建模）→ Neo4j（图数据库）→ FC（编译 Agent）→ 用户对话。FC 只读 Neo4j，本体以 OntoStudio 为唯一数据源。

## 快速开始

### 后端（端口 9004）

```bash
cd backend
# 创建虚拟环境（首次）
python -m venv venv
venv\Scripts\activate          # Windows
pip install -r requirements.txt

# 启动开发服务（端口从 backend/.env 的 API_PORT 读取，9004）
uvicorn app.main:app --port 9004
```

Windows 一键启动：`start.bat`（后端 9004 + 前端 5004）

### 前端（端口 5004）

```bash
cd frontend
npm install
npm run dev                    # 开发服务 5004 端口，/api 代理到 :9004
npm run build                  # 生产构建（独立部署）→ dist/
npm run build:subapp           # 生产构建（子应用部署 /AI-OS/）→ dist/
```

### 测试与代码质量

```bash
cd backend
# 测试
pytest tests/ -v --tb=short

# 代码质量
ruff check app/                         # Lint
mypy app/ --explicit-package-bases      # 类型检查
pip install -r requirements-dev.txt     # 安装开发工具
```

### Docker

```bash
docker build -t factory-copilot .
docker run -p 9004:9004 --env-file backend/.env factory-copilot
```

### 关键文件

| 文件                     | 说明                    |
| ---------------------- | --------------------- |
| `backend/.env`         | 当前配置（API Key、模型选择、端口、Neo4j） |
| `backend/.env.example` | 配置模板                  |
| `start.bat` / `stop.bat` | Windows 启动/停止脚本      |
| `sync.sh` / `sync-backend.sh` | 同步服务器（前端/后端）       |
| `frontend/.env.production` | 独立部署配置（VITE_ENV_SITE=main） |
| `frontend/.env.subapp`     | 子应用部署配置（VITE_ENV_SITE=main-sub, /AI-OS/） |

## 架构

### 后端 (`backend/app/`)

**入口**：`main.py` → `create_app()` 注册 20+ 路由组（health/chat/messages/conversations/notifications/memory/approval/explorer/alerts/mcp/a2a/vectorization 等）。

**核心流式路径**：`/api/messages/stream` — `MessageService` 编排：记忆检索 → 历史加载 → Agent 路由 → 执行 → LLM 格式化 → 持久化。

```
backend/app/
├── api/                  # FastAPI 路由处理
├── core/
│   ├── config.py         # Pydantic 配置
│   ├── model_config.py   # LLM 供应商配置（通义千问/DashScope/DeepSeek）
│   ├── chain_engine.py   # 分析链引擎（多步推理链执行）
│   └── prompts.py        # 系统提示词
├── models/               # SQLAlchemy ORM 模型
├── repositories/         # CRUD 层
├── services/
│   ├── llm_service.py           # LLM 流式调用
│   ├── message_service.py       # 核心编排
│   ├── action_executor.py       # 本体动作执行（查询/写入/删除，DataBackend）
│   ├── intent_router.py         # 意图路由（L2 LLM 语义分类）+ 参数提取
│   ├── rule_engine.py           # 规则引擎（约束/推理/触发器/计算字段）
│   ├── data_backend.py          # DataBackend 抽象（Neo4j/API/SQLite 降级链）
│   ├── ontology_service.py      # 从 Neo4j 加载本体（Concept/Action/Property/Relation）
│   ├── multi_system_backend.py  # 多系统后端（按概念路由到 MES API）
│   ├── event_bus.py             # 内存 SSE 广播（审批状态实时推送）
│   └── vector_memory_service.py # SQLite 向量存储
├── agents/
│   ├── __init__.py       # compile_and_register()：编译本体生成 Agent
│   ├── compiler/         # OntologyCompiler：本体 → Skill → Agent（含 DynamicPlanner）
│   ├── agent_config.py   # AGENT_DEFINITIONS 缓存（对话路由用）
│   ├── base.py           # BaseAgent 抽象类 + _standard_process 标准流程
│   └── router.py         # route_intent：LLM 语义路由到 Agent
└── tools/
```

### Agent 系统（编译器驱动）

**不再是固定角色化 Agent**。`compile_and_register()` 从 Neo4j 本体动态编译：

1. **OntologyCompiler** (`compiler/compile.py`)：读本体概念 → 每个概念生成 `{Concept}_query` Skill + 关联 actions → 按业务域配置分组 → Agent
2. **业务域配置**（`/chains/compile/namespace/{name}`）：DB 存储 domains（`{agent: {display_name, concepts[]}}`），决定哪些 Skill 进哪个 Agent
3. **切换本体图谱**：前端"业务配置"页下拉切换 namespace → 只编译预览（不生效）→ 点"全部应用"才刷新 `AGENT_DEFINITIONS` 和意图路由，对话切换到新业务域

**切换/应用机制**：
- `switch_namespace`：`compile_and_register(sync_to_db=False)` — 只编译预览，不写 DB 不刷新路由
- `compile_reload`（全部应用）：`sync_to_db=True` + `reload_agents()` — 写 DB + 刷新 AGENT_DEFINITIONS，对话生效

### 意图路由 (`intent_router.py`)

- **L2 (llm_classify) 主路由**：候选 action 按 concept_label 分组，LLM 语义匹配。约束输出防幻觉。
- **L1 (keyword) 已废弃**作主路由（中文口语误判率高），仅辅助保留。
- **参数提取**：正则 pattern 从消息提取（不由 LLM 生成），支持模糊搜索（`_fuzzy` + `_fuzzy_op`，中文/编码/数字）。

### 模糊搜索（企业级多字段 OR + 命中分级）

用户查询"38开头的工单"：
1. `extract_params` 识别"XX开头/包含XX"句式 → `_fuzzy='38'` + `_fuzzy_op='prefix'`
2. `_query_via_backend` 用 action schema 构建 `_fuzzy_fields`（可搜索字段）
3. `Neo4jBackend.query` 多字段 OR（`STARTS WITH`/`CONTAINS`）+ 精确>前缀>包含排序

前置条件：OntoStudio 里给概念 query action 补全可查询参数（如 WorkOrder: code/materialCode/materialName 等）。

### 数据后端抽象 (`data_backend.py`)

**DataBackend** 是业务数据访问的统一接口（`resolve_entity()`/`query()`/`create()`/`delete()`）。三实现 + 降级链：

- **Neo4jBackend** — Cypher 查询，支持图遍历
- **ApiBackend** — HTTP REST 调用 MES 系统
- **SqliteBackend** — SQL 查询 mes_demo.db
- **FallbackDataBackend** — 按优先级 Neo4j → API → SQLite 自动降级

**硬边界**：所有业务数据访问必须通过 DataBackend 接口，不能直接调 SQLite/Neo4j。

### 规则引擎 (`rule_engine.py`)

按 ruleType 分发到评估器：`constraint` / `inference` / `trigger` / `computed`。

- `evaluate_all(concept, params, action)` — 约束校验 + 推理 + 审批门禁
- `computed` 规则（计算字段）不在规则引擎执行，由 DB Sync Phase 4 动态生成 Cypher
- 规则包校验（`_validate_bundle`）跳过 computed 规则（表达式是 Cypher 非条件式）

### 记忆系统（三层）

1. **短期记忆**：从 DB 加载最近 `MAX_HISTORY_LENGTH`（50）条消息
2. **摘要压缩**：超 50 条时 LLM 压缩为摘要，缓存 `conversations.summary`
3. **长期记忆**：SQLite 向量存储，余弦相似度，0.95 去重阈值

### 前端 (`frontend/`)

**技术栈**：React 18 + Ant Design 5 + Vite 5 + Context + ECharts + Mermaid

**多环境构建**：`VITE_ENV_SITE`（main 独立 / main-sub 子应用）+ `VITE_BASE`（资源路径）。

**布局**：`App.jsx` 根据 `VITE_ENV_SITE` 切换——main 模式有侧栏 + header，main-sub 模式（Wujie 嵌入）无侧栏无 header。

**核心组件**：

- `ChatInterface.jsx` — SSE 流式渲染 + 执行链路步骤（意图识别/匹配工具/参数提取/确认/执行/结果）
- `ChainManager/` — 业务配置：业务域配置 Tab（含"本体图谱"切换下拉 + 全部应用）、链条配置、API 接口、向量化
- `AgentSidebar/` — Agent 列表
- `ConversationDrawer/` — 历史会话
- `NotificationList.jsx` / `PendingApprovalView.jsx` — 通知/审批
- `MarkdownRenderer.jsx` — 流式/完成模式 Markdown（ECharts/Mermaid 懒加载）

**Services**：

- `services/request.js` — Axios，鉴权拦截器，30s 超时
- `services/messageService.js` — 主流式路径（fetch + ReadableStream）
- `services/sse.js` — 全局 SSE 单例（onmessage 解析 `__type`，配合后端 event_bus 双通道）

**Vite 代理**：`/api` → `http://127.0.0.1:9004`。

## SSE 协议

流式响应使用 Server-Sent Events，带 `event` 字段：

```
event: agent_info      data: {"agent_name": "contract_legal", "confidence": 0.95}
event: thinking        data: 推理过程文本
event: content         data: 响应文本片段
event: route_l2        data: {"candidateCount": 5, "concepts": ["工单","物料"]}
event: route_match     data: {"method": "llm_classify", "tool": "Contract_query", "confidence": 0.95}
event: param_extract   data: {"params": {"_fuzzy":"示例公司","_fuzzy_op":"contains"}, "tool": "Contract_query"}
event: confirm_required data: {"tool": "...", "params": {...}, "param_schema": [...]}
event: tool_start      data: {"tool": "Contract_query", "params": {...}}
event: tool_result     data: {"tool": "Contract_query", "rowCount": 3, "source": "neo4j"}
event: format_start    data: {}
event: execution_done  data: {"method": "llm_classify", "tool": "Contract_query"}
```

**双通道广播**（`event_bus.py`）：每个事件同时发命名事件（`event: xxx`）+ 默认消息（`data: {"__type": ...}`），前端 `onmessage` 解析 `__type` 分发。审批通过 `approval_done` 实时推送到对话页。

**执行链路**（前端展示）：意图识别 → 匹配工具 → 参数提取 → 数据过滤 → 人机确认（写操作）→ 执行 → 查询结果 → LLM 格式化 → 执行完成。

### 写操作确认流程

`requiresConfirmation` 的 Action 执行前通过 `confirm_required` 推参数 schema + 预填值。前端 ConfirmCard 渲染表单，审批后 `confirm_result` 回传。规则门禁（如"数量审批" quantity>200）触发审批委托。

**参数预填层级**：L1 规则提取 → L2 实体解析 → L3 LLM 回退 → L4 本体图遍历 enrich。

### 审批通过通知

审批通过后写入对话消息，含：审批人、操作参数（中文标签）、执行结果（成功/影响行数）、结果表格（创建后回读完整节点）。

## 工作准则（不迎合）

- **独立专业判断**：用户提出观点或质疑时，基于事实与工业实践给出独立判断，不因用户态度摇摆。明确「应该/可以/不建议」并有立场。
- **坚持治理原则**：责任分离（执行与复核分离）、确定性优先（能程序判定不用 LLM）、可追溯（审计）。用户观点与原则冲突时说明取舍，不盲目服从。
- **诚实报告**：对测试覆盖、局限、未验证项如实说明，不夸大、不掩盖。
- **批评是审视**：用户的批评/质疑是对工作的审视，回应要专业、直接，不防御性辩解，也不谄媚附和。
- **验收看目的，不只看跑通**：「验证通过」的判据是功能是否达成目的，不是「接口 200 / 页面能渲染 / build 通过」。可观测性、统计、监控类功能必须用真实场景跑一遍，确认输出的数据有意义、能指导决策（例如 tracing 的 span 要覆盖主要耗时环节、能定位耗时大头），而不是只确认「能渲染出图」。

## 注意事项

- **DataBackend 抽象是硬边界**：所有业务数据访问必须通过 DataBackend 接口
- **注释/docstring/日志约定**: 全部使用中文，技术术语（Agent、SSE、LLM、Neo4j、Cypher、API）保留原名
- **Windows 启动**: 不加 `--reload`（僵尸进程 bug），端口用 `.env`（9004），前端 5004
- **禁止 git checkout/restore 恢复文件**: 会不可逆丢弃会话中所有修改，操作出错时手动修复
- **禁止私自同步服务器**: 任何修改完成后，先在本地构建验证（9004 后端 + 5004 前端），然后告知用户"本地验证通过，要同步吗？"，**必须等用户明确确认后才执行 `sync.sh`**。绝不改完就自动同步——未验证的代码上线浪费资源且有风险
- **禁止动服务器数据（铁律）**: 同步服务器**只执行同步脚本**（`sync.sh` / `sync-backend.sh`），脚本只动代码，不碰数据。**绝对禁止**：scp 覆盖服务器 `.onto.yaml` / 数据文件、在服务器执行 Neo4j 推送或 DB 修改。服务器 `.onto.yaml` 是别人在用的生产数据，本地文件可能只是下载测试的副本，两者不得等同。服务器数据操作由用户自己执行，我只提供本地文件/差异说明。不确定服务器与本地关系时先问用户，绝不假设
- **`chatService.js` 是旧代码**: 主流式路径是 `messageService.sendMessageStream()`
- **切换本体后查询走智能分析是正常的**: 未点"全部应用"前 intent_router 还是旧工具索引，匹配不到新工具 → 回退 DynamicPlanner；应用后才走工具直查

---
> Source: [nium119/factory-copilot](https://github.com/nium119/factory-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
