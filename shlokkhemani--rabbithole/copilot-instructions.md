## rabbithole

> If you were sent here to **install** Rabbithole for a user, stop — you don't

# Notes for agents working in this repo

If you were sent here to **install** Rabbithole for a user, stop — you don't
need to clone or build anything. Follow the Quick start in [README.md](./README.md)
(one `claude mcp add` / `codex mcp add` line). This file is for agents
**developing** the repo.

## What this is

An MCP server (stdio) that opens a branching-document canvas in the browser.
Plain ES modules, no build step, no TypeScript, no test framework yet.

- `bin/mcp-server.js` — entry; just imports `src/mcp/server.js`
- `src/mcp/` — MCP wiring (server name `rabbithole`, tools from `src/tools/manifest.js`)
- `src/core/` — sessions, storage, local HTTP + SSE transport, markdown
- `src/core/html/` — the entire canvas UI, served as ONE self-contained HTML
  document (see `src/core/html/README.md`). Client code is authored as JS
  strings/template literals — mind your escaping, especially backslashes
- `website/` — the Next.js site for rabbithole.ing; own package.json,
  `cd website && npm install && npm run dev`

## Run / debug

```bash
npm install
RABBITHOLE_NO_BROWSER=1 node bin/mcp-server.js   # speaks MCP on stdio
```

Storage is JSON files under `~/.rabbithole/` (`RABBITHOLE_DIR` overrides).
Logs go to stderr — stdout is reserved for the MCP protocol; never print to
stdout.

## Conventions

- The product name is **Rabbithole** — one word, no space, in all copy.
- Node ≥ 18, ES modules everywhere.
- The canvas page must stay fully self-contained (one HTML response, no
  external assets) — that constraint is load-bearing for export/snapshots.

---
> Source: [shlokkhemani/rabbithole](https://github.com/shlokkhemani/rabbithole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
