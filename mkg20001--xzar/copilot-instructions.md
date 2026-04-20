## xzar

> Use `nix develop --command` for all cargo commands (provides PostgreSQL libs and other dependencies).

# Development

Use `nix develop --command` for all cargo commands (provides PostgreSQL libs and other dependencies).

# Testing

Run integration tests with PostgreSQL:
```
nix run .#integration-tests
```

This sets up a temporary PostgreSQL instance and runs all interop tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mkg20001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
