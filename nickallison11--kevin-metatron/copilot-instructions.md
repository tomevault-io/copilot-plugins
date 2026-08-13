## kevin-metatron

> metatron is a global platform connecting founders, investors, and ecosystem partners via on-chain anchored pitch data and AI-powered matching. Co-founded by Nick Allison (technical lead) and Rianna (business development/investor relations). The project is venture-backed by Flori Ventures and uses Apache-2.0 licensing.

# CLAUDE.md — kevin_metatron Project Context

## What is metatron?

metatron is a global platform connecting founders, investors, and ecosystem partners via on-chain anchored pitch data and AI-powered matching. Co-founded by Nick Allison (technical lead) and Rianna (business development/investor relations). The project is venture-backed by Flori Ventures and uses Apache-2.0 licensing.

Nick has spoken to founders across Africa, Asia, India, USA, Europe, LATAM, and Australia, maintaining connections with angels and VCs across multiple regions.

## Domain & Product Structure

| Domain | Purpose |
|---|---|
| `metatrondao.io` | DAO / foundation layer (stays as-is, will become metatron foundation) |
| `metatron.id` | Consumer product / Kevin landing page. Links to Kevin on Telegram/WhatsApp/email and to the platform |
| `platform.metatron.id` | Web app (this repo replaces the legacy CRA version) |
| `agent.metatrondao.io` | OpenClaw web interface (stays as-is) |

## Repository Structure

This is a monorepo at `github.com/nickallison11/kevin_metatron`:

```
kevin_metatron/
├── frontend/          # Next.js + Tailwind CSS + TypeScript
├── backend/           # Rust / Axum API server
│   └── migrations/    # PostgreSQL migrations (sqlx)
├── solana/            # metatron_core Solana program
├── reference/         # platform-live design reference files
├── docker-compose.yml # PostgreSQL container
└── CLAUDE.md          # This file
```

### Frontend (Next.js)
- Port: 3000
- Pages: Landing/role selection, auth/signup, startup dashboard, investor dashboard, connector dashboard
- API calls go to `NEXT_PUBLIC_API_BASE_URL` (default `http://localhost:4000`)

### Backend (Rust/Axum)
- Port: 4000
- Routes: auth (JWT + Argon2), pitches, pools, investments, compliance
- Database: PostgreSQL via sqlx
- Env vars: `BACKEND_DATABASE_URL`, `BACKEND_JWT_SECRET`, `RUST_LOG`

### Solana Program
- `metatron_core` — handles pitch hashes, pool manifests, stablecoin commitments
- Developed with Anchor framework

### Running locally
```bash
docker compose up -d          # Start PostgreSQL
cd backend && cargo run       # Start Rust API on :4000
cd frontend && npm run dev    # Start Next.js on :3000
```

## Design System (metatron.id)

All UI must match the metatron.id design system. **Never deviate from these tokens.**

| Token | Value |
|---|---|
| Font (body) | DM Sans (Google Fonts) |
| Font (mono/labels) | JetBrains Mono |
| Background | `#0a0a0f` |
| Card background | `#16161f` |
| Text | `#e8e8ed` |
| Muted text | `#8888a0` |
| Accent (primary) | `#6c5ce7` (purple) |
| Accent glow | `rgba(108,92,231,0.2)` |
| Borders | `rgba(255,255,255,0.06)`, 1px solid |
| Surface overlays (dividers, pill backgrounds, hover states) | `var(--overlay-2/3/4/6/8/12)` — never raw `rgba(255,255,255,0.0X)`, which is invisible against light-mode's light surfaces. Dark mode = white tints, light mode = matching black tints (defined in `globals.css`). |
| Border radius | 12px |
| Nav | Sticky, `rgba(10,10,15,0.85)`, `backdrop-filter: blur(12px)`, border-bottom `rgba(255,255,255,0.06)` |
| Logo | `https://metatron.id/metatron-logo.png` at 42px height, no separate wordmark. NEVER use the WordPress `/wp-content/...` URL — metatron.id is no longer on WordPress. |
| Background effects | 52px grid (`grid-bg`) + purple orb glow (radial-gradient) |

The reference design files are in `reference/platform-live/` (App.js, App.css from the live CRA version on KVM2).

## User Roles

Three roles on the platform:
1. **Founder** — creates pitch profiles, uploads decks, records calls
2. **Investor** — browses founders, requests intros, manages deal flow
3. **Connector** — ecosystem partners with their own dashboard for managing their investor network contacts (`connector_network_contacts` table). This is live and already populated by connectors.

Authentication: Connect Solana wallet (Phantom/Solflare) **OR** email/password (both options).

## Feature Roadmap (build in this order)

### ✅ Completed
- metatron.id design system applied to Next.js frontend
- Role selection (Founder/Investor/Connector) on landing page
- Signup with email/password → JWT auth
- Startup and investor dashboard shells
- Rust backend with auth, pitches, pools, investments, compliance routes
- Telegram bot (wallet_bot.py on KVM2) with /pitch, /investor, /find, /findinvestor, /intro, /approve, /reject
- IPFS profile storage via Pinata
- MTN token gating on Telegram (10k tMTN for Kevin access)

### ✅ Completed
- metatron.id design system applied to Next.js frontend
- Role selection (Founder/Investor/Connector) on landing page
- Signup with email/password → JWT auth
- Startup and investor dashboard shells
- Rust backend with auth, pitches, pools, investments, compliance routes
- Telegram bot (wallet_bot.py on KVM2) with /pitch, /investor, /find, /findinvestor, /intro, /approve, /reject
- IPFS profile storage via Pinata
- MTN token gating on Telegram (10k tMTN for Kevin access)
- **Founder Profile & Pitch Deck Upload** — `/startup/profile`, Pinata IPFS
- **Kevin Chat Widget** — floating chat on all dashboards, `POST /api/kevin/chat`
- **Weekly Matches Email Engine** — founder + investor weekly digest, Vercel cron, Resend, HMAC unsubscribe

### 🔨 To Build (priority order)
1. **Call Intelligence** — `/startup/calls` page, upload audio recordings (.m4a/.mp3/.wav), backend sends to Whisper API Docker container on KVM2 (port 9000) for transcription, then Claude for analysis (summary, key takeaways, action items, investor sentiment). Display in card layout. Both founder and investor get tailored insights if investor opts in
2. **Investor Deal Flow** — investor dashboard shows matched startups, each card: company name, stage, sector, one-liner, "View pitch" + "Request intro" buttons
3. **Wallet Connect** — Phantom/Solflare integration, MTN token balance check, alternative to email auth
4. **Pro Tier** — $9.99/month subscription via Stripe

### Pro Tier Features (gated behind subscription)
- Private deck storage on Pinata (free tier = public IPFS only)
- Full contact card shared on intros (free = deck link only)
- `startup_name.metatron.id` — custom subdomain with own AI agent
- Choice of AI backend (Claude, GPT-4, Gemini, etc.)
- Custom system prompt and knowledge base for their agent
- Embeddable widget on founder's own website
- Call Intelligence access

### Free vs Pro Contact Sharing
| | Free Founder | Pro Founder |
|---|---|---|
| Investor sees on intro | Deck link only | Full contact card (name, email, LinkedIn, website) |
| Profile on IPFS | Summary only | Full profile |
| Deck storage | Public IPFS | Private Pinata |
| Custom subdomain | No | `startup_name.metatron.id` |

## MTN Token (Solana)

- **Token**: MTN — native governance and utility token
- **Mint address**: `2tUS8sXb1U84sE2q2NbgmoUQ47s97LRNM58EJ5a1Rhed` (Solana mainnet)
- **Supply**: 1 billion MTN
- **Mint/freeze authority**: `DXGYnw8hUK9e4upyvDQPKhgQu7x6hD9DeWQV6GgbdJ29` (Nick's wallet, managed internally)
- **Functions**: Kevin access gating (minimum MTN balance), DAO governance voting
- **Tokenised ETF mechanic**: holders gain collective exposure to vetted startups; investors receive exposure through MTN, startups receive USDC
- **Devnet**: 10k tMTN required for Kevin access via Telegram

## KVM2 Server (Production — 31.97.189.18)

The production server runs:
- `wallet_bot.py` — Telegram bot (sole Telegram handler, conversational AI + notifications, calls NadirClaw for Kevin responses)
- `wallet_webhook.py` (port 8857) — wallet registration webhook
- `nadirclaw` (port 8856) — Kevin AI backend
- `openclaw` — web interface only (Telegram disabled)
- `ipfs.py` — Pinata IPFS helper module
- Whisper API Docker container (port 9000) — audio transcription
- nginx serving platform.metatron.id (port 8080, `/var/www/platform`)
- Key data files: `/root/.metatron_profiles.json`, `/root/.metatron_investors.json`, `/root/.metatron_intros.json`
- Env: `/root/.env` (contains `PINATA_JWT`, `ANTHROPIC_API_KEY`, etc.)

## IPFS / Data Architecture

Founder data is stored on IPFS linked to wallet and (future) NFT:
1. Founder completes onboarding → profile JSON + deck uploaded to Pinata IPFS → gets CID
2. (Phase 2) Mint metatron Founder NFT on Solana with metadata URI pointing to IPFS CID
3. Founders own their data in their wallet — decentralised, verifiable on-chain

## Crossmint Integration (ON HOLD)

Crossmint was evaluated for fiat-to-USDC checkout and embedded wallet creation. Pricing received:
- $1,000 onboarding fee (one-time)
- $1,899/month (includes 10k wallets)
- 6.9% + $0.30 per checkout transaction

**Decision: Too expensive for MVP.** Exploring alternatives (MoonPay, Transak, crypto-only launch with Phantom/Solflare) or negotiating a startup tier. Contact: Fonz at Crossmint.

## Conventions

- All colours use the design system tokens above — never use emerald/green/slate from old designs
- Keep the metatron.id logo as-is (no separate wordmark text next to it)
- Sidebar nav on dashboards with links: Dashboard, Profile, Pitches, Calls (founder) / Dashboard, Deal Flow, Watchlist (investor)
- Use DM Sans everywhere, JetBrains Mono for code/labels/mono elements
- Cards: `#16161f` bg, 12px radius, `rgba(255,255,255,0.06)` border
- Buttons: `#6c5ce7` primary, hover slightly lighter, 12px radius
- All API calls from frontend go through `NEXT_PUBLIC_API_BASE_URL`
- Backend uses sqlx migrations — add new tables via `backend/migrations/`
- Solana interactions use Helius RPC (devnet for testing, mainnet for production)

## White Paper

A metatron white paper rewrite (from V1.3) is pending, targeting institutional partners, crypto-native investors, traditional investors, and founders.

## Subscription Architecture

### Columns on `users` table
- `subscription_plan` — plan level: `'free'` | `'basic'` | `'pro'`
- `subscription_tier` — billing period: `'monthly'` | `'annual'` (NOT the plan level)
- `is_basic` — boolean, true when plan='basic' AND subscription active. Managed by `sync_is_basic` trigger.
- `is_pro` — boolean, true when plan='pro' AND subscription active. Managed by `sync_is_pro` trigger.
- **IMPORTANT:** `is_pro` is NOT true for basic subscribers. Basic subscribers have `is_basic=true`, `is_pro=false`.
- The `/subscriptions/status` endpoint returns `subscription_plan` aliased as `subscription_tier` so the frontend reads the plan level, not the billing period.

### Tier Feature Matrix (applies to ALL roles — founders, investors, connectors)

| Feature | Free | Basic | Pro |
|---|---|---|---|
| Kevin messages/day | 20 (shared all channels) | 200 (shared all channels) | Unlimited |
| Kevin matches/week | 1 | 10 | Unlimited (0 in env = unlimited) |
| Match cache refresh | 7 days | 6 hours | 6 hours |
| Pitch deck | External URL only | Permanent public IPFS | Permanent private IPFS |
| Deck re-uploads | 1 (14-day pin) | Unlimited | Unlimited |
| Call Intelligence | No | Yes | Yes |
| Angel Score detail | Summary only | Full breakdown | Full breakdown |

### Match Limits — Critical Rules
- Match limits apply to **ALL roles equally** — founders AND investors see the same limits
- Limits are configured via env vars: `FOUNDER_FREE_MATCH_LIMIT`, `FOUNDER_BASIC_MATCH_LIMIT`, `FOUNDER_PRO_MATCH_LIMIT`
- `FOUNDER_PRO_MATCH_LIMIT=0` means unlimited (maps to `i64::MAX` in Rust)
- Both `get_matches` and `generate_matches` handlers use `is_basic`/`is_pro` from `users` table for ALL roles (not role-specific queries)
- Cache interval: `is_basic || is_pro_user` → 6 hours; else → 7 days

### Kevin Daily Limits
- Single counter per `user_id + usage_date` in `kevin_daily_usage` table
- All channels (platform, email, Telegram, WhatsApp) draw from the same pool
- Limit function: `kevin_daily_limit(is_pro, subscription_tier)` in `backend/src/routes/kevin.rs`

### Pitch Deck Expiry
- Free decks: 14-day Pinata pin. Emails sent at day 7 and day 13. After day 14, `pitch_deck_expires_at` is past.
- **Do NOT unpin from Pinata** — IPFS is public network, unpinning only removes from our account. Just hide the URL.
- Expired deck URL is hidden from investors by a CASE expression in SQL (returns NULL if expired).
- This CASE is applied in: `deals.rs`, `connections.rs`, `kevin_matches.rs`
- Deduplication flags: `deck_7day_email_sent`, `deck_1day_email_sent`, `deck_expired_email_sent` on `pitches` table

### Payment Providers
- **Paystack** — fiat/ZAR recurring subscriptions
- **NowPayments** — crypto (USDC/USDT) payments
- Sphere Pay: dropped. Do not reference.
- When any subscription activates, set `subscription_plan = 'basic'` (Basic tier). Pro tier is future.

## CSS & Light Mode Rules

- **Always use CSS variables** for colours: `var(--text)`, `var(--bg-card)`, `var(--border)`, `var(--text-muted)`
- **Never hardcode hex values** in component styles — they break in light mode
- Tailwind `metatron-accent`, `metatron-accent-hover` tokens are dark-mode only — OK for interactive elements (buttons, badges) but not for text/backgrounds
- `grid-bg` class: **always `display: none`** — never re-enable
- `.orb` glow: 800px, `top: -20vh`, `rgba(108,92,231,0.12)`, fixed position — do not animate
- Landing page inline glow: `radial-gradient(ellipse 80% 40% at 50% calc(8% + 250px), rgba(108,92,231,0.22), transparent 60%)`

## Claude Workflow

- Claude writes and edits code directly (Cursor no longer used, dropped 2026-07-18 to save on costs)
- Always read the current file before editing — never rely on memory alone
- Always confirm the plan with Nick before taking any action
- Only change exactly what was asked — never add unrequested changes
- All shell commands must be single line (zsh executes line-by-line on paste)
- Always end responses with Mac push + KVM2 deploy commands
- Nick edits `.env` files directly with nano — never suggest echo/sed for env vars
- Never create `.env.example`
- Always build/test on Solana devnet; mainnet only at production deployment

---
> Source: [nickallison11/kevin_metatron](https://github.com/nickallison11/kevin_metatron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
