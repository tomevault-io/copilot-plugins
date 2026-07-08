## leadbroker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # local dev server
npm run build        # static export to out/
npm run lint         # ESLint
npm run deploy:hosting  # build + firebase deploy --only hosting
```

No test suite. No `npm test`.

## Architecture

**Next.js 14 static export** — `output: 'export'` in next.config.js. All pages render to static HTML in `out/`. No server-side rendering; no Next.js API routes. Deployed to Firebase Hosting.

**App Router** (`app/`) with global layout wrapping Header + Footer around all pages:
- `app/page.js` — homepage
- `app/tools/page.js` — EMI calculator + eligibility checker (client-side math only)
- `app/guidance/` — loan guides and FAQs
- `app/services/[slug]/` — dynamic service pages driven by `app/services/serviceData.js`
- `app/contact/` — lead capture form
- `app/components/` — Header, Footer, Logo (shared layout components)

**Lead capture flow:**
1. Contact form submits → `lib/leads.js:saveLead()` writes to Firestore `leads` collection with `status: 'new'`
2. `lib/leads.js:notifyLeadOnWhatsApp()` POSTs to Twilio webhook URL
3. Firebase Function `sendLeadWhatsApp` (in `functions/index.js`, deployed to `asia-south1`) receives the webhook, sends WhatsApp message via Twilio SDK, then updates the lead doc with `whatsappNotification` status

**Firebase:**
- `lib/firebase.js` — client-side Firestore init, reads from `NEXT_PUBLIC_FIREBASE_*` env vars
- `firestore.rules` — security rules
- `functions/` — separate Node.js package with Cloud Functions v2; Twilio secrets stored via Firebase Secret Manager (not env vars)
- `firebase.json` — hosting serves `out/`, static asset cache headers set to 1 year

## Environment Variables

Required in `.env.local` for local dev:
```
NEXT_PUBLIC_FIREBASE_API_KEY
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN
NEXT_PUBLIC_FIREBASE_PROJECT_ID
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID
NEXT_PUBLIC_FIREBASE_APP_ID
NEXT_PUBLIC_TWILIO_WEBHOOK_URL   # URL of the deployed sendLeadWhatsApp function
```

## Styling

No CSS framework. All styles are inline or in `app/globals.css` with utility classes (`.container`, `.card`, `.btn`, `.section`, `.section-dark`, `.hero`, etc.). Responsive breakpoints handled via CSS in globals.css and inline media queries. Brand colors: primary `#2563eb`, dark `#0f172a`, accent `#00d084`.
    
## Brand Rules

Never use: agent, broker, DSA, lead selling, marketplace.  
Use instead: loan assistance, loan offers, eligibility check, financial partners, loan comparison.

Primary CTA: "Check Eligibility". Secondary CTA: "Get Loan Offers".

## Project Spec

`plan.md` is the canonical YAML spec for the site — covers services, tools, validation rules, design tokens, navigation, SEO keywords, and content. Read it before adding or changing features.

---
> Source: [dilip90/LeadBroker](https://github.com/dilip90/LeadBroker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
