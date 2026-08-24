## pkg

> Platform-wide Go packages for Latere AI.

# pkg

Platform-wide Go packages for Latere AI.

## Gold Standards

Every package in this repo must meet these requirements:

### Testing

- All packages must have `_test.go` files with unit tests.
- Run tests with the race detector: `go test -race ./...`
- Include fuzz tests (`FuzzXxx`) for functions that accept string or byte inputs.
- Use `t.Setenv` and `t.Cleanup` for test isolation — no global state leaks between tests.

### Coverage

- Statement coverage must be **>= 95%** at all times.
- CI enforces this threshold; a PR that drops coverage below 95% will fail.
- Use package-level function variables for external constructors to make error paths testable without adding dependencies.

### Dependencies

- Minimize external dependencies. Prefer the standard library.
- Do not add test-only dependencies — use `net/http/httptest`, `errors`, etc. from stdlib.
- When adding a new direct dependency, justify it.

## Commands

```
make test       # run tests
make race       # run tests with race detector
make fuzz       # run fuzz tests (30s)
make cover      # run tests + enforce 95% coverage floor
make cover-html # open coverage report in browser
```

---
> Source: [latere-ai/pkg](https://github.com/latere-ai/pkg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
