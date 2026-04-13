## hiway-social-integration

> Microservice that distributes video clips from Hiway portal to social media platforms via Ayrshare's API.

# Hiway Social Integration

## What This Is
Microservice that distributes video clips from Hiway portal to social media platforms via Ayrshare's API.

## Stack
- Node.js + Express + TypeScript
- PostgreSQL + Prisma ORM
- Jest + Supertest for testing
- Ayrshare API for social media distribution

## Commands
- `npm run dev` — Start dev server
- `npm test` — Run all tests
- `npm run test:unit` — Unit tests only
- `npm run test:integration` — Integration tests only
- `npm run prisma:migrate` — Run DB migrations
- `npm run build` — Build TypeScript

## Workflow
Follow senior-dev.md: TDD approach, write tests first, implement to pass.

## Project Structure
- `src/routes/` — Express route definitions
- `src/controllers/` — Request handlers
- `src/services/` — Business logic + Ayrshare API client
- `src/middleware/` — Auth, validation, error handling
- `src/prisma/` — Prisma schema
- `src/types/` — TypeScript types
- `tests/unit/` — Unit tests
- `tests/integration/` — Integration/route tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HiwayDev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HiwayDev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
