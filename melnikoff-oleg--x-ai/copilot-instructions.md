## x-ai

> Use when adding new functionality, commands, scripts, or making structural changes. Produces a thorough plan document in `plans/` that captures context, rationale, and step-by-step tasks.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Is

This is a **Claude Workspace Template** — a structured environment designed for working with Claude Code as a powerful agent assistant across sessions. The user will spin up fresh Claude Code sessions repeatedly, using `/prime` at the start of each to load essential context without bloat.

**This file (CLAUDE.md) is the foundation.** It is automatically loaded at the start of every session. Keep it current — it is the single source of truth for how Claude should understand and operate within this workspace.

---

## The Claude-User Relationship

Claude operates as an **agent assistant** with access to the workspace folders, context files, commands, and outputs. The relationship is:

- **User**: Defines goals, provides context about their role/function, and directs work through commands
- **Claude**: Reads context, understands the user's objectives, executes commands, produces outputs, and maintains workspace consistency

Claude should always orient itself through `/prime` at session start, then act with full awareness of who the user is, what they're trying to achieve, and how this workspace supports that.

---

## Workspace Structure

```
.
├── CLAUDE.md              # This file — core context, always loaded
├── package.json           # Root scripts (npm run dev proxies into app/)
├── .env                   # API keys (APIFY_API_TOKEN, GEMINI_API_KEY, ANTHROPIC_API_KEY, KIE_AI_API_KEY)
├── .claude/
│   └── commands/          # Slash commands Claude can execute
├── app/                   # X AI — Next.js web application
│   └── src/
│       ├── app/           # Pages + API routes
│       │   ├── layout.tsx         # Root layout (sidebar + top-bar + content)
│       │   ├── globals.css        # Dark theme, glass-morphism, oklch colors
│       │   ├── page.tsx           # Redirects to /posts
│       │   ├── posts/page.tsx     # Post browser with filters, dual-view cards, modals
│       │   ├── creators/page.tsx  # Creator management (X accounts)
│       │   ├── configs/page.tsx   # Pipeline config management
│       │   ├── run/page.tsx       # Pipeline execution with live progress (4 phases)
│       │   └── api/               # API routes (posts, creators, configs, pipeline, proxy-image)
│       ├── lib/           # Core logic
│       │   ├── apify.ts           # X/Twitter scraping via Apify
│       │   ├── gemini.ts          # Media upload + deep multimodal analysis
│       │   ├── claude.ts          # Ready-to-post content + image prompt generation
│       │   ├── kie.ts             # Kie AI infographic image generation
│       │   ├── brand-style.ts     # Brand style constants + reference image helper
│       │   ├── pipeline.ts        # 4-phase orchestrator
│       │   ├── csv.ts             # CSV read/write with typed helpers
│       │   └── types.ts           # All TypeScript interfaces
│       └── components/    # UI components
│           ├── app-sidebar.tsx    # Navigation sidebar
│           ├── top-bar.tsx        # Sticky backdrop-blur header
│           ├── markdown-content.tsx # Markdown renderer
│           └── ui/                # shadcn/ui primitives
├── data/                  # CSV data store (creators.csv, posts.csv, configs.csv)
│   └── generated-images/  # Kie AI generated infographic PNGs
├── references/            # Reference infographic images (5 Jake Ward style examples)
├── scripts/               # One-off utility scripts (generation demos, testing)
├── output/                # Generated output files (HTML comparisons, demos)
├── context/               # Background context about the user and project
└── plans/                 # Implementation plans
```

**Key directories:**

| Directory    | Purpose                                                                             |
| ------------ | ----------------------------------------------------------------------------------- |
| `app/`       | X AI Next.js app — viral content analyzer + generator for marketing/CRO niche. |
| `data/`      | CSV storage for creators, posts, configs, and generated images.                    |
| `references/`| 5 reference infographic images (Jake Ward style) for Kie AI style matching.       |
| `scripts/`   | Generation scripts: `generate-demo.ts` (single post), `generate-batch.ts` (3-post batch with QC), `generate-smart.ts` (10-post with AI-driven style classification). |
| `output/`    | Generated output files (HTML comparison pages, demo artifacts).                   |
| `context/`   | Who the user is, their role, current priorities, strategies. Read by `/prime`.      |
| `plans/`     | Detailed implementation plans. Created by `/create-plan`, executed by `/implement`. |

---

## X AI App

A Next.js web application that analyzes viral X/Twitter posts and generates ready-to-publish content with branded infographic images.

**Run:** `npm run dev` → http://localhost:3000

**Niche:** Marketing, CRO, growth. Pre-configured with "Growth Marketing" config.

**Pipeline:** Apify (scrape X posts) → Gemini (deep analysis) → Claude (generate post text + image prompt) → Kie AI (generate infographic)

### Pages

- `/posts` — Twitter-authentic tweet cards with side-by-side layout (original tweet left, AI-generated draft right). Cards use real X/Twitter dark theme styling: `#000` bg, `#e7e9ea` text, `#71767b` secondary, `#2f3336` borders, `#1d9bf0` blue accents. Each tweet shows avatar + display name + verified badge + @handle + relative time. Twitter action bar (reply/repost/like/views/bookmark/share) with colored hover states. Generated side shows draft from @olegrows. 3-tab modal (Original/Analysis/Generated) with expanded tweet view, engagement counts, download button, star toggle
- `/creators` — Manage X creator accounts with profile pics, stat boxes (followers, following, posts), blue verified badge, hover-reveal actions
- `/configs` — Manage analysis/generation prompts with preview boxes, creator/post counts per config
- `/run` — Execute 4-phase pipeline with collapsible advanced settings, gradient progress bar, collapsible log, completion CTA

### Tech Stack

- **Next.js 16.2.4** with Turbopack, App Router
- **TypeScript**, **Tailwind CSS v4**
- **shadcn/ui** with base-ui primitives (NOT Radix — Next.js 16 breaking change)
- **CSV file-based storage** in `data/` directory
- **SSE** (Server-Sent Events) for pipeline progress streaming

### Design System

Dark theme with glass-morphism, matching the Instagram Social Media AI app exactly:

- **Background:** `oklch(0.12 0.005 260)` (near-black)
- **Glass cards:** `bg-white/[0.02] backdrop-blur-xl border border-white/[0.06]`
- **Stat boxes:** `bg-black/20 border border-white/[0.04]`
- **Primary buttons:** `bg-gradient-to-r from-purple-500 to-indigo-600`
- **Ghost buttons:** `glass border-white/[0.06]`
- **Badges:** `bg-white/[0.05] border border-white/[0.08]`
- **Inputs:** `rounded-xl glass border-white/[0.08] h-11`
- **Dialogs:** `glass-strong rounded-2xl border-white/[0.08]`, use `!flex !flex-col` to override base-ui grid layout
- **Progress bars:** Purple-indigo (running), emerald-teal (complete), red-orange (error)

**Posts page uses authentic X/Twitter styling** (not glass-morphism):
- **Card bg:** `bg-black/40` with `border-[#2f3336]`
- **Text colors:** `#e7e9ea` (primary), `#71767b` (secondary/handles)
- **Action hover colors:** Reply `#1d9bf0`, Repost `#00ba7c`, Like `#f91880`, Views/Bookmark/Share `#1d9bf0`
- **Verified badge:** Inline SVG blue checkmark matching X's exact path
- **Media:** `rounded-2xl border-[#2f3336]`, single images render at full height (no max-h or object-cover), multi-image grids use aspect ratios with object-cover
- **Tweet text:** `text-[15px] leading-[20px]` in cards, `text-[17px] leading-[24px]` in modal (matches X)
- **Fonts:** Geist Sans + Geist Mono via `next/font/google`

### Next.js 16 / base-ui Gotchas

- `SidebarMenuButton` uses `render` prop instead of `asChild`: `<SidebarMenuButton render={<Link href={...} />}>`
- `Select.onValueChange` passes `string | null` — always guard: `onValueChange={(v) => v && setter(v)}`
- `DialogTrigger` does NOT support `asChild` — style the trigger directly or use `render` prop
- `Button` does NOT support `asChild` — use `<Link>` with inline styles instead
- `DialogContent` base class uses `grid` layout — override with `!flex !flex-col` when you need flex scrolling
- CSV boolean fields: compare with `String(value) === "true"` (csv-parse auto-casts)

### External Services

- **Apify** — `apidojo~twitter-scraper-lite` (tweets, optimized `twitterHandles` query) + `quacker~twitter-scraper` (profile stats)
- **Google Gemini 2.5 Flash** — Deep multimodal analysis (text + images only, video posts are skipped).
- **Anthropic Claude Sonnet** — `claude-sonnet-4-5-20250929` for generating ready-to-post X content + infographic image prompts (JSON output)
- **Kie AI** — `nano-banana-2` model for generating branded 3:4 vertical infographic images with reference image style matching
- **Twitter CDN** — Media proxied through `/api/proxy-image` (domains: `pbs.twimg.com`, `video.twimg.com`, etc.)

### Required Environment Variables

In `.env` at workspace root:
- `APIFY_API_TOKEN` — for X/Twitter scraping
- `GEMINI_API_KEY` — for media upload and multimodal analysis
- `ANTHROPIC_API_KEY` — for content generation with Claude
- `KIE_AI_API_KEY` — for infographic image generation

### Seeded Data

- **8 marketing/CRO creators:** CarlWeische, boringmarketer, blvckledge, PJaccetturo, KateBour, thejustinwelsh, amandanat, alexgarcia_atx
- **1 config:** "Growth Marketing" with analysis + generation prompts for marketing/CRO niche

### Pipeline Architecture

1. **Scraping phase** — All creators scraped in parallel via Apify (optimized `twitterHandles` query, ~5s per creator). Video/gif posts are filtered out. Sorts by engagement, takes top K.
2. **Analysis phase** — 3 concurrent workers. Downloads images → uploads to Gemini → deep multimodal analysis. Text-only and text+image posts only.
3. **Generation phase** — Starts with a classification sub-step: a single Claude call assigns each post one of three categories (60/30/10 split):
   - **Infographic (60%)** — One of 5 styles (Bar Chart Comparison, Proportional Blocks, Ranked Horizontal Bars, Timeline, Pyramid/Funnel). Claude generates post text + style-specific image prompt → Kie AI generates branded infographic.
   - **Personal (30%)** — Paired with a random personal photo from `references/personal *.{jpg,jpeg,png}`. Claude generates text optimized for personal/authentic tone. No Kie AI call.
   - **Text-only (10%)** — No image. Claude generates standalone text that carries all the weight.
   Username attribution (`@olegrows`) injected into infographic prompts. Ratios enforced programmatically after classification.
4. **Persistence** — Writes results to `data/posts.csv` (includes `infographicStyle` field), saves generated images to `data/generated-images/`.

**5 Infographic Styles:** Bar Chart Comparison, Proportional Blocks, Ranked Horizontal Bars, Timeline, Pyramid/Funnel. Defined in `brand-style.ts`.
**18 Personal Photos:** `references/personal 1-18` — randomly assigned to personal-category posts.

Gemini has exponential backoff retry (5 attempts, delays 5s→15s→30s→60s→90s) for 429 and 503 errors. Free tier has daily quota limits.

### Robustness

- All external API calls have timeouts (Apify 180s, Gemini upload 180s, analyze 180s, download 120s, Kie AI 3min polling)
- CSV writes use atomic temp-file + rename with promise-chain write lock
- Pipeline input validation/clamping (maxPosts 1-100, topK 1-10, daysLookback 1-365)
- Apify response normalization handles multiple field name formats
- Kie AI images downloaded immediately (CDN URLs expire after 24h)
- Verbose pipeline logs with per-step timing, file sizes, character counts

### Brand Style System

The reference infographics (`references/infographic 1-5.jpeg`) define the visual identity for generated infographics — all in the "Jake Ward" dark data-viz style:
- Pure black (#000000) background with subtle diagonal line texture
- Neon yellow-green / chartreuse (#C8FF00) as primary accent
- White bold condensed sans-serif titles (Impact/Bebas Neue style)
- Data visualization focused (bar charts, comparisons, timelines, ranked lists)
- 3:4 vertical aspect ratio
- Bold, modern, data-forward aesthetic

Style constants in `brand-style.ts` are injected into Claude's system prompt. Primary reference: `infographic 1.jpeg`.

**Note:** Kie AI's API does not support `image_input` as base64 data URI — style matching relies on detailed text prompts only. Also, image prompts must avoid real person/brand names (Kie AI content filter rejects them).

### Batch Generation Script

`scripts/generate-batch.ts` — scrapes posts from a creator, generates new posts + infographics in parallel with automated QC.

**Configuration** (constants at top of script):
- `USERNAME` — Attribution handle for infographics (default: `@olegrows`)
- `CREATOR` — X/Twitter username to scrape (default: `CarlWeische`)
- `NUM_POSTS` — Number of posts to process (default: 3)
- `MAX_QC_ATTEMPTS` — Max retries per post on QC failure (default: 3)

**Pipeline:** Apify scrape → 3× parallel (Claude content gen → Kie AI infographic → Claude Vision QC → retry if fail) → HTML output

**Quality Control:** After each infographic is generated, Claude Vision reviews it for: duplicated text, garbled text, missing content, layout issues, and style mismatch. Failed QC triggers automatic retry with a fresh prompt. Results shown in toggleable QC tab in HTML output.

**Run:** `npx tsx scripts/generate-batch.ts` → produces `output/batch-comparison.html`

### Smart Generation Script

`scripts/generate-smart.ts` — scrapes 10 posts, uses AI-driven classification to assign optimal infographic styles, generates 7 infographic + 3 text-only posts.

**Key innovation:** A classification step sends all 10 posts to Claude in a single call. Claude analyzes each post's content and assigns the best-fit infographic style (or text-only), ensuring all 5 styles are used at least once and exactly 3 posts are text-only.

**5 Infographic Styles:**
1. Bar Chart Comparison — A vs B, before/after data
2. Proportional Blocks — market share, percentage breakdowns
3. Ranked Horizontal Bars — top N lists, leaderboards
4. Timeline — historical evolution, milestones
5. Pyramid/Funnel — frameworks, hierarchies, layered strategies

**Text-only criteria:** Personal stories, opinion-only takes, CTAs, short hot takes without enough substance for a visual.

**Pipeline:** Apify scrape (optimized `twitterHandles` query) → classify all posts (single Claude call) → 10× parallel generation (7 with Kie AI + QC, 3 text-only) → HTML with two sections

**Run:** `npx tsx scripts/generate-smart.ts` → produces `output/smart-comparison.html`

### Markdown Content System

The analysis modal uses custom `.markdown-body` CSS (in `globals.css`) instead of Tailwind prose:
- All typography colors tuned for dark oklch background
- Custom purple bullet points and numbered list counters
- Code blocks with glass-style backgrounds
- Blockquotes with purple left border
- Tables with subtle dividers
- 15px base font, 1.75 line-height for comfortable reading of long-form AI output (10-15K chars)

---

## Commands

### /prime

**Purpose:** Initialize a new session with full context awareness.

Run this at the start of every session. Claude will:

1. Read CLAUDE.md and context files
2. Summarize understanding of the user, workspace, and goals
3. Confirm readiness to assist

### /create-plan [request]

**Purpose:** Create a detailed implementation plan before making changes.

Use when adding new functionality, commands, scripts, or making structural changes. Produces a thorough plan document in `plans/` that captures context, rationale, and step-by-step tasks.

Example: `/create-plan add a competitor analysis command`

### /implement [plan-path]

**Purpose:** Execute a plan created by /create-plan.

Reads the plan, executes each step in order, validates the work, and updates the plan status.

Example: `/implement plans/2026-01-28-competitor-analysis-command.md`

---

## Critical Instruction: Maintain This File

**Whenever Claude makes changes to the workspace, Claude MUST consider whether CLAUDE.md needs updating.**

After any change — adding commands, scripts, workflows, or modifying structure — ask:

1. Does this change add new functionality users need to know about?
2. Does it modify the workspace structure documented above?
3. Should a new command be listed?
4. Does context/ need new files to capture this?

If yes to any, update the relevant sections. This file must always reflect the current state of the workspace so future sessions have accurate context.

---

## Session Workflow

1. **Start**: Run `/prime` to load context
2. **Work**: Use commands or direct Claude with tasks
3. **Plan changes**: Use `/create-plan` before significant additions
4. **Execute**: Use `/implement` to execute plans
5. **Maintain**: Claude updates CLAUDE.md and context/ as the workspace evolves

---

## Notes

- Keep context minimal but sufficient — avoid bloat
- Plans live in `plans/` with dated filenames for history
- Reference materials go in `references/` for reuse
- The Instagram source app is at github.com/melnikoff-oleg/social-media — the canonical reference for styling and UX patterns

---
> Source: [melnikoff-oleg/x-ai](https://github.com/melnikoff-oleg/x-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
