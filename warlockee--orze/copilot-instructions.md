## orze

> Read `SKILL.md` in this directory for the full operations & extension reference (CLI, API, architecture, diagnostics).

# Orze — Development Guide

Read `SKILL.md` in this directory for the full operations & extension reference (CLI, API, architecture, diagnostics).

## Quick Reference

- **Run**: `orze -c orze.yaml` (auto-detect GPUs)
- **Stop**: `orze --stop`
- **Check**: `orze --check` (validate config)
- **Admin UI**: `orze --admin` → http://localhost:8787
- **MCP**: Claude Code connects via `.mcp.json` → `POST /mcp`

## Architecture (post-LOD refactor)

Every module in `src/orze/engine/` is <800 LOC with a calling spec at the top. Read the calling spec before reading the implementation.

```
engine/orchestrator.py      (679) — Orze class, run() recipe
engine/phases.py            (600) — main loop phases (mixin)
engine/lifecycle.py         (390) — startup, shutdown, PID
engine/retrospection.py     (362) — signal detection + dispatch
engine/experiment_analysis.py(375) — cross-experiment regression analysis
engine/smart_suggestions.py (200) — rule-based idea generation
engine/auto_ideas.py        (180) — parameter variation generator
engine/reporter.py          (338) — notifications + plateau
engine/launcher.py          (324) — training subprocess lifecycle
engine/evaluator.py         (298) — eval subprocess + sealed verification
engine/failure.py           (240) — failure tracking
engine/failure_analysis.py  (210) — structured failure classification
engine/family_guard.py      (180) — approach family taxonomy
engine/sealed.py            (143) — sealed file integrity
engine/cluster.py           (140) — multi-machine coordination
engine/config_dedup.py       (85) — config hash dedup
engine/gpu_slots.py         (230) — VRAM-aware GPU scheduling
skills/loader.py            (140) — composable prompt fragments
admin/mcp.py                (280) — MCP server for Claude Code
extensions.py               (140) — orze-pro plugin discovery

# Agent modules (in orze-pro, not in open orze):
# engine/role_runner.py, agents/research.py, agents/code_evolution.py, etc.
# Install orze-pro for autonomous research agents.
```

## Conventions

- LOD: max 800 LOC per file, calling spec at top, pure functions for tools
- Config: `orze.yaml` is the single source of truth
- Ideas: `ideas.md` is append-only, consumed into `idea_lake.db`
- Results: `results/{idea_id}/metrics.json` is the training contract

---
> Source: [warlockee/orze](https://github.com/warlockee/orze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
