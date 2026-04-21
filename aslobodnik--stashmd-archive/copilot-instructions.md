## stashmd-archive

> "Your stash for Claude" - the simplest way to share and discover Claude skills.

# Stashmd

"Your stash for Claude" - the simplest way to share and discover Claude skills.

## Stack

- **Framework:** Next.js 15 with TypeScript (App Router)
- **Hosting:** Vercel
- **Database:** Supabase (project: tmdzhwqcubpbixbppqoe)
- **Domain:** stash.md

## Development

```bash
npm run dev      # Start dev server on localhost:3000
npm run build    # Build for production
npm run seed     # Seed test data (requires schema to be applied)
```

## Working Style

When working on this project, use agents for parallel tasks whenever possible:
- Research tasks: Use Explore agent
- Multi-file changes: Split into parallel agents
- Build/lint/test: Run concurrently when independent

---

## Core Concept

**Zero-friction skill sharing:**

1. Anyone can share skills (no auth required)
2. GitHub login = verified badge (optional)
3. Pretty website for discovery
4. Claude skill for interaction

**Verified vs Unverified:**
- Verified: Logged in via GitHub → shows `@username ✓`
- Unverified: No login → shows `anonymous`

---

## Current State

**Working:**
- Next.js app structure
- GitHub OAuth via Supabase (auth cookies persist correctly)
- Basic pages: `/`, `/login`, `/settings`, `/skill/[slug]`
- D2-inspired CSS in `globals.css` (styled-jsx works in App Router)
- Anonymous skill submissions (POST /api/skills without auth)
- Verified badge UI for authenticated users
- `is_verified` field in database schema
- Settings page with API key management

**Ready for:**
- Deploy to Vercel

---

## Project Structure

```
stashmd/
├── app/
│   ├── layout.tsx
│   ├── page.tsx            # Uses HomePage component
│   ├── globals.css         # D2 design system (working)
│   ├── login/page.tsx
│   ├── settings/page.tsx   # Token management
│   ├── skill/[slug]/page.tsx
│   └── api/
│       ├── skills/route.ts
│       ├── skills/[slug]/route.ts
│       └── settings/route.ts
├── components/
│   ├── Header.tsx
│   ├── HomePage.tsx
│   └── SkillCard.tsx
├── lib/
│   ├── database.types.ts
│   ├── hooks/useAuth.ts
│   └── supabase/
├── supabase/schema.sql
└── PLAN.md                 # Full design doc
```

---

## Database Schema (Simplified)

```sql
-- Users (optional, for verified contributors)
users (
  id uuid primary key,
  github_id text unique,
  username text,
  avatar_url text,
  token text unique,  -- for verified uploads
  created_at timestamp
)

-- Skills
skills (
  id uuid primary key,
  content_hash text unique,     -- SHA256 dedup
  name text,
  description text,
  content text,                 -- full SKILL.md
  author_id uuid,               -- null if unverified
  author_username text,         -- null if unverified
  is_verified boolean default false,
  install_count int default 0,
  created_at timestamp
)
```

---

## API

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/api/skills` | GET | None | List skills |
| `/api/skills` | POST | Optional | Create single skill (verified if token) |
| `/api/skills/bulk` | POST | Optional | Create multiple skills at once |
| `/api/skills/[slug]` | GET | None | Get skill by slug |
| `/api/skills/[slug]/download` | POST | None | Increment install count |
| `/api/settings` | GET | Required | Get user's API key |
| `/api/settings` | POST | Required | Regenerate API key |

---

## NEXT STEPS (Priority Order)

### 1. Deploy to Vercel
- Push to GitHub
- Connect to Vercel
- Add env vars: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`
- Update Supabase Site URL to `https://stash.md`
- Add `https://stash.md/**` to Redirect URLs

### 3. Update Claude Stash Skill
Update `~/.claude/skills/stash/SKILL.md` to use the bulk endpoint:
- `POST /api/skills/bulk` accepts `{"skills": [{name, content}, ...]}`
- Optional `Authorization: Bearer TOKEN` for verified uploads

---

## Claude Skill Location

The stash skill is at: `~/.claude/skills/stash/SKILL.md`

Commands:
- `share my [skill] to stash` → uploads (anonymous or verified)
- `install [skill] from stash` → downloads
- `search stash for [query]` → searches

---

## Key Files to Edit

1. **CSS fix:** `app/globals.css` or `components/*.tsx`
2. **Schema:** `supabase/schema.sql` + run in Supabase dashboard
3. **API:** `app/api/skills/route.ts`
4. **UI badge:** `components/SkillCard.tsx`
5. **Skill detail:** `app/skill/[slug]/page.tsx`

---

## References

- **PLAN.md** - Full design doc with wireframes
- **Supabase Dashboard:** https://supabase.com/dashboard/project/tmdzhwqcubpbixbppqoe
- **Design:** D2 (Diablo 2) stash aesthetic - parchment, metal frames, gold accents

---

## Prompt for Continuation

When returning to this project, use this prompt:

```
Continue building stashmd. Read CLAUDE.md for context.
Next step: Deploy to Vercel (see NEXT STEPS section).
```

---

## Startup Checklist

When starting work on this project:

1. `cd /Users/slobo/Documents/coding/stashmd`
2. `npm run dev` - Start dev server
3. Verify http://localhost:3000 loads
4. Check CSS is rendering (parchment background, styled cards)
5. If CSS broken, see "Known Issues" above

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aslobodnik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
