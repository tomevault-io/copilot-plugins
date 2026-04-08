## uigen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator. Users describe components in a chat UI; Claude generates code using tool calls into a virtual in-memory file system; results render live in a sandboxed iframe via client-side Babel/JSX transformation.

## Commands

```bash
npm run setup        # First-time setup: install deps, generate Prisma client, run migrations
npm run dev          # Start dev server (Turbopack) at localhost:3000
npm run build        # Production build
npm run lint         # ESLint
npm run test         # Vitest (all tests)
npx vitest run src/path/to/__tests__/file.test.tsx  # Run a single test file
npm run db:reset     # Reset SQLite database to fresh state
```

The dev server requires `NODE_OPTIONS='--require ./node-compat.cjs'` (handled by the npm scripts).

## Environment

Add `ANTHROPIC_API_KEY` to `.env` to use real Claude. Without it, the app uses `MockLanguageModel` in `src/lib/provider.ts` which returns static example components.

## Architecture

### AI Generation Flow

1. Chat message → `POST /api/chat` (`src/app/api/chat/route.ts`)
2. Route calls `streamText()` (Vercel AI SDK) with the current virtual file system serialized into the system prompt
3. Claude responds using two tools defined in `src/lib/tools/`:
   - `str_replace_editor` — view/create/edit files via string replacement
   - `file_manager` — rename/delete files
4. Tool calls are executed against the in-memory `FileSystem` class and streamed back to the client
5. On completion, project state is saved to SQLite (authenticated users only)

### Virtual File System

`src/lib/file-system.ts` — in-memory tree structure. All generated files live here; nothing is written to disk. The `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`) exposes this to React components.

### JSX Preview Pipeline

`src/lib/transform/jsx-transformer.ts` — uses `@babel/standalone` client-side to:
1. Transform JSX → JS
2. Generate an import map pointing to `esm.sh` CDN for third-party packages
3. Produce an HTML `srcdoc` string loaded into a sandboxed `<iframe>` (`src/components/preview/PreviewFrame.tsx`)

Entry point auto-detection looks for `App.jsx`, `index.jsx`, etc.

### State Management

- `ChatContext` (`src/lib/contexts/chat-context.tsx`) — chat messages, streaming state, tool call handling
- `FileSystemContext` — virtual FS state and operations
- No external state library; React context only

### Authentication

JWT tokens in HTTP-only cookies (7-day expiry). Logic in `src/lib/auth.ts`. Server actions in `src/actions/index.ts` handle sign-up/sign-in/sign-out. Anonymous users can generate components without accounts; their work is tracked via `src/lib/anon-work-tracker.ts`.

### Database

Prisma + SQLite (`prisma/dev.db`). Two models: `User` and `Project`. Projects store `messages` and `data` (file system state) as JSON strings. `userId` is optional to support anonymous projects.

### Key Paths

| Path | Purpose |
|------|---------|
| `src/app/api/chat/route.ts` | Streaming AI endpoint |
| `src/lib/file-system.ts` | Virtual file system core |
| `src/lib/transform/jsx-transformer.ts` | Babel JSX → iframe HTML |
| `src/lib/provider.ts` | Claude vs Mock model selection |
| `src/lib/prompts/generation.tsx` | System prompt for component generation |
| `src/lib/tools/` | AI tool definitions (Zod-validated) |
| `src/lib/contexts/` | React context providers |
| `src/components/ui/` | Radix UI primitive wrappers |
| `prisma/schema.prisma` | Database schema |

## Testing

Tests use Vitest + jsdom + `@testing-library/react`. Test files are colocated in `__tests__/` directories next to the code they test. Coverage spans chat components, file system logic, contexts, and the JSX transformer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scottlaw1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/scottlaw1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
