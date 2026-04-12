## uigen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
npm run dev          # Start dev server with Turbopack (port 3000)
npm run build        # Production build
npm run lint         # ESLint via Next.js

# Testing
npm run test         # Run all Vitest tests
npx vitest run src/lib/__tests__/file-system.test.ts  # Run a single test file

# Database
npm run setup        # Install deps + generate Prisma client + run migrations
npm run db:reset     # Reset database

# Prisma
npx prisma migrate dev   # Create and apply migration
npx prisma generate      # Regenerate Prisma client after schema changes
```

## Architecture

UIGen is a Next.js 15 app (App Router) where users chat with Claude AI to generate and preview React components in real-time. The three core systems that power this:

### 1. Virtual File System (`src/lib/file-system.ts`)
All generated component files are stored in-memory via `VirtualFileSystem` — no disk I/O. The AI manipulates files through two tools:
- `str_replace_editor` — creates, views, and edits files (string replacement, line insertion)
- `file_manager` — renames and deletes files

Files are serialized as JSON and persisted to the database under `Project.data`.

### 2. Live Preview (`src/components/preview/PreviewFrame.tsx`)
The preview is a sandboxed `<iframe>` that runs the virtual FS files. Flow:
1. Babel (browser standalone) transpiles JSX/TypeScript to JS
2. Blob URLs created per module
3. An import map resolves `@/` aliases and CDN modules via `esm.sh`
4. Entry point is always `/App.jsx`

### 3. AI Chat Endpoint (`src/app/api/chat/route.ts`)
Streaming endpoint using Vercel AI SDK + Anthropic Claude. Uses prompt caching (ephemeral) and allows up to 40 agentic steps. On completion, projects are auto-saved for authenticated users.

### State Management
Two React contexts bridge everything:
- `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`) — holds the `VirtualFileSystem` instance, selected file, triggers preview refreshes
- `ChatContext` (`src/lib/contexts/chat-context.tsx`) — chat history, tool call handling, anonymous work tracking

### UI Layout (`src/app/main-content.tsx`)
Resizable panels (react-resizable-panels): Chat panel on the left, Preview+Code tabs on the right. Code view uses Monaco Editor with a `FileTree` sidebar.

### Auth
JWT sessions stored in HttpOnly cookies (7-day TTL, HS256 via `jose`). Server actions in `src/actions/` handle sign-up/sign-in/sign-out. Middleware in `src/middleware.ts` protects API routes. Passwords hashed with bcrypt.

### Database
Prisma + SQLite. Two models: `User` and `Project`. Projects store chat history (`messages`) and virtual FS state (`data`) as JSON strings.

### Mock Provider
`src/lib/provider.ts` exports `getModel()` which returns the Anthropic provider if `ANTHROPIC_API_KEY` is set, otherwise a mock provider. The mock is useful for development/testing without API costs.

## Key Conventions
- Path alias `@/` maps to `src/`
- All generated components use React + Tailwind CSS; no plain HTML files
- The AI system prompt lives in `src/lib/prompts/generation.tsx`
- shadcn/ui components are in `src/components/ui/`
- Tests use Vitest + jsdom + @testing-library/react

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tahir-aijaz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tahir-aijaz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
