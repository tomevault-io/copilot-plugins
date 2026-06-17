## winegpt

> Read and follow `docs/engineering-standards.md` before editing code.

# Gemini Project Instructions

Read and follow `docs/engineering-standards.md` before editing code.

## Main rules

- Keep changes small, local, and easy to review.
- Understand the relevant existing code before editing.
- Do not broaden scope without a strong reason.
- Preserve existing behavior unless the task explicitly changes behavior.

## Required workflow

For non-trivial changes:

1. Inspect the relevant modules first.
2. Add or update tests for behavior changes.
3. Implement the smallest correct change.
4. Refactor only if it clearly simplifies the result.
5. Run relevant tests.
6. Summarize:
  - what changed
  - tests added or updated
  - commands run
  - assumptions and follow-up items

## Project-specific expectations

- For bugs, add a failing regression test if feasible.
- Avoid duplication, but do not abstract prematurely.
- Keep backend transport, business logic, and persistence concerns separate.
- Keep frontend components simple and isolate side effects.
- Keep camera and other browser API integration inside dedicated wrappers/modules.

## Docker-related changes

- When changing runtime or deployment behavior, update the relevant Dockerfile, Compose files, and docs.
- Keep development and production Compose configuration separate.
- Do not commit secrets or real credentials.
- Persist database data in named volumes.
- Expose only required services publicly.

## Response style

Be concise, concrete, and implementation-focused.

---
> Source: [matssigge/winegpt](https://github.com/matssigge/winegpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
