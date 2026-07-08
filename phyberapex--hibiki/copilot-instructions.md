## verify-after-changes

> Run lint and tests after making code changes


# Verify after code changes

After editing application code (bot, web, or shared config), **run lint and tests** to ensure nothing is broken:

1. **Lint:** From repo root run `pnpm run lint`. Fix any reported issues.
2. **Tests:** From repo root run `pnpm run test`. Fix any failing tests.

Do this before considering the change complete. If you introduce a regression, fix it or revert the change.

---
> Source: [PhyberApex/hibiki](https://github.com/PhyberApex/hibiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
