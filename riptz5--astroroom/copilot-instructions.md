## astroroom

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

AstroRoom is a provenance-aware, evidence-first astronomy workspace for Sagittarius A* analysis. FastAPI backend (26 routers, 39 services) + React 19/Vite frontend. The mounted UI follows a calm stage-based investigation flow: Session Entry, Astronomical Asset Gallery, Build Sets, then downstream placeholders for Process, Analyze, Evidence, and Export. Licensed PolyForm Noncommercial 1.0.0.

## Governing Taglines

> **"No construyan más amplitud. Construyan cierre, trazabilidad y convergencia."**

> **"Toda imagen importante debe poder explicarse hacia atrás, paso por paso, hasta sus FITS."**

Before writing any code, verify:
- [ ] Does this serve the evidence question for Sgr A* activity across epochs?
- [ ] Does it have a canonical entity or does it create a duplicate?
- [ ] What artifact does it produce to disk?
- [ ] What intermediate can be inspected for debugging?
- [ ] Does it follow the input → intermediate → output → provenance chain?
- [ ] Is it processing, visual, or evidence product?
- [ ] Is the claim language calibrated to evidence, not physics?
- [ ] Does the timeline drive this, or is it decorative?
- [ ] Does it read from session context, not maintain shadow state?
- [ ] Is this new, or does similar logic already exist that should be adapted?

## Audit Infrastructure

Use `backend/app/services/audit/` for all scientific computations:
- `AuditLogger`: timestamps, SHA-256 hashing, parameter logging
- `sha256_array()`, `sha256_file()`: integrity verification
- `log_feature_matrix_csv()`: export features to CSV for verification

Reference implementation: `flares_service.py` (FDR, permutation tests, bootstrap CI with audit).

## Commands

```bash
# Backend
pip install -r requirements.txt
uvicorn backend.main:app --reload --port 8000

# Frontend
cd frontend && bun install
VITE_API_BASE_URL=http://localhost:8000 bun run dev

# Docker (full stack with PostgreSQL)
docker compose up --build

# Docker (SQLite-only, lightweight)
docker compose -f docker-compose.yml -f docker-compose.sqlite.yml up --build

# Docker (with Fermi science toolchain — heavy)
docker compose --profile fermi up --build

# Backend tests
pytest tests_backend -q                        # all
pytest tests_backend/test_pipeline.py -q       # single file
pytest tests_backend/test_fits.py::test_name -q  # single test
pytest tests_backend --cov=backend/app --cov-report=term-missing  # with coverage

# Frontend lint & build
cd frontend && bun run lint && bun run build

# Playwright E2E (auto-launches backend+frontend)
cd tests && bunx playwright install chromium   # required first-time setup
cd tests && bun install && bunx playwright test
ASTROROOM_NO_WEBSERVER=1 bunx playwright test  # skip auto-launch

# Scientist CLI (autonomous 6-phase analysis pipeline)
python astro_scientist.py --help

# Database migrations
alembic upgrade head
alembic revision --autogenerate -m "description"
alembic current
alembic history
```

## Architecture

### Backend Layers

```
backend/main.py                    ← App entrypoint, registers all 26 routers
backend/app/config.py              ← Settings (pydantic-settings), storage paths
backend/app/core/database.py       ← SQLAlchemy engine, Base, SessionLocal, get_db()
backend/app/core/seed.py           ← Auto-seeds default CompatibilityProfile + 7 InstrumentProfiles
backend/app/models/db_models.py    ← 12+ SQLAlchemy models (the schema of record)
backend/app/models/*.py            ← Pydantic request/response models per domain
backend/app/api/*.py               ← 26 FastAPI routers (thin: validate → call service → return)
backend/app/services/*.py          ← 39 service modules (all business logic lives here)
backend/app/evidence/*.py          ← Unified evidence engines and schema exports
backend/app/scientist/             ← Autonomous analysis agent (see below)
backend/app/agents/                ← Fermi/GCE specialized agent components
```

Request flow: **Router** (api/) validates input via Pydantic → calls **Service** (services/) → reads/writes **DB Models** (models/db_models.py) via `get_db()` dependency → returns Pydantic response.

The pipeline router (`api/pipeline.py`) is the largest — it implements all 7 phases and uses **SSE streaming** (`StreamingResponse` with `text/event-stream`) for long-running operations like epoch frame building and full pipeline runs.

### Frontend Layers

```
App.tsx → SessionProvider            ← Sovereign session store
  → AstroWorkspace                   ← Active orchestration surface (no shadow session truth)
    → SessionEntryStage              ← Investigation entry, recent sessions, target selection
    → AssetGalleryStage              ← SessionAsset inventory, filters, inspector, working tray
    → BuildSetsStage                 ← WorkingSet suggestions, compatibility reasoning, warnings
    → WorkflowStagePlaceholder       ← Process / Analyze / Evidence / Export placeholders
```

`AstroWorkspace.tsx` is now a store-driven stage orchestrator. `WorkflowStudio.tsx`, `LightroomShell.tsx`, and the old panel-first flow are legacy reference material only. Rescue logic from them when useful, but do not restore them as the mounted truth.

Key frontend files:
- `src/context/SessionContext.tsx` — Session source of truth (`SessionAsset`, `WorkingSet`, stage state, filters, evidence context)
- `src/components/AstroWorkspace.tsx` — Investigation hydration and stage orchestration
- `src/api/index.ts` — Re-exports from modular API files (`core.ts`, `evidence.ts`, `fermi.ts`, `fits.ts`, `flares.ts`, `objects.ts`, `pipeline.ts`, `referenceOutputs.ts`, `schemas.ts`, `zoomLadder.ts`)
- `src/types.ts` — Shared TypeScript contracts and canonical frontend session types
- `src/hooks/usePipelineProgress.ts` — Polls pipeline status for progress bar updates

### Database

12 SQLAlchemy models in `db_models.py`: `SourceAsset`, `EpochBucket`, `EpochAssetLink`, `EpochFrame`, `DerivedProduct`, `AnalysisRun`, `ReportArtifact`, `EvidenceRun`, `EvidenceRecord`, `CompatibilityProfile`, `InstrumentProfile`, `AgentMemory`.

- IDs are truncated UUIDs (12 chars).
- `render_as_batch=True` in Alembic for SQLite ALTER TABLE support.
- On startup, `init_db()` calls `create_all` (dev), then `seed_defaults()` inserts the default Sgr A* compatibility profile and 7 instrument profiles if missing.
- When modifying models, always create an Alembic migration.

### 7-Phase Pipeline

1. **Phase 0** — Source Census: `/pipeline/assets/register`
2. **Phase 1** — Compatibility Engine: `/pipeline/compatibility/run` (configurable profiles with astrometric, photometric, WCS, semantic thresholds)
3. **Phase 2** — Epoch Construction: `/pipeline/epochs/create-buckets` + `/pipeline/epochs/build-frame`
4. **Phase 3** — Time-Series Products: `/pipeline/time-series/render` (timelapse from epoch frames)
5. **Phase 4** — Mega-Exposure Stack: `/pipeline/mega-exposure/build`
6. **Phase 5** — Volumetric 3D: `/pipeline/volumetric/build`
7. **Phase 6** — PDF Reporting: `/pipeline/reports/generate`

### Scientist Subsystem (Autonomous Analysis Agent)

`backend/app/scientist/` is a recently added autonomous 6-phase analysis pipeline with its own agent, types, and ML training/inference modules:

- `agent.py` — Main orchestrator; runs inference using trained models on pipeline assets
- `types.py` — Shared types for the scientist subsystem
- `training/` — Training scripts: CNN visual source classifier (CAS morphology), Chronos time-series anomaly analyzer (t5-tiny), FermiNet multi-class classifier
- `report/` — JSON + Markdown report generator for scientist findings
- `models/` — Persisted model checkpoints (not committed)

Entry point: `astro_scientist.py` at repo root (CLI wrapper). The scientist runs its own 6-phase pipeline on top of AstroRoom's pipeline outputs and produces structured ML-based evidence reports. Tests: `test_agent_orchestrator.py`, `test_chronos_analyzer.py`, `test_visual_classifier.py`, `test_ferminet_multiclass.py`, `test_synthesis_models.py`, `test_scientist_types.py`, `test_report_generator.py`.

### Agents Subsystem (Fermi/GCE)

`backend/app/agents/` contains specialized components for Fermi-LAT and GCE analysis:
- `base.py`, `catalog.py`, `context.py` — Base classes and shared catalog context
- `diffuse_model.py`, `exposure.py` — Diffuse background modeling and exposure map handling
- `fermi_ingest.py` — Fermi event data ingestion
- `subhalo/` — Subhalo detection sub-pipeline

### Zoom Ladder / Multi-Band Matrix (Primary Output)

The zoom-ladder system generates the "zoomizoomout" image: a large matrix where rows = pure bands (one survey per row, never mixed) and columns = 20 log-uniform scales from 0.01° to 180°.

**Service:** `backend/app/services/zoom_ladder_service.py`

- **SURVEYS registry**: 10 bands — 2MASS J/H/K, WISE 12/22, IRAS 100µm, DSS optical, NVSS radio, Planck 857GHz
- **`build_zoom_matrix()`**: Full N-survey × M-scale matrix + 3 extra rows (median stack, RGB composite, statistics per scale)
- **`build_maximum_zoom_out()`**: 5-panel strip from nucleus (0.3°) to quarter-sky (90°)
- **`build_zoom_ladder()`**: Backward-compat wrapper (3 bands only: 2MASS K, WISE 12, IRAS 100)
- Data source priority: local corpus stacking → cached downloads → SkyView on-demand
- Smart survey routing: for FOV > 10°, slow surveys (2MASS, DSS, NVSS) fall back to Planck 857

**Router:** `backend/app/api/zoom_ladder.py` — only exposes `/build` (legacy 3-band). The `/matrix` and `/maximum` endpoints for the full system are not yet wired.

**Quick script:** `scripts/leandro_zoom_kwi_ladder.py` (pass `--quick` for shorter runs) drives log-spaced zoom-in/out steps; `gradual_zoom_ladder_geometry` is the key function. For large FOV downloads use longer HTTP timeouts.

**Frontend:** `frontend/src/api/zoomLadder.ts` — only has `build()` for the legacy endpoint.

### Spatial Context API

`backend/app/api/spatial.py` — multiscale navigation endpoints:
- `GET /spatial/context/{asset_id}` — FOV, pixel scale, zoom level, hierarchy
- `GET /spatial/ladder/{object_name}` — ordered rungs from widest to narrowest FOV
- `GET /spatial/cone-search` — assets within angular radius, with FOV filtering
- `GET /spatial/hierarchy/{object_name}` — assets grouped by zoom level
- `POST /spatial/link-hierarchy` — establish parent/child spatial relationships

Zoom levels: `ultra-close` (<0.5°) → `close` (0.5°-2°) → `medium` (2°-10°) → `wide` (10°-30°) → `ultra-wide` (>30°)

### Data Ingestion & FITS Corpus

**Current state: FITS storage is typically empty at startup.** All data is fetched on-demand from SkyView or MAST.

Ingestion paths:
- `POST /fits/ingest` — single URL (SkyView, MAST, or `file://` local)
- `POST /fits/ingest-local` — scan a local directory
- `POST /pipeline/corpus/ingest` — bulk from `data/fits/` directory
- `GET /objects/{id}/discovery` — archival discovery (MAST, SkyView, Chandra, production bins)
- `naco_service.py` — ESO/NACO query (requires `ESO_USERNAME` authentication)

**Object catalog** (`backend/app/data/objects.json`): 5 objects defined (sgra, m87, m42, crab, m31). Only Sgr A* has `sample_fits_urls` (8 SkyView URLs: 2MASS J/H/K, DSS Red/Blue, WISE W1/W2, ROSAT).

**Bulk corpus scripts** (in `scripts/`):
- `corpus_unified_master_fits.py` — builds unified master FITS from a corpus; always pass `--out` with a real writable path (e.g. `outputforleandro/unified_master`) and `--roi-ra 266.416817 --roi-dec -29.007825 --roi-radius-deg 3.5` when the corpus includes wide-field tiles (otherwise union WCS collapses to coarse scales)
- `embed_corpus_dinov2.py` — DINOv2 embeddings over the corpus
- `segment_corpus_sam2.py` — SAM2 segmentation pass
- `leandro_full_corpus_figures.py`, `regenerate_reference_figures_for_leandro.py` — figure regeneration against `Reference/previousimages/`

**Output directories** (not committed, written by scripts/services):
- `outputforleandro/` — zoom ladders, embeddings, segmentation, corpus manifests, hero figures
- `stacked_output/`, `volumetric_output*/` — stacking and volumetric pipeline outputs

### Evidence Engine

- `backend/app/evidence/engine.py` — legacy `2.9` logic (**planned for retirement**, see convergence plan P0.2)
- `backend/app/evidence/engine_v295.py` — calibrated `2.95` engine, **used by default**
- `backend/app/services/evidence_service.py` — runtime bridge; the version-switching logic here should be removed once v2.9 is retired
- `/evidence/*` persists `EvidenceRun` and `EvidenceRecord` rows and exports downloadable artifacts

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `BASE_STORAGE_PATH` | `/tmp/astrooom` | Root for all persisted data (production: `/data/astroroom`) |
| `DATABASE_URL` | SQLite at `{BASE_STORAGE_PATH}/astroroom.db` | PostgreSQL: `postgresql+psycopg://user:pass@host:5432/astroroom` |
| `ALLOWED_ORIGINS` | `["http://localhost:5173"]` | CORS origins |
| `VITE_API_BASE_URL` | (empty) | Frontend → backend base URL |
| `ASTROROOM_NO_WEBSERVER` | unset | Set to `1` to skip Playwright auto-launching dev servers |
| `MAX_STORAGE_GB` | `80` | Disk safety cap |

Storage sub-directories (`fits/`, `asdf/`, `cubes/`, `epochs/`, `mega_exposures/`, `reports/`, `evidence/`, `volumetric/`, `timelapse/`, `zoom_ladder_runs/`, `fermi_mocks/`, `fermi_analysis/`) are auto-created from `BASE_STORAGE_PATH`.

## Testing

- **Backend tests** (`tests_backend/`): 19 test files using `pytest-asyncio` + `httpx.AsyncClient` with `ASGITransport` (in-process, no real server). The `conftest.py` provides `synthetic_fits_bytes` and `synthetic_fits_url` fixtures. Scientist subsystem tests (`test_agent_orchestrator.py`, `test_chronos_analyzer.py`, `test_visual_classifier.py`, `test_ferminet_multiclass.py`, `test_synthesis_models.py`, `test_scientist_types.py`, `test_report_generator.py`) are the newest additions. `test_evidence.py` has a known bug: `test_blinding_enforcement` is defined three times — pytest only runs the last definition (planned fix in P0.2).
- **Playwright E2E** (`tests/e2e/`): 13 spec files. Config auto-launches backend+frontend. `sgra_full_audit.spec.ts` is inside `testDir` and **does** run by default. Some specs target older selectors — compare against current components before editing.
- Backend tests are the dependable safety net; Playwright specs are more fragile and require `bunx playwright install chromium` before first run.

## Editing Guidance

- Keep `frontend/src/api/index.ts` and `frontend/src/types.ts` aligned with backend Pydantic models and router responses. The frontend API is modular — changes to a domain's contract go in the matching `src/api/<domain>.ts` file, then re-exported from `index.ts`.
- Keep the Fermi session contract aligned across `api/fermi.py`, `models/fermi.py`, `services/fermi_real_service.py`, `api.ts`, and `types.ts`.
- Router files should stay thin — put logic in services.
- Use absolute imports under `backend.app`.
- Use strict TypeScript; extend `types.ts` and `api.ts` together when contracts change.
- When touching routes or UI workflow, update `README.md`, `AGENTS.md`, and this file.
- Prefer describing implemented behavior over research aspirations in docs.
- Imaging deliverables: prefer clean stretches, minimal NaN/edge holes, and gradual multi-scale zoom ladders; include wide context out to ~180° when producing SkyView-style ladder figures. Avoid seams, patchy backgrounds, and artifact noise in final images.
- `Reference/previousimages/` is the visual target for figure parity checks.

## Golden Rules (Enforced)

See `ASTROROOM_GOLDEN_RULES.md` for the complete rule set. Key principles:

1. **Single canonical session unit**: Mounted frontend work converges on `SessionAsset`; `Frame` is a compatibility alias for older image-centric consumers until they are migrated.
2. **Evidence-first**: AstroRoom is a GC evidence system, workbench is infrastructure.
3. **Auditability mandatory**: Every feature must produce artifacts + intermediate + provenance.
4. **No final-image-only**: Stack, reproject, cube, RGB, diff, extraction all need input/intermediate/output/provenance.
5. **Audit before reimplement**: Check `WorkflowStudio`, old stages, unmounted backend, sleeping panels first.
6. **Three levels**: Every artifact is `processing` | `visual` | `evidence`.
7. **No inflated claims**: "Consistent with" not "truth", "suggests" not "causes".
8. **Timeline commands**: If epoch exists, timeline affects viewport, selection, analysis.
9. **Context is source**: `SessionContext` is source of truth, not a late mirror.
10. **Hero figures after audit**: Reproducibility > beauty.

**Quick test**: "Can I trace this image backwards to its original FITS?"

## Known Gaps & Active Convergence

See `ASTROROOM_CONVERGENCE_PLAN.md` for the full 3-phase, 22-task plan. Key items:

- **Evidence engine fork** (P0.2, Critical): `engine.py` (v2.9) and `engine_v295.py` (v2.95) coexist, violating Rule 1. Plan: delete v2.9, rename v2.95 → `engine.py`, remove version-switching from `evidence_service.py`.
- **`test_evidence.py` triple-defined test** (P0.2): `test_blinding_enforcement` appears 3 times; only the last runs. Fix by giving each a unique name.
- **Playwright suite was broken** (P0.1): Missing Chromium binary. Fix: `cd tests && bunx playwright install chromium`.
- **Zoom router is behind zoom service**: `build_zoom_matrix()` and `build_maximum_zoom_out()` exist in the service but `/matrix` and `/maximum` endpoints are not wired in the router.
- **Root-level scripts bypass canonical pipeline**: `run_real_pipeline.py`, `batch_fits_downloader.py`, `generate_*_atlas.py` duplicate or circumvent `backend/app/api/pipeline.py`. These are utility scripts, not the production path.
- **FITS corpus is empty at dev startup**: For productive development, run `/objects/sgra/discovery` then ingest results.
- **`Reference/bom/`** contains an unrelated audio research project — do not treat as astronomy reference material.
- **4 of 5 catalog objects have no sample_fits_urls**: Only Sgr A* has pre-configured SkyView URLs.
## Strict Discovery Vetting

Use `POST /fits/discovery-candidates` when the task is "is this object real?"
rather than simple per-frame source extraction.

This endpoint is the current strict workflow baseline:

- requires multiple FITS IDs
- rejects or downgrades single-frame detections
- scores edge proximity, cosmic contamination, PSF plausibility, and photometric stability
- runs SIMBAD vetting by default and supports extra VizieR/HEASARC crossmatch through `astroquery`
- does not allow a candidate to remain `candidate` if catalog vetting failed or was incomplete
- returns `blocked` instead of `rejected` when the remaining problem is catalog availability rather than science

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/riptz5) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
