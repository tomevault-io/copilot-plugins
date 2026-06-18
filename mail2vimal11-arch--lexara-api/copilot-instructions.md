## lexara-api

> AI-powered contract analysis engine for Canadian legal professionals.

# Lexara API — Session Playbook

AI-powered contract analysis engine for Canadian legal professionals.
Live at **lexara.tech / api.lexara.tech**. Ontario / PIPEDA-focused.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Backend | FastAPI 0.109 + Python 3.11 |
| Database | PostgreSQL 16 + SQLAlchemy 2.0 ORM |
| Migrations | Alembic (`alembic revision --autogenerate -m "..."` → `alembic upgrade head`) |
| Cache | Redis 7 |
| Auth | JWT (python-jose) + bcrypt via `app/security.py` |
| LLM | Groq → HuggingFace/SaulLM → Claude (waterfall, see below) |
| Search | FAISS (`app/nlp/`) + Sentence Transformers `all-MiniLM-L6-v2` |
| Billing | Stripe |
| Infra | Docker Compose + Traefik (SSL) on a single VPS |
| CI/CD | GitHub Actions: test on push, deploy to VPS on merge to `main` |

---

## Project Layout

```
app/
  routers/      ← thin HTTP layer only — no business logic here
  services/     ← all business logic lives here
  models/       ← SQLAlchemy ORM: User, Tender, SOWClause, Audit, Account
  middleware/   ← auth.py (API key + JWT validation)
  nlp/          ← FAISS index, embeddings, text pipeline
  ingestion/    ← TED / OCP tender ingestion pipeline
  config.py     ← all settings via Pydantic BaseSettings / env vars
  security.py   ← JWT creation/validation, password hashing
```

---

## Architecture Rules

1. **Routers are thin.** HTTP in → call one service function → HTTP out. No DB queries, no LLM calls inside routers.
2. **All config via env vars.** `app/config.py` is the single source. Never hardcode keys, URLs, or feature flags.
3. **Migrations via Alembic only.** Never `ALTER TABLE` manually in prod (except the one-time stamp workaround — see Deploy section).
4. **LLM waterfall order:** `use_groq=true` → Groq first; `use_local_llm=true` → HF/SaulLM second; Claude always last. Both flags default `False` — without them set in `.env`, every request hits Claude (paid). Confirm `USE_GROQ=true` is set in prod.
5. **Jurisdiction:** Ontario law / PIPEDA. Don't add generic "Canadian" logic that ignores provincial variation.
6. **No contract text stored by default.** PIPEDA compliance. Don't add persistence of contract content without explicit design review.

---

## Frozen / Sensitive — Don't Touch Without Review

- `app/middleware/auth.py` — auth bypass was fixed in `25fdee4`; any change here needs a security review
- `app/security.py` — JWT and password logic
- Stripe price IDs in `app/config.py` — changing these breaks live subscriptions
- `alembic/versions/` — never edit a committed migration file; always create a new one

---

## LLM Tier Reference

```
USE_GROQ=true        → activates Groq tier (llama-3.1-8b-instant, free)
USE_LOCAL_LLM=true   → activates HF/SaulLM tier (Equall/Saul-7B-Instruct-v1 or fine-tune)
HF_MODEL_ID=...      → which HF model to call (no default; set to base or fine-tuned)
```

Prompt formats differ per tier — don't mix them:
- Groq: OpenAI chat format (system + user messages)
- HF/SaulLM: Alpaca format (`### Instruction / ### Input / ### Response`)
- Claude: Anthropic Messages API

---

## Current State (updated 2026-05-02)

**Highest priority — billing system incomplete:**
- `app/routers/billing.py` has 6 TODOs: Stripe webhooks received but plan changes not written to DB
- `app/routers/usage.py:30` returns hardcoded mock data instead of DB query
- **Do not onboard paying users until these are resolved**

**Recently shipped:**
- Groq as primary free inference tier (`832dff3`)
- SaulLM/HF as secondary tier (`36aa414`)
- Automated deploy via GitHub Actions SSH (`tests.yml`)
- HF warmer interval: 600s (was 240s)
- 429 retry in `hf_llm_service.py`
- Model attribution fix: response `model` field now uses `settings.hf_model_id`

**Known technical debt:**
- No circuit breaker on LLM tiers (failing tier retried every request)
- Rate limiting (slowapi) configured but enforcement unverified
- `APIKeyMiddleware` depends on undocumented DB query path

---

## Deploy Workflow

```bash
# Automated: push to main → GitHub Actions runs tests → SSH deploy
# Manual fallback:
cd /opt/lexara-api
git pull origin main
docker compose pull
docker compose restart api
```

**One-time prod gotcha (first deploy with new migrations):**
```bash
# Run once manually before the automated deploy if alembic versions diverge:
alembic stamp head
```

**GitHub Secrets required for automated deploy:**
`VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY`, `VPS_PORT` (optional, defaults 22)

---

## Session Conventions

- **Branch naming:** `feature/<session-letter>-<kebab-task>` (e.g. `feature/a-billing-webhooks`)
- **Commit style:** Conventional Commits — `feat:`, `fix:`, `chore:`, `docs:`, `test:`
- **Never push to `main` directly.** PRs only; tests must pass.
- **Never run `alembic downgrade`** without explicit human approval.
- **After completing work:** update `PROJECT_STATUS.md` and run `/pre-merge-check` before opening a PR.

---
> Source: [mail2vimal11-arch/lexara-api](https://github.com/mail2vimal11-arch/lexara-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
