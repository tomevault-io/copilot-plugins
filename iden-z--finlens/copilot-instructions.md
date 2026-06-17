## finlens

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

FinLens (财报智析) — an A-share stock financial report analysis tool. Fetches financial data via AKShare, sends it to DeepSeek API for AI-powered analysis, and displays results in a React chat interface.

- **Backend** (Python/FastAPI, port 8000) — data fetching + DeepSeek API orchestration
- **Frontend** (React/Vite, port 3000) — chat UI with Markdown rendering

## Commands

```bash
# Backend (from backend/)
pip install -r requirements.txt
uvicorn main:app --reload --port 8000

# Frontend (from frontend/)
npm install
npm run dev        # dev server on port 3000
npm run build      # production build
```

One-click: `start.bat` (Windows only, requires `DEEPSEEK_API_KEY` env var).

## Architecture

### Backend (`backend/main.py`)

Single-file FastAPI app. All financial data fetching and AI conversation management lives here.

**API routes:**
- `GET /health` — health check
- `POST /analyze` — fetch 4 financial statements for a stock, build a system prompt containing all data, get initial AI analysis, return `session_id`
- `POST /chat` — continue conversation within a session (full message history preserved)
- `GET /sessions` — list active sessions
- `DELETE /sessions/{id}` — delete a session

**Data flow for `/analyze`:**
1. AKShare pulls income statement, balance sheet, cash flow statement, and key financial indicators (新浪财经 source via `stock_financial_report_sina` and `stock_financial_analysis_indicator`)
2. `_clean_df()` trims to latest 8 quarters and filters empty rows to control token usage
3. `_build_system_prompt()` constructs a Chinese-language analyst persona prompt embedding all four statements
4. DeepSeek (via OpenAI-compatible API) generates the initial analysis; the session (system prompt + message history) is stored in an in-memory `sessions` dict

**Key details:**
- Sessions are in-memory only — restarting the backend loses all sessions
- Stock code → exchange prefix: codes starting with `6` → `sh`, codes starting with `0`/`3` → `sz`
- CORS restricted to `localhost:3000`

### Frontend (`frontend/src/App.jsx`)

Single React component managing all state. Two modes:
1. **Form mode** (no session) — input stock code, name, optional focus area → calls `/analyze`
2. **Chat mode** (session active) — Markdown-rendered conversation, calls `/chat` for follow-up questions

Renders assistant messages as Markdown via `react-markdown`. Enter sends message; Shift+Enter for newlines in the textarea. "新分析" button resets the session.

### Data sources

AKShare free tier (新浪财经). Subject to occasional timeout — errors are surfaced directly to the user in the UI.

## Configuration

Edit `backend/config.json` (primary, in .gitignore):

```json
{
  "api_key": "sk-xxx",
  "model": "deepseek-v4-pro",
  "base_url": "https://api.deepseek.com"
}
```

Environment variables (`DEEPSEEK_API_KEY`, `DEEPSEEK_MODEL`, `DEEPSEEK_BASE_URL`) work as fallbacks. `config.json` takes priority.

Python 3.11+, Node.js 18+.

---
> Source: [IDen-z/FinLens](https://github.com/IDen-z/FinLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
