## cli-test-fixture-cleanup

> Avoid committing generated CLI test fixtures


# CLI Test Fixture Cleanup

## Key points

- Treat `test-*/` directories under fixture apps as ephemeral test output.
- Keep gitignore patterns in sync for fixture apps to avoid committing generated artifacts.
- Cleanup logic in tests should remove temporary directories after each run.

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
