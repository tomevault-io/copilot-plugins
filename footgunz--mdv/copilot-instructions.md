## mdv

> CLI Markdown viewer: opens a native webview window, renders GFM + Mermaid,

# mdv — agent notes

CLI Markdown viewer: opens a native webview window, renders GFM + Mermaid,
live-reloads on save. See README.md for user-facing behavior and config keys.

## Build & test

    go build -tags desktop,production -o mdv ./cmd/mdv    # or: task build
    task app                     # macOS .app bundle (Finder "Open With")
    task install-app             # -> /Applications + symlink on PATH
    go test ./...                # headless, no window needed
    ./mdv README.md              # manual check; Ctrl-C or close window to quit

cgo is required for `cmd/mdv` (Wails v2). Linux needs `libgtk-3-dev
libwebkit2gtk-4.1-dev` and the extra `webkit2_41` build tag. The library
packages (`internal/...`, `pkg/mermaid`) build and test without cgo or a
display.

## Layout

- `cmd/mdv` — flag parsing, Wails app wiring, dock icon.
- `internal/config` — `~/.config/mdv/config` parser. New key = field on
  `Config` + case in `parseConfig` + README + `examples/config`.
- `internal/render` — goldmark + chroma + mermaid → HTML. `Renderer.Body`
  for fragments, `StaticPage` for `-html` export. `assets/` (base.css,
  mermaid.min.js) is embedded; served at `/_assets/`.
- `internal/server` — file serving (mounted as the Wails asset server
  handler; "/" serves the entry file) and the fsnotify watcher.
- `pkg/mermaid` — native mermaid→SVG engine (parse → IR → dagre layout in
  goja → SVG). Unsupported syntax returns `ErrUnsupported` and the caller
  falls back to bundled mermaid.js; never partially render.

## Gotchas

- **The Wails asset server buffers whole responses** (it serves over a
  custom scheme, not TCP) — streaming responses like SSE/websockets can't
  pass through it. Live reload is a Wails event (`mdv:reload` emitted in
  main.go, subscribed in the page template in render.go).
- **Wails v2 needs build tags**: `desktop,production` always (plain
  `go build` produces a dev-mode binary that expects a frontend dev
  server), plus `webkit2_41` on Linux. golangci-lint gets them from
  `.golangci.yml`.
- **`-framework UniformTypeIdentifiers`** in icon_darwin.go's LDFLAGS is
  load-bearing: wails v2.12 references UTType but doesn't link the
  framework itself.
- **"Open With" is an Apple Event, not argv.** Finder never puts the
  document in `os.Args`; it sends `kAEOpenDocuments`, surfaced by Wails as
  `mac.Options.OnFileOpen`. So `main.go` accepts zero args when `bundled()`
  (exe path inside `.app/Contents/MacOS/`) and the server starts with an
  empty entry, serving a placeholder until one arrives. Finder reuses a
  running instance for a second document, hence `server.SetEntry` — which
  moves `baseDir`, i.e. the traversal boundary, not just the entry file.
  Wails buffers open events (100 deep), so one landing before the window
  exists is picked up by the initial load of `/`; `OnDomReady` covers the
  narrow case of one landing just after the placeholder rendered.
- **One binary, two entry points:** `task install-app` symlinks
  `$GOPATH/bin/mdv` at the executable inside `/Applications/mdv.app`.
  `os.Executable()` on darwin reports the path *as invoked*, so through the
  symlink `bundled()` is false and CLI semantics hold, while Finder's launch
  of the real path gets bundle semantics — the same binary, behaving
  correctly both ways. The CLI instance has a NULL bundle id, so
  LaunchServices routes "Open With" documents to a separate bundle instance
  rather than hijacking a terminal-launched window. Install to
  `/Applications`, never symlink into the repo: `task clean` removes
  `./mdv.app`.
- **Icons:** `task icons` rasterizes `cmd/mdv/icon.svg` into the embedded
  dock icon (`icon.png`) and the bundle's `build/darwin/mdv.icns`. Needs
  `rsvg-convert` (`brew install librsvg`) — it renders real transparency,
  unlike qlmanage, which is why `icon_darwin.go` no longer clips corners.
  `setDockIcon` still matters for the bare binary; inside `mdv.app` the
  `.icns` wins.
- Binaries at repo root (`mdv`) are gitignored build artifacts.

---
> Source: [footgunz/mdv](https://github.com/footgunz/mdv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
