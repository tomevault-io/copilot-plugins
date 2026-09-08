## job-finder-sales-funnel

> > Antigravity IDE only. Applies to every backend task opened in this workspace.

# SnipeJob Sales Funnel — Backend Ops Rules
> Antigravity IDE only. Applies to every backend task opened in this workspace.
> This funnel has **no backend of its own** — it calls the same Cloudflare Worker
> as the SaaS app. For all MCP server config and full backend rules, see:
> - `JOB FINDER SAAS V3/AGENTS.md` — primary backend ops rules
> - `~/.gemini/GEMINI.md → ## Backend Ops Security Rules` — global security rules
> - `~/.gemini/config/mcp_config.json` — MCP server definitions

---

## Project Identity

| Field | Value |
|---|---|
| **Product** | SnipeJob Sales Funnel |
| **Workspace** | JOB FINDER SALES FUNNEL |
| **Frontend** | `snipe_jobs_sales_funnel.html` — static marketing + checkout-entry page |
| **Hosting** | GitHub Pages or Cloudflare Pages (separate repo from SaaS app) |
| **Backend** | **None** — calls Cloudflare Worker `my-sniper-worker` directly (CORS open `*`) |
| **Worker URL** | `https://my-sniper-worker.daniellancce1.workers.dev/api` |
| **App URL** | `https://626gl1ch.github.io/JOB-FINDER-SAAS-WEB-APP/` |
| **Supabase** | Shared project ref `mdmpcxtjwnovbhidwwhj` — managed from SaaS workspace |
| **Paystack mode** | Test-mode default; live-mode requires explicit confirmation per R8 |

---

## What This Funnel Does (Backend-Relevant Only)

The funnel page has **one backend interaction pattern**:

```
Visitor clicks CTA (annual or monthly)
  └─ POST /api/payment/create-checkout-public   ← no auth token, no Supabase session
       └─ Worker creates Paystack checkout transaction/subscription, returns URL
            └─ Browser → Paystack Checkout
                 └─ Paystack redirects to app:
                    .../index.html?premium_signup=1&reference=...
                         └─ App verifies transaction/reference → creates account → claims premium
```

**Funnel-specific Worker routes used:**
| Route | Method | Notes |
|---|---|---|
| `/api/payment/create-checkout-public` | POST | No auth required — funnel entry point |
| `/api/payment/verify-session` | GET | Used by the app after redirect (verifies transaction reference) |

**Funnel JS config block (near bottom of `snipe_jobs_sales_funnel.html`):**
```js
var APP_URL        = 'https://626gl1ch.github.io/JOB-FINDER-SAAS-WEB-APP/';
var WORKER_API_URL = 'https://my-sniper-worker.daniellancce1.workers.dev/api';
```
Changing the Worker URL here is the only funnel-side config change needed if the Worker moves.

---

## A.0 — How Antigravity Should Use This File

1. **This workspace has no Supabase or Paystack configuration of its own.** If a task touches the database, Worker secrets, or Paystack directly, switch to (or note that you're acting on) the `JOB FINDER SAAS V3` workspace's MCP connections.
2. **MCP servers to use for this workspace** (same as SaaS workspace — shared infrastructure):
   - `supabase-snipejob` (read-only) — if inspecting DB state related to funnel signups
   - `paystack-snipejob` — if checking transaction status or plan codes in test mode
   - `paystack-snipejob-live` — **only with explicit "yes, live and I mean it" confirmation**
   - `cloudflare-snipejob` — if tailing Worker logs for funnel-triggered requests
3. **Pre/post-flight rules (A.5 in the SaaS AGENTS.md) apply here equally.** State your MCP scope and confirm test vs. live before any Paystack or Worker action.
4. **Obey Security Rules R1–R12** in `GEMINI.md` regardless of what's asked.

---

## Funnel-Specific Checks

### Before testing the funnel end-to-end
- Confirm `PAYSTACK_PRO_ANNUAL_PLAN_CODE` is set in the Worker (check `/debug/env` → `hasPaystackProAnnualPlanCode: true`).
- Confirm `PAYSTACK_PRO_PLAN_CODE` is also set (monthly plan).
- CORS is open `*` on the Worker — no funnel-domain-specific config needed.
- The funnel does **not** duplicate checkout logic — it calls the Worker's public endpoint which is already hardened.

### Paystack test-mode loop (run before any paid traffic)
1. Annual plan: click CTA → Paystack Checkout shows correct price → test card succeeds → `current_tier`/`plan_type`/`signup_source` update correctly in Supabase → webhook returns 200.
2. Monthly plan: same flow, confirms monthly subscription creation.
3. Handle failure modes cleanly.
4. Re-submitted reference is rejected as already-claimed.
5. Cancellation flips `current_tier` back to `'free'`.

### Going live checklist (do not skip)
- [ ] Privacy Policy / Terms of Service / Refund Policy pages exist and are linked in the footer (currently dead `<a>` tags with no `href` — **hard blocker for Paystack live-mode activation**).
- [ ] Paystack account fully activated, out of test mode.
- [ ] Live-mode plan codes re-created in Paystack and set as Worker secrets (`PAYSTACK_PRO_PLAN_CODE`, `PAYSTACK_PRO_ANNUAL_PLAN_CODE`).
- [ ] Live webhook signing secret rotated (`PAYSTACK_WEBHOOK_SECRET`).
- [ ] Two real self-purchases done and refunded to verify live flow.
- [ ] Funnel countdown deadline is a real date being honored.
- [ ] UptimeRobot (or equivalent) monitors on funnel URL + Worker `/debug/env`.

---

## Open Items (Funnel-Specific)

- [ ] Confirm the funnel's footer legal links have real `href` values pointing to `privacy.html`, `terms.html`, `refunds.html` — these files exist in the SaaS repo but are not linked in the funnel page.
- [ ] Decide on final funnel hosting URL and update `APP_URL` / `WORKER_API_URL` constants if using a custom domain.
- [ ] Marketing Playbook (`SnipeJob_Marketing_Playbook.html`) needs a copy pass before any posts go live — see SNIPEJOB_MASTER_DOCUMENTATION.md §6 for specific corrections needed.

---
> Source: [626gl1ch/JOB-FINDER-SALES-FUNNEL](https://github.com/626gl1ch/JOB-FINDER-SALES-FUNNEL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
