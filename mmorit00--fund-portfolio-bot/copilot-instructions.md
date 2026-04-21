## fund-portfolio-bot

> > 你是本项目的 Python 开发助手，只在**开发阶段**使用，不参与线上运行。人类始终是总指挥。（始终使用简体中文回答）

# Portfolio Engine · Claude Code 协作说明

> 你是本项目的 Python 开发助手，只在**开发阶段**使用，不参与线上运行。人类始终是总指挥。（始终使用简体中文回答）

## 项目概述

本仓库是一个个人基金投资管理工具，目标是聚合公募基金持仓、管理定投与 T+1/T+2 确认、计算资产配置与权重、生成文本日报并推送。

> **当前版本**：v0.4.2+（业务闭环 100% 完整，生产可用）
> **Schema 管理**：开发阶段，由 `db_helper.py` 创建（SCHEMA_VERSION = 8），可随时重建，详见 `docs/sql-schema.md`
> **版本规划**：见 `docs/roadmap.md`，禁止实现任何 AI 产品功能，仅做数据准备
> 不允许你自动 git 
### 算法 vs AI 分工（核心设计原则）

本项目是 **AI 驱动** 的投资管理工具，核心设计原则是 **算法处理事实，AI 处理语义**：

```
┌─────────────────────────────────────────────────────────────┐
│                        分工边界                              │
├──────────────────────┬──────────────────────────────────────┤
│       规则/算法       │                 AI                   │
├──────────────────────┼──────────────────────────────────────┤
│ • 日期匹配计算        │ • 判断是否归属定投                    │
│ • 金额偏差率          │ • 解释偏差原因（限额/调整）            │
│ • 限购公告查询        │ • 综合多因素做语义判断                 │
│ • 交易模式统计        │ • 向用户提问澄清                      │
├──────────────────────┼──────────────────────────────────────┤
│     可计算/可验证      │         需要理解/判断                 │
└──────────────────────┴──────────────────────────────────────┘
```

**交互流程**：规则层输出事实 → AI 层做语义判断 → 用户确认修改

**核心约束**：规则只输出**事实**，AI 只做**解释**，修改需**用户确认**

> 详细说明见 `docs/architecture.md` 的"算法 vs AI 分工"节

## 4. 任务分派与工具选择

根据任务复杂度选择合适的工具：

### 优先级规则：gemini-agent 子代理 > Explore 子代理 > 直接工具调用

1) **复杂任务 → 使用 gemini-agent 子代理**
   - **调用方式**：`Task(subagent_type="gemini-agent", prompt="...")`
   - **适用场景**：
     - 深度代码分析（算法探索、性能瓶颈分析）
     - 架构设计讨论与方案对比
     - 需要推理与多轮对话的研究性任务
     - 需要理解业务逻辑并提供优化建议

2) **中等任务 → 使用 Explore 子代理**
   - **调用方式**：`Task(subagent_type="Explore", prompt="...")`
   - **适用场景**：
     - 快速定位文件位置
     - 简单的代码结构梳理
     - 关键字搜索与代码导航

3) **简单任务 → 直接工具**
   - **直接使用**：`Read`、`Edit`、`Write`、`Bash`、`Grep`、`Glob` 等
   - **适用场景**：
     - 已知路径的文件读取/编辑
     - 明确的 git 操作
     - 单一命令执行

**⚠️ 注意**：不要混淆 Task tool 的子代理和 MCP 工具
- ✅ 正确：`Task(subagent_type="gemini-agent")` - 调用专用子代理
- ❌ 错误：`mcp__pal__clink(cli_name="gemini")` - 这是底层 MCP 工具，不应直接使用

**示例**：
- ❌ 错误：用 Explore 探索算法优化空间 → 太复杂，应该用 gemini-agent
- ✅ 正确：用 gemini-agent 深度分析性能瓶颈并讨论优化方案
- ✅ 正确：用 Explore 快速找到某个功能相关的文件

---

### 环境说明

**⚠️ 重要**：本项目使用 **uv** 管理 Python 依赖。

所有 CLI 命令必须使用 `uv run` 前缀：

```bash
# ✅ 正确
uv run python -m src.cli.fund list
uv run python -m src.cli.history_import --csv data/alipay.csv --mode apply

# ❌ 错误（会报 ModuleNotFoundError）
python -m src.cli.fund list
```

**原因**：uv 使用独立虚拟环境（`.venv/`），直接运行 `python` 无法访问依赖。

**测试命令时请务必加上 `uv run` 前缀。**

---

## 1. 目录 & 文档导航

```
src/
  cli/           # 命令行入口（原 jobs/）
  flows/         # 业务流程（原 usecases/）
  core/          # 核心逻辑
    ├─ models/   #   领域数据类（Trade, Fund 等）
    └─ rules/    #   纯业务规则（settlement, rebalance 等）
  data/          # 数据访问层（原 adapters/）
    ├─ db/       #   数据库 Repo
    └─ client/   #   外部客户端（HTTP、Discord 等）
docs/
  architecture.md      # 架构与分层（含 ASCII 图）
  settlement-rules.md  # 业务规则权威
  operations-log.md    # 运维操作手册
  coding-log.md        # 开发变更记录
  roadmap.md           # 版本规划
  sql-schema.md        # Schema 说明
```

**开始任务前必读**：`architecture.md` / `roadmap.md` / 最近的 `coding-log.md`
**编码规范**：在本文件第 3 节 + `.claude/skills/`

---

## 2. 工作流程（核心原则）

> 详细流程见 `.claude/skills/dev-workflow/SKILL.md`

1. **不直接写代码**，先读文档和代码
2. **先给计划**，等确认后再编码
3. **默认只改 1-3 个文件**
4. **遵守分层**：`core` 不依赖 `adapters`
5. **编码后运行** `ruff check --fix .`

**文档同步**：
- 架构/行为决定 → 更新 `docs/coding-log.md`
- Schema/涉及分层架构变更 → **必须先问**

**文档清理**（我会主动提醒）：
- 发现过时说明/与代码不一致/引用已删除模块时提醒你
- 历史决策统一记在 `coding-log.md`，不在其他文档堆"草稿"

**图表化原则**：
- 修改/新增文档时，适合用图表的地方优先用 ASCII 图（数据流/状态机/调用链/文件树）
- 代码总结时主动用图表展示变更（详见 `.claude/skills/code-style/SKILL.md`）

---

## 3. 编码规范（核心约束）

> 完整规范见 `.claude/skills/code-style/SKILL.md` 和 `.claude/skills/architecture/SKILL.md`

**类型与精度**：
- 类型注解必须全；金额/净值/份额用 `Decimal`（金额 2 位、净值/份额 4 位小数）
- Docstring 用中文

**分层约束**：
- `core`：核心逻辑 + 依赖注入（models + rules + dependency + container + config/log）
- `flows`：业务流程函数（纯函数 + `@dependency` 装饰器），依赖 core
- `data`：数据访问，依赖 core（DB Repo + 外部客户端）
- `cli`：只做参数解析与 Flow 函数调用，不写业务逻辑

**命名规范**：
- Repo 类：`TradeRepo`、`NavRepo`（直接具体类名）
- Service 类：`CalendarService`、`EastmoneyNavService`
- Flow 函数：小写蛇形（`create_trade()`、`confirm_trades()`、`make_daily_report()`）
- Result 类：`{名词}Result`（如 `ReportResult`、`ConfirmResult`）
- 依赖注入：`@dependency`、`@register`

**日志前缀**：`[EastmoneyNav]` `[LocalNav]` `[Discord]` `[Job:xxx]`（详见 `operations-log.md`）

**代码检查**：已配置 hook 自动运行（见 `.claude/settings.json`）
- 每次 `Edit` / `Write` 后自动执行 `ruff check --fix` + `pyright`
- 无需手动运行，hook 会自动触发
- 可捕获：字段名错误、参数错误、导入错误等

### 3.x 领域命名规范（DCA & AI 分工）

涉及 DCA、历史扫描、AI 分析的模块，遵循 **"规则算事实，AI 做解释"** 原则。

**数据模型**：`*Facts`（快照）/ `*Check`（检查）/ `*Flag`（标记）/ `*Draft`（建议）/ `*Result`（汇总）

**Flow 函数**：`build_*_facts()`（只读）/ `scan_*()`（只读）/ `draft_*()`（建议）/ `backfill_*()`（写库）

**AI 层预留**：`*Insight` / `*Explanation` / `*Label`（注释，不修改核心数据）

> 详细规范见 `.claude/skills/code-style/SKILL.md` 的"DCA & AI 分工命名"节

---

## 4. 配置与环境变量

敏感信息和可变参数通过环境变量 / `.env` 提供，**禁止写死在代码中**。

**典型配置**（由 `src/core/config.py` 读取，完整列表见该文件及 `docs/operations-log.md`）：
- `DISCORD_WEBHOOK_URL` / `DB_PATH` / `ENABLE_SQL_DEBUG`

**新增配置项**：先在文档写明用途 → 再在 `core/config.py` 集中读取 → **不直接用** `os.getenv`

---

## 5. 当前版本行为（只做跳转）

- **展示日 / NAV 严格口径 / 再平衡规则** → `docs/settlement-rules.md`
- **日报与抓取 Job 用法** → `docs/operations-log.md`
- **版本目标与完成状态** → `docs/roadmap.md`

---

## 6. 禁止事项（以 roadmap 为准）

当前阶段明确不做：
- 历史导入（只允许预留 TODO）
- 盘中估值作为核心口径（只允许未来做"附加字段"）
- 任意 AI 产品功能（仅做数据准备：标签/快照/Outcome）
- 复杂错误处理/审计框架
- 破坏性操作（删除顶层目录、重建 DB 结构等）

---

## 7. 提问原则

当不确定时，**先问**，例如：
- "当前任务是否允许修改 schema？"
- "这个 TODO 是现在实现，还是留到未来版本？"
- "是否可以新增第三方依赖 X？"

**涉及 schema/DB 或分层架构变更时，必须先问**。

---

## 8. 工具使用策略（极简版）

> 详细说明见各工具文档，这里只列核心原则

**基础优先**：
- 简单查询/修改 → `Grep` / `Glob` / `Read` / `Edit`

**按需升级**：
- 复杂设计/重构 → `Sequential-Thinking` MCP（先推理方案）
- 全局搜索 → `Code-Index` MCP（大范围/复杂模式）
- 外部资料 → `Exa` MCP（视环境而定）

**工具调用优先级**：
1. 简单 → Grep/Glob/Read/Edit
2. 探索 → Explore subagent
3. 全局搜索 → Code-Index MCP
4. 复杂推演 → Sequential-Thinking MCP
5. 外部调研 → Exa MCP

**流程梳理与可视化**：
- 当用户询问"某个功能怎么工作"时，使用 `.claude/skills/code-flow-viz/SKILL.md` 模板
- 生成 ASCII 流程图、数据流图、表格分析
- 适用场景：代码梳理、新功能设计、技术文档编写

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MMorit00) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
