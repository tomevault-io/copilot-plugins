## mimo2codex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

mimo2codex is a local HTTP proxy (Node.js / TypeScript) that lets OpenAI's Codex CLI and desktop app talk to non-OpenAI LLM providers (MiMo, DeepSeek, and generic OpenAI-compatible services). It translates Codex's Responses API into Chat Completions on the fly, with per-request provider routing based on the `model` field.

## Build and run commands

```bash
npm install                # install backend deps
npm run build              # compile TypeScript → dist/
npm run build:all          # backend + frontend (web/ admin UI)
npm run dev                # run from source via tsx (auto-ensures web build)
npm start                  # run compiled dist/cli.js
npm test                   # vitest run (all tests)
npm run test:watch         # vitest watch mode
npx vitest run test/reqToChat.test.ts   # single test file
npm run web:install        # install web/ deps (needed once before web:build)
npm run web:dev            # Vite dev server on :5173, proxies /admin/api → :8788
npm run web:build          # production build of admin UI → dist/web/
```

Release: `npm run release:patch`, `release:minor`, `release:major`, `release:beta`. See PUBLISHING.md for the full runbook.

## Architecture

### Request flow

```
Codex Client → mimo2codex (:8788) → Provider selection (model field)
  → reqToChat (Responses API → Chat Completions)
  → upstream HTTP call to provider
  → respToResponses / streamToSse (Chat Completions → Responses API SSE)
  → back to Codex
```

### Key source directories

- **`src/providers/`** — Provider abstraction. `types.ts` defines the `Provider` interface; `mimo.ts`, `deepseek.ts`, `generic.ts` implement it. `registry.ts` maps model names to providers. MiMo-specific behavior (web_search, thinking injection) is confined to `mimo.ts` — don't leak it to generic layers.
- **`src/translate/`** — Bidirectional API translation. `reqToChat.ts` (Responses → Chat Completions), `respToResponses.ts` (reverse), `streamToSse.ts` (streaming), `minimaxCompat.ts` (MiniMax quirks). These are provider-agnostic.
- **`src/upstream/`** — HTTP client for upstream providers. `openaiCompatClient.ts` handles both streaming and non-streaming calls. `contextOverflow.ts` handles token limit errors.
- **`src/db/`** — SQLite persistence via better-sqlite3. Chat logs, model catalog, runtime settings (e.g. thinking mode toggle). Default path: `~/.mimo2codex/data.db`.
- **`src/admin/router.ts`** — REST API (`/admin/api/*`) and SPA hosting (`/admin/`).
- **`src/server.ts`** — Core HTTP handler. `selectProvider` resolves which provider serves each request. `resolveDisableThinking` / `resolveForceHighEffort` read DB settings per-request (no restart needed).
- **`src/cli.ts`** — Entry point. Subcommands: default (start server), `init`, `update`, `print-config`, `print-cc-switch`.
- **`web/`** — React 18 + Ant Design 5 admin console, separate Vite workspace. Bilingual (EN/中文) via i18next. Builds to `dist/web/`.
- **`mimoskill/`** — Python helpers (stdlib only, no pip). OCR, image gen, pet gen. Used when MiMo lacks native capabilities.

### Provider routing

The `model` field in each request determines the provider. Routing is per-request, configured via `src/providers/registry.ts` and overridable at runtime through the admin UI (DB overrides). Generic providers can be loaded from a `providers.json` file via `src/providers/genericLoader.ts`.

### Config resolution order

Settings like `disableThinking` follow: CLI flag/env → admin DB setting → default. This lets the admin UI change behavior without restarting.

## TypeScript conventions

- ESM (`"type": "module"` in package.json, `NodeNext` module resolution)
- Strict mode with `noUnusedLocals`, `noUnusedParameters`, `strictNullChecks`
- All imports use `.js` extension (NodeNext requirement)
- Target: ES2022

## Testing

Tests live in `test/` and follow the naming pattern `<module-path>.test.ts` (dots replacing slashes, e.g. `providers.deepseek.test.ts`). Vitest with Node environment. Tests are self-contained — no external services needed.

## Hard rules from AGENTS.md

- Never `pip install openai` or `import openai` — the user's keys are for MiMo/DeepSeek, not OpenAI.
- Never assume image generation is available natively — use `mimoskill/` helpers.
- For image input on non-vision models, use `mimoskill/scripts/ocr.py` instead of asking users to switch models.
- MiMo quirks: use `max_completion_tokens` (not `max_tokens`); `image_url` content requires a `text` part in the same array; reasoning is `reasoning_content` (not `reasoning_summary`); `web_search` is a builtin tool type, not a function tool.

## Change workflow rules

Project-level rules for every change made in this repo.

### 1. Never commit on the user's behalf

Do **not** run `git commit`, `git push`, `git add` with intent to commit, `git tag`, or any release script (`npm run release:patch`, `release:minor`, `release:major`, `release:beta`). The user reviews the working tree and commits / publishes manually.

Read-only git inspection (`git status`, `git diff`, `git log`) and branch creation (`git checkout -b`) are fine. End-of-task summaries should describe what changed and is ready for review, not "committed as <hash>".

### 2. Log every non-trivial change in BOTH places

A change worth a `[new]` / `[opt]` / `[fix]` / `[doc]` tag must land in **both** files. They complement each other — never update only one or they will drift:

1. **`doc/tag-log.md` + `doc/tag-log.zh.md`** — developer-facing verbose changelog. Append under the current upcoming-version block, or create a new `## vX.Y.Z — YYYY-MM-DD` section at the top. Use the existing `[new]/[opt]/[fix]/[doc]` category tags. Bilingual files stay in lockstep.

2. **`web/src/release-notes.tsx`** — user-facing in-app `What's new` modal. Add a `ReleaseHighlight` to the matching `ReleaseNote`, or prepend a new entry when bumping the version. Each highlight needs:
   - bilingual `title` and `description` (`{ en, zh }`)
   - a `kind` (`"new" | "improved" | "fixed" | "doc"`)
   - optional `location` (bilingual) — *where* in the UI to find the new thing, the "教学" part of the modal
   - optional `ctaLabel` + `ctaPath` (in-app route) or `ctaHref` (external link)

After the user bumps the version with `npm run release:*`, restarting the proxy makes the new `What's new` modal pop on first admin load for everyone whose `localStorage.lastSeenReleaseVersion` is below the new version.

---
> Source: [7as0nch/mimo2codex](https://github.com/7as0nch/mimo2codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
