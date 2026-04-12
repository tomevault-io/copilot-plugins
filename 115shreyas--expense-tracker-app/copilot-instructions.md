## expense-tracker-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

### Backend (`backend/`)
```bash
npm run dev                  # Local dev server (port 3001, NODE_ENV=des)
npm run dev:hosted           # Hosted dev server (NODE_ENV=dev)
npm run build                # TypeScript compile
npm run start                # Run compiled JS
npm run prisma:generate      # Generate Prisma client
npm run prisma:migrate       # Run migrations (local/des)
npm run prisma:migrate:dev   # Deploy migrations (dev env)
npm run prisma:migrate:prod  # Deploy migrations (prod env)
npm run prisma:seed           # Seed categories + rules (des env)
```

### Frontend (`frontend/`)
```bash
npm run dev                  # Local dev server (port 5173, mode=des)
npm run dev:hosted           # Hosted dev server (mode=dev)
npm run build                # Production build (mode=prod)
npm run build:dev            # Dev environment build
npm run lint                 # ESLint
```

Frontend dev server proxies `/api` requests to `http://localhost:3001`.

## Architecture

### Multi-Environment Setup
Three environments: **DES** (local), **DEV** (hosted dev), **PROD** (production). Backend uses `cross-env NODE_ENV=<env>` with `dotenv` loading `.env.<env>`. Frontend uses Vite's `--mode <env>` which natively loads `.env.<env>`. Prisma CLI also loads env-specific files via `prisma.config.ts`.

### Backend Structure
- **Express v5** server on port 3001 with JWT Bearer auth middleware
- **Prisma v7** with `@prisma/adapter-pg` driver adapter (PostgreSQL)
- Routes: `auth`, `transactions`, `categories`, `analytics`, `upload`
- All routes except auth and health are protected via `auth` middleware that sets `req.userId`
- Bank statement parsers implement `BankParser` interface in `parsers/` (currently HDFC XLS)
- Auto-categorization service matches transaction descriptions against keyword rules with confidence scores
- `learnFromUserEdit()` creates new categorization rules when users manually recategorize

### Frontend Structure
- **React 19** with react-router-dom v7, Vite 7, Tailwind CSS v4, Recharts v3
- `AuthContext` manages user/token state, stores token in localStorage
- Axios instance in `lib/api.ts` adds Bearer token via interceptor, handles 401 → logout
- Pages: Dashboard (analytics charts), Transactions (filterable table), Upload (drag-drop), Login

### Transaction Upload Flow
File → `HdfcParser.parse()` → deduplicate by SHA256 hash → `categorizeTransaction()` → `inferPaymentMode()` → batch insert

### Database Models
User → Transaction (with hash-based dedup) → Category ← CategorizationRule (keyword + confidence)

## Key Technical Constraints

- **Prisma v7**: Import from `generated/prisma/client`. Constructor requires `PrismaPg` adapter — `new PrismaClient({ adapter })`. The `datasourceUrl` option is removed. `groupBy` calls need `as any` cast.
- **Express v5**: `req.params` and `req.query` return `string | string[]`, cast with `as string`.
- **Tailwind v4**: Uses `@tailwindcss/vite` plugin and `@import "tailwindcss"` in CSS. No config file.
- **Recharts**: Tooltip `formatter` value param is `number | undefined`, cast as `number`.

## Adding a New Bank Parser
1. Create `backend/src/parsers/<bank>-parser.ts` implementing `BankParser` interface from `base-parser.ts`
2. Return `ParsedTransaction[]` with SHA256 hash for dedup
3. Add upload route in `backend/src/routes/upload.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/115SHREYAS)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/115SHREYAS)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
