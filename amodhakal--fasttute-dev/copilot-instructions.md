## fasttute-dev

> An AI-powered YouTube learning platform that transforms passive video tutorials into interactive lessons with AI-generated chapters, synchronized transcripts, and context-aware assistance.

# Fasttute.dev - Developer Guide

An AI-powered YouTube learning platform that transforms passive video tutorials into interactive lessons with AI-generated chapters, synchronized transcripts, and context-aware assistance.

## Project Overview

- **Monorepo:** Turborepo with Bun workspaces
- **Frontend:** Next.js 16 (App Router) + Turbopack
- **Backend:** Convex (serverless functions + database)
- **AI:** Google Gemini (chapters, Q&A, focus mode)
- **Auth:** Clerk
- **Styling:** Tailwind CSS 4

## Project Structure

```
fasttute.dev/
├── apps/web/                    # Next.js frontend
│   └── src/
│       ├── app/                 # App Router pages
│       │   ├── page.tsx        # Landing page
│       │   ├── video/[id]/     # Video interaction page
│       │   └── ...
│       ├── components/          # Shared UI components
│       ├── hooks/               # React hooks
│       └── utils/               # Frontend utilities
├── packages/backend/            # Shared Convex backend
│   ├── convex/                 # Database schema & functions
│   │   ├── schema.ts           # Main schema
│   │   ├── videoInfo.ts        # Video CRUD
│   │   ├── videoChat.ts        # Chat with streaming
│   │   ├── focus.ts            # Focus mode
│   │   └── retrieveVideoInfo.ts # YouTube fetching
│   ├── utils/ai/               # AI modules
│   │   ├── chapters.ts         # Chapter generation
│   │   ├── qna.ts              # Q&A streaming
│   │   └── focus.ts            # Focus segment detection
│   └── api/                    # Type exports
├── package.json                 # Root workspace
└── turbo.json                   # Turborepo config
```

## Tech Stack

| Category | Technology                           |
| -------- | ------------------------------------ |
| Runtime  | Bun 1.3.5+                           |
| Frontend | Next.js 16.1.5, React 19.1.0         |
| Backend  | Convex 1.27.3                        |
| AI       | Google Gemini (@google/genai 1.22.0) |
| Auth     | Clerk 5.51.0                         |
| Styling  | Tailwind CSS 4.1.14                  |
| YouTube  | youtubei.js 15.1.1                   |

## Commands

```bash
# Install dependencies
bun install

# Run development (Next.js + Convex)
bun run dev

# Build for production
bun run build

# Start production server
bun run start

# Type check all packages
bun run check-types

# Lint all packages
bun run lint

# Format code
bun run format
```

## Database Schema (Convex)

### video_info

```typescript
{
  _id: Id<"video_info">,
  youtubeId: string,
  title: string,
  transcript: Array<{ text: string, offset: number, duration: number }>,
  status: "pending" | "completed" | "failed",
  chapters: Array<{ title: string, offset: number }> | undefined
}
```

### video_chat

```typescript
{
  _id: Id<"video_chat">,
  videoId: Id<"video_info">,
  userId: string,
  chat: Array<{ id: string, role: "User" | "Model", text: string }>
}
```

### video_focus

```typescript
{
  _id: Id<"video_focus">,
  videoId: Id<"video_info">,
  segments: Array<{ startOffset: number, endOffset: number, reason: string }>,
  generatedAt: number
}
```

## Code Conventions

### Naming

- **Components:** PascalCase (`LandingPage.tsx`, `ChatItem.tsx`)
- **Hooks:** camelCase with `use` prefix (`useYoutubePlayer.ts`)
- **Utilities:** camelCase (`errorToast.ts`, `transcript.ts`)
- **Database Tables:** snake_case (`video_info`, `video_chat`)

### React Patterns

- Use `"use client"` for client components
- Server Actions in `actions.ts` files
- Context API for complex state (`VideoPageContext`)
- Custom hooks for reusable logic

### Backend (Convex)

- **Queries:** Read-only data fetching
- **Mutations:** Write operations
- **Actions:** Server-side logic with external API calls
- Always regenerate types after schema changes: `npx convex dev`

## AI Integration

### Models

| Feature    | Model                 | Use Case                        |
| ---------- | --------------------- | ------------------------------- |
| Chapters   | gemini-2.5-flash-lite | Generate TOC from transcript    |
| Q&A        | gemini-2.5-pro        | Answer questions with citations |
| Focus Mode | gemini-2.5-flash-lite | Detect skippable segments       |

### Citation Format

AI responses use `[[OFFSET IN SECONDS]]` format (e.g., `[[124]]`) which frontend parses into clickable timestamps.

## Environment Variables

```env
GEMINI_KEY=your_gemini_api_key
NEXT_PUBLIC_CONVEX_URL=your_convex_deployment_url
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_JWT_ISSUER_DOMAIN=your_clerk_jwt_issuer
```

## Adding New Features

### 1. New Database Table

1. Create schema in `packages/backend/convex/schema/<name>.ts`
2. Import in `packages/backend/convex/schema.ts`
3. Run `npx convex dev` to generate types

### 2. New Convex Functions

1. Add to appropriate file in `packages/backend/convex/`
2. Export query/mutation/action
3. Use from frontend via `api.module.functionName`

### 3. New Frontend Component

1. Follow existing component patterns
2. Use Convex for data: `useQuery(api.module.function, args)`
3. Use Server Actions for mutations

### 4. New AI Feature

1. Create module in `packages/backend/utils/ai/`
2. Use GoogleGenAI SDK
3. Define system instruction and response schema
4. Call from Convex action/mutation

## Key Files

| File                                           | Purpose                     |
| ---------------------------------------------- | --------------------------- |
| `apps/web/src/app/video/[id]/page.tsx`         | Main video interaction page |
| `apps/web/src/hooks/useYoutubePlayer.ts`       | YouTube player + focus mode |
| `packages/backend/convex/retrieveVideoInfo.ts` | YouTube data fetching       |
| `packages/backend/utils/ai/chapters.ts`        | Chapter generation          |
| `packages/backend/utils/ai/qna.ts`             | Q&A with streaming          |

## Testing

- Framework: Bun's built-in test runner
- Run tests: `bun test`
- Add tests alongside utilities in `apps/web/src/utils/`

## Deployment

- Platform: Vercel
- Runs `bun run build` which builds both web and backend
- Convex auto-integrates with Vercel

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amodhakal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/amodhakal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
