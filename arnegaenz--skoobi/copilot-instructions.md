## skoobi

> The collectibles platform. Books first, then everything.

# Skoobi

The collectibles platform. Books first, then everything.

## Tech Stack
- TypeScript everywhere
- Next.js (frontend)
- Supabase (database, auth, storage)
- Vercel (hosting)
- Stripe (payments, when needed)

## Infrastructure
- Domain: skoobi.com (GoDaddy DNS → Vercel)
- GitHub: arnegaenz/skoobi
- Vercel project: skoobi
- Email: arne@skoobi.com (Microsoft 365)

## Brand
- Colors: #00E5A0 (mint), #00D4FF (cyan), #8B5CF6 (purple)
- Background: #08080F
- Dark/premium aesthetic

## DNS
- **skoobi.com** — A record → 76.76.21.21 (Vercel). Was missing, re-added April 2026.
- **www.skoobi.com** — CNAME → cname.vercel-dns.com. Added as domain in Vercel project April 2026.
- DNS managed at GoDaddy

## app-ads.txt
- Served at skoobi.com, www.skoobi.com, and labs.skoobi.com
- Content: `google.com, pub-2092614136459898, DIRECT, f08c47fec0942fa0`
- Required for AdMob ad serving on HearVerse (App Store ID 6761432386)
- AdMob verification was failing because the A record for skoobi.com was missing — fixed April 7 2026
- AdMob app-ads.txt verified successfully April 8 2026 — ads now serving

## Parent
Built by SkoobiLabs (labs.skoobi.com)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arnegaenz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
