## creator-breakout-kit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Creator Breakout Kit is the product that finds the creator angles, hooks, and concepts most likely to break out for a brand — before the brand pays to source or produce. Strategy before sourcing, pattern fit over follower count. The final shareable artifact is the **Creator Breakout Brief** (`creator-breakout-brief.md` and `creator-breakout-brief.html`).

Internally, it's a Python + prompt package: no app, server, or build step — just a pipeline of deterministic scripts, thin connector adapters, and skill prompt modules stitched together via a shared run directory.

One brand = one `runs/<date>-<slug>/` directory. Every stage reads and writes files in that directory; do not introduce global state or cross-run coupling.

## Common commands

```bash
# Setup / check connector env
cp .env.example .env
bash scripts/preflight.sh

# Tests (stdlib unittest, no pytest, no package __init__.py files).
# All test commands must be run from the repo root.
python3 -m unittest discover -s tests -v                                # all tests
python3 tests/test_enrich_breakout_run.py -v                            # one file
python3 tests/test_enrich_breakout_run.py TestEnrichBreakoutRun.test_one_sentence_preserves_abbreviations -v  # one method
```

There is no linter, formatter, or package manager — all scripts use only Python 3 stdlib plus optional HTTP calls inside adapters. Don't add a `requirements.txt`, `pyproject.toml`, or framework dependency without discussing first.

## Pipeline architecture

The end-to-end flow moves through three layers that share a run directory:

1. **Base run** — `scripts/create_base_breakout_run.py` writes `input.json`, `breakout-kit.json`, `creator-breakout-brief.md` (the Creator Breakout Brief), `internal_notes.md`, (optional) `sales_handoff.md`. This alone is the `quick-pass` mode and requires no connectors.
2. **Connector research** — `scripts/run_creator_breakout_research.sh` drives `skills/brand-snapshot/scripts/firecrawl_brand_discovery.py` and `skills/breakout-pattern-research/scripts/virlo_outlier_research.py` into a separate `$RESEARCH_DIR`, then `scripts/enrich_breakout_run.py` folds those outputs back into the run directory. Optional stages: `skills/breakout-pattern-research/scripts/virlo_tracking_intelligence.py` (audience-signal pass) and `skills/breakout-pattern-research/scripts/video_intelligence.py` (ScrapeCreators + OpenRouter Gemini video asset review). **Adapter scripts live inside the skill that uses them**, per the pattern in the other kits (landing-page-factory, meta-ads-kit) — each skill is self-contained with its own `scripts/` and `references/` subdirectories.
3. **Synthesis** — `scripts/synthesize_full_kit.py` is the only stage allowed to produce the shareable final brief. It materializes `brand-snapshot.json`, `breakout-pattern-research.json`, `creator-archetype-map.json`, `concept-slate.json`, `skill-run-manifest.json`, refreshes the brief/notes/handoff (markdown), and **also emits `creator-breakout-brief.html`** — a dark-terminal-aesthetic presentation-layer version of the same brief rendered by `scripts/render_breakout_brief_html.py`. Both the markdown and HTML renderers read from the same `breakout-kit.json` and share `concept_scores` / `one_sentence` via `importlib`, so score labels and first-test copy never drift between formats. Synthesis imports both `enrich_breakout_run` and `render_breakout_brief_html` dynamically — keep both modules importable as scripts (no top-level side effects at import time).

The run mode (`quick-pass`, `brand-pass`, `pending-data`, `outlier-pass`) is chosen inside `enrich_breakout_run.py` from what's present in the research directory. `synthesize_full_kit.py` separately stamps `strategy_stage: full-kit-synthesis` on the kit once synthesis has run. Optional stages like audience-signal tracking and video intelligence layer on top of `outlier-pass` — they don't define their own mode. Missing connectors must degrade the mode label honestly — see "Evidence rules" below.

## Adapters vs skills

- **Skill-owned adapter scripts** (`skills/<skill>/scripts/*.py`) are thin HTTP clients for external services (Firecrawl, Virlo, Virlo Tracking, ScrapeCreators, OpenRouter). Keep them narrow: fetch, normalize to the fields listed in `docs/integration-notes.md`, write JSON + a sibling markdown summary to disk. No strategic reasoning here. Adapters colocate with the skill that consumes their output.
- `skills/<name>/SKILL.md` are prompt modules consumed by a calling agent. They're referenced by name from `skills/creator-breakout-kit/SKILL.md` (the orchestrator). When editing a skill, preserve its YAML frontmatter and the normalized input/output contract — other skills and `synthesize_full_kit.py` assume those field names.
- `shared/schemas/breakout-kit.schema.json` is the canonical shape for `breakout-kit.json`. `shared/templates/` holds the markdown templates that the formatter skills follow.

## Evidence rules (non-negotiable)

These are product constraints, not style preferences. Violating them breaks the core positioning.

- Never imply creators are booked, sourced, or committed. Creator names are comps only.
- Virlo outlier data is **metadata-only** unless `video-intelligence.json` exists for that post with `video_asset_sent_to_model: true`. Do not infer visual beats from captions.
- Virlo Tracking output is **comment-derived directional signal**, not statistically representative market research. Label it that way in every output.
- An `outlier-pass` run is not client-ready until `synthesize_full_kit.py` has run. Keep that gate intact.
- When connectors are missing or pending, downgrade the mode label (`quick-pass`, `brand-pass`, `pending-data`) rather than silently filling gaps.

## Run artifact conventions

- `runs/` is gitignored. Never commit run output, and never hardcode a specific local run path into scripts or tests.
- Tests use `tempfile`-based fake runs and load scripts via `importlib.util.spec_from_file_location` because `scripts/` is not a package. Follow that pattern for new tests instead of adding `__init__.py` files.
- Scripts read/write JSON with `json.dumps(..., indent=2)`; markdown outputs use the subtitle helpers in `create_base_breakout_run.py` / `enrich_breakout_run.py`. If you add a new mode, update `subtitle_for_mode` in both places (they're intentionally duplicated to keep scripts runnable standalone).

## Public-release bar

This package is intended to be cloneable by an operator with no prior context. Before changes land, verify:
- `.env.example` still lists every connector key referenced by adapters, with no private defaults.
- `scripts/preflight.sh` reports on any new connector you add.
- A fresh clone can run the README path in `quick-pass` with no connectors configured.

---
> Source: [TheMattBerman/creator-breakout-kit](https://github.com/TheMattBerman/creator-breakout-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
