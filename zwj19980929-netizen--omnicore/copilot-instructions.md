## omnicore

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OmniCore is a personal-use general-purpose Agent Runtime — a personal digital assistant backbone that accepts natural language tasks, plans them, dispatches tools, runs in background, supports human-in-the-loop approvals, and persists outcomes across sessions. It is **not** a demo or multi-tenant platform.

## Commands

**Setup:**
```bash
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
playwright install chromium
cp .env.example .env  # then fill in API keys
```

**Run:**
```bash
python main.py                        # Interactive CLI
python main.py "task description"     # One-shot task
python main.py worker                 # Foreground queue worker
python main.py worker --process-loop  # Long-running worker
streamlit run ui/app.py               # Streamlit UI
```

**Test:**
```bash
pytest tests -q                              # Full suite
pytest tests/test_router_unit.py -q          # Single file
python -m utils.encoding_health              # Encoding health check
```

Test files follow the naming convention `test_<feature>_unit.py` (unit) or `test_<feature>_integration.py` (integration). pytest config is in `pytest.ini`; test artifacts go to `data/test-runtime/`.

## Architecture (7 Layers)

| Layer | Responsibility | Key Files |
|---|---|---|
| 1. Interaction | CLI, Streamlit UI, session history, daily dashboard | `main.py`, `ui/app.py`, `ui/pages/` |
| 2. Runtime & Orchestration | Job queue, worker, checkpoint, task lifecycle | `core/runtime.py`, `core/graph.py` |
| 3. Planning & Decision | Intent routing, task planning, validation, policy | `core/router.py`, `core/task_planner.py`, `core/policy_engine.py`, `agents/validator.py`, `agents/critic.py` |
| 4. Tool Dispatch | Tool-first scheduling, registry, adapters, executor | `core/tool_registry.py`, `core/tool_adapters.py`, `core/task_executor.py`, `core/tool_protocol.py` |
| 5. Tool Capability | Actual execution: web, browser, file, system, API | `agents/web_worker.py`, `agents/browser_agent.py`, `agents/file_worker.py`, `agents/system_worker.py` |
| 6. State, Memory & Work Context | Session/Job/Artifact, Goal/Project/Todo, vector memory | `core/state.py`, `utils/runtime_state_store.py`, `utils/artifact_store.py`, `utils/work_context_store.py`, `memory/` |
| 7. Automation & Collaboration | Scheduled tasks, directory watching, templates, approvals, notifications | `utils/workflow_automation_store.py` |

## Core Data Model

- **Session** — persistent work session aggregating multiple Jobs
- **Job** — single task submission (`queued` → executing → done/waiting/blocked)
- **Task** — sub-step inside a Job, bound to a specific tool
- **Artifact** — reusable output (files, downloads, structured results)
- **Goal / Project / Todo** — continuous work context threading multiple Jobs
- **PolicyDecision** — approval/risk gate records

## Key Execution Flows

1. **Normal:** User input → `submit_job` → Router/Planner → Tool Dispatch → Validator/Critic → Artifact → persist
2. **Background:** Queue → Worker → `run_next_queued_task` → write back Job/Artifact/notification
3. **Approval gate:** High-risk action → `waiting_for_approval` → user approve/reject → resume or block
4. **Schedule/Watch:** Cron trigger or file-system event → auto-create Job → worker consume

## LLM Routing

Multi-provider via LiteLLM. Supported providers: OpenAI, Anthropic, Gemini, DeepSeek, MiniMax, Kimi. Model capability defaults are in `config/models.yaml`. Active provider is controlled by `DEFAULT_MODEL` and `PREFERRED_PROVIDER` env vars.

LLM prompt templates are `.txt` files under `prompts/` (router, planner, browser, critic, etc.).

## Key Config

| File | Purpose |
|---|---|
| `.env` (from `.env.example`) | API keys, proxy, model routing, feature flags |
| `config/models.yaml` | Model capability overrides and provider API base URLs |
| `config/settings.py` | Runtime settings module |
| `prompts/` | LLM prompt templates |

**Notable env vars:**
- `DEFAULT_MODEL`, `PREFERRED_PROVIDER` — model routing
- `REQUIRE_HUMAN_CONFIRM` — approval gate toggle
- `BROWSER_FAST_MODE`, `BLOCK_HEAVY_RESOURCES`, `STATIC_FETCH_ENABLED` — browser tuning
- `OMNICORE_HTTP_PROXY` — project-scoped proxy (system proxy is disabled by default)
- `CHROMA_PERSIST_DIR` — vector memory path

## Coding Conventions

- Python: 4-space indent, `snake_case` for functions/variables, `CamelCase` for classes, UTF-8 encoding throughout
- Commit style: `feat:` / `fix:` prefixes, short imperative subject lines
- Never commit `.env` or the `data/` directory
- The authoritative architecture reference is `docs/2026-03-04-通用Agent当前架构说明.md`; earlier docs in `docs/` are historical snapshots only

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zwj19980929-netizen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
