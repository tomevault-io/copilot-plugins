## comment-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
pnpm dev      # Start development server at http://localhost:3000
pnpm build    # Production build
pnpm lint     # Run ESLint
```

## Architecture

This is a Next.js 14 (App Router) application that generates report card comments for Ontario Grade 6 teachers. Users select subjects, sections, and proficiency levels to either pull pre-written comments from a database or refine comments using OpenAI GPT.

### Tech Stack
- **Frontend**: Next.js 14, React 18, TypeScript, Tailwind CSS, Shadcn/ui (Radix primitives)
- **Backend**: Supabase (PostgreSQL + Auth)
- **AI**: OpenAI via Vercel AI SDK (model: gpt-4.1-mini)

### Key Directories
- `app/components/subjects/` - Subject-specific form components (Math, Language, Science, LearningSkills)
- `app/constants/` - Comment bank (`commentBank.ts`), subject definitions (`subjects.ts`), AI prompts (`promptConfig.ts`)
- `app/utils/supabase/` - Supabase client factories (separate client/server implementations)
- `app/api/chat/` - POST endpoint for GPT comment refinement
- `app/api/comments/` - DELETE endpoint for CRUD operations

### Database Schema
```
comment_versions (user_id, version_name, id)
  └── subjects (version_id, subject_name, id)
      └── sections (subject_id, section_name, id)
          └── levels (section_id, level_name, comment, id)
```

### Comment Generation Flow
1. **Static path**: User selects dropdowns → `submitForm.ts` uses Strategy pattern (MathStrategy, LanguageStrategy, etc.) → pulls from `commentBank.ts` → updates TextContext
2. **AI path**: "Use GPT" button → `handleUseGPT()` → POST `/api/chat` → OpenAI refines comment → updates TextContext

### State Management
- `TextContext` - Shared comment text across components
- `ConfettiSwitch` - Triggers celebration animation after GPT success

### Prompt System
AI prompts preserve token placeholders for pronoun substitution:
- *N* = student name
- *P* = subject pronoun (he/she)
- *H* = object pronoun (him/her)
- *R* = possessive pronoun (his/her)

Character limits:
- Default subjects: 900-1300 characters
- Learning Skills: 1600-2000 characters

## Environment Variables

Required in `.env.local`:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`
- OpenAI API key (for GPT features)

## Configuration Notes

- TypeScript strict mode is OFF (`tsconfig.json`)
- Build errors are ignored (`next.config.mjs`: `typescript.ignoreBuildErrors: true`)
- Path alias: `@/*` maps to `./app/*`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WattersIV) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
