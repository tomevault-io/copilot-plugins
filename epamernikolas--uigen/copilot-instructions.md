## uigen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# First-time setup
npm run setup          # installs deps, generates Prisma client, runs migrations

# Development
npm run dev            # Next.js with Turbopack
npm run dev:daemon     # Background mode, outputs to logs.txt

# Production
npm run build
npm run start

# Testing
npm test               # Vitest

# Linting
npm run lint

# Database
npm run db:reset       # Full database reset (drops and remigrates)
```

## Environment

Copy `.env.example` to `.env` and set `ANTHROPIC_API_KEY`. If left blank, the app runs with a `MockLanguageModel` that simulates Claude responses — useful for development without an API key.

## Architecture

UIGen is an AI-powered React component generator with live preview. Users describe UI in natural language; Claude generates code that renders in an iframe in real time.

### Request Flow

1. User sends a chat message
2. `ChatProvider` (uses `@ai-sdk/react` `useChat`) POSTs to `/api/chat`
3. `/api/chat` calls `streamText` (Vercel AI SDK) with the current virtual file system serialized as context
4. Claude uses two tools — `str_replace_editor` (create/view/modify files) and `file_manager` (rename/delete) — to produce code
5. `FileSystemContext` handles incoming tool call results and updates the in-memory virtual FS
6. `PreviewFrame` detects FS changes via `refreshTrigger`, Babel-transforms JSX, generates an import map (esm.sh for npm packages, blob URLs for local files), and renders in a sandboxed iframe
7. On stream finish, the project is saved to SQLite (authenticated users only)

### Key Abstractions

**Virtual File System** (`src/lib/file-system.ts`): An in-memory `Map`-based tree. No disk I/O. Serializes to JSON for database storage and for seeding Claude's context on each request.

**AI Tools** (`src/lib/tools/`): Built dynamically per request, bound to the current `VirtualFileSystem` instance. `str_replace_editor` supports `view`, `create`, `str_replace`, and `insert` commands. `file_manager` handles `rename` and `delete`.

**Preview Engine** (`src/lib/transform/jsx-transformer.ts`): Babel (`@babel/standalone`) transforms JSX at runtime. The iframe uses ES module import maps — external packages resolve to esm.sh, local files become blob URLs. The iframe is sandboxed.

**Provider** (`src/lib/provider.ts`): Uses `claude-haiku-4-5` by default. Falls back to `MockLanguageModel` if `ANTHROPIC_API_KEY` is absent.

**Auth** (`src/lib/auth.ts`): JWT tokens in httpOnly cookies (7-day expiry). Middleware protects `/api/projects` and `/api/filesystem`.

### Database Schema

Prisma + SQLite (`prisma/dev.db`).

- `User`: id, email, password (bcrypt), timestamps
- `Project`: id, name, userId (FK → cascade delete), `messages` (JSON string), `data` (JSON string for the virtual FS), timestamps

### AI System Prompt

Defined in `src/lib/prompts/generation.tsx`. Key constraints Claude follows when generating code:
- Every project must have a root `/App.jsx`
- Use Tailwind for styling (no inline styles)
- All local imports use the `@/` alias
- File system root is `/`

### Path Aliases

`@/*` maps to `src/*` (configured in `tsconfig.json`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EpamerNikolas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EpamerNikolas)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
