## galasteps

> GalaSteps is a bilingual (EN/ES) early childhood development app for ages 0-5, tracking milestones and providing AI-powered parenting guidance. Built with Next.js (web), Expo/React Native (mobile), and Supabase.

@AGENTS.md

# GalaSteps — Hermes Agent Context

## What This Is

GalaSteps is a bilingual (EN/ES) early childhood development app for ages 0-5, tracking milestones and providing AI-powered parenting guidance. Built with Next.js (web), Expo/React Native (mobile), and Supabase.

**Repo:** `b1tf0rbyt3s/galasteps`
**Monorepo:** pnpm workspaces + Turborepo
**Web:** Next.js 16 on Vercel (galasteps.com)
**Mobile:** Expo SDK 55 via EAS
**Database:** Supabase (`wmweqtpzaqipxpoyfoaf` — dev)

## TypeScript — Incremental Compilation

**CRITICAL:** Running bare `tsc --noEmit` on this monorepo will OOM or take 5+ minutes. Always use incremental mode:

```bash
# Correct — incremental, uses cache
tsc --noEmit --incremental --tsBuildInfoFile .tsbuildinfo

# Also correct — turbo handles parallelism
pnpm typecheck

# WRONG — will crash or hang
tsc --noEmit
```

If your run takes more than 60 seconds, you're doing it wrong. Kill and retry with `--incremental`.

## Work Patterns

### Before Starting Any Task
1. Read the issue/task description fully
2. Check which files are involved: `git diff main --name-only` or look at the issue labels
3. Plan your approach in a comment before writing code
4. Run `pnpm typecheck` (incremental) to establish baseline — fix nothing that wasn't already broken

### During Development
1. Make focused, minimal changes — one concern per commit
2. Run typecheck after each logical change, not just at the end
3. If you break something unrelated, revert and isolate
4. Use conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `test:`

### Before Completing
1. `pnpm typecheck` — must pass clean
2. `pnpm lint` — must pass clean
3. Verify your changes make sense as a coherent diff
4. Write a summary comment on the issue explaining what you did and why

## What NOT To Do

**Do NOT make Paperclip API calls.** You are an agent managed by Paperclip — you do not need to call the Paperclip API. Your job is to work on the GalaSteps codebase, not manage yourself. If you find yourself writing `curl` commands to `localhost:3100` or any Paperclip endpoint, STOP — that's not your task.

**Do NOT:**
- Run bare `tsc --noEmit` (use `--incremental`)
- Push directly to `main` — always use feature branches
- Add secret keys to client-side code or `EXPO_PUBLIC_` vars
- Include child PII in AI prompts (COPPA)
- Remove `"packageManager": "pnpm@10.28.0"` from root package.json
- Change `node-linker=hoisted` in `.npmrc`
- Use npm or yarn — this is pnpm only
- Create separate CSS files — use Tailwind utilities inline

## Content Pipeline (Sage-specific)

Sage writes proposed content to the `content_staging` table in Supabase dev. Sean reviews via the admin platform.

| Status | Meaning |
|--------|---------|
| `pending` | Newly staged, awaiting review |
| `approved` | Moves to live tables |
| `rejected` | Archived |
| `needs_revision` | Sean left notes — Sage picks up |

Content types: `activity`, `milestone`, `domain_update`, `tip`, `resource`
Domains: social, creative, physical, language, math, science
Age bands: 0-3mo, 3-6mo, 6-9mo, 9-12mo, 12-18mo, 18-24mo, 2-3yr, 3-4yr, 4-5yr

All content must have source citations (CDC/AAP/WHO) and bilingual coverage (EN + ES).

## COPPA Compliance (Kira-specific)

This is a children's app. COPPA compliance is non-negotiable:
- No child PII in analytics events (Vercel Analytics + `sanitizeEventProperties`; COPPA-blocked routes skip scripts)
- No child names/photos/DOB in AI prompts — use anonymized age + milestone data
- RLS on all tables — verify with `supabase inspect db policies`
- Consent gates before data collection
- Mobile: no product analytics; structured errors via `@galakids/logger` scrub only when used

## Key Supabase Tables

| Table | Purpose |
|-------|---------|
| `milestones` | CDC developmental milestones by age/domain |
| `activities` | Parent-facing activities linked to milestones |
| `milestone_activities` | Junction table |
| `children` | Child profiles (parent_id FK, PII-sensitive) |
| `profiles` | Parent user profiles |
| `content_staging` | Sage's content pipeline staging area |

## Monorepo Structure

```
galasteps/
├── apps/
│   ├── web/          # Next.js 16 (App Router, Tailwind v4, shadcn/ui, next-intl)
│   └── mobile/       # Expo SDK 54 (React Native 0.81, Expo Router v6)
├── packages/
│   ├── shared/       # @galasteps/shared — types, theme, constants, i18n
│   └── logger/       # @galakids/logger — COPPA scrub, structured logging (vendored from GalaKids)
├── turbo.json
├── pnpm-workspace.yaml
└── package.json
```

## External Services

| Service | Purpose |
|---------|---------|
| Vercel | Web hosting (galasteps.com) |
| Supabase | Database, auth, storage, pgvector |
| EAS | Mobile builds (Expo) |
| Mux | Video content |
| Vercel Analytics / Speed Insights | Web product analytics & Web Vitals (COPPA-gated routes) |
| Vercel Observability Plus | Logs, traces, metrics (Vercel dashboard) |
| `@galakids/logger` (`packages/logger`) | Structured server logging, COPPA scrub / route drop |
| Cloudinary | Image/asset CDN |
| Claude Haiku (via AI Gateway) | AI chat via /api/chat — routes through `@ai-sdk/gateway` `createGateway()`, models DB-configurable via `ai_model_config` |

## PostgREST Cross-Schema Rules

PostgREST `db_schemas` is configured to include only `public`. This means:

- **NEVER use embedded resource selects** (the colon syntax like `auth_user:user_id(email)`) that reference `auth.users` or any `auth.*` table. PostgREST cannot resolve cross-schema foreign keys.
- Violating this causes **PGRST200** errors that are **opaque in production** — RSC streaming swallows them, Vercel logs show 200 with no error.
- **Fix pattern:** Use separate parallel queries with the admin client instead of embedded selects crossing schema boundaries.

---

## RSC Error Debugging

When investigating React Server Component error boundaries:

- **NEVER rely on Vercel runtime logs or HTTP status codes.** RSC streaming errors return HTTP 200 and the error message is invisible in production.
- **MUST render the page in a browser** (agent-browser or Chrome MCP) to verify error boundaries.
- Add explicit `try/catch` with `console.error` in layout files when debugging opaque errors.

---

## Agent Pre-Push Checklist

Before pushing any code change, agents MUST verify:

1. **No PostgREST cross-schema selects:** `grep -rn 'auth_user:' --include='*.ts' src/` — must return zero results
2. **Nullable fields handled:** DB columns typed as `| null` must be checked with `=== null`
3. **TypeScript clean:** `pnpm typecheck` passes (always use `--incremental`)
4. **Lint clean:** `pnpm lint` passes
5. **Supabase Realtime topic uniqueness:** If creating/modifying Realtime hooks, verify no two hooks share the same channel topic string

---

## Chain-of-Thought Logging

When working on complex tasks, log your reasoning as comments on the issue. This creates an audit trail that helps Sean understand your decisions and helps other agents learn from your approach. Structure:

```
## Approach
[What you're going to do and why]

## Changes Made
[List of files changed with brief explanation]

## Verification
[What you checked: typecheck, lint, manual testing]

## Open Questions
[Anything that needs human review or decision]
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/b1tf0rbyt3s) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
