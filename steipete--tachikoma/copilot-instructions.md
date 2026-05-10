## tachikoma

> READ ~/Projects/agent-scripts/{AGENTS.MD,TOOLS.MD} BEFORE ANYTHING (skip if files missing).

# AGENTS.MD

READ ~/Projects/agent-scripts/{AGENTS.MD,TOOLS.MD} BEFORE ANYTHING (skip if files missing).

Tachikoma notes:
- Keep this repo in sync with Peekaboo; bump the submodule there after changes.
- Batch git network ops with Peekaboo: commit related changes first, then push/pull repos together so the submodule pointer never races the source repo.
- Default workflow: `pnpm install`, `pnpm run lint`, `pnpm run test` before publishing.
- Adapters live under `src/providers`; keep new providers consistent with existing patterns.

---
> Source: [steipete/Tachikoma](https://github.com/steipete/Tachikoma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
