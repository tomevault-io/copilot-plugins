## clawfu-skills

> **ClawFu** 🦞🥋 - "I know ClawFu" - Download marketing skills directly into Claude.

# CLAUDE.md - ClawFu (ex-MKTG Skills)

## Project Overview

**ClawFu** 🦞🥋 - "I know ClawFu" - Download marketing skills directly into Claude.

> Like Neo downloading Kung Fu in The Matrix, ClawFu lets you instantly acquire expert marketing capabilities.

**Stats**: 119 skills | 20 domains | 25+ expert methodologies | 50+ templates | 25 automation scripts

**Brand Universe**: The Lobster Matrix
- **Gammarus** (creator) = Morpheus - the guide who reveals the truth
- **ClawFu Skills** = the programs Neo downloads
- **Users** = Neo - ready to master marketing

**Differentiator**: Skills built on quality inputs (April Dunford, Eugene Schwartz, Ogilvy, Cialdini, Hormozi) with professional structure, concrete examples, and verifiable references.

**Business Model**: 79€ one-time (100 first) → 149€/year subscription

**Domain**: clawfu.com ✅ (OVH)

**Brand Doc**: See `docs/brand-clawfu.md` for full identity guidelines.

---

## The 201 Gap (Core Positioning)

> "The training market has bifurcated into 101 (basics) and 401 (technical implementation). It has skipped the middle entirely, and the middle is where most of the productivity gains live."

**GUIA fills the 201 Gap** — the applied judgment layer between "how to prompt" and "how to build AI agents."

### The 6 Skills That Actually Matter (201 Level)

None of these are prompting techniques:

| Skill | Description | How GUIA Addresses It |
|-------|-------------|----------------------|
| **Context Assembly** | Knowing what info to provide, from which sources | Templates prepare optimal context |
| **Quality Judgment** | Knowing when to trust output vs verify | Skill Boundaries + Checkpoints |
| **Task Decomposition** | Breaking work into AI-appropriate chunks | Step-by-step Instructions |
| **Iterative Refinement** | Moving from 70% to 95% through passes | Iteration Guide section |
| **Workflow Integration** | AI embedded in work, not separate activity | Use Cases + How to Use |
| **Frontier Recognition** | Knowing where AI excels vs fails for YOUR work | Skill Boundaries section |

### Centaur vs Cyborg Modes

Skills are tagged by work pattern (Harvard/BCG study):

| Mode | Icon | Description | Best for |
|------|------|-------------|----------|
| **Centaur** | 🐴 | Divided work — human strategy/judgment, AI executes | High-stakes, accountability needed |
| **Cyborg** | 🤖 | Integrated workflow — fluid human/AI collaboration | Creative, iterative, building |
| **Both** | ⚡ | Flexible — use either depending on context | Thinking tools, flexible frameworks |

**Distribution**: 52 Centaur (57%) | 26 Cyborg (29%) | 13 Both (14%)

### Key Insight

> "AI isn't a tool skill. It's a management skill. The best users of AI are good managers."

Skills teach management principles applied to AI collaboration, not prompting tricks.

---

## Key Documents

| Document | Purpose |
|----------|---------|
| `docs/positioning-guia-relaunch-2026.md` | Full positioning analysis (Dunford 5+1) |
| `docs/skills-catalog-2026.md` | Complete skills inventory + gap analysis + Mode tags |
| `docs/sources-strategy.md` | Source acquisition strategy |
| `docs/insights-201-gap-2026.md` | Research insights backing the 201 positioning |
| `docs/skills-mode-tags.md` | Full Centaur/Cyborg/Both tagging for all 91 skills |
| `docs/linkedin-posts-201-gap.md` | LinkedIn content campaign (6 posts ready) |
| `templates/skill-template-v2.md` | Updated template with 201 sections |

---

## Project Structure

```
MKTG_Skills/
├── docs/                   # Strategy & business docs
│   ├── positioning-guia-relaunch-2026.md  # Positioning strategy
│   ├── skills-catalog-2026.md             # Full skills catalog
│   └── sources-strategy.md                # Source acquisition
├── skills/                 # The actual SKILL.md files (91 total)
│   ├── strategy/           # 16 skills - Positioning, pricing, analysis
│   ├── content/            # 18 skills - Copywriting, storytelling
│   ├── audio/              # 16 skills - Podcast, voice, sound design
│   ├── video/              # 5 skills - AI video production
│   ├── sales/              # 4 skills - Pitch, negotiation
│   ├── validation/         # 8 skills - Research, interviews
│   ├── thinking/           # 3 skills - Decision frameworks
│   ├── startup/            # 3 skills - Pitch decks, metrics
│   ├── product/            # 3 skills - Discovery, sprints
│   ├── leadership/         # 3 skills - Management
│   ├── funnels/            # 2 skills - Launch, sequences
│   ├── growth/             # 2 skills - PLG, loops
│   ├── branding/           # 1 skill - Brand strategy
│   ├── acquisition/        # 1 skill - Google Ads
│   └── meta/               # 1 skill - Orchestration
├── sources/                # Input documents for skill creation
│   ├── books/              # 34 expert book summaries
│   └── youtube/            # Video transcripts
│       └── channels/       # Organized by channel
├── templates/              # Skill template, production guidelines
├── scripts/                # Automation scripts
│   └── youtube_extractor.py
└── website/                # Next.js frontend (future)
```

---

## Skill Development Workflow

### 1. Source Collection

**Books/Courses** - Manual summaries in `sources/books/`
```bash
sources/books/april-dunford-obviously-awesome.md
```

**YouTube Videos** - Automated transcript extraction
```bash
# Extract transcript from YouTube
python scripts/youtube_extractor.py "URL" \
  --channel "channel-slug" \
  --title "Video Title" \
  --tags "tag1,tag2"

# Output: sources/youtube/channels/{channel}/{video-slug}.md
```

**See** `docs/sources-strategy.md` for full source acquisition strategy.

### 2. Create Skill
```bash
# Follow template in templates/skill-template.md
# Output to skills/[category]/[skill-name]/SKILL.md
```

### 3. Quality Review
Apply checklist:
- [ ] Methodology sourcée et crédible
- [ ] Structure conforme au template v2
- [ ] Minimum 2 exemples concrets
- [ ] Instructions actionables
- [ ] Références vérifiables
- [ ] **NEW**: "What Claude Does vs What You Decide" section present
- [ ] **NEW**: "Skill Boundaries" with clear limitations
- [ ] **NEW**: "Iteration Guide" with follow-up prompts
- [ ] **NEW**: Mode tag in metadata (centaur/cyborg/both)

---

## Skill Template Structure (v2)

Every skill MUST follow this structure (template v2 with 201 sections):

```markdown
# [Skill Name]
> [One-liner with methodology source]

## When to Use This Skill
## Methodology Foundation
## What Claude Does vs What You Decide     # NEW: 201 section
## What This Skill Does
## How to Use
## Instructions (detailed steps)
## Examples (minimum 2)
## Skill Boundaries (Frontier Recognition)  # NEW: 201 section
## Iteration Guide                          # NEW: 201 section
## Learning Curve                           # NEW: 201 section
## Checklists & Templates
## References
## Related Skills
## Skill Metadata (with mode: centaur|cyborg|both)
```

**See** `templates/skill-template-v2.md` for full template with guidance.

---

## Commands

### Source Acquisition
```bash
# Extract YouTube transcript
python scripts/youtube_extractor.py "https://youtube.com/watch?v=XXX" \
  --channel "ai-engineer" \
  --title "Video Title" \
  --tags "marketing,strategy"

# Print transcript without saving
python scripts/youtube_extractor.py "URL" --print-only
```

### Skill Production
```bash
# Validate all skills in a category
python scripts/validate_skills.py skills/audio/

# Validate single skill
python scripts/validate_skills.py skills/audio/podcast-production/SKILL.md

# Validate ALL skills
python scripts/validate_skills.py

# Generate skill from source (legacy)
python scripts/generate_skill.py sources/books/some-book.md \
  --category content --name copywriting-expert

# Build skills catalog
python scripts/build_catalog.py
```

---

## Skills Overview (91 Total)

### By Domain

| Domain | Count | Key Skills |
|--------|-------|------------|
| **Strategy** | 16 | positioning, competitive-analysis, grand-slam-offers, jobs-to-be-done |
| **Content/Copy** | 18 | copywriting-schwartz, copywriting-ogilvy, persuasion-cialdini, storytelling-storybrand |
| **Audio** | 16 | podcast-production, sound-design-murch, sonic-branding, voice-design |
| **Video** | 5 | ai-video-concept, ai-storyboard-2x2, ai-video-prompting |
| **Validation** | 8 | mom-test, customer-discovery, lean-canvas, problem-interview |
| **Sales** | 4 | sales-pitch-dunford, never-split-difference, spin-selling, challenger-sale |
| **Thinking** | 3 | second-order-thinking, regret-minimization, reversible-decisions |
| **Startup** | 3 | yc-pitch-deck, startup-metrics, fundraising-narrative |
| **Product** | 3 | product-discovery, shape-up, design-sprint |
| **Leadership** | 3 | high-output-management, radical-candor, one-on-ones |
| **Funnels** | 2 | dotcom-secrets, launch-formula |
| **Growth** | 2 | product-led-growth, growth-loops |
| **Other** | 3 | brand-strategy, google-ads-expert, skill-orchestrator |

### Priority Gaps (P1)

| Domain | Skill Needed | Why |
|--------|--------------|-----|
| **GEO** | llm-optimization | Core differentiator |
| **Social** | linkedin-content-strategy | Launch requirement |
| **Branding** | brand-voice-guide | Positioning deliverable |
| **Analytics** | ga4-setup-guide | Common need |
| **Email** | welcome-sequence | High conversion impact |

**Full catalog**: See `docs/skills-catalog-2026.md`

---

## Tech Stack

**Skills**: Pure Markdown (SKILL.md standard)
**Website** (future): Next.js 14, Tailwind, shadcn/ui, Supabase, Stripe
**Automation**: Python scripts for validation/generation

---

## References

**Internal Docs**:
- `docs/positioning-guia-relaunch-2026.md` - Full positioning strategy
- `docs/skills-catalog-2026.md` - Complete skills inventory
- `docs/sources-strategy.md` - Source acquisition

**External References**:
- [Claude Skills Documentation](https://code.claude.com/docs/en/skills)
- [Agent Skills Spec](https://agentskills.io)
- [Swipefile.com](https://swipefile.com) - Catalog UI inspiration
- [SkillsMP Marketplace](https://skillsmp.com) - Competitor reference

**Competency Framework**:
- [Marketing Competency Matrix](https://docs.google.com/spreadsheets/d/18QHznHaw_-kPXOTRnnXuBWxJTzEfWY46lJ3p3Ze_BmA/) - 15 domains, 80+ clusters, 1000+ competencies

---
> Source: [guia-matthieu/clawfu-skills](https://github.com/guia-matthieu/clawfu-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
