## sql-client

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
mvn clean package          # Build fat JAR → target/sql-client-1.0.0.jar (~7.4MB)
mvn compile                # Compile only
mvn test                   # Run all tests
mvn test -Dtest=ConfigLoaderTest  # Run single test class
```

Run the client:
```bash
java -jar target/sql-client-1.0.0.jar -c config.yaml
java -jar target/sql-client-1.0.0.jar -url jdbc:oracle:thin:@host:1521:orcl -user scott -pass tiger
```

## Project Overview

Interactive terminal SQL client for Oracle and DM (达梦) databases. Java 8 Maven project producing a self-contained fat JAR via `maven-shade-plugin`.

## Architecture

Entry point: `com.sqlclient.SqlClient` → loads config → starts `SqlRepl` interactive loop.

**config/** — Configuration loading and merging:
- `ConfigLoader` parses YAML (SnakeYAML) and merges with CLI args (`-url`, `-user`, `-pass`, `-type`). CLI args override/add to file config.
- `AppConfig` holds a list of `ConnectionConfig` entries + default connection name.
- `DatabaseType` enum maps type strings to JDBC driver class names and auto-detects from URL prefix.

**db/** — Database operations:
- `ConnectionFactory` loads the JDBC driver and creates connections via `DriverManager`.
- `QueryExecutor` uses `Statement` (not `PreparedStatement`) for both queries and updates. Query results capped at 500 rows (`MAX_ROW_DISPLAY`).
- `QueryResult` is an immutable data holder for columns, rows (`List<Map<String, Object>>`), and elapsed time.

**repl/** — Interactive loop (`SqlRepl`):
- Uses JLine 3 for terminal handling, history persistence (`~/.sqlclient-history`), and multi-line input (semicolon-terminated).
- Backslash commands: `\help`, `\connections`, `\switch`, `\current`, `\reconnect`, `\quit`.
- SQL classification by prefix: SELECT/WITH/SHOW/DESC/EXPLAIN → query; INSERT/UPDATE/DELETE/CREATE/ALTER/DROP/TRUNCATE → update.

**display/** — `TableFormatter` renders `QueryResult` as ASCII table with auto-width, NULL display, 50-char truncation, and newline merging.

## Testing

JUnit 5 + Mockito + H2 in-memory database. `QueryExecutorTest` uses H2 to test actual SQL execution. `ConfigLoaderTest` uses `@TempDir` for YAML file tests. `TableFormatterTest` tests formatting logic directly.

## Key Dependencies

- JLine 3.25.1 — terminal interaction and history
- SnakeYAML 2.2 — config file parsing
- ojdbc8 21.5.0.0 — Oracle JDBC driver
- Dm8JdbcDriver18 8.1.1.49 — DM (达梦) JDBC driver
- H2 1.4.200 — test-only in-memory database

---
> Source: [mofishless/sql-client](https://github.com/mofishless/sql-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
