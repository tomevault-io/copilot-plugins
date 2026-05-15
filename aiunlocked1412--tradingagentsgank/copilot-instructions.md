## tradingagentsgank

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common commands

### Install / run

**Preferred — `./run.sh` launcher** (handles venv bootstrap, port collisions, and HTTP health-check automatically):
```bash
./run.sh              # start web UI at 127.0.0.1:8787 (default)
./run.sh web          # explicit
./run.sh cli          # interactive Typer/Rich CLI
./run.sh script       # one-shot main.py
./run.sh stop         # stop the running web server
./run.sh logs         # tail web log
./run.sh status       # check if web is running
HOST=0.0.0.0 PORT=9000 ./run.sh web   # override host/port
```

`run.sh` writes the PID to `.run/web.pid` and logs to `.run/web.log`. It auto-creates `.venv` (prefers `uv`, falls back to `python3 -m venv`) and `pip install -e .` if missing — so a fresh clone runs with a single command.

**Manual / raw entry points:**
```bash
pip install .                        # install the package + console scripts
.venv/bin/python main.py             # one-shot scripted run (NVDA via OpenRouter, see file)
.venv/bin/python -m cli.main         # interactive CLI from source
.venv/bin/python -m tradingagents.webui.server --host 127.0.0.1 --port 8787
tradingagents                        # console script (only after `pip install`)
tradingagents-web                    # console script (only after `pip install`)
docker compose run --rm tradingagents
```

Note: the project uses a `.venv` at the repo root (Python 3.11 via `uv`). The `tradingagents` / `tradingagents-web` console scripts only exist **after** `pip install` — when working from source without install, use the `python -m ...` forms above (or just `./run.sh`).

CLI flags worth knowing (see `cli/main.py`):
- `tradingagents analyze --checkpoint` — opt into LangGraph SQLite checkpoint resume.
- `tradingagents analyze --clear-checkpoints` — wipe checkpoints before running.

### Tests
```bash
pytest                                          # full suite; markers configured in pyproject.toml
pytest -m unit                                  # fast unit-only
pytest -m smoke                                 # quick sanity tests
pytest -m "not integration"                     # skip tests that hit real services
pytest tests/test_memory_log.py                 # single file
pytest tests/test_memory_log.py::TestParseEntry # single class
pytest tests/test_memory_log.py -k "rotation"   # filter by name substring
```

`tests/conftest.py` autouses a fixture that injects placeholder API keys for every supported provider, so the suite runs without credentials. There is no separate lint config — `pyproject.toml` only configures `pytest`.

### Diagnostics
```bash
# Verify a provider's structured-output path end-to-end (Research Manager → Trader → Portfolio Manager → SignalProcessor)
OPENAI_API_KEY=... python scripts/smoke_structured_output.py openai
GOOGLE_API_KEY=... python scripts/smoke_structured_output.py google
```

## Architecture

TradingAgents is a **LangGraph state machine** that walks a trade through a fixed pipeline of LLM-powered roles. The orchestrator is `tradingagents/graph/trading_graph.py`; the graph topology is built in `tradingagents/graph/setup.py`.

### Pipeline flow (one `propagate(ticker, date)` call)
```
START
  → [selected analysts in sequence: Market → Social → News → Fundamentals]
       each analyst: agent → optional ToolNode → loop until done → Msg Clear
  → Bull Researcher ⇄ Bear Researcher (debate, max_debate_rounds)
  → Research Manager           (structured output: ResearchPlan)
  → Trader                     (structured output: TraderProposal)
  → Aggressive ⇄ Conservative ⇄ Neutral (risk debate, max_risk_discuss_rounds)
  → Portfolio Manager          (structured output: PortfolioRating)
  → END → SignalProcessor extracts rating from rendered markdown (no LLM call)
```

`AgentState` (`tradingagents/agents/utils/agent_states.py`) is a `MessagesState` subclass that carries all per-section reports plus two nested `TypedDict`s (`InvestDebateState`, `RiskDebateState`). Every node returns a partial state update.

### LLM provider abstraction
`tradingagents/llm_clients/factory.py` returns a `BaseLLMClient` instance based on `config["llm_provider"]`. Provider modules are imported lazily so test collection never pulls in heavy SDKs.

- OpenAI-compatible providers (`openai`, `xai`, `deepseek`, `qwen`, `glm`, `ollama`, `openrouter`) → `openai_client.py`.
- `anthropic`, `google`, `azure` have dedicated clients.
- `claude_cli` / `codex_cli` shell out to the locally installed CLI via `cli_client.py` and reuse that CLI's existing auth — no API key needed.

`tradingagents/llm_clients/model_catalog.py` is the single source of truth for which models the CLI offers per provider, in `quick` and `deep` modes. The framework runs two LLMs: `quick_think_llm` (analysts, debaters, trader) and `deep_think_llm` (research manager, portfolio manager).

`config["backend_url"]` defaults to **`None`** so each provider falls back to its own native endpoint; setting it to a provider-specific URL (e.g. OpenAI's) leaks into other providers and produces malformed requests. CLI flow overrides this per provider; Python users must override per provider too.

### Structured-output decision agents
Research Manager, Trader, and Portfolio Manager use `tradingagents/agents/utils/structured.py`:

1. `bind_structured(llm, Schema, name)` wraps the LLM with `with_structured_output(Schema)`. Returns `None` (with a warning) if the provider doesn't support it.
2. `invoke_structured_or_freetext(...)` runs the structured call and renders the typed Pydantic instance to markdown via the schema's render helper. **Any failure** (malformed JSON, transient provider error) falls back to `plain_llm.invoke()` so the pipeline never blocks.

Schemas and renderers live in `tradingagents/agents/schemas.py`. The render helpers preserve the exact markdown headers downstream consumers grep for (`**Recommendation**:`, `**Action**:`, `FINAL TRANSACTION PROPOSAL:`, `**Rating**:`). **Never break those headers** — `SignalProcessor` and the memory log both depend on them.

### Persistence

Two distinct persistence systems, both rooted at `~/.tradingagents/` (override base via `TRADINGAGENTS_CACHE_DIR`):

1. **Decision log** (`tradingagents/agents/utils/memory.py`, `TradingMemoryLog`) — always on. Append-only markdown at `~/.tradingagents/memory/trading_memory.md` (override with `TRADINGAGENTS_MEMORY_LOG_PATH`). Each `propagate()` call appends a `pending` entry; the **next** same-ticker run fetches realised return + alpha vs SPY via yfinance, generates a one-paragraph reflection, and atomically rewrites the entry. `get_past_context(ticker)` is the only consumer — it injects the most recent same-ticker decisions plus cross-ticker lessons into the Portfolio Manager prompt. The HTML comment `<!-- ENTRY_END -->` is the hard delimiter; it cannot appear in LLM prose. The `<!-- ENTRY_END -->` separator format and the bracketed tag-line format `[date | ticker | rating | raw% | alpha% | Nd]` are both load-bearing — parsing in `_parse_entry` and updates in `update_with_outcome` / `batch_update_with_outcomes` depend on them.

2. **LangGraph checkpoint resume** (`tradingagents/graph/checkpointer.py`) — opt-in via `config["checkpoint_enabled"]` or `--checkpoint`. Per-ticker SQLite at `~/.tradingagents/cache/checkpoints/<TICKER>.db`. Thread ID is `(ticker, trade_date)` so same date resumes, different date starts fresh. Cleared on successful completion.

The two systems are intentionally orthogonal: checkpoint = mid-run resume; memory log = cross-run learning.

### Data layer
`tradingagents/dataflows/interface.py` is a vendor-routing dispatcher. Tools in `tradingagents/agents/utils/*_tools.py` call `route_to_vendor(method, ...)`, which:

1. Reads `config["data_vendors"][category]` (or per-tool override in `config["tool_vendors"]`).
2. Builds a fallback chain (configured vendor first, then the rest).
3. Invokes vendors in order; **only `AlphaVantageRateLimitError` triggers fallback** — other exceptions propagate.

Currently `yfinance` and `alpha_vantage`. `yfinance` is the default and needs no key.

### Encoding & paths
- All file I/O **must** pass `encoding="utf-8"` explicitly. Windows defaults to cp1252 and silently mangles non-ASCII content. The v0.2.4 release was almost entirely about plugging these.
- Tickers can contain `.` and `/` (e.g. `BRK.B`, `7203.T`). Use `tradingagents/dataflows/utils.py::safe_ticker_component` before joining a ticker into any path — it rejects values that would escape the results dir.

### Output language
`config["output_language"]` (default `"Thai"`) controls the language of analyst reports and the final decision via `get_language_instruction()` in `agent_utils.py`. Internal debaters (Bull/Bear, Aggressive/Conservative/Neutral) **stay in English** for reasoning quality — do not propagate the language instruction to them.

### CLI vs Python entry points
`main.py` is a minimal scripted example. The real user-facing surfaces are:
- `cli/main.py` — Typer + Rich live display, with a `MessageBuffer` that maps LangGraph node updates to a fixed agent-status grid.
- `tradingagents/webui/server.py` — stdlib `ThreadingHTTPServer` (no Flask/FastAPI dependency). Settings persist to `~/.tradingagents/webui_settings.json`.

## Conventions specific to this repo

- **Test markers:** `unit`, `integration`, `smoke` (declared in `pyproject.toml`). Apply with `@pytest.mark.unit` etc. CI-friendly default is `pytest -m "not integration"`.
- **Provider lazy import:** Never import provider SDKs at module top-level in `llm_clients/`. The factory's lazy-import pattern is what lets the test suite run without any real API keys.
- **Markdown render preservation:** When changing structured-output schemas, the render helpers must keep producing the existing markdown headers (see `scripts/smoke_structured_output.py` for the contract).
- **Memory log atomicity:** Always go through `TradingMemoryLog.update_with_outcome` / `batch_update_with_outcomes` — they use temp-file + `os.replace()` so a crash mid-write never corrupts the log.
- **`reflect_and_remember()` is gone.** It was removed in v0.2.4 along with the BM25 per-agent memory; the persistent decision log replaces it. Do not reintroduce per-agent memory.

---
> Source: [aiunlocked1412/TradingAgentsGank](https://github.com/aiunlocked1412/TradingAgentsGank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
