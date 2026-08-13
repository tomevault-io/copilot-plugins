## scuta

> A unified CLI that manages SID's developer tools. Engineers install Scuta once, then use it to install, update, and discover all SID CLI tools.

# Scuta — SID Developer Toolbox

A unified CLI that manages SID's developer tools. Engineers install Scuta once, then use it to install, update, and discover all SID CLI tools.

## Quick Reference

| Command | Description |
|---------|-------------|
| `scuta init` | Setup ~/.scuta/, detect auth, configure PATH |
| `scuta install <tool>` | Install a tool from the registry |
| `scuta install --all` | Install all tools |
| `scuta uninstall <tool>` | Remove a tool |
| `scuta update [tool]` | Update one or all tools |
| `scuta list` | Show all tools + versions + install status |
| `scuta doctor` | Health check (PATH, binaries, state) |
| `scuta self-update` | Update Scuta itself |
| `scuta version` | Print version |

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `cmd/` | CLI commands (verb-based: install.go, update.go, list.go) |
| `lib/` | Domain packages (noun-based: registry/, installer/, output/) |
| `docs/` | Architecture and planning docs |

## Architecture

- Scuta is purely a **tool manager** — no domain tool code lives here
- **Registry** is a YAML manifest (go:embed + remote fetch from GitHub)
- **Tools** live in their own repos (pilum, and others as they ship)
- **State** lives in `~/.scuta/` (config, installed versions, update timestamps)

## Development Guidelines

- **Errors**: Use `lib/errors.Wrap()` — never `fmt.Errorf`
- **Output**: Use `lib/output` — never `fmt.Print*` or `log`
- **Exit codes**: Use `lib/exitcodes` constants
- **Style**: Exit early, no else blocks, no one-line error checks
- **Linting**: `make lint` runs pre-commit + golangci-lint
- **Testing**: `make test` runs all tests with race detector

## Persistent Flags

| Flag | Description |
|------|-------------|
| `--verbose` | Show detailed output |
| `--quiet` | Suppress non-error output |
| `--json` | Output in JSON format |

## Dependencies

| Package | Purpose |
|---------|---------|
| cobra | CLI framework |
| pkg/errors | Error wrapping |
| yaml.v3 | Registry/config parsing |

---
> Source: [SID-Technologies/Scuta](https://github.com/SID-Technologies/Scuta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
