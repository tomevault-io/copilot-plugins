## apy

> `apy` is a Python CLI for adding and editing [Anki](https://apps.ankiweb.net/)

# AGENTS.md

`apy` is a Python CLI for adding and editing [Anki](https://apps.ankiweb.net/)
notes from the command line, without Anki running. Source lives in
`src/apyanki/`; tests in `tests/`.

## Workflow

- Run `mise check` for formatting, linting, and type checking.
- Run `mise test` to run the test suite.
- Don't run checks or tests after every change. Make the full set of edits
  first, then run `mise check` and `mise test` once at the end.
- Leave commits and git operations to the user.

---
> Source: [lervag/apy](https://github.com/lervag/apy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
