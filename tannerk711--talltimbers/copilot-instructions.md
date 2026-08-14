## talltimbers

> **The new Tall Timbers DSCR funnel, target domain `dscr.talltimbersrfs.com`.** Built

# CLAUDE.md. Tall Timbers DSCR Funnel (v2 site)

**The new Tall Timbers DSCR funnel, target domain `dscr.talltimbersrfs.com`.** Built
2026-07-21 by cloning `clients/dscr-funnel-template/` (funnel v1, "private credit house"
design) and rebranding it for Tall Timbers. The old Astro LP project lives in
`clients/tall-timbers/` and stays in the tree because skills and history reference it.

**Domain reality (discovered 2026-07-22):** the apex talltimbersrfs.com is Adam's
AgentFire-hosted realty site behind Cloudflare. It was NEVER served by our Vercel project,
and nothing we deploy touches it. Our Vercel project's public production URL is
**talltimbers.vercel.app** (deployment-protection SSO covers the hash/branch URLs; the
stable alias is public), which serves the funnel now. The dscr subdomain goes live once a
Cloudflare CNAME (`dscr` -> `cname.vercel-dns.com`, DNS-only/grey cloud) exists and the
domain is added to the Vercel project. Traffic switches when Google Ads final URLs point at
the subdomain (CUTOVER.md Phase E).

## Client facts (locked)

- **Brand:** Tall Timbers Realty and Financial Services (short: Tall Timbers)
- **Domain:** talltimbersrfs.com
- **Specialist:** Adam C. Cunningham, Mortgage Loan Originator, NMLS #312817. Single-broker:
  Adam takes every lead, every state. Never reintroduce per-state routing.
- **Phone:** (888) 931-0211 (every tel: link)
- **Locked proof numbers:** 250+ DSCR/non-QM loans closed, 70+ lender network. Do not inflate.
- **The 15 DSCR hard rules in `clients/tall-timbers/CLAUDE.md` apply to every line of copy
  here.** Highlights: no close-day counts, no rate-shopping language, no "soft pull" claims,
  same-day (never 24-hour) pre-approval, no em-dashes, consultative not gatekeeper.

## Tracking (wired and verified 2026-07-21)

- **Google Ads gtag:** `AW-18132955750`, loaded site-wide from `Layout.astro` via
  `brand.gtagId`.
- **New Lead conversion:** `AW-18132955750/Tbu1CMmLkrUcEObku8ZD`, fired from the
  `/thank-you` inline script ONLY when a real submission exists in sessionStorage
  (`lead-summary`) OR `?demo=1` is present (Tag Assistant verification:
  `https://talltimbersrfs.com/thank-you?demo=1`). Bare visits and bots never fire it. Do not
  simplify into an unconditional call.
- **Hotjar:** site id `6725186`, loaded site-wide from `Layout.astro` via `brand.hotjarId`.
- **Booking calendar:** Adam's GHL widget `9CBi2dkCfuszehuLkyA1`
  (`https://api.leadconnectorhq.com/widget/booking/...`) embedded on `/thank-you` via
  `brand.bookingEmbedUrl`, using the known-good GHL iframe pattern (scrolling="yes",
  capped height, no overflow-hidden wrapper).

## Everything brandable lives in `src/config/funnel.ts`

Name, NMLS, phone, logo path, proof numbers, booking URL, gtag ids, Hotjar id, specialist
(+ headshot), ticker deals, deal-story reviews, FAQs, TCPA copy.

## Lead flow

Form (`FunnelForm.tsx`) → POST `/api/lead` (serverless) → forwards server-side to
`LEAD_WEBHOOK_URL` env var (set in Vercel; never read webhooks in browser code). Payload is
the funnel-template contract documented in `deliverables/WIRING.md` (NOT the old site's flat
Zapier payload; the old Zap's field mapping does not match). Honeypot field `website` drops
bots server-side.

**ONE webhook per lead, no partial captures (2026-07-27).** `sendPartial` used to fire a
`partial: true` webhook after name+email, which forces two intake automations in the CRM.
Removed. The form fires once from `submit()` with name + email + phone all present;
`partial` stays in the payload hardcoded `false` so a mapped CRM field never sees a missing
key. **Do not re-add it.**

**TCPA consent is an explicit gated checkbox (2026-07-27).** On the phone step, ABOVE the
submit button, starting UNCHECKED, blocking submit until checked, gated in `submit()` AND
in `/api/lead` (400 `consent required` when `tcpaConsent !== true`), because a client-only
gate is bypassable and this is a legal record. Never pre-check it, never move it below the
button. The payload carries a seven-field record, not a bare boolean: `tcpaConsent`,
`tcpaConsentText` (verbatim, ~550 chars, needs a Multi Line field in GHL or it truncates),
`tcpaConsentAt`, `tcpaConsentUrl`, plus server-stamped `tcpaConsentIp`,
`tcpaConsentUserAgent`, `tcpaConsentReceivedAt`. Verify with `node tools/tcpa-test.mjs`.

Rationale and the standing rules live in the workspace CLAUDE.md ("Lead capture: standing
rules for every funnel"); reference build is `clients/Internet-Loans-Direct/`.

## Old-site route redirects (in `astro.config.mjs`)

`/lp/georgia-dscr` → `/`, `/lp/florida-dscr` → `/`, `/privacy-policy` → `/privacy`,
`/terms-of-service` → `/legal`. Keep these; live Google Ads final URLs pointed at the old LP
paths until the campaigns are updated.

## Build / QA

- `npm run dev` (port 4321) · `npm run build` must pass before commit.
- `node tools/shoot.mjs <prefix>`: screenshot harness (desktop + mobile emulation + form
  steps + decline + thank-you).
- `node tools/verify-tracking.mjs`: asserts gtag/Hotjar load, conversion gating (bare
  thank-you fires nothing, ?demo=1 and real submits fire the labeled conversion), full
  form submit → /api/lead 200 → redirect → personalization, honeypot value reaching the
  payload, old-route redirects, and mobile booking iframe sizing. All 16 checks passed
  2026-07-21 (after the verification-workflow fixes below).
- Verification workflow findings fixed 2026-07-21: "50 States served" stat replaced with
  "Fast / Closings on clean files" (unlocked claim, contradicted by old EXCLUDED_STATES);
  ticker reframed from invented "Funded $X" amounts to approved deal-story lines;
  footer/legal/privacy rewritten from directory voice to licensed-shop "we" voice (Rule 9);
  thank-you tab title "You're Eligible" → "Eligibility Check Complete"; honeypot moved to
  the always-mounted form shell (it unmounted before submit and could never catch a bot;
  QA tools must exclude `#ff-company` when selecting form inputs).

## Deliverables

`deliverables/` (eligibility email, follow-up emails + SMS, call-prep one-pager, WIRING.md)
are rebranded for Tall Timbers and ready to build in GHL. The funnel copy promises an
eligibility summary email; until Workflow A in WIRING.md exists in GHL, that promise is
unbacked. Wire it before or immediately after cutover.

## Deploy

**Repo:** `github.com/tannerk711/talltimbers`, production branch **`main`** (this folder is
the canonical clone). **Flipped 2026-07-22 on Tanner's call:** `main` now IS the funnel, so
the existing talltimbers Vercel project serves it at talltimbersrfs.com; the old site's
last commit is `97ff274` (fully recoverable; rollback = revert/reset push). Add
`dscr.talltimbersrfs.com` as an additional domain on the same project (it serves
production, no branch assignment needed). Env: `LEAD_WEBHOOK_URL` (set for all
environments). `@astrojs/vercel` adapter handles `/api/lead`. The `dscr-funnel` branch
matches main; push to `main` for deploys.

## Lessons Learned

- **[2026-07-27] Removing partial captures broke two verify-tracking assertions, not the site:** the end-to-end block and the honeypot block both depended on the partial POST firing at the contact step, so with partials gone they reported 5 failures on a perfectly healthy build. Both now finish the form (phone + consent) instead. Lesson: when a QA script asserts on a side effect of a feature you are deleting, the script needs updating in the same commit, and a wall of failures right after a change is worth reading carefully before assuming a regression.
- **[2026-07-27] The email promise in the thank-you phone branch is dead code here:** Adam has a booking calendar, so `bookingEmbedUrl` is set and the iframe branch always renders. The "reply to the email we just sent" line lives in the unreachable `else` branch and was left alone. The visible one was the contact-step subtitle ("Your eligibility summary lands in your inbox"), reworded since Workflow A still does not exist in GHL.
- **[2026-07-21] shoot.mjs path bug:** `new URL().pathname` keeps `%20` for spaces and
  breaks on this workspace path; use `fileURLToPath`. Fixed here; fix upstream in
  dscr-funnel-template if reused.
- **[2026-07-21] gtag config ping vs conversion:** the `viewthroughconversion` request with
  `en=gtag.config` fires on every page load with an Ads tag. Only requests carrying the
  conversion label are real conversions; assert on the label when verifying.

---
> Source: [tannerk711/talltimbers](https://github.com/tannerk711/talltimbers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
