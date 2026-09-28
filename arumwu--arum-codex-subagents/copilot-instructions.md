## arum-codex-subagents

> - 全程使用台灣繁體中文撰寫說明、註解與 commit message。

# 專案工作規則

- 全程使用台灣繁體中文撰寫說明、註解與 commit message。
- `agents/` 是 12 個公開 profiles 的 source of truth；不要直接從上游整包覆蓋。
- 每個 agent 必須使用 `.toml`，且包含 `name`、`description`、`developer_instructions`。
- `name` 只能使用小寫英文字母、數字與底線，並與檔名一致。
- production 永遠唯讀；不得加入 production 寫入、部署或資料庫 mutation 指令。
- 不得重新加入 `localhost:3000/mcp` 或 `gpt-5.3-codex-spark`。
- 修改後執行 `python3 scripts/validate_agents.py`；交付時如實回報已驗證與未驗證項目。
- 保留 VoltAgent 上游 MIT 授權與 `NOTICE.md` 歸屬說明。

---
> Source: [arumwu/arum-codex-subagents](https://github.com/arumwu/arum-codex-subagents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
