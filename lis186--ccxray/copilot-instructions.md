## ccxray

> Guidance for Claude Code when working with this repository.

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## What is ccxray

A transparent HTTP proxy that sits between Claude Code and the Anthropic API. It records every request/response, serves a real-time Miller-column dashboard at the same port, and supports request interception/editing. Zero config, zero dependencies beyond Node.js.

## Commands

```bash
npx ccxray claude                                # One command: proxy + Claude Code
ccxray claude                                    # Multiple terminals auto-share one hub
ccxray --port 8080 claude                        # Custom port (opts out of hub, independent server)
ccxray status                                    # Show hub info and connected clients
ccxray                                           # Proxy + dashboard only
npm run dev                                      # Dev mode (auto-restart on server/public changes)
npm test                                         # Run tests
```

No build step. No linting. Restart to apply changes.

## Architecture

### Server (`server/`)

| Module | Purpose |
|--------|---------|
| `server/index.js` | Entry point: HTTP server, request routing, startup |
| `server/config.js` | PORT, ANTHROPIC_HOST/PORT/PROTOCOL, LOGS_DIR, MAX_ENTRIES, model context windows |
| `server/pricing.js` | LiteLLM price fetch, 24h cache, fallback rates, cost calculation |
| `server/cost-budget.js` | Cost data orchestration: cache, warm-up, grouping |
| `server/cost-worker.js` | Child process: scans `~/.claude/` JSONL files without blocking event loop |
| `server/store.js` | In-memory state: entries[] (capped at MAX_ENTRIES), sseClients[], sessions, intercept, versionIndex (keyed by `agentKey::coreHash`). Session detection with subagent inference (inflight + temporal heuristic) |
| `server/sse-broadcast.js` | SSE broadcast to dashboard clients, entry summarization |
| `server/helpers.js` | Tokenization, context breakdown, SSE parsing, formatting |
| `server/system-prompt.js` | KNOWN_AGENTS registry, agent type detection, B2 block splitting, unified diff |
| `server/restore.js` | Startup log restoration, lazy-load req/res from disk |
| `server/forward.js` | HTTP/HTTPS proxy to Anthropic, SSE capture, response logging, proxyRes error handling |
| `server/routes/api.js` | REST endpoints for entries, tokens, system prompt |
| `server/routes/sse.js` | SSE endpoint |
| `server/routes/intercept.js` | Intercept toggle/approve/reject/timeout |
| `server/routes/costs.js` | Cost budget endpoints |
| `server/hub.js` | Multi-project hub: lockfile (`~/.ccxray/hub.json`), discovery (with orphan port probe fallback), client registration, idle shutdown (injectable via setOnShutdown), crash auto-recovery |
| `server/auth.js` | API key auth middleware (enabled via `AUTH_TOKEN` env) |
| `server/storage/` | Storage adapters (local filesystem, S3/R2). `statShared()` for file mtime |

### Client (`public/`)

| File | Purpose |
|------|---------|
| `public/index.html` | Dashboard shell |
| `public/style.css` | Dark theme, Miller column layout |
| `public/app.js` | App initialization |
| `public/miller-columns.js` | Projects → Sessions → Turns → Sections → Timeline → Detail |
| `public/entry-rendering.js` | Turn rendering, session/project tracking |
| `public/messages.js` | Merged steps: thinking + tool groups, timeline detail, minimap rendering + layout |
| `public/cost-budget-ui.js` | Cost analysis page, heatmap, burn rate |
| `public/intercept-ui.js` | Pause/edit/approve/reject requests |
| `public/system-prompt-ui.js` | Multi-agent browsing (3-column Miller), version history, unified diffs |
| `public/keyboard-nav.js` | Arrow keys, Enter, Escape |
| `public/quota-ticker.js` | Topbar quota ticker |

### Hub Mode (multi-project)

```
ccxray claude (1st)  → fork detached hub → connect as client → spawn claude
ccxray claude (2nd)  → discover hub via ~/.ccxray/hub.json → connect as client → spawn claude
                              ↓
                     Hub (detached process)
                       ├── HTTP proxy on :5577
                       ├── Dashboard (same port)
                       ├── Client registry (register/unregister/health)
                       └── Idle shutdown (5s after last client exits)
```

- Hub lockfile: `~/.ccxray/hub.json` (written after `listen()` succeeds = readiness signal)
- Hub log: `~/.ccxray/hub.log` (stdout/stderr of detached process)
- `--port` opts out of hub mode entirely (independent server)
- Crash recovery: clients monitor hub pid every 5s, auto-fork new hub using port as mutex
- Version check: semver major mismatch → reject, minor → warn, patch → silent

### Data Flow

```
Claude Code → proxy receives request → detect session (explicit or inferred)
  → [intercept check] → log {id}_req.json → forward to Anthropic
  → capture SSE response → log {id}_res.json → calculate cost
  → broadcast via SSE (includes sessionInferred flag) → dashboard updates
```

Logs stored in `~/.ccxray/logs/` (not package-relative). Respects `CCXRAY_HOME` env var.

---
> Source: [lis186/ccxray](https://github.com/lis186/ccxray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
