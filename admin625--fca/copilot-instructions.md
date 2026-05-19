## fca

> FCA is an AI-powered social media content platform for fitness studios. It generates captions, images, and video reels tailored to each studio's brand, instructors, and schedule.

# FCA — Fitness Content Agent

## What This Project Is
FCA is an AI-powered social media content platform for fitness studios. It generates captions, images, and video reels tailored to each studio's brand, instructors, and schedule.

## Pricing
FCA Studio is $599/mo standard, with quarterly ($1,617/3mo, ~10% savings) and annual ($5,750/yr, ~20% savings) commitment options.

The first 100 founding studios pay $299/mo, locked in for life — quarterly $807/3mo and annual $2,870/yr also available at founding rates. Founding pricing is automatic at signup based on availability; no coupon code required.

**Note:** $299/$599 is all-in pricing per HQ — no per-seat add-on. Instructor seats are included. Founding cohort gating is server-side via the `claim_founder_slot` RPC, not a coupon code.

## Launch Status
- Launched March 2026
- Active beta at **The Local Kollective** (instructors: Katie, Jackie, Ella)
- Core features live: AI content generation, photo editor, video reels, instructor seat management, Stripe billing

## Stack
| Layer | Service | Detail |
|---|---|---|
| Frontend | Netlify | `studio-dash.netlify.app` |
| Backend / API | Digital Ocean | Node.js server |
| Database | Supabase | Project ID: `kidgcrqxrfcbsaeguwop` |
| Payments | Stripe | Webhook workflow: `q7IuW7Q85mpgOYub` on n8n |
| Automation | n8n | `jmac.app.n8n.cloud` |

## Supabase Schema — Key Tables
- `studios` — one row per studio account
- `trials` — auto-created via DB trigger on studio insert
- `studio_instructors` — instructor seats per studio
- `leads` — outreach prospects (Lead Gen)
- `lc_leads` — leads from The Local Kollective channel
- `lc_lead_events` — activity events on lc_leads
- `clients` — active paying clients
- `content_deliveries` — generated content records
- `fitness_content` — content library
- `email_templates` — outreach email templates
- `commission_ledger` / `commission_payouts` — affiliate/sales rep tracking
- `referral_sources` / `search_keywords` — attribution data
- `trial_config` — global trial settings
- `agents` — AI agent configurations
- `reel_music_library` — music assets for reels

### Views (read-only, do not modify)
- `v_attribution_dashboard`
- `v_channel_performance`
- `v_hot_leads_today`
- `v_monthly_payout_report`
- `v_sales_rep_dashboard`

## Trial System
Every new studio gets a free trial automatically on signup. No credit card required.

**Trial limits:**
- 5 days access, OR
- 5 generated posts
Whichever is hit first ends the trial.

**Trial statuses:** `active`, `expired`, `converted`

**Beta accounts:** Any studio with `is_beta = true` on `studios` is permanently exempt from all trial logic. Never modify trial behavior for beta accounts.

**Attribution:** The `trials` table has `source` (text) and `lead_gen_id` (text) columns. When a studio signs up via an outreach link, these are populated from URL params:
```
https://studio-dash.netlify.app/signup?source=email_outreach_march2026&lead_gen_id=abc123
```
`lead_gen_id` maps to `leads.id` (uuid). Two DB triggers handle attribution back to the `leads` table:
- `on_trial_created` — sets `signup_completed_at` and `trial_started = true` on INSERT
- `on_trial_converted` — sets `converted = true` on UPDATE when `trial_status = converted`

**Conversion:** When a studio converts, the Stripe webhook fires → n8n workflow `q7IuW7Q85mpgOYub` → sets `trial_status = converted` and `converted_at = NOW()` on the `trials` row.

## Current Focus
Customer acquisition and lead generation. The trial system and attribution loop are the active build area.

## Important Rules
- **Never touch beta studio accounts** (`is_beta = true`)
- **Never modify existing triggers** without reading them first — there is already a trigger on `studios` INSERT that creates the `trials` row
- **Always check views before querying raw tables** for reporting — the views are pre-built for attribution and performance data
- **The Stripe webhook workflow ID is `q7IuW7Q85mpgOYub`** — reference by ID, do not recreate
- Signup URL is `https://studio-dash.netlify.app/signup`

## Repo Activity Note (2026-05-05)
This `admin625/FCA` repo last received commits on 2026-03-19 (~7 weeks ago). Active development has shifted to `admin625/studio-dashboard` (current dashboard) and `admin625/fca-leadgen` (acquisition pipeline). Treat this repo as historical reference for the early-2026 launch state.

---
> Source: [admin625/FCA](https://github.com/admin625/FCA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
