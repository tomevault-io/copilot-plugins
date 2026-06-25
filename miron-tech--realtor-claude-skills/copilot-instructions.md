## realtor-claude-skills

> 13 AI skills that turn Claude Code into your real estate business assistant — researching leads in seconds, generating listing marketing, building follow-up sequences, pulling live comps, creating listing presentations, and running your sphere program. Built for agents who want to work smarter, not just harder.

# Real Estate Agent Claude Skills — by CyclSales

## What This Is
13 AI skills that turn Claude Code into your real estate business assistant — researching leads in seconds, generating listing marketing, building follow-up sequences, pulling live comps, creating listing presentations, and running your sphere program. Built for agents who want to work smarter, not just harder.

## First Step On Any Task
1. Read `.claudeskills/start-here-SKILL.md` — it routes every request to the correct skill
2. Follow the skill's process and output format
3. All output follows `_system/output-format.md` formatting rules
4. All client data follows `_system/client-memory.md` persistence rules

## System Files
| File | Purpose |
|------|---------|
| `_system/output-format.md` | Visual design system — character palette, section order, templates |
| `_system/client-memory.md` | Persistent state — how to read/write `./clients/` directory |

## Data Sources

| Connection | What It Does | Required? |
|------------|-------------|-----------|
| **Perplexity** | Market data, school ratings, neighborhood intel, rent comps | Recommended |
| **Firecrawl** | Property details from Redfin/Zillow (sqft, beds/baths, price, photos) | Recommended |
| **Glif** | Branded graphics — Just Listed, Open House, market stat visuals | Optional |
| **Notion** | Pipeline tracking, content calendar, sphere database | Optional |
| **Playwright** | Browser automation for MLS/county lookups | Optional |

## How To Use
1. Clone this repo into your project directory (see README for setup)
2. Connect Perplexity and Firecrawl for live data (see README)
3. Tell Claude what you need — it picks the right skill automatically
4. Or be specific: "Run lead recon on 4821 Cedar Ln, Dallas TX 75214"

## Available Skills

| Skill | Use When |
|-------|----------|
| **Lead Recon** | A lead comes in — research property, score motivation, generate speed response |
| **Nurture Coach** | A lead went cold — build 7-touch follow-up sequence with scripts |
| **Listing Arsenal** | New listing signed — generate 25+ marketing assets from one address |
| **Comp Crusher** | Need pricing — pull live comps, 3-tier strategy, seller net sheet |
| **Listing Presentation** | Listing appointment — build a data-driven presentation that wins |
| **Open House Machine** | Open house coming — pre-event promo, day-of scripts, post-event follow-up |
| **Market Intel** | Need content — weekly market reports, social posts, newsletter segments |
| **Neighborhood Dominator** | Want to own an area — guides, SEO pages, 12-month content calendar |
| **Prospector** | Pipeline thin — expired listing, FSBO, and circle prospecting scripts |
| **Sphere Engine** | Stay top-of-mind — 12-month sphere of influence touch calendar |
| **Review Engine** | Deal just closed — review requests, referral sequences, testimonial extraction |
| **Investment Analyzer** | Investor client — cash flow, cap rate, BRRRR analysis, deal scorecard |
| **CyclSales Connect** | Push leads to CRM, view pipeline, sync contacts, trigger automated follow-up |

## How Claude Picks the Right Skill

```
Lead comes in (address, inquiry, phone call)     → Lead Recon
Lead went cold or stopped responding              → Nurture Coach
New listing signed, need marketing                → Listing Arsenal
Need comps, CMA, or pricing strategy              → Comp Crusher
Listing appointment coming up                     → Listing Presentation
Open house this weekend                           → Open House Machine
Need weekly content or market update              → Market Intel
Want to own a neighborhood or ZIP code            → Neighborhood Dominator
Need more listings, pipeline is thin              → Prospector
Past clients, sphere of influence                 → Sphere Engine
Deal just closed, need review + referral          → Review Engine
Working with an investor, need numbers            → Investment Analyzer
Push a lead to CRM, sync, view CRM pipeline       → CyclSales Connect
```

## How the Skills Work Together

Each skill passes information to the next, so you don't repeat yourself:

```
LEAD SOURCES (Zillow, sign call, open house, referral, web form)
                    ↓
              Lead Recon (research + speed response)
                    ↓
              Nurture Coach (follow-up until they convert)
                    ↓
    ┌──── Seller ────┤──── Buyer ────┬── Investor ──┐
    │                │               │              │
    Comp Crusher     │               │    Investment │
    (live comps)     │               │    Analyzer   │
    ↓                │               │              │
    Listing          │               │              │
    Presentation     │               │              │
    (win the listing)│               │              │
    ↓                │               │              │
    Listing Arsenal  │               │              │
    (25+ assets)     │               │              │
    ↓                │               │              │
    Open House       │               │              │
    Machine          │               │              │
    └────────┬───────┘               │              │
             ↓                       │              │
         DEAL CLOSES ←───────────────┘──────────────┘
             ↓
         Review & Referral Engine
             ↓
         Sphere Engine (keep them forever)
             ↓
         Pipeline thin? → Prospector (expired, FSBO, circle)

Any stage → CyclSales Connect (push to CRM → automations fire)
```

## Weekly Content Engine

```
Market Intel → generates weekly content package
    ├── Social posts (Instagram, Facebook, TikTok, LinkedIn, Nextdoor, GBP)
    ├── Email newsletter segment
    └── Market report (client-ready)

Neighborhood Dominator → generates long-term positioning
    ├── Area guide (lead magnet PDF)
    ├── SEO content pages
    ├── 12-month content calendar
    └── Google Business Profile strategy
```

## Context Matrix
Skills receive ONLY the data they need from prior skills. See `start-here-SKILL.md` for the full matrix. Key rule: selective context produces better output than dumping everything.

## Client Memory
All client data persists in `./clients/`. See `_system/client-memory.md` for the full protocol:
- Per-client folders with recon, sequences, comps, assets
- Market profile cache by ZIP (reusable across clients)
- Pipeline registry (buyer leads, seller leads, active listings)
- Sphere of influence tracking
- Content calendar
- Learnings journal (calibrates lead scoring over time)

## Time Savings

| Task | Before | After | Time Back |
|------|--------|-------|-----------|
| Research a lead + respond | 15-20 min | 60 seconds | ~18 min |
| New listing marketing (all assets) | 2-3 hours | 10 min | ~2.5 hours |
| Follow-up sequence (7 touches) | 30-45 min | 3 min | ~35 min |
| Comp analysis + pricing strategy | 45-60 min | 5 min | ~50 min |
| Listing presentation | 2-3 hours | 10 min | ~2.5 hours |
| Weekly market content | 3-4 hours | 15 min | ~3.5 hours |
| Expired/FSBO prospecting (per property) | 20-30 min | 2 min | ~25 min |
| Sphere touches (monthly) | 2-3 hours | 10 min | ~2.5 hours |
| Post-close review + referral | 20-30 min | 3 min | ~25 min |
| Investment analysis | 30-45 min | 5 min | ~35 min |
| Open house campaign (full cycle) | 2-3 hours | 15 min | ~2.5 hours |

**At 10 leads/week, 2 listings/month, and 2 closings/month: 8-12 hours back every week.**

## Built by CyclSales

These skills were built by [CyclSales](https://cyclsales.com/agents) — an AI-powered CRM for real estate professionals.

Claude Skills generate the strategy. CyclSales executes it on autopilot:
- Missed call text-back in under 60 seconds
- Automated follow-up sequences (the 7-touch cadences run themselves)
- Post-close review requests sent automatically
- Sphere nurture campaigns that run year after year
- Pipeline management with every lead in one place

**Claude is the brain. CyclSales is the body.**

The skills work great on their own. When you're ready for autopilot, we're here.

---
> Source: [miron-tech/realtor-claude-skills](https://github.com/miron-tech/realtor-claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
