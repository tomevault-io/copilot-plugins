## upllyft

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Upllyft is a multi-app monorepo for a neurodivergent community platform, built with Turborepo + pnpm workspaces. **Mira** (Mindful Intelligent Response Assistant) is the platform's primary touchpoint — an empathetic AI guide that helps parents navigate their child's developmental journey.

## Monorepo Structure

```
upllyft/
├── apps/
│   ├── api/                    # NestJS 11 backend (port 3001)
│   ├── web-main/               # Main hub: dashboard, feed, profile, settings, admin (port 3000)
│   ├── web-community/          # Community: posts, Q&A, events, crisis, providers (port 3002)
│   ├── web-screening/          # Screening: developmental assessments, reports (port 3003)
│   ├── web-booking/            # Booking: therapist marketplace, sessions, Stripe (port 3004)
│   ├── web-resources/          # Learning: AI worksheets, assignments, community library (port 3005)
│   ├── web-cases/              # Cases: therapist case management, IEPs, milestones (port 3006)
│   ├── web-admin/              # Admin panel (port 3007)
│   └── mobile/                 # Expo 54 React Native app (iOS + Android)
├── packages/
│   ├── ui/                     # @upllyft/ui — shared React components (web only, includes MiraNudge)
│   ├── api-client/             # @upllyft/api-client — axios client, token refresh, AuthProvider
│   ├── types/                  # @upllyft/types — shared TypeScript types
│   └── config/                 # @upllyft/config — shared tsconfig, eslint, tailwind
└── turbo.json
```

**Shared packages are TypeScript-first** — they export raw `.ts` from `src/index.ts` with no build step. Consuming apps transpile them via `transpilePackages` in `next.config.ts`.

## Common Commands

```bash
pnpm install                # Install all dependencies
pnpm build                  # Build all packages and apps
pnpm dev                    # Start all apps in dev mode (parallel)
pnpm lint                   # Lint all packages and apps
pnpm type-check             # Type-check all packages and apps
pnpm clean                  # Clean all build artifacts (.next, dist, .turbo)
```

### Run specific apps:

```bash
# Recommended: API + one frontend
pnpm --filter @upllyft/api --filter @upllyft/web-main dev

# Single app
pnpm --filter @upllyft/web-main dev
pnpm --filter @upllyft/api dev

# Mobile
cd apps/mobile && pnpm start
```

### API commands:

```bash
# Database
pnpm --filter @upllyft/api prisma:generate   # Generate Prisma client
pnpm --filter @upllyft/api prisma:migrate    # Run migrations
pnpm --filter @upllyft/api prisma:studio     # Open Prisma Studio

# Seeding
pnpm --filter @upllyft/api db:seed           # Base data only
pnpm --filter @upllyft/api db:seed:all       # All seeds (base + events + Q&A + crisis)
pnpm --filter @upllyft/api db:seed:cases     # Case management data (Dr. Meena cases)
pnpm --filter @upllyft/api db:seed:events    # Questions & events
pnpm --filter @upllyft/api db:seed:qa        # Questions & answers
pnpm --filter @upllyft/api db:seed:crisis    # Crisis resources

# Testing (Jest)
pnpm --filter @upllyft/api test              # Run unit tests
pnpm --filter @upllyft/api test:watch        # Watch mode
pnpm --filter @upllyft/api test:cov          # With coverage
pnpm --filter @upllyft/api test:e2e          # End-to-end tests
pnpm --filter @upllyft/api test:debug        # Debug mode
```

### First-time setup:

```bash
pnpm install
cp apps/api/.env.example apps/api/.env       # Then edit with your DB URL, JWT secrets, API keys
pnpm --filter @upllyft/api prisma:generate
pnpm --filter @upllyft/api prisma:migrate
pnpm --filter @upllyft/api db:seed:all       # Optional
```

Prerequisites: Node.js 20+, pnpm 10+, PostgreSQL with pgvector extension, Redis (optional).

## Architecture

### Backend (NestJS 11 + Fastify + Prisma)
- Modular architecture: each feature is a NestJS module in `apps/api/src/<feature>/`
- **Prisma ORM** with PostgreSQL + pgvector for embeddings (schema: `apps/api/prisma/schema.prisma`, ~3400 lines)
- **Auth**: JWT + Passport (local, Google OAuth strategies); JWT in response body, refresh via `POST /auth/refresh`
- **Real-time**: Socket.IO WebSockets
- **AI**: OpenAI (GPT-5, DALL-E 3, embeddings) + Anthropic Claude (worksheets)
- **Payments**: Stripe Connect with escrow
- **Storage**: Supabase (PDFs, images)
- **Email**: MailerSend / SendGrid / AWS SES (configurable)
- **Push**: Firebase Cloud Messaging
- Swagger docs at `/api/docs`
- Build allocates 4GB: `node --max-old-space-size=4096`

### Frontend (Next.js 16 + React 19)
- All web apps use the same stack: Next.js App Router + Tailwind v4 + Radix UI
- **State**: TanStack Query for server state, React Hook Form + Zod for forms
- **API proxy**: Each app's `next.config.ts` rewrites `/api/:path*` → `localhost:3001/api/:path*`
- **Auth**: `<AuthProvider baseURL="/api">` from @upllyft/api-client wraps each app
- **Cross-app auth**: Cookies on `localhost` are shared across all ports in dev. In production, cookies scoped to `.safehaven-upllyft` domain.

### Mobile (Expo 54 + React Native 0.81)
- Expo Router for file-based navigation
- Own axios API client with token refresh (does NOT use @upllyft/api-client)
- expo-secure-store for token storage
- Role-based tab navigation (parent vs therapist)

### Mira — AI Conversational Guide

Mira lives in web-main and reaches into every app via contextual nudges.

- **Backend**: `apps/api/src/mira/` — streaming SSE chat (`POST /mira/chat-stream`), conversation CRUD
- **Frontend**: `apps/web-main/src/components/mira/` — `MiraProvider` + `useMira()` hook, slide-over panel, FAB
- **Cross-app nudges**: `<MiraNudge>` component from `@upllyft/ui` renders in other apps, redirects to web-main with `?openMira=true&message=...` URL params
- **Onboarding handoff**: After onboarding step 5, stores `mira_onboarding_handoff` in localStorage; `MiraProvider` auto-opens Mira on dashboard mount
- **Mira.png** avatar must exist in each app's `public/` folder
- Nudges are role-guarded to `user.role === 'USER'` (parents only) via `MiraNudgeForParent` helper

### Key Patterns
- **Roles**: USER (parents), THERAPIST, EDUCATOR, ORGANIZATION, ADMIN, MODERATOR
- **Brand**: Teal gradient (teal-400 to teal-600), rounded corners, subtle shadows
- **Environment**: `turbo.json` globalEnv includes `NEXT_PUBLIC_API_URL` and `NEXT_PUBLIC_APP_*_URL` for each app

### @upllyft/ui Component API

These components differ from common shadcn/ui conventions — read the source before using:

- **Avatar**: `<Avatar src={url} name={name} size="md" />` (no sub-components)
- **Card**: Simple div wrapper (no CardHeader/CardTitle/CardContent)
- **Badge**: `color` prop (green, blue, yellow, red, gray, purple) — not `variant`
- **Button**: variants: primary, secondary, outline, ghost (no "destructive")
- **MiraNudge**: `<MiraNudge nudgeId="..." message="..." chipText="..." mainAppUrl={APP_URLS.main} />` — use `onAskMira` callback for in-app (web-main) usage instead of `mainAppUrl`. Supports `childName` for personalized `[child name]` replacement.

## Troubleshooting

### Prisma migration fails (pgvector)
Shadow database creation fails due to pgvector extension. Create migrations manually:
```bash
pnpm --filter @upllyft/api prisma:migrate --create-only
# Edit the SQL, then apply
```

### Mobile can't reach API
Set `EXPO_PUBLIC_API_URL` in `apps/mobile/.env` to your machine's local IP (not localhost).

### Cross-app auth issues in development
Clear cookies for `localhost` in your browser. Auth tokens are stored in both cookies and localStorage.

## Adding a New Web App

1. Create `apps/<app-name>/` with package.json named `@upllyft/<app-name>`
2. Add `@upllyft/ui`, `@upllyft/api-client`, `@upllyft/types` as `workspace:*` deps
3. Copy from existing app: `next.config.ts`, `tsconfig.json`, `postcss.config.mjs`, `globals.css`
4. Set `transpilePackages: ["@upllyft/ui", "@upllyft/api-client", "@upllyft/types"]` in `next.config.ts`
5. Wrap app in `<AuthProvider baseURL="/api">`
6. Create `vercel.json` for deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pocProjectWorkspace) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
