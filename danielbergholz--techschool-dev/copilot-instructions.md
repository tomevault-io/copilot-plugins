## techschool-dev

> - Run `mix precommit` before declaring any code change complete.

# Repository instructions

- Run `mix precommit` before declaring any code change complete.
- For LiveView or other UI changes, add a relevant LiveView test and verify the final behavior with Tidewave's browser tools.
- `mix credo --strict` has known existing cleanup debt and is intentionally outside the green precommit gate. Do not hide or suppress its findings; address them when they are in scope.

---
> Source: [danielbergholz/techschool.dev](https://github.com/danielbergholz/techschool.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
