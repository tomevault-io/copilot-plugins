## minerva

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Minerva** is a web-based trading research copilot designed to automate swing-trading research with a cost-efficient, multi-market architecture.

**Key Principles:**
- **Adaptation, not creation:** All workflow logic adapts skills from the [claude-trading-skills](reference/claude-trading-skills) repository
- **Cost-first architecture:** Python screener runs first for quantitative filtering; LLMs handle only shortlisted candidates
- **Multi-market support:** US markets (S&P 500, Nasdaq) and TASE (Tel Aviv Stock Exchange) from day one
- **Market-aware defaults:** USD for US, ILS for TASE; validates currency/precision in all outputs
- **Deterministic over agentic:** Structured validation and contracts before LLM-dependent nodes

**In Scope (v1):**
- Manual scanning and research execution
- Swing trading strategies (short-to-medium term horizons)
- Structured JSON output with explicit entry/exit rules and position sizing
- Web dashboard for candidate review and ticket management
- Single-user, unauthenticated access

**Out of Scope (v1):**
- Multi-user collaboration, broker execution, real-time streaming, scheduled scans
- Long-term investing, dividends, options trading, statistical arbitrage
- Authoring proprietary strategies independent of source skill adaptation

See [docs/MinervaPRD.md](docs/MinervaPRD.md) for full product specification.

## Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Frontend (Vercel)                       │
│                      Next.js 15+ (App Router)               │
│  ├─ Candidate Queue & Workflow Controls                     │
│  ├─ Ticket Viewer (Research Results)                        │
│  ├─ Watchlist & Portfolio Management                        │
│  └─ Market History Charts (lightweight-charts)              │
└─────────────────────────────────────────────────────────────┘
                           ↕ HTTP/REST
┌─────────────────────────────────────────────────────────────┐
│                  Backend (Railway)                          │
│                   FastAPI (Python 3.x)                      │
│  ├─ Scanner Service: symbol filtering (yfinance + pandas)   │
│  ├─ Workflow Engine: LangGraph-style orchestration         │
│  ├─ Research Client: OpenRouter API connector              │
│  ├─ Risk/Sizing: position validation & guardrails          │
│  └─ Market Data: OHLC history endpoint                     │
└─────────────────────────────────────────────────────────────┘
                           ↕ SQL/TCP
┌─────────────────────────────────────────────────────────────┐
│            Persistence Layer (Supabase PostgreSQL)          │
│  ├─ Candidates (filtered symbols, screening metadata)      │
│  ├─ Research Tickets (structured outputs, entry/exit)      │
│  ├─ Watchlist Items (user-tracked symbols)                 │
│  └─ Scan History (runs, timestamps, results cache)         │
└─────────────────────────────────────────────────────────────┘
```

### Key Component Responsibilities

**Frontend (`src/` or similar):**
- React components with App Router for UI layout
- API integration layer (request normalization, error handling)
- Chart rendering with lightweight-charts library
- Market data normalization: `time -> ts` (epoch ms), OHLC values
- Execution level overlays (support, resistance, checkpoint lines on charts)

**Backend (`backend/` or similar):**
- **Scanner:** Fetches symbols via yfinance, applies screening triggers, returns candidates
- **Workflow Engine:** Executes adapted source-skill playbooks (see "Workflow Adaptation" below)
  - Deterministic validation nodes run first
  - LLM nodes follow for shortlisted candidates only
  - Output contracts enforced (JSON validated before persistence)
- **Research Client:** OpenRouter API integration
  - Retry policy: exponential backoff for 429/5xx/timeout
  - Strict JSON response format enforcement
  - Cache deduplication to avoid repeated LLM calls
- **Risk & Sizing:** Computes share quantity, validates max-risk constraints
- **Market Data Endpoint:** `/market/history` returns OHLC candles in frontend-normalized format

### Workflow Adaptation (From claude-trading-skills)

All research workflows adapt skills from the reference repository:

1. **Source Mapping:** Identify applicable skills from `reference/claude-trading-skills/skills/`
   - For swing trading: focus on Technical Analyst, Theme Detector, Uptrend Analyzer, Breadth Chart Analyst
   - Exclude: options-strategy-advisor, kanchi-dividend-*, institutional-flow-tracker, macro-regime-detector
2. **Prompt Porting:** Extract SKILL.md instructions and references, adapt to OpenRouter API format
3. **Guardrails:** Add deterministic validation before LLM invocation (e.g., is symbol valid? is price in expected range?)
4. **Output Schema:** Define JSON contract for each workflow step (validated before database commit)
5. **Market Localization:** For US vs TASE branching, include market-specific defaults (USD/ILS, trading hours, sector mapping)

### Technology Stack

| Component | Technology | Notes |
|-----------|-----------|-------|
| **Frontend Hosting** | Vercel | Git-triggered deployments, preview URLs |
| **Frontend Framework** | Next.js 15+ (App Router) | React 18+, TypeScript recommended |
| **Frontend Charts** | lightweight-charts | OHLC rendering, pan/zoom/fullscreen support |
| **Backend Hosting** | Railway | Managed Python runtime, PostgreSQL connection pooling |
| **Backend Framework** | FastAPI | Python 3.9+, async/await ready |
| **Data Fetching** | yfinance, pandas | Screener and market data ingestion |
| **LLM Integration** | OpenRouter API | Multi-model support, retry middleware |
| **Database** | Supabase PostgreSQL | Row-level security, vector extensions optional |
| **Workflow Orchestration** | LangGraph-compatible design | Deterministic state graph execution |

## Development Setup

### Prerequisites

- **Node.js** 18+ (for Next.js frontend)
- **Python** 3.9+ (for FastAPI backend)
- **Git** (for cloning the reference repository)
- **Environment Variables:**
  - `OPENROUTER_API_KEY`: OpenRouter API token
  - `RESEARCH_MODEL`: Model ID (e.g., `openai/gpt-4-turbo`)
  - `RESEARCH_OPENROUTER_RETRY_COUNT`: Number of retries (default: 3)
  - `RESEARCH_OPENROUTER_BACKOFF_SECONDS`: Backoff interval (default: 2)
  - `SUPABASE_URL`, `SUPABASE_KEY`: Database connection
  - `DATABASE_URL`: PostgreSQL connection string for backend
  - Additional Vercel, Railway, and Supabase secrets as per deployment config

### Project Structure (Target Layout)

```
Minerva/
├── CLAUDE.md                  # This file
├── docs/MinervaPRD.md              # Product specification
├── reference/                 # Source material (read-only)
│   └── claude-trading-skills/
│
├── frontend/                  # Next.js application
│   ├── src/
│   │   ├── app/               # App Router pages/layouts
│   │   ├── components/        # React components
│   │   ├── lib/               # API client, utilities
│   │   └── ...
│   ├── package.json
│   ├── tsconfig.json
│   ├── next.config.js
│   └── ...
│
├── backend/                   # FastAPI application
│   ├── app/
│   │   ├── main.py            # FastAPI app initialization
│   │   ├── routers/           # API route handlers (scanner, research, market)
│   │   ├── services/          # Core services (workflow, openrouter, risk)
│   │   ├── models/            # Pydantic schemas and database models
│   │   ├── config.py          # Configuration and environment
│   │   └── utils/             # Helpers (validation, formatting, market utils)
│   ├── tests/                 # Unit/integration tests
│   ├── scripts/               # Utility scripts for dev/ops
│   ├── requirements.txt
│   ├── pyproject.toml
│   └── ...
│
└── docs/                      # Optional: deployment guides, API specs
    ├── ARCHITECTURE.md
    ├── API.md
    └── ...
```

### Frontend Setup

```bash
cd frontend
npm install
npm run dev          # Start dev server (http://localhost:3000)
npm run build        # Production build
npm run lint         # ESLint check
npm run format       # Prettier formatting
npm test             # Run tests (jest or vitest)
```

### Backend Setup

```bash
cd backend

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
pip install -e ".[dev]"   # With dev dependencies

# Run development server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Run tests
pytest

# Run linting
ruff check .

# Format code
ruff format .
```

### Database Setup

1. Create Supabase project
2. Initialize schema using migration scripts (TBD)
3. Set `SUPABASE_URL`, `SUPABASE_KEY` environment variables
4. Ensure connection pooling is configured for Railway backend

## Key Development Patterns

### Workflow Definition

When adapting a skill into a backend workflow:

1. **Create workflow state schema** (Pydantic model):
   - Input: candidate symbol, market, context
   - Intermediate: analysis stages, LLM responses
   - Output: structured ticket (entry, exit, risk metrics)

2. **Define nodes** (deterministic first, LLM last):
   ```
   Input Validation → Data Fetch → Deterministic Filters → LLM Analysis → Output Validation → Persist
   ```

3. **Load skill references** only when needed (minimize token usage)

4. **Enforce output contracts** before any database write

### Output Schema Example (Research Ticket)

```json
{
  "id": "uuid",
  "symbol": "AAPL",
  "market": "US",
  "created_at": "2026-03-19T10:00:00Z",
  "workflow_type": "technical-swing",
  "analysis": {
    "entry_price": 190.50,
    "entry_rationale": "...",
    "stop_loss": 185.00,
    "target": 200.00,
    "position_size": 100,
    "max_risk_usd": 550.00,
    "bullish_probability": 0.72,
    "key_triggers": ["breakout above resistance", "volume confirmation"],
    "caveats": []
  },
  "source_skill": "technical-analyst",
  "research_model": "openai/gpt-4-turbo",
  "status": "approved"
}
```

### API Response Normalization

Frontend market data endpoint (`/market/history`) normalizes OHLC:

```python
# Backend response format
{
  "symbol": "AAPL",
  "market": "US",
  "candles": [
    {"ts": 1710867600000, "open": 190.2, "high": 191.5, "low": 190.0, "close": 191.0},
    ...
  ],
  "execution_levels": {
    "entry": 190.5,
    "stop": 185.0,
    "target": 200.0,
    "checkpoint": null
  }
}
```

### Testing Strategy

- **Unit tests** for validator functions, market data parsers, risk calculators
- **Integration tests** for full workflow execution (mock OpenRouter API)
- **E2E tests** for critical paths (scan → research → ticket creation)
- Do not test external API calls directly; mock with fixtures

## Common Development Tasks

### Adding a New Research Workflow

1. Identify source skill from `reference/claude-trading-skills/skills/<skill-name>/`
2. Create workflow state schema in `backend/app/models/workflows/`
3. Create workflow handler in `backend/app/services/workflows/`
4. Add endpoint to `backend/app/routers/research.py`
5. Add integration test covering happy path + error cases
6. Update frontend to expose workflow in UI dropdown

### Debugging LLM Responses

- Check `RESEARCH_MODEL` environment variable matches intended model
- Enable debug logging in OpenRouter client to see raw request/response
- Cache responses: check if duplicate prompts are being sent (check deduplication logic)
- Validate response against Pydantic schema; log validation errors before rejection

### Testing Screener Logic

1. Run scanner with a small test set (e.g., top 10 S&P 500 symbols)
2. Log candidate counts and exclusion reasons
3. Verify market-specific filters (US vs TASE) are applied correctly
4. Check that invalid symbols are gracefully skipped

## Deployment

### Frontend Deployment (Vercel)

- Automatic deployments from `main` branch
- Preview deployments for all PRs
- Configure build settings in `vercel.json`:
  ```json
  {
    "buildCommand": "cd frontend && npm run build",
    "outputDirectory": "frontend/.next"
  }
  ```
- Set environment variables in Vercel dashboard (`NEXT_PUBLIC_API_URL`, etc.)

### Backend Deployment (Railway)

- Connect GitHub repository
- Railway auto-detects Python + FastAPI
- Configure environment variables in Railway dashboard
- Database connection: Railway PostgreSQL addon
- Domain: Railway auto-assigns; configure custom domain if needed

### Environment Parity

- Ensure `.env.local` matches deployed environment variables
- Use `.env.example` to document required variables (never commit actual secrets)
- Supabase keys: staging/prod separation recommended for v1+

## Important Guardrails

### Market Validation

- **US symbols:** Validate against S&P 500 / Nasdaq ticker list
- **TASE symbols:** Validate against TASE ticker format (Hebrew letters or common acronyms)
- Reject invalid symbols before screening starts
- Log rejection reasons for debugging

### Risk Management

- Position size calculation: use consistent formula (% of account, Kelly criterion, or fixed risk)
- Max risk validation: reject trades exceeding risk threshold
- Stop loss: always required; target: always required
- Cash/margin impact: surface explicitly in UI

### Workflow Determinism

- No LLM calls during data fetching or validation
- Reproducible screening: same inputs → same candidate list
- Cached outputs: tag with model version and generation timestamp
- Deduplication: check if same symbol + market + analysis type was recently run

### Market Hours & Data Freshness

- Document trading hours for US (9:30-16:00 ET) and TASE (9:15-17:00 IL)
- Warn if market data is more than N hours stale
- Frontend displays last-updated timestamp on charts

## References & Resources

- **Product Spec:** [docs/MinervaPRD.md](docs/MinervaPRD.md)
- **Source Skills:** [reference/claude-trading-skills](reference/claude-trading-skills) – read CLAUDE.md and skill-specific SKILL.md files
- **Frontend Framework:** [Next.js Docs](https://nextjs.org/docs)
- **Backend Framework:** [FastAPI Docs](https://fastapi.tiangolo.com)
- **Database:** [Supabase Docs](https://supabase.com/docs)
- **Charts:** [lightweight-charts API](https://tradingview.github.io/lightweight-charts/)
- **LLM API:** [OpenRouter Documentation](https://openrouter.ai/docs)

## Claude Behavior Guidelines

### Core Principles
- **No Fluff:** Skip pleasantries ("Sure!", "I can help with that"). Go straight to the solution.
- **Thinking Process:** For complex logic, open with a brief `<thinking>` block — analyze edge cases, bugs, and trade-offs before writing code.
- **Code Quality:** Clean Code, DRY, SOLID. Python 3.10+ syntax, type hints everywhere, robust error handling and logging by default.
- **Modularity:** Modular, reusable functions over monolithic blocks.

### Communication Style
- **Conciseness:** Bullet points and bold for key terms. No long paragraphs.
- **Directness:** If a request is ambiguous or the stack is suboptimal, say so and suggest better.
- **No Over-Explaining:** Assume high technical proficiency. Skip basics unless asked.

### Technical Preferences
- **Architecture:** Repository pattern, Hooks, Services — scalable and maintainable.
- **Security:** OWASP Top 10 by default — input validation, sanitization at system boundaries.
- **Performance:** Consider runtime complexity and memory usage.

### Output Format
- Structure long responses with `##` / `###` headings.
- Provide complete, copy-pasteable code blocks with file names.
- Separate distinct answer sections with `---`.

---

## Available Claude Skills & Plugins

### Skills (`.claude/skills/`) — invoke via Skill tool
- **senior-backend** — FastAPI, async Python, DB optimization, API design
- **senior-frontend** — Next.js App Router, React, TypeScript, Tailwind
- **vercel-react-best-practices** — 80+ React performance and pattern rules
- **skill-creator** — meta-skill for building new skills

### Active Plugins — use proactively when relevant
- **frontend-design** — production-grade UI components with high design quality
- **context7** — up-to-date library docs (use for Next.js, FastAPI, Supabase, etc.)
- **code-simplifier** — refactor/simplify recently written code
- **code-review** — review PRs or changed code
- **playwright** — browser automation for E2E testing
- **claude-md-management** — audit and update CLAUDE.md files

**Rule:** Prefer a skill or plugin over raw generation when the task clearly maps to one.

---

## Notes for Future Instances

- If the project structure deviates from the target layout, update the "Project Structure" section above
- When adapting new skills, always verify they are **swing-trading focused** (not dividends, options, long-term, or statistical arb)
- Keep market-aware defaults (USD/ILS, trading hours) consistent across all workflow nodes
- Prefer deterministic validation early and LLM calls late in the pipeline

### Codebase State (updated 2026-03-22)

- **mean-reversion-bounce workflow added (2026-03-22):** second workflow alongside `technical-swing`. Files: `workflows/mean_reversion.py`, `prompts_mean_reversion.py`. Pre-screen gate in `pre_screen.py` (`pre_screen_mean_reversion()`). New indicators in `indicators.py` (`compute_mean_reversion_indicators()`): Bollinger Bands (20,2), Capitulation Volume detector, RSI Divergence.
- **Workflow auto-detection in scanner (2026-03-22):** Scanner now fetches `period="1y"` (was `"1mo"`) and runs both pre-screen gates per candidate. Results stored in `candidates.metadata.applicable_workflows[]`. Frontend candidate card shows `Swing`/`MR` badges. `ResearchModal` shows a strategy picker when both workflows qualify.
- **WORKFLOW_REGISTRY in research.py:** dispatcher maps `workflow_type` string → executor function. Adding a new workflow = one dict entry.
- **SynthesizedScoreTable is now dynamic:** reads all dimension keys from the JSON object; works for both swing (6 Minervini dimensions) and MR (6 MR dimensions) without any hardcoded key list.
- **Vercel Analytics added:** `@vercel/analytics` installed; `<Analytics />` mounted in `frontend/src/app/layout.tsx`.
- **Phase 7 complete:** rich analytical depth — RS scoring, multi-target scale-out, 4-scenario planning, Synthesized Setup Score, Final Recommendation, execution checklist
- **Phase 6 complete:** persistence & data integrity — output validation (14 tests), 24h dedup, scan history panel, stale data warnings, updated_at trigger
- **Phase 5 complete:** full frontend UI — watchlist, candidates, research execute flow, ticket detail, dashboard
- **Phase 3 complete:** workflow engine lives in `backend/app/services/workflows/swing_trade.py`
- **DB migrations applied:** 001–005; 005 adds `rs_rank_pct FLOAT`, `setup_score SMALLINT`, `verdict TEXT`, `entry_type TEXT` flat columns to `research_tickets`
- **research_tickets** has `metadata`, `key_triggers`, `updated_at`, `rs_rank_pct`, `setup_score`, `verdict`, `entry_type` — always include all in inserts
- **Workflow nodes** (in order): fetch_data → **fetch_rs** → pre_screen → fetch_breadth → llm_research → compute_sizing → persist_ticket
- **Agorot fix (2026-03-19):** TASE prices from yfinance are in agorot (1/100 ILS). Fixed in `_node_fetch_data()` (divides df OHLC ÷ 100) and `market.py` (_normalize_candles + _fetch_quote_sync). All TASE price display and research is now in correct ILS units.
- **Relative screening refactor (2026-03-19):** Scanner now uses RVOL (relative volume = today / 20d avg) and ATR% (ATR-14 / price × 100) instead of absolute volume/volatility. Pre-screen liquidity gate uses RVOL from indicators (fallback: absolute avg_vol floor). `indicators.py` now emits `rvol` and `atr_pct`. Score is market-agnostic: `min(rvol, 3) × 10 + atr_pct × 10`. Also fixed a score bug where `df["Close"]` was used instead of `df["Volume"]` in the old score fn.
- **Workflow docs:** See `docs/WORKFLOWS.md` — full node reference, known issues, improvement roadmap, prompt engineering notes
- **Pre-screen gate** returns 422 with structured JSON on failure; `force=true` param bypasses it; `force_refresh=true` bypasses 24h dedup
- **Market breadth** (Monty's CSV) is US-only; TASE always returns a neutral stub — expected
- **Theme system:** `ThemeProvider` context + CSS variables, dark default + `.light` class override; "Assistant" + "JetBrains Mono" Google Fonts; base font 16px
- **Color palette:** accent is teal `#57c1d5` (dark) / `#1a8fa6` (light) — NOT amber; no hardcoded `#f59e0b` anywhere
- **Quote endpoint:** `POST /market/quotes` (batch, async parallel via `asyncio.gather`) returns price + change_pct
- **Single-symbol scan:** `POST /scanner/scan` accepts optional `symbols: string[]` to bypass watchlist; `.TA` suffix stripped + market auto-inferred
- **Candidates aggregation:** `GET /scanner/candidates` deduplicates in Python across ALL scan runs — single-symbol scans don't wipe watchlist candidates
- **Research model:** `deepseek/deepseek-chat` — non-reasoning, supports `response_format: json_object`; stepfun free model was dropped (reasoning burns all tokens, returns null content)
- **apiClient interceptor:** wraps network failures with `ApiError(0, "Backend not reachable...")`
- **Ticket layout (Phase 7):** header (symbol + VerdictBadge + SetupScoreBadge + RS rank) → chart → stats row → Synthesized Score table → Scale-out Plan → Scenarios → Final Recommendation + checklist → Key Triggers/Caveats → context sections → Pre-screen accordion
- **RS calculator:** `backend/app/services/rs_calculator.py` — computes excess return vs SPY (US) or ^TA125.TA (TASE) for 63/126/189d periods; RS composite = 0.40×63d + 0.35×126d + 0.25×189d; percentile rank within watchlist universe (capped 30 symbols, 10s timeout)
- **New LLM JSON fields:** `technical_analysis`, `scale_out_targets` (T1/T2/T3), `scenarios` (4 items summing to 1.0), `synthesized_score` (6 dimensions × 0-10, total/60), `execution_checklist`, `final_recommendation` (verdict/action/conviction/narrative)
- **max_tokens bumped 2500→3500** for LLM research call (richer schema needs more output budget)
- **vol_dry_up_ratio:** `indicators.py` now emits `vol_dry_up_ratio` (10d avg / 50d avg) as quantitative compression metric alongside bool `vol_dry_up`
- **Port gotcha:** Windows Hyper-V reserves port 8000 range. Run backend on `--port 8001`; set `NEXT_PUBLIC_API_URL=http://localhost:8001` in `frontend/.env.local`

### Known Gotchas

- **Scanner period change:** Scanner now fetches `period="1y"` for workflow classification. For large watchlists (20+ symbols) this increases scan time ~2-3×. Acceptable — scans are background operations.
- **MR entry_type "buy_stop":** The MR workflow uses `"buy_stop"` as an entry type internally. The DB column constraint only accepts `"current"` or `"breakout"`, so `buy_stop` is mapped to `"breakout"` at persist time. The original value is preserved in `metadata.technical_analysis.entry_type`.
- **MR synthesized_score keys differ from swing:** Swing uses `trend_template/vcp_pattern/...`; MR uses `long_term_trend/dip_depth_quality/...`. `SynthesizedScoreTable` reads keys dynamically — both render correctly.
- **TASE yfinance symbols:** Always append `.TA` suffix (e.g., `OPCE` → `OPCE.TA`). Handled in scanner and workflow automatically.
- **langgraph versions:** Use `langgraph>=1.0.0` + `langchain-core>=0.3.0`. Old pinned versions (0.2.x) cause dependency conflicts.
- **Windows Unicode in scripts:** Add `sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')` when printing non-ASCII (e.g., ₪, ✓).
- **Supabase MCP:** Approved via `enabledMcpjsonServers: ["supabase"]` in `.claude/settings.json`. Use `mcp__supabase__apply_migration` for DDL, `mcp__supabase__execute_sql` for queries.
- **Dev Python env:** Python 3.11 global install (no activated venv in dev shell). `pip install` targets the global interpreter.
- **lightweight-charts SSR:** Always use `dynamic(() => import(...), { ssr: false })` for `CandlestickChart` — it uses browser DOM APIs.
- **OpenRouter free models (stepfun etc.):** Reasoning models burn all tokens on internal thinking → `content=null`. Use non-reasoning models like `deepseek/deepseek-chat` that support `json_object` mode.
- **Symbol normalization:** `.TA` suffix stripped in `POST /scanner/scan` before storage. Market auto-inferred as TASE when suffix present. Symbols stored without suffix.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tura2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Tura2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
