## retri

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Retri is a lightweight CLI tool for automating SSH command execution and log collection across multiple servers. Written in Go, it supports Linux servers and network devices (Cisco IOS, Arista EOS, Juniper, Huawei) with interactive PTY-based prompt handling.

## Build & Test

```bash
CGO_ENABLED=0 go build -o retri -ldflags="-s -w" .
go test ./...
go vet ./...
```

## Architecture

```
main.go                  # embed + main() -> cli.Run()
internal/
  cli/cli.go             # CLI options, Run()
  config/                # structs, loading, validation, host range, target resolution
  executor/              # SSH execution (linux.go: direct exec, interactive.go: PTY, record.go: local PTY recording)
  logger/                # line-by-line writer with timestamps and ANSI stripping
configs/
  config.yaml            # embedded default config (created on first run)
```

## Key Design Decisions

- Linux hosts use direct SSH exec; network devices use PTY interactive sessions
- Config layering: defaults < groups < device_types < hosts < env vars < CLI flags
- Uses system `ssh` command (not a Go SSH library) — respects `~/.ssh/config`
- `${VAR}` syntax in YAML is expanded at runtime

## Release

Tag push (`v*`) triggers GoReleaser via GitHub Actions → publishes `.deb` (amd64/arm64) to GitHub Releases.

## Branch Workflow

- **Always branch before pushing** — never push directly to `main`
- **One feature per branch** — keep features isolated (e.g., `feat/xxx`, `fix/xxx`, `chore/xxx`)
- Merge via PR on GitHub
- **No interactive confirmation needed** — create branch, implement, and push without asking
- **Security checks are mandatory** — verify for security issues before making changes, before committing, and before pushing

---
> Source: [cotta-dev/retri](https://github.com/cotta-dev/retri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
