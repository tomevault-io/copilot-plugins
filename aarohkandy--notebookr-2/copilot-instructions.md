## notebookr-2

> Notebookr is an **agentic writer** application focused on generating **longer text** through AI-powered assistance. The application enables users to create comprehensive documents through a conversational AI interface. It features standalone authentication, AI-powered content generation using OpenRouter (with OpenAI fallback), and a credit-based monetization system. The goal is to streamline long-form writing and make it efficient and accessible.

# Notebookr - AI Context for Development

## Project Overview

Notebookr is an **agentic writer** application focused on generating **longer text** through AI-powered assistance. The application enables users to create comprehensive documents through a conversational AI interface. It features standalone authentication, AI-powered content generation using OpenRouter (with OpenAI fallback), and a credit-based monetization system. The goal is to streamline long-form writing and make it efficient and accessible.

**Communication Style:** Use simple, everyday language when communicating with users or in documentation.

**Application Focus:** This is an agentic writer optimized for longer text generation, not short snippets or quick responses. The AI workflow and UI are designed to support extended writing sessions and comprehensive document creation.

## Tech Stack

### Frontend
- **React 18** with **TypeScript**
- **Vite** for build tooling and dev server
- **Wouter** for client-side routing
- **TanStack Query** for data fetching and state management
- **Radix UI** primitives with **shadcn/ui** components
- **Tailwind CSS** for styling
- **Lucide React** for icons

### Backend
- **Node.js** with **Express.js** (TypeScript)
- **Drizzle ORM** with **PostgreSQL** (Neon serverless)
- **connect-pg-simple** for PostgreSQL session storage
- **Zod** for schema validation
- **Server-Sent Events (SSE)** for streaming AI responses

### AI Services
- **OpenRouter** (primary) - Multi-model fallback system with API key rotation
  - Models: Llama-3.3-70B, Gemini-2.0-Flash, Claude-3.5-Sonnet
- **OpenAI API** (fallback provider)
- Credit-based model selection (free users use OpenRouter, paid users can access faster OpenAI models)

### External Services
- **Stripe** for payment processing and credit purchases
- **Google Fonts**: Inter (UI/content) and JetBrains Mono (code blocks)

## Project Structure

```
/
├── client/          # Frontend React application
│   ├── src/
│   │   ├── components/     # React components
│   │   │   ├── ui/         # shadcn/ui components
│   │   │   └── ...
│   │   ├── pages/          # Page components
│   │   ├── hooks/          # React hooks
│   │   └── lib/            # Utility functions
│   └── index.html
├── server/          # Express backend
│   ├── index.ts     # Server entry point
│   ├── routes.ts    # API route registration
│   ├── auth.ts      # Authentication middleware
│   ├── ai-service.ts # AI generation logic
│   ├── db.ts        # Database connection
│   └── storage.ts   # Storage interface
├── api/             # Vercel serverless API routes
│   ├── _shared/     # Shared utilities (auth, cors, db, storage)
│   ├── auth/        # Authentication endpoints
│   ├── ai/          # AI generation endpoints
│   ├── notebooks/   # Notebook CRUD endpoints
│   ├── sections/    # Section endpoints
│   └── ...
├── shared/          # Shared code between client and server
│   └── schema.ts    # Drizzle schema and Zod validation
└── ...
```

## Architecture Patterns

### API Design
- **RESTful API** with Express routes
- All protected routes require authentication via `isAuthenticated` middleware
- Request validation using Zod schemas from `shared/schema.ts`
- User ownership validation for notebooks and sections
- API routes are in `/api` directory structure (Vercel serverless functions)

### Data Models
- **Users**: Authentication, credits, selected AI model
- **Notebooks**: User-scoped, contain `aiMemory` JSONB field for AI context
- **Sections**: Belong to notebooks, have order index
- **Messages**: Chat messages with role (user/assistant/system), messageType (status/completion)
- **SectionVersions**: Automatic snapshots before updates (for restore functionality)
- **Transactions**: Credit purchase and deduction history

### Authentication
- Standalone username/password authentication
- Password hashing using scrypt
- PostgreSQL session storage
- User sessions stored in `sessions` table

### AI System Architecture
- **Four-Phase Workflow**: Plan → Execute → Review → Post-Process
  - **Planning**: Conversational Q&A to understand user intent, extracts variables, creates document plan
  - **Execution**: Generates sections one at a time (2-5 paragraphs minimum, optimized for longer text)
  - **Review**: Evaluates quality and completeness
  - **Post-Process**: Fixes AI detection patterns
- **Agentic Writer Focus**: Optimized for longer text generation - sections should be substantial (multi-paragraph), with flowing transitions and comprehensive content
- **AI Memory**: Stored in `notebook.aiMemory` JSONB field (TODO lists, document plans, original instructions)
- **Streaming**: SSE with `threePhaseGenerationStream` generator, heartbeat events to prevent timeouts
- **Version History**: Automatic section snapshots before updates, restore functionality available

## Design System

**Reference:** See `design_guidelines.md` for comprehensive UI/UX specifications.

### Key Design Principles
- Information clarity over visual flourish
- Predictable, consistent interaction patterns
- Strong typographic hierarchy for technical content
- Subtle feedback for AI processing states
- **Aesthetic**: Beige and flowy - warm, organic, flowing design that feels natural and uncluttered

### Color Palette
- **Light Mode**: Beige/warm theme (HSL: 40 30% 96%, 38 25% 92%, etc.)
- **Dark Mode**: Warm dark (HSL: 30 15% 12%, 28 12% 18%, etc.)
- **Brand Primary**: Warm terracotta/coral (HSL: 25 70% 55%) for AI actions
- **Brand Accent**: Golden amber (HSL: 45 65% 50%) for highlights

### Typography
- **Primary**: Inter (Google Fonts) for UI and content
- **Monospace**: JetBrains Mono for code blocks and technical data

### Component Library
- Use **Radix UI** primitives with **shadcn/ui** components
- Component files in `client/src/components/ui/`
- Follow shadcn/ui patterns and styling conventions

## Coding Standards

### TypeScript
- **Strict mode enabled** (`"strict": true` in tsconfig.json)
- **ES Modules** (`"type": "module"` in package.json)
- Use type inference where appropriate, explicit types for public APIs
- Path aliases: `@/*` for `./client/src/*`, `@shared/*` for `./shared/*`

### File Naming
- Components: PascalCase (e.g., `NotebookEditor.tsx`)
- Utilities: camelCase (e.g., `authUtils.ts`)
- Constants: UPPER_SNAKE_CASE where appropriate

### Code Organization
- Keep components focused and single-purpose
- Extract shared logic to hooks or utility functions
- Use Zod schemas for all data validation
- Database queries use Drizzle ORM, not raw SQL (where possible)

### Import Patterns
- Use path aliases: `import { something } from '@/lib/utils'`
- Shared code: `import { type } from '@shared/schema'`
- Group imports: external libs, then internal modules

### Language & Tooling Conventions
- **Always use `py` instead of `python`** when referencing Python commands or scripts
  - Correct: `py script.py`, `#!/usr/bin/env py`
  - Incorrect: `python script.py`, `python3 script.py`

## Key Implementation Notes

### User-Scoped Data Access
- All notebooks and sections are user-scoped
- Always verify `userId` matches authenticated user before operations
- Use `notebook.userId === req.user.id` checks in API routes

### AI Generation Flow
1. User sends prompt → Planning phase (Q&A if needed)
2. Plan stored in `notebook.aiMemory`
3. Execution phase → Generate sections one by one
4. Section updates stream via SSE
5. Review and Post-Process phases complete generation
6. Messages saved with appropriate `messageType` (status/completion)

### Credit System
- Users have `credits` field in database
- OpenRouter models: free (uses `selectedAiModel === "free"`)
- OpenAI models: require credits (deducted on use)
- Transactions table tracks all credit movements

### Session Management
- Sessions stored in PostgreSQL via `connect-pg-simple`
- Session middleware configured in `server/auth.ts`
- Authenticated user available as `req.user` in route handlers

### Version History
- Automatic snapshots created in `sectionVersions` table before section updates
- Restore functionality available via `/api/sections/[id]/restore/[versionId]`

## UI/UX Patterns

- **Full-screen document view** for focused writing
- **Real-time chat progress** updates (planning, executing, reviewing, post-processing)
- **Message grouping**: Collapsible "Activity log" blocks for status messages, always-visible completion messages
- **One-at-a-time questions** for natural conversational flow
- **Visual feedback**: Section updates animate, progress indicators show AI state
- **Responsive design**: Mobile-first, drawer navigation on mobile, sidebar on desktop

## Development Workflow

### Scripts
- `npm run dev` - Start development server (Vite + Express)
- `npm run build` - Build for production
- `npm run check` - TypeScript type checking
- `npm run db:push` - Push database schema changes (Drizzle)

### Database Migrations
- Use Drizzle Kit: `npm run db:push`
- Schema defined in `shared/schema.ts`
- Changes should be reflected in schema first, then pushed

## Reference Documentation

- **Architecture Details**: See `replit.md` for comprehensive system architecture
- **Design Guidelines**: See `design_guidelines.md` for complete UI/UX specifications
- **Schema Definitions**: See `shared/schema.ts` for database schema and Zod validators

## Common Patterns

### API Route Handler
```typescript
// Always verify authentication
if (!req.user) return res.status(401).json({ message: "Unauthorized" });

// Validate request with Zod
const validated = schema.parse(req.body);

// Verify ownership
const notebook = await db.select().from(notebooks).where(eq(notebooks.id, id));
if (notebook.userId !== req.user.id) return res.status(403).json({ message: "Forbidden" });
```

### Component Structure
```typescript
// Use React hooks for state
import { useQuery } from '@tanstack/react-query';

// Use shadcn/ui components
import { Button } from '@/components/ui/button';

// Type props properly
interface Props {
  notebookId: string;
}
```

### Error Handling
- API errors return JSON: `{ message: string }`
- Use appropriate HTTP status codes (400, 401, 403, 404, 500)
- Client-side errors use toast notifications (from `useToast` hook)

## Aesthetic Guidelines

### Visual Design Philosophy
- **Beige and Flowy**: The design should feel warm, organic, and naturally flowing
- **Color Palette**: Emphasize beige, warm creams, and soft earth tones
- **Typography**: Flowing, readable text with generous line spacing for longer content
- **Layout**: Organic, non-rigid layouts that breathe - avoid harsh grids where possible
- **Animations**: Smooth, flowing transitions that feel natural (avoid abrupt or mechanical animations)
- **Spacing**: Generous whitespace, especially in long-form content areas

### UI Components for Longer Text
- **Editor/Viewer**: Optimized for reading and writing extended content
- **Progress Indicators**: Subtle, non-intrusive indicators for long-running AI generation
- **Section Navigation**: Easy navigation through longer documents
- **Scroll Behavior**: Smooth, natural scrolling optimized for extended reading sessions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aarohkandy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
