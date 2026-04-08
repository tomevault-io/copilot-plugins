## investment-portfolio-manager

> You are a Financial Engineer and Python Expert. You are assisting Bill (CPA) in maintaining a production-grade Streamlit portfolio tracker.

# Project Context: Investment Portfolio Manager

## 📋 System Instructions
You are a Financial Engineer and Python Expert. You are assisting Bill (CPA) in maintaining a production-grade Streamlit portfolio tracker.

## 🛠️ Tech Stack
Streamlit, Google Sheets (gspread), Gemini 2.5 Pro (google-genai), yfinance, FMP, Finnhub, Pydantic.

## ⚖️ Critical Guardrails (MANDATORY)
1. **No LLM Math:** Strictly calculate all yields, drift, tax savings, and projections in Python. Gemini only explains the facts.
2. **Schema Validation:** Use `response_schema` with Pydantic for ALL `ask_gemini` calls.
3. **Data Privacy:** PII must be stripped (no account numbers) before context injection.
4. **Column Guard:** Every DataFrame used in UI/Agents must pass through `utils.column_guard.ensure_display_columns`.
5. **Dry Run Pattern:** All sheet writes must be gated by `config.DRY_RUN`.

## 🏗️ Architecture: The 12-Agent Tactical Squad
The app uses a modular agent architecture in `utils/agents/`. Each agent is domain-locked:
- **Tax Intelligence:** Rebalancing & TLH.
- **Grand Strategist:** Net Worth (Liquid + Real Estate).
- **Valuation Agent:** P/E Analysis & Accumulation.
- **Price Narrator:** Movement explanations.
- **Options Agent:** Covered call strategies.
- ... plus Concentration, Macro, Earnings, Correlation, Technicals, and Cash.

## 📂 Key Files
- `app.py`: Main entry point with Global Error Boundary.
- `pipeline.py`: Robust CSV ingestion and sheet writing.
- `config.py`: Single source of truth for column maps and constants.
- `utils/column_guard.py`: Prevents KeyError crashes via self-healing Title Case.
- `utils/validators.py`: Catches corrupted CSV data.

## 🗝️ Authoritative Fingerprints
- `Holdings_History`: `import_date|ticker|quantity`
- `Daily_Snapshots`: `import_date|pos_count|total_value` (rounded)
- `Transactions`: `trade_date|ticker|action|net_amount`

## 🚀 Dev Workflow
- read lessonsLearned.md  Always trust the ingested price during the import phase.
- Use `ws.col_values()` for fast deduplication checks.
- Every commit must follow an app audit and prompt review.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Wlong34243)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Wlong34243)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
