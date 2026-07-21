## src-cli

> - When adding new commands, use `urfave/cli` instead of the legacy `commander` pattern.

# Command Guidelines

- When adding new commands, use `urfave/cli` instead of the legacy `commander` pattern.
- Register new `urfave/cli` commands in the `migratedCommands` map in `cmd/src/run_migration_compat.go`.

---
> Source: [sourcegraph/src-cli](https://github.com/sourcegraph/src-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
