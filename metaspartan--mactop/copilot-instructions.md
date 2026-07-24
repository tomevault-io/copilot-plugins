## mactop

> - **Modern Go**: Use Go 1.21+ features where appropriate.

# AI Coding Rules for mactop

## General Principles
- **Modern Go**: Use Go 1.21+ features where appropriate.
- **DRY (Don't Repeat Yourself)**: Avoid code duplication. Extract common logic into helper functions or packages.
- **Clean Code**: Write readable, maintainable, and self-documenting code. No comments.
- **Error Handling**: proper error checks, wrap errors with context using `fmt.Errorf("%w", err)` or `errors.Join`.
- **Concurrency**: Use `sync.Mutex` for shared state, `channels` for communication, and `context` for cancellation.

## Formatting & Style
- Follow `gofmt` and `goimports` standards.
- Use explicit variable names (avoid single letters unless for loop indices like `i`).
- Group imports: standard library first, then 3rd party, then local.
- Ensure `make sexy` passes.

## Testing
- Write table-driven tests.
- Ensure high test coverage for logic-heavy functions.
- Use `t.Parallel()` where safe.

## Project Structure
- `internal/`: Private application code.

## Specific to mactop
- `app.go` should remain the coordinator but delegate work to specific domains (metrics, ui, parsing).
- Ensure cross-compilation compatibility where possible (though heavily macOS focused to support all Apple Silicon devices).

---
> Source: [metaspartan/mactop](https://github.com/metaspartan/mactop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
