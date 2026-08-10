## conversation-logs

> 用户给出 32 位 hex ID / UUID、贴排查包、或说查产品 AI 日志时读——用 logs/dev.jsonl + DB，禁止在文件系统搜文件名。


# 对话日志查询

## 两套日志，不要混

| | 产品 AI 日志 | Cursor IDE Agent 会话 |
|---|---|---|
| **位置** | `logs/dev.jsonl` + Postgres | `agent-transcripts/`（UUID.jsonl） |
| **ID** | 32 位 hex 无连字符 = **trace_id**；带连字符 UUID = **conversation_id** | 文件名 = UUID 带连字符 |

## 铁律

- 用户给 trace_id / conversation_id → **`logs/dev.jsonl` + DB**，**禁止**在文件系统搜。
- `agent-transcripts/` 是 Cursor 编码 Agent 转录，与产品 AI **无关**。
- 日志**不含消息正文**，正文在 Postgres。

## 查询入口（apps/server 下）

```bash
uv run python scripts/log_timeline.py --trace <trace_id>          # 默认 decision_spine
uv run python scripts/log_timeline.py --pack <dir> --trace <tid>  # 排查包制品
uv run python scripts/log_timeline.py --pack <dir> --full --trace <tid>
uv run python scripts/log_timeline.py --raw --trace <trace_id>    # 全量仅需要时
uv run python scripts/log_timeline.py <conversation_id>
uv run python scripts/log_timeline.py --recent
uv run python scripts/log_stats.py --json
```

有 ID → 优先 **decision_spine** / **排查包**（`--pack`）；全量 `--raw` 仅深挖需要时。包内必有 `decision_spine.json` + `timeline.jsonl` + `meta.json`（`schema_version`）；另一会话只读包即可复盘。`spine_events` = **turn_spine**，勿与产品面混称。

**Token 两口径**：spine 的 `llm.*_tokens` = 全 trace `llm.call` 合计；`tail`/`turn_metrics` = 收口折账。`kind=resume` 时后者通常不含 pause 前段——差值为预期，见 [对话日志分析指南 · Token 两口径](/docs/05-平台与运维/对话日志分析指南.md)。

Dogfood 入槽（出包后）：仓库根 `python evals/dogfood/fill_from_pack.py --pack <dir>` → [`evals/dogfood/README.md`](/evals/dogfood/README.md)。

生产：本地无命中且像线上 → `pnpm sync:logs`（Node；读 `DEPLOY_SSH_*`；默认瘦身跳过 `turn_journal`，深挖加 `--full`）后 `--export-dir ../../logs/prod-export`（可再 `--pack`）。勿再用 `AGENTCORE_SSH` / `$AGENTCORE_HOME/logs`（已退役）。

**线上巡检**：父级拆桶 / 默认窗 / 审阅桶纪律 / **归因闸** / **状态两轴（代码态≠发布态）** / ④先纠发布态再归因再码核 → [`logs/reviews/README.md`](/logs/reviews/README.md)；开案/修码前先读 [`logs/reviews/STATUS.md`](/logs/reviews/STATUS.md)。

→ 详解：[对话日志分析指南](/docs/05-平台与运维/对话日志分析指南.md)

## 查询策略

- **无连字符 32-hex** → 一律当 `trace_id`，用 `--trace`（勿先当 conversation_id）
- **带连字符 UUID** → conversation_id；查全貌用位置参数，单轮细节再 `--trace`
- **无 ID / 仅截图 /「查最新」** → `--recent` 后必须用文案/`preview` **确认匹配**再深挖，勿默认第一条
- **本地空、ID 像线上** → 先 sync / `--export-dir`，勿在空库硬猜
- **ID 已齐**（cid 或 trace）→ 主 Agent **直接**跑 `log_timeline --json` 或 `--pack`，勿为单次查询开后台探子干等

---
> Source: [Lawofall/AgentCore](https://github.com/Lawofall/AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
