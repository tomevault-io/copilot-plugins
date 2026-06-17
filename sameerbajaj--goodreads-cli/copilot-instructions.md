## goodreads-cli

> Use the Goodreads CLI for book metadata and authenticated shelf/rating updates.


# Goodreads CLI

## Install

```bash
go install github.com/sameerbajaj/goodreads-cli/cmd/goodreads@latest
```

Verify:

```bash
goodreads --version
goodreads doctor --json
```

## Common Commands

```bash
goodreads search "left hand of darkness" --agent
goodreads enrich "Dune" --agent --select metadata.title,metadata.rating,metadata.ratings_count,book_id
goodreads shelf auth-check --agent
goodreads shelf set "Dune" --shelf currently-reading --agent
goodreads shelf set --book-id 44767458-dune --shelf read --rating 5 --agent
```

## Auth

Read-only commands need no auth.

Shelf/rating writes can use:

- local Brave/Chrome cookies on macOS, automatically
- `GOODREADS_COOKIE`
- `GOODREADS_JWT`

Use `GOODREADS_BROWSER=brave` or `GOODREADS_BROWSER=chrome` to force a browser. Use `GOODREADS_NO_BROWSER_AUTH=1` to disable browser-cookie lookup.

## Agent Rules

Use `--agent` for LLM calls. It enables JSON, compact output, no color, non-interactive mode, and `--yes`.

Use `--dry-run` before a risky write. Mutating commands are under `shelf`.

---
> Source: [sameerbajaj/goodreads-cli](https://github.com/sameerbajaj/goodreads-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
