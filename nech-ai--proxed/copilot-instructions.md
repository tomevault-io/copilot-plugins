## trigger

> Trigger.dev v3 conventions (tasks live in @proxed/jobs)

# Trigger.dev (v3)

## How Trigger is wired in this repo

- Task implementations live in `packages/jobs` (importable as `@proxed/jobs`).
- Trigger.dev loads from `apps/app/jobs` because `apps/app/trigger.config.ts` has `dirs: ["jobs"]`.
- Pattern: implement tasks in `packages/jobs`, then re-export them from `apps/app/jobs/*`.

## Local dev

- From repo root: `bun run dev:trigger`

## Conventions

- Use `@trigger.dev/sdk/v3` (`task` / `schemaTask`) — never v2 APIs.
- Always `export` each task and keep `id` unique and stable.
- Prefer `schemaTask` + Zod schemas for payload validation.
- Avoid `triggerAndWait` in request/response paths; prefer fire-and-forget with observability.

---
> Source: [nech-ai/proxed](https://github.com/nech-ai/proxed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
