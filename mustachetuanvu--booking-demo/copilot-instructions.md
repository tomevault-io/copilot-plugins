## booking-demo

> This workspace contains two TypeScript applications:

# Repository Guidelines

## Project Structure & Module Organization

This workspace contains two TypeScript applications:

- `backend/`: NestJS API. Feature modules under `backend/src/<feature>/` commonly contain controllers, services, repositories, DTOs, and schemas. End-to-end tests are in `backend/test/`; runtime uploads are under `backend/public/`.
- `frontend/`: Next.js App Router application. Routes are in `frontend/src/app/`, reusable UI in `frontend/src/components/`, and shared helpers in adjacent `src/` directories. Static assets belong in `frontend/public/`.

Run commands from the relevant application directory; there is no root package script. Do not edit generated `dist/`, `.next/`, `coverage/`, or `node_modules/` content.

## Build, Test, and Development Commands

Install dependencies separately; the backend README recommends `npm install --legacy-peer-deps`.

- `cd backend && npm run start:dev`: run the API in watch mode with the test environment.
- `cd backend && npm run build`: compile NestJS into `dist/`.
- `cd backend && npm run lint` / `npm run format`: fix ESLint or Prettier issues.
- `cd backend && npm test`: run Jest unit tests; use `npm run test:e2e` or `npm run test:cov` for integration tests and coverage.
- `cd frontend && npm run dev`: start Next.js with Turbopack.
- `cd frontend && npm run build`: create a production build.
- `cd frontend && npm run lint`: run the configured Next.js ESLint checks.

## Coding Style & Naming Conventions

Use TypeScript throughout. Backend Prettier uses four spaces, single quotes, semicolons, trailing commas, and a 100-character line width. Follow NestJS lowercase feature filenames such as `orders.service.ts`; name DTO files by purpose, for example `create.dto.ts`. Frontend route files use Next.js conventions (`page.tsx`, `layout.tsx`), while React components use PascalCase (`BookingCalendarSection.tsx`). Prefer the `@/*` alias for frontend imports from `src/`.

## Testing Guidelines

Backend unit tests must end in `.spec.ts`; e2e tests belong in `backend/test/` and use Supertest/Jest. Add tests for changed service logic and API behavior. The frontend currently has no automated test script, so validate changes with lint, a production build, and manual checks of affected routes.

## Commit & Pull Request Guidelines

Backend history uses short imperative subjects such as `Add Telegram notifications...`. Prefer a specific behavior summary over `Update <file>`. Keep commits scoped to one application. Pull requests should describe the change, list verification commands, note configuration impacts, link the issue, and include screenshots for frontend changes.

## Security & Configuration

Never commit secrets or production credentials. Treat `.env`, `.env.test`, and `.env.product` as local/deployment configuration. Document required variable names without including values.

---
> Source: [MustacheTuanVu/booking-demo](https://github.com/MustacheTuanVu/booking-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
