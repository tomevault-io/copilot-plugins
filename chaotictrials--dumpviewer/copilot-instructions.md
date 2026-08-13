## dumpviewer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Setup

```bash
npm run install:all          # Install frontend + backend dependencies
```

### Development

```bash
npm run dev                  # Start both frontend (port 5173) and backend (port 3001) with hot reload
npm run dev:frontend         # Frontend only
npm run dev:backend          # Backend only (tsx watch)
```

### Build

```bash
npm run build                # Build frontend (tsc + vite)
npm run build:backend        # Build backend (tsc, copies platform-ids.json to dist/)
```

### Testing

```bash
npm test --prefix backend    # Run backend tests
npm test --prefix frontend   # Run frontend tests
npm run test:all             # Run all tests
```

To run a single test file or suite, use vitest's filter:

```bash
cd backend && npx vitest run src/app.test.ts
cd frontend && npx vitest run src/some.test.ts
```

Frontend vitest runs with `environment: 'node'` (no jsdom), and several modules (e.g. `useHighlighterTheme`) touch `document` at import time — so components cannot be imported in tests. To make component logic testable, extract the pure part into `frontend/src/utils/` and test that (see `utils/configDiff.ts`, extracted from `ConfigViewer`).

### Formatting

```bash
npm run format               # Write (Prettier)
npm run format:check         # Check only
```

## Architecture

This is a React + Express monorepo for viewing [Skyblock Builder](https://github.com/ChaoticTrials/SkyblockBuilder) dump files (`.zip` archives containing a `manifest.json` and game files).

### Key design principle: client-side first

The frontend can parse and display dumps entirely locally — no upload required. The backend is optional and adds: storing dumps by manifest ID, importing from URLs, modpack export, and TTL-based cleanup.

### Backend (`backend/src/`)

- **`index.ts`** — Entry point. Runs `cleanupOldDumps()` on startup and every 24 hours, then starts the HTTP server.
- **`app.ts`** — The entire Express application. Exports `app`, plus utility functions used directly by tests (`isSafeUrl`, `isValidId`, `validateAndExtractManifestId`, `extractManifestInfo`, `parseTtlMs`, `cleanupOldDumps`, `generateDeleteKey`, `resolveDeleteKey`).
- **`platform-api.ts`** — HTTP clients for CurseForge (`curse.moddingx.org`) and Modrinth (`api.modrinth.com`) used for modpack export.
- **`platform-ids.json`** — Static map of mod keys → `{ curseforge, modrinth }` IDs used for modpack generation.

**Storage** is file-based, no database. Each dump is stored as `{DUMPS_DIR}/{uuid}.zip` with a sidecar `{uuid}.meta` (JSON with `expiresAt`/`createdAt` timestamps, `manifestVersion`, the full manifest `versions` object, and optional `hashes`). `GET /api/dumps` backfills `manifestVersion`/`versions` into legacy sidecars on first listing; a persisted `manifestVersion: null` marks an unreadable zip (not retried), while a missing key means "not yet inspected".

**Delete keys** are RSA-2048 tokens: `privateEncrypt(id)` → base64url. The key pair is generated on first startup and persisted in `{DUMPS_DIR}/keys/`. `GET /api/delete/:key` shows an HTML confirmation page (never deletes — safe against link prefetchers); `POST /api/delete/:key` performs the deletion. Both recover the ID by decrypting with the public key — no token storage needed — and share a dedicated rate-limit bucket.

**Auth** is global: a single `AUTH_TOKEN` (env var or `/run/secrets/dumpviewer_token`) gates all write endpoints. The delete-by-key routes are intentionally auth-free. Brute-force protection: failed auth attempts are delayed by `AUTH_FAIL_DELAY_MS` (default 500 ms) and counted by a dedicated `authFailureLimiter` (only 401 responses count, via `requestWasSuccessful`; `AUTH_FAIL_LIMIT` failures per 15 min → IP lockout). Every endpoint sits behind a rate-limit bucket except `/health`; `GENERAL_RATE_LIMIT` and `MODPACK_RATE_LIMIT` tune the read and modpack buckets.

**SSRF protection** for URL imports is two-layered: `isSafeUrl()` string checks, plus `safeLookup()` — a validating DNS lookup wired into an undici `Agent` so every connection (including redirect hops) rejects hostnames resolving to private/loopback/link-local addresses (DNS rebinding). Import fetches therefore go through `undici`'s `fetch`, not the global one; `app.test.ts` mocks `undici.fetch` by delegating to `globalThis.fetch` so `vi.stubGlobal('fetch', ...)` still intercepts imports.

**Zip handling**: always check `entry.header.size` (declared uncompressed size) against a limit before calling `getData()` on untrusted zips — decompression-bomb protection (`MAX_MANIFEST_ENTRY_BYTES`, `MAX_OVERRIDE_ENTRY_BYTES`).

**Test rate-limit budget**: the upload rate limiter (10 POSTs / 10 s) is shared across the whole `app.test.ts` run and is fully consumed by the existing suite — a net-new `POST /api/dump/*` request to `app` makes the _last_ POSTs in the run fail with 429. Seed dumps by writing `{id}.zip`/`{id}.meta` directly to disk, piggyback assertions onto existing successful uploads, or unit-test exported helpers instead. The general/modpack/auth-failure buckets are env-configurable and set very high in `vitest.config.ts`; to test limiter behaviour itself, set the env var low and re-import the app with `vi.resetModules()` (fresh module = fresh buckets).

### Frontend (`frontend/src/`)

- **`App.tsx`** — Root component. Handles URL routing (UUID in pathname → fetch from backend), drag-and-drop, and top-level state.
- **`manifest/`** — Version-aware dump parsing:
  - `index.ts` — `parseDump(file)` detects manifest version (1 or 2) and routes to the correct parser. Returns `ParsedDump { manifest: AnyManifest, files: Map<string, DumpFile> }`.
  - `v1/` and `v2/` — Each has `types.ts` (TypeScript interfaces) and `zipParser.ts` (jszip-based parsing logic).
  - v2 extends v1: adds per-mod `hashes` and changes `changedFormat` to always be `'diff'`.
  - jszip cannot read `File` objects under Node/vitest — `parseDump` converts to ArrayBuffer first (`await file.arrayBuffer()`), which keeps zip-parsing testable in the node environment.
- **`components/`** — UI components. `FileViewer.tsx` delegates to type-specific viewers under `viewers/`.

### Frontend ↔ Backend integration

- `VITE_API_URL` (default `http://localhost:3001`) prefixes all API calls.
- In production, the backend serves the frontend's `dist/` as static files and falls back to `index.html` for SPA routing.
- The frontend detects a UUID v4 in the URL path and fetches the dump zip from `GET /api/dump/:id`.
- **Dev has no Vite proxy**: `/api/*` only exists on the backend (port 3001). Hitting `localhost:5173/api/...` reaches the SPA, which silently falls back to the home page — API URLs (uploads, delete links) must use port 3001 in dev. In production both live on one port, so this gotcha is dev-only.

## Workflow

### New features

Plan the feature first, then write tests before implementing. Tests should cover the new behavior end-to-end before the implementation is complete.

### Keeping documentation in sync

- **API changes** (new endpoints, changed request/response shapes): update `API.md`.
- **Docker changes** (env vars, ports, volume mounts, build steps): update the `Dockerfile`, `docker-compose.example.yml`, and any relevant examples in `README.md`; verify the image still builds with `docker build .`.
- **User-visible behaviour changes** (new features, changed defaults, new env vars): update `README.md`.

### After any code change

Always run the formatter and the full test suite before finishing:

```bash
npm run format
npm run test:all
```

### Committing

Run format and tests first (see above). Commit messages must be a single subject line — no body, no bullet points.

### Updating this file

When you learn something new about this project or the preferred workflow during a session — a constraint, a convention, a decision made — append it to the relevant section of this file so future sessions start with that knowledge.

### Manifest versions

| Version | Key difference                                                     |
| ------- | ------------------------------------------------------------------ |
| v1      | Config diffs are always `json5` format                             |
| v2      | Config diffs are always `diff` format; adds per-mod `hashes` field |

When adding support for new dump features, check both `v1/` and `v2/` parsers.

---
> Source: [ChaoticTrials/DumpViewer](https://github.com/ChaoticTrials/DumpViewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
