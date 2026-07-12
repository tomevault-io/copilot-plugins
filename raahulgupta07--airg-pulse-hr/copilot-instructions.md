## airg-pulse-hr

> Other agents (Claude, Cursor, Copilot, devs) read this file first. Skim takes 30s. Useful work in 5min.

# AGENTS.md — Read this before touching code

Other agents (Claude, Cursor, Copilot, devs) read this file first. Skim takes 30s. Useful work in 5min.

---

## What this app is (one line)

**PULSE — multi-tenant HR/talent/comms platform.** FastAPI + SvelteKit 5 + PG18+pgvector. Docker Compose local, ECS Fargate prod-ready.

**Acronym:** People · Updates · Lifecycle · Sourcing · Engagement.

**Brand:** brutalist/industrial. Dark `#383832` ink, yellow `#feffd6` surface, green `#00fc40` CTA, zero border-radius, hard stamp shadows.

---

## Read these in order (priority)

1. `AGENTS.md` (this file) — orientation
2. `ARCHITECTURE.md` — one-page diagram + data flows
3. `CLAUDE.md` — full feature map, env vars, security, debug recipes
4. `compose.yaml` — runtime topology
5. `backend/main.py` — entry, middleware, health
6. `backend/core/` — DB, auth, OCR, LLM, settings, migrations
7. `backend/routes/` — 22 route modules (one per feature)
8. `frontend/src/routes/` — page-per-folder
9. `db/migrations/` — additive-only SQL (sha256-tracked)
10. `tests/` — pytest

If you read 1–4 and the relevant route file, you can ship a feature.

---

## Hard rules (do not break)

- **Migrations are ADDITIVE ONLY.** Guard in `backend/core/migrations.py` blocks DROP COLUMN, DROP TABLE, RENAME COLUMN, ALTER COLUMN TYPE, TRUNCATE. Override flag exists (`ALLOW_DESTRUCTIVE_MIGRATIONS=1`) — never use without explicit user approval.
- **Secrets NEVER in code.** `.env` only. Bcrypt hash on disk, not plaintext.
- **All `{@html}` sinks must escape user input first** (`mdLite`, `renderNoteWithMentions` already do).
- **LLM calls go through `llm_call()`** in `backend/core/config.py` (timeout + cost cap + LLM_GATE semaphore). Never call OpenAI client directly.
- **File access via signed URLs** (`/candidates/{id}/file/sign`) — not bare `?token=`.
- **Auth:** JWT-first via `validate_token`, legacy hex fallback for old tokens. Both live side-by-side.
- **Frontend Svelte 5 Runes only** (`$state`, `$effect`, `$derived`). Never legacy stores or `export let`.
- **Brutalist design only.** Zero border-radius (forced via `* { border-radius: 0 !important }`). Ink borders 2px, stamp shadows `4px 4px 0`, no blur.

---

## How to run locally

```bash
cp .env.example .env   # already populated for dev
docker compose up -d --build
# wait ~6s
open http://localhost:8090/login
# login: pulse_admin / PulseAdmin#2026!
```

Health: `curl http://localhost:8090/api/health`

---

## How to add a feature without breaking data

1. New SQL migration in `db/migrations/NNN_<name>.sql` — **additive only** (CREATE TABLE / ADD COLUMN / CREATE INDEX). Auto-runs on next boot.
2. Route handler in `backend/routes/<area>.py` — register router in `backend/main.py`.
3. Frontend page in `frontend/src/routes/<area>/+page.svelte`.
4. Test in `tests/test_<area>.py`.
5. `docker compose up -d --build api`.

---

## How to NOT break data on upgrade

```bash
./scripts/pre_upgrade.sh                    # snapshot DB + /data/cvs
git pull
docker compose up -d --build
# verify
curl -s http://localhost:8090/api/health | head -c 200
# rollback if broken:
./scripts/restore.sh <dump-file> --confirm
```

Backup sidecar already runs daily (`pg_dump`, 14d retention, `data/backups/`). Symlink `latest.dump.gz` always points to newest.

---

## Concurrency tuning (current: 10 users)

All in `.env`:

| Var | Value | Why |
|-----|-------|-----|
| `WORKERS` | 4 | uvicorn worker processes |
| `DB_POOL_MIN` | 5 | per-worker min |
| `DB_POOL_MAX` | 20 | per-worker max (4×20=80 < 100 PG cap) |
| `OCR_CONCURRENCY` | 2 | PaddleOCR parallel jobs (RAM cap) |
| `LLM_MAX_CONCURRENT` | 8 | OpenRouter calls per worker |
| `LLM_DAILY_CAP_USD` | 200 | per-tenant daily $ cap |
| `RATE_LIMIT` | 600/minute | global default; per-route stricter |
| `OCR_UPSCALE_MIN_PX` | 2000 | low-res image upscale target before OCR |
| `OCR_UPSCALE_MAX_PX` | 4000 | upscale ceiling |
| `REDIS_URL` | `redis://redis:6379/0` | embed (24h) + tool-result (60s) cache; in-mem fallback if down |
| `AGENT_V2` | `false` | master flag for Chat v2 (Agno agent). False = legacy keyword chat |
| `AGNO_MAX_STEPS` | 8 | max tool-loop iterations per agent turn |
| `AGNO_TOOL_TIMEOUT_S` | 15 | per-tool timeout |
| `AGNO_MEMORY_TOPK` | 5 | pgvector recall hits injected per turn |
| `AGENT_SESSION_CAP_USD` | 1 | per-session $ ceiling (read from `agent_runs.cost_usd`) |

To scale to 50 users: add pgbouncer + Redis cache + ECS multi-task. Terraform modules already exist in `deploy/aws/terraform/`.

---

## LLM models (via OpenRouter)

| Slot | Model | Use |
|------|-------|-----|
| `CHAT_MODEL` | `google/gemini-3-flash-preview` | chat, vision, structure step |
| `DEEP_MODEL` | `openai/gpt-5.4-mini` | complex reasoning |
| `LITE_MODEL` | `google/gemini-3.1-flash-lite-preview` | classify, enrich, quality, tag |
| `VERIFIER_MODEL` | `anthropic/claude-opus-4.7` | digit-precision verifier (Step 2.5) |
| Embeddings | `gemini-embedding-2-preview` | 1536-dim |

All overridable via env. Set in `backend/core/config.py`.

---

## Where features live (cheat sheet)

| Feature | File |
|---------|------|
| Login UI | `frontend/src/routes/login/+page.svelte` |
| Auth backend | `backend/routes/auth.py` + `backend/core/jwt_auth.py` |
| CV upload | `backend/routes/ingest.py` → `backend/core/cv_pipeline.py` |
| OCR (Eng + Burmese + Zawgyi) | `backend/core/cv_parser.py` |
| Vision verifier (Step 2.5) | `backend/core/cv_parser.py` (`verify_critical_fields`) |
| Matching engine (7 dims) | `backend/routes/matching.py` |
| HR Brain chat (SSE) | `backend/routes/chat.py` |
| Settings runtime | `backend/core/settings.py` + `backend/routes/settings.py` |
| Storage adapter (Local/S3) | `backend/core/storage.py` |
| AWS Secrets/SSM loader | `backend/core/aws_secrets.py` |
| Migration safety | `backend/core/migrations.py` |
| Background CV pipeline | `backend/core/cv_pipeline.py` (13 steps) |
| Sync worker | `backend/agents/sync.py` |
| Cost cap + LLM gate | `backend/core/config.py` (`LLM_GATE`), `backend/core/cost_cap.py` |
| Rate limit | `backend/core/rate_limit.py` |
| Pipeline trace UI | `frontend/src/lib/PipelineCli.svelte` |
| Profile split-view | `frontend/src/routes/candidates/[id]/+page.svelte` |
| Image enhancement (pre-OCR upscale) | `backend/core/cv_parser.py` (`enhance_image_for_ocr`) |
| Billing dashboard backend | `backend/routes/billing.py` (admin/superadmin only) |
| Billing dashboard frontend | `frontend/src/routes/billing/+page.svelte` |
| LLM ledger table | `db/migrations/031_llm_call_log.sql` |
| Feature flags | `system_flags` table; toggle via `/admin` → SYSTEM tab |
| Nav defaults | `frontend/src/routes/+layout.svelte` `NAV_ALL` array |
| Agent loop (Chat v2) | `backend/agents/hr_agent.py` (`arun()` async gen) |
| Agent session + $ cap | `backend/agents/session.py` (writes `agent_runs`) |
| Agent memory (pgvector recall) | `backend/agents/memory.py` (reads/writes `agent_memory`) |
| Tool providers (11 tools) | `backend/agents/providers/{candidate,position,brain,analytics,email}_provider.py` |
| Agent eval gate (avg≥4.0) | `backend/agents/eval/{golden.jsonl, run_eval.py}` |
| Tool-trace UI | `frontend/src/lib/{ToolTrace,AgentSteps}.svelte` |
| Redis cache (embed + tool) | `backend/core/{cache,embed_cache,tool_cache}.py` |

---

## OCR routing (per language)

| Lang | Step 1 (Paddle) | Step 2 (Vision) | Step 3 (Tesseract) | Normalize |
|------|----------------|-----------------|---------------------|-----------|
| English | ✅ first | fallback | last | no-op |
| Burmese Unicode | ⏭ skipped (no model) | primary | `mya+eng` last | no-op |
| Zawgyi | ⏭ skipped | primary (LLM outputs Unicode) | `mya+eng` last | converts via myanmartools+ICU |
| Shan/Mon/Karen/Kayah/Pa'O | ⏭ skipped | primary | `mya+eng` last | pass |

Lang detected via `_peek_image_language()` (cheap LITE Vision call) before routing.

**Pre-OCR enhancement:** every image passes through `enhance_image_for_ocr()` first — upscale to ≥2000px (LANCZOS), auto-contrast, sharpen, brightness boost. Skipped when already crisp. Vision-handwritten path uses ORIGINAL (not enhanced) to preserve stroke nuance.

## Feature flags (nav visibility)

`system_flags` table (key `feature_*`). Frontend reads `/api/system/features` (public). Superadmin toggles via `/admin` → SYSTEM tab → FEATURES.

**Defaults:** Positions / JD Repo / CV Repo / Analytics / HR Brain = ON. **Interviews + Pools = OFF** (migration 032). Superadmin re-enables per-flag via SHOWN/HIDDEN toggle.

---

## 13-step CV pipeline

```
CLASSIFY → EXTRACT → VERIFY (2.5, optional) → SCREENSHOTS → STRUCTURE
→ ENRICH → SAVE → KNOWLEDGE → HyPE_EMBED → CONTEXT_EMBED
→ QUALITY → TAG → AUTO_MATCH
```

Each step traced in `pipeline_trace` table (run_id, step_order, model, status, latency, cost). Step 13 (AI_SUMMARY) isolated — failure ≠ upload failure. UI streams events from `/candidates/pending?include_recent=true`.

---

## Chat v2 — Agno agent (flag-gated, default OFF)

`backend/routes/chat.py` branches on `AGENT_V2` env. `false` → legacy SSE keyword path (untouched). `true` → `backend/agents/hr_agent.py:arun()` async generator with 11 tools, tool-trace SSE events, and `X-Chat-Version: 2` response header.

**11 tools:** `query_cvs`, `get_candidate_brief`, `score_cv_vs_position`, `list_candidate_pipeline`, `query_positions`, `get_position_brief`, `get_pipeline`, `query_brain`, `update_brain` (write), `query_funnel`, `draft_email`.

**Role allowlist:** recruiter blocked from `update_brain`; analyst blocked from `draft_email`; admin/superadmin = all. Enforced in tool dispatch.

**PII redaction** on logged tool inputs: `national_id`, `dob`, `phone`, `email` masked before write to `tool_traces`.

**Persistence:** migrations 033 (`agent_runs`), 034 (`agent_memory` pgvector), 035 (`tool_traces`). Memory recall via cosine search (`AGNO_MEMORY_TOPK=5`), embed query cached 24h in Redis.

**Cost:** every agent LLM call writes `llm_call_log` with `step='agent'` → billing dashboard auto-aggregates. Per-session cap = `AGENT_SESSION_CAP_USD`.

**Eval gate:** `python -m backend.agents.eval.run_eval` (30 golden cases). CI fails if mean rubric score < 4.0.

**Enable per-tenant:** flip `AGENT_V2=true` in `.env`, `docker compose up -d --force-recreate api`. Rollback = flip back to `false` and recreate; tables remain harmless.

**Container RAM:** api service bumped 6G→8G in `compose.yaml` for agent loop + Redis client.

**Runtime fallback:** Agno SDK not in local pip yet — shim in `hr_agent.py` falls back to direct OpenRouter tool-loop with same SSE event contract. Native after Docker rebuild pulls `agno`.

---

## Known caveats (don't fix unless asked)

- Health JSON has stray control char (col 521); strict parsers fail, lenient ones work.
- Upload routes write `/data/cvs` directly — storage adapter wired but unused. Switch when going S3.
- Anthropic verifier model ID `claude-opus-4.7` uses dotted format; may 404. Verify against current Anthropic naming (real format uses dashes + date suffix) before relying.
- `dev@hire.local` legacy admin row sits in users table with empty hash — can't login, ignore.
- DB user is `hire` (not `pulse`) — renaming postgres role needs superuser; not worth the cost.
- `RUN_MIGRATIONS_ON_BOOT=true` not gated to leader task; advisory lock saves multi-task race but consider gating later.
- Terraform `fmt`/`validate` not run in this sandbox; verify locally before first AWS apply.
- After Aurora bootstrap: `CREATE EXTENSION vector;` once.

---

## Rebuild + verify (one-liner)

```bash
docker compose up -d --build && sleep 6 && curl -s http://localhost:8090/api/health | head -c 200
```

Should return JSON starting with `{"status":"ok"`.

---

## Common debug recipes (already in CLAUDE.md, gist below)

- **Frontend changes not appearing after rebuild** → CSS bundled by content hash; if hash unchanged, force `--no-cache`. Tailwind v4 strips comments — grep for raw rule body, not the comment.
- **Svelte 5 `effect_update_depth_exceeded`** → effect reads-then-writes same state. Wrap callee in `untrack`, or defer write via `queueMicrotask`.
- **Login 401 INVALID_CREDENTIALS** → bootstrap row missing `email` + `display_name` (legacy NOT NULL cols). Fix in `backend/routes/auth.py:bootstrap_superadmin()`.
- **Bcrypt hash with `$` chars not loading** → use `env_file:` not compose-yaml `environment:` (compose substitutes `$$`).
- **Health endpoint code 200 but JSON parse fails** → strip control chars: `tr -d '\000-\037'`.

---

## What this agent should NOT do

- Don't add features the user didn't ask for.
- Don't refactor working code "for cleanliness".
- Don't add error handling for impossible cases.
- Don't write multi-paragraph docstrings or block comments.
- Don't create new `.md` files unless explicitly requested.
- Don't run destructive git/db ops without confirmation.
- Don't bypass migration guard or LLM cost cap.
- Don't touch `/login` styling — it's intentionally dark, isolated via `isPublicRoute`.

---

## Default credentials (dev only)

- `pulse_admin` / `PulseAdmin#2026!`
- Stored as bcrypt hash in `.env` (`SUPERADMIN_PASS_HASH`).
- Rotate by hashing new password (`docker exec pulse-api python -m backend.scripts.hash_pw <new>`) and updating `.env`, then `docker compose up -d --force-recreate api`.

---
> Source: [raahulgupta07/airg-pulse-hr](https://github.com/raahulgupta07/airg-pulse-hr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
