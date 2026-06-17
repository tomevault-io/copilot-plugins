## lead-magnet-quiz-workflow

> Standalone repository for building and deploying quiz funnel and vision board builder lead magnets. This is a packaged version of the lead magnet workflows from Vibe Marketing Studio, designed to run independently with Claude Code.

# Lead Magnet Quiz Workflow

Standalone repository for building and deploying quiz funnel and vision board builder lead magnets. This is a packaged version of the lead magnet workflows from Vibe Marketing Studio, designed to run independently with Claude Code.

> **Maintenance Note**: Keep this file updated when modifying skills, agents, or workflow stages. This is the source of truth for the project.

## Project Structure

```
.claude/
  skills/
    lead-magnet-quiz/           # Quiz funnel orchestrator skill
      SKILL.md
      references/               # Builder prompt template, CSV schemas, video templates
    lead-magnet-vision-board/   # Vision board orchestrator skill
      SKILL.md
      references/               # Glif prompt patterns, vertical templates (wedding, real-estate, contractor)
    setup-quiz-db/              # Supabase setup for quiz funnels
      SKILL.md
    setup-visionboard-db/       # Supabase + Glif setup for vision boards
      SKILL.md
agents/
  lead-magnet-agents/           # Quiz funnel agent definitions
    project-manager/            # Stage 0: Validates inputs, checks website access
    builder-agent/
      research-agent/           # Stage 1A: Market research via MCP tools
    quiz-architecture-agent/    # Stage 2A: Question flow, scoring, profiles
    design-strategy-agent/      # Stage 2B: Brand detection, design mode, motion system
      references/               # Component library, decorative elements, motion patterns, shape vocabulary
    copy-agent/                 # Stage 3: Landing page, quiz, emails, strategy pack
    build-agent/                # Stage 4: Astro project, edge functions, social ad video
      references/               # Astro patterns
    shared/                     # Shared utilities (image gen prompts, Playwright utils, question patterns)
    TROUBLESHOOTING.md          # Diagnostics guide
  vision-board-agents/          # Vision board agent definitions
    vb-architecture-agent/      # Preference dimensions, selection flow, profile matching
    vb-copy-agent/              # Builder copy, reveal page, emails, strategy pack
    vb-build-agent/             # Astro project, builder UI, Glif integration
    service-scraping-agent/     # Service/portfolio extraction
shared/
  templates/                    # Output templates (architecture, copy, design, research)
  examples/                     # Reference examples (landing pages, emails, lead magnets, screenshots)
remotion/                       # Remotion video rendering workspace
  src/remotion/                 # Dynamic composition, compiler, webpack config
  src/skills/                   # Video generation skills (3D, charts, typography, etc.)
  render-quiz-videos.sh         # Shell script to render social ad videos
marketing/
  strategy/                     # Sales collateral (ad strategy, call scripts, reel scripts)
  public/                       # Public-facing landing pages
  references/                   # Landing page design samples
output/                         # Working directory for active builds (gitignored)
scripts/
  setup.sh                      # Interactive configuration script
workflow-config.json            # GitHub username, Notion DB ID, paths, pricing
```

## Skills

| Skill | Description |
|-------|-------------|
| `/lead-magnet-quiz` | Orchestrated 6-agent quiz funnel builder with optional video assets |
| `/lead-magnet-vision-board` | Orchestrated 6-agent vision board builder with Glif graphic generation |
| `/setup-quiz-db` | Automate Supabase setup for quiz funnels (run after `/lead-magnet-quiz`) |
| `/setup-visionboard-db` | Automate Supabase + Glif setup for vision boards (run after `/lead-magnet-vision-board`) |

---

## Lead Magnet Quiz Workflow

Multi-agent workflow that produces a Vercel-deployable quiz funnel.

### Stages

- **Stage 1A+1B**: Research Agent + Product Scraping Agent (parallel)
- **Stage 2A+2B**: Architecture Agent + Design Strategy Agent (parallel)
- **Stage 3**: Copy Agent (+ Strategy Pack: ads, social roadmap, sales scripts)
- **Stage 4**: Build Agent (outputs Vercel-ready Astro app with local images + social ad video)
- **Stage 5**: Publish Agent (GitHub + GitHub Pages + Notion)
- **Post-workflow**: Run `/setup-quiz-db [business-name]` to connect Supabase and deploy

### Trigger

```
/lead-magnet-quiz [website-url]
```

### Output Structure

```
[business-name]/
  README.md                    # Overview (root level)
  builder-prompt.md            # AI-ready development prompt (root level)
  deploy/                      # Vercel-ready Astro project
    astro.config.mjs           # Astro config with Vercel adapter
    tsconfig.json, package.json, vercel.json, .env.example
    public/                    # Static assets
      images/                  # Logo + product images (local)
      styles/global.css        # CSS variables from design.md
      scripts/                 # quiz.js, admin.js
    src/                       # Astro source files
      layouts/Layout.astro
      pages/                   # index.astro, quiz/, admin/
    scripts/, supabase/        # Database setup
    api/                       # Vercel Edge Functions (outside Astro)
      quiz-submit.js           # Quiz submission -> Supabase
      email-sender.js          # Hourly cron for scheduled emails
      analytics-event.js       # POST - logs funnel events
      analytics-query.js       # GET - dashboard data (password protected)
    videos/                    # Social ad video (SocialAd.tsx + SocialAd.mp4)
  client/                      # Strategy docs for client delivery
    research.md/html, products.json/md
    architecture.md, design.md
    landing-page-copy.md, quiz-copy.md
    quiz-copy-explainer.html   # Full breakdown of copy decisions
    email-sequences.md/csv/html
    content-blocks.csv         # Profile blocks + answer callbacks
    questions-answers.md/csv
  client-preview/              # GitHub Pages for client review
    index.html                 # Navigation page (links to all 8 docs)
    walkthrough.html           # Quiz funnel walkthrough and usage guide
    research.html, email-sequences.html, quiz-copy-explainer.html
    ways-to-grow.html          # Included features + growth add-ons
    ad-strategy.html           # Google/Facebook/Instagram ad variations
    social-content.html        # 30-day content calendar + platform strategy
    sales-scripts.html         # Hot/Warm/Cold conversation frameworks
```

### Data Flow

Quiz submission -> Vercel Edge Function -> Supabase -> Email scheduling -> Content block resolution (profile blocks + answer callbacks) -> Resend (hourly cron)

### Email Personalization

26 emails across 5 sequences (Welcome, Cold Nurture, Warm Activation, Hot Path, Re-Engagement) with layered personalization:

- **Temperature** controls which sequence track (hot/warm/cold)
- **Profile** controls `{{profile_block}}` content injected into 10 emails
- **Quiz answers** control `{{answer_callback_N}}` snippets in 7 emails (from diagnostic questions identified by Architecture Agent)
- Content blocks stored in `{PREFIX}content_blocks` table, seeded from `content-blocks.csv`

### Analytics

Page events -> `/api/analytics-event` -> `analytics_events` table -> `/api/analytics-query` -> Dashboard

- **Admin dashboard** at `[deployed-url]/admin` with password protection (ADMIN_PASSWORD env var)
- Tracks 8 event types: page_view, quiz_start, question_viewed, answer_selected, email_captured, quiz_completed, result_page_viewed, cta_clicked
- Uses Chart.js for funnel visualization, temperature distribution, daily activity, answer analysis, UTM sources
- **Data retention**: Daily cron (`/api/data-cleanup` at 3 AM) deletes analytics events older than 90 days and sent email logs older than 365 days. Configurable via `DATA_RETENTION_ANALYTICS_DAYS` and `DATA_RETENTION_EMAIL_DAYS` env vars. Leads and quiz responses are never auto-deleted.

### Results Page Archetypes

Architecture Agent selects one of 5 archetypes based on business type + audience:

| Archetype | Best For | Visualization | Celebration |
|-----------|----------|---------------|-------------|
| `scorecard` | B2B, finance | Radar chart | Confetti |
| `style_profile` | Ecom, lifestyle | Spectrum bars | Shimmer |
| `pathway` | Education, coaching | Milestone map | Cascade |
| `archetype_reveal` | Personal brands | Trait badges | Shimmer |
| `diagnostic` | Agencies, tech | Horizontal bars | Confetti |

### Social Ad Video

Build Agent always creates a 20-second Remotion social ad (`deploy/videos/SocialAd.tsx`) and renders it to MP4 via `render-quiz-videos.sh`. The video MUST show ALL quiz profiles (not a subset).

### Deployment

```bash
cd deploy && npm install && npm run setup-db && npm run build && vercel --prod
```

- `npm run setup-db` creates all tables AND seeds email templates + content blocks from CSV (single command)
- Requires: Supabase project + env vars (SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY, TABLE_PREFIX, optional RESEND_API_KEY, ADMIN_PASSWORD)

### Client Preview

Published to GitHub Pages for client review of walkthrough, research, email sequences, copy explainer, and strategy pack.

- GitHub Pages URL: `https://[github-username].github.io/[business-name]-quiz-preview/`
- Private repo: `[github-username]/[business-name]-quiz-funnel`

### Troubleshooting

- If Playwright MCP fails, workflow automatically falls back to: BrowserBase -> WebFetch -> Manual overrides -> Archetype defaults
- Manual overrides available: `primary_color_override`, `heading_font_override`, `visual_style_override`
- Design Strategy Agent tracks detection method: `detected_from` field shows `playwright|browserbase|webfetch|override|inferred`
- See `agents/lead-magnet-agents/TROUBLESHOOTING.md` for complete diagnostics guide
- Project Manager Agent (Step 3b) validates website accessibility before Design Agent runs

---

## Lead Magnet Vision Board Workflow

Multi-agent workflow that produces a Vercel-deployable interactive vision board builder with AI-generated shareable graphics.

### Stages

- **Stage 1A+1B**: Research Agent (reused) + Service Scraping Agent (forked) (parallel)
- **Stage 2A+2B**: VB Architecture Agent (forked) + Design Strategy Agent (reused) (parallel)
- **Stage 3**: VB Copy Agent (forked) (+ Strategy Pack: ads, social roadmap, consultation scripts)
- **Stage 4**: VB Build Agent (forked) (outputs Vercel-ready builder app with Glif graphic generation)
- **Stage 5**: Publish Agent (reused) (GitHub + GitHub Pages + Notion)
- **Post-workflow**: Run `/setup-visionboard-db [business-name]` to connect Supabase + Glif and deploy

### Trigger

```
/lead-magnet-vision-board [url] --vertical wedding|real-estate|contractor|custom
```

### How It Differs from Quiz

| | Quiz Funnel | Vision Board Builder |
|---|---|---|
| User flow | Linear quiz with scoring | Multi-step builder/configurator |
| Output to user | Score + temperature + recommendations | Shareable graphic + profile + recommendations |
| Lead qualification | Explicit (score 0-100, hot/warm/cold) | Implicit (derived from budget + timeline tags) |
| Viral mechanic | None | Download + social share of branded graphic |
| Best verticals | B2B SaaS, consultants | Real estate, weddings, contractors |

### Key Features

- Preference dimensions replace quiz scoring (tag-based profile matching)
- 5 selection types: card_selection, chip_multi_select, scale_selector, toggle_group, image_grid
- Glif AI graphic generation (build-time via MCP + runtime via REST API)
- Download and social share buttons on reveal page
- 10 emails across 4 sequences (inspiration-first, not sales-first)
- Pre-built vertical templates for wedding, real estate, contractor

### Glif Integration

Uses Nano Banana Pro (ID: `cmi7ne4p40000kz04yup2nxgh`):
- **Build-time**: `run_glif` MCP tool for style cards, hero images, profile mood boards
- **Runtime**: REST API at `https://simple-api.glif.app` for personalized user graphics
- **Caching**: SHA-256 hash of selections stored in `graphic_cache` table

### Vertical Templates

Located in `.claude/skills/lead-magnet-vision-board/references/`:
- `vertical-wedding.json` - 6 dimensions, 4 profiles (Romantic, Classic, Adventurer, Entertainer)
- `vertical-real-estate.json` - 6 dimensions, 5 profiles (Minimalist, Nester, Entertainer, Urbanite, Retreater)
- `vertical-contractor.json` - 5 dimensions, 4 profiles (Modernizer, Craftsman, Host, Sanctuary Seeker)

---

## Configuration

### workflow-config.json

Central configuration file at the repo root:

```json
{
  "github_username": "YOUR_GITHUB_USERNAME",
  "notion_database_id": "YOUR_NOTION_DATABASE_ID",
  "paths": {
    "output_directory": "./output",
    "remotion_workspace": "./remotion"
  },
  "pricing": {
    "quiz_build": 3497,
    "vision_board_build": 2997,
    "bundle": 5497,
    "currency": "USD"
  }
}
```

### .claude/.mcp.json

Generated from `.claude/.mcp.json.template` during setup. Contains API tokens for Notion, Glif, Supabase, and Memory MCP servers. This file is gitignored -- never commit it.

### Setup

Run the interactive setup script after cloning:

```bash
./scripts/setup.sh
```

This will:
1. Set your GitHub username and Notion database ID in `workflow-config.json` and skill files
2. Configure MCP server API tokens (Notion, Glif, Supabase)
3. Optionally install Remotion dependencies
4. Verify prerequisites (GitHub CLI, Node.js)

---

## MCP Tools Required

### Configured via .claude/.mcp.json (setup required)

| Server | Purpose |
|--------|---------|
| **Notion** | Database operations, page creation for published funnels |
| **Glif** | AI graphic generation (vision board workflow) |
| **Supabase** | Database management for deployed funnels |
| **Memory** | Persistent context across sessions |

### Built into Claude Code (no setup needed)

| Server | Purpose |
|--------|---------|
| **Tavily** | Market research, competitor analysis |
| **DataForSEO** | Keyword data, SERP analysis, backlinks |
| **Playwright** | Website scraping, screenshots, brand detection |
| **Browserbase** | Cloud browser automation (Playwright fallback) |

---

## Service Pricing

### Lead Magnet Quiz BUILD Service (One-Time)

| Item | Price |
|------|-------|
| Complete quiz funnel build | **$3,497** |
| Timeline | 7 days |
| Includes | Landing page, quiz, 26 personalized email sequences, analytics dashboard, Vercel deployment |
| Revisions | Included |

### Vision Board Builder BUILD Service (One-Time)

| Item | Price |
|------|-------|
| Complete vision board builder | **$2,997** |
| Timeline | 7 days |
| Includes | Landing page, builder, Glif graphics, 10 email sequences, analytics dashboard, Vercel deployment |
| Revisions | Included |

### Bundle: Quiz + Vision Board

| Item | Price |
|------|-------|
| Quiz funnel + Vision board builder | **$5,497** |

### Funnel Optimizer RETAINER Service (Monthly)

| Tier | Price | Best For |
|------|-------|----------|
| Optimization Insights | $500/mo | Teams who implement themselves |
| Optimization Partner | $1,500/mo | Teams who want implementation done |
| Growth Accelerator | $3,000/mo | Funnels with significant traffic |

Sales collateral located in `marketing/strategy/`.

---

## Conventions

- Output goes in `output/[business-name]/` during active builds
- Completed funnels live on GitHub only -- `output/` is a working directory, cleared after publishing
- Always use MCP tools for research when available (do not generate generic content)
- Validation gates between agent stages (do not proceed if output is incomplete)
- Skills are defined in `.claude/skills/[skill-name]/SKILL.md`
- Agent definitions are in `agents/[workflow-name]/[agent-name]/SKILL.md`
- Design modes: Soft, Sharp, Glass, Glossy, Minimal (auto-detected from brand website)
- Question types must include 3+ different types per quiz (card_selection, scale_slider, multiple_choice, yes_no_toggle, tag_cloud, emoji_scale, star_rating)
- Social ad videos must show ALL profiles, never a subset
- Temperature (hot/warm/cold) is internal only -- never shown to quiz takers
- Update this file when adding or removing skills, agents, or workflow stages

---
> Source: [jpiece3/lead-magnet-quiz-workflow](https://github.com/jpiece3/lead-magnet-quiz-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
