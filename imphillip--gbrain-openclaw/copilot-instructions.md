## gbrain-openclaw

> Personal knowledge brain built for OpenClaw — SQLite + FTS5 + vector embeddings, MCP-native.


# GBrain for OpenClaw

Personal knowledge brain built for OpenClaw — SQLite + FTS5 + vector embeddings, MCP-native.

## Project structure

```
src/cli.ts              # main entrypoint, command dispatcher
src/commands/           # one file per CLI command
src/core/               # db.ts, fts.ts, embeddings.ts, markdown.ts, links.ts
src/mcp/server.ts       # MCP server (stdio transport) — primary OpenClaw integration point
skills/*/SKILL.md       # fat skill files — deploy to ~/.openclaw/workspace/skills/
hooks/*/                # OpenClaw hooks (HOOK.md + hook.ts)
bin/gbrain              # compiled binary (bun build --compile)
test/core.test.ts       # tests
```

## Key conventions

- DB: `bun:sqlite`, default `~/.openclaw/brain.db`, overridable via `GBRAIN_DB` env or `--db` flag
- Slugs: `people/jane-doe`, `companies/acme-corp` (type-prefix + hyphenated name)
- Page architecture: compiled_truth (above `---`, rewritten) + timeline (below `---`, append-only)
- Embeddings: Float32Array blobs, pure JS cosine similarity, no native extensions
- Embeddings need `OPENAI_API_KEY` — gracefully degrade if absent

## Dev workflow

```bash
bun run src/cli.ts <command>   # run in dev mode
bun test                       # run tests
bun build --compile --outfile bin/gbrain src/cli.ts  # compile binary
```

## OpenClaw integration

- MCP server: `gbrain serve --db ~/.openclaw/brain.db` — register in `~/.openclaw/openclaw.json` under `mcp.servers`
- Skills: copy `skills/*/` to `~/.openclaw/workspace/skills/`
- Hook: see `hooks/gbrain-ingest-session/HOOK.md` for session auto-ingest setup

## Skills

Read `skills/ingest/SKILL.md` before ingesting any content.
Read `skills/query/SKILL.md` before answering questions from the brain.
Read `skills/maintain/SKILL.md` before running maintenance.
Read `skills/enrich/SKILL.md` before enriching pages from external APIs.
Read `skills/briefing/SKILL.md` before compiling a briefing.

---

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
- Bun automatically loads .env, so don't use dotenv.

## APIs

- `Bun.serve()` supports WebSockets, HTTPS, and routes. Don't use `express`.
- `bun:sqlite` for SQLite. Don't use `better-sqlite3`.
- `Bun.redis` for Redis. Don't use `ioredis`.
- `Bun.sql` for Postgres. Don't use `pg` or `postgres.js`.
- `WebSocket` is built-in. Don't use `ws`.
- Prefer `Bun.file` over `node:fs`'s readFile/writeFile
- Bun.$`ls` instead of execa.

## Testing

Use `bun test` to run tests.

```ts#index.test.ts
import { test, expect } from "bun:test";

test("hello world", () => {
  expect(1).toBe(1);
});
```

## Frontend

Use HTML imports with `Bun.serve()`. Don't use `vite`. HTML imports fully support React, CSS, Tailwind.

Server:

```ts#index.ts
import index from "./index.html"

Bun.serve({
  routes: {
    "/": index,
    "/api/users/:id": {
      GET: (req) => {
        return new Response(JSON.stringify({ id: req.params.id }));
      },
    },
  },
  // optional websocket support
  websocket: {
    open: (ws) => {
      ws.send("Hello, world!");
    },
    message: (ws, message) => {
      ws.send(message);
    },
    close: (ws) => {
      // handle close
    }
  },
  development: {
    hmr: true,
    console: true,
  }
})
```

HTML files can import .tsx, .jsx or .js files directly and Bun's bundler will transpile & bundle automatically. `<link>` tags can point to stylesheets and Bun's CSS bundler will bundle.

```html#index.html
<html>
  <body>
    <h1>Hello, world!</h1>
    <script type="module" src="./frontend.tsx"></script>
  </body>
</html>
```

With the following `frontend.tsx`:

```tsx#frontend.tsx
import React from "react";
import { createRoot } from "react-dom/client";

// import .css files directly and it works
import './index.css';

const root = createRoot(document.body);

export default function Frontend() {
  return <h1>Hello, world!</h1>;
}

root.render(<Frontend />);
```

Then, run index.ts

```sh
bun --hot ./index.ts
```

For more information, read the Bun API docs in `node_modules/bun-types/docs/**.mdx`.

---
> Source: [imphillip/gbrain-openclaw](https://github.com/imphillip/gbrain-openclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
