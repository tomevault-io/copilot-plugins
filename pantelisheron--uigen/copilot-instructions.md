## uigen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, Claude generates code via tool use, and the result renders live in an iframe — all without writing files to disk.

## Commands

- `npm run setup` — Install deps, generate Prisma client, run migrations (first-time setup)
- `npm run dev` — Start dev server with Turbopack (http://localhost:3000)
- `npm run build` — Production build
- `npm run lint` — ESLint
- `npm run test` — Run vitest (jsdom environment)
- `npx vitest run src/path/to/test.ts` — Run a single test file
- `npm run db:reset` — Reset SQLite database

## Architecture

### Data Flow

User prompt → ChatInterface → `/api/chat` route → Claude (streaming with tool use) → tool calls sync to FileSystemContext → VirtualFileSystem updates → PreviewFrame re-renders in iframe

### Three-Panel Layout (`main-content.tsx`)

Left panel (35%): Chat interface. Right panel (65%): Tabs for live Preview and Code editor (file tree + Monaco).

### Virtual File System (`src/lib/file-system.ts`)

In-memory tree using `Map<string, FileNode>`. No files are written to disk. The VFS serializes to JSON for database persistence and transport to the API route. Entry point for preview is `App.jsx`, `App.tsx`, or `index.jsx`.

### AI Integration

- **API route** (`src/app/api/chat/route.ts`): Receives messages + serialized file system, calls Claude with `streamText`. Max 40 tool-use steps.
- **Two AI tools**: `str_replace_editor` (create/view/edit files) and `file_manager` (rename/delete). Defined in `src/lib/tools/`.
- **System prompt** (`src/lib/prompts/generation.tsx`): Instructs the AI to use `/App.jsx` as entry point, Tailwind for styling, and `@/` import aliases for local files.
- **Provider** (`src/lib/provider.ts`): Configures the Anthropic provider. Falls back to mock/static responses when no API key is set.

### Contexts

- **FileSystemContext** (`src/lib/contexts/file-system-context.tsx`): Wraps VirtualFileSystem, exposes CRUD methods, handles `onToolCall` from the AI SDK to sync tool results into the VFS. Increments `refreshTrigger` to signal preview updates.
- **ChatContext** (`src/lib/contexts/chat-context.tsx`): Wraps AI SDK's `useAIChat`, passes serialized file system state in request body, delegates tool calls to FileSystemContext.

### Live Preview (`src/components/preview/PreviewFrame.tsx`)

Watches `refreshTrigger`, transforms JSX with Babel (`@babel/standalone`), generates an import map pointing to esm.sh CDN for React 19, and renders in a sandboxed iframe.

### Auth & Persistence

- JWT-based auth via `jose` (7-day cookies) in `src/lib/auth.ts`
- Middleware (`src/middleware.ts`) protects API routes
- Anonymous users can generate components; tracked via sessionStorage (`src/lib/anon-work-tracker.ts`)
- Server actions in `src/actions/` handle project CRUD
- **Prisma/SQLite**: `User` and `Project` models. Projects store messages and VFS state as JSON columns. The database schema is defined in `prisma/schema.prisma` — reference it anytime you need to understand the structure of stored data.

## Code Style

- Use comments sparingly. Only comment complex, non-obvious code.

## Path Aliases

`@/*` maps to `./src/*` (configured in tsconfig.json).

## UI Components

Uses shadcn/ui (new-york style) with Radix primitives. Components in `src/components/ui/`. Utility: `cn()` from `src/lib/utils.ts`.

---
> Source: [PantelisHeron/uigen](https://github.com/PantelisHeron/uigen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
