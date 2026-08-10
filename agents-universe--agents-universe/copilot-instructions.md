## agents-universe

> Full-stack enterprise AI agent framework running in Linux containers: autonomous planning, multi-LLM providers (Anthropic / OpenAI / Azure OpenAI / Google Gemini), on-demand knowledge loading, Codex-style web UI.

# Agents Universe — Enterprise AI Agent Framework

## Project Overview

Full-stack enterprise AI agent framework running in Linux containers: autonomous planning, multi-LLM providers (Anthropic / OpenAI / Azure OpenAI / Google Gemini), on-demand knowledge loading, Codex-style web UI.

**Core principle:** project knowledge loads in full on project selection. No embedding model — context via MD cross-references (`[[slug]]`).

## Monorepo Layout

- `packages/agent-core/` — Python 3.12 LLM orchestration engine (pure library, no HTTP)
- `packages/api/` — Python 3.12 FastAPI web service: auth, DB, WebSocket
- `packages/web/` — TypeScript + Vue 3 Codex-style browser UI
- `agents/` — agent definitions (`*.agent.md`) and skills (`skills/**/*.md`)
- `knowledge/` — global framework knowledge base (system/, technical/, _template/)
- `workflows/` — workflow definitions (`*.workflow.md`); agent reads and follows

## Development Commands

```bash
# API (from packages/api/)
PYTHONPATH=src python -m uvicorn api.main:app --port 8000

# Frontend (from packages/web/)
npm run dev

# Run knowledge indexer for a project
python -m agent_core.knowledge.index --project {slug}

# DB migrations (from packages/api/)
alembic upgrade head
alembic revision --autogenerate -m "description"

# Docker local stack
docker compose up
docker compose up --build
```

## Key Conventions

1. **Token security** — Encrypted tokens are never logged, printed, or included in error messages. AES-256-GCM in `token_vault.py`.
2. **Knowledge cross-links** — `[[slug]]` inside Markdown files; resolved to `knowledge_id` at index time.
3. **Project context loading** — `knowledge/loader.py` two-tier model on project selection: primary files load in full from disk; `knowledge_level: detail` files are indexed in DB (metadata + summary only) and loaded on demand via `knowledge_rw`.
4. **DB primary keys** — Always `UNIQUEIDENTIFIER DEFAULT NEWID()`. Never `IDENTITY`.
5. **SQL Server driver** — Use `mssql+aioodbc`. Never `pymssql`.
6. **Project isolation** — Knowledge queries always scope to `project_id = :current OR project_id IS NULL`. No cross-project queries.
7. **Agent definitions** — Markdown frontmatter + body in `agents/*.agent.md` (global, synced at startup). **Project-scoped agents** live in `{PROJECTS_ROOT}/{slug}/agents/{project_slug}--{name}.agent.md` with matching `skills/` and `workflows/` dirs; lazily synced to the `agents` table (column `project_id`), selectable only within their project, shadowing global skills/workflows of the same slug at runtime. Created/deleted via the 智能体定制专家 conversation (file write/delete + lazy sync) — no restart needed. Models are NOT defined per-agent: configured in Settings → AI Models (`user_model_configs` table), one per conversation. `model_low/mid/high` on the `agents` table and `complexity.py` are legacy — no complexity-based routing at runtime.
8. **Skill types** — `guidance` (LLM instructions), `template` (code templates), `executable` (runnable code blocks), `composite` (chains other skills).
9. **Workflow definitions** — Same format as skills; files end in `.workflow.md`. No YAML engine — agent reads and executes.
10. **Image outputs** — Stored in `{PROJECTS_ROOT}/{slug}/.tmp/media/{conversation_id}/`; served via `/api/media/` with JWT auth. Never stored as DB blobs.
11. **Secret management** - Two-tier encrypted storage: `user_tokens` (per-user, cross-project), `project_secrets` (per-project); both AES-256-GCM in `token_vault.py`. `secret_vault` manages the user vault (list/save/delete); `api_request` resolves secrets via `secret_ref`/`secret_refs` + `secret_scope` (project->user fallback). Secret prompts (`user_confirm` / `api_request`) use `save_to_project_secrets` or `save_to_user_tokens` (mutually exclusive) - plaintext never reaches the LLM. Secrets never stored in `personal_memories` (`memory_rw` rejects them). Keys never put in URL query parameters.

## Sub-project Workspace Structure

`PROJECTS_ROOT` is a **required** env var pointing to an external directory (outside the repo). Sub-projects created via `POST /api/projects` get an isolated workspace:

```
{PROJECTS_ROOT}/
└── {slug}/
    ├── agents/        ← project-scoped agent definitions (*.agent.md)
    ├── skills/        ← project-scoped skills (shadow global same-slug)
    ├── workflows/     ← project-scoped workflows (shadow global same-slug)
    ├── knowledge/      ← initialized from knowledge/_template/
    ├── tests/          ← QA agent generates test scripts here
    └── .tmp/
        ├── media/{conversation_id}/   ← screenshots and generated images
        └── work/                      ← temporary working files
```

## Architecture Decisions

- **Python for agent-core + API**: Richer AI/ML ecosystem (all LLM SDKs).
- **No embedding model**: knowledge loaded in full per project; no vector search, no sentence-transformers dependency.
- **CodeMirror 6 in Composer**: Multi-line Markdown input with `@mention` and `/command` support.
- **Completeness score denormalized**: stored in `knowledge_metadata` to avoid recomputation on every read.
- **Redis for session + active users**: OAuth session keyed `session:{session_id}` (TTL=24h), active users tracked via sorted set.
- **Agentic Loop**: `plan_task` tool triggers structured task planning; tasks tracked in `agent_tasks` DB table.

## Files Never to Commit

- `.env` — all secrets
- `.claude/settings.local.json`
- Anything under `PROJECTS_ROOT` — sub-project workspaces are user data, not application code

## Model Routing

- 分析代码或编写代码时，通过 Agent 工具指定 `model: "opus"` 或切换到 Opus 子代理执行

## Testing Strategy

- `agent-core`: pytest + pytest-asyncio; mock LLM providers via recorded VCR cassettes.
- `api`: pytest + httpx AsyncClient; test DB uses SQLite in-memory (SQLAlchemy 2.x is dialect-agnostic for most queries).
- `web`: Vitest + Vue Test Utils.

---
> Source: [agents-universe/agents-universe](https://github.com/agents-universe/agents-universe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
