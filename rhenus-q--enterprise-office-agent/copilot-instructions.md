## enterprise-office-agent

> Guidance for Claude Code when working in this repository.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## 1. Project Overview

This repository — **Enterprise Office Agent** — is organized as named capability modules
(see [ADR 014](docs/adr/enterprise_rag/014-enterprise-rag-package-and-office-agent-placeholder.md)):

- **`enterprise_rag/`** — ✅ **the completed module**, and the subject of everything below:
  an **enterprise internal-document Q&A engine** built with **LangGraph**, implementing a
  self-correcting Agentic RAG (CRAG-style) workflow. It answers questions from an ingested
  knowledge base and falls back to web search when needed. Public entry point:
  `enterprise_rag.graph.engine.answer_question()`.
- **`office_agent/`** — ✅ **implemented — seven capabilities, complete since v1.6.0 / Phase 7.** A
  deterministic, LLM-free intent router — entry point `office_agent.engine.answer_office_request(user_input, options=None)`
  — over local capabilities. Version map: **v1.0.0 / Phases 1-5** — **Knowledge Q&A** (a thin
  adapter over the `enterprise_rag` engine), **Email Summary**, **Calendar Lookup**,
  **Task / Ticket Assistant**, **Daily Briefing**; **v1.5.0 / Phase 6** — **Meeting Agent /
  Meeting Prep**; **v1.6.0 / Phase 7** — **Workflow / Approval Agent**. All tools except
  Knowledge Q&A run on local mock data with no LLM and no external services — the sole
  exceptions are two **optional, default-off** LLM assists, the email digest
  ([ADR 017](docs/adr/office_agent/017-office-agent-llm-assist-email-digest.md)) and the Daily Briefing
  narrative ([ADR 018](docs/adr/office_agent/018-office-agent-llm-assist-daily-briefing.md)), both gated
  by the single `OFFICE_LLM_ENABLED` switch and inert unless it is set. Office-agent
  work **must not change or regress `enterprise_rag` behavior or its tests** (§3 rules apply:
  side-effect-free imports, lazy `@lru_cache` external clients). See
  [`office_agent/README.md`](office_agent/README.md) (the dedicated Office Agent
  demo / usage doc) and [ADR 015](docs/adr/office_agent/015-office-agent-v1-architecture.md); office-agent
  working rules are in §3.

Root docs (`README.md`, `CLAUDE.md`, `structure.md`, `docs/adr/`) are repository-level;
detailed engine usage lives in `enterprise_rag/README.md`, the dedicated Office Agent demo /
usage doc is `office_agent/README.md`, and engineering / release docs live under
`docs/engineering/` and `docs/releases/` (plus the employee-facing quickstart in
`docs/employee-guide/`).
Most of this file is guidance for working in `enterprise_rag`; office-agent-specific rules are
called out in §3.

**Stack:** LangGraph, LangChain, OpenAI (`gpt-5-mini`, `OpenAIEmbeddings`), Chroma (vector
store), Tavily (web search). Managed with **uv**.

**High-level flow** (see `structure.md` for details):

```
question
→ route_question
    ├── websearch → generate
    └── retrieve → grade_documents
            ├── relevant docs → generate
            └── no relevant docs → websearch → generate
generate
→ grounding check (hallucination_grader)
    ├── not grounded → add grounding feedback → regenerate
    └── grounded → usefulness check (answer_grader)
            ├── useful → END
            └── not useful → rewrite search query → websearch
```

Three quality gates: **document relevance**, **answer grounding** (anti-hallucination), and
**answer usefulness**. A `retries` counter in state caps the regenerate/websearch loop at
`MAX_RETRIES = 5` (defined in `enterprise_rag/graph/graph.py`).

External dependency failures (retriever, Tavily, generation LLM, graders, query rewriter)
never crash the graph: each call site catches the exception, degrades or stops safely, and
records a `stop_reason` (`retrieval_error`, `web_search_error`, `generation_error`,
`tool_error`) so the RAG CLI (`enterprise_rag/cli.py`) appends an honest caveat. Console banners
log only the exception type, never the message.

**Runtime privacy modes** (default off, strict truthy parsing — see
[ADR 019](docs/adr/019-hierarchical-runtime-privacy-modes.md)):

- **`PRIVACY_MODE`** — no data leaves the machine except to OpenAI: forces off Tavily web
  search, LangSmith tracing, and both optional Office LLM assists, while preserving the core
  OpenAI RAG path unchanged.
- **`OFFLINE_MODE`** — higher precedence: implies every `PRIVACY_MODE` restriction and
  additionally disables OpenAI chat/embeddings, ingestion, and every other external-service
  path, failing closed with the additive `offline_mode` stop reason.

Precedence is strict and one-directional:
`OFFLINE_MODE` > `PRIVACY_MODE` > individual environment flags > per-run overrides.
**A mode can only restrict** — while active it overrides `WEB_SEARCH_ENABLED=true`,
`OFFICE_LLM_ENABLED=true`, the tracing variables, and an explicit
`AnswerOptions(web_search_enabled=True)`; no lower-level flag can re-enable an external
service.

## 2. Project Structure

| Path | Purpose |
|------|---------|
| `main.py` | Repository-level entry point for the **Enterprise Office Agent** — imports and calls `office_agent.cli.main`, so `uv run python main.py` launches the Office Agent interactive CLI (identical to `uv run python -m office_agent.cli`). It imports nothing from `enterprise_rag`; the old `from main import …` formatting re-export surface is retired ([ADR 020](docs/adr/020-module-owned-cli-entry-points.md)); import those names from `enterprise_rag.graph.formatting`. |
| `enterprise_rag/cli.py` | The standalone Enterprise RAG interactive CLI: loads `.env`, enforces tracing privacy, prints the OFFLINE_MODE / PRIVACY_MODE / WEB_SEARCH_ENABLED banner, then runs the interactive Q&A loop over `enterprise_rag.graph.engine.answer_question()` with `format_answer` output (caveats + Sources). Side-effect-free import (`.env`/tracing done inside `main()`). Run via `uv run python -m enterprise_rag.cli`. See [ADR 020](docs/adr/020-module-owned-cli-entry-points.md). |
| `office_agent/cli.py` | The Office Agent interactive CLI over `office_agent.engine.answer_office_request()` — the product-level/default CLI (root `main.py` launches it): displays routed intent, selected tool, response content, and (when set) stop reason, sources, run id. Pure presentation — no router/tool logic duplicated; imports nothing from `enterprise_rag`. Run via `uv run python -m office_agent.cli`. See [ADR 020](docs/adr/020-module-owned-cli-entry-points.md). |
| `enterprise_rag/__init__.py` | Package marker + docstring for the RAG engine. No clients, no side effects. |
| `enterprise_rag/README.md` | Module-level documentation: detailed engine setup, usage, privacy mode, fallback policy, programmatic API, budgets, and failure handling (the content that used to dominate the root README). |
| `office_agent/` | The **Enterprise Office Agent** — seven capabilities: the v1.0.0 / Phases 1-5 core tools, the v1.5.0 / Phase 6 Meeting Agent, and the v1.6.0 / Phase 7 Workflow / Approval Agent. Deterministic keyword router (`router.py`), `answer_office_request()` entry point (`engine.py`), typed intent constants + `ToolResult` schemas (`schemas.py`), unsupported-intent + presentation (`formatting.py`), `tools/` — `knowledge` (thin `enterprise_rag` adapter) plus `email`, `calendar`, `tickets`, `briefing`, `meeting`, `approvals` (local mock-data tools), and `llm_assist/` — two **optional, default-off** LLM assists, the email digest layered on the `email` tool ([ADR 017](docs/adr/office_agent/017-office-agent-llm-assist-email-digest.md)) and the Daily Briefing narrative layered on the `briefing` tool ([ADR 018](docs/adr/office_agent/018-office-agent-llm-assist-daily-briefing.md)), both gated by the single `OFFICE_LLM_ENABLED` switch (with its own mode readers in `llm_assist/config.py` — no `enterprise_rag` import — so either privacy mode forces the assists off). Local-only and LLM-free except the Knowledge Q&A adapter and (only when explicitly enabled) those two assists; mock data in `mock_data/` is read-only and anchored to the data (not the system clock) — approve/reject and follow-up tasks are *simulated*, never written back. Must never regress `enterprise_rag`. See `office_agent/README.md` and ADRs 015–018. The deterministic capabilities keep working under both privacy modes. |
| `office_agent/run_settings.py` | Pure, request-scoped **Run Settings resolver** ([ADR 021](docs/adr/021-frontend-observability-workspace.md)): `resolve_run_settings(intent, options, server_*=…)` over frozen dataclasses, called by `engine.py` **after** routing (routing is never influenced by per-run options). **Server policy always wins** — a per-run request can only *restrict* (turn an external path off), never re-enable one the server prohibits; precedence is `OFFLINE_MODE` > `PRIVACY_MODE` > server flags > per-run options. Reads no env var and mutates no module global, so two concurrent requests with opposite settings stay isolated. Reports `requested` / `effective` / `applicability` / `constraints`. |
| `api/` | **Thin FastAPI adapter** over `office_agent.engine.answer_office_request()` ([ADR 021](docs/adr/021-frontend-observability-workspace.md)) — the repository's HTTP surface. `create_app()` factory (no module-level app; `.env` + `enforce_tracing_privacy()` inside, like the CLIs). Exactly two routes — `GET /api/health` (the existing flag readers only) and `POST /api/agent/run` (one engine call, `OfficeAgentResponse` mapped 1:1 plus adapter-measured `duration_ms`, adapter-derived `execution_mode`, Knowledge-Q&A-only `observability`, backend-resolved `run_settings`; engine errors → HTTP 500 with the exception **type name only**). Must **not** duplicate routing, intent detection, formatting, privacy, fallback, date interpretation, or tool behavior; localhost-only, no auth. `api/schemas.py` holds the Pydantic v2 wire models. Needs the `api` dependency group (`fastapi`/`uvicorn`/`httpx`). |
| `frontend/` | **Vite + React + TypeScript observability workspace** ([ADR 021](docs/adr/021-frontend-observability-workspace.md)) — one three-pane layout over all seven capabilities plus `unknown`, talking to the adapter through a single `AgentClient` contract shared by its `http` (default) and `mock` (`VITE_API_MODE=mock`) implementations. npm-managed, independent of uv. Must **not** re-derive backend-owned business behavior: backend-reported `effective` Run Settings and `observability` are authoritative and shown **only** from the response (never recomputed, never inferred from the read-only server-policy badges), engine `content` is rendered verbatim, and no date comes from the browser clock. See [`frontend/README.md`](frontend/README.md). |
| `enterprise_rag/graph/engine.py` | Canonical programmatic API: `answer_question(question, options) -> AnswerResult`, `AnswerOptions` (per-run `web_search_enabled` / `web_fallback_policy` / `run_id` / `trace_path` overrides; `None` = env default), and `seed_state()` — the single state-seeding helper shared by CLI, evals, and tests. Applies the runtime privacy floor for every caller: `seed_state()` forces `web_search_enabled=False` under either mode (overriding an explicit per-run `True`), and `answer_question()` calls `enforce_tracing_privacy()` and, under `OFFLINE_MODE`, short-circuits **before the graph** with `STOP_REASON_OFFLINE_MODE` — no client built, no external request attempted. Also owns the lightweight observability: every run gets a `run_id`, the executed `node_path` + per-step timings + `total_duration_ms` are collected by streaming graph updates (additive — merging the updates reproduces `invoke()`), and `trace_path` optionally writes a metadata-only trace JSON (never `page_content`, prompts, raw state, or keys). |
| `enterprise_rag/graph/formatting.py` | Shared presentation: `stop_reason` caveats (`STOP_REASON_NOTES`) plus the deterministic `Sources:` section built from `Document` metadata (`format_answer` / `format_sources` / `source_lines`; local corpus vs. `web_search` supplement). Pure — no clients, no env reads. |
| `enterprise_rag/ingestion.py` | Builds the knowledge base: loads the local Markdown corpus from `enterprise_rag/data/acmecorp_internal_docs/`, splits, embeds, persists to Chroma (idempotent: collection reset + deterministic chunk ids; provenance metadata `source`/`title`/`source_type`/`document_category`). Exposes `get_retriever()` (lazy, `@lru_cache`). Run once before `main.py`. Fails closed under `OFFLINE_MODE`: the script entry refuses (exit 2) and `get_retriever()` raises `RuntimeError`, both before any client is constructed. |
| `enterprise_rag/runtime_privacy.py` | `enforce_tracing_privacy()` — the one early-initialization side effect of the privacy modes: forces both `LANGCHAIN_TRACING_V2` and `LANGSMITH_TRACING` to `"false"` when a mode is active (strict no-op otherwise, idempotent). Kept out of `config.py`, whose contract is pure env reads; called after `load_dotenv()` at each entry point and per-run inside `answer_question()`. |
| `enterprise_rag/data/acmecorp_internal_docs/` | Synthetic AcmeCorp enterprise corpus: 6 fictional internal Markdown documents (VPN, expenses, incident response, on-call, data retention, onboarding). No real company data — safe to edit/extend. |
| `enterprise_rag/graph/graph.py` | Assembles the LangGraph `StateGraph`, wires nodes + conditional edges, exports compiled `app`. Holds `MAX_RETRIES` and the routing decision functions. |
| `enterprise_rag/graph/state.py` | `GraphState` TypedDict: `question`, `documents`, `generation`, `web_search`, `web_search_enabled`, `web_fallback_policy` (resolved per run by the engine; graph decisions read it from state), `retries`, `stop_reason`, `insufficient_context`, `retry_feedback`, `search_query`, plus budget counters (`llm_call_count`, `web_search_count`, `web_result_grading_count`). |
| `enterprise_rag/graph/config.py` | Env-driven runtime flags (pure env reads, no side effects): the runtime privacy modes `privacy_mode()` / `offline_mode()` / `privacy_restrictions_active()` (the "offline implies privacy" hierarchy), `web_search_enabled()` (returns `False` whenever restrictions are active), `web_fallback_policy()` / `normalize_web_fallback_policy()` (conservative/aggressive/disabled, default conservative; the env var is the *default source* — the engine resolves the effective policy into per-run state), and the per-run budgets `max_llm_calls_per_run()` / `max_web_searches_per_run()` / `max_web_results_to_grade()`. |
| `enterprise_rag/graph/consts.py` | Node-name string constants (`RETRIEVE`, `GRADE_DOCUMENTS`, `GENERATE`, `WEBSEARCH`, `WEB_SEARCH_DISABLED_NOTICE`) and `stop_reason` values. |
| `enterprise_rag/graph/nodes/` | Graph node functions: `retrieve`, `grade_documents`, `generate`, `web_search`, retry helpers (`add_grounding_feedback`, `rewrite_query`), plus terminal notice nodes (`web_search_disabled_notice`, `web_fallback_disabled_notice`, `max_retries_not_grounded_notice`, `max_retries_not_useful_notice`, `budget_exhausted_notice`, `tool_error_notice`) that record `stop_reason`, and `clear_transient_tool_error` (success-path pass-through: clears a stale transient `tool_error` once both gates pass). |
| `enterprise_rag/graph/chains/` | LCEL chains: `generation`, `retrieval_grader`, `question_router`, `hallucination_grader`, `answer_grader`, `query_rewriter`. Each exposes a lazy `get_*()` factory. |
| `tests/` | Test tree mirrors the two source modules plus the HTTP adapter: `tests/enterprise_rag/`, `tests/office_agent/`, and `tests/api/`, with `tests/conftest.py` shared at the root. |
| `tests/enterprise_rag/nodes/` | Unit tests for `enterprise_rag` node functions. Fully mocked — no API keys needed. |
| `tests/enterprise_rag/graph/` | Routing / privacy-toggle / compiled-graph tests. Fully mocked — no API keys needed. |
| `tests/enterprise_rag/chains/` | Integration tests for the `enterprise_rag` chains. Call the real `gpt-5-mini` — marked `real_model`, so they skip unless `RUN_REAL_MODEL_TESTS=1` and `OPENAI_API_KEY` are both set. |
| `tests/enterprise_rag/evals/` | Mocked unit tests for the Enterprise RAG eval harness's pure helpers (validation, checks, metrics, rendering). No API keys needed. |
| `tests/office_agent/` | Unit tests for the Office Agent (router, engine dispatch, each mock tool, and the LLM email-digest / briefing assists mocked at their seams). Fully mocked / deterministic — no OpenAI, Tavily, Chroma, or external services; no `enterprise_rag` graph call (the knowledge adapter is patched) and no real assist call. No API keys needed. |
| `tests/office_agent/evals/` | Mocked unit tests for the two Office Agent LLM-assist eval runners (email digest + briefing narrative env loading and CONFIG/INFRA/EVAL_FAIL classification). No API keys needed. |
| `tests/office_agent/integration/` | Gated real-model tests for the Office Agent LLM assists (email digest + briefing narrative; call the real `gpt-5-mini` — marked `requires_openai`/`real_model`, so they skip unless `RUN_REAL_MODEL_TESTS=1` and `OPENAI_API_KEY` are both set). Kept out of the mocked `tests/office_agent/` unit suite so it stays keys-free; gated like `tests/enterprise_rag/chains/`. |
| `tests/api/` | Keys-free **mocked adapter tests** for `api/` (`fastapi.testclient` with `answer_office_request` and the flag readers monkeypatched — no real engine call): the health flag matrix, 1:1 field mapping, the `execution_mode` matrix, `text` length bounds, the type-only 500 handler, the additive `observability` / `run_settings` pass-through, the app-factory tracing-privacy enforcement, and the OpenAPI wire contract. Run in CI's `mocked-tests` job; needs the `api` dependency group installed. |
| `evals/` | Evaluation harnesses, organized by owning module (root `evals/README.md` is a short navigation page). `evals/enterprise_rag/` is the **Enterprise RAG behavioral eval**; `evals/office_agent/llm_assist/` evaluates only the two optional Office Agent LLM assists (email digest + briefing narrative). Not part of CI. |
| `evals/enterprise_rag/` | Behavioral eval harness for the RAG graph: `questions.jsonl` (24-row dataset with multi-document and fallback-policy rows; optional per-row `web_fallback_policy`, source-title, min-local-source, and web-search-count checks), `run_eval.py` (runs the real graph via `enterprise_rag.graph.engine.answer_question()` — **never run the full eval without explicit approval**; `--validate-only` is safe), `results.md` (generated report), and `history/`. A full run refuses under `OFFLINE_MODE` (`CONFIG ERROR`, exit 2, report/history untouched); `PRIVACY_MODE` deliberately does *not* block it, but forces web search off, so web-dependent rows fail by design. Each full run also writes a metadata-only JSON history record and renders a "Delta vs. previous run" section in the report. |
| `evals/enterprise_rag/history/` | Append-only, metadata-only eval history records (one JSON per full run; never answer text, `page_content`, prompts, or raw state). The harness only writes new records — never edits/deletes. `evals/enterprise_rag/history/*.json` is gitignored by default (the dir is tracked via `.gitkeep`); force-add (`git add -f`) to share a known-good baseline. |
| `evals/office_agent/llm_assist/` | Standalone offline-validator + approval-gated real-model evals for the two optional Office LLM assists: `run_email_digest_eval.py` / `email_digest_cases.jsonl` and `run_briefing_narrative_eval.py` / `briefing_narrative_cases.jsonl`, sharing `_env.py` (whose `ensure_openai_api_key()` refuses a full run under **either** privacy mode, since both disable the assists being measured). `--validate-only` is keys-free and mode-free; generated `*_results.md` reports are gitignored. Evaluates the assists only — not the seven deterministic capabilities (those are covered by `tests/office_agent/`). |
| `docs/adr/` | Architecture Decision Records (001–021) with an index in `docs/adr/README.md`, split by owning scope: `docs/adr/enterprise_rag/` (001–014), `docs/adr/office_agent/` (015–018), and the repository-wide ADRs directly under `docs/adr/` — [019 — hierarchical runtime privacy modes](docs/adr/019-hierarchical-runtime-privacy-modes.md), [020 — module-owned CLI entry points](docs/adr/020-module-owned-cli-entry-points.md), and [021 — frontend observability workspace and thin FastAPI adapter](docs/adr/021-frontend-observability-workspace.md) (the presentation tier: `api/` + `frontend/`). When a documented decision changes, update or supersede the matching ADR. |
| `docs/roadmap/` | A **local working-artifact area** (see `docs/roadmap/README.md`): `spec/`, `plan/`, `implementation/`, `commands-review/`, plus per-topic `<topic>-review/` dirs (e.g. `architecture-review/`, `security-review/`, `failure-modes-review/`, `test-coverage-review/`). Its contents are ignored by Git by default, and **exactly four files are version-controlled** (workflow infrastructure the `.claude/commands/` files depend on): `docs/roadmap/README.md`, `docs/roadmap/spec/spec-template.md`, `docs/roadmap/plan/plan-template.md`, and `docs/roadmap/implementation/implementation-template.md`. Everything else — specs, plans, implementation reports, and all review reports — **stays local and is never committed**. **Durable conclusions must be promoted** into the tracked documentation instead: `docs/adr/`, `docs/engineering/`, `docs/releases/`, the READMEs, `structure.md`, tests, or code. Reports from project-level `<topic>-review` commands (architecture, security, failure-modes, test-coverage) go under `docs/roadmap/<topic>-review/` with dated `<YYYY-MM-DD>-<focus-slug>-<topic>-review.md` collision-safe filenames and must not overwrite prior reports; `docs/roadmap/commands-review/` holds command-file review reports (e.g. `/review-command`). |
| `.claude/commands/` | Claude Code slash-command workflow files (spec → plan → implement → review-diff; plus `arch-review`, command-authoring/review, and `update-claude-md`). Each has YAML frontmatter (`description`, `argument-hint`, `allowed-tools`); keep `allowed-tools` minimal and scoped (e.g. `Bash(git status:*)`, not blanket `Bash`). |
| `tests/conftest.py` | Pytest bootstrap: isolates ordinary tests from the local environment (forces `OFFICE_LLM_ENABLED=false` and disables `.env` loading for the pytest process) and owns real-model gating — the canonical `real_model` marker (legacy `requires_openai` alias) skips unless `RUN_REAL_MODEL_TESTS=1` and the marker's named credentials (default `OPENAI_API_KEY`) are set, and always skips under `OFFLINE_MODE`. |
| `pyproject.toml` | uv project config: deps, `[dependency-groups]` — `dev` (pytest, pytest-cov, ruff, mypy, pre-commit) and `api` (fastapi, uvicorn, httpx; required for `api/` and `tests/api/`) — `[tool.pytest.ini_options]`, `[tool.ruff]`/`[tool.ruff.lint]`/`[tool.ruff.lint.per-file-ignores]`, and `[tool.mypy]`/`[[tool.mypy.overrides]]` (the `files` list scopes mypy to the engine-API surface, `office_agent/`, and `api/`). |
| `.gitattributes` | Line-ending policy: `* text=auto` plus explicit `*.py/md/yml/yaml/toml/json text` patterns, marking those files as text so Git normalizes line endings to LF in the repository. No `eol=` is configured, so working-copy endings follow each clone's `core.autocrlf`/platform default. |
| `.pre-commit-config.yaml` | Local hooks mirroring CI: `ruff-check --fix`, `ruff-format`, and basic hygiene hooks (`trailing-whitespace`, `end-of-file-fixer`, `check-yaml`, `check-toml`, `check-added-large-files`, `check-merge-conflict`). |

## 3. Development Rules

- **Preserve behavior by default.** Do not change graph routing, `GraphState` schema, prompts,
  model names (`gpt-5-mini`), `temperature=0`, chain input variables, or node return
  structures unless explicitly asked.
- **No broad architecture changes.** Avoid restructuring the graph or rewriting modules wholesale.
- **`GraphState` fields are plain last-value channels.** Do not add `typing.Annotated` reducers /
  accumulating channels: `enterprise_rag/graph/engine.py` merges streamed node updates with `dict.update()`, which
  only reproduces `app.invoke()` for last-value channels. If a reducer is ever needed, revisit that merge first.
- **Refactors should be small, mechanical, and reviewable.** Prefer minimal diffs.
- **Lazy external clients (required pattern).** `ChatOpenAI`, `OpenAIEmbeddings`,
  `TavilyClient` (`tavily-python`), `Chroma`, retrievers, and any API-backed tool must be constructed
  inside a lazy factory — use `@lru_cache(maxsize=1) def get_x(): ...` — never at module level.
- **Imports must be side-effect-free.** Importing any module (`enterprise_rag.graph.graph`, `enterprise_rag.graph.nodes.*`,
  `enterprise_rag.graph.chains.*`, `enterprise_rag.ingestion`) must NOT require API keys or network, and must NOT construct
  any external client.
- **Backward-compatible chain names.** Chain modules expose `get_*()` factories; old
  module-level names (e.g. `generation_chain`, `question_router`) remain available via a lazy
  module-level `__getattr__`. Don't reintroduce eager module-level chain objects.
- Code comments/docstrings are written in **English**.
- **`office_agent/` working rules.** The Office Agent is deterministic and local by design —
  keep it that way unless a task explicitly says otherwise:
  - **No LLM routing.** Classify intents with the existing keyword router + intent constants /
    typed schemas (`schemas.py`); do not introduce an LLM router.
  - **Two optional, default-off LLM assists only.** The sanctioned LLM paths outside
    Knowledge Q&A live in `office_agent/llm_assist/`: the email digest layered on the `email`
    tool ([ADR 017](docs/adr/office_agent/017-office-agent-llm-assist-email-digest.md), validated
    `EmailDigest` boundary) and the Daily Briefing narrative layered on the `briefing` tool
    ([ADR 018](docs/adr/office_agent/018-office-agent-llm-assist-daily-briefing.md), validated
    `BriefingNarrative` boundary over the pure `collect_briefing_facts()` fact set). Both are
    gated by the single `OFFICE_LLM_ENABLED` switch (default off, and forced off by either
    runtime privacy mode), single-pass structured
    output, deterministic grounding + `llm_assist_error` fallback, no action surface. Preserve
    each assist's **byte-for-byte flag-off** guarantee; adding LLM assistance to any other
    capability requires a new ADR. Follow the repo LLM pattern (lazy `@lru_cache` factory,
    `gpt-5-mini`, `temperature=0`, bounded timeout, side-effect-free imports) and import nothing
    from `enterprise_rag`.
  - **Knowledge Q&A goes through the existing adapter over `enterprise_rag`** — never duplicate
    retrieval/generation/graph logic inside `office_agent`.
  - **Mock tools stay local-only and deterministic.** Read `office_agent/mock_data/` as
    **read-only** and anchor dates to the data, **not the system clock**. **No external
    integrations** (Gmail, Google/Outlook Calendar, Slack, Jira, Linear, Asana, Trello) unless
    explicitly requested.
  - Tools return a `ToolResult`; `answer_office_request(user_input, options=None)` is the single
    entry point. Two additive, default-`None` response fields ([ADR 021](docs/adr/021-frontend-observability-workspace.md)):
    `observability` (`KnowledgeObservability` — set only by the Knowledge Q&A adapter) and
    `run_settings` (`ResolvedRunSettings` — set only when the caller passed `OfficeRunOptions`;
    resolved after routing, server policy wins).
  - Same discipline as `enterprise_rag`: **side-effect-free imports**, lazy data/client access.
  - **`office_agent` tests stay fully mocked / CI-safe** — no OpenAI, Tavily, Chroma, or external
    services, and no real `enterprise_rag` graph call (patch the knowledge adapter) and no real
    LLM-assist call (patch the email digest / briefing narrative at their seams). The real-model
    assist tests (one per assist) live in `tests/office_agent/integration/` (gated by `requires_openai`,
    like `tests/enterprise_rag/chains/`), never in the mocked `tests/office_agent/` unit suite.
- **Presentation-tier (`api/`, `frontend/`) working rules.** The web/HTTP surface is the
  repository-level presentation tier ([ADR 021](docs/adr/021-frontend-observability-workspace.md)).
  It adds **no** engine behavior and must never regress `enterprise_rag` or `office_agent`:
  - **`api/` is a thin adapter.** Its only engine call is
    `office_agent.engine.answer_office_request()`. Keep it to the existing two-route
    health/run boundary and do **not** duplicate routing, intent detection, formatting,
    privacy, fallback, date interpretation, or tool behavior. Preserve the factory pattern
    (side-effect-free import), and surface engine failures as HTTP 500 with the exception
    **type name only** (messages may carry paths/secrets).
  - **`frontend/` must not re-derive backend-owned behavior.** Backend-reported `effective`
    Run Settings and `observability` are authoritative — display them only from the response,
    never recompute or infer them (e.g. from the read-only server-policy badges), and never
    fabricate an execution timeline for a capability that did not report one. The `mock` and
    `http` clients share one `AgentClient` contract; keep them in lockstep.
  - **Request-scoped Run Settings resolve in `office_agent/run_settings.py`** (a pure function,
    server policy passed in as arguments), **after** routing. **Server policy wins:** a per-run
    request may only *restrict*, never weaken a server restriction. Resolution reads no env var
    and mutates no module global, so concurrent requests stay isolated — do not introduce
    `os.environ` writes or mutable module state on this path.
  - **Scope:** localhost-only demo surface — no auth, persistence, or deployment tooling unless
    a task explicitly asks. The `api` dependency group (`fastapi`/`uvicorn`/`httpx`) must be
    installed for `api/` and `tests/api/`.

## 4. Testing Rules

- **Unit tests mock all external dependencies** via `monkeypatch`, targeting the lazy seam
  (e.g. patch `get_node_retriever`, `get_web_search_tool`, `get_retrieval_grader`,
  `generate_answer`).
- **Node tests (`tests/enterprise_rag/nodes/`) must never call real OpenAI, Tavily, Chroma, or embeddings.**
  They must pass with no API keys.
- **Integration tests (`tests/enterprise_rag/chains/`) call real services** and require both the
  `RUN_REAL_MODEL_TESTS=1` opt-in and `OPENAI_API_KEY` — a key alone never authorizes a paid call.
  Label such tests clearly and gate them with the `requires_openai` marker from `conftest.py`
  (an alias of the canonical `real_model` marker).
- **Do not run tests unless explicitly asked.** Writing tests ≠ running them.

## 5. Claude Code Behavior Rules

- **Plan first.** Before changing files, explain the plan and list every file to be changed and why.
- **Summarize after.** After editing, provide a diff summary.
- **Don't run commands without explicit approval** — no `pytest`, `python -c`, `py_compile`, or
  any code-executing command unless the user asks. Provide commands for the user to run instead.
- **Stop and ask** before any change that may affect business logic, graph routing, prompt
  behavior, model behavior, or the state schema.
- **Tests-only tasks:** when the request is only to write tests, prefer asking before touching
  production code; make the smallest safe change if a seam is genuinely needed for testability.
- **Authoritative date for dated artifacts:** Before creating or updating any dated report or artifact, first run:

  `powershell.exe -NoProfile -Command "Get-Date -Format 'yyyy-MM-dd HH:mm:ss zzz'"`

  Treat the returned timestamp as the only authoritative current local time. Use its `YYYY-MM-DD` portion consistently in the filename, title, `Date:` or metadata field, and any generated-date text in the body. Never guess the current date or reuse a date from conversation history, Git history, existing reports, or filenames. Retrieve the timestamp once before the first write and reuse that same value throughout the artifact. If the command fails, stop and report the failure instead of writing with a guessed date.

## 6. Common Commands

> These are for **the user to run manually**. Claude Code should not execute them without approval.

```powershell
# Always work from the project root
cd "<your-local-repo-path>"

# Set up the environment (creates .venv, writes uv.lock). Include the api group:
# pytest collects tests/api/, so FastAPI/httpx must be installed, and mypy type-checks api/.
uv sync --group dev --group api

# Install local pre-commit hooks (one-time per clone)
uv run pre-commit install

# Build the Chroma index (one-time, before first run)
uv run python -m enterprise_rag.ingestion

# Run the Office Agent (main.py launches the Office Agent CLI; deterministic tools need no keys/index)
uv run python main.py
uv run python -m office_agent.cli

# Run the standalone Enterprise RAG CLI
uv run python -m enterprise_rag.cli

# Run the web presentation tier (ADR 021): thin FastAPI adapter + frontend workspace
uv run uvicorn api.app:create_app --factory --host 127.0.0.1 --port 8000

# Frontend (npm-managed, independent of uv; run from frontend/)
npm ci                              # or npm install
npm run dev                         # Vite dev server (proxies /api → 127.0.0.1:8000)
npm run build                       # tsc type-check + vite build
npm test                            # Vitest (jsdom)
npx playwright install chromium     # one-time per machine
npm run test:responsive             # Playwright responsive checks (typed mock mode)

# Node unit tests — fully mocked, NO API keys required
uv run pytest tests/enterprise_rag/nodes/ -v

# Mocked API adapter tests — keys-free, needs the api dependency group
uv run pytest tests/api/ -v

# Chain integration tests — real gpt-5-mini; skip unless RUN_REAL_MODEL_TESTS=1
# and OPENAI_API_KEY are both set
uv run pytest tests/enterprise_rag/chains/ -v

# Whole suite
uv run pytest -v

# Dev hygiene (mirrors the CI lint job — run before committing)
uv run ruff check .                  # lint
uv run ruff check --fix .            # lint + safe autofixes
uv run ruff format .                 # format
uv run ruff format --check .         # format check (CI mode)
uv run mypy                          # type-check scoped modules only

# Run all pre-commit hooks across every file (equivalent to what runs on commit)
uv run pre-commit run --all-files

# Syntax-only check (no test execution)
$files = @(
    "enterprise_rag/graph/graph.py",
    "enterprise_rag/graph/nodes/generate.py",
    "enterprise_rag/graph/nodes/retrieve.py",
    "enterprise_rag/graph/nodes/web_search.py",
    "enterprise_rag/graph/nodes/grade_documents.py",
    "enterprise_rag/graph/chains/generation.py",
    "enterprise_rag/graph/chains/retrieval_grader.py",
    "enterprise_rag/graph/chains/question_router.py",
    "enterprise_rag/graph/chains/hallucination_grader.py",
    "enterprise_rag/graph/chains/answer_grader.py",
    "enterprise_rag/ingestion.py",
    "enterprise_rag/cli.py",
    "office_agent/cli.py",
    "main.py"
)

uv run python -m py_compile $files

# Verify imports construct no clients and need no keys
uv run python -c "import enterprise_rag.graph.graph, enterprise_rag.graph.nodes, enterprise_rag.graph.chains, enterprise_rag.ingestion; print('IMPORT OK')"
```

---
> Source: [rhenus-Q/Enterprise-Office-Agent](https://github.com/rhenus-Q/Enterprise-Office-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
