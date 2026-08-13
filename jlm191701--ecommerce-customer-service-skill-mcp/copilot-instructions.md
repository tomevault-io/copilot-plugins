## ecommerce-customer-service-skill-mcp

> 本文件适用于仓库根目录及全部子目录，供 Codex、其他编码 Agent 和开发者协作时使用。若未来某个子目录增加更具体的 `AGENTS.md`，以距离目标文件最近的规则为准。

# AGENTS.md

本文件适用于仓库根目录及全部子目录，供 Codex、其他编码 Agent 和开发者协作时使用。若未来某个子目录增加更具体的 `AGENTS.md`，以距离目标文件最近的规则为准。

## 项目定位

这是一个电商智能客服工程演示，重点验证 Agent Runtime、文件型 Skill、MCP-compatible 工具层、本地知识检索、MySQL 示例业务数据和 React 界面的解耦方式。

当前实现可用于本地开发、架构验证和离线测试，但不是生产客服系统：

- `backend/db/seed.sql` 和 Mock Gateway 都是合成演示数据，不是真实电商数据源。
- `mcp_server` 暴露的是项目自定义 HTTP facade，不应描述成已经实现完整标准 MCP 传输协议。
- 登录接口不签发会话或令牌，聊天接口也不校验登录身份；不能宣称已具备生产认证。
- MySQL 中的工单和转人工只创建演示记录，没有接入真实坐席、CRM 或工单平台。
- `stream` 字段目前被接受但没有流式返回实现。

对外说明能力时，必须区分“当前已实现”“演示/合成”“规划中”。不要把目标规格写成现状。

## 规范与事实来源

按以下顺序解决冲突：

1. 用户在当前任务中的明确要求。
2. 本文件及目标目录内更具体的 `AGENTS.md`。
3. `docs/specs/` 中标为规范性要求的文档。
4. Pydantic Schema、数据库 Schema、工具注册表和自动化测试所体现的可执行契约。
5. `docs/` 中的设计说明、`PROJECT_STRUCTURE.md` 和 `README.md`。

代码代表“当前行为”，规格可能同时定义“发布目标”。两者不一致时，不要静默选择一方：在变更中同步代码、测试和规格，或明确记录差距。

## 模块边界

| 路径 | 职责 | 不应承担的职责 |
| --- | --- | --- |
| `frontend/` | React UI、本地会话展示、调用后端 HTTP API | 直接访问数据库、直接调用 MCP、决定客服策略 |
| `backend/app/api/` | FastAPI 路由、请求/响应边界 | 沉淀 Agent 决策或跨层业务规则 |
| `backend/app/agent/` | 会话执行循环、Action、Capability、Guardrail、Trace | 直接写 SQL、绑定具体业务供应商 |
| `backend/app/infrastructure/` | LLM、视觉、记忆、知识检索、MCP、会话存储的适配实现 | 改写 Skill 的客服口径 |
| `skills/` | 意图、槽位、对话策略、工具选择和人工升级策略 | 直接访问数据库、保存密钥、绕过权限校验 |
| `backend/knowledge/` | 可追溯的静态知识卡片 | 保存用户私有数据或动态订单状态 |
| `backend/db/` | MySQL Schema 与可重复导入的合成种子数据 | 充当生产迁移历史或真实数据导出 |
| `mcp_server/` | 独立工具服务 HTTP 入口和装配 | 复制 Agent Runtime 或前端逻辑 |
| `docs/specs/` | 产品、系统、接口、数据、评测和非功能规范 | 无状态标记地描述未来能力为现状 |

依赖方向应保持为：`frontend -> backend API -> Agent Runtime -> capability/tool interface -> infrastructure`。Skill 通过 capability 请求能力；真实权限必须由服务端和工具层再次校验。

## 开发工作流

开始修改前：

1. 运行 `git status --short`，识别并保留用户已有改动。
2. 阅读目标模块、相邻测试和对应规格；不要仅凭 README 推断行为。
3. 将任务限制在最小必要范围。未经明确请求，不提交、不推送、不重写历史。
4. 不编辑 `.env`、运行日志、索引、记忆文件、`node_modules/`、`dist/`、`*.egg-info/` 等生成物。

建议从仓库根目录创建统一开发环境：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -e "./backend[dev]" -e ./mcp_server
npm ci --prefix frontend
```

后端测试：

```powershell
python -m pytest backend/tests mcp_server/tests
```

前端类型检查与生产构建：

```powershell
npm run build --prefix frontend
```

运行后端时，从 `backend/` 启动以保持默认相对路径正确：

```powershell
Set-Location backend
uvicorn app.main:app --reload --port 8001
```

独立工具服务从仓库根目录启动：

```powershell
python -m uvicorn mcp_server.app.main:app --host 127.0.0.1 --port 9001 --env-file mcp_server/.env
```

## 实现规范

### Python 与 API

- 目标运行时为 Python 3.11+，新增公共函数和协议应提供类型标注。
- HTTP 输入输出使用 Pydantic 模型；新增字段必须考虑默认值、兼容性和前端类型同步。
- 不在 `async` 路由中直接执行阻塞数据库或文件网络操作；沿用线程封装或异步客户端边界。
- 配置只通过 `backend/app/core/config.py`、`mcp_server/app/config.py` 和环境变量进入。禁止硬编码密钥、密码或环境地址。
- 对外错误应稳定、可处理且不泄露异常正文、SQL、密钥或内部路径。服务端日志可记录异常类型和 trace，但必须脱敏。
- 修改接口时同步更新 `docs/specs/interface-spec.md`、请求/响应模型、前端类型和契约测试。

### Agent、Skill 与 MCP 工具

- 通用执行机制放在 `backend/app/agent/`；客服规则优先放在 `skills/customer_service_core/`。
- Skill 应请求稳定 capability，不直接依赖数据库，也不应把工具成功结果写死在话术中。
- 新工具必须同时具备：注册 Schema、参数校验、明确权限级别、统一成功/失败结构、审计脱敏、单元测试。
- 查询用户订单、物流、支付和售后时，必须校验资源归属。不能仅信任请求体中的 `user_id`；生产接入必须使用服务端认证主体。
- `ticket.create`、`handoff.request` 等副作用能力应具备幂等策略、权限校验和可追踪 ID；在这些条件落地前只可标为演示能力。
- LLM 或视觉模型不可作为订单、价格、库存、政策有效期等动态事实的唯一来源。事实必须由受控数据源或带来源的知识检索提供。
- 工具失败、证据不足或权限冲突时安全降级：澄清、说明限制、重试或转人工，禁止编造成功。

### 数据与知识库

- 测试和示例只使用合成、匿名或明确获准的数据。禁止提交生产数据库导出、真实用户信息、聊天原文或访问令牌。
- Schema 变更必须可审查；不要用破坏性 SQL 代替迁移方案。若修改 `schema.sql` 或 `seed.sql`，应在干净数据库验证可重复初始化。
- 知识卡片必须包含可核验的来源与更新时间；来源未知的内容只能标为示例，不能包装为官方政策。
- 不在静态知识库中保存订单状态、支付结果、用户画像等动态私有信息。
- 长期记忆、审计和 Trace 都按可能含个人信息处理；新增字段前先定义最小化、脱敏、保留期和删除方式。

### 前端

- `frontend/src/api.ts` 是 HTTP 调用边界，组件不散落后端 URL 或直接构造非契约请求。
- `frontend/src/types.ts` 应与后端 Pydantic 响应一致；可选与必填字段不得凭 UI 假设。
- `localStorage` 只能保存演示所需的非敏感 UI 状态。生产模式不得在其中保存令牌、完整个人信息或敏感对话。
- 图片限制不能只依赖浏览器；服务端仍需校验 MIME、大小、数量和来源。
- 交互变更至少保证键盘可用、可辨识错误状态和合理的中文文案。

## 测试要求

测试默认离线、确定性、可重复，不调用真实 LLM、视觉、支付、物流或生产数据库。

| 变更范围 | 最低验证 |
| --- | --- |
| Agent Runtime、Skill loader、会话或记忆 | 对应单元测试，并运行 `backend/tests` |
| MCP 工具、Capability、数据库适配 | 权限成功/拒绝、缺参、上游失败和脱敏用例；运行两组 Python 测试 |
| Backend HTTP Schema/路由 | FastAPI TestClient 契约用例，覆盖 2xx、4xx 和失败结构 |
| Remote MCP facade | `mcp_server/tests` 与 `backend/tests/test_remote_mcp_gateway.py` |
| 知识卡片或检索 | 检索命中、无结果、来源字段和相关离线评测 |
| 前端代码或共享接口 | `npm run build --prefix frontend`，必要时补组件/端到端测试 |
| 数据库 Schema/Seed | 干净 MySQL 初始化、重复导入和关键查询验证；若本地无 MySQL需明确说明未验证项 |
| 仅文档 | 检查相对链接、命令、文件名和“当前/目标”状态是否准确 |

修复缺陷时先补能复现问题的测试。不要为让测试通过而弱化身份、权限、脱敏或错误处理断言。

## 安全底线

- 禁止提交 `.env`、API Key、数据库密码、Cookie、Token、私钥或真实客户数据。
- 不在错误响应、审计日志或 Trace 中记录完整密码、手机号、邮箱、地址、支付流水、图片 data URL 或大段聊天原文。
- 浏览器请求体、模型输出和远程 MCP 响应都属于不可信输入，必须校验。
- 所有用户级资源访问需要服务端认证、对象级授权和租户隔离；现有仅基于 `user_id` 的演示校验不等于生产安全。
- 支付、退款、赔付、账户修改、政策例外等高风险操作在本项目中不得自动执行；新增时必须包含显式确认与人工复核。
- 发现疑似凭据或真实个人数据时停止扩散，不在聊天或 Issue 中复述内容，并通知维护者轮换/删除。

## Definition of Done

一项变更只有在以下条件满足时才算完成：

- 行为满足任务要求，模块边界未被破坏。
- 新增或修改行为有相称的自动化测试，相关测试和构建通过。
- 接口、配置、数据语义或用户流程发生变化时，规格和示例同步更新。
- 安全失败路径、权限校验、隐私最小化和日志脱敏经过检查。
- 没有提交密钥、真实数据或生成物；`git status --short` 中只包含预期文件。
- 最终交付明确列出已验证内容、未验证内容和仍存在的生产差距，不夸大能力。

---
> Source: [jlm191701/ecommerce-customer-service-skill-mcp](https://github.com/jlm191701/ecommerce-customer-service-skill-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
