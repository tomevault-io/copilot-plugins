## selfhost-s3

> - No self-promotional footers, signatures, or "Generated with Claude" branding

# Claude Code Rules

## Git Commits

- No self-promotional footers, signatures, or "Generated with Claude" branding
- No "Co-Authored-By: Claude" attribution
- Keep commit messages clean and focused on changes only

## GitHub Releases

- No self-promotional content in release notes

## Project Structure

- Go project: `cmd/`, `internal/` (auth, config, server, storage), `integration/`
- Entry point: `cmd/selfhost_s3/main.go`

## Testing

- don't write weak tests, if a test fails make sure the issue does not come a bad implementation of the feature before fixing the test
- `make test-unit` - unit tests
- `make test-integration-up` - start container + run integration tests
- `make test-integration-down` - stop container

## Build

- `make build` - build binary to `bin/`

## Code Quality

- Run `gofmt -w` and `golangci-lint run --fix` before committing Go files

---
> Source: [Notifuse/selfhost_s3](https://github.com/Notifuse/selfhost_s3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
