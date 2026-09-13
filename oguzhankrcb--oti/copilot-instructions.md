## oti

> OTI (One Time Information) — AdonisJS 7 (ESM, TypeScript) app for one-time secret sharing. All encryption happens client-side; the server only ever stores ciphertext.

# AGENTS.md

OTI (One Time Information) — AdonisJS 7 (ESM, TypeScript) app for one-time secret sharing. All encryption happens client-side; the server only ever stores ciphertext.

## Commands

- `npm run dev` — dev server with HMR on port 3333 (only `app/controllers/**` and `app/middleware/*` hot-reload; other changes restart the server)
- `npm test` / `node ace test` — Japa runner
  - Single file: `node ace test --files="secret_share"`
  - One suite: `node ace test functional` (suites are positional args; suites: `unit`, `functional`)
- `npm run typecheck` — `tsc --noEmit`; `npm run lint`; `npm run format` (prettier)
- Migrations: `node ace migration:run`

Verification order for non-trivial changes: `lint` → `typecheck` → `test`.

## Infrastructure

- Requires **MySQL** (3306) and **Redis** (6379) running locally for dev. `docker-compose.yml` has no MySQL service — start Redis only with `docker compose up redis`; MySQL must exist on the host.
- Set `LIMITER_STORE=memory` in `.env` to run without Redis.
- Config is MySQL-only (`mysql2` in `config/database.ts`) despite the README claiming SQLite/PostgreSQL support. Trust code over README — README is also stale on the cleanup interval (code runs every 30 min, not 15).

## Tests

- `tests/` contains unit tests and full-app functional tests under `tests/unit/**/*.spec.ts` and `tests/functional/**/*.spec.ts`.
- Functional tests boot the full app and HTTP server, so they need MySQL and (unless `LIMITER_STORE=memory`) Redis.

## Architecture notes that change how you work

- **Encryption boundary is the core invariant**: RSA keys are generated in the browser (`resources/js/encryption.js`, JSEncrypt), the private key is pako-compressed into the URL fragment and never sent to the server. Never add server-side decryption or persist keys — a past migration (`1741639567133_remove_public_key_column...`) deliberately removed the `public_key` column.
- **Payload format is hybrid (v2)**: content envelope `{t, n, c}` → AES-256-GCM (WebCrypto) → AES key wrapped with RSA. Stored payload: `{v: 2, k, iv, d}`. `Encryption.decrypt` still handles legacy raw-RSA ciphertext — keep that backward compatibility. File uploads are read/validated/encrypted fully client-side (`resources/js/file_sanitizer.js`); the server never receives raw files by design.
- **Single-view semantics**: `SharesController.getEncryptedText` calls `markAsViewed()` — ciphertext is nulled immediately, but the row shell stays until expiry so the burn receipt (`/receipt/:receiptId`) keeps working. Do not "fix" this into an idempotent GET, and do not re-delete the row on read. `encrypted_text` is nullable for this reason.
- **All `/share` routes are throttled to 10 req/min** (global limiter in `start/limiter.ts`), including plain page GETs. 429s while clicking around in dev are expected; custom 429 page renders via `app/exceptions/handler.ts`. `/receipt/:id` is deliberately outside the throttled group.
- **Scheduler**: custom node-cron wrapper in `app/services/scheduler/`. It only starts when `START_SCHEDULER=true` — checked via raw `process.env` in `providers/scheduler_provider.ts` (the var is deliberately NOT in the `start/env.ts` schema). New tasks: implement the `Task` interface and register in `app/services/scheduler/index.ts`.
- New env vars must be added to the schema in `start/env.ts` or `env.get()` will fail.
- `passwordHash` on `SecretShare` is plain SHA-256 via `node:crypto`, not `@adonisjs/hash` — deliberate repo choice.

## Conventions

- Imports use `#alias/*` subpath imports (see `package.json#imports`) pointing at `.js` extensions, e.g. `import SecretShare from '#models/secret_share'`. Controllers are lazy-imported in `start/routes.ts`.
- Prettier (from `@adonisjs/prettier-config`): no semicolons, single quotes, print width 100. Match existing style; run `npm run format` before finishing.
- Vite entrypoints are only `resources/css/app.css` and `resources/js/app.js`; views are Edge templates in `resources/views/`. New frontend files must be reachable from an entrypoint or they won't be bundled. Page-inline `<script>` in Edge templates can't use imports — browser utilities are exposed on `window` via `resources/js/app.js` (`Encryption`, `FileSanitizer`, `QRCode`).
- Deploy: the Dockerfile bakes `.env.production` into the image (`COPY .env.production ./`). That file is gitignored (only `.env.example` is tracked) and must exist in the build context; it contains a real APP_KEY/DB password — be careful when rotating secrets or changing the deploy flow.
- No CI workflows, no pre-commit hooks exist in this repo.

## Marketing and promotion writing

- Use a simple, personal and sincere tone for OTI promotion, especially in Chrome-prepared Reddit, forum and community posts.
- Present OTI as a small, simple open source project made to learn, share useful work and contribute to the community.
- Mention recent updates in short, concrete sentences. Keep the writing warm and honest instead of polished, corporate or sales-focused.
- Never use the em dash or en dash characters in promotional copy.
- Do not ask questions in promotional posts or comments. Use statements such as `Feedback is welcome` when appropriate.
- Avoid hype, jargon, rhetorical questions, long technical explanations, dense lists and wording that sounds generated by an AI.

---
> Source: [oguzhankrcb/OTI](https://github.com/oguzhankrcb/OTI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
