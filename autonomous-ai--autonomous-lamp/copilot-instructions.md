## testing

> Testing conventions


# Testing Rules

## Test Files

- Test files live next to the code they test: `internal/led/engine_test.go`
- Use standard `testing` package — no external test frameworks
- Table-driven tests preferred for multiple cases

## Running Tests

```bash
go test ./...                    # All tests
go test ./internal/led/...       # Single package
go test -run TestFunctionName    # Single test
go test -v ./...                 # Verbose output
go test -race ./...              # Race detector
```

## Conventions

- Test function names: `TestMethodName_Scenario` (e.g. `TestEncodeColors_BufferLength`)
- Use `t.Errorf` or `t.Fatalf` — not `panic` or `log.Fatal` in tests
- Test exported behavior, not internal implementation details
- Mock external dependencies (shell commands, hardware), not internal services

---
> Source: [autonomous-ai/autonomous-lamp](https://github.com/autonomous-ai/autonomous-lamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
