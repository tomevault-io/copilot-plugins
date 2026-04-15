## decibel

> Decibel is a two-sided platform connecting live performers with fans who were on their dancefloor.

# DECIBEL

## What This Is
Decibel is a two-sided platform connecting live performers with fans who were on their dancefloor.
- For fans: a live music passport with tiered access rewards (1/3/5/10+ scans unlock more)
- For performers: verified audience ownership with direct messaging, analytics, and booking leverage
- Tagline: "The more you show up, the more you get in."

## Tech Stack
- **Web:** Next.js 15 + TypeScript + Tailwind CSS
- **Mobile:** React Native with Expo (for location-based detection + push notifications)
- **Backend:** Supabase (PostgreSQL + Auth + Realtime + Storage + Edge Functions)
- **Hosting:** Vercel (web), Expo EAS (mobile builds)
- **Notifications:** Firebase Cloud Messaging (push), Twilio (SMS), SendGrid (email)
- **Scraping:** Node.js + Playwright + SoundCloud API + Mixcloud API
- **Image Gen:** React components → PNG via Playwright screenshots

## Design Aesthetic
Dark, underground, not corporate. Think Nerve movie aesthetic.
- Dark backgrounds (#0B0B0F)
- Accent colors: Pink (#FF4D6A), Purple (#9B6DFF), Blue (#4D9AFF), Teal (#00D4AA), Yellow (#FFD700)
- Font: Poppins (headers bold, body regular)
- Never use generic AI aesthetics (no Inter, no purple gradients on white, no cookie-cutter layouts)
- Motion and animations should feel alive but not flashy
- The product should feel like it belongs in an underground music scene

## Key Product Rules
- Fans are ALWAYS free. Never charge fans.
- QR/NFC capture must work in under 5 seconds with no app install required (mobile web)
- Location-based passive detection is the PRIMARY capture method after first scan
- Every performer gets a pre-built profile from scraped data before they claim it
- Fan passport pre-populates via email receipt parsing (AXS, DICE, Eventbrite, Ticketmaster)
- Tiered access: 1 scan = network, 3 = early access, 5 = secret shows, 10+ = inner circle

## Core Database Tables
- performers (profile, social links, claimed status)
- fans (profile, app install status)
- collections (fan_id, performer_id, venue_id, capture_method, verified)
- venues (name, coordinates, geofence_radius)
- events (performer_id, venue_id, date, is_live)
- messages (performer_id, subject, body, stats)
- fan_tiers (fan_id, performer_id, scan_count, current_tier)

## Context Management
When you notice context usage above 50%, run /compact immediately.
Do not wait until 70%+. Quality degrades above 50%.

## Commands
- `npm run dev` — Start Next.js dev server
- `npm run build` — Production build
- `npx expo start` — Start Expo dev server for mobile

## File Structure
- `src/app/` — Next.js app router pages
- `src/components/` — Shared React components
- `src/lib/` — Supabase client, utilities, types
- `scripts/scrapers/` — SoundCloud, RA, DICE scraper scripts
- `content/output/` — Generated Instagram content (images + captions)
- `.claude/skills/` — Installed skills

## Important Context
- Master product doc: See decibel_master_document.docx in project root
- Competitor to study: NoSongRequests.com (16K performers, but wedding/corporate focus)
- Key differentiator: verified physical attendance + tiered access + performer-owned audience
- City focus: Chicago first, then NYC/Detroit
- Target venues: Smartbar, Soundbar, Spybar, 309 N Morgan, Podlasie

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/swarn2099)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/swarn2099)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
