## clash-traffic-monitor

> These instructions apply to the entire repository. If a deeper directory later gets its own `AGENTS.md` or `AGENTS.override.md`, that file should narrow or override these rules for work in that subtree.

# AGENTS.md

## Repository Scope

These instructions apply to the entire repository. If a deeper directory later gets its own `AGENTS.md` or `AGENTS.override.md`, that file should narrow or override these rules for work in that subtree.

## Project Overview

- `traffic-monitor` is a single-binary Go service that polls Mihomo `/connections`, aggregates traffic into SQLite, and serves an embedded web UI.
- Backend and frontend live in the same repo and ship together.
- There is no frontend build step in this repository. The UI is plain static assets under `web/` embedded by Go.

## Important Files

- `main.go`: application entrypoint, HTTP routes, SQLite logic, Mihomo integration, and embedded asset serving.
- `main_test.go`: primary automated test coverage for backend logic and HTML/script assertions.
- `web/index.html`: embedded UI structure.
- `web/styles.css`: embedded UI styling.
- `web/app.js`: embedded UI behavior and API calls.
- `README.md`: user-facing behavior and operational expectations. Update when behavior materially changes.

## Working Agreements

- Keep changes surgical. Do not refactor unrelated areas while fixing one issue.
- Preserve the current architecture: single Go binary, embedded static frontend, SQLite persistence.
- Prefer extending existing patterns in `main.go` and the current frontend files over introducing new layers or frameworks.
- Do not add Node-based tooling, frontend bundlers, or new production dependencies unless explicitly requested.
- Avoid changing persisted behavior, API payloads, or database defaults unless the task requires it. When you do, update tests.

## UI and Frontend Rules

- Treat `web/index.html`, `web/styles.css`, and `web/app.js` as a tightly coupled unit.
- Favor small HTML changes and CSS-first fixes before adding new JavaScript complexity.
- Keep the UI dense and operational rather than marketing-styled. This project is a monitoring console.
- Preserve mobile behavior when adjusting desktop layouts.

## Backend Rules

- Keep Mihomo-facing behavior explicit and observable. Prefer clear logging around auto-switch or restore behavior over opaque control flow.
- Keep SQLite schema and query changes backward-compatible when possible.
- Reuse existing helpers and response patterns instead of adding parallel abstractions.

## Validation

- Run `go test ./...` after code changes.
- If you change HTTP handlers, persistence rules, or auto-switch behavior, add or update tests in `main_test.go`.
- If you change UI copy or layout, sanity-check the affected HTML/CSS/JS together and avoid leaving mismatched labels or dead selectors.

## Commit Scope

- Do not commit unrelated untracked files such as local notes, temporary docs, or `.claude/` contents unless the user explicitly asks for them.
- Keep commits grouped by user-visible outcome, not by file type.

## Release Versioning

- Keep the release tag and the page footer version in sync.
- When bumping a release, update the footer version string in `web/index.html`.
- Use the same version string for the git tag, prefixed with `v`.

## Useful Commands

```bash
go test ./...
go build -o traffic-monitor main.go
```

---
> Source: [zhf883680/clash-traffic-monitor](https://github.com/zhf883680/clash-traffic-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
