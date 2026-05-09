## multi-db-query

> Allow users to connect and query multiple databases simultaneously, compare datasets, and merge query results for unified analysis.

# 🗃️ Task: Multi-Database Query Support for myquery

## 🎯 Goal
Allow users to connect and query multiple databases simultaneously, compare datasets, and merge query results for unified analysis.

## ⚙️ Requirements
- Implement `multi_db_query_tool` in `tools/multi_db_query_tool.py`.
- Support multiple DB connections (PostgreSQL, MySQL, SQLite).
- Store connections in session context.
- When user requests cross-database queries:
  - Run equivalent queries in each DB.
  - Normalize schema (matching column names, data types).
  - Merge results by key columns (like “id” or matching names).
- Add commands:
  - `myquery connect --multi`
  - `myquery query --all`
- Allow side-by-side comparisons and aggregated analysis.

## 🧠 Steps
1. Add a connection pool manager in `core/multi_db_manager.py`.
2. Track active DBs in session context.
3. Implement `multi_db_query_tool` that:
   - Accepts prompt or SQL.
   - Distributes execution.
   - Collects, normalizes, and merges results.
4. Update CLI and Web UI to support multi-db mode.
5. Integrate with visualization (compare results visually).

## 🧩 Cursor Hints
- Use `@cursor: define-tool multi_db_query_tool`
- Use SQLAlchemy or asyncpg for multi-connection pooling.
- Handle schema conflicts gracefully.
- Add summary analysis (e.g., “db_a has 10% higher sales than db_b”).

---
> Source: [zakirkun/myquery](https://github.com/zakirkun/myquery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
