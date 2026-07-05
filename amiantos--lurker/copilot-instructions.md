## lurker

> Guidance for AI coding agents (and the humans driving them) contributing to

# AGENTS.md

Guidance for AI coding agents (and the humans driving them) contributing to
Lurker. If you're a person, [`README.md`](README.md) is the friendlier intro;
this file is the fast, dense orientation an agent needs to make a correct change
and open a clean PR. Everything here is enforced by CI or by review — following
it is the difference between a merge and a round of change requests.

Lurker is a self-hosted IRC client: an always-on Node server that stays
connected to IRC and keeps full history, plus a Vue 3 web UI that reattaches
from any browser. License is **MPL-2.0** throughout.

Please note that while Lurker is an LLM/Agent friendly project, the project is
under the direction of Brad Root (amiantos), and just because a feature
can be added, does not mean it will be accepted. Lurker is deliberately lo-fi
in many ways, despite the feature set being very modern. So, before you
commit yourself to working on something that excites you in Lurker, be sure to
have a discussion with amiantos in the #lurker IRC channel on Libera.Chat to
make sure it fits the project ethos.

It should also be assumed most GitHub issues are 'assigned' by default to amiantos.
If you wish to work on an issue, be sure to ask him about it in the channel first,
to avoid potentially wasted time. Also note that while I have been saying 'you'
in the last two paragraphs, but you should probably let your operator handle the
talking, unless they really would want you to sign onto IRC to try to talk to amiantos
directly. Please let him know you're an agent up-front just to be nice.

## Repository layout

```
server/            TypeScript on Node, run via tsx (no build step)
  server.ts        Process lifecycle: HTTP server, WS hub, IRC manager, identd
  app.ts           Express app construction + route wiring (edition-gated)
  routes/          One Express router per resource (auth, networks, uploads…)
  services/        IRC manager + connection, WS hub, image pipeline, identd,
                   MCP server, connect scheduler, upload providers
  services/verbs/  Shared ACTION registry (sendMessage, searchMessages, …)
                   consumed by BOTH the WS command path and the MCP server
  db/              better-sqlite3 data access, one module per table
  db/index.ts      The single DB connection + schema migrations
  middleware/      auth (session cookie), apiAuth (bearer token), nodeAuth
  utils/           small helpers (edition, ident, secretCrypto, username…)
  types/           ambient *.d.ts shims for untyped deps
  test-utils/      testApp.ts — the integration-test harness
shared/            Code imported by BOTH server and client
  settingsRegistry.ts  Single source of truth for user settings
vue_client/        Vue 3 + Vite + Pinia + vue-router SPA
  src/stores/      Pinia stores, one per domain (buffers, auth, settings…)
  src/components/  Vue components (+ settings-panes/)
  src/composables/ useSocket, usePresence, useKeyboardShortcuts, …
  src/views/       Login, Chat (Desktop/Mobile), Settings, InviteAccept
  src/lib/         framework-free logic (e.g. virtualBuffers)
docs/              SELF_HOSTING, digitalocean, MCP, DESIGN_TOKENS
deploy/            operator deploy scripts (not needed for app dev)
integrations/      autonotes
```

Tests live **next to the code** they cover as `*.test.ts`.

## Setup & dev

- **Node 22** — matches CI and the Docker runtime. better-sqlite3 + sharp are
  native; stay on 22 to avoid ABI surprises.
- `npm run install:all` — installs root **and** `vue_client/` deps.
- `cp .env.example .env` — defaults are documented inline.
- `npm run dev` — runs server + client concurrently.
  - ⚠️ The server **auto-connects (by default) on boot to whatever IRC networks are
    in its database**. Don't point a dev instance at networks you don't control, and
    don't assume "the server is running" is a safe way to test — prefer the
    typecheck/lint/test gate below, which needs no live IRC.

## The CI gate — run before every PR

CI ([`.github/workflows/test.yml`](.github/workflows/test.yml)) runs these on
every PR to `main`, and **all must pass**:

| Command                    | What it is                         |
| -------------------------- | ---------------------------------- |
| `npm run typecheck`        | server type-check (`tsc`, no emit) |
| `npm run typecheck:client` | client type-check (`vue-tsc`)      |
| `npm run lint`             | **oxlint**                         |
| `npm run format:check`     | **oxfmt**                          |
| `npm test`                 | **Vitest**                         |

`npm run check` bundles the first four. Run it plus `npm test` and you've
reproduced the gate locally.

## Code style & conventions

- **Formatter is `oxfmt`, NOT Prettier.** Run `npm run format`. Do **not** run
  Prettier or ESLint — they fight oxfmt's house style (single quotes, 100-col
  width) and will make `format:check` fail. Linter is `oxlint`.
- **ESM with `.js` import specifiers.** The project is `"type": "module"` with
  `verbatimModuleSyntax`. Import sibling TypeScript files using a `.js`
  extension even though the file on disk is `.ts`:
  ```ts
  import { createUser } from '../db/users.js'; // file is users.ts
  ```
  Omitting the extension, or writing `.ts`, breaks module resolution.
- **`import type` for type-only imports** — required by `verbatimModuleSyntax`.
- **TypeScript `strict` is on** (plus `noImplicitOverride`,
  `noFallthroughCasesInSwitch`). Nothing is compiled by `tsc`: `tsx` runs the
  server, Vite builds the client — `tsc`/`vue-tsc` are type-checkers only.
- **Unused vars:** prefix with `_` to silence the lint rule.
- **License header on every new source file.** MPL-2.0, with the comment syntax
  of the file type:
  ```ts
  // Copyright (c) 2026 Brad Root
  // SPDX-License-Identifier: MPL-2.0
  ```
  (`<!-- … -->` for HTML, `/* … */` for CSS.) The whole tree is MPL-2.0 — no
  other SPDX identifier should appear.
- **Match the surrounding code.** Mirror the existing naming, comment density,
  and idiom of the file you're editing rather than importing your own style.

## Testing

- **Vitest**, tests colocated as `*.test.ts`.
- **Prefer integration tests over unit tests** for routes and services: drive
  the real Express router against a real (temporary) SQLite DB rather than
  mocking. The harness is [`server/test-utils/testApp.ts`](server/test-utils/testApp.ts):
  ```ts
  import { setupTestDb, createTestApp, createAuthedAgent } from '../test-utils/testApp.js';
  setupTestDb(); // MUST be at module top level, before any db import
  // …in beforeAll: createTestApp({ '/api/foo': fooRouter }), createAuthedAgent(app, user.id)
  ```
  Reserve plain unit tests for genuinely tricky pure logic (parsers, mask
  matching, message splitting).
- **Never point tests at `data/`.** `setupTestDb()` creates a throwaway temp DB
  per test file; lean on it. Don't read or write the real `data/` directory.

## Architecture notes & gotchas

These are the non-obvious constraints that have bitten changes before:

- **One shared SQLite connection.** `server/db/index.ts` opens a single
  better-sqlite3 connection (WAL, `synchronous=NORMAL`, `busy_timeout=5s`).
  better-sqlite3 is **synchronous** — long queries block the event loop that
  also serves WebSocket fan-out and IRC sockets. **Do not hold a long-lived
  `.iterate()` streaming cursor:** a streamed read open across concurrent writes
  throws `database connection is busy` and crashes the process. Read large sets
  with keyset pagination — `WHERE id > ? ORDER BY id LIMIT N`, a discrete
  `.all()` per page, yielding with `setImmediate` between pages.
- **No `worker_threads`.** Under `tsx` on Node 22 (the deploy runtime) the
  `.js`→`.ts` loader does not propagate into worker threads, so a TS worker
  entry fails with `Cannot find module` at runtime even though it type-checks
  and runs on newer Node. Use in-process `setImmediate` chunking for heavy work.
- **Fold IRC target case when matching buffers.** Servers send channel and nick
  names with inconsistent casing. Never look a buffer up by exact key — match
  case-insensitively (client: the buffers store's `findByTarget`/`findDm`;
  server: the case-folding helpers). House style is ASCII `toLowerCase`.
- **The image pipeline is server-side (`sharp`), not browser canvas** — and it
  **must preserve animation**: never re-encode GIF / animated WebP / APNG.
- **Cross-device features belong on the server.** Presence, read-state sync,
  notifications, away state — anything that must stay consistent across a user's
  tabs and devices lives in the server + WS fan-out, not client-only state.
  (Render-time _display_ filters can be client-side.)
- **Settings are registry-driven.** Add new user settings to
  [`shared/settingsRegistry.ts`](shared/settingsRegistry.ts) (data-only, imported
  by both sides) — don't scatter setting definitions across server and client.
- **Two editions.** `LURKER_EDITION` defaults to `standalone` (normal
  self-hosting). `node` is the managed-hosting "cell" edition, gated by operator
  env (orchestrator URL, fleet secret, forced uploader) and isolated in
  `server/app.ts`. Almost all contributions target standalone; you don't need
  any node config to develop. Just don't break the standalone path when touching
  edition-gated surfaces.

## UI / design conventions

- **One font size across the entire UI.** Never set `font-size` (sole
  exception: `clamp()` on a hero/brand title). Build hierarchy with color,
  weight, spacing, and layout instead.
- **Use design tokens, don't hardcode.** Two tiers (themeable vs internal) —
  spacing scale, z-index ladder, radius, and scrim live in
  `vue_client/src/assets/main.css`. See [`docs/DESIGN_TOKENS.md`](docs/DESIGN_TOKENS.md).

## Contributing & PRs

- Branch off `main` and open your PR against `main`. `main` is protected:
  requires a green CI run (typecheck server + client, lint, format, test) **and**
  a review before it can merge.
- Keep PRs focused on a single change; describe what and why.
- **Security issues:** do not open a public issue — see
  [`SECURITY.md`](SECURITY.md) for private reporting.
- Lurker can be driven programmatically over MCP / HTTP; the API is documented
  in [`docs/MCP.md`](docs/MCP.md).

---
> Source: [amiantos/lurker](https://github.com/amiantos/lurker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
