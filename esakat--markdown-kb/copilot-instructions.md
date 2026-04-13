## markdown-kb

> Git リポジトリ内の YAML frontmatter 付き Markdown を閲覧・検索するシングルバイナリ Web アプリ（Go + Preact）。

# Markdown KB - Development Guide

## Project Overview
Git リポジトリ内の YAML frontmatter 付き Markdown を閲覧・検索するシングルバイナリ Web アプリ（Go + Preact）。

## Commands
- `make build` - Build binary
- `make test` - Run all tests
- `make cover` - Run tests with coverage
- `make lint` - Run golangci-lint
- `go test ./... -v -race` - Run tests with race detection

## Architecture
```
cmd/kb/           - CLI entrypoint (cobra)
internal/
  config/         - Configuration types
  parser/         - Frontmatter + Markdown parsing
  index/          - SQLite FTS5 search index
  server/         - HTTP server + routes
  api/            - API response types
  git/            - Git integration (go-git + CLI)
  watcher/        - fsnotify file watcher
web/              - Preact SPA (embedded via go:embed)
```

## Key Design Decisions
- **Read-only**: Never modifies source files
- **Single binary**: SPA embedded via `//go:embed`
- **Frontmatter-first**: YAML frontmatter drives metadata, tags, filtering
- **Fast frontmatter parse**: `bufio.Scanner` stops at closing `---`
- **Hybrid Git**: go-git for structure, git CLI for blame/diff

## Tech Stack
- Go, Cobra CLI, goccy/go-yaml, goldmark
- SQLite FTS5 (modernc.org/sqlite - Pure Go)
- Preact + D3.js (frontend)
- fsnotify (file watching)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/esakat)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/esakat)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
