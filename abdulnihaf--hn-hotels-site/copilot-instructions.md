## hn-hotels-site

> <!-- Root constitution (the person, above every venture; HN Hotels is ONE node under it,

# HN Hotels — Repo Context for Claude

<!-- Root constitution (the person, above every venture; HN Hotels is ONE node under it,
     alongside Visa Medicals, Wealth, Personal). Byte-identical mirror of the Mac canonical
     ~/.ai-coordination/NIHAF-BRAIN.md, committed here so iPhone Claude Code (cloud sandbox,
     which can only read committed repo files) inherits the same root. When the canonical
     changes, re-copy it here. -->
@NIHAF-BRAIN.md

This file auto-loads in every Claude Code session opened in this repo. It exists
because the owner (Nihaf) does marketing-strategy thinking on iPhone Claude Code,
and the iPhone sandbox cannot reach our private domains. Live business data must
travel into the conversation via files committed here.

## Identity

- **Legal entity:** HN Hotels Private Limited
- **CIN:** U55101KA2023PTC182051
- **PAN:** AAHCH1024M · **TAN:** BLRH15862A · **UDYAM:** UDYAM-KR-03-0606827
- **Incorporated:** 11 Dec 2023
- **Registered office:** #22, 3rd Floor, H.K.P. Road, Shivajinagar, Bangalore 560051
- **Brands:** Hamza Express (HE — QSR biryani/kabab) · Nawabi Chai House (NCH — Irani chai cafe)

## Outlets

- **Hamza Express** — #19, H.K.P. Road, Shivajinagar, Bangalore 560051. QSR. ~176-SKU menu.
- **Nawabi Chai House** — same locality, Shivajinagar. Cafe format, concentrated menu.
- **Hours:** ~7am–11pm IST, both outlets.
- **Neighborhood:** Bangalore Muslim quarter. Dakhni-food belt. Walking distance to MG Rd, Commercial St, Brigade Rd, Shivajinagar bus stand. Foot-traffic from office crowd by day, night-eaters/students after 9pm.

## Heritage positioning

Hamza family in Bangalore food trade since **1918** — four generations. Dakhni cuisine legacy. Public-facing references: [nawabichaihouse.com](https://nawabichaihouse.com), JustDial. Use this in any brand/marketing/PR copy — heritage is the moat against the Andhra-biryani belt next door.

## Menu fingerprint (verified Feb–Mar 2026, [investor.html](investor.html))

**Hamza Express — 176 SKUs.** Heroes (top 5 by revenue):
1. Ghee Rice — 9.7%
2. Chicken Kabab — 8.6%
3. Tandoori Chicken — 5.3%
4. Mutton Biryani — 5.1%
5. Chicken Biryani — 5.1%
Plus Mutton Brain Dry, Hamza Special, Mughlai Chicken, breads, drinks.
Avg ticket ~₹464.

**Nawabi Chai House — concentrated.** Two items = 77% of revenue:
- Irani Chai — **58.7%** (53,169 cups in 57 days)
- Haleem (all sizes) — **18%**
Bun Maska, Osmania biscuit packs, Nawabi Coffee, Malai Bun, chicken cutlet round it out.
Avg ticket ~₹349.

## Competitor set (Shivajinagar)

- **Empire** — chain, mass-market biryani. Not Dakhni.
- **Valima Ki Biryani** — local QSR.
- **Andhra-style biryani belt** — multiple outlets within 500m.
- HE differentiates on Dakhni/Hyderabad style + 1918 heritage. NCH owns the Irani-chai-and-snacks evening crowd; no real direct competitor in 1km radius.

## Channel economics summary (verified Apr 2026)

- **Dine-in:** 0% commission. Best margin.
- **WABA Direct (Wati):** ~2.36% gateway only.
- **Zomato Dining (HE):** ~8.26% effective.
- **Swiggy Delivery:** ~24.60% effective.
- **Zomato Delivery (<4km):** ~23.41% effective.
- **EazyDiner:** ₹30/cover + 5% + 1.8% gateway.

Detailed per-clause contract math lives in `docs/CONTRACT_REGISTRY.md` and the runtime [sales-finance.html](sales-finance.html).

## Aggregator contracts (signed)

- **Zomato Delivery** (HE + NCH): 02 Mar 2026
- **Swiggy** (HE + NCH): 02 Mar 2026
- **EazyDiner** (HE only): 28 Mar 2026
- **Zomato Dining** (HE): valid till 2046

## Truth-source URL register

Read these for live state. **Most are unreachable from iPhone sandbox** — see "Sandbox reachability" below.

| URL | What it holds | How to read |
|---|---|---|
| `hnhotels.in/ops/aggregator/` | Live Swiggy + Zomato orders, ratings, finance | API `/api/aggregator-pulse?action=orders\|finance\|health\|snapshots\|reviews` (key: `DASHBOARD_API_KEY`) |
| `hamzaexpress.in/ops/google-cockpit/` | Google Ads campaign state (HE) | API `/api/google-cockpit?period=today\|7d\|30d\|all` (open CORS, Worker has Google secrets) |
| `hamzaexpress.in/ops/ctwa-cockpit/` | Meta CTWA campaigns + funnel (HE) | API `/api/ctwa-analytics?period=7d\|30d\|all` (open CORS, Worker has Meta secrets) |
| `hamzaexpress.in/ops/leads/` | WABA leads CRM (HE) | API `/api/leads?action=counts\|history\|segments` (open CORS, D1-backed) |
| `app.hnhotels.in` | Operational app shell | Various; PIN-gated |
| `odoo.hnhotels.in` | Unified finance Odoo (expenses, POs, vendor bills, payroll) | JSON-RPC at `/jsonrpc`, db `main`, uid 2, env `ODOO_API_KEY`. Companies: 1=HQ, 2=HE, 3=NCH |
| `test.hamzahotel.com` | **HE Production** Odoo (POS source of truth for HE) | JSON-RPC same shape. Confusingly named "test" — it is HE prod |
| `ops.hamzahotel.com` | **NCH Production** Odoo (POS source of truth for NCH) | JSON-RPC same shape. NCH company_id = 10 |

POS sales data only lives on `test.hamzahotel.com` (HE) and `ops.hamzahotel.com` (NCH). Finance/expense/PO data lives on `odoo.hnhotels.in`. Don't mix them up.

## Standing marketing direction (verbatim from owner)

> **De-prioritize WABA as order destination — market unfamiliar.**
> Push: **Meta Ads, Influencer Marketing, Google Ads, Swiggy/Zomato organic + inorganic.**

WABA stays for retention/CRM, not for first-order acquisition.

## Current targets

- **May 2026** — HE: ₹15,00,000 · NCH: ₹12,00,000

For context, Mar 2026 actuals were HE ₹6,59,903 and NCH ₹10,80,731. May targets are 2.3× HE / 1.1× NCH.

## Sandbox reachability map (iPhone Claude Code)

The iPhone Claude Code sandbox has an egress allowlist. **The following are blocked**, so do not waste turns trying to fetch them from iPhone:

- `hnhotels.in`, `app.hnhotels.in`
- `hamzaexpress.in`, `nawabichaihouse.com`
- `odoo.hnhotels.in`, `test.hamzahotel.com`, `ops.hamzahotel.com`
- `graph.facebook.com`, `googleads.googleapis.com`

To bring data into iPhone sessions, the laptop runs `node scripts/snapshot-context.js` and commits the JSON outputs to `data/snapshots/`. Read those files instead.

## Where to read fresh data inside this repo

- `data/snapshots/aggregator-latest.json` — Swiggy + Zomato orders/ratings/finance
- `data/snapshots/sales-daily-last60d.json` — POS daily totals per brand per channel
- `data/snapshots/google-ads-latest.json` — Google Ads cockpit dump
- `data/snapshots/meta-ctwa-latest.json` — Meta CTWA cockpit dump
- `data/snapshots/waba-leads-latest.json` — WABA leads counts + recent
- `data/snapshots/snapshot-meta.json` — when/where snapshots were captured

Snapshots are **manual-refresh**. Owner runs the script when needed. If a snapshot is stale (>3 days), say so explicitly in any analysis rather than treating it as live.

## Secret handling — non-negotiable

- **NEVER commit API keys, OAuth tokens, refresh tokens, webhook secrets, or xlsx contents.**
- Local tokens go in `.env.local` (gitignored — see `.gitignore`).
- Production tokens are Cloudflare Worker secrets (`wrangler secret put …`), not files.
- If you discover a secret accidentally committed: rotate first, then strip from history.
- Reference env-var **names** in scripts; never hardcode values.

## Working with `hn-winpc` (the always-on Windows appliance) — REQUIRED READING

This appliance is **shared by multiple concurrent Claude Code chats**. As of 2026-05-10
three automations are co-resident on it:

1. **`aggregator-pulse`** — Swiggy + Zomato Online Ordering scraping (Chrome default profile, fragile)
2. **`modash-driver`** — Modash influencer pipeline (isolated Chrome profiles per Modash account)
3. **`dine-aggregator`** — Zomato Partner Dining-Out audit (isolated Chrome profile)

**If your task touches `hn-winpc` in any way** (SSH, file deploy, Chrome automation,
Scheduled Task, registry change), you MUST:

1. Read [`fleet/MULTI-TENANT-WINPC.md`](fleet/MULTI-TENANT-WINPC.md) — the runbook
2. Read the live manifest before mutating: `ssh "HN Hotels@hn-winpc" 'type "C:\hn-control\manifest.json"'`
3. Respect every automation's `do_not_disturb` list
4. Never run `taskkill /IM chrome.exe /F` or `Get-Process chrome | Stop-Process` —
   that kills the aggregator. Always filter by user-data-dir cmdline match.
5. Namespace everything you create — files in `C:\hn-control\<purpose>\`,
   Scheduled Tasks named `HN-<Purpose>-<Job>`

**For a new automation** (4th, 5th, ...): follow the step-by-step in
[`fleet/NEW-OPERATION-TEMPLATE.md`](fleet/NEW-OPERATION-TEMPLATE.md).

**SSH command pattern:** `ssh "HN Hotels@hn-winpc" 'cmd'` (quoted, space in username).
On a laptop where Tailscale is up but routing fails with "Can't assign requested
address", add `-o BindAddress=$(ifconfig | grep "inet 100\." | awk '{print $2}' | head -1)`.

**If Tailscale is down on the laptop**, no chat can reach hn-winpc — owner reconnects
via menu bar before any work proceeds. Tailscale on the PC itself is `set unattended`
and never needs reconnect from Claude.

## Repo execution rules (binding for all sessions)

See `docs/EXECUTION-CHARTER.md` for full charter. Key rules:

- **No build pipeline.** Vanilla HTML/JS/CSS served from Cloudflare Pages. No npm/Vite/Rollup/esbuild.
- **Money in paise (INTEGER).** Convert to rupees only at the display layer.
- **Read before edit.** Read the full file before changing it.
- **Claude executes merges + deploys — never hand them to Nihaf.** He is a non-technical business owner; asking him to merge a PR, run a command, add a domain, or configure a dashboard is a failure of the division of labour (he was explicit about this, 2026-06-15). Workflow: open the PR, **verify the change is safe + isolated (touches no other live surface), then merge and deploy it yourself** (gh CLI, Cloudflare API, wrangler). Report the live result. Only involve Nihaf for what *only he can physically do*: test on his phone, give ground truth, or approve moving money / an outward-facing send. See memory `execute-never-delegate-to-nihaf`.
- **One branch + one PR per phase.** No bundling. Keep each change isolated so it's safe to merge on its own.
- **Mobile-first.** ≥44px tap targets. Test Safari iOS + Chrome Android.

---
> Source: [abdulnihaf/HN-Hotels-Site](https://github.com/abdulnihaf/HN-Hotels-Site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
