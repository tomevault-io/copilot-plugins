## my-portfolio

> Guidance for Claude Code (and any AI assistant) working in this repo.

# CLAUDE.md

Guidance for Claude Code (and any AI assistant) working in this repo.

## What this is
Personal portfolio for **Stepan Roze** — live at **roze.live**. A single-page
React app. Two audiences via a **view mode** toggle: **Client** (hire me:
services, prices, projects) and **Company** (full CV: experience + skills).
Multilingual (en, nl, ar, es), dark/light theme.

## Who this site is for (2026-08 repositioning — read before changing copy)
The goal changed from winning freelance clients to **getting hired as an
employee**, at companies in Belgium, the Netherlands and the rest of Europe.
That decision drives most of what follows:

- **CV mode is the default view.** A recruiter seeing an hourly rate before the
  experience reads the page as a freelance pitch, not a job application.
- **Ukrainian was removed from the codebase entirely**, not hidden — the
  Ukrainian market is not a target and the strings were dead weight in every
  bundle. `L()` now takes `(en, nl, ar, es)` and `Language` has no `"uk"`, so
  new Ukrainian copy will not compile.
- **Dutch is excluded from browser auto-detection** even though the version
  exists. The audience has Dutch browsers, the author cannot proof-read Dutch,
  and weak Dutch costs more trust than plain English does in Benelux IT. Put it
  back only after a native speaker reviews the copy.
- **Client-only content must not leak into CV mode.** It reads as "looking for
  gigs" to someone considering an employment offer.

Client mode still exists and still works — it is one click away, not deleted.

## Stack
- **React 19 + Vite 8 + TypeScript 5.9**
- **Tailwind CSS v4.3**, **Motion** (`motion/react`, the framer-motion successor)
- Services carousel is **CSS scroll-snap**, not a library (react-slick is gone)
- **Supabase** edge function backend (`make-server-a62f57c7`), now used for the
  contact form only (three plain `fetch` callers: `contact-section`,
  `book-call-fixed`, `scroll-to-top-button`)
- Deploy: **Netlify** (`netlify.toml`). `dist/` is **committed to git**.

**Vite 8 bundles with rolldown**, so build config is `build.rolldownOptions`
(not `rollupOptions`) and chunking uses `output.advancedChunks.groups`.
`esbuild` is an explicit devDependency only because `@tailwindcss/node` still
imports it and pnpm does not hoist it.

## Commands
- Dev server: `npm run dev` → **http://localhost:3000/** (another project may
  already hold that port — pass `--port` if the page looks like someone else's)
- `npm run check` = typecheck + lint + build. **All three must be green**;
  they are as of the 2026-07 cleanup, so any error you see is yours.
- Lint is at **0 errors / 20 warnings**, and `--max-warnings 20` is a ratchet:
  the count may go down, never up (it went 32 → 20 when the account surface was
  removed). The remainder needs a judgement call at each call site, so do not
  silence them in bulk.
- Netlify build command: `npm run build` → publishes `dist/`
- **Package manager is pnpm** (`pnpm-lock.yaml`); install via
  `corepack pnpm install`. pnpm 11 no longer reads the `pnpm` field from
  package.json — **overrides and build-script approvals live in
  `pnpm-workspace.yaml`**. `@types/react` is pinned there: transitive peers pull
  their own copy, and two copies of the React types make every component
  "not a valid JSX element type".
- **Do not push unless asked.** Commit when asked; the user often reviews first.

## Where things live
- `src/app/App.tsx` — providers + top-level page routing (hash based)
- `src/app/components/main-page.tsx` — renders sections per view mode
- `src/app/components/hero-ultra-modern.tsx` — hero (client = agency split,
  company = professional profile)
- `src/app/components/services-creative-slider.tsx` — services + **prices**
  (hardcoded array, NOT from translations)
- `src/app/components/how-i-work.tsx` — process + honest part-time timelines
  (self-contained translations via an inline `L()` helper). **Client mode only**
  — its content ("Intro — free", "Quote & timeline") is a sales funnel, and it
  was 1 528 px of it sitting in the middle of the CV.
- `src/app/components/view-mode-toggle.tsx` — the Client/CV switch, rendered
  **inline inside the header** by `navigation.tsx`. It used to be three
  fixed-position variants that all covered page content; do not float it again.
- `src/app/components/portfolio-creative-slider.tsx` — projects grid (owned work
  only: marinek.store, roze.live) → opens `project-fullscreen-view.tsx`
- `src/app/components/github-showcase.tsx` — live GitHub feed (@irozedev)
- `src/utils/translations.ts` — **huge** 4-language dictionary. Newer components
  skip it and localize inline with an `L(en,nl,ar,es)` helper.
- `src/app/contexts/language-context.tsx` — `OFFERED` is the single source of
  truth for which languages exist. The switcher, `?lang=`, the saved choice and
  auto-detection all validate against it; a language removed there is also
  rejected out of a visitor's stale `localStorage`.
- `src/app/contexts/view-mode-context.tsx` — `client` | `cv`, `setViewMode`.
  **Defaults to `cv`.** The saved choice is read in the `useState` initialiser,
  not an effect (as an effect it painted the wrong mode for one frame), and the
  scroll reset is a `useLayoutEffect` on `viewMode`, not a call inside the
  handler (before the commit, scroll anchoring dragged the page back to ~135px).
- `src/app/hooks/use-modal-a11y.ts` — Escape, focus trap and focus restore for
  dialogs. **Every modal must use it**; none of them had any of this before.

## Design tokens
- Weight scale and fluid type scale are Tailwind v4 `@theme` variables in
  `src/styles/tailwind.css`. `font-bold` deliberately emits 600 and
  `font-black` 700 — the design wants everything a notch lighter. Set it there,
  never with `!important` overrides in `fonts.css` (that is what the old code
  did, and a class always beat those element selectors anyway).
- Light-theme accents are darker than the dark-theme cyan **because they must
  clear 4.5:1 on white**. `#0891b2` was 3.68:1 and failed AA; do not "restore"
  the brighter cyan for light mode.
- Always-on cyan glow is capped at `rgba(0,217,255,0.18)` / 24px blur.
  Hover-only glow can be stronger — interaction feedback, not decoration.

## The chat assistant (important)
- **The live chat is in `scroll-to-top-button.tsx`.** It also holds the
  scroll-to-top button. This is the ONLY mounted chat.
- **It renders in client mode only** (`isClientMode` gate on both the launcher
  and the window). In CV mode the reader is a recruiter who will never use it,
  and it competed with the scroll-to-top button in the same corner. The funnel
  itself is untouched — the services CTA still opens it in client mode.
- The old duplicate chats (`chat-bot.tsx`, `ai-assistant-smart.tsx`) were
  deleted in the 2026-07 cleanup along with 98 other unreachable files.
  Before adding a component, check it is actually reached from `src/main.tsx`.
- The chat is a **sales funnel + smart local assistant, 100% in-browser (zero
  API cost)**:
  - Funnel stages: `intro → budget → timeline → name → email → done`.
    Collects a lead, then POSTs to the `/contact` endpoint (saved server-side +
    emailed via Resend). Quick-reply chips drive each stage.
  - `assistantReply(input, language)` = local intent engine (navigation, site
    guide, view-mode switch, prices, timelines, stack, availability, contact,
    github, CV, experience, location, spoken languages). Returns optional
    **action** buttons (nav-scroll, switch view, open link, start quote).
  - `localAnswer(...)` = shared info knowledge base; returns `""` when nothing
    matches so the funnel treats input as a project description.
- **Real LLM is currently NOT called.** An optional backend exists at Supabase
  `POST .../ai/chat` (Claude Haiku) but is unused. If a hybrid is wanted, call it
  only on `matched:false`, and it needs `ANTHROPIC_API_KEY` set in Supabase
  secrets (paid). Without the key it returns `{model:"fallback"}`.

## Supabase / backend
- Project ref: **`saeohtepfpuzzajfduad`** (note: `...tep...`, an old typo
  `...tef...` appears as a harmless default in the server source).
- Function base path: `/functions/v1/make-server-a62f57c7`
- Secrets (set in Supabase dashboard → Edge Functions):
  - `RESEND_API_KEY` — emails contact/chat leads to **rozedev095@gmail.com**.
    Without it, leads are still saved to the KV store (visible on `#admin`).
  - `ANTHROPIC_API_KEY` — real AI chat (optional, unused right now).
- `src/utils/supabase/info.tsx` holds `projectId` + `publicAnonKey`.

## SEO
Two layers, keep them consistent:
- `index.html` — static meta tags + 5 JSON-LD blocks (crawlers/social scrapers).
- `src/app/components/seo-head.tsx` — runtime meta per language.
- OG image: `public/og-image.jpg` (1200×630). Section anchors: `#hero`,
  `#services`, `#how-i-work`, `#projects`, `#github`, `#about`, `#contact`
  (client), `#experience` (company).
- `public/sitemap.xml` and the static `hreflang` block in `index.html` must list
  exactly the languages in `OFFERED`. A `?lang=` URL for a removed language now
  renders English, so leaving it published would be duplicate content.
- Geo meta deliberately targets **Antwerp**, which is where Stepan wants to
  work — not where he lives (Lommel, Limburg). That is a market signal, not a
  claim of residence; keep the two ideas separate in copy. **A `PostalAddress`
  is on the wrong side of that line** — both `Person` blocks now say Lommel,
  Limburg, matching the CV. Only `geo.*` meta still points at Antwerp.
- **Structured data must not out-claim the copy.** It used to: a `LocalBusiness`
  with an Antwerpen address, opening hours, a phone number, a (mojibake) price
  range and nine priced Offers — a registered business, asserted in
  machine-readable form, that does not exist. Removed. `knowsLanguage` listed
  Arabic and Spanish because the *site* is translated into them; it now lists
  what Stepan actually speaks (English, Ukrainian, Russian, Dutch). `worksFor:
  Freelance` became `seeks: Demand` — the site exists to get him hired.
- The two SEO layers share JSON-LD ids. `index.html` tags its static blocks
  `id="ld-person"` / `id="ld-website"`, and `seo-head.tsx` upserts by that id.
  **Do not remove those ids**: without them the runtime layer cannot find the
  static blocks and appends a second Person and WebSite, leaving two
  conflicting entities for the same person on the page.

## Facts about Stepan (keep copy truthful — no fabrication)
- **Front-End / JavaScript developer, 8+ years.** React, Vue, Next.js,
  TypeScript, Node.js, Knockout.js, Magento.
- Real clients (Experience only, per NDA): childrensalon.com, vogacloset.com
  (luxury e-commerce), Oschadbank (banking CRM). **Never** put employer/client
  work in the public Projects gallery — only owned projects there.
- **Those three are clients, not employers.** The employers are **Ronis BT**
  (childrensalon.com, vogacloset.com) and **E-Consulting** (Oschadbank). Copy
  must say "**for** Oschadbank", never "**at** Oschadbank" — `at` names a
  payroll and a reference check would contradict it. Keep the brand names
  regardless: they are what turns "8 years" into a verifiable scale. The same
  applies in every language (nl `voor`, es `para`, ar `لـ`).
- **Employment timeline (corrected 2026-08-04 — the site had it wrong):**
  E-Consulting ran to **Dec 2024**, not Jan 2024. Since **Jun 2025** he is
  *Eerste keukenhulp* at **Albron / Center Parcs** in Lommel on a permanent
  full-time contract. The only real gap is Jan–May 2025 (relocation), so do not
  write about "a long career break" — there isn't one.
- The Albron entry stays in the timeline on purpose. Work outside the field
  beats an unexplained hole, and for a Benelux employer a permanent Belgian
  contract answers the questions they cannot ask: right to work, residence,
  actually living here. It carries **`aside: true`** in `data/experience.ts`,
  which renders it as a footnote rather than a job — a slim dashed strip
  straddling the timeline spine (127px against ~450px for a real card) and one
  muted line on the CV (14mm against 74mm). Present so the months add up, quiet
  so it does not read as a career move competing with the engineering roles.
  Do not restore it to a full card, and do not delete it either.
- Lives in **Lommel (Limburg)**, works his main job **14:00–22:30**, so the free
  block is **mornings until ~13:00** — which is inside the client/interview
  working day, unlike the evenings most side-projects run on.
- Belgium; a **bijberoep** (secondary self-employment) was considered but is not
  registered. Do not state it as fact anywhere in the copy. Small-business
  **VAT franchise** (no VAT charged under ~€25k/yr) would apply if it happens.
- **No self-assigned grades in job titles.** "Middle", "Junior → Middle" and
  "Senior" are all out of `data/experience.ts`. In Belgium the employer sets the
  grade after the interview, so one you award yourself can only cap an offer,
  and on a page that gets scanned rather than read, the lowest word on it wins.
  "Senior Front-End Developer" lives in the **keywords** instead — recruiters do
  filter on it, and a search term is not a claim about a role. Putting it on the
  self-employed entry would also be the one place he outranks himself where
  nobody could contradict it, next to two employer roles carrying no grade.
- **English is the working language; Dutch is weaker than English.** Never claim
  fluent Dutch in copy or structured data.
- Email **rozedev095@gmail.com** (single canonical email). GitHub **@irozedev**.
- Pricing (starting, no VAT), **raised 2026-08-04** off the freelance-era floor:
  automation €65/h (or €500/bot), websites €950, UI design €600, web apps €75/h,
  e-commerce €1,800, consulting €75/h. The old floor was €45/h, which in Belgium
  reads as a warning sign rather than a bargain once roughly half of a fee goes
  to tax and social contributions. Everything was scaled by the same factor as
  the €45 → €65 hourly floor, so the fixed prices still imply the same hours.
  Prices live in **four** places and must move together:
  `services-creative-slider.tsx` (the `services` array *and* `serviceCopy`),
  `how-i-work.tsx`, and the chat price list in `scroll-to-top-button.tsx`.

## Gotchas learned the hard way
- **`backdrop-filter` on an ancestor traps `position:fixed` descendants**
  (mobile menu had to move OUTSIDE the blurred `<nav>`).
- **react-slick `.slick-center`**: with `centerMode:false` no center class is
  added, so blur/dim CSS leaves all cards dimmed. (Fixed by using a plain grid.)
- **Global `keydown` handlers must ignore typing.** A slider hook once called
  `preventDefault()` on Space/arrows globally and ate spaces in every input.
  The services carousel now scopes its arrow handling to the focused track.
- **Keyboard focus is a single global rule** in `index.css`
  (`:focus-visible` + `!important`). 68 buttons had no focus style at all; do
  not go back to per-component focus rings.
- **`react-hooks/rules-of-hooks` is an error, not a warning.**
  `project-fullscreen-view.tsx` had `if (!project) return null` above two
  effects, which crashes with "Rendered more hooks than during the previous
  render" the moment the prop flips. Guards go after every hook.
- Fullscreen project view: no per-frame `setState` on scroll (janky on mobile);
  no parallax transform on the hero image (left a black gap).
- **Theme is driven by `data-theme` on `<html>`**, set by `theme-context.tsx`
  and pre-applied by an inline script in `index.html` (no flash). Do **not**
  go back to `root.style.setProperty(...)` for the palette — inline styles on
  `<html>` outrank every stylesheet, so a partial inline palette leaves the
  vars it doesn't cover stuck on their dark values. The full palettes live in
  `src/styles/theme.css` (`:root` = dark, `[data-theme="light"]` = light).
  `theme-context.tsx` also owns the `theme-color` meta — don't set it elsewhere.
- **Arabic is cursive.** `letter-spacing`, `word-spacing` and a Latin-only font
  (Inter, Manrope, generic `monospace`) all break letter joining or render
  tofu. `src/styles/fonts.css` loads Noto Sans/Naskh Arabic and neutralises
  those properties under `*:lang(ar)`.
- **`?lang=` is real.** `language-context.tsx` reads it on load (priority:
  URL > localStorage > browser) and `handleSetLanguage` mirrors the choice back
  into the URL. `index.html`, `public/sitemap.xml` and `seo-head.tsx` all
  publish hreflang alternates built on it — keep the three in sync. English is
  the bare origin `/` (also x-default); every other offered language is
  `?lang=<code>`. The old Ukrainian `?lang=ua` alias is gone with the language.
- **Never inline a Supabase key as an env fallback in the edge function.**
  `supabase/functions/server/index.tsx` used to default to the real
  `service_role` key; the repo is public. Supabase injects `SUPABASE_URL`,
  `SUPABASE_ANON_KEY` and `SUPABASE_SERVICE_ROLE_KEY` automatically.
- `#admin` is gated on the owner Supabase account (`OWNER_EMAIL` in
  `admin-page.tsx`), not a hardcoded password. It only edits localStorage, so
  it is not a security boundary — anything sensitive needs a server-side check.
- `src/app/components/projects-section.tsx` is dead code with **pre-existing
  syntax errors**; it fails `tsc --noEmit` for the whole project. Vite never
  bundles it, so builds stay green. Delete it or fix it before relying on tsc.

## The account surface is gone (2026-08-04)
Sign-in, the header account menu, the personal cabinet, `/profile`,
`/dashboard`, per-project comments, reactions and favourites were all removed.
A recruiter never signs in, and an account control on a portfolio raises a
question it cannot answer. Deleted: `personal-cabinet`, `profile-settings`,
`cabinet-tabs-extended`, `user-profile-page`, `dashboard`, `project-comments`,
`project-reactions`, `use-favorites`, `utils/supabase/api.ts`, plus the
orphaned `beta-banner` and `site-tour`.

What survives: `auth-context`, `modern-auth-modal` and `admin-page`, reachable
only at `#admin`, which carries its own sign-in button. **`AdminRoute` mounts
its own `AuthProvider`** — App.tsx must not import one. That is what keeps
`@supabase/supabase-js` (202 KB) inside the lazy admin chunk; a static import in
App.tsx put it in the entry bundle for every visitor.

Relatedly, **there is deliberately no `supabase` group in `advancedChunks`**.
Naming the group made rolldown fold the module-preload helper into that chunk,
and since the entry needs the helper, the entry statically imported all 202 KB.
Verify with `grep -o 'assets/[a-z-]*\.[A-Za-z0-9_-]*\.js' dist/index.html` —
supabase must not appear.

## The CV (`#cv`)
`public/Stepan_Roze_CV.pdf` is **gone**. It was maintained by hand and had
already drifted from the site — old E-Consulting end date, no Albron entry — so
the document a recruiter downloaded contradicted the page they downloaded it
from. It is replaced by a generated route:

- `src/app/data/experience.ts` — **the single source of truth for employment
  history.** Both the timeline and the CV render from it. Anything factual about
  a role goes here and nowhere else.
- `src/app/components/cv-print.tsx` + `src/styles/cv-print.css` — the document.
- `#cv` renders **outside every provider except language**: it must not inherit
  the dark theme. "Save as PDF" comes from the browser's own print dialog, so
  there is no PDF library in the bundle and no second file to keep in sync.

The visual design is a deliberate rebuild of the old PDF, measured out of it
(LibreOffice/Carlito): A4, 12.7mm margins, photo 75×90.7pt top-left, name 20pt
**#1F6FB2**, headings 12pt #1F6FB2, body 10.5pt #1A1A1A, labels #555. Keep it —
this is the look, not a starting point. `public/cv-photo.jpg` was extracted from
that PDF; note it is **not** the photo `about-section.tsx` shows.

Two traps, both already hit:
- `index.css` has a print block with a blanket `* { color: black !important }`
  to make the marketing page printable. It flattened the CV and threw away the
  blue. It is now scoped `body:not(:has(.cv-root)) *` — keep it that way.
- The site's generous `p { line-height }` leaked in and cost ~40% of the page
  height. `.cv-sheet p,li,dd,dt { line-height: inherit }` fixes it.

Length is **~1.7 A4 pages**, and that is correct: the document it replaces was
two pages, and two pages is normal for eight years of experience in Belgium. Do
not squeeze it to one.

## Open items (as of 2026-08-04)
Ordered by how much they matter, most first.

1. **Experience section is 3 464 px** for six entries — the largest block left
   on the page. Page total is 11 125 px desktop / 14 051 px mobile.
2. **`knowsLanguage` in `index.html` still lists Ukrainian.** Left deliberately:
   that is a language Stepan speaks, not a language the site is published in.
   Confirm with him before touching it.
3. **The About portrait is hosted on Google Drive**
   (`lh3.googleusercontent.com/d/...`). Confirmed 2026-08-04 as genuinely
   Stepan and loading fine at 1024×1024 — an earlier note in this file guessed
   it was stock, and that guess was wrong. What remains is hosting: a `/d/` URL
   can break on a permission change or rate limit, for reasons nobody here
   controls. It now falls back to `/cv-photo.jpg` (a real photo, lower
   resolution) instead of the Unsplash stranger it used to fall back to.
   Self-hosting the full-resolution original would close this properly.

---
> Source: [irozedev/my-portfolio](https://github.com/irozedev/my-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
