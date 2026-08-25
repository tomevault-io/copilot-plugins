## pledgemeaning-site

> Definition funnel site for people searching "what is a pledge in Star Citizen"

# CLAUDE.md — pledgemeaning.com

## Project Overview
Definition funnel site for people searching "what is a pledge in Star Citizen"
or "what does pledge mean in gaming." Explains SC's unique pledge system (buying
ships = supporting development), then funnels to dayonecitizen.com and referral.

## Quick Reference
```
Referral code:  STAR-GCQJ-N6NC
Enlist URL:     https://www.robertsspaceindustries.com/enlist?referral=STAR-GCQJ-N6NC
Hub:            https://dayonecitizen.com
Conventions:    E:\Claude Code\sc-portfolio\SHARED_CONVENTIONS.md
```

## Agentic Build Instructions
Incremental agents. Confirm each before proceeding.

### Agent 1 — Scaffold
- Next.js 14, TypeScript, Tailwind CSS
- Color palette:
    midnight: '#0f1420'
    midnightMid: '#171e2e'
    amber: '#ff9c00'
    amberDark: '#cc7d00'
    cream: '#f5f0e8'
    muted: '#7a8099'
- 12 hero image placeholders
- Confirm dev server

### Agent 2 — Components
- HeroCarousel.tsx (12 slides)
- CTAButton.tsx: "Create Your Account — Get 50,000 UEC Free"
- SecondaryLink.tsx → dayonecitizen.com
- DefinitionCard.tsx: term + plain definition + example
- Footer.tsx

### Agent 3 — Homepage (/)
  - SEO: title="What Does Pledge Mean in Star Citizen? | pledgemeaning.com"
    description="In Star Citizen, a 'pledge' is a purchase from the RSI store
    that funds game development. Here's everything you need to know."
  - H1: "What Does 'Pledge' Mean in Star Citizen?"
  - Immediate above-fold answer
  - HeroCarousel
  - Sections:
    * "The Quick Answer" — pledge = purchase that funds development
    * "Pledge vs. Buy — What's the Difference?"
    * "What Can You Pledge For?" — ships, packages, gear
    * "Do You Need to Pledge to Play?" — starter packs, free fly
    * "What Is a Starter Package?" — recommended for new players
    * "The Referral Bonus" — use code to maximize first pledge value
    * CTAButton
    * Related terms → dayonecitizen.com/glossary
  - Footer
  Confirm renders.

### Agent 4 — Related Terms Pages
  - /what-is-uec: UEC definition + dayonecitizen.com link
  - /what-is-lti: LTI definition
  - /what-is-ccu: CCU definition
  Each: SEO optimized, CTAButton, internal links

### Agent 5 — SEO & Build
  - Sitemap, robots, OG meta, FAQPage schema
  - npm run build passes

## Color Palette
  --midnight: #0f1420
  --midnight-mid: #171e2e
  --amber: #ff9c00
  --amber-dark: #cc7d00
  --cream: #f5f0e8
  --muted: #7a8099

## Network Conventions
See `E:\Claude Code\sc-portfolio\SHARED_CONVENTIONS.md` for footer spec,
tone rules, commit convention, tech stack, and agentic build pattern.

## Click Tracking

Added 2026-05-17. Every referral CTA click fires a background POST to `/api/log` which writes a row to the shared Google Sheet and posts an embed to the #referral-clicks Discord channel.

**Env vars required** (Vercel project settings + `.env.local`):
- `CLICK_TRACKER_SHEET_URL` — Google Apps Script web app deploy URL
- `DISCORD_CLICK_WEBHOOK_URL` — Discord channel webhook URL

**Key files:**
- `src/app/api/log/route.ts` — server-side handler (parallel Sheet + Discord calls)
- `src/components/CTAButton.tsx` — `handleClick` fires the fetch on CTA click

> **Note:** Endpoint was renamed from `/api/track-click` → `/api/log` because adblocker filter lists (EasyPrivacy, uBlock Origin) blocked the original URL pattern client-side.

### Abuse guards (added 2026-07-14, network-wide)

After a bot replayed 26 identical click POSTs in 31s (2026-07-15 03:37 UTC), `/api/log` gained in-route guards — identical implementation in every network repo:

- **Same-origin check** — POSTs without a matching `Origin`/`Referer` host → 403 (blocks curl/replay loops)
- **Per-IP throttle** — max 10 events/min/IP → 429 (in-memory; effective because Fluid Compute reuses instances)
- **Dedupe** — identical `(ip, site, label, page)` within 30s is dropped silently
- **Discord flood breaker** — max 5 embeds/min, then one "alerts muted" summary embed; Sheet logging continues
- **Forensics** — payload now includes `ipHash` + `userAgent` (Apps Script must be updated to add columns before they appear in the Sheet)

### TODO: Verify end-to-end on this site
- [ ] Click CTA **with** adblocker enabled → Sheet row appears within 5s
- [ ] Click CTA **with** adblocker enabled → Discord embed appears in #referral-clicks
- [ ] Click CTA **without** adblocker → same as above

---

## SEO

**Network role:** Definition funnel satellite
**Search engine:** Both

**Primary keywords:**
- "what does pledge mean in star citizen"
- "star citizen pledge"
- "what is a pledge star citizen"

**Cross-links this site must send (body copy only):**
- → dayonecitizen.com: "New player guide" CTA
- → screferralreward.com: "Get a referral bonus when you make your first pledge"

**Do not:**
- Place cross-portfolio links in footers or link lists — editorial/body copy only

**Full strategy:** `E:\Claude Code\sc-portfolio\docs\seo\README.md`

## Claims ledger (network-wide)

Fact-checked claims are tracked in the portfolio claims ledger: `E:\Claude Code\sc-portfolio\docs\claims\` — one md file per claim (canonical text, status, official sources, lastVerified, usage map). Before adding or rewording a factual claim on this site, grep the ledger; a `verified` entry is network canon. Any edit that adds, changes, or removes a factual claim must update the matching claim file's `usage` list (or create the file — see the ledger README.md).

---
> Source: [doc-flanigan/pledgemeaning-site](https://github.com/doc-flanigan/pledgemeaning-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
