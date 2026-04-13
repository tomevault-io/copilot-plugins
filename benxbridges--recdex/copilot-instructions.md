## recdex

> Be token-efficient: prefer targeted reads (offset/limit) over full file reads, use Grep/Glob before Read, batch independent tool calls, avoid re-reading files already in context, keep responses concise, and use Edit over Write for existing files.

# RecDex (Recipe Index)

## Token Efficiency
Be token-efficient: prefer targeted reads (offset/limit) over full file reads, use Grep/Glob before Read, batch independent tool calls, avoid re-reading files already in context, keep responses concise, and use Edit over Write for existing files.

## What This Is
A recipe site built around actually cooking. People discover recipes here (browse, search, paste a URL) — but every design decision pushes toward getting them into cook mode and sharing what they made with friends. The whole funnel: Find → Cook → Share.

Tagline: "Be a better cook."
URL: recipeindex.org

## Tech Stack
- **Framework**: Next.js 16 (App Router, React 19, TypeScript 5, React Compiler on)
- **Database**: Supabase (Postgres). Anon key for reads, service key for writes (RLS blocks anon writes)
- **Styling**: Inline CSS-in-JS with theme tokens from `src/app/lib/theme.ts` — `C.*` for colors, `SERIF`/`SANS`/`MONO` for fonts. Tailwind is installed but barely used.
- **AI**: Claude Sonnet (recipe extraction, analysis), OpenAI (backup), Gemini (backup)
- **APIs**: Unsplash (images + photo credits), YouTube API (trending), Supadata (transcripts)
- **Deploy**: Vercel (auto-deploys from `main`)

## Env Vars (in `.env.local`)
`ANTHROPIC_API_KEY`, `SUPABASE_SERVICE_KEY`, `UNSPLASH_ACCESS_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, `YOUTUBE_API_KEY`, `SUPADATA_API_KEY`, `ADMIN_PASSWORD`

## Development Workflow

### Worktree Setup
The dev server runs from a git worktree, not the main repo:
- **Main repo**: `/Users/bxb/recdex/` (checked out on `main`)
- **Worktree**: `/Users/bxb/recdex/.claude/worktrees/magical-benz/` (branch `claude/magical-benz`)
- **Pattern**: Edit files in main repo → `cp` to worktree → verify in preview → merge to main and push

### Verify Before Done
1. Copy changed files to worktree
2. Check preview (dev server on port 3000) — screenshot + interact
3. Check console for errors
4. `next build` catches type errors the dev server misses
5. Merge worktree branch to main, push

### Database Changes
- No direct DB access (psql blocked, no Supabase access token linked)
- Schema changes: ask user to run SQL in Supabase dashboard, or try `npx supabase db query --linked`
- Use `SUPABASE_SERVICE_KEY` for any writes — anon key silently fails due to RLS

## Key Files

### Pages
| File | What it does |
|------|-------------|
| `src/app/page.tsx` | Homepage — hero, Tonight's Pick, cuisine stats, Discover (shuffled recipes) |
| `src/app/browse/page.tsx` | Cookbook index — flat alphabetical list, thumbnails, cuisine pills, time filters |
| `src/app/contribute/page.tsx` | Paste URL → extract → review → publish flow |
| `src/app/recipe/[slug]/page.tsx` | Recipe detail — hero image, ingredients, Kitchen Consensus |
| `src/app/recipe/[slug]/cook/page.tsx` | Cook mode — step-by-step with timers, dock-style navigation, share on completion |
| `src/app/pantry/page.tsx` | "What's in your kitchen?" — ingredient scan → recipe matches |
| `src/app/scan/page.tsx` | Cookbook photo scan → recipe extraction |
| `src/app/admin/page.tsx` | Admin dashboard (password-gated) |
| `src/app/trending/page.tsx` | Trending recipes from YouTube |

### API Routes
| Route | Purpose |
|-------|---------|
| `api/extract-recipe` | Claude-powered recipe extraction from URL/text |
| `api/publish-recipe` | Save to Supabase + Unsplash image fallback + photo credit |
| `api/scan-recipe` | Extract recipe from cookbook photo |
| `api/image-search` | Unsplash image search |
| `api/substitute` | Ingredient substitution suggestions |
| `api/admin` | Admin operations (update image, delete recipe) |
| `api/tts` | Text-to-speech (currently unused — voice mode removed) |

### Lib
| File | Purpose |
|------|---------|
| `lib/supabase.ts` | Supabase client (anon key hardcoded) |
| `lib/theme.ts` | Color tokens (`C.*`), font stacks, shared constants |
| `lib/cook-utils.ts` | Timer parsing, step classification helpers |
| `lib/cooking-tips.ts` | Contextual cooking tips database |
| `lib/substitutions.ts` | Ingredient substitution data |
| `lib/classify-steps.ts` | Step type classification (prep, active, passive) |

## Design Sensibility
The user (Ben) has strong design opinions. Key patterns:
- **Cookbook index feel** for browse — not a Pinterest grid. Alphabetical, scannable, minimal.
- **Time over difficulty** — difficulty labels removed. Time is the real differentiator.
- **Images matter on homepage** — Discover section needs thumbnails. Browse can be compact.
- **No publication names in titles** — "Classic Minestrone" not "BA's Classic Minestrone"
- **Photo credits** — Unsplash photographer bylines on recipe images
- **Mobile-first** — `isMobile` breakpoint at 768px (some pages use 700 or 820)
- **No over-engineering** — ship it, iterate based on feedback

## Common Pitfalls
1. **Worktree sync**: Edits in main repo don't appear in dev preview until copied to worktree
2. **RLS writes**: Supabase anon key reads succeed but writes silently fail. Always use service key.
3. **React error #300**: Client-side `router.push()` between pages with different hook counts crashes. Use `window.location.href` for cross-page navigation.
4. **Nested cp**: `cp -r dir/` into a directory that already has that name creates `dir/dir/`. Remove first or copy contents.
5. **Photo IDs**: Unsplash URLs stored in DB don't always map cleanly to API photo IDs for credit lookup.

## Recipe DB Schema (Supabase `recipes` table)
Required fields for every recipe:
- `title` — Clean dish name. NEVER include publication names.
- `slug` — URL-safe, lowercase, unique
- `description` — 1-2 sentences. What it is + what makes this version worth making.
- `cuisine` — Capitalize first letter. Use specific cuisines (e.g. "Sichuan" not "Chinese" when appropriate)
- `time_total` — Human-readable string: "25 min", "1 hr 30 min", "3 hrs"
- `ingredients` — JSON array of `{ name, amount, unit, group? }` objects
- `steps` — JSON array of `{ step, text }` objects (1-indexed)
- `tags` — String array for searchability
- `status` — "published" or "draft"

Optional but encouraged:
- `image_url` — Unsplash regular URL
- `photo_credit` — Photographer name from Unsplash
- `servings`, `servings_label` — e.g. "4", "servings"
- `time_active`, `time_passive`, `time_passive_label`
- `source_attribution`, `source_url` — Credit original author/source
- `video_url`, `creator_name`, `creator_url` — For video-sourced recipes

## Brand Voice (for recipe writing)
- Concise and practical — every word earns its place
- Warm but not cutesy. Confident but not preachy
- Instructions are clear, sequential, action-oriented
- Use specific sensory cues ("until the onions are translucent and fragrant, about 3 minutes")
- Include timing estimates in steps where helpful
- Never use "Enjoy!" or "Serve and enjoy!" — end with the final practical step
- Descriptions: 1-2 sentences — what the dish IS and what makes this version worth making

## Automation & Pipeline

### Scheduled Tasks
1. **Daily Recipe Learning** (every 4 hrs, Claude desktop app) — Imports 10 new recipes per run, rotates cuisines, rewrites in RecDex voice. Must dedup against existing recipes before importing.
2. **Image fill hourly** (hourly, local) — Runs `scripts/recipe-pipeline/4-fetch-images.mjs --from-db` to fill missing Unsplash images.

### Scripts (`scripts/`)
| Script | Purpose | Usage |
|--------|---------|-------|
| `recipe-pipeline/run-pipeline.mjs` | Full 5-stage import pipeline (Source → Synthesize → Validate → Images → Publish) | `node run-pipeline.mjs "recipe name"` |
| `recipe-pipeline/4-fetch-images.mjs` | Fill Unsplash images for recipes missing them | `node 4-fetch-images.mjs --from-db` |
| `check-data-quality.ts` | Detect dupes, missing images, bad titles, missing fields | `npx tsx scripts/check-data-quality.ts` |
| `dedup-recipes.ts` | Remove duplicate recipes (keeps oldest, preserves images) | `npx tsx scripts/dedup-recipes.ts --dry-run` |
| `backfill-photo-credits.ts` | Backfill photographer names for existing Unsplash images | `npx tsx scripts/backfill-photo-credits.ts` |

### Dedup Rules (CRITICAL for scheduled imports)
When importing recipes, check for duplicates:
1. Query ALL existing titles from Supabase
2. Case-insensitive exact match (e.g. "Pad Thai" = "pad thai")
3. Near-match detection: "Classic Tiramisu" ≈ "Tiramisu", "Filipino Chicken Adobo" ≈ "Chicken Adobo"
4. If a recipe exists with a similar name, SKIP IT — don't create a variant

## Session Checklist

**Start of session (parallel sessions):**
1. Read `COORDINATION.md` to understand what other sessions are doing
2. Check "Open Questions" — your answer might already be there
3. Read this file + `tasks/lessons.md`
4. Verify dev server is running (`preview_list`)
5. Mark your session as "In Progress" in COORDINATION.md

**During work:**
- If you make a decision that affects other sessions, update COORDINATION.md immediately
- If you hit a blocker, flag it in COORDINATION.md

**End of session:**
1. Copy files to worktree, verify in preview
2. Commit with descriptive message, push to main
3. Update `COORDINATION.md`: status, accomplishments, new blockers/questions
4. Update `tasks/todo.md` and `tasks/lessons.md` if applicable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benxbridges)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/benxbridges)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
