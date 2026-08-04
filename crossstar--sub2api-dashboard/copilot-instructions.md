## sub2api-dashboard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Toolchain and commands

Use Node.js 22 and pnpm 10. Install dependencies from the repository root with `pnpm install`.

- `pnpm lint` — lint the entire workspace with the flat ESLint configuration.
- `pnpm typecheck` — type-check all workspace projects in dependency order.
- `pnpm test` — run all Vitest projects once.
- `pnpm build` — build all workspace projects in dependency order.
- `pnpm format:check` — check Prettier formatting; `pnpm format` applies it.
- `pnpm --filter @sub2api/contracts test` — run only the contracts tests.
- `pnpm --filter @sub2api/contracts exec vitest run test/contracts.test.ts -t "accepts numeric"` — run one contract test by name.

## Architecture

This is a pnpm monorepo. `apps/server` is the Fastify BFF and `apps/web` is a Vue 3 + Vite SPA using Pinia, Vue Router, and Tailwind. Both consume `@sub2api/contracts` through the `workspace:*` protocol and TypeScript project references, so contracts build before either app. Run the UI with `pnpm --filter @sub2api/web dev`; Vite proxies `/api` and `/health` to the BFF on port 3000. The production web build is emitted to `apps/web/dist` for the server's `WEB_DIST` static hosting.

`packages/contracts` is the shared runtime/type boundary. Its public API is exported from `src/index.ts` and built as ESM declarations and JavaScript in `dist`. It uses Zod schemas as the source of truth and derives TypeScript types with `z.infer`. Upstream response schemas deliberately accept inconsistent numeric/string response codes and use passthrough objects so unknown upstream fields survive parsing. Dashboard-facing schemas normalize the stable auth, groups, snapshot, and error shapes. The compliance error is a specialized dashboard error fixed to HTTP 423 and `COMPLIANCE_LOCKED`.

`apps/server` now provides the task #9 Fastify BFF. Its only upstream admin operations are group-filtered account pagination and account usage with `source=passive`; do not add a generic proxy, active usage, `force`, probes, or billing calls. Authenticated browser sessions survive browser close and dashboard restart through a dedicated encrypted Redis vault backed by a persistent AOF volume, with 7-day idle and 30-day absolute expiry. The anonymous/2FA cookie remains a browser-session cookie with a 10-minute lifetime. Redis outages fail closed with HTTP 503 without clearing the browser cookie. The cookie contains only an encrypted session ID; its key is separate from the vault keyring. Redis records/AOF contain sensitive AES-256-GCM ciphertext. Retain old vault keys until sessions expire. Caches, single-flight coordination, and rate limits remain process-local, so a single dashboard instance is still required. Losing the Redis volume or keys, expiry, logout, or upstream invalidation requires login; the initial migration requires one login. The upstream contract baseline is `Wei-Shaw/sub2api` commit `c2c19a7cbe8486ebb5b56834d1a6e07b3f12cffc` (0.1.159).

Contract regression tests live under `packages/contracts/test`; fixtures are synthetic and use redacted placeholder credentials. Root `vitest.config.ts` discovers per-package/project Vitest configurations through `test.projects`.

The strict compiler defaults are centralized in `tsconfig.base.json`. Each buildable workspace has a composite project configuration, while `tsconfig.eslint.json` includes tests and tool configuration files for type-aware linting.

---
> Source: [CrossStar/sub2api-dashboard](https://github.com/CrossStar/sub2api-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
