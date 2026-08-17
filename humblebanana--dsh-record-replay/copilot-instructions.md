## dsh-record-replay

> Guidance for coding agents working in this repository.

# AGENTS

Guidance for coding agents working in this repository.

## Build and test

```bash
pnpm install
node scripts/link-dsh.mjs   # link @deepseek-ai/* from a DSH harness/checkout
pnpm validate               # build + test
```

## Hard rules

- The plugin module (`src/index.ts`) must export a Cordis namespace plugin
  (`name`, `inject`, `Config`, `apply`) and must NOT add `export default`.
- `ctx.tools.register(...)` and `ctx.skills.register(...)` return disposers that
  are fiber-attached; do not leak them.
- The CLI is invoked only through `ctx.subprocess.spawn` with bounded collected
  output and an abort signal. Never use raw shell string interpolation.
- Recording artifacts (`runsOut`, `skillInputsOut`) must be workspace-relative
  and are validated in `src/config.ts`.

## Layout

- `src/index.ts` — plugin entry (registration).
- `src/config.ts` — schema and validation.
- `src/tools.ts` — model-facing tool definitions.
- `src/runner.ts` — CLI resolution and subprocess invocation.
- `src/skill.ts` — the `open-record-replay` skill body.
- `tests/` — registration-contract and runner unit tests.

---
> Source: [humblebanana/dsh-record-replay](https://github.com/humblebanana/dsh-record-replay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
