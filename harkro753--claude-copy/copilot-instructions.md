## claude-copy

> This repo is a design-cloning workshop: a Chrome extension captures a website's screenshot/DOM/computed styles, and a clone pipeline turns that into a clone verified pixel-by-pixel against the screenshot. Each cloned site lives in `apps/<site>/` — capture inputs, derived `shared/` design system, one folder per route, and `progress.md` as the handoff record.

## Codebase Overview

This repo is a design-cloning workshop: a Chrome extension captures a website's screenshot/DOM/computed styles, and a clone pipeline turns that into a clone verified pixel-by-pixel against the screenshot. Each cloned site lives in `apps/<site>/` — capture inputs, derived `shared/` design system, one folder per route, and `progress.md` as the handoff record.

**Stack**: vanilla Python (PIL/Playwright on the system `python3`) for the measuring oracle; React + Vite + Tailwind v4 + shadcn/ui for the React clone target; Chrome Manifest V3 (`capture-extension/`).
**Structure**: `apps/<site>/` (one folder per cloned site) · `capture-extension/` (capture tool) · `.claude/skills/` (`clone` — the pipeline, a plugin; plus `plugin-creator`, `dense-ui-metrics`, `shadcn`, `skill-creator`).

### The clone plugin

`.claude/skills/clone/` is a Claude Code **plugin** (`clone@skills-dir`, auto-loads from this repo). It ships two skills over **one engine**:

- `/clone:html` — static `page.html` routes, no toolchain
- `/clone:react` — shadcn/ui components with captured states as props

One CLI, on PATH as `clone` while the plugin is enabled:

```
clone prep <session> --target html|react     # target is recorded in routes.json
clone extract | resolve | cluster | check
clone init | map | build                     # react only
```

The engine is `engine/` — one module per concern (`prep`, `extract`, `glyphs`, `assets`, `cluster`, `score`, `check`, `progress`, `styles`, `util`), with the two artefacts as `engine/targets/{html,react}.py` behind a `Target` interface. Adding a third artefact means one file in `targets/`, not a fork. Shared pipeline docs live in `references/pipeline.md`; both skills point at it rather than restating it.

Tests (`clone/tests/`): `test_engine.py`, `test_react.py`, `test_hook.py` (fast), `test_e2e.py` (runs every phase as a subprocess against a real capture). Run all four after touching the engine.

A `PreToolUse` hook (`hooks/guard.py`) blocks scripted rewrites (`python3 - <<`, `sed -i`, `perl -pi`) of clone artefacts. Use `Edit` — a `.replace()` that does not match is a silent no-op, and one measured session made 50 such edits believing they landed.

Pipeline generations: v1 `copy-website` (deleted; left 6 `spec.json`-based apps behind) → v2 `clone-session` → v3 `clone-shadcn` → **v4 `clone` plugin** (v2 and v3 merged; both are in git history). Design rationale for v3 is in [docs/PIPELINE_GRAPH.md](docs/PIPELINE_GRAPH.md).

For detailed architecture, see [docs/CODEBASE_MAP.md](docs/CODEBASE_MAP.md).

---
> Source: [HarKro753/claude-copy](https://github.com/HarKro753/claude-copy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
