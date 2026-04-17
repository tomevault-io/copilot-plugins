## linkedin-automation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Is

Automated LinkedIn content generation pipeline. Reads your profile and content pillars, generates posts/carousels/infographics in your voice, and presents everything in a review dashboard before you publish.

**Profile:** Defined in `context/profile.md` — this is the source of truth for who you are and how to write for you. If this file doesn't exist, copy from `context/profile.template.md`.

**Three-phase pipeline:** Research trends → Create content → Review in dashboard → Publish

> **First time?** See [SETUP.md](SETUP.md) for setup instructions.

---

## First Time?

Run `/setup` once after cloning. Claude will ask you 10 questions and write all config files automatically.

## Session Start

Run `/prime` at the start of every session. Claude will read all context files and confirm readiness.

---

## Commands

| Command | What it does |
|---------|-------------|
| `/prime` | Load all context files and initialize session |
| `/research` | Phase 1: Trend scan → hooks → weekly brief |
| `/create posts` | Generate 5 text posts (Story/Data/Contrarian each) |
| `/create carousel [topic]` | 10-slide carousel PDF via fal.ai nano-banana-pro |
| `/create infographic [topic]` | Portrait PNG infographic via fal.ai nano-banana-pro |
| `/create custom [context]` | Custom post/carousel/infographic from YOUR data |
| `/create batch` | Full week: 5 posts + 1 carousel + 1 infographic |
| `/dashboard build` | Build React content review dashboard |
| `/dashboard update` | Refresh dashboard with new output/ files |
| `/publish` | Publishing checklist + open dashboard |

---

## Content Pillars (4 pillars)

| # | Pillar | % of Content | Frequency |
|---|--------|-------------|-----------|
| 1 | Healthcare AI Insights | 35% | 2x/week |
| 2 | Technical Deep Dives | 25% | 1-2x/week |
| 3 | Career Journey & Personal Stories | 15% | 1x/week |
| 4 | Industry Commentary | 10% | 1x/2 weeks |
| 5 | Claude & AI Tools | 15% | 1x/week |

---

## Content Rules

- First line is ALWAYS a hook (pattern interrupt, bold claim, or curiosity gap)
- One idea per post — if you have 3 ideas, that's 3 posts
- Short paragraphs: 1-3 lines maximum, mobile-first
- Use specific numbers: ~93% accuracy, 200+ hours, 5 models tested
- Write at 6th-grade reading level — short sentences, no jargon without explanation
- 3-5 hashtags at post end ONLY (never in body, algorithm penalizes >5)
- NO corporate buzzwords (synergy, leverage, ecosystem, excited to announce)
- NO engagement bait ("Like if you agree") — ask specific questions instead
- Every post must reinforce the positioning: healthcare AI engineer who ships production systems
- Best posting time: 8-10 AM PKT (Tue-Thu perform best)
- Respond to every comment within 4 hours

---

## Project Structure

```
.claude/
  commands/          # Slash command definitions (short — call skills)
  skills/            # Detailed how-to docs for each content type
    research/        # SKILL.md — trend scanning, hook generation
    post-writing/    # SKILL.md — LinkedIn post rules and formats
    carousel-creation/   # SKILL.md — fal.ai carousel process
    infographic-creation/ # SKILL.md — fal.ai infographic process
    dashboard/       # SKILL.md — React dashboard build process
context/
  profile.md         # Who Naeem is — positioning, voice, tone
  strategy.md        # 4 pillars, format mix, engagement plan, 90-day metrics
  metrics.md         # Performance tracking — update weekly
reference/
  writing-style.md   # 10 golden rules, hook patterns, format templates
  post-examples.md   # Proven post templates
research/
  weekly-intel/      # Research briefs (YYYY-MM-DD.md)
  hooks-bank.md      # Tested hooks library — 4 pillars
  competitor-analysis.md  # Competitor content patterns
output/
  posts/             # Text posts (.md) — 3 versions per topic
  carousels/         # Carousel PDFs + slide PNGs
  infographics/      # Infographic PNGs
  images/            # Social images (.png)
dashboard/
  index.html         # Content review dashboard
scripts/
  generate-image.py      # Core fal.ai image generator
  generate-infographic.py # Infographic via fal.ai
  generate-carousel.py   # Carousel PDF via fal.ai
  research.sh            # CLI research runner
  generate-batch.sh      # CLI batch runner
  build-dashboard.sh     # CLI dashboard builder
templates/
  infographic-content.json  # Template for infographic content input
  carousel-content.json     # Template for carousel content input
config/
  content-pillars.json   # 4 pillar definitions with hashtags
  audience-personas.json # 5 target audience personas
  brand.json             # Colors, fonts, asset dimensions
```

---

## How Commands & Skills Work

**Commands** (`.claude/commands/`) are short — they name the task and point to a skill.
**Skills** (`.claude/skills/*/SKILL.md`) contain the full detailed instructions.

When executing any command, Claude reads the relevant skill file first, then runs the process.

---

## Image Generation (fal.ai nano-banana-pro)

All visuals (infographics, carousel slides) are generated via fal.ai:

```bash
# Single image
python3 scripts/generate-image.py --prompt "..." --output out.png --width 1080 --height 1080

# Infographic (reads content JSON)
python3 scripts/generate-infographic.py --content content.json --output out.png

# Carousel PDF (reads content JSON, generates all 10 slides)
python3 scripts/generate-carousel.py --content content.json --output out.pdf
```

**Setup (once):**
```bash
cp .env.example .env   # add FAL_AI_API_KEY
uv pip install Pillow python-dotenv
```

---

## Configuration Files

- [config/brand.json](config/brand.json) — authoritative colors, fonts, asset dimensions
- [config/content-pillars.json](config/content-pillars.json) — 4 pillar definitions
- [context/profile.md](context/profile.md) — positioning, voice, tone, key projects
- [context/strategy.md](context/strategy.md) — 90-day playbook, format mix, engagement strategy

---

## Weekly Workflow

```
Sunday/Monday   → /research          (~15 min)
Tuesday         → /create batch      (~30 min)
Tuesday         → /dashboard build   (~5 min)
Mon/Wed/Fri     → /publish           (~15 min each)
Daily           → 30 min engagement  (comment, respond, connect)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neemydev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
