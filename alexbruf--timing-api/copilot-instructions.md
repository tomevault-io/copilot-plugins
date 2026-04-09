## timing-api

> Bun-based CLI wrapping the Timing app API (https://web.timingapp.com/api/v1).

# Timing CLI

Bun-based CLI wrapping the Timing app API (https://web.timingapp.com/api/v1).

## Build & Run
- `bun install` — install deps
- `bun run dev -- <command>` — run in dev mode
- `bun run build` — compile to standalone `./timing` binary

## Architecture
- `src/client.ts` — TimingClient class, all HTTP calls
- `src/commands/` — commander subcommands
- `src/config.ts` — env var reading (TIMING_API_TOKEN)
- `src/output.ts` + `src/format.ts` — JSON/table output
- `skill/SKILL.md` — Claude Code skill definition

## Auth
Set `TIMING_API_TOKEN` env var (generate at Timing web app → API Keys).

## Key conventions
- JSON output by default, `--format table` for human-readable
- Array query params use `[]` suffix (e.g. `projects[]`)
- `/time-entries/latest` and `/running` return 302 redirects (Bun follows automatically)
- Activity hierarchy returns text/plain, not JSON

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexbruf)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alexbruf)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
