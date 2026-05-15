## firstpledgeplatform

> Trust-as-a-Service platform for AI-powered product ingredient safety verification.

# FirstPledgePlatform — Claude Code Instructions

## Project Overview

Trust-as-a-Service platform for AI-powered product ingredient safety verification.
Full-stack TypeScript: React 18 + Express.js + Supabase (PostgreSQL) + Drizzle ORM.
AI providers: Groq (primary), OpenAI, Gemini — configured via `AI_PROVIDER` env var.
Deployed on Vercel. Auth via Supabase JWT with API key fallback.

## Tech Stack

- **Frontend**: React 18, Vite (port 5173), Tailwind CSS, Shadcn UI, Wouter, React Query
- **Backend**: Express.js (TypeScript, port 3000), proxied through Vite in dev
- **Database**: Supabase (PostgreSQL), Drizzle ORM (`shared/schema.ts`)
- **Dev command**: `npm run dev` — starts Express on :3000, Vite proxy on :5173
- **Build**: `npm run build` — Vite + esbuild bundle for Vercel
- **Deploy**: Vercel (`vercel.json`), API entry at `api/index.ts`

## Key Architecture

- `server/index.ts` — Express routes (public: GET products; admin: POST/PATCH/DELETE products, vetting)
- `server/middleware/auth.ts` — Supabase JWT verification + API key fallback
- `server/services/aiVettingService.ts` — Multi-layer ingredient analysis: cache → EWG → research → AI
- `server/storage/supabaseStorage.ts` — Product/ingredient persistence layer
- `shared/schema.ts` — Drizzle schema: products, ingredients, ingredient_analyses, user_profiles
- `client/src/pages/` — Home, ProductDetail, AdminDashboard, ProductForm
- `client/src/components/auth/` — AuthProvider (Supabase session), ProtectedRoute

## Environment Variables Required

```
DATABASE_URL, SUPABASE_URL, SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY
AI_PROVIDER=groq|openai|gemini
GROQ_API_KEY / OPENAI_API_KEY / GEMINI_API_KEY
CLIENT_ORIGIN=http://localhost:5173
NODE_ENV=development|production
```

## gstack

Use `/browse` from gstack for all web browsing. Never use `mcp__claude-in-chrome__*` tools.

Available skills:
`/office-hours`, `/plan-ceo-review`, `/plan-eng-review`, `/plan-design-review`,
`/design-consultation`, `/design-shotgun`, `/design-html`, `/review`, `/ship`,
`/land-and-deploy`, `/canary`, `/benchmark`, `/browse`, `/open-gstack-browser`,
`/qa`, `/qa-only`, `/design-review`, `/setup-browser-cookies`, `/setup-deploy`,
`/retro`, `/investigate`, `/document-release`, `/codex`, `/cso`, `/autoplan`,
`/plan-devex-review`, `/devex-review`, `/careful`, `/freeze`, `/guard`, `/unfreeze`,
`/gstack-upgrade`, `/learn`

### Recommended skill order for this project

| Situation | Skills to run |
|-----------|---------------|
| New feature | `/office-hours` → `/plan-eng-review` → build → `/review` → `/qa http://localhost:5173` → `/ship` |
| Bug fix | `/investigate` → `/careful` → fix → `/review` → `/ship` |
| Security concern | `/cso` → `/freeze server/` → fix → `/review` → `/ship` |
| Pre-deploy | `/cso` then `/qa https://<vercel-url>` then `/canary https://<vercel-url>` |
| Design change | `/plan-design-review` → build → `/design-review` → `/qa http://localhost:5173` |

### High-priority first runs

1. `/cso` — security audit (auth middleware, admin routes, AI input, external API calls)
2. `/qa http://localhost:5173` — QA the full ingredient vetting workflow in real browser
3. `/ship` — sets up test framework (project currently has no tests)

## Testing

Run: `npm test` (Vitest, 15 tests, ~1s)
Coverage: `npm run test:coverage`
Test directory: `tests/server/`
See [TESTING.md](TESTING.md) for full conventions.

- New function → write a corresponding test
- Bug fix → write a regression test before fixing
- Security fix → write a test proving the vulnerability is closed
- Never commit code that makes existing tests fail

## Skill routing

When the user's request matches an available skill, ALWAYS invoke it using the Skill
tool as your FIRST action. Do NOT answer directly, do NOT use other tools first.
The skill has specialized workflows that produce better results than ad-hoc answers.

Key routing rules:
- Product ideas, "is this worth building", brainstorming → invoke office-hours
- Bugs, errors, "why is this broken", 500 errors → invoke investigate
- Ship, deploy, push, create PR → invoke ship
- QA, test the site, find bugs → invoke qa
- Code review, check my diff → invoke review
- Update docs after shipping → invoke document-release
- Weekly retro → invoke retro
- Design system, brand → invoke design-consultation
- Visual audit, design polish → invoke design-review
- Architecture review → invoke plan-eng-review
- Save progress, checkpoint, resume → invoke checkpoint
- Code quality, health check → invoke health

---
> Source: [maitreyupatel/FirstPledgePlatform](https://github.com/maitreyupatel/FirstPledgePlatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
