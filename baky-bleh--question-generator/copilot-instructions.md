## question-generator

> > Language learning + math/video education platform. Duolingo-style exercises

# LinguaLeap — Language & Math Learning Platform

> Language learning + math/video education platform. Duolingo-style exercises
> for language, Khan Academy-style video lessons + quizzes for math.
> Free 3-month trial → ad-supported → premium subscription.

## Quick Context

- **Backend**: FastAPI (Python 3.12+) · PostgreSQL · Redis · S3
- **Mobile**: React Native + Expo SDK 52+ · Expo Router · TypeScript
- **Web**: Next.js 14+ (App Router) · TypeScript · Tailwind CSS
- **Video**: Local mp4 (dev) → Mux (production), abstracted via VideoStorage
- **Payments**: RevenueCat (mobile) · Stripe (web) · Google AdMob/AdSense
- **Infra**: Docker Compose (local) · Railway (API) · Vercel (web) · Supabase (DB) · Upstash (Redis)

## Documentation Map

> **IMPORTANT**: Do NOT put detailed docs in this file. This file is the index.
> Detailed documentation lives in `/docs/`. Read the relevant files before working.

| Document | What's In It | When To Read |
|----------|-------------|--------------|
| [docs/architecture.md](docs/architecture.md) | System design, service boundaries, data flow | Always read first |
| [docs/conventions.md](docs/conventions.md) | Code style, naming, patterns, file structure | Always read first |
| [docs/api-contracts.md](docs/api-contracts.md) | All API endpoints, request/response shapes | When building any client or API work |
| [docs/database-schema.md](docs/database-schema.md) | All tables, relationships, migrations | When touching data layer |
| [docs/phase-status.md](docs/phase-status.md) | What's built, what's in progress, what's next | Always read first |
| [docs/agent-teams.md](docs/agent-teams.md) | Team configs, spawn prompts per phase | When starting a new agent team |
| [docs/decisions.md](docs/decisions.md) | Architecture Decision Records (ADRs) | When making or questioning a design choice |
| [docs/video-architecture.md](docs/video-architecture.md) | Video system design, storage abstraction, Mux migration | When working on video/math features |

## Project Structure

```
lingualeap/
├── CLAUDE.md                    ← You are here (index only)
├── docs/                        ← All detailed documentation
│   ├── architecture.md
│   ├── conventions.md
│   ├── api-contracts.md
│   ├── database-schema.md
│   ├── phase-status.md
│   ├── agent-teams.md
│   └── decisions.md
├── api/                         ← FastAPI backend
│   ├── src/
│   │   ├── core/                ← Config, DB connection, deps, middleware
│   │   ├── auth/                ← JWT, OAuth2, password hashing
│   │   ├── courses/             ← Course models, content pipeline, S3
│   │   ├── lessons/             ← Lesson serving, exercise validation
│   │   ├── progress/            ← Completion tracking, XP calculation
│   │   ├── srs/                 ← Spaced repetition engine (SM-2)
│   │   ├── streaks/             ← Streak logic, timezone handling
│   │   ├── gamification/        ← Achievements, leaderboards, leagues
│   │   ├── video/               ← Video storage abstraction, upload, progress
│   │   ├── subscriptions/       ← RevenueCat webhooks, feature flags
│   │   └── notifications/       ← Push notification triggers
│   ├── tests/
│   ├── alembic/                 ← Database migrations
│   ├── scripts/                 ← Content build, S3 upload, utilities
│   └── pyproject.toml
├── mobile/                      ← React Native + Expo
│   ├── app/                     ← Expo Router screens
│   │   ├── (auth)/              ← Login, signup, onboarding
│   │   ├── (tabs)/              ← Main tab navigator (home, learn, profile)
│   │   └── (lesson)/            ← Lesson player (modal stack)
│   ├── src/
│   │   ├── components/          ← Reusable UI components
│   │   ├── services/            ← API client, offline, auth, purchases, ads
│   │   ├── hooks/               ← Custom React hooks
│   │   ├── stores/              ← Zustand state management
│   │   └── utils/               ← Helpers, constants, types
│   └── package.json
├── web/                         ← Next.js
│   ├── app/
│   │   ├── (marketing)/         ← Landing, pricing, course catalog (SSR)
│   │   ├── (app)/               ← Authenticated SPA (dashboard, lessons)
│   │   └── api/                 ← API routes (auth proxy, webhooks)
│   ├── src/
│   │   ├── components/
│   │   ├── services/
│   │   └── hooks/
│   └── package.json
├── shared/                      ← Shared between mobile & web
│   ├── api-client/              ← Auto-generated TypeScript API client
│   ├── types/                   ← Shared TypeScript types
│   └── constants/               ← Shared constants (XP values, etc.)
├── content/                     ← Course content source files
│   ├── courses/                 ← Course JSON definitions
│   ├── audio/                   ← Audio files for listening exercises
│   ├── videos/                  ← Video files (local dev) and metadata
│   ├── schemas/                 ← JSON schemas for exercise types
│   └── build.py                 ← Content build & validation script
├── infra/                       ← Infrastructure configs
│   ├── docker-compose.yml
│   ├── Dockerfile.api
│   └── .github/workflows/
└── .claude/                     ← Claude Code configuration
    ├── settings.json
    ├── hooks/
    │   ├── PreSession.sh
    │   ├── PostSession.sh
    │   └── TaskCompleted.sh
    └── scripts/
        └── doc_sync.py
```

## Rules For All Agents

### DO
- Read `docs/phase-status.md` and `docs/conventions.md` before writing any code
- Read `docs/api-contracts.md` before making or consuming any API endpoint
- Own your files — only edit files in your assigned directories
- Write tests alongside implementation (not after)
- Use existing patterns from the codebase — run `grep -r "pattern" src/` to find examples
- Commit with conventional commits: `feat(auth): add JWT refresh endpoint`

### DO NOT
- Edit files outside your assigned directory without explicit coordination
- Add new dependencies without noting them in your task completion message
- Change database schema without updating `docs/database-schema.md`
- Change or add API endpoints without updating `docs/api-contracts.md`
- Modify `CLAUDE.md` — only the doc-sync agent updates documentation
- Hardcode configuration values — use environment variables via `src/core/config.py`
- Skip type hints in Python or TypeScript types

### Documentation Updates
When you finish a task that changes the API, schema, or architecture:
1. Do NOT edit docs yourself
2. Instead, leave a `CHANGELOG_ENTRY` in your task completion:
   ```
   CHANGELOG_ENTRY: Added POST /api/v1/auth/refresh endpoint.
   Returns new access token given valid refresh token.
   Request: { refresh_token: string }
   Response: { access_token: string, expires_in: number }
   ```
3. The post-session hook will run the doc-sync agent to incorporate changes cleanly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Baky-bleh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
