## hanzi-browse

> This file is for AI agents (Claude Code, Cursor, Codex, etc.) working on this codebase. It describes what this project is, how it works, and where things live.

## Hanzi Browse — Architecture Guide

This file is for AI agents (Claude Code, Cursor, Codex, etc.) working on this codebase. It describes what this project is, how it works, and where things live.

### What is Hanzi Browse?

A browser automation platform that gives AI agents access to a real Chrome browser with the user's signed-in sessions. The AI agent sends a task, the browser executes it autonomously.

Two distribution paths:
- **Skills** — for users who run Hanzi Browse locally via their AI agent (Claude Code, Cursor, etc.)
- **Free tools** — public web apps that demonstrate use cases (e.g. tools.hanzilla.co/x-marketing)

Both paths require the same infrastructure: Chrome extension + site patterns + LLM.

### Architecture

```
┌─────────────────────────────────────────────────┐
│  AI Agent (Claude Code / Cursor / Codex / etc.) │
└──────────────────────┬──────────────────────────┘
                       │ MCP protocol (stdio)
              ┌────────▼────────┐
              │   MCP Server    │  server/src/index.ts
              │  (Node.js CLI)  │  5 tools: browser_start/message/status/stop/screenshot
              └────────┬────────┘
                       │ WebSocket (ws://localhost:7862)
              ┌────────▼────────┐
              │  Chrome Extension│  src/background/service-worker.js
              │  (service worker)│  13 tool handlers, CDP, DOM service
              └────────┬────────┘
                       │ Chrome DevTools Protocol
              ┌────────▼────────┐
              │   Real Browser   │  User's signed-in Chrome
              └─────────────────┘
```

**Alternative path (Managed API):**
```
Partner app → REST API (api.hanzilla.co) → Agent loop on server → Extension executes tools
```

### Key directories

| Path | What | Key files |
|------|------|-----------|
| `src/background/` | Chrome extension core | `service-worker.js`, `modules/mcp-bridge.js`, `modules/api.js`, `modules/cdp-helper.js` |
| `src/background/tool-handlers/` | 13 browser tools | `computer-core.js`, `navigation-core.js`, `form-core.js`, `read-page-core.js`, `utility-core.js` |
| `src/background/managers/` | Tab, debugger, DOM, license | `tab-manager.js`, `debugger-manager.js`, `dom-service/` |
| `server/src/` | MCP server + CLI | `index.ts` (MCP), `cli.ts` (CLI), `agent/loop.ts` (agent loop) |
| `server/src/llm/` | LLM providers | `client.ts` (unified), `vertex.ts` (Vertex AI), `credentials.ts` (key detection) |
| `server/src/managed/` | REST API backend | `api.ts` (73KB, main API), `store-pg.ts` (Postgres), `schema.sql`, `deploy.ts` |
| `server/skills/` | Agent skills (markdown) | Each skill is a `SKILL.md` with instructions |
| `server/src/agent/domain-skills.json` | Domain interaction patterns | Single source of truth for per-domain tips (x.com, linkedin, zillow, amazon…) |
| `sdk/src/` | TypeScript client | `index.ts` — HanziClient class |
| `landing/` | Marketing site (static HTML) | `index.html`, `docs.html`, `embed.js` |
| `examples/` | Demo apps | `x-marketing/` (free tool), `partner-quickstart/` (API demo) |
| `native-host/` | OAuth bridge for extension | `native-bridge.cjs` |
| `server/dashboard/` | Web UI (Preact) | `src/App.jsx` |

### Value propositions

Two positioning anchors. Reuse verbatim when editing marketing copy. Source of truth: `~/.claude/projects/-Users-apple-Dev-llm-in-chrome/memory/product_two_surfaces.md`.

**MCP / CLI — "For your agent"**
A browser sub-agent for your coding agent. One command installs it. Your agent delegates browser work and keeps its context free for code.
- One command setup (`npx hanzi-browse setup` detects 12 agents, wires each one's MCP config).
- Site knowledge built in (21 site playbooks in `server/src/agent/domain-skills.json`).
- Offloads the browser, not your context (main agent fires one tool call; sub-agent runs the loop; returns a clean answer).

**SDK / REST API — "For your product"**
Browser automation for your users, described in English. Your backend sends a natural-language task; your users' own Chrome runs it.
- Describe, don't script (`runTask({ task: "…" })`).
- Your users stay in control (they pair their own browser via a one-click link; you never touch credentials or cookies).
- Same infra as the CLI (every playbook in `domain-skills.json` works on both tracks).

### Modes of operation

Two product tracks sharing one infrastructure (extension + relay + LLM client).

**1. MCP / CLI track (primary product) — agent drives the developer's own Chrome.**
- Installed via `npx hanzi-browse setup`. User's AI agent gains 5 tools: `browser_start`, `browser_message`, `browser_status`, `browser_stop`, `browser_screenshot`.
- BYOM (bring your own model): reads Claude Code OAuth, Codex `auth.json`, macOS Keychain, or `ANTHROPIC_API_KEY`. No data leaves the user's machine.
- Managed: server runs the agent loop against Vertex AI (Gemini). $0.05 per completed task, 20 free/month.
- **Standalone sidepanel** is a minor sibling of MCP: same extension, its own agent loop + native-host credential bridge. Used for direct-chat UI in Chrome's side panel.

**2. SDK / REST API track — partner drives their users' Chrome.**
- REST API (`api.hanzilla.co`) and TypeScript SDK (`@hanzi-browse/sdk`).
- Partner creates pairing token → sends link to end user → end user clicks → browser is connected to the partner's workspace.
- Partner calls `runTask({ browserSessionId, task })` — server runs the agent loop, sends tool execution to the paired extension via the relay, returns result.
- **Free tools** (e.g. `tools.hanzilla.co/x-marketing`) are SDK demos, not MCP demos. They exist to prove the SDK and drive extension installs.

### Skills

Skills are markdown files (`SKILL.md`) that teach AI agents when and how to use browser automation for specific workflows. They're installed into the agent's skills directory during `npx hanzi-browse setup`.

| Skill | What it does |
|-------|-------------|
| `hanzi-browse` | Core — when to use browser tools |
| `e2e-tester` | Test web apps like QA |
| `social-poster` | Post to LinkedIn/X/Reddit |
| `linkedin-prospector` | Find and connect with prospects |
| `a11y-auditor` | Run accessibility audits |
| `data-extractor` | Extract structured data from websites into CSV/JSON |
| `x-marketer` | X/Twitter marketing |

Each skill can also be built as a free tool (web app). The skill provides instructions for local agents; the free tool provides a hosted UI for anyone.

### Domain skills

Domain-specific interaction tips live in `server/src/agent/domain-skills.json` — a single JSON array shared between the server agent, the extension, and the SDK. Each entry has `domain`, `skill` (markdown body), and optional `antiBot: true`.

They're loaded into the agent's system prompt when the task URL matches a known domain (x.com, linkedin.com, gmail, github, zillow, amazon, and ~20 more). They prevent the agent from making known mistakes — e.g., using `form_input` on Draft.js editors (X, LinkedIn), which silently fails.

To add a new domain: append an entry to `server/src/agent/domain-skills.json`. The old `server/site-patterns/` directory is removed — do not re-add it.

### Build

```bash
cd server && npm run build     # TypeScript → dist/
cd .. && npm run build         # Extension → dist/ (Vite)
```

### CLI

```bash
node server/dist/cli.js start "task" --url <url> --context "extra"
node server/dist/cli.js status [session_id]
node server/dist/cli.js message <session_id> "follow-up"
node server/dist/cli.js stop <session_id> [--remove]
```

### Development

```bash
make fresh    # First time: deps + build + DB + start
make dev      # Start everything (DB + migrate + server)
make build    # Rebuild all
make stop     # Stop Postgres
```

- API: http://localhost:3456
- Dashboard: http://localhost:3456/dashboard
- Relay: ws://localhost:7862
- Extension: Load `dist/` in chrome://extensions

### Database

Production: Neon Postgres. Schema: `server/src/managed/schema.sql`.
Development: Docker Postgres on port 5433.
Tables: workspaces, api_keys, browser_sessions, task_runs, task_steps, usage_events.

### Deployment

VPS: DigitalOcean (165.227.120.122). Process manager: systemd.
Domains: api.hanzilla.co (API), relay.hanzilla.co (WebSocket), tools.hanzilla.co (free tools), browse.hanzilla.co (landing, Vercel).
Reverse proxy: Caddy (auto SSL).

Deploy: `ssh vps "cd /opt/hanzi && git pull && cd server && npm run build && systemctl restart hanzi-managed"`

### Tips

- The `--context` flag passes info the agent needs (form data, preferences, tone)
- The `--url` flag sets the starting page for the task
- The Chrome extension must be loaded and running for any mode to work
- Session state stored in `~/.hanzi-browse/sessions/`
- `chrome.tabs.group()` can move tabs across windows — MCP sessions with dedicated windows must skip tab grouping
- Extension code changes require reloading in chrome://extensions, not just restarting the server
- `read_page` returns accessibility tree; `get_page_text` returns visible text. For SPAs like X, `get_page_text` is more reliable.
- Never use `form_input` on Draft.js editors (X, Facebook). Use `javascript_tool` with `execCommand('insertText')` instead.

---
> Source: [hanzili/hanzi-browse](https://github.com/hanzili/hanzi-browse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
