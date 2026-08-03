## clickhouse-sql-parser

> This document outlines guidance for AI coding agent including project structure, coding style, testing, and contribution practices for the ClickHouse SQL Parser project.

# AI Agents Guidline

This document outlines guidance for AI coding agent including project structure, coding style, testing, and contribution practices for the ClickHouse SQL Parser project.


## Development Commands

```shell
# Build the CLI binary
make

# Run tests
make test

# Update golden fixtures after intentional output changes
make update_test
```

After editing code, use `goimports` and `gofmt` to maintain code style, and run `make lint` to check for any issues before committing or requesting a review.

## Project Structure & Module Organization
- `main.go` is the CLI entry point (`clickhouse-sql-parser`) for AST output and SQL formatting.
- `parser/` contains core parser code: lexer (`lexer.go`), AST definitions (`ast.go`), traversal helpers (`walk.go`), and grammar-specific parser files (`parser_query.go`, `parser_table.go`, `parser_alter.go`, etc.).
- Tests live next to source as `*_test.go` files, with fixtures under `parser/testdata/`.
- Fixture groups are organized by SQL type (`basic/`, `query/`, `dml/`, `ddl/`), with generated expectations in `output/` (AST JSON) and `format/` (formatted SQL).

## Coding Style & Naming Conventions
- Use Go 1.21 conventions (`go.mod`) and keep code `gofmt`/`goimports` clean (enforced by lint).
- Naming is the most important style aspect, try you best to choose a clear and descriptive name for variables, functions, types, and files. For example, use `parseSelect` for a function that parses a SELECT statement, and `SelectStatement` for the corresponding AST node type.
- Place parsing logic in the matching module by statement family (for example, query parsing in `parser/parser_query.go`).
- Follow existing parser naming patterns such as `parseXxx` helpers and explicit AST type names.
- Keep AST `FormatSQL()` output deterministic; formatting changes must be reflected in golden files.
- You must go through the repository before adding new code to ensure consistency with existing patterns and styles. If you are unsure about where to place new code or how to format it, please refer to the existing codebase or ask for guidance.
- Reusing existing code and patterns is encouraged to maintain consistency and reduce redundancy. If you find a similar function or pattern in the codebase, consider adapting it for your needs instead of creating something new from scratch.

## Testing Guidelines
- Use Go’s `testing` package with `testify/require` assertions and `goldie` snapshot comparisons.
- Add new SQL cases as `.sql` files under the appropriate `parser/testdata/<category>/` directory.
- If expected outputs change, run `make update_test` and commit updated files in both `output/` and/or `format/`.
- Prefer descriptive test names (`TestParser_*`, `TestWalk_*`) and subtests for per-fixture coverage.

## Commit & Pull Request Guidelines
- Match existing commit style: concise, imperative subjects like `Add support for ...` or `Fix parsing failure ...`, optionally with issue refs (for example `(#235)`).
- Keep PRs focused; describe grammar/AST impact, include representative SQL examples, and note regenerated fixtures.
- Before requesting review, run `make lint` and `make test` locally to mirror CI expectations.


## Important Notes

- You must confirm it's correctly added to `visitor.go`, `walk.go` and `format.go` when adding a new expression or statement type. This ensures that the new AST node is properly traversed and formatted.
- Newly added test cases must be concise and cover the core functionality being tested first.

---
> Source: [AfterShip/clickhouse-sql-parser](https://github.com/AfterShip/clickhouse-sql-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
