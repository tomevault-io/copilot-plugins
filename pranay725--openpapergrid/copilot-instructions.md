## openpapergrid

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
OpenPaperGrid is an open-source AI-powered biomedical literature search and analysis platform, designed as a self-hosted alternative to proprietary AI research tools. It combines OpenAlex metadata with full-text extraction and LLM-powered summarization capabilities.

## Tech Stack
- **Framework**: Next.js 15.3.4 with App Router and TypeScript
- **UI**: Tailwind CSS v4, React 19
- **Database & Auth**: Supabase (PostgreSQL with RLS)
- **AI Integration**: 
  - Vercel AI SDK for streaming responses
  - Multiple LLM providers: OpenAI, Anthropic, OpenRouter
  - LlamaIndex for document processing
- **External APIs**: OpenAlex (38M+ papers), LlamaParse (PDF extraction)

## Commands
```bash
# Development
npm run dev          # Start development server with Turbopack

# Production
npm run build        # Build for production
npm run start        # Start production server

# Code Quality
npm run lint         # Run ESLint
```

## Environment Variables
Required environment variables:
```
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_KEY=

# AI Providers (at least one required)
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
OPENROUTER_API_KEY=

# Document Processing
LLAMA_CLOUD_API_KEY=
```

## Design System

### Colors
- **Primary Blue**: `blue-600` (#2563EB) - Used for primary actions, links, and brand elements
- **Hover Blue**: `blue-700` (#1D4ED8) - Hover state for primary elements
- **Background**: White with gray-100 accents
- **Text**: `gray-900` for headings, `gray-600` for body text
- **Borders**: `gray-200` or `gray-300`

### Typography
- **Font Weight**: Normal (400) for most text, avoiding bold/semibold in favor of clean hierarchy
- **Headings**: `text-2xl` with `font-normal`
- **Body**: Default text size with `text-gray-600`

### Components
- **Buttons**: Use the Button component from `/components/ui/button.tsx`
  - Primary: `variant="default"` (blue-600 background)
  - Secondary: `variant="ghost"` (transparent with hover state)
  - Sizes: `sm`, `default`, `lg`
- **Modals**: 
  - Backdrop: `bg-black/50 backdrop-blur-sm` for blurred background
  - Container: White with `rounded-lg shadow-2xl` and `border-gray-200`
  - Padding: `p-8` for modal content
- **Links**: `text-blue-600 hover:underline`

### Spacing & Layout
- Consistent use of Tailwind spacing utilities
- Modal max-width: `max-w-md` 
- Button full width in modals: `w-full`
- Vertical spacing between elements: `space-y-4` or `mb-6`

## Key Architecture Patterns

### API Routes Structure
- `/app/api/ai/` - AI endpoints (chat, extraction, confidence scoring)
- `/app/api/fulltext/` - Document retrieval and parsing
- `/app/api/search/` - OpenAlex integration
- All API routes include rate limiting and authentication checks

### Authentication Pattern
- Middleware at `/middleware.ts` handles auth for protected routes
- Two Supabase clients: 
  - Client-side for user operations
  - Server-side with service key for admin operations
- Session management through cookies

### AI Integration Pattern
- Streaming responses using Vercel AI SDK
- Provider abstraction allowing multiple LLMs
- Retry logic for external API failures
- Structured extraction with Zod schemas

### Search Results Architecture
The main search interface (`/app/search-results/`) uses:
- Server Components for initial data fetching
- Client Components for interactivity
- Custom hooks for data management
- Comprehensive error handling and loading states

## Important Files and Directories
- `/app/search-results/SearchResultsClientRefactored.tsx` - Main search interface
- `/lib/AuthContext.tsx` - Authentication context and hooks
- `/middleware.ts` - Auth and rate limiting middleware
- `/app/api/ai/chat/route.ts` - Main chat endpoint with streaming
- `/app/search-results/hooks/` - Custom React hooks for features

## Development Notes
1. Always use Supabase RLS for data security
2. Implement rate limiting for all AI endpoints
3. Use streaming for LLM responses to improve UX
4. Handle external API failures gracefully with retries
5. Keep sensitive operations server-side only

## Current Features
- PubMed-style search with OpenAlex
- AI-powered paper summarization and Q&A
- Custom field extraction from papers
- Full-text PDF parsing
- Screening configurations for systematic reviews
- Export functionality (BibTeX, RIS)
- User authentication and saved searches

## Testing Approach
- Test authentication at `/test-auth`
- API testing endpoints available (e.g., `/api/test-db`)
- Manual testing guides in `/docs/` directory

---
> Source: [pranay725/openpapergrid](https://github.com/pranay725/openpapergrid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
