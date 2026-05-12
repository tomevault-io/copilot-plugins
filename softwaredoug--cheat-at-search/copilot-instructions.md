## cheat-at-search

> Note: This project uses Poetry for dependency management and test execution.

# AGENTS.md

Note: This project uses Poetry for dependency management and test execution.

## How to commit

- Stage only relevant files.
- Prepare a commit message with:
  - A short headline.
  - A longer paragraph describing the change.
  - A co-author line for Codex: `Co-authored-by: Codex <codex@openai.com>`.
- Show the full commit message before committing.

## Testing preferences

- When patching in tests, prefer `unittest.mock.patch` decorators.
- Prefer patch decorators over patch context managers.
- Prefer patch decorators over pytest `monkeypatch` unless there is a specific need.

---
> Source: [softwaredoug/cheat-at-search](https://github.com/softwaredoug/cheat-at-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
