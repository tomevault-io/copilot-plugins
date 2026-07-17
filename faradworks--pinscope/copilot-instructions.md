## pinscope

> Pinscope validates hardware schematics against component datasheets. It extracts constraints from PDFs, parses netlists and BOMs into a queryable graph, and runs an agentic validation loop to flag design violations.

# Pinscope — Agentic Schematic Validation

Pinscope validates hardware schematics against component datasheets. It extracts constraints from PDFs, parses netlists and BOMs into a queryable graph, and runs an agentic validation loop to flag design violations.

> **Open-core note.** This is the open-source core. A small set of files are
> "gateway-owned seams" — pass-through stubs here (`frontend/src/proxy.ts`,
> `use-optional-auth.ts`, `clerk-theme-provider.tsx`,
> `components/billing/*`, `sidebar-auth.tsx`, `pricing-section.tsx`,
> `analytics/*`, `lib/csp-hosts.ts`) that the hosted-cloud repo replaces
> with auth/billing implementations. Keep their export signatures stable,
> and never import auth/billing SDKs anywhere else in the frontend. On the
> backend, everything reaches billing only through
> `backend/services/billing_hook.py:get_billing()` (a no-op here).

## System Overview

Three layers:

| Layer | Location | Purpose |
|-------|----------|---------|
| **Core library** | `backend/pinscopex/` | Models, parsers, graph builder, agentic validator, passive resolver, taxonomy, BOM summary, derating |
| **Backend** | `backend/` | FastAPI app — async pipeline orchestration, SSE progress, project/file storage |
| **Frontend** | `frontend/` | Next.js 16 app — project dashboard, pipeline progress, report viewer, derating, admin dashboard |

Plus `skills/` — Claude Console Skills for datasheet extraction (pintable, patterns, specs).

The pipeline stages: Parse BOM → Extract IC Pintables → Extract Simple Components → Extract Passives → DigiKey Auto-Resolve + Value Fallback → Build Graph → Direct Datasheet Review. Pipeline runs can be cancelled mid-execution via `POST /api/pipeline/{id}/cancel`.

## Example Project

`simple_project/` is the reference design for development and testing:

- **MCU**: TI MSPM0G3507SPTR (U3) — 48-pin LQFP
- **USB-UART Bridge**: CH340E (U2)
- **LDO Regulator**: SPX3819M5-L-3-3 (U1) — 5V to 3.3V
- **ESD Protection**: USBLC6-2SC6 (D1)
- **Crystal**: 8 MHz (X1) with 18pF load caps (C9, C10)

Files: `.asc` (PADS-PCB netlist; `.edn` EDIF 2.0.0 also accepted), `.csv`/`.xlsx` (BOM), `design_graph.json` (committed reference fixture used by tests).

## Architecture Principles

- **Modular extractors** — Domain-specific extraction per component type, unified constraint schema
- **Netlist as graph** — Queryable bipartite graph (components + nets) with traversal helpers
- **Claude API for PDF extraction** — Forced tool calls for structured output (pintable, passive patterns, specs)
- **Prompt caching** — Extraction and review API calls use `cache_control={"type": "ephemeral"}` on system prompts and input context to reduce cost on repeated calls
- **Claude Console Skills** — Extraction prompts deployed as managed skills; skill_ids and versions loaded from `backend/skills_manifest.json` (upload your own via `scripts/upload_skills.py`)
- **Direct datasheet review** — Claude reads the IC datasheet PDF and circuit neighborhood together, compares to reference application circuit, and flags issues via graph query tools (`find_connected_components`, `get_net_for_pin`, `get_pintable`)
- **Datasheet page trimming** — Large PDFs are keyword-trimmed to relevant pages before sending to Claude, reducing token cost (`pypdf`)
- **DigiKey fallback (exact MPN only)** — When pattern-based and direct extraction fail, DigiKey API fetches product parameters for auto-resolve. DigiKey matches only on exact MPN; fuzzy hits are rejected to avoid polluting the shared library with wrong-dielectric / wrong-voltage parts.
- **Value-string fallback** — When DigiKey misses an R/C/L/FB passive, a value-string resolver maps the BOM `Value` string to typed passive specs. Value-derived specs are persisted per-project only — never to the shared library.
- **Per-IC review error isolation** — Direct datasheet review runs each IC independently; one malformed payload or bad response cannot kill the whole run. Failed ICs surface as skipped components with the error.
- **Cross-IC excerpt budget (per-neighbor)** — To verify an interface finding the reviewer can pull a *connected* IC's datasheet pages (`get_datasheet_excerpt`). The budget is a global per-review page ceiling **plus a per-neighbor sub-budget**, so verifying one interface is never starved by pages already spent on other neighbors.
- **Finding normalization is downgrade-only** — A post-review per-IC normalize pass (`services/normalize_findings.py`) drops self-cancelling findings, merges same-root-cause findings, and re-grades severity — but only ever *downward*. A deterministic clamp caps each finding at the reviewer's calibrated severity (and any `Unverified:` finding at WARNING, preserving the prefix).
- **Cross-IC finding dedup** — After all per-IC reviews complete, a single pass (`services/dedupe_findings.py`) collapses one physical interface defect reported from both endpoints into a single finding. Gated by `cross_ic_dedup_enabled`; fail-soft.
- **Capacitor voltage derating** — Deterministic derating table computed from graph (ceramic/tantalum/electrolytic percentages, pass/fail per capacitor)
- **Deterministic checks over heuristics** — Exact checks where possible
- **Zero coupling between layers** — Backend calls pinscopex functions with paths; frontend talks to backend via REST + SSE
- **Library deduplication** — Shared library (`library/extracted/`, `library/patterns/`, `library/models/`, `library/passives/`, `library/datasheets/`) caches extractions across projects
- **Content-addressed datasheets** — `library/datasheets/blobs/{md5}.pdf` stores unique PDFs once; `library/datasheets/refs/{safe_mpn}.json` maps MPNs to blobs (dedupe + multi-MPN sharing)
- **Taxonomy-driven extraction** — Living component taxonomy (`taxonomy/`) with per-subtype classification and specs schemas
- **Per-stage model config** — Each pipeline stage can use a different Claude model (e.g., Sonnet for review, Haiku for auto-resolve)
- **API call logging** — Every Claude API call is logged with token counts, cost, and timing per pipeline run
- **Report versioning** — Each project run is stamped with the current app version on the first `/start` transition (`ProjectMeta.pinscope_version`). The version comes from `frontend/content/changelog.md`'s latest `##` heading — single source of truth — read at backend startup via `backend/_version.py`.

## Datasheet Extraction

Extracted data lives in `library/extracted/` (shared) or per-project under the storage backend. One JSON per MPN, schema in `backend/pinscopex/models.py`.

Per-MPN IC extraction captures:
1. **Pintable** — Pin number + name (required), description + alt functions (optional)
2. **Package info** — Base family, package, pin count, description
3. **Component subtype** — Dotted taxonomy path (e.g., `ic.mcu`, `ic.power.ldo`)

For discrete/simple components:
4. **Specs** — Component specs (value, tolerance, package, voltage rating, etc.); parameters are filtered against taxonomy specs schemas

Extraction uses **Claude Console Skills** (required, via `skill_id` in `backend/skills_manifest.json`). No inline fallback — raises error if skill not configured. Skills are defined in `skills/` and uploaded via `scripts/upload_skills.py` — run it once against your own Anthropic Console account to populate the manifest with your skill IDs.

## Claude Console Skills

```
skills/
├── extract-pintable/    # Pin table + package info + taxonomy
│   ├── SKILL.md         # System prompt (YAML frontmatter + markdown)
│   ├── schema.json      # Tool output schema
│   └── validate.py      # Validation script
├── extract-pattern/     # Passive MPN pattern
└── extract-specs/       # Component specs (discrete, connectors, crystals, etc.)
```

## Taxonomy

Living component taxonomy in `taxonomy/` — one JSON file per top-level type (ic, passive, connector, crystal, discrete, fuse, switch, test_point, transformer). Each subtype entry includes `description` and `example_mpn`.

Key taxonomy features:
- **Ref prefix mapping** — `U→ic`, `R/C/L→passive`, `D/Q→discrete`, `X→crystal`, etc.
- **Dotted subtype paths** — e.g., `ic.mcu`, `passive.capacitor.ceramic`, `ic.protection.esd`
- **Dynamic growth** — `add_subtype()` adds new entries; concurrent-safe JSON writes
- **Specs schema auto-generation** — Type-level and subtype-level parameter specs schemas are auto-generated via Claude when a taxonomy entry has none; extraction discards parameters not in the schema (`extra_specs` field)

## Scripts

- `scripts/upload_skills.py` — Create, update, or list Claude Console Skills. Reads/writes skill IDs to `backend/skills_manifest.json`
- `scripts/migrate_datasheets_to_library.py` — One-time migration: copy per-project datasheets to `library/datasheets/` (dry-run by default, `--apply` to execute)
- `scripts/migrate_datasheets_to_blobs.py` — Migrate named-PDF datasheets into the content-addressed blobs/refs layout (dry-run by default, `--apply` to execute)
- `scripts/dedup_library_datasheets.py` — Remove redundant per-MPN datasheet PDFs when a passive pattern already has a `datasheet_key` (dry-run by default, `--apply` to execute)
- `scripts/gc_orphan_blobs.py` — Garbage-collect `library/datasheets/blobs/*.pdf` not referenced by any ref file
- `scripts/clear_rules_from_extractions.py` — Strip deprecated `rules`/`absolute_maximum_ratings` from existing library extractions

## Tech Stack

- **Core**: Python 3.12+, Pydantic 2.x, Anthropic SDK (async + sync), openpyxl (XLSX BOM support), pypdf (datasheet page trimming)
- **Backend**: FastAPI, uvicorn, sse-starlette, pydantic-settings
- **Frontend**: Next.js 16 (App Router, Turbopack), React 19, Tailwind CSS v4, shadcn/ui (Base UI), react-pdf
- **AI**: Claude API with forced tool calls for extraction, direct datasheet review for validation
- **Model**: `claude-sonnet-4-6` default for extraction and review, `claude-haiku-4-5` for DigiKey auto-resolve and passive value fallback (per-stage overrides via `.env`)
- **Skills**: Claude Console Skills API for managed extraction prompts (3 active skills: pintable, pattern, specs)
- **External APIs**: DigiKey API v4 (OAuth2) — optional datasheet auto-fetch and parameter-based auto-resolve (`DIGIKEY_CLIENT_ID`, `DIGIKEY_CLIENT_SECRET`)

## Extracted Model Versioning

All `ComponentConstraints` extracted JSON files carry a `model_version` semver field:

- **Initial value** — set from `default_model_version` in `backend/skills_manifest.json` (starts at `1.0.0`)
- **Minor bump** — `default_model_version` in `skills_manifest.json` is incremented by `scripts/upload_skills.py --update`, so all new extractions after a skill update start at the new minor (e.g. `1.0.0` → `1.1.0`)

**Rule**: When committing or pushing changes under `skills/`, run `python3 scripts/upload_skills.py --update` before the commit/push to sync skill versions and bump `default_model_version`.

## Development Guidelines

- Write tests against `simple_project/` — it's the ground truth
- Netlist parser and BOM parser are pure functions with no side effects
- All data structures use Pydantic models in `backend/pinscopex/models.py`
- Frontend types in `frontend/src/lib/types.ts` must stay in sync with `backend/pinscopex/models.py`
- Extraction prompts live in `skills/` as Claude Console Skills (SKILL.md + schema.json + validate.py)
- **Never swallow exceptions silently** — prefer logging or re-raising over bare `except: continue`. Silent failures hide real bugs.

## Running

```bash
# Backend (copy backend/.env.example to .env at repo root first)
python3 -m uvicorn backend.main:app --reload    # localhost:8000

# Frontend
cd frontend && npm run dev                       # localhost:3000
```

Local mode needs no cloud services and no auth — projects are stored in `data/` and you are `user_id="local"` with admin access.

---
> Source: [Faradworks/Pinscope](https://github.com/Faradworks/Pinscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
