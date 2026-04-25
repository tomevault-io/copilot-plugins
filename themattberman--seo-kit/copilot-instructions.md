## seo-kit

> **Purpose:** Keyword discovery, ranking monitoring, competitor gap analysis

# AGENTS.md - SEO Kit

## Skills

### skills/seo-agent — Discovery + Monitoring
**Purpose:** Keyword discovery, ranking monitoring, competitor gap analysis
**Scripts:** seo-discover.sh, seo-monitor.sh, seo-compete.sh
**Data Sources:** Google Search Console API, DataForSEO API
**Schedule:** Weekly (Monday discover + monitor, Friday compete)

### skills/seo-forge — Content Engine
**Purpose:** Brand-voice content generation with psychology frameworks
**Scripts:** seo-interview.sh, seo-research.sh, seo-check.sh
**Modes:** Interview (new brands), Refinement (weekly), Default (no brand context)
**Output:** Draft articles in markdown with frontmatter

### skills/seo-links — Backlink Acquisition
**Purpose:** Competitor backlink mining, unlinked mentions, broken link building, internal link audit, resource page prospecting
**Scripts:** link-mine.sh, link-mentions.sh, link-broken.sh, link-internal.sh, link-prospect.sh
**Data Sources:** DataForSEO Backlinks API (optional), web search fallback
**Schedule:** Weekly (one script per day, Mon-Fri)

### skills/seo-health — Technical SEO Monitoring
**Purpose:** PageSpeed audits, Core Web Vitals tracking, crawl health, image optimization
**Scripts:** health-speed.sh, health-crawl.sh, health-images.sh
**Data Sources:** Google PageSpeed Insights API (free), direct crawl
**Schedule:** Weekly (Monday morning)

### skills/seo-checklist — Reference
**Purpose:** Technical SEO reference (meta tags, schema, llms.txt, topical authority)
**Type:** Reference document, not executable
**Usage:** Agent consults this when publishing or auditing pages

## The Complete Loop

```
1. seo-check.sh        → Verify connections (GSC + DataForSEO)
2. seo-interview.sh    → Build brand voice (first run)
3. health-speed.sh     → Check site performance baseline
4. health-crawl.sh     → Fix technical issues first
5. seo-discover.sh     → Find keyword opportunities
6. seo-forge SKILL     → Write content targeting top opportunities
7. link-internal.sh    → Add internal links to new + existing content
8. seo-monitor.sh      → Track what ranked, find strike zone pushes
9. link-mine.sh        → Find backlink opportunities
10. link-mentions.sh   → Find unlinked brand mentions
11. seo-compete.sh     → Competitor gap analysis
12. Repeat 5-11 weekly
```

## Data Files

| File | Purpose |
|------|---------|
| `workspace/brand/voice-profile.md` | Brand voice from interview |
| `workspace/brand/audience.md` | Target audience profile |
| `workspace/brand/positioning.md` | Competitive positioning |
| `workspace/seo/rankings-snapshot.json` | Latest ranking data |
| `workspace/seo/opportunities.json` | Prioritized keyword list |
| `workspace/seo/competitor-gaps.json` | Keywords competitors have, you don't |
| `workspace/seo/health/speed-*.json` | PageSpeed snapshots over time |
| `workspace/seo/health/crawl-*.json` | Crawl health snapshots |
| `workspace/seo/health/images-*.json` | Image audit results |

---
> Source: [TheMattBerman/seo-kit](https://github.com/TheMattBerman/seo-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
