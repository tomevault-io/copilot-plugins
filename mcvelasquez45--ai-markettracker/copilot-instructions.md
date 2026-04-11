## ai-markettracker

> > **Purpose.** This document gives Codex CLI and human contributors a **single source of truth** for how to build and evolve **AI-MarketTracker** without code drift. It defines non-negotiables, coding standards, data contracts, the Zero-DTE options strategy, and acceptance checks the agent must pass before making changes.

# AGENTS.md — AI-MarketTracker (Codex CLI Rules & Context)

> **Purpose.** This document gives Codex CLI and human contributors a **single source of truth** for how to build and evolve **AI-MarketTracker** without code drift. It defines non-negotiables, coding standards, data contracts, the Zero-DTE options strategy, and acceptance checks the agent must pass before making changes.

---

## 0) Project Charter (Do Not Drift)

- **Goal:** A hedge-fund-grade trading assistant powered by **open-source LLMs**, **MCP servers**, and **RAG** over finance knowledge.
- **Sources of truth:**  
  - **Polygon.io** for stocks, options, indices (SPX), greeks, IV, snapshots (via **Polygon MCP** + REST fallback).  [oai_citation:0‡GitHub](https://github.com/polygon-io/mcp_polygon?utm_source=chatgpt.com)  
  - **Congressional activity** (CapitolTrades) for issuer↔ticker signals.  
  - **Macro calendar** (Fed/FOMC, CPI/PCE) to detect high-vol events.  
- **Memory & RAG:** **MongoDB Atlas Vector Search** embedded alongside OLTP data.  [oai_citation:1‡MongoDB](https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-overview/?utm_source=chatgpt.com)
- **LLM serving:** **vLLM** (OpenAI-compatible server) or **TGI** behind an internal router.  [oai_citation:2‡VLLM Documentation](https://docs.vllm.ai/en/latest/serving/openai_compatible_server.html?utm_source=chatgpt.com)
- **Agent runtime:** **Codex CLI** operating in repo root with this file as authoritative context.  [oai_citation:3‡OpenAI Developers](https://developers.openai.com/codex/cli/?utm_source=chatgpt.com)

---

## 1) Non-Negotiables

- **Do not alter** existing **trade evaluation logic**, **GPT prompt format**, or **MongoDB field structures** without explicit approval.
- All new API contracts must use **TypeScript types or Zod schemas** and be **backward-compatible**.
- **Security:** no secrets in code; use env vars and local secret managers.
- **Reproducibility:** every code change must come with docs updates (README, this file, or `/docs`).

---

## 2) Tech & Environment

- **Frontend:** React + Vite + TypeScript
- **Backend:** Node.js + Express + TypeScript
- **Data:** MongoDB (OLTP) + **Atlas Vector Search** for embeddings  [oai_citation:4‡MongoDB](https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-overview/?utm_source=chatgpt.com)
- **LLM:** vLLM OpenAI-compatible endpoint (or TGI)  [oai_citation:5‡VLLM Documentation](https://docs.vllm.ai/en/latest/serving/openai_compatible_server.html?utm_source=chatgpt.com)
- **Agent:** Codex CLI (local)  [oai_citation:6‡OpenAI Developers](https://developers.openai.com/codex/cli/?utm_source=chatgpt.com)
- **MCP:** Prefer **Polygon MCP server** with our thin adapter for normalization/caching; REST fallback for gaps.  [oai_citation:7‡GitHub](https://github.com/polygon-io/mcp_polygon?utm_source=chatgpt.com)

---

## 3) Directory Contracts

> Keep this shape. Additions are allowed; moving/removing top-level folders requires approval.

AI-MarketTracker/
├─ client/                     # React + Vite + TS (SSE/WebSocket for streams)
│  ├─ src/
│  │  ├─ pages/ (Dashboard, Ideas, History, Settings)
│  │  ├─ features/ (trade, auth)
│  │  ├─ services/ (axios client, SSE, typed APIs)
│  │  ├─ components/ (ui, charts, layout)
│  │  ├─ contexts/ (AuthContext, ThemeContext)
│  │  ├─ types/ (DTOs)
│  │  └─ utils/ (formatters, guards)
│  └─ .env.example
├─ server/                     # Node/Express + TS
│  ├─ src/
│  │  ├─ routes/ (trades, features, health, sse)
│  │  ├─ controllers/ (preserve evaluation logic)
│  │  ├─ services/
│  │  │  ├─ orchestrator/
│  │  │  ├─ risk-engine/
│  │  │  ├─ evaluator/ (ex-ante, ex-post)
│  │  │  ├─ rag/ (retriever, indexer, corpus)
│  │  │  ├─ llm-router/ (prompts, model selection)
│  │  │  ├─ mcp-clients/ (polygon, congress, news, fed)
│  │  │  ├─ feature-store/ (RSI, MACD, VWAP)
│  │  │  └─ cache/ (Redis)
│  │  ├─ models/ (Mongo schemas)
│  │  ├─ workers/ (feature-ingest, outcome-cron, rag-indexer, finetune)
│  │  ├─ config/ (env, mongo, redis, otel)
│  │  └─ types/ (shared DTOs)
│  └─ .env.example
├─ docs/                       # Keep lightweight until endpoints stabilize
│  └─ (optional future) ARCHITECTURE.md
└─ AGENTS.md                   # THIS FILE (rules & context for Codex)

---

## 4) Strategy Playbook — **Zero-DTE Options Selling (SPX)**

> The assistant must follow these steps and **explain rationale** in outputs.  
> Core principle: sell **zero-DTE OTM** options with **~20→12 delta** targeting high probability of expiring worthless (≈70–80%). *(This is a heuristic, not a guarantee; always enforce risk caps.)*

### Step A — **Macro/Event Check**
1) Query today’s macro calendar for **Fed speeches, FOMC, CPI/PCE** or other market-moving events.  
2) If **no major events**, proceed; if **major events**, expect higher IV & widen buffers (farther OTM, smaller size), or skip.  
*Why:* Event-driven volatility elevates tail risk.

### Step B — **Market Context (“Information Radar”)**
- **Market-cap leaders:** MSFT, AAPL, NVDA, GOOGL, AMZN, META, TSLA → gauge market beta.
- **Risk-on ETFs:** XLK, XLF, XLY, XLI, XLB, SMH, ARKK → breadth/risk appetite.
- **Defensive ETFs:** XLV, XLP, XLU, XLRE, XLC → leadership implies caution.
- **Internals:** A/D line, RSI, momentum gauges → confirm bias (bullish / bearish / neutral).

### Step C — **SPX Range Framing (Trade Window ~ last 2 hours)**
- Time window guideline: **1:30–2:00 PM ET** to close.  
- On **15-min SPX**: mark **day high/low**, compute **midpoint**, identify **support/resistance**; include Quantum VPOC if available.  
- **Strike selection:** choose OTM options with **delta 20→12**.  
  - **Bullish/neutral:** sell **put credit spread** (short put + long lower put).  
  - **Bearish:** sell **call credit spread** (short call + long higher call).  
- **Premium targeting:** if late-day premium is light, adjust **contract count** (respect risk caps).

### Step D — **Monitor to Expiry**
- Keep a **news feed** open; track **tick speed** and **1m/5m/15m** charts.  
- Watch **RSI/TRIX**, A/D line, and **delta drift** on your strikes.  
- **Risk rule:** if price approaches strikes or breaks key levels, reduce risk or exit.  
- If safe into close, let expire to collect credit (avoid extra day trades).

> **Mapping to Data Inputs (must be wired by the agent):**
- **SPX index data:** via **Indices API** (`I:SPX`) for levels & aggregates.  [oai_citation:8‡Polygon](https://polygon.io/docs/rest/indices/overview?utm_source=chatgpt.com)  
- **SPX options chain:** via Options API; confirm **options ticker format** (YYMMDD + C/P + strike).  [oai_citation:9‡Polygon](https://polygon.io/docs/rest/options/contracts/contract-overview?utm_source=chatgpt.com)  
- **Greeks/IV & snapshots:** via Polygon options endpoints/MCP tools.  [oai_citation:10‡Polygon](https://polygon.io/docs/rest/options/contracts/contract-overview?utm_source=chatgpt.com)  
- **Macro schedule:** Fed/FOMC/CPI/PCE via our macro MCP (data sources vary; document when locked).  
- **ETF/Leaders quotes:** via Polygon equities endpoints or snapshots.

---

## 5) Data Contracts & Schemas

### 5.1 Recommendation Output (JSON)
The LLM must emit this exact shape (stringified JSON), validated in the backend:

```json
{
  "ticker": "SPX",
  "strategy": "zero_dte_credit_spread",
  "bias": "bullish | bearish | neutral",
  "window": { "startET": "13:30", "endET": "16:00" },
  "spread": {
    "type": "put_credit | call_credit",
    "short_leg": { "option": "O:SPX<YYMMDD><C|P><strike*1000>", "delta": 0.18 },
    "long_leg":  { "option": "O:SPX<YYMMDD><C|P><strike*1000>", "delta": 0.10 },
    "width": 5
  },
  "sizing": { "contracts": 2, "per_contract_credit": 0.65, "est_max_loss": 435.00 },
  "risk": { "stop_rule": "breach_midpoint_or_SR", "event_risk": "none|elevated" },
  "context": {
    "leaders": { "green": ["MSFT","NVDA"], "red": ["AAPL"] },
    "risk_on_etfs": { "green_count": 5, "tickers": ["XLK","SMH","XLF"] },
    "defensive_etfs": { "leading": false },
    "internals": { "ad_line": "positive", "rsi_15m": 54 }
  },
  "notes": "Late-day theta favorable; choose 20→12 delta OTM; widen if IV high."
}

Options symbol format reference (underlying + YYMMDD + C/P + strike×1000).  ￼
```

### 5.2 Retrieval Query (RAG)
- Use MongoDB Atlas Vector Search; hybrid filter by { ticker: "SPX" | "SPY", doc_type, date_range }.  ￼
- Top-k 8–20; apply recency bias.

⸻

## 6) MCP Usage Policy
- Default to the Polygon MCP server for tool calls. It exposes Polygon APIs as MCP tools and is actively maintained/experimental.  ￼
- If a needed endpoint isn’t exposed, call Polygon REST directly through our adapter.
- Normalize all responses to our DTOs before passing to RAG/LLM.

⸻

## 7) LLM Serving Rules
- Prefer vLLM OpenAI-compatible server (drop-in for Chat API). Configure model, max tokens, temperature, and enable continuous batching for throughput.  ￼
- Use two paths:
  - Fast path: short prompt for quotes/what-ifs.
  - Deep path: full RAG context + strategy rubric above.
- Always produce valid JSON per §5.1; no extra prose in the JSON block.

⸻

## 8) Risk & Compliance Guardrails
- Capital check: ensure credit spread cost/risk ≤ user capital slice.
- Event risk: widen OTM or reduce size on macro days (from Step A).
- Liquidity: minimum open interest & spread width sanity checks.
- Stop/Exit rules: codify breach of midpoint or key S/R.

⸻

## 9) Logging, Testing, Git
- Logs: Pino JSON w/ ISO timestamp + requestId; log input params, chosen strikes, deltas, credits, risk flags.
- Tests:
  - Unit: DTO validators, deltas-in-range, JSON schema.
  - Integration: MCP adapter happy/edge paths; RAG retrieval filters.
- Commits: Conventional Commits (feat:, fix:, docs:).
- PRs: include screenshots or sample JSON outputs.

⸻

## 10) Codex CLI — Task Templates

Run Codex in repo root. If Codex proposes changes that violate §1 non-negotiables, reject and re-plan.  ￼

Task: Wire Polygon MCP + REST fallback

```
/task Create server/src/services/mcp-clients/polygon with client.ts calling Polygon MCP tools, restFallback.ts for missing endpoints, and adapter.ts that normalizes to our DTOs. Add unit tests for both paths.
```

Task: Implement SPX chain fetch + delta filter

```
/task Add orchestrator step to fetch SPX options chain, compute deltas, select OTM 20→12, and return candidate spreads with credits. Include min OI and max spread width checks.
```

Task: RAG retriever

```
/task Implement server/src/services/rag/retriever.ts using Atlas Vector Search with filters {ticker, doc_type, date_range}, topK=12, recency bias. Add tests using fixtures.
```

Task: LLM deep path

```
/task Add llm-router deep path that builds system+context prompt and enforces JSON schema from §5.1. Use vLLM OpenAI-compatible endpoint.
```

⸻

## 11) MVP Checklist (Agent must update status in PRs)
- Polygon MCP adapter + REST fallback wired (SPX, options, greeks)  ￼
- SPX range framing + delta 20→12 spread selector
- Macro event gate (Fed/FOMC/CPI/PCE)
- Risk engine: affordability + size + guardrails
- RAG retriever (Atlas Vector Search) + embeddings job  ￼
- vLLM deep path with strict JSON output  ￼
- Evaluator (ex-post) to label outcomes and feed fine-tune sets
- Frontend: Ideas page (SSE), History, Dashboard (leaders/ETFs/internals)

⸻

## 12) Quick Reference (Docs & Formats)
- Polygon MCP (official GitHub) — capabilities & tool list.  ￼
- Polygon indices & tickers (I:SPX) — SPX levels & aggregates.  ￼
- Polygon options contracts — contract overview; symbol format guide.  ￼
- Atlas Vector Search — overview and $vectorSearch aggregation.  ￼
- vLLM OpenAI-Compatible Server — serving models behind Chat API.  ￼
- Codex CLI (local agent) — usage, AGENTS.md integration.  ￼
- Mermaid in GitHub (optional future docs) — render diagrams in Markdown.  ￼

⸻

### Appendix: Sample Queries
- All indices tickers (includes SPX):

```
GET /v3/reference/tickers?market=indices&active=true&limit=1000&apiKey=...  (Polygon)  ￼
```

- Options contract detail (by symbol):

```
GET /v3/reference/options/contracts/{options_ticker}  (Polygon)  ￼
```

Keep this file updated as endpoints mature. End of AGENTS.md.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MCVelasquez45)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MCVelasquez45)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
