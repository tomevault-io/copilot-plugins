## castorworks-ng

> **IMPORTANT — CastorWorks vs CastorWorks-NG (do not change existing CastorWorks):**

# CastorWorks

**IMPORTANT — CastorWorks vs CastorWorks-NG (do not change existing CastorWorks):**
- **Do not change any existing configuration from CastorWorks.** The previous CastorWorks working environment is **UNTOUCHABLE**.
- **Add new configuration** to support CastorWorks-NG in any file (nginx, containers, deploy scripts, env examples, etc.). Do not modify or remove original CastorWorks config.
- When in doubt: add new files, new server blocks, new hosts (e.g. devng.castorworks.cloud, studiong.castorworks.cloud), or new sections; never alter existing CastorWorks (dev.castorworks.cloud, port 8000, original stack) config.

**IMPORTANT — Execute, do not just recommend:**
- **Do not recommend that the user run commands;** run them yourself (deploy, restart, scp, ssh, scripts, npm, etc.) to ensure speed. Only document commands in runbooks for reference or when execution is not possible (e.g. user-only steps like entering a password in a UI).

---

## Overview

CastorWorks is a TypeScript + React **construction / engineering project management platform** built on Vite, Supabase, shadcn/ui (Radix), and i18next (multi-language).

Core capabilities span Projects, Financials, Procurement/Materials, Schedule/Calendars, Daily logs/photos, and role-based portals (including Architect + Client Portal experiences).

## Architecture

### Frontend

- **React 19 + React Router 7** SPA built with **Vite 7**
- **UI system**: Tailwind + shadcn/ui components (`src/components/ui/*`) using Radix primitives
- **Data fetching**: TanStack Query for server state; hooks encapsulate Supabase queries/mutations
- **Forms**: React Hook Form + Zod schemas (runtime validation)
- **i18n**: i18next + react-i18next (en-US, es-ES, fr-FR, pt-BR)

### Key patterns

- **Feature folders** under `src/components/*` (Architect, ClientPortal, Projects, Financial, Schedule, Settings, etc.)
- **Custom hooks** in `src/hooks/*` for Supabase access, derived state, and side effects
- **Utilities** in `src/utils/*` for pure functions (e.g., date format detection/formatting)

### Calendars (recently modernized)

- Primary calendar primitive is `src/components/ui/calendar.tsx` (shadcn Calendar wrapper)
- Uses **react-day-picker v9** classNames + component slots; supports localized caption formats
- New companion component: `src/components/ui/calendar-with-events.tsx`

### Backend (Supabase)

- **Self-hosted Supabase** (see `AGENTS.md` for SSH, `docker exec`, and applying migrations).
- **Typical API URLs**: `https://dev.castorworks.cloud` (main / legacy stack) or `https://devng.castorworks.cloud` (CastorWorks-NG).
- Runs on the provider in **Docker** (not in the local dev environment).
- **Postgres containers on the host** (for `docker exec` / `psql`): `supabase-castorworks-db` (main stack) or `castorworks-ng-db` (NG stack). The name `supabase-db` is obsolete and not used on current production.
- Do not install any tool to work with Supabase locally; treat it as a remote dependency
- Uses Auth + Postgres + Storage + Realtime + Edge Functions
- **RLS is mandatory** on tables; Storage is also protected via policies and helper functions

## Setup & Installation

### Prerequisites

- Node.js (recommended v18+) and npm
- Supabase credentials for the target environment

### Install

```bash
npm install
```

### Environment variables

Create `.env.local` (do not commit) with:

```env
VITE_SUPABASE_URL=https://dev.castorworks.cloud
VITE_SUPABASE_ANON_KEY=<your-anon-key>
```

### Important dev environment limitation

Local development **does not have access** to run Supabase tooling (Supabase CLI) or connect via `psql`. Treat Supabase as a remote managed dependency reachable only via HTTPS (`VITE_SUPABASE_URL`).

## Development Workflow

### Start the app

- `./castorworks.sh start` (recommended wrapper)
- `./castorworks.sh restart`
- `./castorworks.sh clean` (clean build + restart)

### Useful npm scripts

All these scripts can be run via `npm run <script>` to ensure code quality, run tests, and perform maintenance tasks. NO bugs should be merged without running the relevant scripts.
GitHub CI also runs these scripts on each PR to enforce quality gates.
You must run the relevant scripts locally before pushing code.

- **Quality**: `npm run lint` | `npm run lint:hooks` | `npm run validate:json` | `npm run ci`
- **Tests**: `npm test` | `npm run test:run` | `npm run test:coverage` | `npm run test:e2e`
- **Security**: `npm run test:security` | `npm run test:rls:policies` | `npm run precommit`
- **Storage maintenance**: `npm run backfill:signed-urls:dry` | `npm run backfill:signed-urls:apply`

### UX / i18n

- Codebase supports i18n - English, Portuguese, Spanish, French
- Use `src/locales/` for JSON bundles
- Always generate new strings for the 4 Languages and verify in `src/locales/`
- No placeholders - all strings must be in the i18n system
- No hardcoded strings in components in any of the Languages above

### Visual development

- Design references: `context/design-principles.md`, `context/style-guide.md`

## Visual Testing Tool

- We do not use Playwright. We use Vercel agent-browser for all E2E and browser automation; do not add or use Playwright.
- Quick check after UI changes: for all sorts of testing (UI, testing, visual testing, end-to-end testing, browser testing, etc.), use the `agent-browser` tools. If you are unsure how to use `agent-browser`, please run:

```bash
agent-browser --help
```

This will provide you with a list of available commands and options to get started.

## Visual Testing Process

  1. Navigate to affected pages
  2. Verify layouts + a11y basics
  3. Capture a 1440px screenshot
  4. Check console for errors

## API Documentation

This repo is primarily a **frontend** that talks to **Supabase** (PostgREST + Storage + Edge Functions).

### Supabase client

- Client setup: `src/integrations/supabase/*`
- Environment-driven configuration via `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`

### Database access

- Prefer TanStack Query hooks to encapsulate all queries/mutations
- Keep query keys stable and invalidate on mutations
- Enforce authorization via **RLS helper functions** (`has_project_access`, `has_role`, etc.)

### Storage access

- Use signed URLs for private buckets (max 3600s)
- Avoid `getPublicUrl()` for private data
- Moodboard storage is handled via a dedicated bucket + helper function (see Recent Updates)

### Edge Functions

- Located at `supabase/functions/*`
- Must use `supabase/functions/_shared/authorization.ts` to verify access **before** DB work

## File Structure

```text
src/
  components/
    ui/                     shadcn/ui primitives (Button, Calendar, Dialog, ...)
    Architect/              architect portal components (meetings, tasks, moodboard, ...)
    ClientPortal/           client portal UI (schedule/calendar, tasks, dialogs, ...)
    Schedule/               schedule/calendar integrations and widgets
    Photos/                 photo gallery and uploads
    Settings/               admin/user settings panels
  hooks/                    Supabase data hooks + business logic hooks
  contexts/                 global contexts (auth/localization/etc.)
  utils/                    pure utilities (date format detection, etc.)
  locales/                  i18n JSON bundles (en-US, es-ES, fr-FR, pt-BR)
supabase/
  migrations/               DB + RLS + storage policy migrations
  functions/                Supabase Edge Functions (Deno)
docs/
  sprint-handbook/          developer handbook (architecture, patterns, security, testing, etc.)
```

**Import alias:** `@/*` → `src/*`

- **Developer handbook**: comprehensive sprint handbook added under `docs/sprint-handbook/` (architecture, patterns, Supabase, edge functions, testing, security, troubleshooting, automation).
- **Calendar modernization**:
  - Updated **react-day-picker** to `^9.13.0`.
  - `src/components/ui/calendar.tsx` migrated to the v9 classNames + component-slot approach (button variants, improved layout, better localization hooks).
  - New `src/components/ui/calendar-with-events.tsx` utility component.
  - Client Portal month calendar refactored to align with shadcn Calendar patterns.
- **Date formatting utilities**:
  - Introduced/updated date format detection utilities to support localized rendering in calendar/task forms.
- **Image uploads**:
  - Added `browser-image-compression` to compress images before upload (used by photo/moodboard flows).

### Database / RLS / Storage

- **Architect moodboards storage bucket**:
  - New migration creates `architect-moodboards` storage bucket and policies.
  - Helper function `has_moodboard_access(user_id, project_id)` introduced and then corrected to use the repo’s role model via `has_role(user_id, 'admin'::app_role)` (fixing an earlier “roles table” assumption).

## Important Notes

### Security Guidelines (CRITICAL)

#### Database security (RLS)

- ✅ **RLS enabled on ALL tables**
- ✅ Policies should use helper functions: `has_project_access()` / `has_role()`
- ✅ Roles live in `user_roles` using `app_role` (enum) — do not reintroduce a `roles` table assumption
- ✅ Admin-only access must be enforced at DB level: `has_role(auth.uid(), 'admin')`

Example:

```sql
CREATE POLICY "Users can view projects they have access to"
  ON projects FOR SELECT USING (has_project_access(auth.uid(), id));
```

#### Edge Functions security

- ✅ Verify access first using `supabase/functions/_shared/authorization.ts`
- ✅ Never expose service role keys in client code

#### Client-side security

- ✅ Use DOMPurify for any user-generated HTML rendered via `dangerouslySetInnerHTML`
- ✅ Use signed URLs for private storage (≤ 3600s)
- ✅ All sensitive config via `import.meta.env.VITE_*`

### CHANGELOG.md requirement

For any major feature/bugfix release, update `CHANGELOG.md` as the **last step** (Keep a Changelog + SemVer).

### What we do NOT use

Vue/Angular, Next.js, Jest, Cypress, Prettier, Husky, Webpack/Rollup/esbuild.

### Your role and responsibilities

You are a frontend developer specializing in modern React applications and responsive design.

## Focus Areas

- React component architecture (hooks, context, performance)
- Responsive CSS with Tailwind/CSS-in-JS
- State management (Redux, Zustand, Context API)
- Frontend performance (lazy loading, code splitting, memoization)
- Accessibility (WCAG compliance, ARIA labels, keyboard navigation)

## Approach

1. Component-first thinking - reusable, composable UI pieces
2. Mobile-first responsive design
3. Performance budgets - aim for sub-3s load times
4. Semantic HTML and proper ARIA attributes
5. Type safety with TypeScript when applicable

## Output

- Complete React component with props interface
- Styling solution (Tailwind classes or styled-components)
- State management implementation if needed
- Basic unit test structure
- Accessibility checklist for the component
- Performance considerations and optimizations

Focus on working code over explanations. Include usage examples in comments.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alex-macedo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alex-macedo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
