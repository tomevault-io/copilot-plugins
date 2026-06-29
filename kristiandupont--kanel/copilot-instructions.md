## kanel

> End-to-end tests that spin up a real PostgreSQL Docker container and run kanel against actual schemas.

# integration-tests

End-to-end tests that spin up a real PostgreSQL Docker container and run kanel against actual schemas.

Each test file covers a specific feature or config scenario: `basics.test.ts`, `configurations.test.ts`, `markdown-generator.test.ts`, `tagged-comments.test.ts`, `v3-compatibility.test.ts`, `v4-config.test.ts`. Rendered output is compared against snapshots in `__snapshots__/`.

These tests are slower than unit tests and require Docker. Use `test-helpers/` utilities for container/schema lifecycle.

---
> Source: [kristiandupont/kanel](https://github.com/kristiandupont/kanel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
