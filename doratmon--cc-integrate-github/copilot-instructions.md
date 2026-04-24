## cc-integrate-github

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
npm run setup          # Install dependencies, generate Prisma client, run migrations
npm run dev            # Start dev server with Turbopack (http://localhost:3000)
npm run build          # Production build
npm run start          # Start production server
npm run lint           # Run ESLint
npm run test           # Run Vitest tests
npm run db:reset       # Reset database (deletes all data)
```

## Architecture Overview

UIGen is an AI-powered React component generator built with Next.js 15. The application uses a **virtual file system** approach - no files are written to disk during development. All code generation and editing happens in-memory, with persistence handled through the database.

### Key Architectural Concepts

**Virtual File System (`src/lib/file-system.ts`)**
- In-memory file system with `VirtualFileSystem` class
- Supports files/directories with full CRUD operations
- Serializable via `serialize()` and `deserializeFromNodes()`
- Methods like `viewFile`, `createFileWithParents`, `replaceInFile`, `insertInFile` are exposed as AI tools

**AI Tool System**
- Two main tools in `src/lib/tools/`:
  - `str_replace_editor` - View/create/str_replace/insert operations on files
  - `file_manager` - List/delete/rename file operations
- Tools receive a `VirtualFileSystem` instance and expose `execute()` methods
- AI calls these tools to manipulate the virtual file system

**Chat API Flow** (`src/app/api/chat/route.ts`)
1. Receives messages + serialized file system state
2. Reconstructs `VirtualFileSystem` from `files` parameter
3. Calls AI model with tools attached
4. AI uses tools to modify the virtual file system
5. On completion, serializes file system and saves to database (if authenticated)

**Provider Pattern** (`src/lib/provider.ts`)
- Falls back to mock AI provider when `ANTHROPIC_API_KEY` is not set
- Mock provider generates static component code based on keywords (counter, form, card)
- Real provider uses Anthropic Claude via Vercel AI SDK

### Project Structure

```
src/
├── app/                      # Next.js App Router
│   ├── api/chat/            # POST endpoint for AI chat
│   ├── [projectId]/         # Project workspace pages (requires auth)
│   └── page.tsx             # Landing page with auth
├── components/
│   ├── chat/                # ChatInterface, MessageInput components
│   ├── editor/              # CodeEditor (Monaco), FileTree
│   ├── preview/             # PreviewFrame (iframe rendering)
│   └── ui/                  # Radix UI base components
├── lib/
│   ├── contexts/            # React contexts (FileSystemContext)
│   ├── tools/               # AI tool implementations
│   ├── transform/           # Code transformation utilities
│   ├── auth.ts              # JWT authentication with bcrypt
│   ├── file-system.ts       # VirtualFileSystem class
│   ├── prompts/             # AI system prompts
│   └── provider.ts          # AI provider selection (real vs mock)
```

### Database Schema (`prisma/schema.prisma`)

```prisma
User {
  id, email, password (bcrypt)
  projects[] (cascade delete)
}

Project {
  id, name, userId (optional)
  messages (JSON: chat history)
  data (JSON: serialized file system)
}
```

- SQLite for development (`file:./dev.db`)
- Prisma client output: `src/generated/prisma`
- Projects can be anonymous (no userId) or owned by registered users

### Authentication Flow

- JWT-based auth using `jose` library
- Session stored in HTTP-only cookies
- `src/lib/auth.ts`: `getSession()`, `createSession()`, `verifySession()`
- Server actions in `src/actions/` handle auth operations
- Anonymous users can create projects but data is not persisted across sessions

### Code Preview System

**Live Preview** (`src/components/preview/PreviewFrame.tsx`)
- Uses iframe to isolate generated components
- Dynamic import map for module resolution (`@/` alias support)
- Babel transforms JSX/TSX in-browser via `@babel/standalone`
- CSS inlined into generated modules
- Blob URLs created for each file change

**Code Editor** (`src/components/editor/CodeEditor.tsx`)
- Monaco Editor integration
- Two-way sync with virtual file system
- Updates trigger preview refresh via FileSystemContext

### Important Implementation Details

1. **No File Writes**: The virtual file system never writes to disk. All persistence is through the Project.data JSON field.

2. **Path Normalization**: The file system normalizes paths (removes trailing slashes, collapses multiple slashes). Always use normalized paths when working with files.

3. **Import Resolution**: The preview system supports `@/` imports through an import map that maps to blob URLs.

4. **CSS Handling**: Styles are extracted and inlined into module blobs to work within the iframe context.

5. **Error Boundaries**: Syntax errors in generated code are displayed in the preview iframe with line numbers.

6. **Mock Provider Fallback**: When no API key is set, the app still functions - the mock provider returns pre-written component code based on keyword detection.

### Environment Variables

```bash
ANTHROPIC_API_KEY=sk-ant-xxx  # Optional: enables real Claude AI
```

Without this key, the application uses the mock provider.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Doratmon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
