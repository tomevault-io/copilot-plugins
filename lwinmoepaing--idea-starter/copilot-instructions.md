## idea-starter

> Idea-to-MVP scaffolding for **one product per repo**. IDs (`#0001`, `#0002`, …) are feature drops on the same product — `#0001` scaffolds `mvp/`, later IDs extend it. Different product → clone the scaffolding into a new folder.

# CLAUDE.md

Idea-to-MVP scaffolding for **one product per repo**. IDs (`#0001`, `#0002`, …) are feature drops on the same product — `#0001` scaffolds `mvp/`, later IDs extend it. Different product → clone the scaffolding into a new folder.

## Layout
- `workflow/` — per-idea evidence, `#NNNN-<slug>.md`; same slug threads through subfolders
  - `ideas/` ← `validate-idea`
  - `approach/` ← `pain-hunt`
  - `marketing/` ← `marketing-plan`
- `tools/`
  - `drawing/` — Vite + React + tldraw v3. `cd tools/drawing && npm i && npm run dev`
  - `scripts/setup-mcp.sh` — bootstraps MCPs into `.mcp-servers/`
- `docs/` — specs + Lean Canvas PNGs + pitch summaries
- `plans/` — `{id}-plan-root.md` + phase files in `todo/`, moved to `done/` when shipped
- `mvp/` — prototype code (Vite + React + RR + Zustand)

## Sub-agents (`.claude/agents/`)
Workers that skills delegate to. Invoke via the Agent tool with `subagent_type: <name>`.
- `workflow-reader` — reads `workflow/{ideas,approach,marketing}/#NNNN-*.md`, returns digest + gaps
- `scaffold-inspector` — inspects `mvp/`, returns stack + conventions + gaps + scaffold-phase recommendation
- `dependency-mapper` — takes phase specs, returns DAG (blocked-by + parallel-safe + critical path)
- `phase-writer` — renders one phase spec to markdown. Spawn N in parallel for N phases.
- `phase-executor` — implements one phase: reads `plans/todo/{id}-{NN}-<slug>.md`, writes code to `mvp/`, returns done-when results. `mvp-execute` spawns N in parallel for parallel-safe groups.
- `pitch-summary` — synthesizes workflow + root plan + shipped phases into `docs/#{id}-<slug>-pitch.md`. Auto-triggered when an ID's last phase ships.
- `deck-builder` — distills one `docs/#{id}-<slug>-pitch.md` into a slide-deck PDF (`docs/#{id}-<slug>-deck.pdf` + editable `-deck.json`). Spawned by `pitch-deck`, one per drop.

---
> Source: [lwinmoepaing/idea-starter](https://github.com/lwinmoepaing/idea-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
