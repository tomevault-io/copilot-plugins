## arta

> cargo build                  # dev build

# ARTA - Development Guidelines

## Build & Run

```bash
cargo build                  # dev build
cargo build --release        # release build
cargo run                    # run in dev mode
cargo test                   # run unit tests
```

Binary output: `target/debug/arta` (dev) or `target/release/arta` (release). The crate is published to crates.io as `arta-tui` (the binary stays `arta` via `[[bin]]`).

## Project Structure

```
src/
├── main.rs           # Entry point, terminal setup, event loop
├── app.rs            # App state, focus routing, event dispatch, agent-title polling
├── claude_hook.rs    # Idempotent merge of Notification hook into ~/.claude/settings.json
├── config.rs         # Config loading (YAML), env vars, session naming
├── context_bar.rs    # Command hint lines (global:/context: under the terminal): action tables + renderers
├── host_term.rs      # Host-terminal escapes: BEL bell, OSC 52 clipboard
├── multiplexer.rs    # MultiplexerBackend trait, TmuxBackend, ZellijBackend
├── sidebar.rs        # Sidebar widget (project/thread tree)
├── terminal_pane.rs  # PTY lifecycle, vt100 parser, reader thread
├── input_panel.rs    # Text input with path completion
├── welcome.rs        # Welcome screen ASCII art
├── workspace.rs      # WorkspaceStore: Project/Workspace/Thread persistence (YAML, serde aliases for legacy `sessions:` keys)
└── keys.rs           # KeyEvent → raw PTY bytes
```

## Vocabulary

- **thread** — the user-facing unit (what's shown in the sidebar). State type is `Thread` in `workspace.rs`.
- **workspace** — a user-facing *group of projects* (collapsible sidebar header). State type is `Workspace` in `workspace.rs`. A project belongs to at most one workspace (`Project.workspace`). NOT the persisted document — that's `WorkspaceStore` (still saved as `workspace.yaml`).
- **session** — preserved for tmux/zellij vocabulary only: CLI args, env vars (`ARTA_SESSION_PREFIX`), backing-session names (`arta_t_myproj-1`), and `MultiplexerBackend` methods.

## Sidebar model notes

- The sidebar's top level is an explicit ordered list, `WorkspaceStore.top_order`
  (`Vec<TopLevelRef>` — workspaces and ungrouped projects intermixed), repaired by
  `normalize_top_order()` on load and after membership changes. Projects inside a
  workspace keep their relative order in the `projects` Vec (`swap_project_in_group`).
- Workspace open/closed state persists (`Workspace.open`); project expand state is
  in-memory only (`Sidebar.expanded`, all-open at startup).
- Setting resolution is project override > workspace override > global config
  (`effective_agent_command` / `effective_open_command` in `app.rs`).
- The two lines under the terminal's bottom border are the command hint lines
  (`context_bar.rs`, pure functions): `context:` first (selection/menu keys),
  `global:` below it (always-available Ctrl+Space commands; contextual
  bindings shadow globals). Messages share the `context:` slot — red
  `status_message` wins, then the 3s `timed_message`, then the hints. Layout math lives in
  `App::main_height()` (single source of truth for `render()` and `pane_size()`).

## Configuration

Config root: `~/.arta/` (override with `ARTA_CONFIG_ROOT` env var).

- `~/.arta/config.yaml` — user settings
- `~/.arta/workspace.yaml` — project/session state

### config.yaml

```yaml
coding_agent_command: claude     # default command sent to new threads (args allowed, e.g. "claude --resume")
default_open_command: vi         # fallback for "open ide" when a project has no override; "" disables fallback
multiplexer: tmux                # tmux | zellij
bell: terminal                   # terminal (host BEL) | sound (afplay, falls back to BEL) | both
```

All fields are editable in-app via `Ctrl+Space s` (ARTA settings); the menu
calls `Config::save()` which writes the full file (no comment preservation).

Per-project agent overrides (`workspace.yaml`: `agent_command:` on a project) take
precedence over the global `coding_agent_command`. Per-project `open_command`
overrides take precedence over `default_open_command`. Both are read at
thread-create / open-IDE time respectively (not cached on App). Helpers:
`effective_agent_command()` and `effective_open_command()` in `app.rs`.

Multiplexer changes require a restart — the backend is built once at startup
and stored on `App.mux`. `Multiplexer::is_installed()` walks `PATH` directly
(no `--version` subprocess; tmux's `--version` flag spelling differs).

The legacy `multiplexer_init_script` config key is no longer supported. If still
present in `config.yaml` it is silently ignored, with a startup warning surfaced
as a timed message.

### Environment Variables

- `ARTA_CONFIG_ROOT` — config directory (default: `~/.arta/`)
- `ARTA_SESSION_PREFIX` — session name prefix for profile isolation (default: empty)

### Session Naming

Format: `arta_{prefix?}_{tag}_{session_id}` where tag is `t` (tmux) or `z` (zellij).
Examples: `arta_t_myproj-1`, `arta_work_z_myproj-1`

## Commits

- Never include "Co-Authored-By" lines in commit messages

## Releases

- Update `CHANGELOG.md` (add the `## X.Y.Z - date` section) **before** tagging:
  cargo-dist generates the GitHub release body from the matching changelog
  section. Release flow: bump version in Cargo.toml, update CHANGELOG.md,
  commit, then `git tag vX.Y.Z && git push && git push --tags`.

---
> Source: [cjurjiu/arta](https://github.com/cjurjiu/arta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
