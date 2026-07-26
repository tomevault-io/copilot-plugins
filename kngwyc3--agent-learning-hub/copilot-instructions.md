## agent-learning-hub

> > 本文档面向维护者与 AI Agent：快速理解仓库目标、目录结构、各 Stage 关系与修改原则。人类学习者请优先读 [README.md](README.md) 和 [index.html](index.html)。

# Agent Learning Hub 总览

> 本文档面向维护者与 AI Agent：快速理解仓库目标、目录结构、各 Stage 关系与修改原则。人类学习者请优先读 [README.md](README.md) 和 [index.html](index.html)。

## 仓库目标

这个仓库的核心目标，是把 AI Agent 学习路径整理成一份可执行、可打勾、可落地的 roadmap，而不是单纯堆链接。

它强调的不是“花哨的多智能体演示”，而是更接近真实生产的能力：

- 最小 agent loop
- 工具调用与严格 schema
- RAG 与引用
- 长期记忆与上下文压缩
- harness engineering
- Skills / MCP / A2A / ACP 等能力封装与协议
- Browser / computer-use agent
- 权限与安全边界
- 评测、trace 与可观测性

## 展示入口

| 入口 | 路径 | 用途 |
| --- | --- | --- |
| 主路线图 | [README.md](README.md) | Stage 0–9 学习清单、Project Ladder、精选资源 |
| 交互式学习页 | [index.html](index.html) | Stage 导航、资源卡片、进度勾选（本地 `python -m http.server` 访问） |
| 仓库总览 | [agent.md](agent.md) | 本文件：结构说明、Stage 映射、维护原则 |
| 分步教程 | [stage-1/](stage-1/) … [stage-9/](stage-9/) | 可运行代码与递增练习 |

## 仓库结构

```text
Agent-Learning-Hub/
  README.md                 # 主路线图
  index.html                # 交互式学习页
  agent.md                  # 本总览文档
  CONTRIBUTING.md           # 贡献指南
  stage-1/                  # 最小 agent loop（6 步 Python）
  stage-2/                  # RAG + 记忆（7 步 Python）
  stage-3/                  # Claude Code harness 12 章导读
  stage-4/                  # Multi-agent coordination（pipeline / supervisor）
  stage-5/                  # Skills 能力打包（4 步 + smoke test）
  stage-6/                  # Browser agent（Playwright + 安全边界）
  stage-7/                  # Eval / trace / 安全门禁
  stage-8/                  # 可部署 CLI Agent（trace / 安全 / 成本上限）
  stage-9/                  # 上下文压缩与记忆（compaction + memory 最小实现）
  skills/teach/             # AI 导师 skill（本仓库自带，学习时直接使用）
```

---

## 各 Stage 说明

### `stage-1/` — 最小 Agent Loop

**目标**：用 Python + OpenAI 兼容 API 搭出「能选工具、能执行、能循环」的最小 Agent。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_chat.py` | LLM 普通对话 |
| 2 | `step02_json.py` | 结构化 JSON 输出 |
| 3 | `step03_tools_def.py` | 工具 schema 定义 |
| 4 | `step04_one_round_tool.py` | 单轮 tool call 解析与执行 |
| 5 | `step05_agent_loop.py` | 完整 agent loop |
| 产出 | `agent.py` | 50–150 行最小 agent，含步数/超时/错误处理 |

```bash
cd stage-1 && pip install -r requirements.txt && cp .env.example .env
python step01_chat.py
```

---

### `stage-2/` — 工具、RAG 与记忆

**目标**：在 Stage 1 之上加入检索增强、长期记忆和上下文压缩。

| 模块 | 文件 / 工具 | 内容 |
| --- | --- | --- |
| 记忆分层 | `step01_memory_layers.py` | 短期 / 会话 / 长期记忆区分 |
| RAGFlow | `step02`–`step04` | chunk / embed / retrieve / 带引用回答 |
| mem0 | `step05_mem0_memory.py` | 长期记忆写入与召回 |
| Letta | `step06_letta_compaction.py` | 上下文压缩与长对话管理 |
| RAG as Tool | `step07_rag_as_tool.py` | 把 RAG 封装成 agent 工具 |
| 产出 | `agent.py` | 带引用的资料研究助手 |

```bash
cd stage-2 && pip install -r requirements.txt && python step01_memory_layers.py
```

---

### `stage-3/claude-code-docs/` — Claude Code Harness 导读

**目标**：研究现代 coding agent harness 的工程实现，不是教学脚本，而是「生产级样本」的拆解文档。

> 路径已从旧名 `claude-code-source-code/` 更正为 `claude-code-docs/`。

| 章节 | 文件 | 核心内容 |
| --- | --- | --- |
| 00 | [00-概览与项目结构.md](stage-3/claude-code-docs/00-概览与项目结构.md) | 架构全貌、技术栈、最小 Agent 循环 |
| 01 | [01-Tool系统.md](stage-3/claude-code-docs/01-Tool系统.md) | Tool 接口、buildTool、40+ 工具、BashTool |
| 02 | [02-Query引擎.md](stage-3/claude-code-docs/02-Query引擎.md) | Agent 主循环、流式、Auto Compact |
| 03 | [03-Agent系统.md](stage-3/claude-code-docs/03-Agent系统.md) | 子 Agent、Fork Worktree、蜂群协作 |
| 04 | [04-Task系统.md](stage-3/claude-code-docs/04-Task系统.md) | TaskType、状态机、磁盘输出流 |
| 05 | [05-状态管理.md](stage-3/claude-code-docs/05-状态管理.md) | AppState、Store 模式、Speculation |
| 06 | [06-权限系统.md](stage-3/claude-code-docs/06-权限系统.md) | PermissionMode、规则引擎、DenialTracking |
| 07 | [07-MCP集成.md](stage-3/claude-code-docs/07-MCP集成.md) | MCP 协议、ToolSearch 延迟加载 |
| 08–11 | 服务层 / UI 层 / CLI / 设计精髓 | 压缩、Analytics、TUI、设计模式 |

**与 Stage 7 的衔接**：[stage-7/docs/claude-code-permissions.md](stage-7/docs/claude-code-permissions.md) 把 CC 权限链路与 `safety_gate.py` 做了对照说明。

---

### `stage-4/` — Multi-Agent 协调

**目标**：理解多 agent 是协调问题，不是魔法。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_roles_contracts.py` | 角色职责、输入输出契约、停止条件 |
| 2 | `step02_fixed_pipeline.py` | research → write → review → revise 固定流水线 |
| 3 | `step03_supervisor_router.py` | supervisor 路由下一步 |
| 4 | `step04_stop_conditions.py` | 防止循环、争论和任务漂移 |
| 5 | `step05_single_vs_multi.py` | 判断什么时候单 agent 更好 |
| 专题 | [docs/learn/a2a-vs-shared-state.md](docs/learn/a2a-vs-shared-state.md) | A2A 与共享状态的工程边界 |
| 产出 | `agent.py` | 可调试的多 agent 写作系统 |

```bash
cd stage-4 && pip install -r requirements.txt
python agent.py "写一段解释 supervisor 模式的短文"
```

---

### `stage-5/` — Skills 与能力打包

**目标**：把一类 agent 能力从「临时 prompt」升级成可复用、可测试、可分发的 skill。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_boundaries.py` | Skill vs Tool / Prompt / MCP 边界 |
| 2 | `step02_load_skill.py` | 加载并校验 `SKILL.md` frontmatter |
| 3 | `step03_validate_report.py` | 验收报告格式检查 |
| 4 | `step04_run_smoke_cases.py` | smoke test 跑通 |
| 公共模块 | `skill_common.py`, `report_check.py` | skill 加载与报告校验 |
| 产出 | `my-skill/` | 完整 skill 示例（SKILL.md + 模板 + 脚本 + 测试） |

```bash
cd stage-5 && python step01_boundaries.py && python step04_run_smoke_cases.py
```

---

### `stage-6/` — Browser Agent

**目标**：让 agent 操作公开网页，并记录截图、DOM、动作日志，同时严守安全边界。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_validate_url.py` | URL 白名单 / 黑名单校验（无 Playwright 依赖） |
| 2 | `step02_observe_page.py` | 页面观察与 DOM 摘要 |
| 3 | `step03_run_agent.py` | 完整 browser agent 运行 |
| 公共模块 | `browser_policy.py`, `browser_common.py` | URL 策略、ActionLogger、登录拦截 |
| 产出 | `browser-agent/agent.py` | 公开网页 extractor |
| 策略 | `browser-agent/policies.md` | 不登录、不越权、不绕过平台规则 |

```bash
cd stage-6 && pip install -r requirements.txt && playwright install chromium
python step01_validate_url.py https://example.com
python step03_run_agent.py https://example.com
```

---

### `stage-7/` — Evaluation、Observability、Safety

**目标**：把 agent 从 demo 推进到可评测、可追踪、可回归、可控风险。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_load_tasks.py` | 加载 eval 任务集 |
| 2 | `step02_run_eval.py` | 运行 eval，写入 results + trace |
| 3 | `step03_safety_gate.py` | 安全门禁演示 |
| Eval | `evals/tasks.csv`（20 条） | input / must_have / must_not / risk_level |
| Runner | `scripts/eval_runner.py` | 成功率、失败分类、trace JSONL |
| 回归 | `scripts/compare_results.py` | baseline 对比 |
| 安全 | `safety_gate.py`, `safety/policy.md` | block / approval_required / allow |
| 专题 | `docs/claude-code-permissions.md` | CC 权限与 safety_gate 对照 |

```bash
cd stage-7
python step01_load_tasks.py
python step02_run_eval.py
python step03_safety_gate.py
```

---

### `stage-8/` — 交付真实 Agent

**目标**：有明确用户、任务、成功标准，带日志/trace/权限/部署方式的完整项目。

| 模块 | 文件 | 内容 |
| --- | --- | --- |
| 配置 | `.env.example`, `common.py` | settings、trace、成本上限 |
| 工具 | `tools.py` | calculator + repo-scoped read_file |
| 安全 | `safety.py` | block / approval_required / allow |
| Agent | `agent.py` | tool loop、重试、超时、max steps |
| CLI | `cli.py` | 可运行入口 |
| Smoke | `step01_smoke.py` | 离线 smoke test |

```bash
cd stage-8 && pip install -r requirements.txt
python step01_smoke.py
```

---

### `stage-9/` — 上下文压缩与记忆

**目标**：从零实现上下文压缩（compaction）与长期记忆（memory）的最小版本，理解「长会话为什么不爆窗、用户事实怎么跨任务保留」。与 Stage 2 用 mem0 / Letta 的「用现成库」形成递进：本阶段自己写压缩与记忆，把机制看清。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_window_basics.py` | 认识上下文窗口与 token 成本 |
| 2 | `step02_sliding_window.py` | 滑动窗口压缩（保锚点 + 最近消息） |
| 3 | `step03_summarize_compact.py` | 摘要式压缩（旧对话压成一条） |
| 4 | `step04_memory_read_write.py` | 长期记忆写入与召回 |
| 5 | `step05_loop_with_memory.py` | compaction + memory 接进 60 轮 loop |
| 公共模块 | `compactor.py`, `memory_store.py` | 压缩与记忆核心实现 |
| 产出 | 带 compaction + memory 的长任务 agent | 可见 token 节省 |

```bash
cd stage-9 && pip install -r requirements.txt
python step01_window_basics.py
python step05_loop_with_memory.py
```

**与 cc流程图 的衔接**：[stage-3/cc流程图.jpg](stage-3/cc流程图.jpg) 的 Context Compact / Reactive Compact 节点，正是 `compactor.py` 里「主动每轮压缩 / 被动 413 紧急压缩」两种模式的工程来源；其中的「锚点（anchor）」机制保证系统约束、用户要求、工具 schema 不被压丢。

---

## 主学习路线（Stage 0 → 9）

| Stage | 主题 | 本仓库材料 | 产出 |
| --- | --- | --- | --- |
| 0 | 理解 agent 是什么 | README 清单 | 一页笔记：为什么需要 agent |
| 1 | 最小 agent loop | `stage-1/` 代码 | 50–150 行可运行 agent |
| 2 | RAG + 记忆 | `stage-2/` 代码 | 带引用的资料研究助手 |
| 3 | 现代 harness | `stage-3/claude-code-docs/` | harness demo + trace 解读 |
| 4 | 多 agent 协调 | `stage-4/` 代码 | research → write → review 流水线 |
| 5 | Skills 打包 | `stage-5/` 代码 | 可复用 SKILL.md + smoke test |
| 6 | Browser agent | `stage-6/` 代码 | 公开网页 agent + action log |
| 7 | Eval + 安全 | `stage-7/` 代码 | 20 条 eval + trace + 安全门禁 |
| 8 | 交付真实 agent | `stage-8/` 代码 | 可 clone 运行的 CLI agent |
| 9 | 上下文压缩与记忆 | `stage-9/` 代码 | 带 compaction + memory 的 60 轮长任务 loop |

**推荐顺序（有代码阶段）**：

```text
Stage 1 → Stage 2 → Stage 3 导读 → Stage 5 → Stage 6 → Stage 7 → Stage 8
                              ↘ Stage 4 可与 Stage 3 并行 ↗
Stage 9（上下文压缩与记忆）可在 Stage 2 之后任意阶段深入
```

Stage 3 与 Stage 7 建议对照阅读：先读 `06-权限系统.md`，再读 `stage-7/docs/claude-code-permissions.md`，最后跑 `step03_safety_gate.py`。

---

## 项目阶梯

仓库把实战目标拆成 11 个层级（详见 README Project Ladder）：

1. Calculator Agent
2. Web Research Agent
3. PDF QA Agent
4. Coding Review Agent
5. Browser Agent
6. Claude Code-like Nano Agent
7. OpenClaw-like Gateway
8. Reusable Skill Pack
9. Multi-Agent Writer
10. Personal Agent
11. Production Harness

意图：先做小而稳的 agent，再逐步补齐工程能力、记忆、协议和安全边界。

---

## Stage 与 Project Ladder 映射

| Project Ladder | 对应 Stage |
| --- | --- |
| 1 Calculator Agent | Stage 1 |
| 2 Web Research Agent | Stage 2 |
| 5 Browser Agent | Stage 6 |
| 8 Reusable Skill Pack | Stage 5 |
| 9 Multi-Agent Writer | Stage 4 |
| 11 Production Harness | Stage 7 + Stage 8 |
| 10/11 Personal / Production Agent | Stage 9（长运行记忆与上下文压缩） |

---

## 资源选择原则

仓库不鼓励“无脑刷链接”，而是按用途选资源：

- 官方文档优先
- 可运行开源项目优先
- 真实工程经验优先
- 高质量论文和 benchmark 优先
- 小而明确的项目优先

要避免：

- 低质量社媒搬运
- 付费课广告
- 无法验证的资料
- 诱导绕过平台规则的内容

---

## 核心判断标准

- 先做出能跑的最小 agent，再加功能
- 先 trace，再扩展复杂度
- 先 eval，再加更多 agent
- 多 agent 不是魔法，本质是协调问题
- 工具必须有严格 schema 和权限边界
- 风险操作必须保留人工确认

---

## 对 Agent / 维护者的使用建议

### 人类学习者

1. 读 [README.md](README.md) 或打开 [index.html](index.html)
2. 按 [stage-1/README.md](stage-1/README.md) 跑最小 loop
3. 按 [stage-2/README.md](stage-2/README.md) 加 RAG 和记忆
4. 读 [stage-3/claude-code-docs/README.md](stage-3/claude-code-docs/README.md) 学 harness
5. 按 stage-5 → stage-6 → stage-7 顺序补 Skills、Browser、Eval

### AI Agent 修改本仓库时

优先遵守：

- **README-first**：主路线图变更同步更新 README 与 agent.md
- **高信号、低噪音**：不堆无关链接
- **资料可验证**：引用官方文档或可运行项目
- **目标明确**：每个 stage 目录有清晰产出与运行命令
- **最小 diff**：Stage 代码遵循递增 step 结构，公共逻辑抽到 `*_common.py`
- **不把仓库改成链接垃圾场**

### 学习者使用 AI 导师

本仓库 [skills/teach/SKILL.md](skills/teach/SKILL.md) 提供了一个 `teach` skill。当学习者希望 AI 协助学习时，可以告知 AI Agent 加载此 skill：

- AI Agent 会以"教学空间"方式工作，创建 MISSION.md、课程、参考材料和练习。
- 学习者只需告诉 Agent "请加载 skills/teach/SKILL.md 来教我学习 Agent 开发"，Agent 就能按"知识-技能-智慧"递进方式引导学习。
- 推荐配合 README 中的 Learning Todo List 使用。

修改 Stage 代码时的约定：

| 约定 | 说明 |
| --- | --- |
| step 脚本 | `step01_*.py` … `stepNN_*.py`，每步可独立运行 |
| 公共模块 | `common.py` / `skill_common.py` / `eval_common.py` 等 |
| 环境 | 各 stage 自带 `requirements.txt` 和 `.env.example`（如需要） |
| 文档 | 各 stage 的 README 与主 README 的检查项表格对齐 |
| 测试 | smoke test 或 eval CSV，不追求全覆盖单元测试 |

---

## 关键文件速查

| 需求 | 文件 |
| --- | --- |
| 学习路线图 | `README.md` |
| 交互式浏览 | `index.html` |
| 仓库结构说明 | `agent.md`（本文件） |
| CC 权限源码导读 | `stage-3/claude-code-docs/06-权限系统.md` |
| CC 权限与 Stage 7 对照 | `stage-7/docs/claude-code-permissions.md` |
| Eval 任务定义 | `stage-7/evals/tasks.csv` |
| 安全门禁实现 | `stage-7/safety_gate.py` |
| Skill 示例 | `stage-5/my-skill/SKILL.md` |
| Teach Skill（AI 导师） | `skills/teach/SKILL.md` |
| Browser 安全策略 | `stage-6/browser-agent/policies.md` |
| 上下文压缩与记忆示例 | `stage-9/` |

---
> Source: [kngwyc3/Agent-Learning-Hub](https://github.com/kngwyc3/Agent-Learning-Hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
