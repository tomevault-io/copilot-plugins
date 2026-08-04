## obedi

> **IMPORTANT: Claude must NEVER start dev servers automatically.**

# Obedi - Lunch Management App

## Development

### Starting Dev Servers

**IMPORTANT: Claude must NEVER start dev servers automatically.**

Only the user can start dev servers manually using:

```bash
pnpm dev
```

This runs both backend (port 3001) and frontend (port 3002) in parallel.

### URLs
- Frontend: http://localhost:3002
- Backend API: http://localhost:3001/api/v1
- Swagger: http://localhost:3001/api/docs

### Test Accounts
- demo@example.com / password123
- chef@example.com / password123
- foodie@example.com / password123

## Stack

- **Frontend**: Vue 3 + TypeScript + Vite + Tailwind + FSD
- **Backend**: NestJS + Prisma + PostgreSQL
- **Storage**: Supabase S3

## Commands

```bash
pnpm dev          # Start both servers
pnpm dev:front    # Frontend only
pnpm dev:back     # Backend only
pnpm build        # Build frontend
pnpm lint         # Lint all
```

---
> Source: [billyhargroveofficial/obedi](https://github.com/billyhargroveofficial/obedi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
