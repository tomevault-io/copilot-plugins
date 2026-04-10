## viddhana

> 1. Use Node 18+; install deps with `npm ci` in root, `backend/`, `web/`, `admin/` before CI-like runs.

# AGENTS

1. Use Node 18+; install deps with `npm ci` in root, `backend/`, `web/`, `admin/` before CI-like runs.
2. Backend dev/prod: `cd backend && npm run dev` for local API, `npm run build && npm start` for production.
3. Backend tests: `cd backend && npm test`; single test: `npx vitest run src/<path>/<file>.test.ts -t "test name"`.
4. Web/admin (Vite React): `cd web|admin && npm run dev|build|test|lint|typecheck` as needed.
5. Block reward bot: `cd block-reward-bot && npm start` (runs `tsx index.ts`).
6. Lint/typecheck/format backend: `cd backend && npm run lint && npm run typecheck && npm run format` (CI mirrors this in `.github/workflows/ci.yml`).
7. Formatting: Prettier (`.prettierrc.json`) with 100-char width, single quotes, semicolons, no trailing commas; run format before large diffs.
8. TypeScript style: prefer `strict`-friendly code, avoid `any`, and give explicit return types for exported functions and shared utilities.
9. Imports: group as Node builtins, external libs, then internal modules; use relative paths within each package (avoid cross-package imports without a clear reason).
10. Naming: `camelCase` for variables/functions, `PascalCase` for types/classes, `SCREAMING_SNAKE_CASE` for env-style constants and config keys.
11. Error handling: never swallow errors; centralize with Express middleware where possible, return sanitized messages, and log full details server-side only.
12. HTTP/Express layout: keep middleware in `backend/src/middleware`, routes in `backend/src/routes`, and keep each route file cohesive and minimal.
13. Tests: colocate Vitest files as `*.test.ts` under `src`; prefer HTTP-level tests via supertest plus focused unit tests for utilities and services.
14. When touching mining reward, withdrawal, or pool-like logic, always review and update related tests and architecture docs under `docs/`.
15. For "pool" / "VIDDHANA pool" work, first read `contracts/pools/`, `contracts/rewards/`, `contracts/oracles/`, `contracts/risk/` and relevant `docs/` to understand current design and risks.
16. For significant pool changes, sketch a brief blueprint (overview, asset flow, reward model, oracle updates, risk/circuit breaker, upgrade path) before coding large refactors.
17. Pool-related tests must at least cover deposit/withdraw happy paths, reward/APY calculation, emergency stop/pause, basic reentrancy attempts, and bad/stale oracle data using the repo's test framework.
18. When editing Solidity/contract-like code (e.g., under `contracts/`), keep changes small and audited-style, target Solidity 0.8.x, prefer OpenZeppelin over custom primitives, and document key invariants/assumptions in comments.
19. Security: treat all external input (HTTP, env, RPC, DB, oracle data) as untrusted; validate/parse minimally but explicitly, and respect existing auth, rate limiting, and circuit breaker mechanisms.
20. Repo hygiene/tooling: avoid large refactors or deletions without explicit user approval, summarize diffs by area (backend/web/admin/contracts/infra), and update this file if Cursor or Copilot instruction files are added.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NQKhaixyz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NQKhaixyz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
