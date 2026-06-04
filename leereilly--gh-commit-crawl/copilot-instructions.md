## gh-commit-crawl

> `gh-commit-crawl` is a `gh` CLI extension written in Go. It is a terminal

# GitHub Copilot instructions

`gh-commit-crawl` is a `gh` CLI extension written in Go. It is a terminal
roguelike themed on Microsoft Build. Pure Go, no external services besides
the public `https://github.com/<handle>.contribs` JSON endpoint.

Read [`AGENTS.md`](../AGENTS.md) for the full briefing. The notes below are
the high-leverage rules Copilot should always honor:

- **Language: Go 1.26+.** No JavaScript, Python, C, or shellouts in the hot
  path. The single intentional shellout is `gh` for the `--pr` flag.
- **Build/test/lint:**
  - `go build -o gh-commit-crawl ./cmd/commit-crawl`
  - `go test ./...`
  - `go vet ./...`
  - `gofmt -l .` (must print nothing)
- **Module path:** `github.com/leereilly/commit-crawl`.
- **Entrypoint:** `cmd/commit-crawl/main.go`.
- **Package boundaries:** `internal/world` knows nothing about tcell;
  `internal/contribgraph` knows nothing about the game; UI packages import
  `game` read-only.
- **Style:** short godoc on exported symbols; paragraph comments above
  non-obvious blocks (explain *why*). Match the existing prose tone.
- **Tests:** table-driven where useful, deterministic seeds, no
  `time.Now()` in fixtures.
- **No new dependencies** without a strong reason. The standard library plus
  `tcell` is the whole toolkit.
- **No secrets, ever.** The `.contribs` endpoint is anonymous.
- **Preserve easter eggs:** Konami code, triple-tap `1`–`5` warp, Clippy on
  floor 2, jester `j`, Copilot blink, rick roll on the final ascent.
- **Determinism:** anything driven by `--seed` must stay reproducible across
  runs and platforms.

---
> Source: [leereilly/gh-commit-crawl](https://github.com/leereilly/gh-commit-crawl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
