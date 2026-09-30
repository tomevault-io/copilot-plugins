## opensdl

> - All metadata access goes through `Repositories` or a future narrow protocol.

# Storage package instructions

- All metadata access goes through `Repositories` or a future narrow protocol.
- Model changes require Alembic migration and round-trip tests.
- Artifact bytes are immutable and hash-verified.
- Test against SQLite. No PostgreSQL runs anywhere in CI, so dialect-specific behavior — the
  rowcount of a conditional `UPDATE`, savepoints, conflict handling — is unverified on the other
  supported backend. Prefer portable constructs and say in the change where one is relied on.
  Adding that CI job is tracked in the development backlog.

---
> Source: [fl-sean03/OpenSDL](https://github.com/fl-sean03/OpenSDL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
