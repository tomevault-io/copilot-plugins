## provenancestamp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Package manager**: pnpm 9.0.0+ required. Node 18.0.0+ required.

```bash
pnpm build          # Build all packages
pnpm dev            # Dev watch mode for all packages
pnpm test           # Run all tests
pnpm lint           # Run linters
pnpm format         # Format with Prettier
pnpm clean          # Clean dist folders
```

**Run tests for a single package:**
```bash
cd packages/sdk && pnpm test
cd packages/sdk && pnpm test -- --run         # single run (no watch)
cd packages/sdk && pnpm test -- tests/crypto.test.ts  # single file
```

**Server environment**: Copy `.env.example` to `.env`, set `MASTER_API_KEY`, then `pnpm dev` from root.

## Architecture

This is a pnpm + Turbo monorepo with three packages:

- **`packages/sdk/`** — `@provenancestamp/sdk`: npm library for stamping AI-generated content. Dual ESM/CJS output via `tsup`. No external crypto dependencies — uses native Web Crypto API throughout.
- **`packages/server/`** — `@provenancestamp/server`: Hono HTTP server with SQLite (better-sqlite3) storing stamp records. Also serves an SSR (no client JS) verification page at `/verify/:id`.
- **`packages/cli/`** — `provenancestamp`: Commander.js CLI that wraps the SDK for file/stdin stamping and verification.

### SDK core flow (`packages/sdk/src/`)

1. `stamp.ts` — `createStamp()` normalizes content (NFC, trim, whitespace collapse), hashes it with SHA-256, builds a deterministically-sorted metadata bundle, computes `HMAC-SHA256(apiKey, stampId + contentHash + metadataHash)`, then POSTs to the server.
2. `crypto.ts` — All crypto primitives via Web Crypto API: SHA-256, HMAC-SHA256, ULID generation (Crockford Base32, 26 chars).
3. `badge.ts` — Generates embeddable badges as HTML, Markdown, JSON-LD structured data, or React JSX.
4. `verify.ts` — `verifyStamp()` fetches stamp from server; `verifyContent()` re-hashes provided content and POSTs to server for comparison.

### Server routes (`packages/server/src/routes/`)

- `POST /api/v1/stamps` — Create stamp (requires `X-API-Key` header)
- `GET /api/v1/stamps/:id` — Retrieve stamp data
- `GET /api/v1/stamps/:id/verify` — Verify stamp exists
- `POST /api/v1/stamps/:id/verify-content` — Verify content hash matches stored record
- `POST /api/v1/keys` — Create API key (requires master key)
- `GET /verify/:id` — Public SSR HTML verification page

SQLite schema is in `packages/server/src/db/schema.ts`. Database uses WAL mode, stored at `./data/provenancestamp.db` or `DB_PATH` env var.

### Key design decisions

- Content hashing is normalized (NFC Unicode + whitespace collapse) so hashes are consistent across environments.
- Metadata bundle keys are sorted before hashing to ensure determinism.
- The server stores `SHA-256(api_key)` in `api_keys.key_hash`, never the raw key.
- The server never stores original prompts — only `prompt_fingerprint` (SHA-256 of prompt).
- Stamp IDs are ULIDs (sortable, no external dep).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nbk7822)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nbk7822)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
