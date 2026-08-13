## knightaurachess

> This file gives Codex repository-specific guidance for KnightAuraChess.

# AGENTS.md

This file gives Codex repository-specific guidance for KnightAuraChess.

## Build, Run, Test

Node requirement:
- Node 24 is recommended for local development and GitHub Actions.
- `package.json` still allows `>=20.19.0` for local compatibility.

Commands:

```bash
npm install
npm run dev
npm run build
npm run preview

npm run lint
npm run test:engine
npm run test:ui
npm run test:e2e
npm run test

# Firestore rules tests
npm run test:rules:emulator
```

Notes:
- `npm run test:rules` requires Firestore emulator host/port; use `test:rules:emulator`.
- CI runs lint + test + build on Node 22 and 24.

## Environment Variables

Firebase (frontend):

```env
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_PROJECT_ID=

# Optional: full web config JSON or explicit overrides.
VITE_FIREBASE_CONFIG=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=
```

Notes:
- The frontend auto-configures `authDomain` and `storageBucket` from `VITE_FIREBASE_PROJECT_ID`.
- `VITE_FIREBASE_CONFIG` can contain the full Firebase web config JSON.
- Individual `VITE_FIREBASE_*` values override `VITE_FIREBASE_CONFIG`.

Optional frontend:

```env
VITE_AI_DIFFICULTY=medium             # easy|medium|hard|expert
VITE_BASE_PATH=/                      # override Vite base path
VITE_TEXT_AI_BASE_URL=/api/text-ai    # explicit text AI endpoint
VITE_TEXT_AI_MODEL=deepseek/deepseek-r1-0528-qwen3-8b

VITE_MOVE_API_ENABLED=true
VITE_MOVE_API_STRICT=true
VITE_MOVE_API_URL=/api/move
```

Cloudflare Functions backend env:

```env
FIREBASE_PROJECT_ID=
FIREBASE_WEB_API_KEY=
FIREBASE_SERVICE_ACCOUNT_EMAIL=
FIREBASE_SERVICE_ACCOUNT_PRIVATE_KEY=

TEXT_AI_MODEL=@cf/meta/llama-3-8b-instruct
TEXT_AI_UPSTREAM_URL=
TEXT_AI_UPSTREAM_AUTH_BEARER=

```

Cloudflare bindings:
- Workers AI binding name: `knightaurachess` in `wrangler.toml`.
- `/api/text-ai` also auto-detects `AI`, `ai`, and `KNIGHTAURACHESS` to tolerate dashboard-created bindings.
- Run `npm run validate:cloudflare` before Cloudflare deploys when changing `wrangler.toml`.

## Secrets (Do Not Commit Tokens)

Never place real tokens in source files, `AGENTS.md`, or git history.

Use these secret names instead:

- GitHub Actions secrets:
  - `GH_PAT_REPO_RW` (repo content write if needed)
  - `GH_PAT_ACTIONS_RW` (only if workflow updates/dispatch are required)
  - `VITE_FIREBASE_API_KEY`
  - `VITE_FIREBASE_PROJECT_ID`
  - `VITE_FIREBASE_CONFIG` (optional full web config JSON)
  - `VITE_FIREBASE_AUTH_DOMAIN`
  - `VITE_FIREBASE_STORAGE_BUCKET`
  - `VITE_FIREBASE_MESSAGING_SENDER_ID`
  - `VITE_FIREBASE_APP_ID`

- Cloudflare (Pages/Workers) secrets or env:
  - `FIREBASE_PROJECT_ID`
  - `FIREBASE_WEB_API_KEY`
  - `FIREBASE_SERVICE_ACCOUNT_EMAIL`
  - `FIREBASE_SERVICE_ACCOUNT_PRIVATE_KEY`
  - `TEXT_AI_UPSTREAM_URL`
  - `TEXT_AI_UPSTREAM_AUTH_BEARER`

- Local development (`.env`, gitignored):
  - use the same key names above for local testing.

## Deployment Model

- Frontend: Vite SPA deployed to Cloudflare Pages (`wrangler.toml`).
- Backend APIs (same origin):
  - `functions/api/move.js`: server-authoritative move execution.
  - `functions/api/text-ai.js`: proxy endpoint for chat AI.
- Firestore rules are in `firestore.rules`.

## Architecture Summary

Core files:

```text
src/
  App.jsx                      # Primary app state/controller
  KnightJumpChess.js           # Variant engine on top of chess.js
  contexts/AuthContext.jsx     # Firebase auth/profile wiring
  utils/firebase.js            # Firebase init + firebaseEnabled
  utils/moveApi.js             # Client wrapper for /api/move
  utils/textAi.js              # Bot DM Text AI calls + sanitization
  workers/aiWorker.js          # Web Worker chess AI engine
  workers/useOnlineClock.js    # Online clock hook
  components/
    BoardShell.jsx
    GameSidebar.jsx
    PlayTabPanel.jsx
    HomePage.jsx
    social/DmConversation.jsx
```

Game modes:
- Practice: `!isOnline && !aiEnabled`
- Local AI: `!isOnline && aiEnabled`
- Online: `Boolean(gameId)`

`gameId` is persisted in `localStorage` (`cr_gameId`).

## Current Important Behaviors

- Promotion supports `q/r/b/n` choices.
- Online games use `moveSeq` sequencing to prevent stale writes.
- When enabled, `/api/move` is used for authoritative move handling.
- Bot DM chat retries retryable Text AI failures (e.g., 405/503) silently.
- On `knightaurachess.com`, Text AI defaults to same-origin `/api/text-ai`.

## Firestore Data (High-level)

`games` documents include:
- player ids/names/ratings
- `fen`, `moveHistory`, `lastMove`, `moveSeq`
- `status`, `winner`, `result`
- clocks: `timeControl`, `whiteTimeLeft`, `blackTimeLeft`, `lastMoveAt`

`users`, `dms`, `friend_requests`, `game_challenges`, `announcements` are used by profile/social flows.

## Editing Guidance for This Repo

- Do not reintroduce client-only authoritative online move writes in strict mode.
- Keep `/api/move` response/error codes stable (`STALE_MOVE_SEQ`, `ILLEGAL_MOVE`, etc.).
- Keep bot DM behavior non-spammy: retryable AI errors should stay quiet to users.
- Preserve fallback behavior when Firebase is not configured (`firebaseEnabled === false`).

---
> Source: [maiwending/KnightAuraChess](https://github.com/maiwending/KnightAuraChess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
