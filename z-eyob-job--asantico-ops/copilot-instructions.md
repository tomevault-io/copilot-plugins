## asantico-ops

> Asantico Operations: a local-first operations agent (`ops-agent/`) plus a LlamaIndex

# CLAUDE.md - Project Context and Conventions

## What this is
Asantico Operations: a local-first operations agent (`ops-agent/`) plus a LlamaIndex
knowledge RAG subsystem (`knowledge-rag/`). The agent reaches a single operator over
chat channels (CLI and Telegram), runs operational tools, and gates every money or
client action behind human approval. The agent, tests, and demo all run offline with
no keys; a real model and a live Telegram channel are optional, enabled by environment
variables. The human-in-the-loop gate has been verified both by a deterministic
worst-case test and live over a real Telegram channel.

## Golden rules
- The approval policy in `ops-agent/src/policy.py` is the safety spine. Any new tool
  MUST be registered there with a risk class (read / draft / gated). Unregistered
  tools are denied. Never weaken or bypass the gate, and never weaken a test to make
  it pass; fix the behavior and make the test assert the intended behavior.
- Reads and drafts run freely; sends and finalizations require explicit approval.
- The gate sits after routing, so it holds regardless of which router backend ran.
  Tests must prove a gated tool stays gated even when the router selects it.
- Every agent step emits a structured event via `ops-agent/src/observability.py`.
  Any new side-effecting path must log an event.
- Business invariants (enforced in the tool layer): Seattle tax 10.55% on every line
  item including labor; the company name is "Asantico" (never "Asantico LLC"); no
  tenant names on documents; client messages never state a dollar amount; no em dashes.
- Tests must not assert on non-deterministic model wording. Assert on deterministic
  signals: retrieved sources and their order, computed figures, risk class, and the
  gate's behavior.

## Runtime switches (config in the environment, not the code)
- `ROUTER_BACKEND`: `keyword` (default, no key) or `llm` (Anthropic function calling;
  falls back to keyword on no key or error).
- `KB_BACKEND`: `offline` (default, dependency-free hash embedding) or `rag` (the real
  LlamaIndex pipeline; falls back to offline if unavailable).
- `KB_GROUNDED`: set to `0` to force deterministic raw-snippet answers even with a key.
- `ANTHROPIC_MODEL`: overrides the model; default `claude-sonnet-4-6`.
- `TELEGRAM_BOT_TOKEN`: enables the Telegram channel; read from the environment only.
- Keys live only in `ops-agent/.env` (gitignored); never hardcode, log, or commit them.

## Layout
- `ops-agent/src/gateway.py` - entrypoint; wires a channel to the agent loop
- `ops-agent/src/agent/` - loop (human-in-the-loop state) and router (keyword
  default; `ROUTER_BACKEND=llm` uses Anthropic function calling, falls back to keyword)
- `ops-agent/src/llm.py` - optional Anthropic access (lazy import; key from env only)
- `ops-agent/src/channels/` - channel interface plus CLI and Telegram (email/WhatsApp
  are stubs for future work); Telegram keys approval state by chat id
- `ops-agent/src/tools/` - registry + domain tools (wrap the real asantico-cli in production)
- `ops-agent/src/policy.py` - approval gate (read / draft / gated)
- `ops-agent/src/observability.py` - structured JSON logging
- `ops-agent/src/mcp_server.py` - exposes tools over MCP (stdio)
- `knowledge-rag/knowledge_rag/` - the RAG pipeline behind the knowledge_base tool
  (importable as `knowledge_rag`; selected at runtime via `KB_BACKEND=rag`)

## Commands
- Run the agent demo (offline): `cd ops-agent && python -m src.gateway`
- Run with the model and real RAG: `ROUTER_BACKEND=llm KB_BACKEND=rag python -m src.gateway`
- Run the Telegram channel: `ROUTER_BACKEND=llm python -m src.gateway telegram`
  (on macOS first: `export SSL_CERT_FILE=$(python -c "import certifi; print(certifi.where())")`)
- Run the MCP server: `cd ops-agent && python -m src.mcp_server`
- Tests: `pytest` from each component folder
- Retrieval eval: `cd knowledge-rag && python -m knowledge_rag.ingest && python -m knowledge_rag.evaluate`
- Lint/format: `ruff check .` and `ruff format .`

## Conventions
- Python >= 3.11, run modules as `python -m src.<module>` from the component folder so
  relative imports resolve.
- Keep changes small and reviewable, one focused branch at a time.
- The safety gate, the observability logging, and the business invariants are not
  optional and must hold across every change.

---
> Source: [Z-eyob-Job/asantico-ops](https://github.com/Z-eyob-Job/asantico-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
