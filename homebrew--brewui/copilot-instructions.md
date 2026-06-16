## workflow-ci

> Testing and verification workflow expectations.


# Workflow and CI

- Prefer small, focused stories and commits.
- Run relevant checks for modified scope before concluding work.
- Treat repository git hooks and CI workflows in `.github/workflows/` as the mandatory enforcement layer.
- Keep local progress tracking out of PR diffs; use local `.ai/progress.md` only when needed.

## Swift quality (parity with `swift_quality.yml`)

Canonical instructions: **`AGENTS.md` → Workflow → Swift quality (local parity with CI)**. Same commands as `.github/workflows/swift_quality.yml`; this rule is a Cursor-scoped reminder when `**/*.swift` is in context.

---
> Source: [Homebrew/BrewUI](https://github.com/Homebrew/BrewUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
