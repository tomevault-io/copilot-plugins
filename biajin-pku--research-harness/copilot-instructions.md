## research-harness

> Trigger this mode when the task is to modify, refactor, test, evaluate, or

## RH Self-Development Mode

Trigger this mode when the task is to modify, refactor, test, evaluate, or
document Research Harness itself.

In RH self-development mode:

- First read `docs/feedback/rh-self-dev-control-plane.md`.
- Do not automatically read `docs/agent-guide.md`; that guide is for using RH
  on research projects, not for modifying RH itself.
- Do not load or invoke RH research workflow skills unless the task is to use RH
  for a research project or to edit that exact skill/tool.
- Work only on the current slice from
  `docs/feedback/rh-self-dev-slices.md`.
- Stop and produce a gate report before applying cloud task diffs, changing
  public surface, touching storage/provenance/orchestrator gates, changing
  benchmark promotion claims, adding dependencies, or exceeding the slice scope.

## Research Harness 集成

本项目使用 Research Harness 进行科研流程管理。

### 系统位置

- Research Harness: project root
- Agent 使用手册: `docs/agent-guide.md`
- 数据库: `.research-harness/pool.db`

### 必读

开始任何科研工作前，先读取使用手册：

```bash
cat docs/agent-guide.md
```

### Codex 上下文压缩规则

为避免 Codex 在接近上下文上限时自动压缩失败（例如 `Error running remote compact task: stream disconnected before completion`），必须采用主动压缩策略：

1. **40% 软阈值**：当可见 context 剩余不足 40%（或没有可见仪表但会话已经完成一个主要阶段/读取了大量文件/进行了大量工具调用）时，不要继续开启新的大型检索、批量读取或多文件改动，先进行一次 checkpoint。
2. **Checkpoint 内容**：压缩前先输出或保存简短状态摘要，包含当前目标、已完成事项、关键发现、未完成步骤、相关文件、需要保留的用户偏好/约束。
3. **主动压缩**：checkpoint 后主动建议/执行 Codex `/compact`；如果当前环境不能由 agent 直接执行 `/compact`，则明确提示用户执行 `/compact` 并附上 checkpoint 摘要。
4. **30% 硬阈值**：当 context 剩余不足 30% 时，只允许做收尾、记录状态或请求压缩；禁止继续启动长任务。
5. **失败兜底**：如果 `/compact` 或远程 compact 仍失败，立即把 checkpoint 摘要写入可持久位置（memory 或项目文档中已有合适位置），然后建议开启新会话并粘贴该摘要恢复。

### 三条铁律

1. 论文必须入库：通过 research-harness MCP 的 `paper_ingest` 或 `rh paper ingest` CLI 入库，不要把论文散放在项目目录
2. 产出必须记录：关键产出通过 `orchestrator_record_artifact` 记录到数据库
3. 经验必须反馈：工具不足、流程改进、bug 优先开 GitHub Issue 或进入项目正式 backlog；不要在公共仓库新增一次性调试日志/报告

### ⚠️ MCP 优先，CLI 只做数据管理

**所有 research primitive（gap_detect、claim_extract、baseline_identify、section_draft、consistency_check 等）必须通过 MCP 工具调用，不要用 `rh` / `rh` CLI 执行这些操作。**

原因：CLI 绕过了 provenance 追踪、orchestrator gate 检查和 artifact 版本管理。

| 场景 | 用 MCP 工具 | 禁止 |
|------|------------|------|
| 识别 baseline | `baseline_identify(topic_id=X)` | `rh baseline-identify ...` |
| 提取 claims | `claim_extract(paper_ids=[...])` | `rh claim-extract ...` |
| 起草章节 | `section_draft(section=..., topic_id=X)` | `rh section-draft ...` |
| 检测 gap | `gap_detect(topic_id=X)` | `rh gap-detect ...` |

`rh` / `rh` CLI 只用于：`topic list`、`paper list`、`paper ingest`、`orchestrator status` 等只读/入库操作。

### 可用工具（MCP research-harness）

- `paper_search`：多源论文搜索
- `paper_ingest`：论文入库（arxiv_id / DOI / PDF）
- `paper_summarize`：论文摘要
- `claim_extract`：提取研究声明
- `gap_detect`：研究空白分析
- `baseline_identify`：baseline 识别
- `section_draft`：章节起草
- `consistency_check`：一致性检查
- `orchestrator_resume`：**接管已有项目的首选入口**，自动从现有 artifact 推断当前阶段（不会从头开始）
- `orchestrator_status` / `orchestrator_advance` / `orchestrator_gate_check`：流程编排
- `adversarial_review`：**独立跨模型对抗审查**（Claude 环境自动调 Codex/GPT，Codex 环境自动调 Opus，保证 reviewer 独立性）
- `advisory_check` / `provenance_summary` / `provenance_export`：工作流建议与溯源审计

### Codex 中的 Research Harness Skills

如果已在 `~/.codex/skills/` 安装 Research Harness skills，优先按以下触发词理解：

- `research-harness`
- `research-init`
- `literature-search`
- `literature-mapping`
- `citation-trace`
- `claim-extraction`
- `gap-analysis`
- `evidence-gating`
- `section-drafting`
- `paper-verify`
- `provenance-review`

兼容输入写法：

- 直接写 skill 名称，如 `literature-search`
- 使用 Claude 风格前缀，如 `/literature-search`
- 入口分流，如 `/research-harness 帮我初始化一个新 topic`
- 在中文里明确说“用 literature-search skill”或“执行 /research-init”

注意：Codex 没有 Claude Code 那种原生 `/命令` 注册表。这里的 `/xxx` 是约定式触发词，不是 REPL 内建命令。

### 可用 Agents

- `literature-mapper`
- `proposer`
- `challenger`
- `adversarial-resolver`
- `synthesizer`

### 自由度

你可以按任意顺序、任意方式使用以上工具。12 阶段流程是参考框架，不是强制要求。人类研究员随时可以干预。

---
> Source: [Biajin-PKU/research-harness](https://github.com/Biajin-PKU/research-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
