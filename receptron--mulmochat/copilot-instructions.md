## mulmochat

> - Voice-driven multi-modal chat client that connects to OpenAI's Realtime API by WebRTC and renders tool outputs in the main canvas.

# MulmoChat Agent Guide

## Overview
- Voice-driven multi-modal chat client that connects to OpenAI's Realtime API by WebRTC and renders tool outputs in the main canvas.
- Sidebar lists tool invocations (images, browsing, search, games) and lets the user start/stop voice chat or send text messages.
- Tool executions are coordinated through `src/tools/type.ts`; results flow back into `App.vue` to update the UI and inform the model.

## Repository Layout
- `src/` — Vue 3 + TypeScript client. Entry: `src/main.ts`; root component: `src/App.vue`; sidebar UI: `src/components/Sidebar.vue`.
- `src/tools/` — Tool plugins (`generateImage`, `editImage`, `browse`, `mulmocast`, `presentMap`, `exaSearch`, `playOthello`) plus result views under `src/tools/views/` and previews under `src/tools/previews/`.
- `server/` — Express server in TypeScript. Entry: `server/index.ts`; REST endpoints live in `server/routes/api.ts`; shared types in `server/types.ts`.
- Builds and configs: `vite.config.ts`, `tailwind.config.cjs`, `postcss.config.cjs`, `tsconfig*.json`, `eslint.config.mjs`.
- `dist/` — Vite build output (client). Do not edit.

## Client Architecture
- `App.vue` manages WebRTC setup, receives the `/api/start` payload, and registers tools returned by `pluginTools(startResponse)`.
- Outgoing tool calls stream arguments through the data channel; `processToolCall` dispatches to `pluginExecute` with any cached image context.
- Views in `src/tools/views/*.vue` render the active tool result (images, browse summaries, Exa hits, MulmoCast HTML, map, Othello board) based on the `ToolResult` payload.
- `Sidebar.vue` shows the execution history, exposes audio playback for remote audio, and provides a text input that sends conversation items via the same data channel.
- Tool enablement is dynamic: `presentMap` requires a Google Maps key; `exaSearch` is available only when `EXA_API_KEY` is configured; all others are always enabled.

## Server APIs
- `/api/start` — Exchanges the long-lived OpenAI API key for a short-lived Realtime client secret; returns feature flags (`hasExaApiKey`) and the optional Google Maps key.
- `/api/generate-image` — Uses Google Gemini `gemini-2.5-flash-image-preview` to create or edit images (edits receive the previous image bytes).
- `/api/browse` — Delegates to `mulmocast.puppeteerCrawlerAgent` for structured web-page captures.
- `/api/exa-search` — Wraps `exa-js` `search`/`searchAndContents` depending on payload flags.
- `/api/twitter-embed` — Proxy around Twitter/X oEmbed so the client can safely inject embeds.
- Health/config helpers: `/api/health`, `/api/config` for service diagnostics.

## Scripts
- `yarn dev` — Run the server (`tsx server/index.ts`) and Vite dev server concurrently.
- `yarn dev:server` / `yarn dev:client` — Start either side in isolation (`yarn server` is an alias for the server script).
- `yarn build` — Type-check (`vue-tsc -b`), build the client, and compile the server (`yarn build:server`).
- `yarn preview` — Serve the Vite production build.
- `yarn start` — Run the compiled server (`server/dist/index.js`).
- `yarn lint` / `yarn format` — ESLint across `src` and `server`; Prettier for `{src,server}/**/*.{ts,json,yaml,vue}`.

## Environment Variables
- Required: `OPENAI_API_KEY` (Realtime client secret exchange), `GEMINI_API_KEY` (image generation).
- Optional: `GOOGLE_MAP_API_KEY` (enables the map tool), `EXA_API_KEY` (enables Exa search). Optional keys are surfaced to the client via `/api/start`; `.env` is ignored by git.
- Missing optional keys log warnings and gracefully disable the dependent tool.

## Development Guidelines
- TypeScript strict mode, Vue SFCs with `<script setup lang="ts">`, 2-space indentation, Unix line endings, semicolons required.
- Prefer the existing structured logging pattern (minimal `console.*` already in place); avoid introducing noisy logs in committed code.
- Keep tool definitions in `src/tools` declarative and update both the plugin module and any corresponding view/preview when adding new capabilities.
- Client/server communication assumes JSON payloads under 500 MB (see Express body parser limits).

## Testing
- No automated tests today. If you add tests, use Vitest for both client and server (`*.spec.ts` adjacent to source) and cover new tool behaviors or API routes.

## Release Checklist
- Run `yarn lint` and `yarn format` before committing.
- Verify `.env` contains the needed keys for any new feature paths.
- For UI/API changes, capture screenshots or logs when preparing a PR.
- Avoid shipping the temporary `/api/start` key-exchange flow to production without hardening.

---
> Source: [receptron/MulmoChat](https://github.com/receptron/MulmoChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
