## c-r-i-s-p

> Master context for every Claude Code session. Read this before touching any code.

# Night Errand Runner — CLAUDE.md
Master context for every Claude Code session. Read this before touching any code.

---

## What This App Is

A night errand delivery web app for Novi Beograd, Belgrade, Serbia. A solo driver runs errands after midnight — pharmacy items, food, drinks, forgotten things — anything from anywhere that's open. Customers order via a mobile-first web app. Driver receives push, accepts, delivers. Customers get real-time status updates so they never have to call.

**Brand positioning:** "Komšija koji ti pomaže" — the neighbour who helps. Warm, personal, human. Not a Glovo clone.

---

## Problem Being Solved

> "Currently, a solo night errand operator in Novi Beograd is losing time and delivery capacity to unstructured order intake across WhatsApp, Viber, and phone — because there is no single place where customers place complete, structured orders and the driver manages them in one view."

Full discovery docs in `/docs/`.

---

## Tech Stack — Pinned Versions (do not upgrade without explicit instruction)

| Layer | Tool | Version |
|---|---|---|
| Framework | Next.js | 16.2.3 |
| Language | TypeScript | 5.7.x |
| UI Runtime | React | 19.x |
| Styling | Tailwind CSS | 4.2.2 |
| PWA | @serwist/next | 9.5.7 |
| DB + Auth + Realtime | @supabase/supabase-js | 2.103.2 |
| Supabase SSR | @supabase/ssr | latest compatible with 2.x |
| Push notifications | firebase | 12.12.0 |
| Maps | @vis.gl/react-google-maps | 1.x |
| State management | zustand | 5.0.12 |
| Hosting | Railway | — |

**Version rules (mandatory):**
- Use only the versions listed above. Do not upgrade silently.
- If a library's documented API differs from what you know — trust the pinned version, not training data.
- If a version conflict arises, stop and flag it. Do not resolve silently.
- Tailwind v4 is a complete rewrite from v3. CSS-first config. Do not follow v3 patterns.
- React 19: `createRoot` only. No `ReactDOM.render`.
- Supabase JS v2: `createBrowserClient` / `createServerClient` from `@supabase/ssr` for Next.js App Router.

---

## Language

**All UI copy, push notifications, error messages, button labels, and placeholder text must be in Serbian.** No English strings visible to users in production. Developer code (variable names, comments, API routes) stays in English.

---

## API Keys & Security (non-negotiable)

- **Never expose API keys, tokens, or secrets on the client side.**
- All calls to 3rd party APIs with credentials go through Next.js API routes (server-side).
- `SUPABASE_SERVICE_ROLE_KEY` — server-side only. Never in any `NEXT_PUBLIC_` variable.
- `FIREBASE_SERVICE_ACCOUNT` — server-side only. Never in any `NEXT_PUBLIC_` variable.
- `GOOGLE_MAPS_API_KEY` — server-side only. A separate restricted key (`NEXT_PUBLIC_GOOGLE_MAPS_PUBLIC_KEY`) is used for the client-side map display only — restricted to Maps JS API + HTTP referrer.
- Every secret goes in `.env.local` — verified in `.gitignore` before first commit.
- Before any deploy: `grep -r "SERVICE_ROLE\|SERVICE_ACCOUNT" .next/` must return nothing.

---

## Non-Functional Requirements

| NFR | Requirement |
|---|---|
| Availability | Occasional downtime tolerable at MVP. Railway free tier. |
| Performance | LCP <2s mobile 4G. Form submit response <1s. Push delivery <5s. |
| Security | HTTPS everywhere. Supabase RLS on all tables. Driver session expiry: 7 days. No PII in logs. |
| Data residency | EU region. Supabase Frankfurt. Railway EU region. ZZPL (GDPR-equivalent) compliant. |
| Deployment | Railway Next.js Node.js runtime. No Docker at MVP. |
| Monitoring | Reactive. Railway log viewer. |
| Push | FCM best-effort. Pull-to-refresh fallback on customer status screen. |
| Privacy | No customer addresses or order contents in logs. No data shared between parties. `/privacy` page required. |

---

## Logging

See `docs/logging-spec.md` for full spec.

**Key rules:**
- Log `orderId`, never order contents (item_description, address, etc. are PII-adjacent)
- Never log: passwords, FCM tokens, IBAN, customer location, item descriptions
- Format: structured JSON with `timestamp`, `level`, `event`, `requestId`, `orderId`
- Production: INFO minimum. No `console.log` — use structured logger only.
- Destination: Railway stdout → Railway log viewer

---

## Human-in-the-Loop Zones

**Item substitution is always a human decision.** The driver calls the customer when an item is unavailable at the store. The app never automates this. No substitution logic, no suggestion engine, no automated alternatives.

---

## Database Schema

See `docs/ai-spec-supabase.md` for full schema, RLS policies, and Supabase client setup.

**Tables:** `orders`, `driver_profiles`
**Auth:** Supabase Auth — driver only (email + password). Customers have no accounts.
**Realtime:** Enabled on `orders` table for customer status screen subscription.

---

## Navigation Structure

**Customer app:** Stack-only. No nav bar.
- `/` — Order form
- `/order/[id]` — Status screen
- `/order/[id]/declined` — Declined screen
- `/privacy` — Privacy policy

**Driver app:** 2-tab bar (Novo/Aktivno | Istorija)
- `/driver` — Login (redirect to `/driver/orders` if authenticated)
- `/driver/orders` — Orders tab
- `/driver/orders/[id]` — Order detail (accept/decline)
- `/driver/orders/[id]/active` — Active order (status controls, store suggestions, IPS QR)
- `/driver/history` — History tab (Post-MVP content)

---

## Sprint Sequence

| Sprint | Goal | AI Spec |
|---|---|---|
| 1 | Foundation + Customer Order Form | `docs/ai-spec-sprint1.md` |
| 2 | Driver Dashboard + Accept/Decline + Push Notifications | `docs/ai-spec-sprint2.md` |
| 3 | Active Order + Store Suggestions + IPS QR | `docs/ai-spec-sprint3.md` |
| 4 | Polish + PWA + Railway Deploy + Launch | `docs/ai-spec-sprint4.md` |

**Rule:** Do not start Sprint N+1 before Sprint N is working end-to-end.

---

## Integration Specs

| Integration | Spec file |
|---|---|
| Supabase | `docs/ai-spec-supabase.md` |
| Firebase FCM | `docs/ai-spec-firebase-fcm.md` |
| Google Maps | `docs/ai-spec-google-maps.md` |
| NBS IPS QR | `docs/ai-spec-nbs-ips-qr.md` |

---

## Analytics

See `docs/analytics-spec.md`. GA4 via `@next/third-parties/google`. No PII in event parameters.
Environment variable: `NEXT_PUBLIC_GA_MEASUREMENT_ID`

---

## Quality Gates (every sprint)

- Bearer security scan — no Critical/High findings before merge
- No secrets in client bundle (verify with grep before deploy)
- Supabase RLS tested — customer cannot read another customer's order
- Push notification tested on real Android device
- All UI copy in Serbian — no English strings in production
- Order state changes logged per `docs/logging-spec.md`
- No `console.log` in production

---

## Environment Variables Master List

```
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=          ← server-side only

# Google Maps
NEXT_PUBLIC_GOOGLE_MAPS_PUBLIC_KEY= ← restricted: Maps JS API + HTTP referrer only
GOOGLE_MAPS_API_KEY=                ← server-side only: Geocoding + Places

# Firebase
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=
NEXT_PUBLIC_FIREBASE_VAPID_KEY=
FIREBASE_SERVICE_ACCOUNT=           ← server-side only (JSON string)

# Analytics
NEXT_PUBLIC_GA_MEASUREMENT_ID=
```

All go in `.env.local` (not committed) AND Railway environment variables.

---

## CRISP Output Manifest

All discovery and spec documents live in `docs/`:

| File | Phase | Contents |
|---|---|---|
| `problem-statement.md` | C | Problem, constraints, NFRs, Go/No-Go |
| `buy-vs-build-matrix.md` | C | Tool decisions |
| `market-research.md` | C | TAM, competitors, USP |
| `swot.md` | C | Strengths, weaknesses, opportunities, threats |
| `value-proposition-canvas.md` | C | USP and positioning |
| `decisions.md` | C+R+I+S | All key decisions |
| `stakeholder-register.md` | R | Stakeholders, HITL zones |
| `success-metrics.md` | R | Baseline + targets |
| `process-flow.md` | I | AS-IS and TO-BE flows |
| `user-journey-map.md` | I | Customer + Driver journeys |
| `project-goals.md` | I | Goals, non-goals, success criteria |
| `ux-discovery.md` | I | Mental models, visual direction, navigation, screens |
| `design-system.md` | S | Colors, typography, components |
| `ux-spec.md` | S | Sitemap, flow specs, screen specs |
| `initial-backlog.md` | S | All stories with MVP tags |
| `assumptions-log.md` | S | Assumptions and risks |
| `risk-assessment.md` | S | Risk register |
| `mvp-prioritization.md` | S | HVLE scoring, MVP line |
| `logging-spec.md` | S | Logging rules |
| `analytics-spec.md` | S | GA4 events |
| `landing-page-brief.md` | S | Marketing copy |
| `sprint-plan.md` | S | Sprint sequence and goals |
| `ai-spec-sprint1.md` | S | Sprint 1 AI spec |
| `ai-spec-sprint2.md` | S | Sprint 2 AI spec |
| `ai-spec-sprint3.md` | S | Sprint 3 AI spec |
| `ai-spec-sprint4.md` | S | Sprint 4 AI spec |
| `ai-spec-supabase.md` | S | Supabase integration spec |
| `ai-spec-firebase-fcm.md` | S | Firebase FCM integration spec |
| `ai-spec-google-maps.md` | S | Google Maps integration spec |
| `ai-spec-nbs-ips-qr.md` | S | NBS IPS QR integration spec |

---
> Source: [radekamirko/C.R.I.S.P](https://github.com/radekamirko/C.R.I.S.P) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
