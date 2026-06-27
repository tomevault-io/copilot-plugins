## arogyadhamaweb

> > Context handoff for stateless Claude Code sessions. Read this fully before touching anything.

# CLAUDE.md

> Context handoff for stateless Claude Code sessions. Read this fully before touching anything.

---

## What this is

Public website for **Arogyadhama** — a 350-bed integrative medicine hospital at S-VYASA Yoga University, Prashanti Kutiram, Bengaluru. Founded in the lineage of Padma Shri Dr. H. R. Nagendra. Forty years of practice, 30,000+ patients treated, 400+ indexed research papers.

This is the **public-facing marketing & enquiry site** — not the patient app, not the clinical app. It exists to:

1. Position Arogyadhama for donors, international patients, S-VYASA stakeholders
2. Capture OPD/IPD enquiries (form posts to ERPNext, eventually)
3. Showcase the five healing systems, eleven clinical departments, 177-therapy catalog
4. Drive traffic to the existing arogyadhama.com — eventually replacing it

**Not in scope here:** patient app (separate Next.js project — 58-screen blueprint), clinical app (already deployed at `arogyadhama-connect.vercel.app`), HIMS, ERPNext customization.

---

## Stack

```
Next.js 14.2.35 (App Router)
TypeScript 5.6.3
Tailwind 3.4.15 (custom design tokens)
React Three Fiber 8.17 + drei 9.114 + three 0.170
Framer Motion 11.11
GSAP 3.12 (loaded but not yet used heavily)
Lucide React (icons)
Node 24.x on Vercel

Hosted: Vercel (e-digivault team)
Repo: github.com/abi412003-hub/arogyadhamaweb
```

No Supabase. No mock data layer. No App Router server components doing fetches yet (intentional — the data is inline until ERPNext wiring lands).

---

## Live URLs

| Surface | URL |
|---|---|
| Production (team-aliased) | https://arogyadhamaweb-e-digivault.vercel.app |
| Latest deployment | https://arogyadhamaweb-kp9r3wp66-e-digivault.vercel.app |
| `arogyadhamaweb.vercel.app` | **Owned by another Vercel team — DO NOT rely on this URL** |
| Inspector | https://vercel.com/e-digivault/arogyadhamaweb |
| GitHub | https://github.com/abi412003-hub/arogyadhamaweb |

**Vercel Deployment Protection** is currently ON — only e-digivault team members can view. To make public: Settings → Deployment Protection → set to Disabled.

---

## Vercel & GitHub identifiers

```
Team:        e-digivault            team_H3smw5RN6X19PkPl7KE2p0C3
Project:     arogyadhamaweb         prj_PdHrBsFqLH8YyL4QFZ5AdhLzrVaW
GitHub repo: abi412003-hub/arogyadhamaweb
Default branch: main
```

**Commits MUST be authored as `Abinash <abi412003@gmail.com>`.** Vercel only auto-deploys on commits matching this email. Set git config when starting a session:

```bash
git config user.name "Abinash"
git config user.email "abi412003@gmail.com"
```

---

## Environment variables

Set in Vercel dashboard → Project Settings → Environment Variables. **Never commit actual values.** See `.env.example` for the full list:

```
ERPNEXT_API_TOKEN          # rotate from arogyadhama.m.frappe.cloud → API Access
NEXT_PUBLIC_ERPNEXT_URL    # https://arogyadhama.m.frappe.cloud
```

---

## Design system — "Sacred Modernism"

Editorial luxury (Aman / Six Senses tier) with sacred geometry undertones. Slow breathing motion, never kinetic.

**Type**
- Display: Cormorant Garamond (with italic for emphasis)
- Body: DM Sans
- Sanskrit: Tiro Devanagari Hindi
- Mono / data: JetBrains Mono

All loaded via `next/font/google` in `src/app/layout.tsx` and exposed as CSS variables.

**Palette (CSS variables in `globals.css`, Tailwind tokens in `tailwind.config.ts`)**

| Token | Hex | Usage |
|---|---|---|
| `forest-deepest` | `#0F2419` | Footer, deepest backgrounds |
| `forest-deep` | `#1A3A2E` | Dark sections |
| `forest` | `#2D5A3D` | Primary brand, CTAs, Yoga system |
| `forest-soft` | `#5C8A6E` | Naturopathy system |
| `gold` | `#C9A961` | Sacred accent, hairlines, Ayurveda system |
| `gold-pale` | `#E8D5A3` | Italic display emphasis |
| `cream` | `#F5EFE0` | Section canvas |
| `paper` | `#FAF7F0` | Body background |
| `ink` | `#1C1C1A` | Body text, Physiotherapy system |
| `terracotta` | `#B8694A` | Acupuncture system, sparing accent |

**Motion vocabulary**
- Easing: `cubic-bezier(0.16, 1, 0.3, 1)` (out-expo) for entrances
- Durations: 700ms–1.6s for entrances, 200–400ms for hover
- Animations: `breathe` (6s loop), `drift` (24s), `rotate-slow` (60s), `fade-up`
- `prefers-reduced-motion: reduce` is honored globally

**Don't:** add new fonts. Don't use generic SaaS purples/blues. Don't add box-shadows that aren't `--shadow-soft` or `--shadow-deep`.

---

## File map

```
src/
├── app/
│   ├── layout.tsx              # Font loading, root nav + footer
│   ├── page.tsx                # Home composition (8 sections)
│   ├── globals.css             # CSS vars, paper-grain bg, gold-rule, eyebrow
│   ├── systems/page.tsx        # 5 systems + mandala + per-system editorial
│   ├── departments/page.tsx    # 11 clinical departments
│   ├── therapies/page.tsx      # Filterable catalog (Suspense + URL ?section=)
│   ├── campus/page.tsx         # Flythrough + campus details
│   ├── about/page.tsx          # S-VYASA history, timeline, leadership
│   └── contact/page.tsx        # Form + phone tree (currently placeholder POST)
│
├── components/
│   ├── ui/
│   │   ├── Navigation.tsx      # Editorial top bar, scroll bg-change, mobile drawer
│   │   └── Footer.tsx          # Brand + visit + reach + nav, Vivekananda quote
│   ├── home/
│   │   ├── Hero.tsx            # Higgsfield video bg + ParticleField + staggered type
│   │   ├── Introduction.tsx    # Asymmetric editorial + 4-stat strip
│   │   ├── SystemsTeaser.tsx   # Wraps SystemsMandala
│   │   ├── DepartmentsTeaser.tsx
│   │   ├── TherapiesTeaser.tsx # 9-section grid linking to /therapies?section=X
│   │   ├── Testimonials.tsx    # 5 patient quotes (paraphrased)
│   │   └── DonorCTA.tsx        # Forest-deep + yantra SVG bg
│   ├── three/
│   │   ├── SystemsMandala.tsx  # R3F: bindu + 5 petals + pentagram lines
│   │   ├── ParticleField.tsx   # 800 gold particles, AdditiveBlending
│   │   └── CampusFlythrough.tsx# Sticky-scroll Three.js scene + commented video swap
│   └── therapies/
│       └── TherapyGrid.tsx     # Filter (section + system + search) + modal
│
└── lib/
    ├── systems-data.ts         # 5 systems (Yoga, Ayurveda, Naturopathy, Acupuncture, Physio)
    ├── departments-data.ts     # 11 clinical departments
    ├── therapies-data.ts       # ~50 representative therapies (production has 177)
    └── utils.ts                # cn, clamp, lerp

public/
├── videos/                     # Drop Higgsfield MP4s here. README.md inside has prompts
└── images/
    └── hero-poster.svg         # SVG fallback for hero <video> poster

vercel.json                     # Forces framework: nextjs (don't remove)
.env.example                    # Documents env vars
README.md                       # Public-facing repo readme
CLAUDE.md                       # This file
```

---

## Data layer — ERPNext mapping (when wiring)

ERPNext: `arogyadhama.m.frappe.cloud` (39 custom DocTypes already exist on the server)

| Frontend file | ERPNext DocType (target) | Status |
|---|---|---|
| `lib/therapies-data.ts` | `Therapy` (custom) | NOT WIRED — local data |
| `lib/departments-data.ts` | `Clinical Department` (custom) | NOT WIRED |
| `lib/systems-data.ts` | Likely keep local — taxonomic, not transactional | INTENTIONALLY LOCAL |
| `app/contact/page.tsx` form | `Patient Enquiry` (custom — needs creation) | PLACEHOLDER |
| Future: doctor profiles | `Doctor` or `Healthcare Practitioner` | NOT BUILT |
| Future: case studies | `Blog Post` (standard) | NOT BUILT |

**Wiring approach when ready:**
1. Create `src/app/api/proxy/route.ts` — Edge function that forwards to ERPNext with `Authorization: token ${ERPNEXT_API_TOKEN}` server-side
2. Create `src/lib/erpnext.ts` — typed fetch helpers (`getList`, `getDoc`, `createDoc`)
3. Replace `THERAPIES` const in `therapies-data.ts` with a server component fetch
4. Wire `/contact` form to POST `Patient Enquiry`

**Reference:** the e-DigiVault Admin app already has a working proxy pattern at `/api/proxy` using `@/lib/erpnext.ts` helpers — copy that pattern, don't reinvent.

---

## What's done · what's pending

### ✅ Done
- 7 pages, full editorial copy paraphrased from arogyadhama.com (real S-VYASA content, real phone tree, real history)
- 3 Three.js scenes (mandala, particle field, scroll-driven campus flythrough)
- Sacred Modernism design system fully implemented
- Real content: 11 departments, ~50 therapies, 5 testimonials, 6-point timeline, 12 distinguished visitors
- Deployed to Vercel, framework correctly detected via `vercel.json`
- Mobile responsive (tested via build, not yet via real device)

### 🔧 Pending — high priority

1. **Higgsfield hero video** — drop `hero-loop.mp4` (8–12s loop, <4MB, 1920×1080) into `public/videos/`. Prompt library in `public/videos/README.md`.
2. **Disable Vercel Deployment Protection** to make site publicly accessible.
3. **ERPNext wiring** — proxy + therapies catalog + contact form.
4. **Custom domain** — point `arogyadhama.com` (currently on the legacy WordPress site) at Vercel when ready to cut over. Will need to coordinate DNS + redirect plan.
5. **OG image** — `public/images/og-arogyadhama.jpg` (1200×630). Currently no preview image when shared on WhatsApp/LinkedIn.
6. **Favicon set** — `public/favicon.ico` + `apple-touch-icon.png`. Default Next.js icon currently.
7. **Bump therapies catalog** from ~50 representative → all 177. Best done via ERPNext seed + live fetch, not as a static file.

### 🔧 Pending — medium priority

8. **Doctor profile pages** — `/about/team/[slug]`. Need photography first.
9. **Case studies / blog** — there are 4 case study URLs on arogyadhama.com (back pain, diabetes, etc.). Either migrate or rebuild.
10. **Virtual tour integration** — currently linked out to `turiya.co/360/SVYASA/` from contact page. Could embed.
11. **Tariff page** — `/tariff` (legacy site has it). Probably needs ERPNext tariff master.
12. **FAQ page** — `/faqs`. Static markdown content acceptable.
13. **Daily schedule page** — show what a typical IPD day looks like.

### 🔧 Pending — low priority / nice-to-have

14. **GSAP scroll-trigger choreography** — the import is in `package.json` but unused. Could enrich section transitions, but not needed for v1.
15. **Higgsfield B-roll plates** — therapy room interiors, naturopathy garden, panchakarma scene. Add as silent ambient cutaways between sections.
16. **English ↔ Kannada toggle** — S-VYASA serves a heavily local audience. Defer until v2.
17. **Patient testimonial videos** — arogyadhama.com has video testimonials (Naina, Pishu, Ranganath, etc.). Embed when assets available.

---

## Working agreements

1. **Commit author must be `Abinash <abi412003@gmail.com>`** — non-negotiable, Vercel webhook depends on it.
2. **Never commit secrets** — token leaked once already. Always check `git diff` before committing config files.
3. **No Supabase, no Firebase.** Backend is ERPNext only. Per project-wide rule.
4. **Mock data is acceptable temporarily** but every static-data file in `lib/` should have a comment noting which DocType it eventually maps to.
5. **Three.js scenes are wrapped in `dynamic(... ssr: false)`** — never render them server-side.
6. **`prefers-reduced-motion`** must be honored on any new motion you add.
7. **Tailwind classes only** — no styled-components, no CSS modules, no inline `<style>` tags except for one-off dynamic transforms.
8. **One commit per logical change** — small, atomic, reviewable.

---

## Cross-project context (for Claude — won't show up to user otherwise)

This site is part of a broader Arogyadhama ecosystem Abinash is building:

| Surface | Repo / Path | Status |
|---|---|---|
| **Public website** (this) | `abi412003-hub/arogyadhamaweb` | Deployed, content complete, video pending |
| **Clinical App** | `abi412003-hub/arogyadhama-connect` | Deployed at `arogyadhama-connect.vercel.app`, fully ERPNext-wired |
| **Patient App** | not yet started | 58-screen blueprint exists |
| **HIMS / ERPNext backend** | `arogyadhama.m.frappe.cloud` | 39 custom DocTypes, 177 therapies seeded |
| **Legacy WordPress site** | arogyadhama.com | Source of content + photography for migration |

The clinical app shares design DNA with this site (Forest Green / Gold / Cream) but is a different visual register — utilitarian, dense, dashboard-like. This site is editorial, sparse, slow.

---

## Resume-from-here protocol

When starting a new Claude Code session on this repo:

1. **Read this file fully.**
2. **`git pull --rebase origin main`** — there may be commits from other sessions or directly from Lovable/Vercel deploy hooks.
3. **Set git config** as above.
4. **Run `npm install` then `npm run dev`** — verify the site renders locally before changing anything.
5. **Check `vercel.json` exists.** If it doesn't, the routing will silently break on next deploy (do not remove it).
6. **Look for the user's intent in their first message.** If they say "deploy" — verify the live site after pushing. If they say "fix" — reproduce locally first. If they say "add" — find the closest existing pattern in the codebase and match it.

When ending a session:

- Update this file's **Done / Pending** lists if anything moved
- Push commits with the proper author email
- Verify the Vercel deployment turned green (`Vercel:list_deployments`)

---

*Last updated: deployment commit `40f17be` (vercel.json fix). Site live behind team SSO. Higgsfield video and ERPNext wiring are the next big drops.*

---
> Source: [abi412003-hub/arogyadhamaweb](https://github.com/abi412003-hub/arogyadhamaweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
