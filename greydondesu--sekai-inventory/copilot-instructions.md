## sekai-inventory

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build
go build

# Run (Windows)
.\sekai-inventory.exe <command> [arguments]

# Lint (requires golangci-lint installed)
golangci-lint run

# Test
go test ./...
```

## Architecture

Single-module Go CLI with three packages under the root module:

- **`function/`** — One file per CLI command (`add`, `remove`, `change`, `search`, `list`, `update`, `convert`, `init`, `help`). Each exports a single top-level function called by `main.go`.
- **`model/`** — Data structs: `Card` (immutable game data), `CardEntity` (extends Card with user state: Level 1–60, MasteryRank 0–5, SkillLevel 1–4, SideStory1/SideStory2/Painting bool), `Inventory` (CardEntity slice + timestamps), `Character`.
- **`tools/`** — Storage I/O (`storage.go`), HTTP fetching from Sekai-World GitHub (`fetcher.go`), output formatting/colorization (`list_item.go`, `utils.go`), and update metadata (`metadata.go`).

`main.go` handles all argument parsing and routes to `function/` — there is no flag library; args are parsed manually.

## Data files (`res/`)

| File | Purpose |
|------|---------|
| `inventory.json` | User's card collection (persisted locally) |
| `cards.json` | Game card database (fetched from Sekai-World) |
| `gameCharacters.json` | Character metadata (fetched) |
| `skills.json` | Skills data (fetched) |
| `metadata.json` | Last fetch timestamps + Git commit ID (used to skip redundant fetches) |

## Key conventions

- Inventory is always sorted by card ID after any mutation.
- Linter is strict: `gofumpt`, `goimports`, `gosec`, `errcheck`, `gocyclo` (max 15) are all enabled via `.golangci.yml`.
- Colors use `github.com/fatih/color` with hex RGB values for attributes, rarities, and levels.
- The `update` command compares the upstream Git commit ID against `metadata.json` and skips downloading if already current.

---
> Source: [GreydonDesu/sekai-inventory](https://github.com/GreydonDesu/sekai-inventory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
