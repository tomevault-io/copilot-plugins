## aeo-radar

> > Context for Claude Code (or any LLM assistant) working on this repo. Keep it short; if the code answers the question, don't repeat the answer here.

# CLAUDE.md — Dev playbook for AEO Radar

> Context for Claude Code (or any LLM assistant) working on this repo. Keep it short; if the code answers the question, don't repeat the answer here.

## Tech stack

- **CLI:** TypeScript + tsx, Playwright (stealth plugin), Claude CLI via `execSync`
- **DB:** SQLite via Prisma 7 (file at `data/aeo.db`)
- **Dashboard:** Next.js 16 + React 19 + Ant Design 6 + Recharts (port 3003)
- **Package manager:** npm

## Common commands

```bash
# Crawl (defaults to headless)
npx tsx src/cli.ts run                 # ChatGPT
npx tsx src/cli.ts run chatgpt --visible  # debug with visible browser

# Seeding & analysis
npx tsx src/cli.ts seed                # generate prompts from brand.ts
npx tsx src/cli.ts analyze <runId>     # re-run analysis on a past run

# Database
npm run db:push                        # apply schema changes to SQLite

# Dashboard
cd web && npm run dev                  # http://localhost:3003
```

## Source layout

```
src/
├── cli.ts                  # Entry point (run / analyze / seed)
├── types.ts                # LLM_PLATFORMS + shared types
├── config/
│   └── brand.ts            # ← single source of truth for brand/keywords
├── crawlers/
│   ├── base.ts             # Stealth + challenge detection + persistent profile
│   └── chatgpt.ts          # DOM selectors: #prompt-textarea, stop-button
├── pipeline/
│   ├── runner.ts           # Orchestrates prompts × platforms × analysis
│   └── analyzer.ts         # Claude CLI → structured JSON
├── prompts/
│   └── generator.ts        # Seeds keywords/prompts from brand.ts
└── db/
    └── prisma.ts           # Prisma singleton

prompts/
├── generator-template.md   # Claude template for prompt generation
└── analysis-template.md    # Claude template for answer analysis

web/                        # Next.js dashboard
├── src/app/                # Pages + API routes
├── src/components/         # KPI cards, charts, tables, settings
└── src/lib/theme.ts        # Grayscale tokens, AntD ConfigProvider
```

## Database schema (5 tables)

- **AeoKeyword** — seed keywords from `brand.ts`
- **AeoPrompt** — 4 intent variants per keyword
- **AeoCrawlRun** — one row per `npm run crawl` invocation
- **AeoResult** — raw response + screenshot, per prompt × platform
- **AeoAnalysis** — LLM-extracted fields (brand mention, rank, sentiment, competitors, URLs)

## Things to know before editing

- **Do not hardcode brand strings.** Anything brand-specific belongs in `src/config/brand.ts`. Prompts use `{{brand_name}}`, `{{brand_description}}`, `{{target_market}}` placeholders.
- **Dashboard + CLI share the same SQLite file.** `web/src/lib/prisma.ts` and `src/db/prisma.ts` both point at `data/aeo.db`. Don't split them.
- **Analysis goes through the Claude CLI**, not the Anthropic API. Reason: reuses whatever subscription the user already has, no extra keys.
- **Playwright uses a persistent context** under `auth/profiles/<platform>/`. The directory is gitignored.
- **Stealth plugin is already applied** at module load in `base.ts`. Don't re-apply.
- **Challenge detection** (Cloudflare, reCAPTCHA) blocks for 120s waiting for the user to solve manually. Only works in non-headless mode.
- **Prompt generation is idempotent per keyword.** If a keyword already has prompts, `seed` skips it. Delete rows if you need to regenerate.

## Adding a platform

1. `src/crawlers/<name>.ts` extends `BaseCrawler`, implements `crawl()`.
2. Add name to `LLM_PLATFORMS` in `src/types.ts`.
3. Add a case to `createCrawler()` in `src/pipeline/runner.ts`.
4. If the engine fingerprints Chrome-for-Testing as a bot, override `launchChannel` to `'chrome'` in your subclass (see comments in `base.ts`).

## Dashboard theme

`web/src/lib/theme.ts` is the single source of truth. It's a **dark-mode, grayscale-only** theme: near-black backgrounds (`#000` / `#0a0a0a`), near-white text, subtle gray borders. Exports:

- `COLORS` — grayscale ramp (`black`, `gray950…gray50`, `white`)
- `SHAPE`, `SPACER`, `ELEVATION` — grouped token objects
- `appTheme` — AntD `ConfigProvider` config (uses `theme.darkAlgorithm`)
- `CHART_COLORS`, `CHART_TOOLTIP_STYLE`, `CHART_PALETTE` — Recharts styling (white → dark-gray ramp)
- `PLATFORM_COLORS` — per-platform color map (currently `{ chatgpt: white }`)
- `DS`, `STATUS_TAG_STYLES` — semantic aliases used throughout

If you need an accent color, the right move is usually to push brightness contrast further (pure white on gray-900) — not to introduce chroma.

---
> Source: [hellowalt/aeo-radar](https://github.com/hellowalt/aeo-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
