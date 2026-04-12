## echo-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **customer support chat system** built as a monorepo with two Next.js applications (dashboard and widget) and a Convex backend. The system uses AI-powered customer support agents with tool calling capabilities.

**Key Technologies:**
- Turborepo monorepo with pnpm workspaces
- Next.js 15 (React 19) for frontend apps
- Convex for backend/database with AI agents (@convex-dev/agent)
- Clerk for authentication
- shadcn/ui components in shared UI package
- OpenAI GPT-4o-mini for AI agent responses

## Development Commands

**Initial setup:**
```bash
pnpm install
```

**Development:**
```bash
# Start all apps in development mode (runs web, widget, and Convex backend)
pnpm dev

# Start specific app only
cd apps/web && pnpm dev        # Dashboard on port 3000
cd apps/widget && pnpm dev     # Widget on port 3001
cd packages/backend && pnpm dev # Convex backend

# Setup Convex backend (first time)
cd packages/backend && pnpm setup
```

**Building and Linting:**
```bash
pnpm build      # Build all packages
pnpm lint       # Lint all packages
pnpm format     # Format code with Prettier
```

**Type checking:**
```bash
cd apps/web && pnpm typecheck
cd apps/widget && pnpm typecheck
```

## Architecture

### Monorepo Structure

**Apps:**
- `apps/web` - Main dashboard application (port 3000)
  - Uses Clerk authentication with organization support
  - Contains auth and dashboard modules
  - App Router with route groups: `(auth)` and `(dashboard)`

- `apps/widget` - Customer-facing chat widget (port 3001)
  - Embeddable widget for customer conversations
  - Uses Vapi AI integration for voice
  - Widget module with atoms (Jotai state), hooks, and UI

**Packages:**
- `packages/backend` - Convex backend (`@workspace/backend`)
  - **Directory structure:**
    - `convex/public/` - Public API endpoints (organizations, conversations, messages, contactSessions)
    - `convex/private/` - Internal functions (conversations, messages, files)
    - `convex/system/ai/agents/` - AI agent definitions (supportAgent)
    - `convex/system/ai/tools/` - Agent tools (escalateConversation, resolveConversation)
    - `convex/lib/` - Utility functions (extractTextContent)
  - Uses `@convex-dev/agent` for AI agent orchestration
  - Uses `@convex-dev/rag` for RAG capabilities

- `packages/ui` - Shared UI components (`@workspace/ui`)
  - shadcn/ui components
  - Exports: `@workspace/ui/components/*`, `@workspace/ui/lib/*`, `@workspace/ui/hooks/*`
  - Tailwind CSS v4 with globals exported

- `packages/math` - Shared utilities (`@workspace/math`)
- `packages/eslint-config` - Shared ESLint config
- `packages/typescript-config` - Shared TypeScript config

### Key Architectural Patterns

**Convex Backend Organization:**
- Public functions go in `convex/public/` - these are exposed to frontend
- Private functions go in `convex/private/` - internal only
- AI system code in `convex/system/ai/` with agents and tools separated
- Uses Convex components: `@convex-dev/agent` and `@convex-dev/rag`

**AI Agent System:**
- Support agent defined in `convex/system/ai/agents/supportAgent.ts`
- Agent uses GPT-4o-mini with tool calling
- Tools: `escalateConversation`, `resolveConversation`
- Messages saved using `@convex-dev/agent`'s `saveMessage` function
- Thread-based conversations with pagination support

**Authentication Flow:**
- Clerk handles authentication in both apps
- Organization-based access control (orgId required)
- Identity checked in Convex mutations/queries via `ctx.auth.getUserIdentity()`
- Clerk JWT issuer domain configured in backend `.env.local`

**State Management:**
- Jotai for widget state (see `apps/widget/modules/widget/atoms`)
- Convex for real-time data syncing

**Module Organization:**
- Both apps use a `modules/` directory for feature organization
- Each module contains: components, UI, atoms/state, hooks, types, constants
- Dashboard has `auth` and `dashboard` modules
- Widget has a `widget` module

### Environment Variables

Each app requires its own `.env.local` file:

**apps/web/.env.local:**
- `NEXT_PUBLIC_CONVEX_URL` - Convex deployment URL
- Clerk keys: `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`
- Clerk URLs for sign-in/sign-up

**packages/backend/.env.local:**
- `CONVEX_DEPLOYMENT` - Convex deployment identifier
- `CONVEX_URL` - Convex API URL
- `CLERK_JWT_ISSUER_DOMAIN` - Clerk issuer for JWT validation
- `CLERK_SECRET_KEY` - Backend Clerk authentication
- `OPENAI_API_KEY` - Required for AI agent functionality

**apps/widget/.env.local:**
- Similar to web app (Convex URL, etc.)

## Adding shadcn/ui Components

To add components to the shared UI package:

```bash
pnpm dlx shadcn@latest add button -c apps/web
```

This places components in `packages/ui/src/components/` and makes them available via `@workspace/ui/components/*`.

## Working with Convex

**Convex functions:**
- Use `query` for read operations
- Use `mutation` for write operations
- Use `action` for external API calls (e.g., OpenAI)

**AI Agent Development:**
- Agent definitions in `convex/system/ai/agents/`
- Tools in `convex/system/ai/tools/`
- Use `Agent` class from `@convex-dev/agent`
- Tools are auto-registered and callable by agents

**Conversation Status Flow:**
- `unresolved` → `escalated` (when operator responds)
- `escalated` → `resolved` (when conversation finalized)
- Cannot send messages to `resolved` conversations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Virabyan99)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Virabyan99)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
