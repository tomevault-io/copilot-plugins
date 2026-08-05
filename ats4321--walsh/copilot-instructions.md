## walsh

> Walsh is a multi-agent trading research system. Each agent produces an

# Walsh Architecture

Walsh is a multi-agent trading research system. Each agent produces an
independent structured thesis for a ticker:

- Fundamental Agent: company quality, valuation, financials, filings.
- Technical Agent: price action, trend, momentum, volume, levels.
- Sentiment Agent: news, market narrative, social or analyst tone.
- Macro Agent: rates, inflation, sector, currency, geopolitical context.
- Risk Manager Agent: reviews proposed trades and can veto unacceptable risk.
- Portfolio Manager Agent: synthesizes all agent theses into the final call.

The scaffold is intentionally fake-data-only. Real API wiring for market data,
SEC filings, news, and model/tool orchestration comes later.

# Required Agent Output

All agents must output `agents.schema.AgentThesis`:

```python
AgentThesis(
    ticker=str,
    agent_name=str,
    thesis=str,
    confidence=float,  # 0.0 through 1.0
    key_risk=str,
    supporting_data=dict,
    timestamp=datetime,
)
```

The schema is the shared contract between specialist agents, the Risk Manager,
the Portfolio Manager, backtests, memory, and evals.

# Project Structure

- `agents/`: base class, shared schema, and agent implementations.
- `tools/`: MCP-style tool interfaces. Current modules return mock data only.
- `memory/`: future persistence and retrieval layer.
- `backtest/`: future strategy simulation and historical replay.
- `eval/`: future evaluation harnesses and quality checks.
- `orchestrator/`: future multi-agent coordination workflows.
- `tests/`: pytest coverage.

# Coding Conventions

- Use Python 3.11+.
- Use Pydantic for all structured data exchanged between components.
- Use pytest for tests.
- Keep real API clients behind small interfaces so they can be mocked in tests.
- Do not add real trading or portfolio logic in scaffold modules.
- Do not make live external API calls in unit tests.

# Agent Contract

Any new agent MUST subclass `agents.base.Agent` and MUST return
`agents.schema.AgentThesis` from `analyze()`.

Subclasses should implement:

- `build_prompt()` for the agent-specific prompt.
- `parse_response(response)` to convert the Anthropic response into
  `AgentThesis`.

---
> Source: [ats4321/walsh](https://github.com/ats4321/walsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
