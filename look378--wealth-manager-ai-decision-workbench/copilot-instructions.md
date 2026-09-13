## wealth-manager-ai-decision-workbench

> - Install locked dependencies: `uv sync` (Python 3.11-3.12; uv.lock pins everything).

# Wealth Manager AI Decision Workbench — Copilot Instructions

## Build and test commands

- Install locked dependencies: `uv sync` (Python 3.11-3.12; uv.lock pins everything).
- Run the Gradio UI: `python app.py` (http://127.0.0.1:7862, `WEALTH_PORT` to override).
- Run the CLI workbench entry: `wealthdesk wealth --profile '{"risk_level":"C3"}' --products 510300.SH`
- Run the full test suite: `uv run pytest -q`
- Run one focused test: `uv run pytest tests/test_compliance.py::SuitabilityTests -q`

## High-level architecture

- `wealthdesk/graph/wealth_graph.py` — `WealthAgentsGraph` is the workbench facade: a client
  profile + product pool flow assembled by `WealthGraphSetup` (`wealth_setup.py`) on top of
  the shared `RuntimeGraph` runtime spine (`runtime_graph.py`: deep-copies config, calls
  `set_config(config)`, creates quick/deep LLMs via `wealthdesk.llm_clients.create_llm_client()`,
  wires memory, compiles the LangGraph workflow, optionally recompiles with a SQLite
  checkpointer, persists state logs).
- Pipeline: six dimension analysts (liquidity / profitability / risk / tax / succession /
  suitability) run serially with a Msg Clear node isolating each private chat → bull/bear
  debate (bounded by `max_debate_rounds`) → research manager (asset-class allocation bands +
  deterministic 实质分歧 marker) → serial risk review (aggressive → conservative → neutral) →
  portfolio manager emitting a rule-based four-tier conclusion (适合/混合/不适合/信息不足) →
  compliance gate (banned-term scan + C1-C5 suitability + top disclaimer injection).
- `wealthdesk/agents/utils/decision_rules.py` owns the deterministic four-tier `classify_decision`;
  `wealthdesk/compliance.py` owns banned-term scanning, suitability, and disclaimer injection.
  Deterministic transforms belong in code, never in prompts — the model only writes prose.
- `wealthdesk/dataflows/interface.py` `route_to_vendor()` is the vendor routing layer for
  agent-facing tools. Quotes route through akshare first (stock: `stock_zh_a_hist`, ETF:
  `fund_etf_hist_em` → Sina fallback), with tushare used for static info and stock-daily fallback.
- Runtime artifacts: reports under `results_dir` (default `~/.wealthdesk/logs`), checkpoint
  databases under `data_cache_dir/checkpoints/`, analysis memory under `results_dir/memory/`.
- Gradio UI: `wealthdesk/webapp.py` builds the demo; `app.py` launches it. The CLI:
  `cli/main.py` (`wealthdesk wealth`, plus `detail`/`cache`/`watchlist`/`memory` utilities).

## Key conventions

- **Compliance hard rules**: every report carries the financial disclaimer at the top
  (injected once by the compliance module); no node may output investment recommendations,
  buy/sell advice, target prices, or position instructions — only stance tiers
  (适合/混合/不适合/信息不足) and risk lists. Extend banned terms in
  `DEFAULT_CONFIG["compliance_banned_terms"]`, not in prompts.
- Do not mutate `DEFAULT_CONFIG` via a shallow copy — start from `copy.deepcopy(DEFAULT_CONFIG)`.
  Treat config as global runtime state (`wealthdesk/dataflows/config.py`).
- Do not instantiate provider-specific LangChain clients directly in graph or agent code.
  Use `create_llm_client()` so OpenAI-compatible backends and base-URL handling stay normalized.
- **Mock everything in tests**: fake LLMs and monkeypatched tool data; never call
  DeepSeek/tushare/akshare in tests.
- Localization is helper-driven (`output_language`, Chinese default): use the localization
  helpers in `wealthdesk/agents/utils/agent_utils.py` instead of embedding bilingual strings
  by hand.
- The graph skeleton (`runtime_graph.py`, `wealth_setup.py`) is shared infrastructure — adapt
  it via subclasses and configuration; never rewrite the StateGraph assembly from scratch.

---
> Source: [look378/wealth_manager_ai_decision_workbench](https://github.com/look378/wealth_manager_ai_decision_workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
