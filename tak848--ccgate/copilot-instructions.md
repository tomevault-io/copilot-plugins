## ccgate

> Claude Code PermissionRequest hook written in Go.

# ccgate

Claude Code PermissionRequest hook written in Go.

## Install

```bash
mise use -g aqua:tak848/ccgate
# or
go install github.com/tak848/ccgate@latest
```

## Development

```bash
mise run build    # Build binary (dev)
mise run test     # Run tests
mise run vet      # Run go vet
```

## Coding conventions

- Go 1.25
- Wrap errors with `fmt.Errorf("...: %w", err)`
- Never silently discard errors
- Table-driven tests
- Named constants for magic numbers

---
> Source: [tak848/ccgate](https://github.com/tak848/ccgate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
