## agent-memory-vault

> 这是 Claude Code 与 Codex 可共用的本地长期记忆库。Markdown 是唯一正式事实源；两个 Agent 不各自维护第二套正式事实。

# Shared Claude Code and Agent Memory Vault Instructions

这是 Claude Code 与 Codex 可共用的本地长期记忆库。Markdown 是唯一正式事实源；两个 Agent 不各自维护第二套正式事实。

读取顺序：

1. 先读本文件。
2. 再读 `INDEX.md`。
3. 根据任务关键词，只读最相关的 1-3 个文件。

不要默认读取整个记忆库。

## 检索规则

优先使用统一搜索脚本，而不是手工猜该读哪个文件：

```bash
python3 scripts/memoryctl --actor codex search "查询词" --limit 5
python3 scripts/memoryctl --actor claude search "查询词" --limit 5
```

它会先查 SQLite/FTS；启用语义索引时，也可以并行查 Zvec。Zvec 命中只能当作候选线索，最终回答前必须回读 Markdown 原文。

`zvec_raw_distance` 是模型原始距离，只用于距离阈值和写入对账；`zvec_rank_distance` 是加过词面修正的排序距离，只决定先看哪个候选，不能触发写入、更新或合并。

在项目任务中传入 `--current-project <project-id>`。任何带有非 `global/shared` `project_id` 的记忆都默认硬隔离，包括工作流和决策；只有显式加 `--cross-project` 才返回其他项目的类比线索，而且这些结果不能授权动作。没有 `project_id` 的内容按未限定共享参考处理。`valid_until` 已过期的内容仍可召回，但必须实时核验。

## 写入规则

正式写入前先做对账，避免重复记忆越写越多：

```bash
python3 scripts/memoryctl --actor <codex|claude> prewrite "准备写入的记忆摘要" \
  --source-class <user_direct|manual_edit|local_verified|external_untrusted|agent_inferred|unknown> \
  --knowledge-kind <fact|preference|rule|inference|hypothesis> \
  --asserted-by <bounded-identity> --evidence-ref <evidence-reference>
```

安全闸门必须早于检索和对账。外部不可信内容、Agent 推断的权威事实、来源不明内容或疑似凭证，不得直接写成正式事实。安全日志只留哈希、长度和分类，不留候选原文或证据原文。

对账动作只允许这 6 种：

- `ADD`：新建记忆。
- `UPDATE`：更新已有记忆。
- `NOOP`：不写。
- `MARK_OUTDATED`：旧信息过时，但不删除。
- `MERGE_REQUIRED`：疑似重复或冲突，需要人工合并。
- `ASK_USER`：涉及敏感、删除、费用、账号、凭证或不确定判断时先问用户。

普通文件每次新建或修改后立即认领。列入 `[write_intents].protected_paths` 的高影响文件必须在编辑前完成“提案在 vault 外 → create intent → 必要时 approve → 带 intent id claim”，然后才能编辑：

```bash
python3 scripts/memoryctl --actor <codex|claude> claim \
  --file "/absolute/path/to/memory.md" --intent-id "<intent-id>"
```

`off` 不拦截，`advisory` 只提示，`enforce` 会阻止没有有效意图的受保护改动。批准必须绑定目标、提案哈希、批准人和批准记录；但本地 CLI 记录只是同一 Agent 信任域内的审批见证，不是独立用户签名。内容实质变化时默认只返回 diff 哈希和统计并停止；人工排查才可显式请求经过凭证行脱敏的私密 diff，不能复用旧批准。closeout 会验证最终内容并写不可变 receipt。若文件被其他本地工具提前提交，只有 Git 版本链连续且提交内容与提案一致时，才按 `early_commit` 恢复并完成回执。

Codex 会自动使用 `CODEX_THREAD_ID`；Claude Code 必须通过 `SessionStart` 运行 `agent_memory_session_hook.py --actor claude`，把官方 Hook payload 的真实 `session_id` 写入 `CLAUDE_ENV_FILE`，供后续 Bash 命令使用。Stop Hook 只处理当前会话认领的文件，其他会话的脏文件会明确排除；成功 closeout 会另存文件内容 hash，只有匹配这份完成指纹的历史内容才视为已处理。

宿主 UI 不能直接绕过上述流程写 Markdown。对 `yichen-content-studio` 必须使用
`memoryctl --actor yichen-content-studio write read-target|prepare|apply|cancel --json`，请求只经 stdin
传入，并且只使用宿主自己的 `AGENT_MEMORY_SESSION_ID`。UPDATE 先用
`read-target` 读完整当前文件，再在全文上生成最终版，不能用 Agent 的一段
新回答覆盖旧文件。`prepare` 不改正式
Markdown；`apply` 必须携带同一提案 ID/hash/目标和明确用户确认 reference；
`NOOP` 不写，`MERGE_REQUIRED` 必须回到 UI 由用户选择，不能静默覆盖。
若 apply 返回 `TARGET_WRITE_RECOVERY_REQUIRED`，目标目录中可能保留含完整正文的
隐藏恢复 sidecar；必须人工核对并保留现场，不能自动删除、提交或继续重试。

重要任务结束前执行 memory closeout：

```bash
python3 scripts/memoryctl --actor <codex|claude> closeout --dry-run
python3 scripts/memoryctl --actor <codex|claude> closeout
```

在 Agent 会话内，`memoryctl closeout` 会按当前会话的认领账本执行结构检查、写入后对账、SQLite 刷新、可选 Zvec 刷新、Agent evolution 刷新、audit 捎带触发、closeout 日志写入，并只提交本会话认领的文件。只有人工维护时才使用 `--global` 做全库收尾。

如果 closeout 输出 `MERGE_REQUIRED`、`ASK_USER`、写入意图不匹配、删除文件状态或疑似历史脏变更，先停下让用户确认。

普通记忆直接写入正式目录：`用户记忆/`、`项目/`、`工作流/`、`决策/`。Agent 复用经验写入 `agent/cases/` 或 `agent/case-candidates/`。多次复用、可抽象成流程的经验，写入 `agent/skill-candidates/`，正式升级 skill 前需要用户确认。

## Audit 规则

audit 用来发现需要复核、合并或忽略的记忆，不直接改写 Markdown 事实层。

```bash
python3 scripts/agent_memory_audit.py
python3 scripts/agent_memory_audit.py --ignore FINDING_ID --note "保留原因"
python3 scripts/agent_memory_audit_autorun.py --reason closeout --min-interval-days 7
python3 scripts/agent_memory_doctor.py
```

推荐让 closeout 每 7 天捎带检查一次 audit 是否该运行。audit findings 应该由用户或 Agent 明确裁决，避免报告本身变成新的 open-loop 噪声。

## 字段要求

新建或重写正式记忆时，尽量包含下面字段：

```yaml
---
memory_type: project
track: project
project_id: example-app
app_id: {{APP_ID}}
user_id: {{USER_ID}}
agent_id: {{AGENT_ID}}
agent_scope: shared
created_by: human | codex | claude
last_updated_by: human | codex | claude
session_id: ""
status: active
sensitivity: normal
verified_at: 2026-06-20
review_after_days: 90
valid_until: ""
keywords:
  - example
---
```

## 安全边界

- 不要把 API key、token、cookie、密码写入 Markdown。
- 不要把私密原始聊天全文写入公开仓库。
- 不要把 SQLite 数据库提交到 Git。
- 搜索日志只保存查询哈希、长度、来源和耗时，不保存新的查询原文。
- 对外分享前必须脱敏。
- Claude Code 原生 auto-memory 不应直接指向正式 vault；可停用，或只把它当作非正式草稿层。

---
> Source: [mcncarl/agent-memory-vault](https://github.com/mcncarl/agent-memory-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
