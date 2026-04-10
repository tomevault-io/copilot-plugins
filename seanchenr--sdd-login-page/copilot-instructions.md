## sdd-login-page

> <!-- OPENSPEC:START -->

<!-- OPENSPEC:START -->

# OpenSpec Instructions

This project uses OpenSpec for Spec-Driven Development (SDD). Specs live in `openspec/specs/`, change proposals in `openspec/changes/`.

## Use `/opsx:*` skills when:

- User wants to explore ideas before committing → `/opsx:explore`
- User wants to propose a new change with all artifacts → `/opsx:propose` or `/opsx:ff`
- User wants to start a change step by step → `/opsx:new`
- Tasks are ready to implement → `/opsx:apply`
- There's a next artifact to create → `/opsx:continue`
- Implementation is done, verify before archive → `/opsx:verify`
- Sync delta specs to main specs → `/opsx:sync`
- Implementation is done → `/opsx:archive`

## Workflow

explore? → propose/ff → apply → verify → sync → archive

- `explore` is optional — skip if requirements are clear
- `/opsx:ff` fast-forwards all artifacts in one shot (recommended)
- `/opsx:propose` same as ff but with a prompt-driven approach

<!-- OPENSPEC:END -->

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
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SeanChenR)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SeanChenR)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
