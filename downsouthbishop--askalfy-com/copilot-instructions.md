## askalfy-com

> > **You're the finishing engineer.** Every outside app — Google (Gmail/Calendar/Drive/

# 🚀 Finishing Alfy — START HERE

> **You're the finishing engineer.** Every outside app — Google (Gmail/Calendar/Drive/
> Docs/Sheets, bundled as Composio's 'googlesuper' toolkit), Slack, Notion, GitHub, Outlook,
> Linear, Trello, Asana, HubSpot, Discord, Zoom — connects through Composio
> (`supabase/functions/_shared/composio.ts`), triggered by texting "connect <app>" (the
> `connect_app` agent tool), never a dashboard button. Standing instructions + a cron
> automation runner and Stripe billing round out the backend (ported from PrymalAI-dashboard
> — see `docs/prymal-port-reference.md` for the full spec and `docs/alfy-handoff.md` for the
> VERIFY list; both predate the Composio-for-everything move, so treat their Google-OAuth-
> specific steps as historical). A fresh Supabase project (`askalfy`) is already provisioned
> and migrated. What's left is Composio, Twilio, Anthropic, Stripe, and deploy.

### What's already done (don't rebuild it)
Marketing site, the 3-tab dashboard (Today / Handled / Alfy knows), phone-OTP login, the
`/a` magic-link handler, the DB schema (`supabase/migrations/0001`-`0007_*.sql`, applied to
the live `askalfy` Supabase project), and the edge functions (`supabase/functions/alfy-*`,
sharing logic via `supabase/functions/_shared/`). It runs on demo data right now with zero
setup.

### Step 1 — Get it running locally (2 min)
```bash
npm install
npm run dev          # open the printed URL → /app shows the dashboard on demo data
```

### Step 2 — Make the remaining accounts (the only thing code can't do)
1. **Supabase** — done: the `askalfy` project exists and is migrated. Copy its URL + anon key.
2. **Telnyx** — done: number `+1-561-813-7525` is provisioned. Still need an API v2 key, the
   account's public key (for webhook signature verification), and A2P 10DLC campaign
   registration for the messaging profile.
3. **Composio** — an API key, plus one auth config per toolkit you want live (Google via
   'googlesuper', Slack, Notion, GitHub, Outlook, ...). Each auth config needs its ID set as
   `COMPOSIO_AUTHCFG_<TOOLKIT>`. See the sovereignty note in `_shared/composio.ts` — this
   stays off until the founder's cost/trust review is signed off.
4. **Anthropic** — one API key.
5. **Stripe** — an account, two Products with recurring Prices (Alfy / Alfy Plus), and a
   webhook endpoint registered against `alfy-stripe-webhook`'s deployed URL. See
   `docs/alfy-handoff.md`'s Phase 5 section for the exact events to listen for.

### Step 3 — Wire it up
```bash
cp .env.local.example .env.local        # fill PUBLIC_SUPABASE_URL + PUBLIC_SUPABASE_ANON_KEY
supabase link --project-ref kpybomnunyhazkenyoeb
# set the backend secrets (names listed in .env.local.example):
supabase secrets set SUPABASE_URL=... SUPABASE_ANON_KEY=... SUPABASE_SERVICE_ROLE_KEY=... \
  ANTHROPIC_API_KEY=... \
  COMPOSIO_API_KEY=... COMPOSIO_TOOLKITS=googlesuper,slack,notion,github,outlook,... \
  COMPOSIO_AUTHCFG_GOOGLESUPER=... COMPOSIO_AUTHCFG_SLACK=... # one per toolkit in COMPOSIO_TOOLKITS \
  TELNYX_API_KEY=... TELNYX_PHONE_NUMBER=+15618137525 TELNYX_PUBLIC_KEY=... \
  SEND_SMS_HOOK_SECRET=...       # from Auth -> Hooks -> Send SMS, see .env.local.example \
  INTERNAL_FUNCTION_SECRET=...   # must match the secret baked into the live cron.schedule() call, see docs/alfy-handoff.md \
  STRIPE_SECRET_KEY=... STRIPE_WEBHOOK_SECRET=... STRIPE_PRICE_ALFY=... STRIPE_PRICE_ALFY_PLUS=...
supabase functions deploy alfy-agent alfy-sms-inbound alfy-link alfy-approve alfy-composio-connect alfy-automation-runner alfy-digest alfy-stripe-checkout alfy-stripe-webhook alfy-send-sms-hook
```
Then:
- **Supabase → Auth → Hooks → Send SMS** → point at the deployed `alfy-send-sms-hook` URL
  (Telnyx isn't one of Supabase Auth's built-in SMS providers, so phone-login OTPs route
  through this hook instead of a dashboard dropdown).
- **Telnyx** → Mission Control Portal → the number's messaging settings → point the inbound
  webhook at the `alfy-sms-inbound` function URL.
- `src/lib/config.ts`'s `ALFY_PHONE` is already set to the real number.
- Deploy the site (Vercel/Netlify) with the `PUBLIC_` env vars.

### Step 4 — Verify (see `docs/alfy-handoff.md` for the exact calls to confirm)
Composio connect flow (text "connect slack" → get a link → tap it → connected), a connected
app's read tools returning instantly and its write tools queuing for approval, Telnyx
signature check, Telnyx send, phone-login OTP arriving via the Send SMS hook.

### Step 5 — Smoke test
Text the number → get a reply + an `Approve:` link → tap it → tap Approve → the action fires
→ a confirmation text arrives. That's the whole loop.

### The one rule you cannot break
**Everything below this line is the design constitution. Match it exactly** — the three
sections, the palette, Fraunces/Inter, the voice, and fern = the approval/trust colour.
Any screen you touch must still look like the rest. Do not add a fourth dashboard tab.

---

# Alfy Design Constitution
Brand: Alfy (askalfy.com). A warm, comfortably competent AI assistant anyone can
text. Tagline: "Just text Alfy." Trust narrative: "Alfy asks first."
Never call it "AI-powered" in copy — it is "an assistant."

## Palette (hardcode these, never substitute)
- Linen (canvas): #FAF5EC | Card surface: #FFFDF8 | Hairline border: #E7DFD0
- Espresso (text): #2E2A24 | Secondary text: #5C554A | Muted: #8A7F6E
- Marigold (primary action ONLY): #E08A2E, text on it #FFF8ED
- Fern (reserved EXCLUSIVELY for approval/trust moments): #4E7D68, tint #EDF3EF
Never introduce purple, neon, gradients, or glassmorphism. Warm neutrals only.

## Type
- Fraunces (weights 500/600) for headlines only — serif, warm, editorial
- Inter (400/500) for everything else: UI, body, buttons, Alfy's speech
- Self-hosted via Fontsource. No layout shift on font load.

## Voice (all UI copy and demo content)
Plain words, contractions, no exclamation marks, no emoji, sentence case.
Alfy reports what it did, then asks. Max 5 lines per Alfy message. Sign-off "— A".
Never "As an AI...". Never guilt, urgency, or hype words (unlock, seamless, supercharge).

## Design laws
1. The product is a phone number — the site's only CTA is "Text Alfy" (+ QR on desktop).
2. Nothing leaves without a yes — approval UI is the hero motif; fern = granted trust.
3. Grandmother-comprehensible: if a screen needs explaining, redesign it.
4. One signature animation per section max. Craft > quantity.
5. Every section must survive this test: would it look at home on an Awwwards site
   while remaining legible to a 65-year-old on a mid-range Android phone?

## Iteration protocol (mandatory)
After building or changing any section: render with Playwright, screenshot,
critique against /references images (spacing rhythm, type contrast, warmth,
hierarchy), list 5 specific deficits, fix, re-screenshot. Minimum 3 rounds
per section before calling it done.

---
> Source: [DownSouthBishop/AskAlfy.com](https://github.com/DownSouthBishop/AskAlfy.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
