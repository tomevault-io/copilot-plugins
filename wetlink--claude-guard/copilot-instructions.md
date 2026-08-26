## claude-guard

> Use a small verification harness before changing behavior.

## Development Workflow

Use a small verification harness before changing behavior.

- Run `./scripts/check.sh` before claiming the project is healthy.
- Keep `bin/claude-guard` dependency-light: bash, curl, jq.
- Do not commit real tokens, OAuth credentials, local Claude settings, or live diagnostic logs.
- Keep network-heavy checks out of the default test suite.

---
> Source: [wetlink/claude-guard](https://github.com/wetlink/claude-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
