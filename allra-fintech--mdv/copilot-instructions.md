## mdv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build
make build                          # produces ./mdv
go build -o mdv ./cmd/mdv           # equivalent

# Test
make test-unit                      # go test ./...
make test-integration               # go test -tags integration ./...
make test                           # both

# Run a single test
go test ./internal/mdv/ -run TestLiveReloadOnFileChange
go test -tags integration ./internal/mdv/ -run TestLiveReloadOnFileChange

# Format / lint
make format                         # go fmt ./...
make lint                           # go vet ./...

# Tidy deps
go mod tidy
```

## Architecture

```
cmd/mdv/main.go       — flag parsing, resolvePort, openBrowser, server wiring
internal/mdv/
  hub.go              — SSE broadcast hub
  watcher.go          — fsnotify file watcher
  server.go           — HTTP routes and handlers
  renderer.go         — goldmark Markdown→HTML pipeline
  template.go         — full page HTML template (CSS + JS inline)
  pdf.go              — headless Chrome PDF export
```

Live reload data flow:

```
fsnotify event → WatchFile() → Hub.Broadcast()
                                     ↓
                        SSE clients (/events) receive "event: reload"
                                     ↓
                   Browser fetches /content → swaps #content innerHTML
```

**Key design points worth knowing before changing things:**

- **Watcher watches both the file and its parent directory.** Atomic-write editors (Vim, JetBrains) replace the inode on save, so watching the file alone misses events. Only `Write`/`Create` events whose path matches the target file trigger a broadcast.

- **`/content` returns an HTML fragment; `GET /<file>.md` returns a full page.** The SSE JS calls `/content?path=` for partial refresh (preserving scroll), while direct navigation renders through `pageTemplate`.

- **`newMarkdown` is stateless and created per request.** goldmark isn't safe for concurrent use so a fresh instance is built each time.

- **Integration tests are behind the `integration` build tag.** Add `//go:build integration` at the top of any new integration test file. Unit tests (`go test ./...`) skip them; CI runs both suites separately.

- **Mermaid rendering is client-side only.** The server emits `<pre class="mermaid">` blocks; the browser lazily loads `mermaid.min.js` from CDN on first encounter.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Allra-Fintech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Allra-Fintech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
