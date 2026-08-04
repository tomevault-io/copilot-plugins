## cli

> - Migration functions must print a message only if they performed changes.

# Migration Guidelines

- Migration functions must print a message only if they performed changes.
- Use `internal.ChangeFileContent`, which returns a boolean, and print the message only when it is `true`.

---
> Source: [gofiber/cli](https://github.com/gofiber/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
