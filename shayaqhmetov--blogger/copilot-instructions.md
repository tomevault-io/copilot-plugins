## blogger

> This project uses the **GSD (Get Shit Done)** workflow. Planning lives under `.planning/`. Treat those documents as load-bearing context — read them before answering questions about scope, decisions, or what to build next.

# Blogger Analytics MCP — Project Guide for Claude

This project uses the **GSD (Get Shit Done)** workflow. Planning lives under `.planning/`. Treat those documents as load-bearing context — read them before answering questions about scope, decisions, or what to build next.

## What this is

A single-user, local **Python MCP server** that connects to the user's own YouTube, TikTok, and Instagram accounts via official APIs, syncs analytics into a local SQLite store, and exposes that data through MCP tools so Claude can answer questions about videos/posts, views, audience, and trends. Read-only, manual sync, JSON export, rolling 90-day window, stdio transport.

## Canonical planning docs (read first)

| Doc | What it has |
|-----|-------------|
| `.planning/PROJECT.md` | Core value, scope, key decisions, constraints |
| `.planning/REQUIREMENTS.md` | 35 v1 requirements with REQ-IDs (AUTH-/SYNC-/STORE-/MCP-/DIST-/CONV-) + traceability to phases |
| `.planning/ROADMAP.md` | 6 phases (Foundation → FakeConnector keystone → YT → IG → TT → Cross-platform tools) |
| `.planning/STATE.md` | Current execution state, blockers |
| `.planning/research/SUMMARY.md` | Synthesis of stack/features/architecture/pitfalls — read for any "how should we do X" question |
| `.planning/research/PITFALLS.md` | 26 platform-specific pitfalls; check before implementing connectors |

## Locked decisions (do not re-litigate without explicit approval)

- **Python**, not TypeScript
- **Local stdio MCP** only (no HTTP, no hosting)
- **Official APIs** only (no scraping)
- **Single-user, own-accounts** only (no multi-user, no competitor analytics)
- **SQLite** with manual sync (no cron, no scheduler)
- **Rolling 90-day** window (no >90d backfill)
- **JSON export** only (no CSV/MD/XLSX)
- **Read-only** (no posting)
- **Pre-baked reports are anti-features** — Claude composes reports from primitive tools

See `PROJECT.md` "Out of Scope" for the full list with reasoning.

## Stack (locked — see `.planning/research/STACK.md` for rationale)

- `mcp` 1.27.1 with bundled FastMCP, **stdio transport**
- `google-api-python-client` 2.196.0 + `google-auth-oauthlib` 1.4.0 for YouTube
- Hand-rolled `httpx` 0.28.1 + `authlib` 1.7.2 PKCE for TikTok and Instagram (no usable first-party SDK)
- SQLAlchemy 2.0.49 + Alembic 1.18.4 (`render_as_batch=True`)
- `keyring` 25.7.0 (Fernet fallback) for token storage
- `tenacity` 9.1.4 for HTTP retries
- `Typer` 0.25.1 for CLI
- `uv` 0.11.12 packaging; two console scripts: `blogger` (CLI) and `blogger-mcp` (stdio MCP server)
- `structlog` to **stderr only** (stdout is the JSON-RPC channel)

## Non-negotiable conventions (Day-1)

1. **No `print()` in non-test code.** Stdout is the MCP JSON-RPC channel; one stray write corrupts the stream and silently kills the client. Lint rule enforces this.
2. **No plaintext tokens on disk.** Every token write goes through `TokenVault`. CI grep blocks tokens-shaped strings in any path the project writes to.
3. **All HTTP through the shared `httpx` + `tenacity` client.** No ad-hoc `requests` calls.
4. **Tool descriptions document per-platform behavior** including `null` vs `0` vs `unavailable` distinctions — load-bearing for Claude's accuracy.
5. **MCP read tools never call platform APIs.** They query SQLite only. The only "live" path is `sync_start` (and the CLI `blogger sync`).

## Workflow

This project uses GSD. Common commands:

- `/gsd-progress` — current state, what's next
- `/gsd-plan-phase <N>` — create plan for phase N
- `/gsd-execute-phase <N>` — execute phase N
- `/gsd-discuss-phase <N>` — gather context before planning a phase

**Mode:** YOLO (auto-approve gates). **Granularity:** coarse. **Models:** balanced (Sonnet).

## Day-1 calendar gate

Meta and TikTok dev-app submissions take 1–4 weeks each and gate Phases 4 and 5. They must be submitted on Day 1, in parallel with Phase 1. Phases 1, 2, 3 are calendar-independent thanks to the FakeConnector pattern + YouTube's instant Google approval.

---
> Source: [shayaqhmetov/blogger](https://github.com/shayaqhmetov/blogger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
