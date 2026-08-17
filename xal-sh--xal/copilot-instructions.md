## xal

> A terminal coding harness: an agentic TUI that integrates with AI providers to do real development work in your project. Built in small working increments — `0.0.x` releases grow feature by feature until `0.1.0`, which is **v0, the beta**.

# Terminal coding harness

A terminal coding harness: an agentic TUI that integrates with AI providers to do real development work in your project. Built in small working increments — `0.0.x` releases grow feature by feature until `0.1.0`, which is **v0, the beta**.

## Vision

Customizable and extensible full coding harness that remains as small as possible and allows everyone to extend and change its behaviours.

### Principles

- Customizable and Extensible by supporting plugins
- Plugins must not rely on eachother. This will make a circular dependencies and this is a hard violation.

## Conventions

- Early returns over nested conditions.
- Typed wire boundaries; narrow `unknown` with `lib/json` guards, never `as` casts.
- Typed unions at seams; handle every case so a new one cannot slip through silently.
- No code comments.
- Write simple and readable code and avoid extra complexity.
- If a constant will be used only one time, Do not overengineer to make it `const`. this reduces the code readability.
- Do not write what already exists; reuse it or extract it to one shared place.
- Do not patch symptoms. Fix the root cause.
- Fail loudly. Never swallow an error, and never write data after ignoring one.
- Whatever you write, you must read back in the same shape.
- Build on guarantees, not coincidences.
- Everything must have a consumer today; delete what nothing uses.
- Leave every file you touch consistent with the rules and its neighbors.
- Do not update AGENTS.md unless user specificly asks for.
- Do not overtest, only critical paths need testing
- Docs, especially the configuration docs must be always kept up to date.

## Linting & Formatting

Use `bun checks:fix` to fix any formatting or linting if you modified files that are subject to this.

---
> Source: [xal-sh/xal](https://github.com/xal-sh/xal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
