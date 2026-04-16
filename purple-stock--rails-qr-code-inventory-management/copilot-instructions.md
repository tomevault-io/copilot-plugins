## rails-qr-code-inventory-management

> - Use a single short imperative line.

# Repository instructions

## Commit style
- Use a single short imperative line.

## Checks before committing
Run the following commands:
1. `bin/rubocop`
2. `bin/brakeman --no-pager`
3. `bin/importmap audit`
4. `bin/rails db:test:prepare`
5. `bin/rails test`

If commands fail because dependencies are missing, mention the limitation in the PR.

## PR body
Summarize notable changes with citations. Include a Testing section summarizing command results.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Purple-Stock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
