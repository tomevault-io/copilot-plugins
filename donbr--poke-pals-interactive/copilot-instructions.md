## poke-pals-interactive

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PokéPals Interactive is a kid-friendly (grades 2-4) Next.js 16 application combining Pokémon exploration with AI-powered features using OpenAI's GPT-5-nano model. The app features Professor Pine (AI chat), story generation, interactive games, and natural language Pokémon search.

## Essential Commands

### Development
```bash
npm install                   # Install dependencies
npm run dev                   # Start dev server (localhost:3000)
npm run build                 # Production build
npm start                     # Run production server
npm run lint                  # Run ESLint
```

### Environment Setup
```bash
cp .env.example .env.local    # Create local env file
# Add OPENAI_API_KEY to .env.local
```

## Architecture Overview

### Three-Tier Layered Architecture

**Presentation Layer** (`/app`, `/components`)
- Server Components for initial SSR renders
- Client Components marked with `"use client"` for interactivity
- Theme switching via React Context + CSS custom properties

**Business Logic Layer** (`/app/api/llm/*`)
- 7 AI endpoints: chat, quiz, story, query-pokeapi, game-hints, color-prompt, fun-fact
- All use GPT-5-nano model
- Content safety filtering with blocked pattern detection
- 30-second timeout protection (`maxDuration = 30`)
- Zod schema validation for structured outputs

**Data/Infrastructure Layer**
- PokeAPI: Direct client-side fetch (no proxy needed, public API)
- OpenAI: Server-side via Vercel AI SDK (API key required)
- localStorage: Theme preferences (client-side persistence)

### Key Data Flow Patterns

1. **Streaming AI Pattern** (chat): Uses Server-Sent Events (SSE) with `streamText()` for real-time responses
2. **Structured Generation** (quiz/story): Uses `generateObject()` with Zod schemas for type-safe JSON
3. **Direct API Pattern** (Pokémon data): Client components fetch directly from PokeAPI
4. **Theme Management**: React Context → localStorage → CSS custom properties

## Critical Implementation Details

### AI Endpoint Pattern
All LLM endpoints follow this structure:
- Export `maxDuration = 30` for Vercel timeout protection
- Use pre-filtering for content safety (see `/app/api/llm/chat/route.ts:40-56`)
- GPT-5-nano configuration: Exclude temperature for reasoning model, set `maxOutputTokens` ~2000-3000
- Return fallback content on errors to maintain kid-friendly experience

### Content Safety System
**Professor Pine System Prompt** (`/app/api/llm/chat/route.ts:6-35`):
- Grade 2-4 reading level
- <100 word responses
- No copyrighted content/battle mechanics
- Redirect scary topics to positive ones
- Pre-filter blocks patterns: kill, hate, weapon, violence, inappropriate, adult

### Theme System Architecture
- Three themes: candy (default), space, ocean
- CSS custom properties in `/app/globals.css` using OKLCH color space
- Theme classes: `.theme-space`, `.theme-ocean` (root = candy)
- Provider: `/components/theme-provider.tsx` manages state + localStorage
- Path alias `@/*` maps to project root (configured in `tsconfig.json`)

### TypeScript Configuration
- Strict mode enabled
- Path alias: `@/*` → `./*` (use `@/components`, `@/lib`, etc.)
- React 19 with new `react-jsx` transform
- Target: ES2017 for broad compatibility

### Image Handling
Next.js configured to allow Pokemon sprites from `raw.githubusercontent.com` (see `next.config.ts:4-11`)

## Component Organization

### UI Primitives (`/components/ui/`)
shadcn/ui components built on Radix UI: button (6 variants/sizes), card, input, dialog, tabs. Styled with Tailwind using `cn()` utility from `/lib/utils.ts`.

### Feature Components (`/components/`)
- **Chat**: `professor-pine-chat.tsx` - Uses `useChat` hook from AI SDK
- **Exploration**: `pokemon-explorer.tsx` + `pokemon-grid.tsx` + `pokemon-detail-modal.tsx`
- **Games**: `games/guess-game.tsx`, `games/quiz-game.tsx`, `games/color-playground.tsx`
- **Stories**: `story-builder.tsx` - Branching narratives with AI choices
- **Layout**: `page-wrapper.tsx`, `navigation.tsx`, `footer.tsx`

### API Routes (`/app/api/llm/`)
Each route is a POST endpoint returning JSON or SSE stream. Only `/chat` streams; others return structured JSON.

## Important Patterns & Conventions

### When Working with AI Features
- Review v0 generation spec: `/v0-generation-spec-prompt.md` for original design intent
- Professor Pine personality is core to brand - maintain encouraging, silly, patient tone
- All AI responses must be kid-safe; use fallbacks rather than exposing errors to users
- Natural language search (`/api/llm/query-pokeapi`) translates kid phrases → PokeAPI filters

### When Modifying Themes
- Edit CSS custom properties in `/app/globals.css` (lines 6-103)
- Use OKLCH color space for perceptually uniform colors
- Test all three themes: candy (root), space (`.theme-space`), ocean (`.theme-ocean`)
- Custom animations defined: bounce-gentle, wiggle, float, sparkle (lines 156-212)

### When Adding Components
- UI primitives go in `/components/ui/` (follow shadcn pattern)
- Feature components go in `/components/`
- Use Server Components by default; add `"use client"` only when needed for state/effects
- Wrap pages with `<PageWrapper>` for consistent layout

### Security & Safety Considerations
- NEVER commit `.env.local` - API keys must stay private
- Only push `.env.example` with placeholder keys
- All OpenAI calls MUST go through `/app/api/llm/*` endpoints (never expose API key to client)
- Content filtering is defensive - pre-filter user input, post-filter AI output
- PokeAPI has no auth, but respect their rate limits (cache results when possible)

## Testing & Quality

This project was built with TDD principles but test files are not currently in the repository. When adding tests:
- Use Vitest + React Testing Library (declared in README)
- Add contract tests for API endpoints
- Add integration tests for user journeys
- Use MSW for mocking API responses

## External Services

### PokeAPI
- Base URL: https://pokeapi.co/api/v2/
- No authentication required
- Returns data for 151 original Pokémon (Generation I)
- Sprites hosted on raw.githubusercontent.com

### OpenAI API
- Model: gpt-5-nano (reasoning model, fast and cost-effective)
- Key stored in: `OPENAI_API_KEY` environment variable
- Integration via: Vercel AI SDK (`@ai-sdk/openai`)
- Streaming: `streamText()` for chat
- Structured: `generateObject()` for quiz/story/search

## Key Files Reference

- `/app/api/llm/chat/route.ts` - Streaming chat with Professor Pine (has best examples of safety filtering)
- `/app/globals.css` - Theme CSS variables + custom animations
- `/components/theme-provider.tsx` - Theme state management
- `/lib/utils.ts` - cn() utility for Tailwind class merging
- `/next.config.ts` - Image domain allowlist
- `/v0-generation-spec-prompt.md` - Original design specification
- `/architecture/README.md` - Comprehensive architecture documentation

## Deployment Notes

Optimized for Vercel deployment:
- Set `OPENAI_API_KEY` in project environment variables
- SSR + API routes work out of box
- Image optimization handles Pokemon sprites automatically
- Edge runtime compatible (if needed for lower latency)

## Areas for Future Enhancement

Based on architecture analysis (`/architecture/README.md:566-608`):
- PokeAPI response caching to reduce redundant requests
- Service worker for offline support
- Unit/E2E tests (Vitest, Playwright)
- Error boundaries for graceful failure handling
- Loading skeletons instead of spinners
- Expand beyond 151 Pokémon (all generations)
- Multi-language support
- Analytics/monitoring integration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/donbr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/donbr)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
