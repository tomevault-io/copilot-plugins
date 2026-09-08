## 2026-1-contradito

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ContraDito** is a political transparency **consultation portal** that cross-references Brazilian federal legislators' speeches against their voting records using AI. It is an academic project for the MDS discipline at UnB (Universidade de Brasília).

> Product note: the **Score de Coerência** and the `dados_insuficientes` flag are **no longer surfaced in the product**. The `score_coerencia` column still exists in the database as dead/internal data, but no ranking, badge, ordering, or score is exposed in the UI. The `eh_coerente` column is **no longer populated** (permanent data limitation), so no coherence-derived metric exists anywhere. The product is a **consultation portal**, not a coherence ranking.

## Running the Project

All services run via Docker Compose. Requires a `.env` file at the root:

```env
SUPABASE_URL=...
SUPABASE_KEY=...
LLM_PROVIDER=groq   # "groq" or "colab"
GROQ_API_KEY=...
NEXT_PUBLIC_API_URL=http://localhost:8001
```

```bash
docker compose up --build
```

- API docs (FastAPI Swagger): http://localhost:8001/docs
- PostgreSQL: localhost:5432

> On Apple Silicon (ARM64): ensure `platform: linux/arm64` is set in `docker-compose.yml`. The first build downloads the HuggingFace model (~2.3GB) and caches it in a Docker volume.

## Linting (CI enforced on PRs to `develop` and `main`)

```bash
# Format check
black --check app/

# Lint (critical errors only)
flake8 app/ --count --select=E9,F63,F7,F82 --show-source --statistics

## Architecture

The system is a strict **CQRS** pattern split into two isolated Python services:

### Read Side — `app/` (FastAPI, port 8000)
- Entry point: `app/main.py`
- Routes: `app/rotas/politicos.py` and `app/rotas/logs.py`
- Pydantic schemas: `app/modelos/schemas.py`
- Supabase client: `app/bancos/supabase.py`
- Uses `fastapi-cache2` with `InMemoryBackend` (1h TTL on list endpoints)
- Rate-limited via `slowapi` (5/min on semantic search)
- CORS is locked to `http://localhost:3000`

### Route structure — `{casa}` segment

All public read routes are now namespaced by **casa** (`"camara"` or `"senado"`)
in the path, **except** `GET /api/comparar` (which takes `casa` as a query param).
Implemented in `app/rotas/dados.py` (router `prefix="/api"`); `validar_casa` rejects
anything other than `camara`/`senado` with HTTP 400.

- `GET /api/{casa}/politicos` — list/filter (busca, partido, estado, pagina, tamanho)
- `GET /api/{casa}/politicos/{id_parlamentar}` — profile (`politico` + `resumo_votos`)
- `GET /api/{casa}/politicos/{id_parlamentar}/timeline` — voting timeline
- `GET /api/{casa}/politicos/{id_parlamentar}/afinidades` — gêmeo / antípoda (concordance)
- `GET /api/{casa}/politicos/{id_parlamentar}/fidelidade` — raw party fidelity
- `GET /api/{casa}/politicos/{id_parlamentar}/discursos` — speeches of one legislator
- `GET /api/comparar` — direct comparison (concordância + divergências)
- `GET /api/{casa}/partidos/coesao` — party cohesion (adapted Rice index)
- `GET /api/{casa}/proposicoes[/{id_proposicao}[/polarizacao]]` — propositions
- `GET /api/{casa}/discursos[/{discurso_id}[/chunks]]` — speeches
- `GET /api/{casa}/votos` — raw nominal votes (+ nearest speech chunks, may be empty)

None of these routes read or return `eh_coerente`. The `politicos` payload still
includes `score_coerencia`, but it is dead data for the product (never rendered).

### Write Side — `worker_api.py` (FastAPI, port 8001, internal only)
- Entry point: `worker_api.py`
- Exposes a single route: `POST /gerar-embedding`
- NLP engine: `utils/motor_nlp.py` — wraps `sentence-transformers` model `paraphrase-multilingual-mpnet-base-v2` (768-dimension vectors)
- Port 8001 is **not exposed externally** (Docker-internal only); the API calls it via `http://worker:8001`

### Database — Supabase (PostgreSQL + pgvector)
Tables are mirrored per casa (`camara_*` / `senado_*`). Core tables:
- `{casa}_politicos` — legislators. Still has a `score_coerencia` column, but it is
  dead data for the product (never shown).
- `{casa}_votos` — nominal votes (`voto_oficial`, `partido_na_epoca`). The
  `eh_coerente` column is **no longer populated** (permanent data limitation).
- `{casa}_proposicoes`, `{casa}_discursos`, `{casa}_discurso_chunks` — propositions,
  speeches and AI-mapped chunks.
- `politico_resumo_votos` — consolidated vote tallies (Sim/Não/ausências/etc.).
- Vector similarity search uses `pgvector` over the speech chunks.

In local Docker, the DB is `pgvector/pgvector:pg15` on port 5432 (exposed for DBeaver/pgAdmin inspection).

### Frontend — `frontend/` (Next.js 16 + React 19 + Tailwind CSS 4)
- App Router under `frontend/app/`
- Consultation portal (no ranking). Pages (under restructuring — see `PLANEJAMENTO.md`):
  `/` (Home — vitrine/preview: hero, storytelling, real numbers, search + Casa selector
  preview, "Sobre"/team section), `/diretorio` (full unified Câmara+Senado listing with
  Todos/Câmara/Senado view modes, fetch-all-once — ADR 003), `/politico/[id]` (dossiê),
  `/comparacao` (1×1 comparison by vote — concordância).
- No score, nota, ranking ordering, or `dados_insuficientes` rendered anywhere.
- Design system: Fraunces (display) + DM Sans (body); palette/tokens in
  `frontend/app/globals.css`.
- Communicates with the API at `NEXT_PUBLIC_API_URL`; routes namespaced by `{casa}`.

---
> Source: [unb-mds/2026.1-ContraDito](https://github.com/unb-mds/2026.1-ContraDito) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
