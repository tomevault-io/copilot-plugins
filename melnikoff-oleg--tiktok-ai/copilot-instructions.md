## tiktok-ai

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
├── .env                   # API keys (APIFY_API_TOKEN, GEMINI_API_KEY, ANTHROPIC_API_KEY)
├── .claude/
│   └── commands/          # Slash commands Claude can execute
├── app/                   # TikTok AI — Next.js web application
│   └── src/
│       ├── app/           # Pages + API routes
│       │   ├── layout.tsx         # Root layout (sidebar + top-bar + content)
│       │   ├── globals.css        # Dark theme, glass-morphism, oklch colors
│       │   ├── page.tsx           # Redirects to /videos
│       │   ├── videos/page.tsx    # Video browser with filters, modals
│       │   ├── creators/page.tsx  # Creator management
│       │   ├── configs/page.tsx   # Pipeline config management
│       │   ├── run/page.tsx       # Pipeline execution with live progress
│       │   └── api/               # API routes (videos, creators, configs, pipeline, proxy-image)
│       ├── lib/           # Core logic
│       │   ├── apify.ts           # TikTok scraping via Apify
│       │   ├── gemini.ts          # Video upload + multimodal analysis
│       │   ├── claude.ts          # Concept generation with Claude
│       │   ├── pipeline.ts        # 3-phase orchestrator
│       │   ├── csv.ts             # CSV read/write with typed helpers
│       │   └── types.ts           # All TypeScript interfaces
│       └── components/    # UI components
│           ├── app-sidebar.tsx    # Navigation sidebar
│           ├── top-bar.tsx        # Sticky backdrop-blur header
│           ├── markdown-content.tsx # Markdown renderer
│           └── ui/                # shadcn/ui primitives
├── data/                  # CSV data store (creators.csv, videos.csv, configs.csv)
├── context/               # Background context about the user and project
└── plans/                 # Implementation plans
```

**Key directories:**

| Directory    | Purpose                                                                             |
| ------------ | ----------------------------------------------------------------------------------- |
| `app/`       | TikTok AI Next.js app — viral content analyzer for beauty/lifestyle/fashion niche. |
| `data/`      | CSV storage for creators, videos, and configs. Shared between app and workspace.    |
| `context/`   | Who the user is, their role, current priorities, strategies. Read by `/prime`.      |
| `plans/`     | Detailed implementation plans. Created by `/create-plan`, executed by `/implement`. |

---

## TikTok AI App

A Next.js web application that analyzes viral TikTok content and generates content concepts. Port of the Instagram Social Media AI app (github.com/melnikoff-oleg/social-media), matching its exact dark glass-morphism aesthetic.

**Run:** `npm run dev` → http://localhost:3000

**Niche:** Beauty, lifestyle, fashion. Pre-configured with "Bella Glow" brand persona.

**Pipeline:** Apify (scrape TikTok) → Gemini (analyze video) → Claude (generate concepts)

### Pages

- `/videos` — Browse analyzed videos with glass cards, 9:16 thumbnails, star toggle, Analysis/Concepts modal with underline tabs, full markdown rendering
- `/creators` — Manage TikTok creators with profile pics, stat boxes (followers, hearts, videos), hover-reveal actions
- `/configs` — Manage analysis/concept prompts with preview boxes, creator/video counts per config
- `/run` — Execute pipeline with collapsible advanced settings, gradient progress bar, collapsible log, completion CTA

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
- **Fonts:** Geist Sans + Geist Mono via `next/font/google`

### Next.js 16 / base-ui Gotchas

- `SidebarMenuButton` uses `render` prop instead of `asChild`: `<SidebarMenuButton render={<Link href={...} />}>`
- `Select.onValueChange` passes `string | null` — always guard: `onValueChange={(v) => v && setter(v)}`
- `DialogTrigger` does NOT support `asChild` — style the trigger directly or use `render` prop
- `Button` does NOT support `asChild` — use `<Link>` with inline styles instead
- `DialogContent` base class uses `grid` layout — override with `!flex !flex-col` when you need flex scrolling
- CSV boolean fields: compare with `String(value) === "true"` (csv-parse auto-casts)

### External Services

- **Apify** — `clockworks~tiktok-scraper` actor for TikTok video scraping and creator stats
- **Google Gemini 2.5 Flash** — Resumable upload protocol for video files, multimodal analysis
- **Anthropic Claude Sonnet** — `claude-sonnet-4-5-20250929` for concept generation, max_tokens 4096
- **TikTok CDN** — Thumbnails proxied through `/api/proxy-image` (domains: `*.tiktokcdn-us.com`, `*.tiktokcdn.com`)

### Required Environment Variables

In `.env` at workspace root:
- `APIFY_API_TOKEN` — for TikTok scraping
- `GEMINI_API_KEY` — for video upload and multimodal analysis
- `ANTHROPIC_API_KEY` — for concept generation with Claude

### Seeded Data

- **6 beauty/lifestyle creators:** xoxoemira, alixearle, golloria, lydiamillen, leticiafgomes, haileybieber
- **1 config:** "Bella Glow" with analysis + concepts prompts for beauty/lifestyle niche

### Pipeline Architecture

1. **Scraping phase** — Sequential per creator via Apify. Fetches recent videos, sorts by plays, takes top K.
2. **Analysis phase** — 1 worker (sequential to avoid rate limits). Downloads video → uploads to Gemini → analyzes → generates concepts with Claude.
3. **Persistence** — Writes results to `data/videos.csv`.

Gemini has exponential backoff retry (5 attempts, delays 5s→15s→30s→60s→90s) for 429 and 503 errors. Free tier has daily quota limits.

### Robustness

- All external API calls have timeouts (Apify 180s, Gemini upload 180s, analyze 180s, download 120s)
- CSV writes use atomic temp-file + rename with promise-chain write lock
- Pipeline input validation/clamping (maxVideos 1-100, topK 1-10, daysLookback 1-365)
- Download validation rejects files < 1KB
- Verbose pipeline logs with per-step timing, file sizes, character counts

### Markdown Content System

The analysis/concepts modal uses custom `.markdown-body` CSS (in `globals.css`) instead of Tailwind prose:
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
- Outputs are organized by type/purpose in `outputs/`
- Reference materials go in `reference/` for reuse
- The Instagram source app is at github.com/melnikoff-oleg/social-media — the canonical reference for styling and UX patterns

---
> Source: [melnikoff-oleg/tiktok-ai](https://github.com/melnikoff-oleg/tiktok-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
