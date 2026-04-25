## hagaflow

> - **Monorepo**: pnpm workspaces

# HagaFlow — Project Conventions

## Stack
- **Monorepo**: pnpm workspaces
- **Frontend**: Next.js 15 (App Router, Tailwind CSS v4, RSC where practical)
- **Backend**: NestJS 11 (modular monolith, Swagger, Prisma, BullMQ)
- **Shared**: `@hagaflow/shared` — Zod schemas, TypeScript types, DTOs
- **Animations**: `@hagaflow/remotion` — Remotion compositions
- **Database**: PostgreSQL 16 via Prisma ORM
- **Queue**: Redis 7 + BullMQ
- **AI**: Abstracted provider interface with RAG + Tools + Skills pipeline

## Commands
```bash
pnpm dev              # Start web + api in parallel
pnpm dev:web          # Start Next.js only
pnpm dev:api          # Start NestJS only
pnpm docker:up        # Start Postgres + Redis
pnpm docker:down      # Stop infrastructure
pnpm db:generate      # Prisma generate
pnpm db:migrate       # Prisma migrate dev
pnpm db:seed          # Run seed script
pnpm db:studio        # Prisma Studio
pnpm lint             # Lint all packages
pnpm type-check       # Type-check all packages
pnpm test             # Run all tests
```

## Architecture Rules
- Backend modules are bounded — no cross-module service imports without explicit exports
- Shared package contains NO Prisma types — only Zod schemas and plain TypeScript interfaces
- Frontend talks to backend through `apiClient` (`apps/web/src/lib/api-client.ts`)
- All API responses use `{ data: T }` wrapper
- AI provider is abstracted behind `AiProvider` interface — never import a specific provider directly
- Background jobs use BullMQ queues — never run heavy tasks in request handlers
- Progressive disclosure pipeline processes stages sequentially — each stage can be reviewed independently

## Naming
- NestJS modules: `<feature>.module.ts`, `<feature>.service.ts`, `<feature>.controller.ts`
- DTOs: `dto/index.ts` inside each module
- Shared schemas: `packages/shared/src/schemas/index.ts`
- Remotion components: PascalCase in `packages/remotion/src/components/`

## Environment
- Copy `.env.example` to `.env` before starting
- Docker Compose for Postgres + Redis: `docker/docker-compose.yml`
- API runs on port 3001, Web on port 3000

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alvaroofernaandez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
