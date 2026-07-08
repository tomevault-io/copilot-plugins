## velonyx-portfolio

> **Local working tree:** `/Users/apple/Cursor-Claude/`

# Velonyx Systems — Claude Code Handoff

**Local working tree:** `/Users/apple/Cursor-Claude/`
**GitHub:** [`carlitolamar1989/velonyx-portfolio`](https://github.com/carlitolamar1989/velonyx-portfolio) (default branch: `main`)
**Live site:** `https://velonyxsystems.com` / `https://www.velonyxsystems.com`
**Subdomain (RETIRED 2026-06-16):** `gdk.velonyxsystems.com` is being taken down — the 6 in-site demos (`/demos/*`) replace it. The hero "Explore the Live Demo" button now routes to `/demos/tax/` (Benjamin Lewis). Do not re-link the gdk subdomain.
**Founder/Operator:** Carlos Glover ([admin@velonyxsystems.com](mailto:admin@velonyxsystems.com), (877) 317-8643, San Diego CA)

> **Last updated: 2026-06-13.** This file was significantly rewritten after the June audit
> because the previous version was stale (it still described the $3,000 Founding Member model,
> a Stripe checkout funnel, a hero video, and a `for-barbers.html` page — none of which match
> the current site). If something here disagrees with the code, trust the code and fix this file.

---

## ⚠️ Repo hygiene — local clone drifts behind `origin/main`

Recent site work has been landing through GitHub PRs (#22–#39). The local clone is easy to leave
several commits behind, which means editing stale files. **Always `git pull` before starting work**
and verify `git rev-list --left-right --count main...origin/main` shows `0 0`.

---

## What this site is

A marketing site for **Velonyx Systems**. As of **June 16, 2026** the site is **repositioned** around
an aspirational, worldwide message: the hero is *"The Future of Business Runs on AI."* — eyebrow
*"The businesses that embrace AI win,"* subhead *"A custom AI that works while you do."* The buyer is
**any business owner, anywhere in the world** (no industry/geo targeting); the pitch is empowerment, not
fear. Core promise: a custom AI that does the work of a help desk / assistant — answers chat + phone
24/7, captures, qualifies, and books leads, and runs the busywork — so owners scale **without hiring**.

> **⚠️ Retired framing — do NOT reintroduce:** "Never Miss Another Lead" and all fear/loss language
> ("leads slip," "78% first to respond," "losing money"); home-services / trades targeting (HVAC,
> plumbing, electrical, garage doors, "under a sink/truck" + those keywords + JSON-LD `serviceType`);
> San Diego / local / `geo.*` meta (areaServed is now "Worldwide"); and the rent-vs-own angle
> ("yours to own," "own it," "no rent"). The 6 demo cards (Garage Door Kings, etc.) stay — they're
> intentional cross-industry *examples* that prove the AI works for any business.

Underneath the message, the actual product is the same **custom platform**: branded website + AI chatbot
+ AI voice agent + lead automation + booking + payments + customer financing + SMS + an owner dashboard.

The site exists to:
1. Capture leads via the floating lead-form widget and route serious prospects to **book a call**.
2. Showcase the 6 in-site live demos under `/demos/*` (the old `gdk.velonyxsystems.com` subdomain is retired).
3. Provide an "in-bio" landing at `/connect/` for QR-code / DM / SMS sharing.

### ✅ Positioning alignment (resolved 2026-06-16)
The old title/meta/JSON-LD vs. body mismatch is **fixed** — the `<title>`, meta description, keywords,
OG/Twitter, and all JSON-LD now carry the unified "The Future of Business Runs on AI" / worldwide /
AI-system message. Next marketing step on Carlos's list: **SEO + organic content** built around this
new positioning (rank for "AI for business / custom AI system," not local/trades terms).

---

## Funnel & pricing (current)

**Funnel = book a call.** There is **no live self-serve payment** on the site right now.
- Primary CTAs open the floating lead-form widget (`data-vx-form-open`) → POSTs a lead.
- `book.html` is the real booking step — an inline **Calendly** embed (`calendly.com/admin-velonyxsystems/30min`).
- `checkout.html` and `financing.html` are informational; their CTAs route to **`/book.html`** ("book a
  call and we'll send the secure link"). They must **not** present "Pay now / Go to checkout" dead-ends.

**Pricing:** **$700 one-time build + $70/month.** Optional growth add-ons ($250 / $500 / $1,500/mo)
appear in the pricing/JSON-LD. The old **$3,000 Founding Member + $100/mo** model is **retired** — if you
still see it anywhere, it's stale.

**Stripe (pending):** Carlos is creating **new Stripe Payment Links for the $700 / $70 prices**. Until
those exist and are wired in, keep the funnel honest as book-a-call. When the links are ready, wire them
into `checkout.html` (and anywhere a "pay" CTA belongs) — do not reintroduce "pay" language before then.

---

## Lead capture plumbing

- **Floating lead-form widget:** `assets/velonyx-lead-form.js`, opened by any `data-vx-form-open` element.
  POSTs to the conversational endpoint `…/form-turn` with a fallback to the leads endpoint:
  `https://jyo775chsk.execute-api.us-east-1.amazonaws.com/leads`.
- **Lead payload shape** (reused by `sms-opt-in.html`): `{ firstName, lastName, phone, email, service, description }`;
  success when the JSON response has `success: true`.
- **Floating AI demo widget:** `assets/velonyx-ai-demo-widget.js` — a self-contained scripted/canned
  conversation that demos the product. No live backend.
- **Orphaned code:** `index.html` still contains the old two-step `bookingForm`/`bookingOverlay` modal +
  `openBooking()`, which is **defined but never called** (superseded by the widget). Safe to remove when
  convenient; left in place for now to avoid risk.

---

## Tech stack

| Layer | Tool | Notes |
|---|---|---|
| Markup | Static HTML5 | No framework, no build step |
| Styling | Vanilla CSS, `:root` custom properties | Inline in `<style>` blocks (homepage `<style>` is large) |
| JS | Vanilla JS | GSAP + ScrollTrigger via CDN (homepage only, deferred) |
| Fonts | Google Fonts — Space Grotesk + DM Sans | Non-blocking preload + onload swap; `<noscript>` fallback |
| Hosting | GitHub Pages | Auto-deploy on push to `main` via `.github/workflows/deploy.yml` (~20–30s) |
| Domain | `velonyxsystems.com` (apex) | CNAME at root; DNS at Namecheap |
| Booking | Calendly inline | `book.html` |
| Payments | Stripe Payment Links | **None live right now** (see Funnel & pricing) |
| Analytics | GA4 (`G-F838ZEJ22J`) + Meta Pixel (`1486954096175579`) | Gated by CCPA cookie consent (`assets/cookie-consent.js`) |
| Marketing banner | `assets/urgency-banner.js` | Thin fixed top banner. Space is reserved pre-paint (`body.vx-banner-active` + `#vx-banner-reserve` styles) to avoid CLS |

---

## Folder structure (current)

```
Cursor-Claude/                        ← velonyx-portfolio repo
├── index.html                        Homepage (AI lead-capture hero; static AI-helpdesk image bg)
├── checkout.html                     Offer page — CTA books a call (noindex; NOT in sitemap)
├── book.html                         Calendly inline booking (the real conversion step)
├── financing.html                    BNPL explainer — CTAs book a call
├── privacy.html, terms.html, msa.html, sow.html, sms-terms.html, sms-opt-in.html, refund-policy.html
├── 404.html                          Brand-styled 404
├── sitemap.xml, robots.txt, CNAME, favicon.ico, humans.txt, contact.vcf
├── connect/index.html                "Link in bio" landing
├── assets/
│   ├── hero-bg-ai-helpdesk.webp / -1440.webp   Homepage hero background (LCP element)
│   ├── velonyx-lead-form.js          Floating lead-form widget (data-vx-form-open)
│   ├── velonyx-ai-demo-widget.js     Floating scripted AI demo
│   ├── urgency-banner.js             Top marketing banner
│   ├── cookie-consent.js             CCPA banner — gates GA4 + Meta Pixel
│   ├── marketing-config.js           Pixel/GA4 IDs
│   ├── motion/hero-slide-{1,2,3}.{webm,mp4}    Remotion motion clips (3-image story bar; preload=none)
│   ├── hero-slide-{1,2,3}.webp       Posters for the motion clips
│   ├── vs-logo-shield-512.webp       Brand mark (hero/logo). NOTE: vs-logo-shield-clean.png is 404 — don't reference it
│   ├── vs-logo-monogram.{webp,png}   Nav mark
│   ├── gdk-preview.{webp,png}        Homepage portfolio screenshot of gdk.velonyxsystems.com (<picture>, WebP primary)
│   └── velonyx_hero_web.mp4          Old hero video — no longer used on the homepage (kept in repo)
├── demos/garage/                     GDK static demo (live operational demo is on the gdk subdomain)
├── demos/smp/                        Second demo
├── client-demos/, platform/          Legacy / learning-lab (unused on live; see platform/README.md)
├── remotion/                         Remotion workspace for the motion-graphics pipeline
├── docs/                             Audit, perf, decision, and summary docs
├── scripts/refresh-gdk.sh            One-command GDK demo screenshot refresh ("refresh GDK")
└── CLAUDE.md                         This file
```

### Sibling working directories (NOT inside this repo)

| Path | What it is |
|---|---|
| `/Users/apple/Cursor-Claude-trades-template/` | Next.js + Supabase + Stripe + Twilio portal foundation. Deployed at `gdk.velonyxsystems.com`. The `gdk` demo is a **separate codebase** — don't fix gdk bugs by editing this repo. |
| `/Users/apple/Cursor-Claude-external/` | Cloned third-party SDKs (e.g. `notebooklm-py`). |
| `/Users/apple/Cursor-Claude-archive/` | Old Lambda handler backups. |

---

## Compliance & consent

- **CCPA cookie banner** on every public page (`assets/cookie-consent.js`); GA4 + Meta Pixel only fire
  after explicit accept (`localStorage.velonyx_cookie_consent === 'accepted'`).
- **Legal pages** live + linked from every footer: privacy, terms, msa, sow, sms-terms, sms-opt-in, refund-policy.
- **SMS consent:** `sms-opt-in.html` has a non-pre-checked SMS consent checkbox + a separate privacy/terms
  checkbox, and (as of the June audit) the form actually submits to the leads API. Keep STOP/HELP language intact.
- **robots.txt:** allows root; disallows `/platform/portal/`, `/platform/admin/`; points to sitemap.

---

## Common operations

- **Deploy:** push to `main` → GitHub Actions deploys in ~20–30s. Verify `curl -sI https://velonyxsystems.com/`.
- **Refresh the GDK screenshot:** `bash scripts/refresh-gdk.sh` (or say "refresh GDK").
- **Wire the new Stripe links (when Carlos provides them):** set the link as the CTA on `checkout.html`
  (replacing the `/book.html` interim CTA + the "link is being prepared" note), and update any pay CTAs.
- **Sitemap:** keep `checkout.html` OUT of `sitemap.xml` (it's `noindex`). `for-barbers.html` was deleted —
  don't re-add it.

---

## Performance notes

- Mobile is the dominant profile (QR / DM / SMS shares). The legal/utility pages score 85–100.
- The **homepage regressed** after the AI-repositioning work (LCP ~5.6s, CLS 0.217 in the June audit).
  Fixes applied on the audit branch: hero-image preload hoisted to the top of `<head>`; urgency-banner
  space reserved pre-paint (CLS); footer link contrast raised to pass WCAG AA.
- **Still open:** homepage LCP render-delay is dominated by main-thread work (large inline `<style>` parse
  + GSAP/ScrollTrigger init before paint). Reducing it means deferring GSAP init until after LCP and/or
  splitting the inline CSS — higher-risk changes that should be tested live (a prior loader-script change,
  PR #39, briefly blanked the page, so be careful with first-paint JS).
- See `docs/AUDIT_2026-06-13.md` for the full June audit and `docs/PERF_AUDIT_SWEEP.md` for the May baseline.

---

## Pointers for future sessions

1. `git pull` first (the local clone drifts behind `origin/main`).
2. The funnel is **book-a-call** until Stripe links exist — never reintroduce "Pay now / Go to checkout"
   dead-ends.
3. **Edit the root files** (`index.html`, etc.) — there's no mirror folder.
4. Follow established perf patterns: WebP-first with `<picture>` fallback, the preload+onload+noscript
   font pattern, defer JS where possible.
5. Be careful with first-paint JS on the homepage (see Performance notes / PR #39).
6. Close every page with the motto **"Your Legacy, Engineered With Precision."**

---
> Source: [carlitolamar1989/velonyx-portfolio](https://github.com/carlitolamar1989/velonyx-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
