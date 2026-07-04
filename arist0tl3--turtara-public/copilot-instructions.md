## turtara-public

> This repository is a full-stack engineering manager tool (`people-manager`) with:

# AGENTS.md

## Purpose
This repository is a full-stack engineering manager tool (`people-manager`) with:
- `client/`: React + Vite + TypeScript + Apollo Client + MUI
- `server/`: Node + TypeScript + Apollo Server + MongoDB + Mongoose

Use this file as the default working agreement for contributors and coding agents.

## Repo Map
- `README.md`: currently minimal, needs expansion
- `client/src/App.tsx`: route map and auth-gated app shell
- `client/src/generated.ts`: GraphQL generated types/hooks used by UI
- `server/src/server/index.ts`: server bootstrap, Mongo connect, Apollo + Express context
- `server/src/schema/index.ts`: GraphQL schema composition from domain modules
- `server/src/models/**`: domain models, resolvers, and GraphQL typeDefs

## Local Dev
Prereqs:
- Node.js `23.x` (server `engines.node` requirement)
- MongoDB available locally or via `MONGO_URI`

Suggested startup flow:
1. `cd server && npm install`
2. `cp .env.sample .env` and set values (`MONGO_URI`, `JWT_SECRET`, API keys as needed)
3. `npm run start-dev`
4. `cd ../client && npm install`
5. Ensure `client/.env.development` points to server GraphQL endpoint
6. `npm run dev`

Codegen:
- `cd client && npm run generate`
- This updates `client/src/generated.ts` and copies generated types to `server/src/generated.ts`

## Engineering Guardrails
- Do not log secrets, auth headers, or raw tokens.
- Avoid fallback secrets in production paths (for example JWT secrets).
- Keep GraphQL operations and generated types in sync after schema changes.
- Prefer small PRs with one clear purpose.
- Add or update tests when changing behavior (unit/integration where practical).
- Keep docs current when scripts, env vars, or architecture change.

## Current High-Risk Areas
- Auth/security hardening (token logging and default JWT secret handling).
- No active test suite in either `client/` or `server/`.
- Root README does not document setup or architecture.
- AI/Jira integrations need clear env and failure-path documentation.

## PR Checklist
- [ ] Branch is scoped to one task
- [ ] Lint/build/typecheck run for touched package(s)
- [ ] GraphQL codegen run if operations/schema changed
- [ ] No sensitive logs introduced
- [ ] Docs updated (`README.md`, `TODOS.md`, or module docs) when relevant

---
> Source: [arist0tl3/turtara-public](https://github.com/arist0tl3/turtara-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
