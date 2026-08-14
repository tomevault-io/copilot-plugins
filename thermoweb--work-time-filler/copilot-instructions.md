## work-time-filler

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

WTF (Work Time Filler) is a Rust TUI application that helps developers fill Jira worklogs by integrating Google Calendar meetings, GitHub activity, and sprint data. The main workflow is a guided "Chronie Wizard" that syncs data, auto-links meetings to Jira issues, creates worklogs, and pushes them to Jira.

## Commands

```bash
cargo build --workspace          # build
cargo test --workspace           # all tests
cargo test -p wtf-lib            # lib tests only (faster)
cargo test <test_name>           # single test by name
cargo clippy --workspace         # lint (required before commits)
cargo fmt --all                  # format (CI auto-formats on push)
cargo run                        # run TUI (dev mode)
cargo run -- <subcommand>        # run specific CLI command
```

> **Note:** `cargo test -p wtf-lib` requires the `[dev-dependencies]` tokio `rt-multi-thread` feature — already added. Running `-p wtf-cli` tests will fail due to TUI/terminal dependencies.

Coverage (lib only):
```bash
cargo tarpaulin --out Lcov --output-dir coverage/ --workspace --exclude-files 'wtf-cli/*'
```

To test with an isolated config (won't touch your real data):
```bash
WTF_CONFIG_HOME=/tmp/wtf-test cargo run -- init
```

## Architecture

The workspace has two crates:

- **`wtf-lib`** — all business logic, data models, services, API clients, and storage. No TUI dependencies.
- **`wtf-cli`** — CLI entry point, TUI, and async task orchestration. Depends on `wtf-lib`.

### wtf-lib structure

| Layer | Path | Purpose |
|---|---|---|
| Models | `src/models/data.rs` | Core structs: `Sprint`, `Meeting`, `Issue`, `LocalWorklog`, `GitHubSession`, etc. |
| Services | `src/services/` | Business logic: `MeetingsService`, `WorklogsService`, `JiraService`, `GitHubService`, `GoogleService`, `AchievementService` |
| Clients | `src/client/` | HTTP clients for Jira and GitHub APIs, with pagination helpers |
| Storage | `src/storage/database.rs` | `GenericDatabase<T>` wrapper around sled; requires `T: Identifiable + Serialize + Deserialize` |
| Config | `src/config.rs` | TOML config loading; path via `WTF_CONFIG_HOME` or `~/.config/wtf/config.toml` |

**Services** are stateless structs owning a `GenericDatabase`. They have a `.production()` constructor that opens the default DB path. The test pattern is to construct them with `Database::temporary()`.

**`Sprint::contains_meeting(&self, meeting: &Meeting) -> bool`** is the canonical way to check if a meeting belongs to a sprint — it expands sprint boundaries to full UTC days (00:00–23:59:59) so meetings aren't missed due to sprint hour offsets. Do not inline this logic elsewhere.

### wtf-cli structure

| Path | Purpose |
|---|---|
| `src/main.rs` | Clap CLI setup, command dispatch |
| `src/commands/` | One file per CLI subcommand |
| `src/tui/mod.rs` | `Tui` struct, event loop, keyboard handling (~2100 lines — known violation) |
| `src/tui/data.rs` | `TuiData` — collects all state from services at startup |
| `src/tui/types.rs` | State enums: `Tab`, `WizardStep`, `WizardState`, `PopupState`, etc. |
| `src/tui/wizard.rs` | Chronie Wizard steps (state machine) |
| `src/tui/tab_controller.rs` | `TabController` trait — shared contract each tab implements for rendering and key handling |
| `src/tui/ui/` | Rendering split by tab: `tabs/sprints.rs`, `tabs/meetings.rs`, `tabs/worklogs.rs`, etc. |
| `src/tui/operations/` | Async task wrappers that call services and report progress back to the TUI |
| `src/tasks/` | Background async tasks (implement `Task` trait) |

### Data flow

1. `TuiData::load()` — calls services at startup, populates in-memory state
2. Event loop — renders frame → reads keyboard → dispatches to handler
3. Handlers call `operations/` functions which spawn a new tokio `Runtime` per operation (known architectural issue — don't add more of these)
4. Operations update `TuiData` fields; next render pass picks up the changes

## Conventions (from `doc/BEST_PRACTICES.md`)

**Read `doc/BEST_PRACTICES.md` before writing code.** Key rules:

- **Function parameters**: max 5–7. Pass the owning struct instead of individual fields when multiple params come from it.
- **File size**: < 500 lines preferred, never > 1000 without justification. `tui/mod.rs` at ~2000 lines is a known violation — don't make it larger.
- **mod.rs**: declarations and re-exports only. No implementation logic.
- **Feature cohesion**: keep rendering, keyboard handling, popup state, and filtering rules with the feature that owns them. Don't scatter one feature's logic across generic files.
- **Do not modify `doc/BEST_PRACTICES.md`** — suggest changes in PR descriptions instead.

## Known architectural issues

See `doc/ARCHITECTURE_ISSUES.md` for the full list. The most relevant:

- Each async operation spawns a new `tokio::Runtime` — expensive but pervasive; don't add new instances.
- Services use `lazy_static` global databases — this prevents unit-testing services in isolation; use `Database::temporary()` for tests. `AchievementService` is the reference DI implementation — follow its pattern when refactoring other services.
- `TuiData` is a large god struct — new fields are acceptable for now but consider whether they belong in a sub-struct.

---
> Source: [thermoweb/work-time-filler](https://github.com/thermoweb/work-time-filler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
