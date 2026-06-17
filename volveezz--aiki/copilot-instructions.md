## aiki

> Persistent vector database that gives AI agents searchable memory across sessions. Indexes project source code and stores manual knowledge entries (decisions, patterns, bugs). Uses Mistral embeddings + Zvec HNSW for semantic search.

# Aiki — Semantic Memory for AI Agents

## What It Is

Persistent vector database that gives AI agents searchable memory across sessions. Indexes project source code and stores manual knowledge entries (decisions, patterns, bugs). Uses Mistral embeddings + Zvec HNSW for semantic search.

## Architecture

```
Claude Code ──HTTP POST──▸ Aiki daemon (WSL2, port 4242)
                            ├─ MCP Streamable HTTP transport
                            ├─ Per-session McpServer instances
                            ├─ Zvec collections (file-locked)
                            └─ Mistral API (embeddings)
```

Single persistent daemon, multiple concurrent MCP sessions. Zvec uses exclusive file locks per collection — this is why it runs as a daemon instead of stdio (only one process can hold the write lock).

### Session lifecycle
- Each `POST /mcp` with `isInitializeRequest` creates a new `StreamableHTTPServerTransport` + `McpServer` pair
- `resolveRoot()` lazily calls `listRoots()` from within tool handlers (not during init — Claude Code bug #3315 causes timeout during init)
- Root is cached per session, `project_path ?? await resolveRoot()` in every tool handler

### Networking
- Server binds `0.0.0.0:4242` — WSL2 mirrored networking routes Windows→WSL traffic through `loopback0`, not `lo`, so `127.0.0.1` binding is unreachable from Windows
- MSYS2 curl can't reach WSL2 localhost — always use `wsl -d Debian -- bash -c 'curl ...'` from Claude Code's shell
- `.wslconfig` has `networkingMode=mirrored` and `firewall=false` for Windows↔WSL2 loopback

### WSL keepalive
- Claude Code's statusline daemon (`~/.claude/statusline-daemon.exe`) spawns `wsl -d Debian -- sleep infinity` on startup to prevent WSL idle termination
- PID file at `~/.claude/wsl-keepalive.pid` prevents duplicate spawns on daemon restarts
- `.wslconfig` has `vmIdleTimeout=4294967295` to keep the WSL2 VM alive

## File Layout

```
src/
  server.ts           HTTP server, MCP session management, admin endpoints
  server-factory.ts   Creates McpServer instances with tool registrations
  tools/
    search.ts         Semantic search (project, global, both scopes)
    remember.ts       Store knowledge entries with embeddings
    index-files.ts    Index source files with mtime-based change detection
  store/
    collection.ts     Zvec collection lifecycle, schemas, cache, admin ops
  indexer/
    chunker.ts        File discovery, gitignore parsing, code chunking
  embedding/
    mistral.ts        Mistral codestral-embed-2505 API with retry
  utils/
    config.ts         Environment config (MISTRAL_API_KEY, AIKI_PORT)
    paths.ts          Windows↔WSL path conversion, hashing
.claude/
  commands/
    aiki-forget.md    Slash command for collection cleanup via admin endpoints
```

## Two Collection Types

**Project** (`~/aiki/collections/<hash>/`) — per-project, keyed by path hash
- Fields: `file_path`, `chunk_type`, `language`, `content`, `summary`, `tags`, `last_modified`
- Populated by `index` tool (source files) and `remember` tool (manual entries)

**Global** (`~/aiki/global/`) — cross-project knowledge
- Fields: `category`, `topic`, `content`, `confidence`, `source_project`
- Requires `category` (style|pattern|preference|solution) and `topic`

## MCP Tools (agent-facing)

| Tool | Purpose |
|------|---------|
| `search` | Semantic search across project/global/both scopes |
| `remember` | Store knowledge with embeddings for future retrieval |
| `index` | Index source files from a project directory |

All tools accept optional `project_path` — defaults to client workspace root via `resolveRoot()`.

## Admin Endpoints (not MCP, not agent-visible)

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/admin/delete` | POST | Delete by IDs or filter expression |
| `/admin/purge` | POST | Destroy entire collection |
| `/admin/stats` | GET | Doc count and index completeness |
| `/health` | GET | Uptime and session count |

Use `/aiki-forget` slash command to interact with these.

## Daemon Management

```bash
# Systemd service on WSL2 Debian
wsl -d Debian -- sudo systemctl restart aiki
wsl -d Debian -- sudo systemctl status aiki
wsl -d Debian -- sudo journalctl -u aiki -f
```

Service file: `/etc/systemd/system/aiki.service`
Env file: `/home/aiki/.config/aiki/env` (MISTRAL_API_KEY, AIKI_PORT)

## Indexer Behavior

- Supported extensions: see `EXT_TO_LANG` in `chunker.ts` (.ts, .py, .go, .rs, .java, .sql, .md, etc.)
- Max file size: 100KB. Lock files, binaries, large JSON/YAML silently skipped.
- Custom `patterns` use `endsWith` matching (supports compound extensions like `.d.ts`, `.test.ts`)
- Gitignore: parsed with anchoring (`/build` anchored to root), negation (`!`), character classes (`[Dd]ebug`)
- Chunking: 150 lines per chunk, 30 line overlap. Files ≤150 lines stored as single chunk.
- Dedup: content-hash-based IDs, mtime cache for skipping unchanged files

## Build

```bash
pnpm install
pnpm build    # tsc → dist/
```

---
> Source: [volveezz/aiki](https://github.com/volveezz/aiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
