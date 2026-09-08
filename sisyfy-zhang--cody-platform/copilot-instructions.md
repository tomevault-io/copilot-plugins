## cody-platform

> > 本文件是仓库的常驻说明：回答"这个项目是什么、怎么跑"；企业规范/知识库文档由 RAG

# cody-platform 项目规则

> 本文件是仓库的常驻说明：回答"这个项目是什么、怎么跑"；企业规范/知识库文档由 RAG
> 负责，本文件不覆盖。
>
> 每条规则块是一个 `## ` 标题，标题下一行紧跟 `<!-- tags: ... -->` 声明标签（可多个，
> 逗号分隔）。加载逻辑按 intent 分类结果做标签匹配：`code_qa` 只加载 `overview`/
> `coding` 标签的块，`knowledge_qa` 不加载本文件。新增规则块时记得打标签，没打标签
> 的块不会被任何 intent 匹配到。

## 项目概览
<!-- tags: overview -->

cody-platform（CLI 命令 `cody`）是一个 AI Code Agent 平台，用 LangGraph 显式状态机
（不是黑盒 Agent 循环）实现。目录结构：

- `app/graph.py` —— LangGraph `StateGraph` 组装入口。
- `app/state.py` —— `AgentState`（Pydantic 模型，图节点间传递的唯一状态载体）。
- `app/nodes/` —— 状态机节点实现（意图分类、上下文检索、计划生成、人工审批、权限
  检查、工具执行、沙箱验证、测试与审查等）。
- `app/subgraphs/` —— 可被父图动态委派的子图（当前只有 PLANNER 一个成员；
  Test&Review 不含 `interrupt()`，用普通节点函数实现，不需要子图机制）。
- `app/tools/` —— 内建工具工厂（`read_file`/`search_file`/`list_files`/`rag_search`/
  `write_file`/`edit_file`/`run_command`/`run_git`）+ `gateway.py`（Provider 统一
  注册层，内建与外接 MCP 工具都从这里发现）+ `mcp_client.py`（外部 MCP server 连接
  管理，stdio 子进程）+ `subtask.py`/`subtask_roles.py`/`subtask_reports.py`
  （`dispatch_subtask` 只读子任务工具，三个角色 general/code_explorer/
  dependency_analyzer 各自的只读工具白名单/轮次/预算/输出契约）。
- `app/permission/` —— `policy.py`（四元组 Subject+Action+Resource+Context 策略
  引擎，`evaluate()` 是全平台唯一的权限裁决入口）+ 敏感路径 deny 名单 +
  `command_rules.py`（`run_command`/`run_git` 的命令语义规则引擎，并入 `policy.py`
  内置层，不是第二套引擎）。
- `app/sandbox/` —— `executor.py`（`SandboxSpec`/`SandboxResult`/
  `SubprocessExecutor`/`DockerExecutor`/`get_executor`）+ `worktree.py`（任务
  worktree 生命周期管理，事务性写入的落点）。
- `app/auth/` —— 口令哈希/token/登录编排。`app/admin.py`（独立于 `app.cli`）是
  `python -m app.admin users add/list/disable/enable` 用户管理命令。
- `app/guardrails/` —— 成本护栏（单 task token 上限、连续失败熔断、外部 MCP 调用
  限流）。
- `app/context/cache.py` —— `VersionedCache`，精确版本匹配，接入 AGENTS.md 解析/
  project-map 扫描/RAG 检索三处。
- `app/rag/` —— 文档解析、按标题分块、向量检索 + 关键词兜底。
- `app/memory/` —— 会话记忆与 Checkpoint 恢复前的工作区漂移检测。
- `app/kg/` —— 代码知识图谱（Neo4j 后端），`graph_query` 工具与 `code_qa` 检索融合
  的数据层；Neo4j 是可选服务，未配置时整个模块降级隐身。
- `app/model/` —— 模型接入：云端 OpenAI 兼容接口客户端 + 本地小模型意图路由（本地
  不可用时自动降级云端）。
- `app/lsp/` —— LSP client 单例治理，供 TUI 的跳转到定义功能使用；`get_lsp_client`/
  `reset_lsp_client` 全程 `async def`，必须在调用方自己的事件循环里 `await`。
- `app/review/` —— Review Agent 核心分析能力：diff 静态审查 + 反馈驱动的规则蒸馏
  （三层：单条反馈记录、批量蒸馏、冲突留人工裁决）。
- `app/skill/` —— Skill 机制：coding 专用的可复用任务流程手册。`schema.py`（YAML
  frontmatter + markdown 正文解析/校验）、`registry.py`（两层目录扫描 + 缓存 +
  落盘）、`retrieval.py`（两阶段检索）、`tool.py`（`consult_skill` 工具）。
- `app/project/` —— 入口层"本机最近打开过哪些项目、每个项目的本机 workdir 在哪"的
  注册表，纯本机、不进数据库、不跨机器同步。
- `app/workspace/` —— 入口层"资源隔离粒度"：每个 workspace 对应一个独立的 Neo4j
  进程，一个 workspace 内可以容纳多个 project 并自由 `/switch`，不同 workspace
  之间物理隔离（各自独立容器/端口/数据卷）。
- `app/tui/` —— Textual 全屏交互界面：消费 `graph.stream()` 事件、驱动计划/工具
  审批弹窗，是当前唯一的交互式入口。
- `app/ui/` —— 渲染层纯函数（面板、表格、Diff 等）与 slash 命令解析，被 `app/tui/`
  复用。
- `app/rendering/` —— Headless Runner（`-p`/`--print` 一次性模式）专用的事件渲染
  层：`EventRenderer` Protocol（`render(event)`/`finish() -> bool`），只消费
  `graph.stream()` 事件，不碰核心状态机，不依赖任何终端富渲染库。
- `app/config/agents_md.py` —— 本文件的加载器。
- `app/db/` —— SQLAlchemy 模型与数据库访问；`migrate.py` 含幂等 `ALTER TABLE` 补列
  + bootstrap admin 引导。
- `app/security/` —— 敏感值脱敏。
- `tests/` —— 测试；`tests/eval/` 是集成任务评估集雏形。

已实现能力：只读代码问答（受限文件工具 + AGENTS.md 按需加载）、企业知识库 RAG 问答
（pgvector 向量检索 + 关键词兜底 + 护栏）、会话记忆与工作区漂移检测、Plan Mode
（结构化计划 + 人工审批 + Diff 展示）、MCP 工具集成与四元组权限审批引擎、会话模式
default/plan/auto 与项目级治理上限、Sandbox 事务性写入（worktree 隔离 + 验证命令 +
全部成功才应用回真实工作区）、代码知识图谱检索融合（Neo4j，可选服务）、本地小模型
意图路由、Supervisor 动态调度拓扑（含委派 PLANNER 子图）、Skill 机制（检索 + 用户
钉住强制召回）、Review Agent 与反馈驱动的规则蒸馏、GitHub 平台集成（自我修复 draft
PR、PR 评论摘要）、TUI 全屏交互界面与 Headless（`-p`）一次性模式。

## 代码约定
<!-- tags: coding -->

- `AgentState`（`app/state.py`）必须用 Pydantic `BaseModel`，不能用
  `TypedDict`——LangGraph 大版本升级时 `TypedDict` 状态会导致 checkpoint 反序列化
  报错。
- `AgentState` 只能加字段，不能删除或重命名已有字段（只增量演进）。新字段找不到
  合适的默认值时，宁可加一个可选字段恒为 `None`/空容器，也不要改动已有字段的类型
  或语义。
- 权限判断（敏感路径 deny 名单等）永远不能写死在工具函数内部，必须走独立的判断层
  （`app/permission/deny_list.py::check_path_deny`）。工具执行的唯一入口是
  `app/tools/dispatch.py::dispatch_tool_call`，不要绕开它直接调用工具。
- 参数校验（这个请求本身合不合法）和权限判断（允不允许执行）是两个独立串联的检查
  环节，参数校验不过不会进入权限判断。
- 给 `app/db/models.py` 里**已经存在**的表加列（不是新建表）时，光改 `Mapped`
  声明不够——`Base.metadata.create_all()` 只建"元数据里有但库里没有的表"，对已存在
  表的新增列**静默跳过**。必须同步在 `app/db/migrate.py` 里补一段幂等的
  `ALTER TABLE ... ADD COLUMN IF NOT EXISTS`（Postgres 原生语法，可重复执行），
  否则新代码在老库上运行时会在查询期直接报 `UndefinedColumn`/`UndefinedTable`。
  新建表不受此影响，`create_all` 正常处理。
- LangGraph 对 `dict[str, Any]` 类型的 `AgentState` 字段（`permission_context`/
  `context_flags`/`session_memory` 等）是**整体替换**语义，不是深合并——一个节点
  的返回 dict 里如果重新构造 `{"denied": ..., "details": ...}` 会把更早节点写入的
  其他键（比如 `approved_files`）整个冲掉。任何要往这类字段"追加"信息的节点，
  必须基于 `state.xxx` 现有值做增量更新（`{**state.xxx, "new_key": ...}`），不能
  凭空新建一个只含自己关心字段的 dict。
- `AgentState` 里 `Optional[...] = None` 的字段（如 `plan`/`approval_status`/
  `diff_result`/`error`），如果某次图执行路径上从来没有任何节点显式写过非 `None`
  值，`graph.invoke()`/`.stream()` 的结果 dict 里**这个 key 会直接不存在**（不是
  `None`）。读取这类字段一律用 `.get(key)`，不要用 `result[key]`。
- 权限判断唯一入口是 `app/permission/policy.py::evaluate`（`app/nodes/
  permission_check.py` 图内正式裁决点、`app/tools/dispatch.py` 防御性兜底都调用
  它）——给 `dispatch_tool_call` 新增调用点/新增参数时，必须检查所有既有调用点
  （`retrieve_context.py`/`plan.py`/`tool_execution.py` 等）是否都传全了新参数。
  策略引擎参数漏传的后果是：`HUMAN_TOOL_APPROVAL` 批准的调用在 `TOOL_EXECUTION`
  兜底复核时又被裁成 `needs_approval`——人工批准形同虚设，且不会报错，只是安静地
  卡住不执行，很难从日志直接看出来是哪个调用点漏传了参数。
- **worktree 清理纪律**：每个验证脚本/会话结束后必须 `git worktree remove` +
  `git worktree prune`；`app/cli.py` 启动时要清理本工具残留的过期（>24h）worktree。
  验证脚本泄漏一个 worktree 就会污染用户仓库的 `git worktree list` 输出，算作
  事故，不是无害残留。
- **subprocess 纪律**：沙箱执行（`SubprocessExecutor`/`run_command`/`run_git`）
  一律 `shell=False` + 显式解析后的 argv 列表，不传原始字符串给 shell；env 只注入
  显式声明的键值对，绝不继承 Cody 进程完整环境（跟 MCP 子进程 env 隔离同一哲学——
  被测代码是用户仓库里的任意代码，信任等级等同第三方，不能把 `CODY_MODEL_API_KEY`/
  DB 连接串暴露给它）。
- **能力边界诚实纪律**：`SubprocessExecutor` 在 Windows 上**没有** CPU/内存硬限制
  （没有 Linux cgroups），只有超时和输出长度上限两项软约束——代码注释、
  `SandboxResult` 字段、面向用户的报告都不能声称有资源硬限制。CPU/内存限制只在
  `DockerExecutor` 真实存在。
- **git 参数注入防护纪律**：`run_git` 必须剥离/拒绝用户输入里的全局参数
  （`-C`/`--git-dir`/`--work-tree`/`-c`/`--exec-path`），工作目录只能由 Cody 自己
  注入——否则三级权限分类可以被 `-C /other/repo` 之类的参数绕过。
- **工具元数据双表登记纪律**：`app/tools/gateway.py` 的 `_BUILTIN_RISK`（风险档位）
  和 `_BUILTIN_CAPABILITY`（副作用/并发能力）是两张独立维度的表，新增/改动内建
  工具时必须同步登记两处——`_BUILTIN_RISK` 里的工具名缺少 `_BUILTIN_CAPABILITY`
  条目会在模块导入期直接报错，不会留到运行时才发现。
- **上下文压缩纪律**：`app/context/compaction.py` 对 Supervisor 连续上下文的压缩
  是机械的（不调模型、按字段拼结构化索引行），绝不删除消息、绝不破坏
  `AIMessage(tool_calls)` 与 `ToolMessage` 的配对结构——只允许原地替换
  `ToolMessage.content`。Supervisor 连续上下文治理一共三层，各自独立生效：
    1. `app/context/prompt_view.py`——工具结果的"模型视图"：把
       `ToolCallRecord.tool_output`（原始记录，UI/审计权威数据源，不受影响）
       按工具类型分派成喂给模型的精简文本，纯函数、不改变原始记录。
    2. `app/context/read_dedup.py`——同版本 `read_file` 重复读取去重：去重键
       为规范化绝对路径+内容版本 hash+行范围，命中同键时只给短引用不重复
       贴正文；`write_file`/`edit_file` 成功后精确失效对应路径的索引，
       `run_command`/`run_git` 成功后保守清空整个索引（可能改动索引记录
       路径之外的任意文件）。
    3. `app/context/compaction.py`——超预算时的结构化索引压缩（本条开头所述），
       以及 `is_context_overflow_error`/`_invoke_with_overflow_retry`：真正
       撞到模型端"上下文超长"报错时，带更严格的 budget/recent_full 参数
       单次重试一次，不循环重试。

## 测试约定
<!-- tags: test -->

- 跑全部测试：`uv run pytest -q`。
- 涉及真实 LLM 调用的节点（`app/nodes/retrieve_context.py`、`app/nodes/summary.py`
  等）在测试里要 monkeypatch 模块级的 `get_chat_model`（`from app.model.client
  import get_chat_model` 这个 import 点就是特意留的测试打桩位置），不要在自动化
  测试里真的打网络请求调用模型 API。
- 权限相关测试直接构造带 `pending_tool_calls` 的 `AgentState`，不依赖真实工具执行
  也能验证 deny 逻辑，见 `tests/test_permission.py`。
- code_modify 全链路端到端测试见 `tests/test_graph_code_modify.py`（全部用假模型，
  LangGraph 的 `interrupt()`/`Command(resume=...)` 是真实机制，不打桩；覆盖 ask 态
  工具审批、auto/plan 会话模式等场景）。要在真实 CLI 里手动验证审批流程，
  `run_mode` 默认是 `readonly`（DB 未配置时的降级默认值），需要先给目标
  `project_id` 在 `cody_project_config` 表里写一行 `run_mode='plan_only'` 或
  `'approval_write'`（没有专门的 CLI flag，走的是既有的 DB 配置通道，跟
  `allowed_tools`/`enabled` 的配置方式一致）；`approval_write` 项目内可以再用
  `/mode auto`/`/mode plan` 切换会话模式验证三态行为，不需要改 DB。

## 安全与权限
<!-- tags: security, coding -->

- 敏感路径 deny 名单：`.env`、`.env.*`、`secrets/**`、`.ssh/**`、
  `config/prod.yml`。命中后不可重试、不可换路径绕过，必须记录安全事件。
- 所有路径参数在匹配 deny 名单前必须先规范化（展开所有 `../`），防止
  `../../secrets/key` 这类路径穿越绕过检查，实现见
  `app/permission/deny_list.py::safe_resolve`。
- 校验失败时只能说"路径不合法"，不能暴露白名单/黑名单的具体结构。

## 启动与构建
<!-- tags: build -->

本地开发环境搭建、启动、测试的完整命令（与 `README.md` 保持一致）：

```bash
# 1. 启动本地 Postgres（pgvector 镜像，docker-compose.yml 管理）
docker compose up -d

# 2. 安装依赖、建虚拟环境
uv sync

# 3. 配置环境变量（模型 API Key 等）
cp .env.example .env

# 4. 建表
uv run python -m app.db.migrate

# 5. 跑测试
uv run pytest -q

# 6. 启动交互式 CLI
uv run python -m app.cli chat
```

`app.cli` 顶层注册了 `chat`/`project`/`workspace` 三个命令（后两个是子命令组）。
不带子命令直接跑 `cody`（可选 `-d`/`--dir` 快速指定项目根目录）等价于最简参数的
`chat`；需要 `--project-id`/`-p`/`--verbose` 等完整参数时用 `cody chat ...`。首次
在某个目录下启动、且未显式传 `--project-id`/`-d` 时，会走交互式入口选择器确认
项目归属的 workspace。

Checkpointer 使用 LangGraph `PostgresSaver`，复用 `CODY_DATABASE_URL`
（`app/db/session.py::get_checkpointer_conn_string`）——没有独立的
`--checkpoint-path` 参数，`CODY_CHECKPOINT_PATH` 环境变量已废弃、不再生效。

用户管理是**独立**的 `python -m app.admin users add/list/disable/enable`（多命令
Typer app，需要 admin 身份登录）——绝不要给 `app.cli` 加子命令，会破坏它"不带子
命令名直接调用"的既有行为。

---
> Source: [sisyfy-Zhang/cody-platform](https://github.com/sisyfy-Zhang/cody-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
