## ai-blog

> **Pageo** is an AI-powered automated news blog built with Next.js 16 and Convex. It autonomously discovers trending topics, generates articles using Google Gemini, validates content through a critic-generator feedback loop, and publishes to a PWA-enabled frontend.

# AGENTS.md

## Project Overview

**Pageo** is an AI-powered automated news blog built with Next.js 16 and Convex. It autonomously discovers trending topics, generates articles using Google Gemini, validates content through a critic-generator feedback loop, and publishes to a PWA-enabled frontend.

## Tech Stack

- **Framework**: Next.js 16 (App Router), React 19, TypeScript
- **Database**: Convex (serverless)
- **Authentication**: Auth0
- **AI**: Google Gemini 2.5 Flash Lite via `@ai-sdk/google`
- **Search**: Tavily API
- **Styling**: Tailwind CSS 4 with CSS variables
- **Validation**: Zod

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── api/               # API routes (run-job, rss, archive-old)
│   ├── feed/              # Main news feed
│   ├── posts/[slug]/      # Article pages
│   └── bookmarks/         # Saved articles
├── components/ui/         # React components
└── lib/
    ├── ai/                # AI pipeline modules
    │   ├── pipeline.ts    # Main orchestrator
    │   ├── generator.ts   # Article generation
    │   ├── critic.ts      # Content validation
    │   ├── refinement.ts  # Iterative improvement
    │   ├── search.ts      # Tavily integration
    │   └── issues.ts      # Issue taxonomy
    ├── hooks/             # React hooks
    └── schemas/           # Zod schemas
convex/
├── schema.ts              # Database schema
├── posts.ts               # Post mutations/queries
├── audit.ts               # Pipeline logging
└── subscribers.ts         # Newsletter
```

## AI Content Pipeline

The pipeline runs hourly via GitHub Actions:

1. **Discovery** (`extractTopicFromTavily.ts`) - Queries Tavily for trending news, avoids recent topics
2. **Search** (`search.ts`) - Fetches additional context from Tavily
3. **Generation** (`generator.ts`) - Creates structured article using Gemini
4. **Critique** (`critic.ts`) - Separate Gemini instance validates content
5. **Refinement** (`refinement.ts`) - Loops up to 3 times until publish/reject decision

### Issue Categories

Content is evaluated across 5 categories with severity levels:

| Category | High Severity (Reject) | Medium (Fix) | Low (Pass) |
|----------|----------------------|--------------|------------|
| Safety | violence, hate_speech, extremism | political_persuasion | - |
| Factual | - | unsupported_numbers, hallucinated_events | speculative_language |
| Quality | - | ai_repetition, weak_intro | grammar_flow |
| SEO | - | - | title_clarity, heading_structure |
| Compliance | illegal_products, personal_data | clickbait_financial | - |

## Database Schema (Convex)

**posts**: `slug`, `title`, `summary`, `content[]`, `sources[]`, `tags[]`, `status`, `publishedAt`, `readingTime`

**audit**: `runId`, `stage`, `status`, `input`, `output`, `durationMs`

## Key Commands

```bash
npm run dev          # Start dev server
npm run build        # Production build
npm run test-cron    # Test pipeline locally
```

## Environment Variables

```
NEXT_PUBLIC_CONVEX_URL=     # Convex deployment URL
GOOGLE_GENERATIVE_AI_API_KEY=  # Gemini for generation
GEMINI_CRITIC_API_KEY=      # Gemini for critique (separate key)
TAVILY_API_KEY=             # News search
CRON_SECRET=                # API auth for cron jobs
OPENVERSE_CLIENT_SECRET& CLIENT_ID=  # Image sourcing
```

## Coding Conventions

- Use `@/` path alias for imports from `src/`
- CSS variables for theming: `var(--color-primary)`, `var(--color-background)`
- Content blocks use discriminated union: `{ type: "paragraph" | "heading", ... }`
- All AI outputs validated with Zod schemas
- Convex mutations/queries in `convex/` directory

## API Endpoints

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/api/run-job` | POST/GET | Bearer token | Trigger content pipeline |
| `/api/archive-old` | GET | Bearer token | Archive posts >30 days |
| `/api/rss` | GET | None | RSS feed |

## GitHub Actions

- **cron.yml**: Hourly pipeline trigger with 3 retries
- **archive.yml**: Weekly cleanup of old posts

---
> Source: [NewGuy20-dev/AI-blog](https://github.com/NewGuy20-dev/AI-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
