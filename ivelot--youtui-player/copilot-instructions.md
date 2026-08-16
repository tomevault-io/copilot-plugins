## youtui-player

> This file provides guidance to AI coding agents (Claude Code, Gemini, etc.) when working with code in this repository. `CLAUDE.md` and `GEMINI.md` are symlinks to this file.

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Gemini, etc.) when working with code in this repository. `CLAUDE.md` and `GEMINI.md` are symlinks to this file.

## Stack

- **Go 1.24** (toolchain `go1.24.7`, per `go.mod`)
- Deps (from `go.mod`/`go.sum`): `github.com/rivo/tview`, `github.com/gdamore/tcell/v2` v2.7.4, `github.com/BurntSushi/toml` v1.5.0, `github.com/nfnt/resize`
- No database, no queue, no external API service — the app shells out to local CLI tools instead (see Runtime Dependencies)
- Packaged for Arch Linux via `PKGBUILD` (AUR)

## Commands

```bash
make build       # Compile the project
make run         # Build and run
make test        # Run all tests (go test ./...)
make fmt         # Format code (go fmt ./...)
make vet         # Static analysis (go vet ./...)
make check-deps  # Verify runtime deps (mpv, yt-dlp, socat, ffmpeg)
make clean       # Remove compiled binaries
make deps        # Download and tidy Go modules
```

Run a single test: `go test ./internal/... -run TestName`

**Note:** `go`/`make` toolchain execution was permission-gated in the adoption sandbox and could not be interactively approved, so `make build`/`make test`/`make vet` were not actually run during this adoption pass. Treat them as documented-but-unverified until run once in a normal shell.

## Runtime Dependencies

The app shells out to external tools at runtime:
- `mpv` — media player backend
- `yt-dlp` — YouTube search and extraction
- `socat` — IPC communication with mpv via Unix socket
- `ffmpeg` — used by `yt-dlp` to extract audio to MP3 and merge video into MP4 on download

## Environment variables

No secrets or required env vars. The code reads a few optional XDG base-dir overrides, all with hardcoded fallbacks:

| Variable | Used in | Fallback |
|---|---|---|
| `XDG_CONFIG_HOME` | `internal/config/config.go` | `~/.config` |
| `XDG_STATE_HOME` | `internal/config/state.go` | `~/.local/state` |
| `XDG_CACHE_HOME` | `internal/ui/thumbnail.go` | `~/.cache` |
| `XDG_DATA_HOME` | `internal/ui/download.go` | `~/.local/share` |
| `XDG_DOWNLOAD_DIR` | `internal/ui/download.go` | `~/Downloads` |
| `LC_ALL` / `LC_MESSAGES` / `LANG` | `internal/config/config.go` | English (`en`) |

See `.env.example` for a reference copy of these (all optional).

## Architecture

**Module:** `github.com/IvelOt/youtui-player`

Three internal packages:

- `internal/ui/` — Main TUI layer. The central `SimpleApp` struct (defined in `app.go`) owns all state and UI components. All subsystems are wired in `setup.go`; input handling lives in `handlers.go`.
- `internal/search/` — YouTube search via `yt-dlp` subprocess; parses JSON output into `Result` structs. Three entry points: `SearchVideos()`, `GetPlaylistVideos()`, `GetVideoDetails()`.
- `internal/config/` — TOML config (`~/.config/youtui-player/youtui.conf`) and JSON session state (`~/.local/state/youtui-player/state.json`).

**Key data flow:**

1. User search → `search.SearchVideos()` spawns `yt-dlp`, returns `[]Result`
2. Playback → `playTrackSimple()` spawns `mpv` with `--input-ipc-server`; progress polling uses `socat` to query the IPC socket
3. State is auto-saved to JSON after changes and restored on startup. `PlayerState` persists search term, search results, playlist, scroll positions, current page, and playlist/play modes.

**Concurrency:** `SimpleApp.mu` (sync.Mutex) guards shared state. All UI mutations must go through `app.QueueUpdateDraw()`; background goroutines (playback, thumbnail fetching, search) must never touch UI primitives directly.

**yt-dlp health check:** `health_check.go` checks the installed `yt-dlp` version at startup and warns if it is more than 14 days old.

**Theme system:** Four built-in Catppuccin variants (Latte, Frappé, Macchiato, Mocha), a `Terminal` theme that inherits the host palette (`tcell.ColorDefault` backgrounds/text + ANSI accents), plus custom themes loaded from TOML files. `colorTag()` emits `-` (reset) for `ColorDefault` so the Terminal theme renders natively in dynamic-color markup. Theme and language changes apply in real time via `applyTheme()`/`applyLanguage()`; because `HelpView` bakes theme colors into its text at construction, `applyTheme()` rebuilds it (`setupHelpView()`) so the help screen follows the new palette. See `THEMES.md` for the color palette format.

**i18n:** `internal/ui/i18n.go` provides PT-BR and EN string packs, auto-detected from `LC_ALL`/`LANG` environment variables or the config file.

**Playback config:** `[playback]` section in the TOML config exposes three user-facing options — `default_mode` (`"audio"`/`"video"`), `video_quality` (`"best"`, `"360"`, `"480"`, `"720"`, `"1080"`, `"tct"` for terminal video), and `video_codec` (`""`, `"vp9"`, `"av1"`). These are read at startup and editable in the settings view (Ctrl+C), a `tview.Form` built by `buildConfigForm()` in `setup.go` (dropdowns for language/theme/quality/codec, applied live; an input field for the download dir). `tview.Form` re-applies field colors every draw, so `styleConfigItems()` indicates the focused field with a `▶` label marker rather than per-item coloring. The values are translated into a `--ytdl-format` selector by `buildYtdlFormat()` in `player.go` and passed to mpv at play time; the `tct` path is only taken in video mode so audio mode is respected.

**Download (`download.go`):** Ctrl+D downloads the context track (`getContextTrack()` resolves playing track > focused list item) by shelling out to `yt-dlp`. Progress is streamed live to the status bar: yt-dlp runs with `--newline --progress-template` writing `dlProgressPrefix`-tagged percent lines to a shared pipe, which a scanner goroutine turns into a bar via `renderProgressBar()`. The `[download]` config section has one option, `dir` (editable in the settings form; supports `~/` and falls back to `$XDG_DOWNLOAD_DIR` then `~/Downloads`). The format mirrors playback settings via `buildDownloadFormat()`, except the playback-only `tct` quality falls back to 360p. Audio-only downloads are extracted to **MP3** (`-x --audio-format mp3`) and video downloads default to an **MP4** container (`--merge-output-format mp4`) — both require `ffmpeg`. The `Y` key opens a modal showing the track URL and also writes it to `$XDG_DATA_HOME/youtui-player/last_url.txt` (for copying over tmux/SSH where clipboard yank isn't available).

**UI framework:** `github.com/rivo/tview` + `github.com/gdamore/tcell/v2`. The custom `CustomList` widget (`custom_list.go`) extends tview with thumbnail rendering support.

## Project state (as of adoption)

- **Tests:** none exist in the repo (`find . -name "*_test.go"` returns nothing). `make test` / `go test ./...` will currently report no tests to run — this is a real gap, not a broken suite.
- **Build:** not executed during this adoption pass (toolchain execution was permission-gated in the sandbox). No known build issues from prior commits; `go.mod`/`go.sum` are consistent.
- **CI:** no `.github/workflows` or other CI config found — builds/tests/releases are manual (see `PKGBUILD` for the AUR release path).
- **Docs:** `README.md` and this repo's original `CLAUDE.md` were already thorough and up to date; this adoption pass mainly added `AGENTS.md`/`GEMINI.md` symlinks and an `.env.example`, and confirmed `.gitignore` covers build artifacts.

---
> Source: [IvelOt/youtui-player](https://github.com/IvelOt/youtui-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
