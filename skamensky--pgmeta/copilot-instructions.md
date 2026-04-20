## pgmeta

> This rule ensures that when modifying Go files, developers maintain code quality and consistency by adding/updating tests, formatting code, and fixing linting errors.

When modifying Go files:
- **Add or update tests** if the changes represent functional or semantic modifications.
- **Run `go fmt`** on all changed files to maintain proper formatting.
- **Run `golangci-lint run`** and fix any errors or warnings found.

Following this rule helps ensure Go code remains well-tested, properly formatted, and free of common issues.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skamensky) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
