## duckdb-sql

> This rule applies to all SQL files in the project.


# SQL Rules
This rule applies to all SQL files in the project.
## File Pattern
*.sql
## Description
When working with SQL files, we use DuckDB as our database engine. SQL files should be executed using the command `duckdb $DUCKDB_DATABASE -f {file}`.
## Formatting
- Use 4 spaces for indentation
- Use SQLFluff for formatting with DuckDB dialect
- Format on save
## Commands
- Run SQL file: duckdb $DUCKDB_DATABASE -f {file}
## Best Practices
- Use consistent naming conventions
- Include comments for complex queries
- Use proper indentation for readability
- Follow DuckDB's SQL dialect specifications

---
> Source: [richban/opendata-stack-platform](https://github.com/richban/opendata-stack-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
