## curso-agente-riesgos-back

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FastAPI-based REST API that acts as an AI agent for construction project risk management. It conducts a structured multi-turn conversation with users (via OpenAI GPT-4o), collects project details, identifies risks per ISO 31000/PMBOK standards, and generates professional PDF reports with heat maps and Monte Carlo simulations.

## Commands

**Install dependencies:**
```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**Run locally:**
```bash
uvicorn main:app --reload
```
Server starts at `http://localhost:8000`.

**Required environment variable** (copy `.env.example` to `.env`):
```
OPENAI_API_KEY=sk-...
```

**Deploy to Railway:** `procfile` handles production startup with `$PORT`.

## Architecture

```
POST /iniciar       → Creates session, initializes chat with system prompt
POST /chat          → Multi-turn conversation with GPT-4o, detects completion
GET  /reporte/{id}  → Generates and returns PDF report
GET  /health        → Health check
```

**Session state** is stored in two in-memory dicts in `main.py`:
- `sessions[session_id]` — full conversation history (list of messages)
- `sessions[session_id + "_data"]` — parsed risk analysis JSON (set when LLM outputs `ANÁLISIS_LISTO`)

**Conversation flow** (defined in `app/prompt.py`): 4 phases × 3 questions each, covering general context → technical/external → cost/schedule → legal/safety. When the LLM has enough info, it outputs the marker `ANÁLISIS_LISTO` followed by a JSON block. `main.py` detects this marker, extracts the JSON, and stores it.

**PDF generation** (`app/pdf_generator.py`): 4-page report — cover, executive summary, risk register table, and a heat map + Monte Carlo histogram. Monte Carlo runs 10,000 simulations using triangular distributions over each risk's economic impact range to produce P50/P80/P90 cost percentiles.

## Key Constraints

- **In-memory sessions only** — all session data is lost on server restart; no database or Redis.
- **Single LLM call per `/chat`** — full conversation history is sent each turn (context grows with conversation length).
- **`max_tokens=2000`** in the OpenAI call — overly complex analyses may be truncated.
- **CORS is open (`allow_origins=["*"]`)** — intended for Lovable frontend integration during development.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EffrenAnthony)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EffrenAnthony)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
