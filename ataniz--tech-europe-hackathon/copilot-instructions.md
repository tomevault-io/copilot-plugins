## tech-europe-hackathon

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev              # Start dev server (uses Turbopack)
pnpm build            # Run migrations + build for production
pnpm lint             # Check code with Ultracite/Biome
pnpm format           # Auto-fix lint issues

# Database (Drizzle ORM + PostgreSQL)
pnpm db:migrate       # Apply migrations
pnpm db:generate      # Generate new migration from schema changes
pnpm db:studio        # Open Drizzle Studio GUI
pnpm db:push          # Push schema directly (dev only)

# Testing (Playwright)
pnpm test             # Run all E2E tests (sets PLAYWRIGHT=True)

# Local Services (macOS with Homebrew)
brew services start postgresql@14
brew services start redis
```

## Architecture

**Chat SDK** - Next.js 15 AI chatbot template using AI SDK with Google Gemini.

### Route Groups
- `app/(auth)/` - Authentication (NextAuth v5 beta, guest login support)
- `app/(chat)/` - Main chat application and API routes

### Core Directories
- `lib/ai/` - AI provider config, model definitions, system prompts, tool implementations
- `lib/db/` - Drizzle schema, queries, migrations (local PostgreSQL)
- `artifacts/` - Document types (text, code, image, sheet) with client/server split
- `components/` - React components, `components/elements/` for chat-specific UI

### AI Integration
- Models defined in `lib/ai/models.ts` (chat-model, chat-model-reasoning)
- Provider: Google Gemini 2.0 Flash in `lib/ai/providers.ts`
- Tools: getWeather, createDocument, updateDocument, requestSuggestions

### Local Development Setup
- PostgreSQL: Local instance on port 5432
- Redis: Local instance on port 6379 (for resumable streams)
- File uploads: Stored in `public/uploads/` (not Vercel Blob)

### Database Schema (`lib/db/schema.ts`)
Tables: User, Chat, Message_v2, Vote_v2, Document, Suggestion, Stream
- Uses `_v2` suffix for migrated tables (deprecated versions still present)

### Testing Structure
- `tests/e2e/` - End-to-end tests (chat, artifacts, reasoning, session)
- `tests/routes/` - API route tests
- Mock models in `lib/ai/models.mock.ts` when PLAYWRIGHT=True

## Linting

Uses Ultracite (Biome wrapper) with strict TypeScript and accessibility rules. Config in `biome.jsonc`.

Key disabled rules (intentional):
- `noExplicitAny` - needs refactoring
- `noConsole` - allowed for debugging
- `noBitwiseOperators` - needed for UUID generation

---
> Source: [ataniz/tech-europe-hackathon](https://github.com/ataniz/tech-europe-hackathon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
