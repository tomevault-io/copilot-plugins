## harness

> - **DESIGN.md** — overview with links to docs/

# Masters Of AI Harness

## Related Docs

- **DESIGN.md** — overview with links to docs/
- **docs/** — architecture, agents, tools, configuration, sandbox, design decisions
- **CHANGELOG.md** — version history

## Quick Orientation

Standalone agent runtime built on top of the Claude Agent SDK. Two interfaces, one runtime:

- **Terminal TUI** — `mastersof-ai [--agent x]` — React/Ink, single user, local
- **Web UI** — `mastersof-ai --serve` — Fastify backend + React SPA frontend, multi-user, token auth

Both share agent loading, tools, sub-agents, sessions, and memory.

- TypeScript, runs via tsx (no build step for backend)
- Entry: `bin/mastersof-ai.js` → `src/index.tsx`
- Agent loading: `src/agent-context.ts` + `src/manifest.ts` (frontmatter) → `src/agent.ts`
- Tools: `src/tools/` — in-process MCP servers, one per domain
- Serve mode: `src/serve.ts` — Fastify HTTP/WS server (REST API + WebSocket streaming)
- Web frontend: `web/` — React + Vite + Tailwind SPA (deploys to Cloudflare Pages)
- A2A: `src/a2a/` — agent card generation, A2A protocol server/client
- TUI: `src/components/` — React/Ink (DO NOT TOUCH unless broken)
- Config: `~/.mastersof-ai/config.yaml`
- Auth (serve mode): `~/.mastersof-ai/access.yaml`
- Security: `src/env-safety.ts`, `src/url-safety.ts`, `src/content-safety.ts`
- Credentials: `src/credentials.ts` + `src/egress-proxy.ts`
- Process isolation: `src/ipc-protocol.ts`, `src/session-worker.ts`, `src/worker-manager.ts`
- WS protocol: `src/ws-protocol.ts`, `src/sdk-stream.ts`, `src/query-mutex.ts`
- Health/observability: `src/health.ts`, `src/rate-limit.ts`, `src/cost.ts`
- Privacy: `src/privacy.ts` — LGPD compliance
- CLI subcommands: `src/cli/` — modular command handlers

## Running Locally

```bash
npx tsx bin/mastersof-ai.js                         # TUI with default agent
npx tsx bin/mastersof-ai.js --agent researcher      # TUI with specific agent
npx tsx bin/mastersof-ai.js --serve                 # Web UI server on port 3200
npx tsx bin/mastersof-ai.js --serve --port 5000     # Web UI on custom port
npx tsx bin/mastersof-ai.js --agent researcher --sandbox  # Bubblewrap sandbox
npx tsx bin/mastersof-ai.js --card                  # Output Agent Card JSON
npx tsx bin/mastersof-ai.js --list-agents

# CLI subcommands
npx tsx bin/mastersof-ai.js run billing "Run monthly billing"           # Headless run
npx tsx bin/mastersof-ai.js credentials check --agent billing           # Check agent credentials
npx tsx bin/mastersof-ai.js credentials migrate billing                 # Generate migration YAML
npx tsx bin/mastersof-ai.js access create --name partner --agents a,b   # Create access token
npx tsx bin/mastersof-ai.js access rotate --name partner                # Rotate token
npx tsx bin/mastersof-ai.js status billing                              # Recent run results
npx tsx bin/mastersof-ai.js preflight --agent billing                   # Validate full config
npx tsx bin/mastersof-ai.js create my-agent                             # Scaffold new agent
```

---
> Source: [mastersof-ai/harness](https://github.com/mastersof-ai/harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
