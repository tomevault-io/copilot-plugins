## stas-auth-gateway

> This repo uses a thin local Codex adapter. Shared policy lives in x4.

# AGENTS.md

This repo uses a thin local Codex adapter. Shared policy lives in x4.

Source of truth:

- `/home/codex/codex-work/Projects/x4/AGENTS.md`
- `/home/codex/codex-work/Projects/x4/CURRENT_STATE.md`
- `/home/codex/codex-work/Projects/x4/docs/harness-control-plane.md`

## Local Project Facts

- Name: `stas-auth-gateway`
- Path: `/home/codex/codex-work/Projects/stas-auth-gateway`
- Baseline: `backend-api`
- Harness level: `L4`
- Service mode: `managed`

## Local Scope

- Project-specific code, content, config, and validation live here.
- Shared orchestration policy, role contracts, and rollout rules stay in x4.
- Do not copy long x4 policy blocks into this repo; link back to x4 instead.

## Working Rules

- Use orchestrator-first for non-microscopic work when runtime support exists.
- Max active subagents: 2.
- Max nested delegation depth: 1, unless the parent brief explicitly allows more.
- A spawned executor must not launch nested agents unless explicitly allowed by the parent brief.
- External writes, deploys, permission changes, credential changes, and destructive actions require explicit user confirmation.
- Do not edit `CLAUDE.md` or `.claude/**` unless the user explicitly asks.
- Do not revert unrelated user changes.

## Local Validation

Use `docs/codex-validate.md` in this repo for project-specific checks.

---
> Source: [hivrich/stas-auth-gateway](https://github.com/hivrich/stas-auth-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
