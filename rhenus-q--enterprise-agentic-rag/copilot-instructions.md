## enterprise-agentic-rag

> This file is the durable source of repository guidance for Codex and other coding agents.

# Repository Guidelines

This file is the durable source of repository guidance for Codex and other coding agents.
It applies to the entire repository unless a nested `AGENTS.md` or
`AGENTS.override.md` provides more specific instructions.

## Project overview

This repository implements an enterprise internal-document Q&A assistant with
LangGraph. The self-correcting Agentic RAG workflow routes between local retrieval and
web search, grades document relevance, checks answer grounding and usefulness, and
caps retry loops.

The application stack is LangGraph, LangChain, Chroma, Tavily, and `uv`. OpenAI
(`gpt-5-mini` and `OpenAIEmbeddings`) is the default model provider; the optional
process-level local-provider mode uses `ChatOllama` and `OllamaEmbeddings` through an
Ollama-compatible endpoint. Read `README.md` for setup and usage and `structure.md` for
the detailed architecture map.

External dependency failures must not crash the graph. Retrieval, web search,
generation, graders, and query rewriting degrade or stop safely and record an honest
`stop_reason`. Console banners log exception types, not exception messages.
Deployment-mode and provider configuration is validated before the graph by
`main.py::run_startup_preflight()` so privacy-sensitive misconfiguration fails clearly.

## Important paths

- `main.py`: interactive CLI entry point and startup preflight for deployment modes,
  providers, and local endpoint/model/index compatibility.
- `graph/engine.py`: canonical programmatic API, state seeding, streaming merge,
  observability, and metadata-only traces.
- `graph/graph.py`: graph assembly, routing, and `MAX_RETRIES`.
- `graph/state.py`: `GraphState` schema.
- `graph/config.py`: privacy and deployment modes, provider selection, fallback policy,
  request timeout, and per-run budget settings.
- `graph/nodes/`: graph node implementations and terminal notice nodes.
- `graph/chains/`: lazy LCEL chain factories.
- `graph/chains/_llm.py`: shared provider-aware chat-model factory used by all six
  chains.
- `graph/formatting.py`: pure answer/source formatting.
- `ingestion.py`: corpus loading, splitting, embedding, and provider-scoped Chroma
  rebuilds. It resets the active collection, uses deterministic chunk ids, writes an
  embedding-fingerprint sidecar, and exposes a lazy process-cached retriever. A failed
  mid-ingestion rebuild can leave the active index empty until ingestion succeeds again.
- `server/`: thin FastAPI adapter over the engine, including API schemas, status and
  document views, metadata-only run history, error mapping, and cooperative cancellation.
- `frontend/`: Vite, React, and TypeScript UI. Its API types mirror `server/schemas.py`;
  colocated Vitest tests cover critical states.
- `data/acmecorp_internal_docs/`: synthetic corpus; it contains no real company data.
- `tests/node/`, `tests/graph/`, `tests/evals/`, `tests/server/`, and
  `tests/test_env_isolation.py`: keys-free tests collected by default CI. Pure generation
  helper and empty-context tests live in `tests/node/test_generation_context_delimiters.py`
  and `tests/node/test_generation_short_circuit.py`.
- `tests/chains/`: real-model integration tests requiring `OPENAI_API_KEY`; default CI
  excludes the directory. The current suite covers five chain modules and has no live
  query-rewriter integration test.
- `evals/`: behavioral evaluation harness; full runs call real services.
- `docs/adr/`: architecture decision records.
- `docs/roadmap/`: specifications, plans, implementation reports, and review reports.
- `.agents/skills/`: repository-specific Codex workflows.
- `.codex/config.toml`: trusted-project Codex configuration and MCP servers.

## Architecture and implementation rules

- Preserve behavior by default. Do not change graph routing, the `GraphState` schema,
  prompts, application model names, `temperature=0`, chain input variables, or node
  return structures unless the user explicitly requests it.
- Avoid broad architecture changes and wholesale rewrites. Prefer small, mechanical,
  reviewable diffs.
- Keep `GraphState` fields as plain last-value channels. Do not add
  `typing.Annotated` reducers or accumulating channels without first redesigning the
  `dict.update()` streaming merge in `graph/engine.py`.
- Construct `ChatOpenAI`, `OpenAIEmbeddings`, `ChatOllama`, `OllamaEmbeddings`,
  `TavilyClient` (`tavily-python`), `Chroma`, retrievers, and other API-backed clients
  only inside lazy factories. Cache them with `@lru_cache(maxsize=1)` only where
  process-lifetime reuse and cache invalidation are intentional. Import optional
  provider-specific classes inside their factory.
- Route all six chains through `graph/chains/_llm.py::get_chat_model()`. Provider
  selection is process-level, never per-chain or per-run; do not construct a provider
  client in an individual chain module.
- Keep imports side-effect-free. Importing application modules must not require API
  keys, access the network, or construct external clients.
- Preserve lazy backward-compatible chain names through module-level `__getattr__`;
  do not reintroduce eager module-level chain objects.
- Keep code comments and docstrings in English.
- Preserve privacy guarantees: traces remain metadata-only and must not contain
  document bodies, prompts, raw graph state, secrets, or API keys.
- Preserve deployment-mode precedence in `graph/engine.py::seed_state()`:
  `PRIVACY_MODE=true` and local-provider mode are absolute locks that disable web
  search and LangSmith export even when a per-run option requests them;
  `WEB_SEARCH_ENABLED=false` remains a per-run-overridable default.
- Local-provider failures must never fall back to OpenAI, Tavily, or LangSmith. The
  guarantee is no third-party egress; the configured Ollama-compatible endpoint is
  itself the trust boundary and may be private infrastructure rather than localhost.
- Keep `server/` a thin adapter. It may use `graph.engine`, `graph.config`,
  `graph.consts`, `graph.formatting`, `ingestion`, and `main`; it must not bypass that
  boundary by importing graph nodes or chains or by constructing external clients.
- Preserve the cancellation boundary from ADR 017: the server supplies
  `AnswerOptions.cancel_event`, the engine checks it at streamed node boundaries, and a
  cancelled run raises `RunCancelled` without a `stop_reason` or history record.
- Keep `server/schemas.py` and `frontend/src/api/types.ts` synchronized. Preserve URL
  scheme validation and sanitized error/config responses at the HTTP boundary; frontend
  rendering must not bypass those protections.
- Keep OpenAI and local Chroma indexes provider-scoped and preserve embedding
  fingerprint validation. Changing an embedding model requires rebuilding that
  provider's index; switching between existing matching indexes does not.
- When changing a documented architectural decision, update or supersede the matching
  ADR.

## Testing rules

- Mock every external dependency in unit tests at the lazy factory seam.
- Node, graph, eval-helper, server, and root-level tests must run without API keys and
  must never call real OpenAI, Tavily, Chroma, or embedding services.
- Mark real-service chain tests with `requires_openai` and keep their dependency on
  `OPENAI_API_KEY` explicit. Keep keys-free chain helper tests outside `tests/chains/` so
  the default CI collection runs them.
- Keep frontend tests colocated as `*.test.ts` / `*.test.tsx`. Frontend validation is
  TypeScript type-checking, non-watch Vitest, and the Vite production build.
- Keep mocked tests independent of a developer's deployment settings. The autouse
  fixture in `tests/conftest.py` clears mode/provider environment variables; tests that
  set them must use `monkeypatch` and clear every affected `@lru_cache` factory.
- Do not run tests, linters, type checks, compilation commands, the application, or
  evaluation commands unless the user explicitly asks. Writing tests does not imply
  permission to run them.
- Never run the full behavioral eval without explicit approval. `--validate-only` is
  lower risk but still requires permission under the preceding rule.

## Agent working agreement

- Before editing, state the plan and the files expected to change.
- After editing, summarize the resulting diff and any validation performed or omitted.
- Stop and ask before a change that may affect business logic, graph routing, prompt or
  model behavior, or the state schema unless the user already authorized that exact
  change.
- For tests-only requests, avoid production-code changes unless a minimal test seam is
  genuinely required.
- Preserve unrelated user changes and do not overwrite files outside the requested
  scope.
- Treat current sandbox, approval, and tool policies as authoritative. A Skill may
  narrow behavior but never grants permissions beyond the active environment.
- Run tests, evals, ingestion, the application, networked commands, and real-service
  or API-backed operations only when the user explicitly authorizes them. A narrower
  workflow prohibition still applies after authorization.
- Use the configured `docs-langchain` MCP server only for version-sensitive external
  LangChain or LangGraph documentation. Repository-local guidance, code, tests, ADRs,
  and roadmap artifacts remain authoritative.
- Never write credentials, API keys, authentication state, user-level Codex
  configuration, or model-provider settings as part of a repository workflow.

## Repository Skills

Invoke a Skill explicitly with its `$name`, or allow Codex to select it when the task
matches its description.

- `$eval-imple`: evaluate whether a requested change is justified, then implement only
  the smallest justified change.
- `$imple-spec`: implement an approved specification or plan.
- `$new-function-spec`: create an implementation-ready function specification.
- `$review-diff`: review the current working-tree changes without editing files.
- `$arch-review`, `$security-review`, `$failure-modes-review`,
  `$test-coverage-review`, and `$docs-drift-review`: perform focused audits and write
  report artifacts only.
- `$apply-review-report`: apply scoped findings from a project-level review report.
- `$review-skill` and `$apply-skill-review`: review and maintain Codex Skill
  definitions.
- `$update-agents-md`: record durable repository guidance after a completed change.

Each Skill's `SKILL.md` defines its input and execution boundary. Its
`references/workflow.md` contains the detailed project-specific procedure.

## Common commands

The following commands are documentation for the user or for explicitly authorized
validation. Do not execute them automatically.

```powershell
uv sync --group dev
uv run pre-commit install
uv run python ingestion.py
uv run python main.py
uv run pytest tests/ --ignore=tests/chains/ -q
uv run pytest tests/node/ -v
uv run pytest tests/server/ -v
uv run pytest tests/chains/ -v
uv run pytest -v
uv run ruff check .
uv run ruff check --fix .
uv run ruff format .
uv run ruff format --check .
uv run mypy
uv run pre-commit run --all-files
cd frontend
npm run typecheck
npx vitest run
npm run build
```

---
> Source: [rhenus-Q/Enterprise-Agentic-RAG](https://github.com/rhenus-Q/Enterprise-Agentic-RAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
