## monican

> **Monican is pivoting from consulting to a multi-tenant AI platform.**

# Monican — Project Context

## CURRENT DIRECTION (read this first)

**Monican is pivoting from consulting to a multi-tenant AI platform.**

The vision: an agency that builds AI agent suites for every industry, starting with real estate. Clients sign up on a web app, set their profile, and their agents run within minutes. No n8n manual setup per client. See `PLATFORM_PLAN.md` for the complete architecture, phases, and decisions.

**Short version:** Next.js + Supabase Auth owns the UI/auth/credentials. n8n runs the agent logic via templates that Vercel clones per client via n8n's API. Gmail actions flow through a Vercel proxy endpoint that holds each client's OAuth token.

**Current phase:** Phase 1 — Foundation. Next session starts by reading `PLATFORM_PLAN.md` Section 7 and kicking off the Next.js rebuild.

**What's already built:**
- Lead Responder n8n workflow (Published — will become `template_lead_responder`)
- Past Client Reactivator n8n workflow (Published — will become `template_past_client_reactivator`)
- Static HTML dashboard at `monican-dashboard.vercel.app` (will be rebuilt as Next.js, old URL stays up)
- Supabase project with `prospects`, `outreach`, `revenue` tables (will add platform tables from PLATFORM_PLAN.md Section 2.1)

**Eileen Fitzpatrick (RE/MAX Traditions, Bolton) is NOT urgent.** Platform first, Eileen becomes the first migrated client once Phase 3 is done.

---

## What This Is (Business Model)

Monican is Daniel Weadock's AI platform company. Based in Bolton, MA. Real estate is the first vertical. Revenue comes from three streams:
1. **Short-term (now):** Consulting engagements — $500 Quick Wins and $1,500 Pilots — to fund the platform build
2. **Medium-term:** SaaS subscription tiers once the platform handles self-service signup (see PLATFORM_PLAN.md Section 4.2)
3. **Long-term:** Enterprise + white-label deals for agencies and brokerages

---

## Daniel's Background

- Recent Merrimack College grad (BS Business Admin, Corporate Finance)
- CFO at Kings Court LLC (Boston 420 events company) — built a full-stack Next.js + Supabase app with Claude Code
- Former Northwestern Mutual financial rep (insurance sales, cold calling)
- Built SolEdge autonomous crypto trading system
- Mind Studio AI Agent Certification, MA Insurance License
- Based in Bolton, MA
- Partner: Evan (shared access to dashboard)

---

## Core Positioning

"You're not selling software. You're selling the expertise to pick the right tool, set it up, and wire it into their business." The tool is cheap. Your value is knowing which one to use and making it actually work.

For the platform era, this evolves: *"AI agent suites for any industry. Sign up, set your profile, your agents are running in 10 minutes."*

---

## Tech Stack (Platform Era)

| Layer | Tech |
|---|---|
| Frontend + Auth | Next.js 14 (App Router) on Vercel + Supabase Auth |
| Database | Supabase Postgres |
| Agent Runtime | n8n Cloud (template workflows cloned per client via API) |
| Credential Proxy | Vercel API routes |
| AI | Anthropic Claude (Claude Haiku 4.5 for production, Opus for complex drafts) |

---

## Key Files

### Platform (new)
- `PLATFORM_PLAN.md` — **Master plan.** Read first every session. Architecture, phases, decisions.
- `Monican_Platform_Vision.pptx` — Slideshow walkthrough of the platform vision

### Consulting Era (still relevant, becomes internal tools)
- `prospects.xlsx` — 33 real estate agencies near Bolton (future beta users)
- `01_Discovery_Call_Framework.md` — Call script (becomes enterprise sales collateral)
- `02_Service_Menu_and_Pricing.md` — Pricing (informs SaaS tier design)
- `03_Agentic_AI_Tool_Stack.md` — Tool knowledge reference
- `04_How_Agentic_AI_Works.md` — Plain-English explainer (becomes landing page copy)
- `agents/past_client_reactivator/` — Working workflow source files
- `agents/templates/` — n8n setup guides, system prompts, test cases

### Live Infrastructure
- **n8n Cloud:** (configured via environment)
  - `Lead Responder v1` — Published
  - `Past Client Reactivator v1` — Published, runs Mondays 8am
- **Vercel:** Deployed (static HTML, Phase 1 will replace with Next.js app)
- **Supabase:** Configured via environment variables
- **Google Sheets:** Connected to n8n for lead logging

---

## Agent Commands (updated for platform era)

### "Where are we / what should we work on"
- Read `PLATFORM_PLAN.md` to understand current phase
- Check the TodoWrite list from the last session (if it persists) OR propose the next phase's first step
- Ask Daniel which phase he wants to advance

### "Start Phase 1" / "Build the platform foundation"
- Follow `PLATFORM_PLAN.md` Section 3, Phase 1
- Steps are listed in Section 7 ("Next Session Starting Point")
- Create the new Next.js project, wire up Supabase Auth, deploy

### "Continue from where we left off"
- Read the TodoWrite list
- Read the most recently modified files in the project
- Check git log of `monican-dashboard` and any new `monican-platform` repo

### "Add a new industry" (Phase 5 and beyond)
- Create a new industry pack folder: `industries/{industry_name}/`
- Design 2-4 core modules for that industry
- Write a new template n8n workflow per module
- Update the Supabase `clients.industry` enum
- Add the new industry to the landing page gallery

### Consulting-era commands (still work for short-term cashflow)

#### "Find prospects" / "Search for agencies"
- Search the web for real estate agencies (or other industries) near Bolton, MA
- Add new finds to `prospects.xlsx`
- Prioritize: independent agencies (HIGH), mid-size (MEDIUM), large franchises (LOW)

#### "Draft outreach" / "Write emails"
- Run: `python3 agents/outreach_drafter.py --priority HIGH`
- Or draft custom outreach for a specific prospect

#### "Prep me for a call with [agency]"
- Research the agency, fill in the call_prep template
- Save to `call_prep/[agency_name]_prep.md`

#### "Write a proposal for [agency]"
- Read discovery call notes, fill in `agents/templates/proposal.md`
- Save to `proposals/[agency_name]_proposal.md`

---

## Tone

Daniel is early-career, hungry, and scrappy. Be direct, practical, and action-oriented. No fluff. Every suggestion should end with a specific next step.

Daniel gets frustrated when sessions feel like they're going in circles or when Claude introduces unnecessary steps. If something is already working, don't touch it. If a plan is already decided, execute it — don't re-litigate.

Daniel prefers honest reality checks over optimistic hand-waving. When a plan hits a wall (like the FSBO scraper situation), flag it immediately and propose alternatives rather than pushing forward into a broken build.

---

## Security Notes

- Anthropic API keys have been pasted in chat multiple times. Always remind Daniel to rotate them after a build session.
- Supabase `anon` key is safe to ship in client code (RLS handles access).
- Supabase `service_role` key is NEVER in client code — only in Vercel server environment variables.
- n8n API key (to be generated in Phase 2) goes in Vercel env vars only.
- Gmail OAuth tokens (Phase 3) stored encrypted in Supabase, accessed only by Vercel server routes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AwkCode) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
