## repokai

> A GitHub repository browser with both terminal and desktop interfaces.

# RepoKai

A GitHub repository browser with both terminal and desktop interfaces.

## Workspace Structure

Cargo workspace with three crates under `crates/`:

```
crates/core/   — shared library (GitHub API)
crates/tui/    — terminal UI (ratatui)
crates/gui/    — desktop app (Tauri v2)
```

## Crate Responsibilities

### core (`repokai-core`)
- Authenticates with GitHub via `GITHUB_TOKEN` env var using octocrab
- Fetches the authenticated user's repositories with pagination
- Fetches README content (base64-decoded from GitHub API)
- Exports: `Repo` struct, `create_client()`, `get_authenticated_user()`, `fetch_repos()`, `fetch_starred_repos()`, `fetch_readme()`
- `Repo` fields: owner, name, description, url, language, license, stars, visibility, last_updated, readme

### tui (`repokai-tui`)
- Terminal UI with 3-panel layout: left repo list (30%), top-right info (35%), bottom-right README (65%)
- Info panel: one row per field except Description, which word-wraps with a hanging indent under the value column; a per-render row→field map (`App::info_rows`) keeps mouse clicks and the selection marker accurate
- Uses ratatui + crossterm backend; alternate screen + raw mode + **mouse capture** (`init_terminal`/`restore_terminal` in `main.rs` — mouse capture is turned off on exit and on panic via a chained hook, because `ratatui::restore()` doesn't know about it)
- `src/render.rs` renders the README as styled markdown — ported from `~/dev/main/mdview` and kept visually identical: heading colors with `═`/`─` underbars, syntect-highlighted code blocks, box-drawn tables pre-fit to the panel width, blockquotes, task lists, links. The rendered `Text` is cached on `App` (`readme_rendered`) and rebuilt only when the content or the panel inner width changes; syntect's `SyntaxSet`/`Theme` load once via `LazyLock` statics. Placeholders ("Loading...", "Press Enter to load README") stay plain text
- Scroll model: the repo list owns a view offset (`repo_offset`) and renders with `ListState` whose selection stays `None` — a `Some` selection would snap the viewport to it every frame and fight view-only wheel scrolling; keyboard nav pulls the offset along via `ensure_selected_visible()`. README scrolling clamps to the wrapped line count (`Paragraph::line_count`, ratatui feature `unstable-rendered-line-info`), leaving one blank row past the end; a DarkGray scrollbar appears on the README's right border when content overflows
- Navigation: arrows/j/k browse repos (auto-loads README), Tab/Shift-Tab switch panels, Enter reloads README / opens URL, j/k + PageUp/PageDown scroll the README, mouse wheel scrolls the panel under the cursor (repo list scrolls the view only — selection stays put, no fetches), left-click focuses a panel and selects the clicked repo/info row, v toggle my repos/starred (starred fetched lazily, cached per session; r refreshes the active source), s sort, r refresh, o open, p publish, c clone, e edit, q/Esc quit
- Styled after PanEx TUI: green active border, cyan titles, DarkGray inactive borders, blue folder names

### gui (`repokai-gui`)
- Tauri v2 desktop app with the same 3-panel layout
- Frontend in `crates/gui/ui/` — plain HTML/CSS/JS, no bundler
- PanEx "TUI" theme: Tokyo Night palette (`#0f0f17` bg, `#7aa2f7` accent), monospace font, sharp corners, border-breaking panel titles
- Tauri commands: `get_user`, `get_repos`, `get_starred_repos`, `get_readme`, `publish_repo`, `clone_repo`, `update_repo` — thin wrappers around core
- Navigation: Arrow keys + click, same keyboard nav as TUI (except the my/starred toggle is `v` in both, and `t` cycles themes in the GUI)

## Running

```bash
# Terminal UI
GITHUB_TOKEN=ghp_... cargo run -p repokai-tui

# Desktop GUI
GITHUB_TOKEN=ghp_... cargo run -p repokai-gui
```

## Verifying TUI changes

The TUI can't be driven from a non-tty shell (`enable_raw_mode` fails). Smoke-test with `expect` and a pty:

```sh
expect -c '
spawn ./target/debug/repokai
expect -re "Repositories"
sleep 1
send "j"; sleep 0.5
send "q"
expect eof
'
```

Alt-screen enter/leave (`[?1049h`/`[?1049l`) plus mouse capture off (`[?1000l`) in the output with exit 0 means the boot/teardown path is fine. Mouse events can be injected as SGR sequences (e.g. wheel-down `\x1b[<65;10;10M`, left click `\x1b[<0;10;5M`). Note ratatui renders cell diffs, so don't grep the raw stream for full strings — reconstruct the final screen (replay cursor moves, e.g. with python `pyte`) or force a full repaint first. Visual correctness needs a real terminal — say so rather than claiming success.

Two pty traps that produce phantom bugs:
- expect only drains the pty during `expect` commands. During `sleep`, the app's frame writes fill the pty buffer and **block the event loop mid-`terminal.draw`**; keys sent meanwhile pile up in the kernel and get read as one chunk, so Esc + next key parses as Alt+key ("Esc doesn't work" — it's the harness, not the app). Wait with `expect -timeout N "ZZZ_NEVER_MATCHES"` instead of `sleep N`, and give the pty a size (`stty rows 35 columns 110 < $spawn_out(slave,name)` after spawn — a 0×0 pty renders nothing).
- Wait for README content to render (`expect -re` on some README text) before sending keys: the first render is slow in debug builds (syntect), and keys sent during it coalesce the same way.

## Coding Conventions

- Rust 2021 edition, workspace version/edition inherited
- Async/await throughout (tokio runtime for TUI, Tauri runtime for GUI)
- Errors: `thiserror` in core, `.map_err(|e| e.to_string())` at Tauri command boundaries
- GUI frontend is vanilla JS — no frameworks, no build step
- All colors/styles reference the PanEx design system (see `~/.claude/themes/`)
- Keep core free of UI concerns — it only does data fetching

---
> Source: [Ivapo/RepoKai](https://github.com/Ivapo/RepoKai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
