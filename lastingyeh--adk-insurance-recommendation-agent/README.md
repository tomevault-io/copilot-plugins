# adk-insurance-recommendation-agent

> Source: [lastingyeh/adk-insurance-recommendation-agent](https://github.com/lastingyeh/adk-insurance-recommendation-agent) — distributed by [TomeVault](https://tomevault.io)

基於 Google ADK 與 MCP Toolbox 的保險科技推薦 Agent 原型。核心 Agent 能理解保險需求、主動追問，並透過受控 SQL 工具查詢醫療、意外、家庭與收入保障產品。後端採 FastAPI 提供 SSE 串流與 WebSocket 即時互動，前端以 Next.js 呈現對話介面。知識庫結合 PostgreSQL 與 pgvector 實現 FAQ 語意搜尋與 RAG 查詢，並支援多模態 Live Agent 語音互動。安全層涵蓋 JWT 驗證、PII 脫敏與 SHA-256 雜湊鏈防篡改稽核日誌。品質保證採 ADK evalsets 評估驅動開發，部署支援 Docker Compose、Cloud Run 與 Terraform 多環境配置。

## GitHub Copilot Config

The `copilot-instructions.md` file in this directory is the project config converted for GitHub Copilot.
Original source: `GEMINI.md` in [lastingyeh/adk-insurance-recommendation-agent](https://github.com/lastingyeh/adk-insurance-recommendation-agent).

## Also available for

- **Claude Code** — `CLAUDE.md`
- **Codex** — `AGENTS.md`
- **GitHub Copilot** — `copilot-instructions.md`
- **Cursor** — `project-config.mdc`
- **Windsurf** — `project-config.md`

From [lastingyeh/adk-insurance-recommendation-agent](https://github.com/lastingyeh/adk-insurance-recommendation-agent) — a repo with 9+ stars on GitHub.

---

Own this repo? Install the TomeVault Relay to keep every platform's copy in sync on every push: [https://tomevault.io/install](https://tomevault.io/install).

<!-- genome:d-c-s -->
