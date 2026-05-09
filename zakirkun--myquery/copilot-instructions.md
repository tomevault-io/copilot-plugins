## web-ui

> Build a FastAPI-based Web UI for `myquery`, providing a conversational chat interface, schema explorer, and visualization dashboard.

# 🌐 Task: Web UI for myquery

## 🎯 Goal
Build a FastAPI-based Web UI for `myquery`, providing a conversational chat interface, schema explorer, and visualization dashboard.

## ⚙️ Requirements
- Create `web/main.py` using **FastAPI**.
- Build REST and WebSocket APIs:
  - `/api/query` → Execute natural language or SQL queries
  - `/api/analyze` → Generate AI-based summaries
  - `/api/visualize` → Serve Plotly visualization JSON
  - `/schema` → Get current database schema
- Integrate **React** or **Svelte** frontend:
  - Chat interface with prompt input
  - Query output table
  - Visualization panel
  - Connection status indicator
- Use **MCP (port 7766)** to sync context between CLI and Web UI.

## 🧠 Steps
1. Scaffold FastAPI app with routes in `web/routes/`.
2. Add WebSocket for live responses.
3. Connect backend to LangChain agent pipeline.
4. Add minimal frontend (React/Svelte + TailwindCSS).
5. Integrate chart rendering using Plotly.js.
6. Add “debug mode” toggle in UI to show SQL and logs.
7. Expose `/health` endpoint for monitoring.

## 🧩 Cursor Hints
- Use `@cursor: define-tool web_ui_tool`
- Keep backend modular — do not mix agent logic inside routes.
- Use FastAPI dependency injection for DB and agent context.
- Make the Web UI theme consistent with CLI color palette.

---
> Source: [zakirkun/myquery](https://github.com/zakirkun/myquery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
