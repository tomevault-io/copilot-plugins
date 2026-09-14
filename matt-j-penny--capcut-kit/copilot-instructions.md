## capcut-kit

> Read [`README.md`](README.md) first — it covers everything: the two lanes

# CLAUDE.md — CapCut Bridge Kit

Read [`README.md`](README.md) first — it covers everything: the two lanes
(file vs. live), the full command list, and the seven hard-won rules that
break silently if skipped (quit CapCut before a file-lane write, hardlink
footage under `~/Movies`, wipe `Timelines/` before re-editing an opened
draft, only ADD to a hand-edited draft, repair null `source_timerange`,
never click through the export share screen, call Kie AI's REST API
directly instead of its MCP tools).

For the two input JSON files (`cuts.json`, `graphics-plan.json`) that
`replay` and `graphics` expect to already exist, see
[`INPUT-CONTRACT.md`](INPUT-CONTRACT.md).

Everything else here is content, not code: `presets/` are the two caption
looks the templates implement, `assets/fonts/` are the fonts those looks
assume are installed. `capcut-templates/` (raw material stubs `add-text`/
`graphics` fill in) is **not included** — see README for how to source it.

Run all commands via `uv run capcut-bridge.py <command> ...` — see README for
the full list. macOS only.

---
> Source: [matt-j-penny/capcut-kit](https://github.com/matt-j-penny/capcut-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
