## staffing-optimizer

> FastAPI web app (`webapp/`) over a pure numpy staffing-equilibrium engine (`staffing_optimizer/`).

# Staffing Optimizer — Claude Code guide

FastAPI web app (`webapp/`) over a pure numpy staffing-equilibrium engine (`staffing_optimizer/`).
No build step, no CDN — JS libraries are vendored in `webapp/static/vendor/`. Designs persist as
timestamped JSON files in the gitignored `designs/` folder (no database; ephemeral on Render).

## Commands

- Run the app: `python -m webapp` → http://localhost:8000 (HOST/PORT env vars override)
- Tests: `python -m pytest -q` · Lint: `python -m ruff check .`
- Install: `pip install -e ".[web,dev]"`
- CLIs: `python solve.py <scenario.yaml>` (headless report) · `python design.py <file.flow>` (validate/diagram)

## Architecture

Design-document JSON ↔ `webapp/adapters.py` ↔ `DepartmentNetwork` (the engine). `webapp/api.py`
only delegates — it contains no math. Compute endpoints (`/api/analysis`, `/api/simulate`) are
gated: invalid designs get HTTP 422 with diagnostics via `_load_valid`. Never re-implement engine
math in `webapp/` or in JS.

The living program map is at `/docs/map`, backed by `webapp/static/docs/program-map.json`.
Swagger is at `/api/docs` (NOT `/docs` — that URL space belongs to the documentation pages).

## Documentation maintenance rules

- `webapp/static/docs/program-map.json` is the machine-readable source of truth for the program
  map. `tests/test_webapp.py::test_program_map_matches_registered_routes` enforces a two-way match
  with the live routes — **adding/removing/renaming any route requires updating this file** (the
  `endpoints` and `pages` arrays, plus `updated`).
- Docs pages live in `webapp/templates/docs_*.html` (+ `_docs_nav.html`). `/docs/guide` is the
  task walkthrough; `/help` is the concept reference — keep that split.
- `/docs/canonical` renders the output of `adapters.analysis_json` live — schema changes there
  require updating `docs_canonical.html` and `static/js/docs_canonical.js`. The pinned canonical
  example (`CANONICAL_EXAMPLE_ID` in app.py) must keep a `headcount` and a root department — the
  template assumes both exist.

## Codebase dependency map (docs/)

`tools/codemap.py` (a stdlib-only dev tool, outside the app packages) statically analyzes the
Python via `ast` and regenerates the top-level `docs/` folder: `docs/CODEMAP.md` (the agent-facing
dependency index — **read this to find who-imports-what instead of re-deriving the tree**),
`docs/codebase/*.html` (human-readable), and `docs/codemap.json`. **`docs/` is generated — never
hand-edit it; run `python tools/codemap.py`.** A second `Stop` hook
(`.claude/hooks/gen_codebase_docs.py`) regenerates it automatically when Python changes, and
`tests/test_codemap.py` fails if the committed output is stale (`python tools/codemap.py --check`).
The runtime/HTTP layer is folded in from `program-map.json` (hand-authored; front-end JS is served,
not imported, so it isn't in the Python graph).

## Review & automation rules

A `Stop` hook (`.claude/hooks/check_docs_review.py`) fires when files under `webapp/`,
`staffing_optimizer/`, `tests/`, or the CLIs changed during a session. When it blocks:

1. Launch the **doc-maintainer** and **code-reviewer** subagents in parallel (both are defined in
   `.claude/agents/` as Opus at medium reasoning effort).
2. If the reviewer reports missing test coverage, follow up with the **test-author** subagent.
3. Summarize their results, then stop. Do not bypass or disable the hook.

The hook records a state hash in `.claude/.last-review-state` (gitignored) so it only re-triggers
when the tree changes again. A second, silent `Stop` hook (`gen_codebase_docs.py`, see the
codebase-map section above) runs alongside it — it never blocks; it just refreshes `docs/`.

## Deployment (Render)

`render.yaml` deploys the free tier: build `pip install ".[web]"`, start
`uvicorn webapp.app:app --host 0.0.0.0 --port $PORT`, health check `/healthz`, Python pinned
3.11.4. Render sets `RENDER=true`, which turns on the ephemeral-storage banner in `base.html` —
keep that mechanism working. The filesystem is ephemeral: never design features that assume
`designs/` survives a restart on the hosted app.

## Conventions

- Ruff enforced (line length per pyproject); match existing comment density and docstring style.
- Keep `/api` responses backward-compatible with the JS clients in `webapp/static/js/` — they are
  hand-written fetch consumers with no schema layer.
- Tests use the `client` fixture in `tests/test_webapp.py` (isolated `DESIGNS_DIR`); engine tests
  are plain pytest over numpy.

---
> Source: [EdgyPage/Staffing_Optimizer](https://github.com/EdgyPage/Staffing_Optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
