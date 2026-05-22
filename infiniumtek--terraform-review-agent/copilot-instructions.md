## terraform-review-agent

> > Project contract for Codex / AI coding assistants. Replace `[PLACEHOLDERS]` per project; everything else is fixed standard.

# AGENTS.md

> Project contract for Codex / AI coding assistants. Replace `[PLACEHOLDERS]` per project; everything else is fixed standard.

---

## 1. Project

- **Name:** `terraform-review-agent`
- **Goal:** `A reusable GitHub Actions workflow that uses a LangGraph multi-agent system to review Terraform pull requests for security, cost, and style issues, posting a single severity-ranked comment.`
- **Trigger:** `GitHub `pull_request` events (`opened`, `synchronize`, `reopened`, `ready_for_review`) on Terraform-file changes, via a reusable workflow consumed by other repos. Users: PR authors and reviewers. Secondary: `workflow_dispatch` for manual re-runs.`
- **Framework:** `LangGraph`

---

## 2. Stack (fixed — do not change without explicit instruction)

- **Python 3.13** · **uv** package manager · **`.venv` only** (never system Python)
- **Pydantic v2** for all state, tool I/O, and config schemas
- **LangGraph** or **LangChain** 
- **SQLite** checkpointer (`langgraph-checkpoint-sqlite`) if keeping state is required
- **LLM providers: OpenAI, Anthropic, Google only** — selectable via config
- **Docker** (compose) for reproducibility · `structlog` logging · `pytest` · `ruff` · `mypy --strict`

---

## 3. Layout

Standard LangGraph src-layout (<https://docs.langchain.com/oss/python/langgraph/application-structure>):

```
[project]/
├── AGENTS.md, README.md, .env.example, .gitignore, .python-version
├── pyproject.toml, uv.lock, langgraph.json
├── Dockerfile, docker-compose.yml, Makefile
├── src/[package_name]/
│   ├── agent.py              # exposes compiled `agent` for langgraph.json
│   ├── config.py             # Pydantic Settings (env-driven)
│   ├── llm.py                # provider factory (OpenAI/Anthropic/Google)
│   ├── utils/{state,nodes,tools,prompts}.py
│   └── persistence/checkpointer.py
├── tests/{unit,integration}/
├── data/                     # SQLite, fixtures (gitignored if dynamic)
└── scripts/                  # thin wrappers only — no business logic
```

All importable code lives under `src/[package_name]/`. `agent.py` exposes the compiled graph as a module-level `agent` variable.

---

## 4. Setup & hard rules

```bash
python3.13 -m venv .venv && source .venv/bin/activate
pip install uv
uv sync --inexact --extra dev   # exact versions from uv.lock (--inexact keeps uv/pip)
cp .env.example .env            # fill in keys
```

- ❌ Never `pip install` outside `.venv`
- ❌ Never invoke a bare `python` — always activate `.venv` first
- ❌ Never commit `.env`, `data/*.sqlite`, `.venv/`, `__pycache__`
- 📌 Deps are pinned in `uv.lock` (committed). After editing `pyproject.toml`, run `uv lock` (or `make lock`); `make lint` runs `uv lock --check` and fails on drift.

---

## 5. Required env vars (`.env.example`)

```env
# At least one LLM key required
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
GOOGLE_API_KEY=

DEFAULT_LLM_PROVIDER=anthropic        # openai | anthropic | google
DEFAULT_LLM_MODEL=Codex-sonnet-4-5
DEFAULT_LLM_TEMPERATURE=0.0

SQLITE_PATH=./data/state.sqlite

# Optional observability
LANGSMITH_API_KEY=
LANGSMITH_TRACING=false
LANGSMITH_PROJECT=[PROJECT_NAME]

LOG_LEVEL=INFO
ENVIRONMENT=development               # development | staging | production
```

---

## 6. Patterns (where things live)

- **LLM factory** (`llm.py`): single `get_llm(provider, model, temperature)` switching on `openai|anthropic|google` using `langchain-openai`, `langchain-anthropic`, `langchain-google-genai`. Defaults from `config.settings`.
- **State** (`utils/state.py`): Pydantic models. For LangGraph messages: `Annotated[list[AnyMessage], add_messages]`. No untyped dicts.
- **Checkpointer** (`persistence/checkpointer.py`): `SqliteSaver.from_conn_string(settings.sqlite_path)`. Postgres only on explicit request.
- **Tools** (`utils/tools.py`): `@tool` from `langchain_core.tools` with Pydantic input schemas.
- **Prompts** (`utils/prompts.py`): never inlined in node code.
- **Config** (`config.py`): `pydantic_settings.BaseSettings` reading env — secrets never hardcoded.

---

## 7. Docker

- Base: `python:3.13-slim`, `.venv` at `/app/.venv` (identical to host), non-root user
- Compose: mount `./data` (SQLite persists across restarts) and `./src` (dev hot-reload — remove for prod)
- Run via `docker compose up`

---

## 8. `make` targets

`venv` · `install` · `fmt` · `lint` · `type` · `test` · `run` · `docker-build` · `docker-up` · `clean`

Python targets invoke `./.venv/bin/...` — never bare `python`. `run` is the exception: it executes inside the container (`docker compose run agent …`) so the bundled scanners are on `PATH`. Use these instead of re-inventing commands.

---

## 9. `langgraph.json`

```json
{
  "dependencies": ["."],
  "graphs": { "agent": "./src/[package_name]/agent.py:agent" },
  "env": "./.env",
  "python_version": "3.13"
}
```

---

## 10. Conventions

- Type hints everywhere; `mypy --strict` must pass on `src/`
- Pydantic for all data shapes — no bare dicts across module boundaries
- One node = one small, pure-ish function
- No `print` in `src/` — structured logging only
- Raise typed exceptions; let the graph handle retry/branching, not try/except spaghetti

---

## 11. Testing

- `tests/unit/` — pure function and node tests; mock LLM calls
- `tests/integration/` — compiled graph end-to-end with in-memory SQLite checkpointer
- Every node gets at least one unit test
- Use `pytest.fixture` for graph construction to stay DRY

---

## 12. Workflow when editing this project

1. Read this file, then `pyproject.toml`, then `src/[package_name]/agent.py`
2. Honor framework choice from §1
3. Add Pydantic schemas to `utils/state.py` before writing node code
4. New nodes in `utils/nodes.py`, wired in `agent.py`
5. Add at least one unit test
6. Run `make fmt lint type test` before declaring done

---

## 13. Out of scope (stop and ask first)

Cloud-managed DBs, vector stores, message queues · Kubernetes / Helm / Terraform · LLM providers other than OpenAI / Anthropic / Google · Frontend frameworks · Paid third-party APIs beyond LLMs.

---

## 14. References

- <https://docs.langchain.com/>
- <https://docs.langchain.com/oss/python/langgraph/application-structure>
- <https://docs.langchain.com/oss/python/langgraph/persistence>
- <https://docs.langchain.com/oss/python/langgraph/workflows-agents>
- <https://docs.pydantic.dev/latest/>
- <https://docs.astral.sh/uv/>

---
> Source: [infiniumtek/terraform-review-agent](https://github.com/infiniumtek/terraform-review-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
