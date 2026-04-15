## precursor-sqlite

> SQL query explorer with in-memory database for Precursor.

# SQLite Client — Build Notes
## What This Is
SQL query explorer with in-memory database for Precursor.
## Key Decisions
- In-memory tables (Vec<Vec<String>>) — no actual SQLite dependency
- Tokenizer handles quoted strings, commas, parentheses
- Case-insensitive keyword/column matching
- WHERE with 6 comparison operators, numeric-aware
- 3 sample tables (users, products, orders) with 5 rows each
- 4 states: QueryInput, Results, Samples, History

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tbcolby)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tbcolby)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
