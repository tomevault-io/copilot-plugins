## tuitube

> Guidance for AI agents working in projects based on this template.

# AGENTS.md

Guidance for AI agents working in projects based on this template.

## Project Intent

This is a personal Go TUI template built on the Charm stack (Bubble Tea v2, Lip Gloss v2, Bubbles). Treat the existing structure as the application shell — build product features on top of it, don't bypass the patterns it establishes.

## Checklist for Adding a New Feature

Follow these steps in order. Run the verification commands before finishing.

1. **Add a screen** in `internal/screens/` implementing the `screens.Screen` interface (`Init`, `Update`, `View`, `Title`, `KeyBindings`).
2. **Register the screen** in `internal/app/app.go` → `registerScreens`. Add its ID to the `preferred` slice in `refreshScreenOrder` if it belongs in the primary sidebar.
3. **Register a command** in `registerCommands` (same file) so the screen is reachable via `ctrl+k`. Command `Run` functions must return a `tea.Cmd` that emits a message defined in `internal/app/messages.go`.
4. **Log key events** via `logs.Info` / `logs.Error` so behavior is visible in the Logs screen (see Logging section below).
5. **Verify:** `go test ./...` and `go build ./cmd/tuitube` must pass before committing.

## Key Packages

| Package | Role |
|---|---|
| `internal/app` | Root model — routing, overlays, keymap, theme, message dispatch |
| `internal/screens` | Feature screens implementing `screens.Screen` |
| `internal/commands` | Command palette entries and registry |
| `internal/app/messages.go` | All app-level message types — add new ones here |
| `internal/debug` | In-memory log surfaced in the Logs screen |
| `internal/theme` | Semantic styles and theme definitions |
| `internal/layout` | Terminal size and region calculations |

## Logging

Always log via `internal/debug.Log` — never `fmt.Println` or `panic`. Use the prefix convention `package.Operation` so log entries are easy to grep.

```go
// info
logs.Info("myscreen.Load: loaded 42 items")

// error
logs.Error("myscreen.Load", err)
// produces: "ERROR myscreen.Load: <err message>"
```

Available methods on `*debug.Log`:
- `Info(message string)` — general events
- `Warn(message string)` — unexpected but recoverable
- `Error(op string, err error)` — failures with an error value

## Messages

All app-level message types live in `internal/app/messages.go`. If you need new cross-screen behavior, define a message there and handle it in `Model.Update` (`internal/app/update.go`).

```go
// example
type myFeatureMsg struct{ Data string }
```

Existing messages and their contracts:
- `routeMsg{ScreenID}` — navigate to a registered screen ID. Unknown IDs are logged and ignored.
- `toggleSidebarMsg{}` — show/hide the sidebar.
- `quitMsg{}` — exit the application.
- `commandsExecutedMsg{Title, Cmd}` — carries a command palette action result into the update loop.

## Screen Layout Rules

Screens receive inner content dimensions as `width, height int` in `View()`. Always pad to fill the full area — short content shifts the footer up:

```go
func (s MyScreen) View(width, height int) string {
    content := buildContent()
    return lipgloss.NewStyle().Width(width).Height(height).Render(content)
}
```

## Sidebar Ordering

Screen IDs must be unique. `refreshScreenOrder` in `internal/app/app.go` controls sidebar position — add new primary screen IDs to the `preferred` slice to pin their order. Screens not in `preferred` are appended alphabetically.

## Cross-Screen Navigation

Emit `routeMsg` to navigate:

```go
return m, func() tea.Msg { return routeMsg{"my-screen-id"} }
```

## Lip Gloss v2 Sizing

`Width(n)` and `Height(n)` set the **total outer size** including borders. Do not pre-subtract `GetFrameSize()`:

```go
// correct
style.Width(totalWidth).Height(totalHeight).Render(content)

// wrong — double-subtracts the frame
frameW, frameH := style.GetFrameSize()
style.Width(totalWidth-frameW).Height(totalHeight-frameH).Render(content)
```

Pass inner content dimensions to `View()` using `GetFrameSize()` — call the container style with the full allocated dimensions.

## Bubbles Compatibility

Some Bubbles components still return Bubble Tea v1 `tea.Cmd` types and cannot be used directly with v2 models. Check for type mismatches before wiring a new component into the model.

## UI Invariants

Preserve these behaviors unless the product explicitly requires otherwise:

- `ctrl+c` quits.
- `q` quits when no overlay is open.
- `ctrl+k` opens the command palette.
- `?` opens the help overlay.
- `esc` closes overlays.
- `tab` cycles focus between sidebar and main.
- Sidebar supports arrow keys and vim keys.
- Layout handles small terminal sizes without panics or negative dimensions.

## Testing

Prefer tests for non-visual logic. Do not snapshot-test terminal UI output.

```sh
go test ./...
go build ./cmd/tuitube
```

## Release

GoReleaser config: `.goreleaser.yaml`
Release workflow: `.github/workflows/release.yml` (tag push)
Publish workflow: `.github/workflows/publish.yml` (manual, bump selection)

```sh
goreleaser release --snapshot --clean   # local snapshot check
```

## Coding Style

- `gofmt` on all Go files.
- Small, direct changes over large rewrites.
- Comments only where the why is non-obvious.
- No premature abstractions.
- No optional dependencies unless actively used.

## Persona DB Patterns

tuitube supports multiple independent libraries via `--db`. Use this to create vibe-specific personas:

```bash
# work/focus persona
tuitube --db ~/.local/share/tuitube/focus.db bootstrap
tuitube --db ~/.local/share/tuitube/focus.db sync

# high-energy / gym persona
tuitube --db ~/.local/share/tuitube/gym.db bootstrap
tuitube --db ~/.local/share/tuitube/gym.db add-station \
  --url "https://www.youtube.com/@SomeEnergeticChannel" --name "Hype"

# launch with a specific persona
tuitube --db ~/.local/share/tuitube/focus.db
```

Each persona DB has its own playlists, favorites, and downloads. The MCP server can target any persona:

```json
{ "args": ["--db", "~/.local/share/tuitube/focus.db", "mcp"] }
```

## MCP Tools Reference

The MCP server (`tuitube mcp`) exposes 11 tools. Key ones for agents:

- `search_tracks` — always call this first to get IDs before playing/favoriting
- `add_to_playlist` — accepts an array of `track_ids`, batch in one call
- `sync_station` — returns per-station inserted counts and sync log

Before doing CLI work, run `tuitube doctor` to verify dependencies are healthy.

## Release Workflow (cutting a new version)

When you've synced new channels or curated more playlists:

```bash
# 1. sync any channels with new uploads
tuitube sync

# 2. export the catalog from your working DB
SRC=~/.local/share/tuitube/tuitube.db ./scripts/export_catalog.sh

# 3. commit catalog.db and tag
git add catalog.db && git commit -m "catalog: update to $(date +%Y%m%d)"
git tag v0.x.0 && git push && git push origin v0.x.0
```

GoReleaser builds the binaries and attaches `catalog.db` automatically on tag push.

## Updating Your Own Install

After pulling a new version or syncing channels:

```bash
# if you moved your DB to the default location already
tuitube sync

# if you're still pointing at the repo DB
tuitube sync
```

The default DB location is `~/.local/share/tuitube/tuitube.db`. Copy it there once and `tuitube` needs no flags:

```bash
mkdir -p ~/.local/share/tuitube
cp ~/REPOS/tuitube/tuitube.db ~/.local/share/tuitube/tuitube.db
```

---
> Source: [gitcoder89431/tuitube](https://github.com/gitcoder89431/tuitube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
