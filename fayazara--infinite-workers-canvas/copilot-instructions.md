## infinite-workers-canvas

> Infinite canvas where each node is a live AI-generated Cloudflare Worker sandbox. Full-stack single-package app: React frontend + Cloudflare Worker backend, unified via `@cloudflare/vite-plugin`.

# AGENTS.md

## Project

Infinite canvas where each node is a live AI-generated Cloudflare Worker sandbox. Full-stack single-package app: React frontend + Cloudflare Worker backend, unified via `@cloudflare/vite-plugin`.

## Commands

- `pnpm dev` — Vite dev server with Workers emulation
- `pnpm build` — production build
- `pnpm deploy` — build + `wrangler deploy`
- `pnpm types` — regenerate `env.d.ts` from `wrangler.jsonc` bindings (run after changing bindings)

No test, lint, or typecheck scripts are configured.

## Architecture

- **Server entry**: `src/server.ts` — Worker fetch handler, API routes (`/api/generate`, `/api/run`)
- **Client entry**: `src/client.tsx` → `src/app.tsx` — React Flow canvas
- **Durable Object**: `LogSession` in `src/logging.ts` — captures logs from dynamically created workers
- **Worker Loader**: `DynamicWorkerTail` in `src/logging.ts` — tail handler for dynamic workers
- **Types**: `src/types.ts` (frontend), `env.d.ts` (auto-generated Worker env bindings)

## Key patterns

- AI models are routed two ways: **AI Gateway** (OpenAI-compatible, needs `CF_AIG_TOKEN` secret) vs **Workers AI binding** (`env.AI`). See `GATEWAY_MODELS` / `WORKERS_AI_MODELS` arrays in `server.ts`.
- Dynamic workers are created at runtime via `@cloudflare/worker-bundler` (`createWorker`) through the `LOADER` binding.
- Node-to-app communication uses custom DOM events (`worker-node:submit`, `worker-node:delete`, `worker-node:rerun`) dispatched from `WorkerNode.tsx` and listened to in `app.tsx`.
- Styling is almost entirely custom CSS (`src/styles.css` with CSS variables), not Tailwind utility classes — despite Tailwind v4 being installed via `@tailwindcss/vite`.

## Environment

- `CF_AIG_TOKEN` must be set in `.dev.vars` for local dev (AI Gateway models). File is gitignored.
- Workers AI binding (`AI`) is configured in `wrangler.jsonc` with `"remote": true` — requires `--remote` or a deployed preview for local dev.

## TypeScript

Strict mode with `noUnusedLocals` and `noUnusedParameters` — unused imports/params will cause build errors. `@ts-expect-error` is used in `server.ts` for dynamic Workers AI model names not in local types.

## Package manager

**pnpm** (lockfile: `pnpm-lock.yaml`).

---
> Source: [fayazara/infinite-workers-canvas](https://github.com/fayazara/infinite-workers-canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
