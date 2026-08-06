## circle-take

> Single source of truth for AI agents working on this repo. `CLAUDE.md` points here.

# AGENTS.md — working on Circle Take

Single source of truth for AI agents working on this repo. `CLAUDE.md` points here.
Read `SPEC.md` (product + data model + gate doctrine) and `design.md` (UI system) first.

## What this app is

Electron + TypeScript + React + zustand desktop app: a local dailies room + QC gate for
AI-generated film takes. Python worker (`python/circletake_gate`) measures takes (identity /
temporal / anatomy / continuity / conformance) and writes gate reports; the app binds takes to
shots, lets the director compare + circle winners, and exports FCPXML/EDL/reshoot lists.
Everything also drivable via localhost control server + zero-dep MCP bridge (`mcp/`).

## Commands

```bash
npm install
npm run dev            # app with hot reload
npm run build && npm start
npm run typecheck      # strict TS, two projects
npm run lint
npm test               # Vitest unit
npm run e2e            # Playwright electron e2e
# Python engine (dev): repo-local .venv auto-used when present
python3 -m venv .venv && .venv/bin/pip install -r python/requirements.txt
.venv/bin/python -m pytest python/tests -q
```

**Definition of done: typecheck + lint + test green; engine changes also need pytest green.**

## Hard rules

1. `src/shared/` is PURE (no DOM/Electron/Node) and unit-tested. All cross-process shapes
   live in `src/shared/types.ts`; the IPC surface is `src/shared/api.ts`. Change contracts
   there first, then implementations.
2. MAIN owns the canonical Project. Renderer never mutates its mirror directly — every
   mutation is an IPC call; main persists and pushes `project-updated`.
3. Renderer never touches fs/spawn/network. Media/thumbnails/evidence go through the
   `ctake://` protocol only.
4. Worker protocol = line-JSON on stdout (`progress`/`done`/`error`), parsed by
   `parseWorkerLine`. Anything else on stdout is noise. stderr is logged, never parsed.
5. Model weights are runtime-downloaded, sha256-pinned, cached under the app cache dir —
   never committed, never fetched from non-pinned URLs.
6. Venv + caches must live OUTSIDE iCloud-synced paths. Dev venv is repo-local `.venv`
   (repo is in `~/Projects`, non-synced). Packaged: userData venv (Application Support).
7. Thresholds/constants for gate checks live in `python/circletake_gate/thresholds.py`
   with names + docstrings — never inline magic numbers in check code.
8. Every gate verdict must carry a human reason string. No naked numbers.
9. Design system tokens in `design.md` are law: no default-styled controls, no emoji icons,
   fonts are local woff2 only.
10. Media files are referenced in place (absolute paths); the project folder stores only
    JSON, thumbs, evidence JPEGs, kit images.

## Repo map

```
src/shared/     types.ts (model + protocol + scoring, PURE), api.ts (IPC surface),
                fcpxml.ts / edl.ts / reshoot.ts / csv.ts (pure export builders)
src/main/       index.ts (window, ctake:// protocol, IPC registration), project.ts
                (project store + persistence + mutators), ingest.ts (probe→thumbs→gate queue),
                ffmpeg.ts (spawn helpers), pythonEnv.ts (venv mgmt), gate.ts (worker spawn +
                report intake), control.ts (localhost control server)
src/preload/    typed bridge → window.circletake (implements src/shared/api.ts)
src/renderer/   React UI. store.ts (zustand mirror + UI state), App.tsx (rail + rooms),
                views/ (Bin, Village, Kits, Report, Print, Welcome), components/, styles.css
python/         circletake_gate/ package: cli.py (subcommands), protocol.py (line-JSON),
                probe.py, thumbs.py, models.py (pinned downloads), checks/ (identity.py,
                temporal.py, anatomy.py, continuity.py, conformance.py), report.py,
                thresholds.py; tests/ (pytest, synthetic fixtures via fixtures.py)
mcp/            circle-take-mcp.mjs — zero-dep stdio MCP bridge → control server
scripts/        screenshots.mjs (README shots), make_fixtures.py helpers
capture/        record-ux.mjs CDP rig for launch-video takes
tests/unit      Vitest (shared + renderer pure logic)
tests/e2e       Playwright electron
```

## Lane discipline (parallel agents)

- Lane MAIN: src/main, src/preload, src/shared/{fcpxml,edl,reshoot,csv}.ts, mcp/,
  tests/unit/main-*.test.ts, tests/e2e
- Lane UI: src/renderer only, tests/unit/ui-*.test.ts
- Lane ENGINE: python/ only
- Nobody edits src/shared/types.ts or api.ts without flagging it (contract change).
```

---
> Source: [wassermanproductions/circle-take](https://github.com/wassermanproductions/circle-take) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
