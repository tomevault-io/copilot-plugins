## docbase-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
make build      # Build the docbase binary
make test       # Run tests with go test -v ./...
make lint       # Run golangci-lint
make fmt        # Format code with gofmt -s
make install    # Build and install to $GOPATH/bin
make clean      # Remove binary and dist/
```

Cross-compilation: `make build-darwin`, `make build-linux`, `make build-windows`, or `make build-all`.

## Architecture

This is a CLI tool for the DocBase API (https://help.docbase.io/posts/45703) built with Go 1.25 and Cobra.

### Package Structure

Package names follow Go conventions: https://go.dev/blog/package-names

- **`main.go`** - Entry point, imports all command packages
- **`pkg/docbase/`** - Reusable API client SDK (independent of CLI)
  - `docbase.go` - API facade with `NewAPI()` that creates services
  - `client.go` - HTTP client wrapper using resty
  - `memo.go`, `group.go`, `tag.go`, `comment.go` - Service implementations
  - `models.go` - Data structures
- **`cmd/`** - CLI commands (each subcommand in its own package)
  - Commands register via `init()` calling `root.AddCommand()`
  - Thin wrappers that call pkg/docbase services
- **`internal/`** - Shared CLI utilities
  - `config/` - JSON config at `~/.config/docbase/config.json`
  - `client/` - Client factory (`Create()`) and flag helpers
  - `formatter/` - Output formatting (text, JSON, YAML)
  - `format/` - String formatting helpers
  - `output/` - Colored terminal output
  - `fileio/` - File/stdin reading
  - `groups/` - Group name-to-ID resolution

### Key Patterns

- **Service Object Pattern**: `docbase.API` holds `MemoService`, `GroupService`, etc. Each service handles one resource type.
- **Config Priority**: Flag value → config file → default. Use `config.GetTeamDomain(flagValue)` pattern.
- **Command Error Handling**: Commands use `RunE` and return errors; root handles display.
- **Import Organization**: Local imports prefixed with `github.com/basi/docbase-cli` (enforced by goimports).

---
> Source: [basi/docbase-cli](https://github.com/basi/docbase-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
