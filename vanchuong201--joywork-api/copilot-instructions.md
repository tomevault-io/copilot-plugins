## code-hygiene

> JoyWork API cleanup, logging, and non-production code hygiene


# JoyWork API Code Hygiene

- Do not leave `console.*`, localhost `fetch(...)`, webhook debug calls, or agent instrumentation in committed application code.
- If backend logging is needed, prefer Fastify or request-scoped logger patterns over ad hoc console logging.
- Remove temporary diagnostics, commented-out experiments, and dead branches before finishing the task.
- Do not add test-only or demo-only endpoints under `src/modules/**` unless the user explicitly asks for them.
- When touching a file that already contains temporary debug code, prefer cleaning it up as part of the same change if it is clearly safe to remove.

---
> Source: [vanchuong201/joywork-api](https://github.com/vanchuong201/joywork-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
