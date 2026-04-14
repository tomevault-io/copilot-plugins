## monorepo-template

> - **Monorepo**: Root `package.json` manages `frontend` and `backend`.

# Project Architecture

## Stack
- **Monorepo**: Root `package.json` manages `frontend` and `backend`.
- **Frontend**: React 18 (Vite), TypeScript, TailwindCSS v4, Shadcn/UI.
- **Backend**: Node.js, Express, TypeScript, PostgreSQL, Prisma.
- **Package Manager**: npm.

## Frontend Structure (`/frontend`)
- **State**: `@tanstack/react-query` (Data), React Context (Auth).
- **Routing**: `react-router-dom`.
- **API Layer**: `axios` instance in `src/lib/axios.ts`. Endpoints in `src/api/`.
- **Config**: Environment variables validated via Zod in `src/config/env.ts`.
- **UI**: Shadcn/UI in `src/components/ui`.

## Backend Structure (`/backend`)
- **Pattern**: Controller-Service-Repository (via Prisma).
- **Validation**: Request validation middleware using Zod.
- **Error Handling**: centralized `errorHandler` middleware + `AppError` class.
- **Database**: Prisma Client singleton (`src/lib/prisma.ts`) to prevent connection exhaustion in dev.
- **Config**: Environment variables validated via Zod in `src/config/env.ts`.

## Development
- **Start**: `npm run dev` (Concurrent execution).
- **Check**: `npm run typecheck`.
- **Ports**: Defined in root `.env` (Default: API 20001, Web 20002).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0xForkh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/0xForkh)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
