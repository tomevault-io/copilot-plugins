## irons

> - Use PRs to stage changes for review. Never push directly to main.

# CLAUDE.md

## General Rules

- Use PRs to stage changes for review. Never push directly to main.
- In PR descriptions, never include test plans. Just include a summary.
- Run go mod tidy before committing.

## Testing

- Run `go test ./...` before committing to verify nothing is broken.
- Never hit external/remote resources in tests. Use `httptest.NewTLSServer` or similar local test servers.
- Use `github.com/stretchr/testify/require` for assertions whenever possible.

---
> Source: [ironsh/irons](https://github.com/ironsh/irons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
