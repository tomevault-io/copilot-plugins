## trend-viewer

> > Daily trend viewer port. Upstream: single-file Python stdlib server (`_upstream/`). Target: feature-based modular Python (stdlib-only, no external deps).

# AGENTS.md — trend-viewer

> Daily trend viewer port. Upstream: single-file Python stdlib server (`_upstream/`). Target: feature-based modular Python (stdlib-only, no external deps).

## What this project does

Local web app aggregating YouTube/Shorts (InnerTube), Instagram Reels, X (syndication API), Threads (GraphQL, fallback), TikTok (tikwm), and AI video model/news feeds. Category x period filters, sort menus, 1h cache, image proxy. Serves at `http://localhost:8778`.

## Tech stack & constraints

- **Python 3 stdlib only** — no pip dependencies (upstream promise: double-click run for non-devs). Do not add dependencies without approval.
- Frontend: single `index.html` initially; split into `frontend/` assets during porting phases.
- Feature-based layout under `src/` (snake_case package folders — Python import constraint, `__init__.py` barrels, `_tool.py` suffix, `test_*.py` colocated).

## Conventions

- Follow the Lidge Standard: Screaming Architecture, Colocation, Barrel Export.
- `_upstream/` is read-only reference. Never edit it; port from it.
- Plans live in `devlog/_plan/` (decade-range numbering: 000 research, 010 phase 1, ...). Completed work moves to `devlog/_fin/YYMMDD_title/`.
- Module docs in `devlog/str_func/` — update when a feature module changes.
- External endpoint URLs, public app IDs, and request headers copied from upstream must be preserved byte-exact (see 000_upstream-analysis.md risk section).

## Phase 3 standalone rule

This folder lives under `700_projects/`. When opened as its own workspace root, treat it as an independent project (direct commits, no submodule 2-step).

---
> Source: [xazingatrend/trend-viewer](https://github.com/xazingatrend/trend-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
