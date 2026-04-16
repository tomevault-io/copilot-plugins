## computer-store-ks

> > Think carefully and implement the most concise solution that changes as little code as possible.

# CLAUDE.md

> Think carefully and implement the most concise solution that changes as little code as possible.

## Project Overview

Computer Store KS is the website for a computer repair shop in Topeka, Kansas.

**Live Site:** https://computerstoreks.com
**Hosting:** Render (free tier, kept alive by UptimeRobot)
**Database:** Supabase (PostgreSQL) — 4 tables
**Auth:** Google OAuth (single employee: `contact@computerstoreks.com`)

### What the site does
- **Public site** — 20 pages: homepage, about, contact, services (12 pages), reviews, in-store PCs gallery, photo gallery, why-linux, protection plans
- **Admin panel** — Behind Google login: manage in-store computers (add/edit/archive/stock/flyers), manage photo gallery, activate sales/discounts
- **Contact form** — Multi-layered spam protection + Cloudflare Turnstile, sends emails via Resend

## Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Next.js | 14 (App Router) | React framework |
| TypeScript | 5.x (strict mode) | Type-safe JavaScript |
| React | 18 | UI components |
| Tailwind CSS | 3.x | All styling (only styling system) |
| Supabase | Latest | PostgreSQL database, auth, image storage |
| Resend | Latest | Transactional email (contact form) |
| Sharp | Latest | Server-side image processing (WebP, thumbnails) |
| Zod | 4.x | Form/API input validation |
| Cloudflare Turnstile | Latest | CAPTCHA for contact form |
| Node.js | 22.x | Runtime (matches Render) |
| npm | Latest | Package manager (matches Render) |

## Authentication

Single-employee Google OAuth via Supabase Auth.

- **Authorized email:** `contact@computerstoreks.com` (defined in `lib/constants.ts` as `AUTHORIZED_EMAIL`)
- **Login flow:** `/login` → Google → `/auth/callback` → validates email → session cookie → `/admin`
- **Protection:** `middleware.ts` checks every request. Public routes pass through. Admin routes require valid session.
- **No roles, no RBAC, no customer accounts.** One employee, one login.

## Database

Supabase PostgreSQL — 4 tables:

| Table | Purpose |
|-------|---------|
| `gallery_computers` | Computers for sale (name, price, specs, images, stock, active/archived) |
| `gallery_sales` | Sale/discount definitions (type, percent, categories, active flag) |
| `photo_gallery` | Photos in the public gallery |
| `user_profiles` | Employee profile (created on first login) |

- **RLS enabled** on all tables — never disable it
- **Images** stored in Supabase Storage (`gallery-images` bucket)
- `anon` key = client-side (respects RLS). `service_role` key = server-side only (bypasses RLS).

## Environment Variables

```bash
# Supabase (REQUIRED)
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...              # Server-side only

# Email (REQUIRED for contact form)
RESEND_API_KEY=re_...
NOTIFICATION_EMAIL=contact@computerstoreks.com

# Site URL (REQUIRED for OAuth redirects)
NEXT_PUBLIC_SITE_URL=http://localhost:3000     # Production: https://computerstoreks.com

# Cloudflare Turnstile (optional in dev)
NEXT_PUBLIC_TURNSTILE_SITE_KEY=1x00000000000000000000AA
TURNSTILE_SECRET_KEY=
```

See `.env.example` for full documented list. **Never commit real keys.**

## Development

```bash
npm install              # Install dependencies
npm run dev              # Dev server at localhost:3000
npm run build            # Production build (same as Render)
npm run lint             # ESLint
npm run type-check       # TypeScript strict mode check
npm ci && npm run build  # Exact Render build command — test before pushing
```

**Tool rules:**
- `npm` only — do NOT use bun/yarn/pnpm (must match Render)
- Node.js 22.x (pinned in `.nvmrc` and `package.json` engines)
- Only `package-lock.json` should exist (no yarn.lock, bun.lockb, etc.)

**Code patterns:**
- TypeScript strict mode — no `// @ts-ignore`
- All styling is Tailwind — no CSS modules, no separate stylesheets
- Server Components by default. `'use client'` only for interactivity
- Business logic in `lib/` modules — API routes are thin handlers
- Every file has a JSDoc header comment (see `docs/comment-style.md`)

## Git & Deployment

Work in `Production` branch → test locally → get user approval → push to deploy.

```bash
git checkout Production
npm run dev                    # Test at http://localhost:3000
# Share localhost link with user for review
# WAIT for explicit user approval before pushing
npm run build                  # Verify production build passes
git add <files>
git commit -m "feat: description"
git push origin Production     # Render auto-deploys — LIVE immediately
```

**There is no staging environment.** Pushing to Production goes live immediately. Always test locally and get approval first.

**Commit convention:** `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `test:`

### Render
- **One service:** `Computer_Store_KS` (deploys from `Production` branch)
- **Free tier**, kept alive by UptimeRobot (5-min pings)
- **Health check:** `/api/health`
- **Node.js 22** via `NODE_VERSION` env var
- **Config:** `render.yaml`

### Deploy Checklist
1. `npm run dev` — test at http://localhost:3000
2. Share localhost link with user, get approval
3. `npm run build` passes
4. Push to `Production`
5. Check Render build status
6. Verify `/api/health` on live site

## Reference Docs

Read these when working on specific areas:

| Doc | When to read |
|-----|-------------|
| `docs/architecture.md` | Understanding the codebase structure, key systems, and how they connect |
| `docs/api-reference.md` | Working on API routes or debugging API behavior |
| `docs/comment-style.md` | Adding new files or functions — follow the commenting conventions |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MatthewMcManness) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
