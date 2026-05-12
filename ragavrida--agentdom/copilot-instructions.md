## agentdom

> This file is read by AI coding agents (Claude Code, Cursor, Copilot, etc.) when working in this repository.

# AgentDOM — Agent Instructions

This file is read by AI coding agents (Claude Code, Cursor, Copilot, etc.) when working in this repository.

## What is AgentDOM?

AgentDOM is a universal AI agent protocol. Agents interact with any software — web, desktop, SaaS API, CLI — through a single semantic interface. No screenshots. No scraping.

Core primitive: `dispatch_intent(intent, args, provider)`

## Repository structure

```
agentdom.js              Main entry point / dispatch_intent runtime
cli.js                   CLI entry point (agentdom <command>)
commands/
  auth.js                OAuth/API key auth wallet
  setup.js               One-time interactive credential setup
  wallet.js              Wallet export/import/provision for agents
  launch.js              Browser session launcher
  init.js                Publisher scaffolding
lib/
  secrets.js             Headless credential resolver (7 sources)
  agent-tokens.js        Agent Token Protocol (M2M auth)
  policy.js              Intent approval policy engine
  memory.js              Cross-session episodic memory
  planner.js             Plan-execute-verify runtime
  keychain.js            OS Keychain integration
  oauth-pkce.js          OAuth PKCE + device flow
  resilience.js          Retry/timeout/circuit-breaker
manifests/               Polyfill manifests for 13+ providers
tools/                   Publisher SDK tools
integrations/            Browser/HTTP/tool integrations
desktop-agent/           macOS Accessibility API agent
mcp-*.js                 MCP server implementations
test/                    Test suite (node:test)
website/                 Next.js docs site (getagentdom.com)
docs/                    Markdown documentation
```

## Key design principles

1. **dispatch_intent over everything** — agents declare what, not how
2. **No human in the agent runtime loop** — all auth is headless after one-time setup
3. **Transport hierarchy** — api > cli > browser > desktop (fastest first)
4. **Policy engine** — per-effect allow/prompt/deny before any external action
5. **Publisher protocol** — `.well-known/agentdom.json` + `agent_tokens` endpoint

## Running tests

```bash
npm test   # runs test/core.test.js + test/mcp-servers.test.js via node:test
```

All 33 tests must pass. Tests use CJS (`require`), not ESM.

## Auth / credentials

- `agentdom setup <provider>` — one-time interactive setup
- `agentdom wallet export --base64` — export for agent injection
- `AGENTDOM_<HOST>_KEY` env vars — used in CI/CD
- `~/.agentdom/wallet.json` — local credential store

## Adding a new provider manifest

```bash
# From an OpenAPI spec:
node tools/gen-manifest.js --openapi=./spec.json --host=api.example.com

# Validate:
node -e "const m = require('./manifests/api.example.com.json'); console.log(m.capabilities.length, 'intents')"
```

## Common commands

```bash
agentdom setup linear.app          # one-time OAuth setup
agentdom wallet list               # show stored credentials
agentdom wallet export --base64    # package for agent
agentdom agent-token resend.com    # issue scoped token via protocol
agentdom goal "Create a Linear ticket for the login crash"
```

## Code style

- CJS throughout (`require`/`module.exports`) — no ESM `.js` files
- Use `.mjs` extension for any ESM-only utilities (e.g. `tools/publisher.mjs`)
- No external dependencies for core runtime — stdlib only
- Errors must include actionable hints for the agent

---
> Source: [RagavRida/agentdom](https://github.com/RagavRida/agentdom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
