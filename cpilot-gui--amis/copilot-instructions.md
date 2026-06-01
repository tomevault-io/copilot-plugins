## amis

> This file is committed repo guidance. Keep it short; load `HARNESS.md` before changing code, tests, or repo workflow.

# Amis Agent Harness

This file is committed repo guidance. Keep it short; load `HARNESS.md` before changing code, tests, or repo workflow.

Core rules:
- Source of truth is `src/main`, `src/preload`, and `src/renderer/src`; fix source, not generated output.
- Do not hand-edit generated/runtime artifacts: `out`, `dist`, `build`, caches, logs, or Playwright output.
- Keep formal `/conversation` separate from Agent/OpenClaw `/agents?agent=openclaw&view=chat`.
- Preserve Lucyna/OpenClaw chat invariants: immediate user bubble, ordered assistant replies, real persona/media evidence.
- Validate touched surfaces with targeted tests plus `npm run typecheck`, `npm run lint`, `npm run check:structure`, and `git diff --check` when applicable.

See `HARNESS.md` for compact implementation/test details.

Docs extension:
- For documentation-system work, start at `docs/README.md` and `docs/documentation-standards.md`.
- Keep `docs/architecture.md` and `docs/runtime-flows.md` as current-state maps, not refactor authorization.
- Use ADR for architecture decisions, SRS for requirements and verification methods, and SDD/views for implementation design viewpoints.
- Keep published English/Chinese docs synchronized under `docs/` and `docs/zh-CN/`.
- Mermaid diagrams must follow the GitHub/Obsidian-safe rules in `docs/documentation-standards.md` and include adjacent Evidence/证据 blocks.

---
> Source: [cPilot-GUI/Amis](https://github.com/cPilot-GUI/Amis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
