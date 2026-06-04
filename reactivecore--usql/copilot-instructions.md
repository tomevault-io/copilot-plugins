## usql

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
sbt test                    # Run all tests
sbt "test;lintCheck"        # Run tests + lint checks (what CI runs)
sbt "testOnly *QueryBuilderTest"  # Run a single test class
sbt lint                    # Format + fix: scalafmtAll; scalafixAll
sbt lintCheck               # Check formatting + linting without modifying
```

Tests are forked (`Test / fork := true`). Test databases: H2 (in-memory, always available) and PostgreSQL (requires `POSTGRES_HOST`, `POSTGRES_USER`, `POSTGRES_PASSWORD` env vars).

## Code Style

- Scala 3 with `-feature -new-syntax -rewrite` flags; all warnings are errors
- Scalafmt: max 120 columns, Scala 3 dialect, `align = most`
- Scalafix: no `var`s, no `finalize`, removes unused imports/locals
- No external runtime dependencies — only JDBC

## Architecture

**usql** is a lightweight JDBC toolkit for Scala 3 providing SQL interpolation, type-safe query building, and automatic CRUD generation.

### Core Layer (`src/main/scala/usql/`)

- **`Sql.scala`** — `sql"..."` string interpolation with type-safe parameter embedding
- **`DataType[T]`** — Type class mapping Scala types to JDBC (`ResultSet` extraction, `PreparedStatement` filling)
- **`RowDecoder[T]` / `RowEncoder[T]`** — Compose `DataType` instances for tuples and multi-column reads/writes
- **`Query.scala` / `Update.scala` / `Batch.scala`** — Execute SQL statements against a `Connection`

### DAO Layer (`src/main/scala/usql/dao/`)

- **`SqlFielded[T]`** — Macro-derived field structure for case classes
- **`SqlTabular[T]`** — Extends `SqlFielded` with table name; derive via `SqlTabular.derived[MyCaseClass]`
- **`Crd[T]` / `KeyedCrud[K, T]`** — Auto-generated Create/Read/Delete(/Update) operations
- **`QueryBuilder[T]`** — Functional query builder with `filter`, `map`, `join`, `leftJoin`
- **`ColumnPath[T, R]`** — Type-safe column references for query builder predicates
- **`Macros.scala`** — Scala 3 macro derivation powering `SqlFielded`, `SqlTabular`, column paths

### Database Profiles (`src/main/scala/usql/profiles/`)

- **`BasicProfile`** — Core `DataType` instances (primitives, strings, temporal types, arrays)
- **`H2Profile`** / **`PostgresProfile`** — Database-specific type support

### Test Infrastructure (`src/test/scala/usql/util/`)

- `TestBaseWithH2` — Creates fresh in-memory H2 database per test
- `PostgresSupport` — Connects to external PostgreSQL when env vars are set

---
> Source: [reactivecore/usql](https://github.com/reactivecore/usql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
