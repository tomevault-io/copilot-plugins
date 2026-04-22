## fantasia-writer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fantasia is an AI-powered novel writing platform built with Next.js 16, React 19, and Supabase. It helps authors create novels using Google Gemini AI for story generation, character development, and chapter writing.

## Commands

```bash
pnpm dev      # Start development server (assumed already running)
pnpm build    # Production build
pnpm lint     # ESLint check
```

## Architecture

### State Management

- **StoryProvider** (`components/StoryProvider.tsx`): Central React Context managing all application state
  - User authentication and admin status
  - User credits system
  - Project list via SWR (`lib/hooks/useProjects.ts`)
  - Current project's StoryBible and chapters
  - Auto-save with dirty state tracking (1s debounce)

### Data Model (StoryBible)

The core data structure in `lib/types.ts`:
- **CoreConcept**: Title, theme, genre, setting, style configuration
- **Character[]**: Protagonist/Antagonist/Supporting with motivations and arcs
- **ChapterOutline[]**: Chapter summaries and generation status
- **WritingInstructions**: POV, pacing, dialogue style, things to avoid
- **StoryChapter**: Generated content with ChapterMetadata for continuity

### Database Layer

- **Supabase clients**: `lib/supabase/client.ts` (browser singleton), `lib/supabase/server.ts` (server with cookies)
- **Database operations**: `lib/supabase-db.ts` - CRUD for books, characters, outlines, instructions, chapters, chapter_memories, user_credits, admin_users

### AI Integration

- **Client service**: `services/gemini.ts` - Client-side wrappers calling API routes
- **API route**: `app/api/gemini/route.ts` - Actions: generateFullStoryBible, generateStoryCore, generateCharacterList, generateWritingInstructions, analyzeChapterContext, generateFullOutline, generateNewNames
- **Streaming**: `app/api/gemini/stream/route.ts` - SSE streaming for chapter generation

### Key Patterns

1. **Optimistic updates**: Project deletion/updates use SWR's `mutate` for instant UI feedback
2. **Chapter continuity**: ChapterMetadata (summary, keyEvents, items, location, characters) tracks story state across chapters
3. **Credits system**: Generation costs defined in `lib/types.ts` (GENERATION_COSTS), deducted via `deductUserCredits`

## Environment Variables

Required:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `GEMINI_API_KEY`

## Database Schema

Core tables: books, characters, outlines, instructions, chapters, chapter_memories, user_credits, admin_users, profiles. Migrations archived in `migrations_archive/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phoenixlwpapix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
