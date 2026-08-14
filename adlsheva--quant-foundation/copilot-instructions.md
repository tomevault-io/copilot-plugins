## quant-foundation

> Guidance for Claude Code and other agents working in this repository.

# CLAUDE.md

Guidance for Claude Code and other agents working in this repository.

## Project

Second Foundation-inspired quantitative alpha research platform. Two
tracks: US equities (cross-sectional WorldQuant-style alphas) and European
power markets (energy alphas). The active capstone extends both tracks with
GNN/GAT relational factors — see `docs/gnn_capstone_design.md`.

## Agent skills

### Issue tracker

Hybrid: GitHub Issues is the canonical tracker (`gh` CLI); local markdown
under `.scratch/` is used for drafts and offline work. See
`docs/agents/issue-tracker.md`.

### Triage labels

Five canonical roles, default strings (`needs-triage`, `needs-info`,
`ready-for-agent`, `ready-for-human`, `wontfix`). See
`docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` + `docs/adr/` at the repo root. See
`docs/agents/domain.md`.

---
> Source: [ADLSHEVA/quant-foundation](https://github.com/ADLSHEVA/quant-foundation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
