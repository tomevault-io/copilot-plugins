## zsp

> CLI for publishing Android apps to Nostr relays.

# zsp — Agent Instructions

CLI for publishing Android apps to Nostr relays.

All behavioral authority lives in `spec/guidelines/`. If this file conflicts, guidelines win.

## Quick Reference

| What | Where |
|------|-------|
| Architecture & patterns | `spec/guidelines/ARCHITECTURE.md` |
| Non-negotiable rules | `spec/guidelines/INVARIANTS.md` |
| Quality standards | `spec/guidelines/QUALITY_BAR.md` |
| Product vision | `spec/guidelines/VISION.md` |
| Feature specs | `spec/features/` |
| Active work | `spec/work/` |
| Decisions & learnings | `spec/knowledge/` |

Guidelines are symlinked into `.cursor/rules/` and auto-load.

## File Ownership

| Path | Owner | AI May Modify |
|------|-------|---------------|
| `spec/guidelines/*` | Human | No |
| `spec/features/*` | Human | No (unless asked) |
| `spec/work/*.md` | AI | Yes |
| `spec/knowledge/*.md` | AI | Yes |
| `internal/**`, `main.go` | Shared | Yes |

## Key Commands

```bash
go build -o zsp .     # Build
go test ./...         # Tests
go vet ./...          # Lint
go mod tidy           # After dependency changes
```

## Project Rules

- Reference `internal/source/github.go` when adding new sources.

---
> Source: [zapstore/zsp](https://github.com/zapstore/zsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
