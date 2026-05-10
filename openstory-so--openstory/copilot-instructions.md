## project-context

> Core project context and architecture for OpenStory AI video platform


# OpenStory Project Context

You are working on **OpenStory**, an AI-powered cinematic content creation platform that transforms scripts into consistent, styled video productions.

## Project Overview

OpenStory democratizes cinematic content creation through:

- **Script-to-storyboard AI**: Automatic scene breakdown and frame generation
- **Style Stacks**: JSON-based presets that ensure consistent artistic vision across AI models
- **Multi-model support**: Integration with 19+ AI models (Fal.ai, Runway, Kling, etc.)
- **Character consistency**: LoRA model integration for consistent characters
- **Motion generation**: Image-to-video with VFX capabilities

## Project Architecture

Separated frontend/backend monorepo architecture:

```
openstory/
├── apps/
│   ├── backend/                # Elysia API server
│   │   ├── src/
│   │   │   ├── index.ts       # Elysia app entry point
│   │   │   ├── routes/        # Elysia route handlers
│   │   │   │   ├── sequences.ts
│   │   │   │   ├── frames.ts
│   │   │   │   ├── styles.ts
│   │   │   │   └── jobs.ts
│   │   │   ├── services/      # Business logic layer
│   │   │   │   ├── frame-generator.ts
│   │   │   │   ├── script-analyzer.ts
│   │   │   │   └── style-stack.ts
│   │   │   ├── db/            # Drizzle ORM setup & queries
│   │   │   │   ├── index.ts   # Database client
│   │   │   │   ├── schema/    # Drizzle schema definitions
│   │   │   │   └── queries/   # Reusable query functions
│   │   │   ├── workflows/     # Temporal workflows
│   │   │   │   ├── client.ts  # Temporal client
│   │   │   │   ├── frame-generation.ts
│   │   │   │   ├── motion-generation.ts
│   │   │   │   └── activities/
│   │   │   ├── plugins/       # Elysia plugins
│   │   │   │   ├── auth.ts    # BetterAuth integration
│   │   │   │   ├── cors.ts
│   │   │   │   └── logger.ts
│   │   │   ├── lib/           # Backend utilities
│   │   │   │   ├── ai/        # AI client wrappers
│   │   │   │   ├── storage/   # S3-compatible storage
│   │   │   │   └── errors.ts
│   │   │   └── migrations/    # Drizzle migrations
│   │   └── package.json
│   └── frontend/              # Next.js App Router (UI only)
│       ├── src/
│       │   ├── app/           # Next.js pages & layouts
│       │   │   ├── sequences/ # Sequence pages
│       │   │   ├── (auth)/    # Auth pages
│       │   │   └── layout.tsx
│       │   ├── components/
│       │   │   ├── sequence/  # Storyboard, frame editor
│       │   │   ├── ui/        # shadcn/ui components
│       │   │   └── layout/    # App layout
│       │   ├── hooks/         # TanStack Query hooks (API calls)
│       │   │   ├── use-frames.ts
│       │   │   ├── use-sequences.ts
│       │   │   └── use-user.ts
│       │   └── lib/           # Client-side utilities
│       │       ├── api-client.ts
│       │       └── auth-client.ts
│       └── package.json
├── packages/                  # Shared code
│   ├── database/              # Shared DB schema (future)
│   ├── types/                 # Shared TypeScript types
│   └── validation/            # Shared Zod schemas
└── docs/                      # Architecture & PRD docs
```

## Technology Stack

### Core Framework

- **Frontend**: Next.js 15.5 (App Router), React 19, TypeScript 5
- **Runtime**: Bun (package manager + test runner)
- **Build**: Turbopack (Next.js default)

### UI & Styling

- **UI**: shadcn/ui components, Radix UI primitives
- **Styling**: Tailwind CSS 4
- **Icons**: Lucide React
- **Drag & Drop**: dnd-kit

### Backend & Infrastructure

- **API Server**: Elysia (Bun-native web framework)
- **Database**: Supabase PostgreSQL via Drizzle ORM (type-safe queries, schema-first)
- **Validation**: ArkType (runtime type validation for Elysia routes)
- **Auth**: BetterAuth (anonymous + email/password + OAuth) - NOT Supabase Auth
- **Storage**: Supabase Storage (generated images/videos)
- **Real-time**: WebSockets via Elysia (generation status updates)
- **Workflow Engine**: Temporal (replaces QStash for async orchestration)
- **Caching**: Upstash Redis (session storage, rate limiting)

### State Management

- **Server State**: TanStack Query v5
- **Query Pattern**: Centralized query key factories
- **Caching**: Optimistic updates, automatic invalidation

### AI & Generation

- **Primary Provider**: Fal.ai SDK (@fal-ai/client)
- **Image Models**: flux-pro, imagen4, flux-krea-lora
- **Video Models**: veo3, kling-video, minimax-hailuo, wan-pro
- **Script Analysis**: OpenAI GPT-4 / Anthropic Claude
- **Validation**: ArkType schemas

### Development Tools

- **Testing**: Bun test (Jest-compatible)
- **Linting**: Oxclint (fast, replaces ESLint + Prettier)
- **Git Hooks**: Lefthook
- **Type Safety**: Drizzle Kit generates types from schema
- **API Documentation**: Elysia Swagger plugin (auto-generated)

## Core Features

### 1. Script-to-Storyboard Pipeline

1. User pastes script
2. AI analyzes and breaks into scenes/frames
3. Generates frame descriptions
4. Creates editable storyboard

### 2. Frame Generation

- Per-frame model selection
- Style Stack application for consistency
- Character LoRA injection
- Reference image support
- Parallel generation via Temporal workflows

### 3. Motion Generation

- Image-to-video conversion
- Multiple model options (veo3, kling, wan)
- VFX LoRA support (wan-pro)
- Duration/motion controls

### 4. Style Stacks

- JSON-based style presets
- Model-agnostic adaptation
- Director templates (Tarantino, Nolan, etc.)
- Visual reference upload

### 5. Anonymous-First Flow

1. Users create without signup
2. Generate frames with temporary session
3. Prompted to save (Email login)
4. Session transfers to authenticated user

## Data Model

### Core Entities

- **teams**: Team ownership of all creative assets
- **users**: Team members (anonymous or authenticated)
- **sequences**: Video projects (script → storyboard → video)
- **frames**: Individual shots with thumbnails and motion
- **styles**: Reusable Style Stack presets
- **fal_requests**: API call tracking for Fal.ai

### Key Relationships

- Team → owns Styles, Sequences
- Sequence → has many Frames, uses one Style
- Frame → references Characters, VFX, Audio

## Architecture Principles

### Supabase Usage Clarification

**IMPORTANT:** OpenStory uses Supabase for:

- ✅ **PostgreSQL Database** (accessed via Drizzle ORM, NOT Supabase client)
- ✅ **Storage** (Supabase Storage for images/videos)
- ❌ **NOT for Auth** (we use BetterAuth instead of Supabase Auth)

### Backend-Only Database Access

- **No RLS**: All auth/authorization in Elysia API layer
- **Drizzle ORM**: All DB operations via type-safe queries (connects to Supabase PostgreSQL)
- **Single Connection Pool**: Centralized database access in backend
- **Supabase Storage**: Used for file uploads (images, videos) via Supabase client
- Avoids RLS complexity, clearer security model with API gateway pattern

### State Management Pattern

```typescript
// Centralized query keys
export const entityKeys = {
  all: ['entities'] as const,
  lists: () => [...entityKeys.all, 'list'] as const,
  list: (filter?: string) => [...entityKeys.lists(), filter] as const,
  details: () => [...entityKeys.all, 'detail'] as const,
  detail: (id: string) => [...entityKeys.details(), id] as const,
};

// React hook with TanStack Query (calls backend API)
export function useEntity(id: string) {
  return useQuery({
    queryKey: entityKeys.detail(id),
    queryFn: async () => {
      const response = await fetch(
        `${process.env.NEXT_PUBLIC_API_URL}/api/entities/${id}`,
        {
          credentials: 'include', // Include cookies for BetterAuth
        }
      );

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message || 'Failed to fetch entity');
      }

      const data = await response.json();
      return data.data; // API returns { success: true, data: ... }
    },
  });
}
```

### API Pattern (Elysia Backend)

```typescript
// 1. Validate input with ArkType schemas
// 2. Check authentication via BetterAuth plugin
// 3. Execute business logic in service layer
// 4. Perform DB operations with Drizzle ORM
// 5. Queue async work with Temporal workflows
// 6. Return standardized response { success: boolean, data?: T, error?: string }
```

### Frontend Pattern (Next.js)

```typescript
// 1. User interaction triggers action
// 2. Call backend API via fetch or TanStack Query
// 3. Handle loading/error states
// 4. Update UI with response data
// 5. Listen for real-time updates via WebSocket (optional)
```

### Error Handling

- Custom error classes: `OpenStoryError`, `ValidationError`
- Centralized handler: `handleApiError()`
- User-friendly messages, detailed logs

## Development Workflow

### Local Setup

1. Start PostgreSQL: `docker compose up -d postgres`
2. Run migrations: `cd apps/backend && bun run db:migrate`
3. Generate types: `bun run db:generate`
4. Start Temporal server: `docker compose up -d temporal`
5. Start Temporal worker: `cd apps/backend && bun run worker:dev`
6. Start backend API: `cd apps/backend && bun run dev`
7. Start frontend: `cd apps/frontend && bun run dev`
8. (Optional) Start Redis: `docker compose up -d redis`

### Code Conventions

- **Backend Routes**: Elysia routes in `apps/backend/src/routes/*.ts`
- **Services**: Business logic in `apps/backend/src/services/*.ts`
- **DB Queries**: Drizzle queries in `apps/backend/src/db/queries/*.ts`
- **Workflows**: Temporal workflows in `apps/backend/src/workflows/*.ts`
- **Frontend Hooks**: TanStack Query hooks in `apps/frontend/src/hooks/`
- **Components**: Client components only when needed (React 19 Server Components by default)
- **Validation**: ArkType schemas in `apps/backend/src/schemas/` (backend validation)
- **Types**: Drizzle generates DB types, share via `packages/types/`

## Key Workflows

### Frame Generation Flow

```
User triggers → Frontend calls Elysia API → API starts Temporal workflow →
Workflow executes activities (Fal.ai API, Save to Storage, Update DB) →
WebSocket notification → Frontend updates via TanStack Query invalidation
```

### Style Stack Application

```
User selects Stack → System validates model compatibility →
Adapter adjusts for model → Generation with adapted params →
Consistent style across all frames
```

## Security Model

- **Authentication**: BetterAuth (email/password, OAuth, anonymous sessions)
- **Authorization**: Team-based access control in Elysia middleware/plugins
- **Database**: Backend-only access via Drizzle ORM connection pool
- **API**: Rate limiting (Upstash Redis), ArkType validation, sanitized errors
- **CORS**: Configured in Elysia for frontend origin only
- **Secrets**: Never exposed to client, environment variables in backend only
- **Session Storage**: Redis-backed sessions for scalability

## Development Principles

1. **Type Safety**: TypeScript strict mode, ArkType validation
2. **Component Purity**: Minimal logic, shadcn/ui only
3. **State Separation**: Server state (TanStack Query) vs UI state (local)
4. **Error Handling**: Always handle errors, never expose internals
5. **Performance**: Server Components by default, client when needed
6. **Accessibility**: WCAG 2.1 AA compliance
7. **Testing**: Bun test for unit/integration tests

## Future Roadmap

- Phase 2: VFX LoRAs, marketplace, team collaboration
- Phase 3: DaVinci Resolve plugin, advanced editing
- Phase 4: Enterprise features, API for developers

Always consider the cinematic creation context and artistic consistency when making decisions.

---
> Source: [openstory-so/openstory](https://github.com/openstory-so/openstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
