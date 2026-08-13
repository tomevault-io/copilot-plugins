## bucky

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Bucky** is a Midnight Commander-style dual-panel TUI file manager for S3 and SFTP, written in Go using the Charm Bracelet stack (Bubble Tea, Bubbles, Lipgloss).

The implementation plan lives in `docs/PLAN.md` — read it before starting any work.

## Build & Run

```bash
go build -o bucky ./cmd/bucky
go run ./cmd/bucky
go install ./cmd/bucky          # installs 'bucky' to $GOPATH/bin
```

## Test

```bash
go test ./...                    # all tests
go test ./internal/backend/...   # single package
go test -run TestName ./path/    # single test
```

## Lint & Format

```bash
gofmt -w .
go vet ./...
```

## Architecture

**Backend interface** (`internal/backend/`) is the core abstraction — `S3Backend` and `SFTPBackend` implement a common `Backend` interface so UI code is backend-agnostic. Get this right first; everything else depends on it.

**UI layer** (`internal/ui/`) uses Bubble Tea's Elm architecture (Model/Update/View). Key components:

- `app.go` — root model, orchestrates panels and transfer queue
- `browser/` — file browser panel (one per side of the dual-panel layout)
- `transfer/` — transfer queue display with progress bars
- `status/` — status bar with keybinding hints

**Transfer engine** (`internal/transfer/`) runs outside the Bubble Tea event loop using goroutines, sending progress back via `program.Send()`. Supports concurrent transfers with configurable limits.

**Config** (`internal/config/`) — TOML profiles at `~/.config/bucky/config.toml` managed with Viper.

## Key Dependencies

- `charmbracelet/bubbletea` — TUI framework
- `charmbracelet/bubbles` — table, textinput, spinner, progress components
- `charmbracelet/lipgloss` — styling
- `aws/aws-sdk-go-v2` — S3 backend
- `pkg/sftp` + `golang.org/x/crypto/ssh` — SFTP backend
- `spf13/viper` — config management

## Build Order

Follow `docs/PLAN.md` suggested build order: backend abstraction → SFTP impl → config → single-panel browser → dual-panel → transfer engine → keybindings → profile picker → polish.

## Changelog Generation

You are a changelog agent. Your job is to analyze git commit history for a repository and produce a well-structured CHANGELOG.md file following the Keep a Changelog format (<https://keepachangelog.com>).

1. Run `git log --pretty=format:"%H|%ad|%an|%s|%b" --date=short` to get commit history
2. Run `git tag --sort=-version:refname` to get release tags
3. Group commits between tags into versioned sections — if no tags exist, group by month
4. Classify each commit into one of these categories:
   - **Added** – new features (`feat:`)
   - **Changed** – refactors, updates, dependency bumps (`chore:`, `refactor:`, `bump:`)
   - **Fixed** – bug fixes (`fix:`, `hotfix:`)
   - **Removed** – deleted code or features (`remove:`, `drop:`)
   - **Security** – vulnerability patches (`security:`, `patch:`)
5. Skip merge commits, bot commits, and vague messages (`wip`, `typo`, `temp`, single words)
6. Rewrite commit messages into clear, human-readable past-tense entries — no hashes, no author names
7. Write output to `CHANGELOG.md` in Keep a Changelog format — newest version first, `[Unreleased]` section at the top
8. The `CHANGELOG.md` file should live in the root of the directory

---
> Source: [mrpbennett/bucky](https://github.com/mrpbennett/bucky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
