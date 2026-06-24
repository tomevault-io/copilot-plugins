## self-improvement-frontend-design-skill-loop-for-codex

> This repository benchmarks and improves `frontend-design` skill quality for GPT Codex models (especially GPT-5.2 / GPT-5.3) using repeatable experiments.

# AGENTS.md

## Project Purpose

This repository benchmarks and improves `frontend-design` skill quality for GPT Codex models (especially GPT-5.2 / GPT-5.3) using repeatable experiments.

## Quick Start

1. Read `README.md` for project context.
2. Use `prompt.md` as the single shared benchmark prompt.
3. Work inside `experiments/<version>/` for all versioned changes.

## Experiment Invariants

1. Do not modify `prompt.md` unless explicitly asked.
2. For new iterations, create a new `experiments/version-X/` folder; never overwrite old versions.
3. Always run Codex from inside the active version folder while still using the shared root `prompt.md`.
4. For raw experiments, seed from global `$frontend-design` (`/home/waishnav/.agents/skills/frontend-design/SKILL.md`) and keep app scaffolding fresh.
5. Keep each version self-contained:
   - `.agents/skills/frontend-design/SKILL.md`
   - `t4-canvas/` output workspace (fresh/isolated by default; no inherited implementation)
   - `README.md` documenting hypotheses and changes
   - `CRITQUES.md` comparing outputs against expected reference designs
   - `screenshots/` with full-page captures for `/1` to `/5`
6. Compare against visual targets before scoring:
   - `research/targetted-designs/`
   - `research/theo-screenshots-2k-clean/opus45_with_skill/`
   - `research/theo-screenshots-2k-clean/opus_iterations/`

## Local Experiment Skill

Use `.agents/skills/frontend-design-improvements-loop/SKILL.md` for long-running iterative benchmark loops, mutation strategy, and headless `codex exec` command templates.

Local skill path:
- `.agents/skills/frontend-design-improvements-loop/SKILL.md`

---
> Source: [Waishnav/self-improvement-frontend-design-skill-loop-for-codex](https://github.com/Waishnav/self-improvement-frontend-design-skill-loop-for-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
