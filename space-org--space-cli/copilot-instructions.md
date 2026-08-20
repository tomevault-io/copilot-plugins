## space-cli

> This file provides durable guidance to Codex and other coding agents working in this repository.

# AGENTS.md

This file provides durable guidance to Codex and other coding agents working in this repository.

## Project Overview

CLI tool for querying and writing Space flashcard data directly from/to the local PowerSync SQLite database. No network calls — operates on the synced local DB file. Supports both read (list, search, export, stats) and write (add decks/cards/groups, edit cards, reorder) operations.

Part of the Space monorepo (`../`). The Flutter frontend (`../space-frontend/`) owns the database; this CLI accesses it concurrently via WAL mode.

## Commands

```bash
dart pub get                                          # Install dependencies
dart analyze                                          # Static analysis
dart test                                             # Run all tests
dart run bin/space.dart <command>                      # Run directly
dart compile exe bin/space.dart -o bin/space           # Compile native binary
```

After compiling, `bin/space` is on PATH (configured in `~/.zshrc`).

## Architecture

**Entry point** (`bin/space.dart`): Parses global options (`--db-path`, `--no-color`, `--version`), creates two lazy database wrappers — `readDb` (read-only) and `writeDb` (read-write) — so `--help` works without a DB. Registers the three umbrella commands (`DeckCommand`, `CardCommand`, `GroupCommand`) with `CommandRunner` from the `args` package.

**Database layer** (`lib/src/db/`):
- `database.dart` — Abstract `SpaceDatabase` interface with `open()` factory. Accepts `readOnly` parameter (default `true`). Provides `query()`, `queryOne()`, `execute()`, and `transaction()` methods. Validates `ps_data__Deck` table exists.
- `queries.dart` — All SQL as string constants. Read queries use PowerSync views (`"Deck"`, `"Card"`, `"LearningState"`, `"DeckMember"`, `"Repetition"`, `"CardGroup"`) which auto-extract JSON fields from `ps_data__*` tables. Write queries use `INSERT OR REPLACE` directly on `ps_data__*` tables (bypassing views).

**Commands** (`lib/src/commands/`): Resource-verb pattern. Three resource umbrella commands group sub-commands by verb, and `mcp` runs the local protocol server. Each sub-command extends `Command<void>` and receives `SpaceDatabase` via constructor. Resource commands support prefix-matching on IDs (e.g. `space card show ck3u` resolves to the full UUID).

| Resource | Sub-commands |
|----------|--------------|
| `deck`   | `create`, `show`, `list`, `export`, `groups`, `cards`, `stats` |
| `card`   | `create`, `show`, `edit`, `delete`, `search`, `reorder`, `add-media` |
| `group`  | `create`, `show`, `edit`, `delete`, `cards`, `move-cards` |

Write commands (`create`, `edit`, `delete`, `reorder`, `move-cards`, `add-media`) use `writeDb`; read commands use `readDb`.

**Shared operations** (`lib/src/operations/`): Presentation-neutral deck and card operations shared by human CLI commands and MCP tools. PowerSync write transactions, strict ID resolution, typed results, and stable operation errors belong here. This layer must not import `args`, terminal formatting, or MCP types.

**Local MCP adapter** (`lib/src/mcp/` + `McpCommand`): `space mcp` serves six bounded tools over stdio using `dart_mcp`. It maps schemas and structured MCP results to the shared operations layer. Stdout is protocol-only; diagnostics go to stderr without user content. MCP handlers must not duplicate SQL, spawn the CLI, expose raw SQL, add network calls, or let uncaught SDK errors disclose stack traces.

**Models** (`lib/src/models/`): Plain Dart classes with `fromRow(Map<String, dynamic>)` factories. No codegen. Models: `deck`, `card`, `learning_state`, `card_group`.

**Utils** (`lib/src/utils/`): Ported from `space-frontend` since Flutter projects can't be used as Dart dependencies:
- `fsrs.dart` — FSRS retrievability (`R = (1 + F*t/S)^D`) and strength (`S/(S+20)*R`) formulas
- `card_maturity.dart` — Maturity classification from FSRS state (New/Learning/Young/Mature)
- `utc_datetime.dart` — `parseUtcDateTime()` to handle PostgreSQL stripping the Z suffix
- `platform_paths.dart` — Resolves DB path per platform (macOS sandboxed + non-sandboxed, Linux, Windows)
- `id_resolver.dart` — Resolves ID prefixes to full UUIDs across decks, cards, and groups
- `pending_media_paths.dart` — Resolves the Flutter app's `pending_media/` directory per platform; `card add-media` drops files there as localhost URLs so the running app picks them up via `MediaUploadService` on next sync

**Formatting** (`lib/src/formatting/`): ANSI colors (with `--no-color` support), aligned table printer, HTML tag stripper for terminal display.

## Database Details

- **macOS (App Store):** `~/Library/Containers/app.getspace.space/Data/Library/Application Support/app.getspace.space/space.db`
- **macOS (dev):** `~/Library/Application Support/app.getspace.space/space.db`
- Platform path auto-detection tries sandboxed path first, falls back to non-sandboxed.
- PowerSync views filter `deletedAt IS NULL` for soft deletes.
- LearningState joins use `MAX(createdAt)` subquery to get the latest state per card.

## Distribution & Release

The CLI ships as native binaries via GitHub Releases. Release assets cover four platforms (macOS-x86_64 was dropped):

| Platform | Asset |
|----------|-------|
| macOS (Apple Silicon) | `space-cli-macos-arm64` |
| Linux (arm64) | `space-cli-linux-arm64` |
| Linux (x86_64) | `space-cli-linux-x86_64` |
| Windows (x86_64) | `space-cli-windows-x86_64.exe` |

Distribution channels:
- **Homebrew Tap** — `space-org/tap/space-cli` (formula in `homebrew/`)
- **curl installer** — `scripts/install.sh` for Linux/macOS one-liners
- **GitHub Releases** — direct binary downloads

Release infra lives in `homebrew/`, `scripts/`, and the release workflows under `.github/workflows/`.

## Keeping Utils in Sync

`fsrs.dart`, `card_maturity.dart`, and `utc_datetime.dart` are manual copies from `space-frontend`. Source of truth:
- `../space-frontend/lib/services/fsrs/fsrs_service.dart` (FSRS constants + formulas)
- `../space-frontend/lib/services/fsrs/card_maturity_service.dart` (maturity logic)
- `../space-frontend/lib/data/local/utc_date_time.dart` (UTC parsing)

If the frontend changes these, update the CLI copies accordingly.

---
> Source: [space-org/space-cli](https://github.com/space-org/space-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
