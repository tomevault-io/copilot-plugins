## chinese-language

> 全程使用中文思考、对话与文档产物


# 中文优先

## 思考与对话

- 内部推理、计划、对用户回复一律使用**中文**。
- 技术术语可保留英文（如 Codex、SSE、API），但说明性文字用中文。

## 文件与产物

- 用户可见的 Markdown 文档、`.ai/` 流水线产物、任务单、摘要、评审结论：**必须使用中文**。
- 代码标识符、配置键、Git 哨兵值（`NO_NEXT_TASK`、`NO_EXPANSION`）可保持英文。
- 新增或修改 `scripts/ai_pipeline.py` 生成的提示词、日志、快照文件时，默认中文。

## 流水线

- 运行 `ai_pipeline.py` 时，Codex / Cursor / DeepSeek 交接文件与 `CHAT_SNAPSHOT.md` 均为中文。
- Agent 监控流水线时，将 `.ai/CHAT_SNAPSHOT.md` 内容以中文贴回聊天，不要只报英文状态码。

---
> Source: [lh0614/stock-analysis](https://github.com/lh0614/stock-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
