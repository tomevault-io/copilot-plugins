## ai-planner

> - `backend/`: NestJS 11 API with Prisma, AI pipeline (`src/modules/ai`), auth, analytics, and services. Tests live under `backend/test/`. Assets such as Prisma schema and migrations sit in `backend/prisma/`.

# Repository Guidelines

## Project Structure & Module Organization
- `backend/`: NestJS 11 API with Prisma, AI pipeline (`src/modules/ai`), auth, analytics, and services. Tests live under `backend/test/`. Assets such as Prisma schema and migrations sit in `backend/prisma/`.
- `frontend/`: Next.js 16 App Router app. UI components are under `src/components/`, hooks in `src/hooks/`, and library helpers (API client, auth utilities, shared types) in `src/lib/`.
- `.github/workflows/deploy.yml`: CI/CD flow building Docker images and redeploying to EC2. Infrastructure compose files live on the server under `~/infra/`.

## Build, Test, and Development Commands
- Backend dev server: `cd backend && npm run start:dev` (watch mode on port 4000).
- Backend lint/type-check: `npm run lint` and `npx prisma generate` before commits to stay in sync.
- Backend tests: `npm run test` (unit) and `npm run test:e2e`.
- Frontend dev server: `cd frontend && npm run dev` (Next.js on port 3000).
- Frontend lint: `npm run lint` (runs `eslint`).

## Coding Style & Naming Conventions
- TypeScript across repo; 2-space indentation. Prefer descriptive camelCase for variables/functions, PascalCase for classes/components.
- Use ESLint (`backend` custom config, `frontend` Next.js config) and Prettier (backend) before pushing. Avoid non-ASCII characters unless UI copy requires Korean text.
- Environment variables go in `.env` (backend) and `.env.local` (frontend); reference `.env.production.template` for required keys.

## Testing Guidelines
- Backend uses Jest; create spec files alongside source with `.spec.ts` suffix. Keep tests deterministic; mock external services (OpenRouter, Naver) when possible.
- Run `npm run test:cov` if coverage checks are needed for CI, and ensure Prisma migrations are up to date before running e2e tests.

## Commit & Pull Request Guidelines
- Follow conventional-style prefixes seen in history (`feat:`, `fix:`, `docs:`, `chore:`). Keep subject under ~60 chars and include concise body when context is needed.
- PRs should describe the change, note testing performed, and link to issues/tasks. Attach screenshots/GIFs for UI changes and mention any env or migration impacts.

## Security & Configuration Tips
- Do not commit secrets. Keep `NAVER_*`, `OPENROUTER_API_KEY`, and OAuth keys in local `.env` or GitHub Secrets only.
- When debugging place search, ensure `NAVER_SEARCH_CLIENT_ID/SECRET` are set; missing keys fall back to mock data (which biases results to Gangnam).

---
> Source: [jaylenyu/ai-planner](https://github.com/jaylenyu/ai-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
