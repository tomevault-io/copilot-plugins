## incremento-admin

> Admin panel for incremento.co enquiry management — pipeline, contacts, analytics.

# Incremento Admin Dashboard

Admin panel for incremento.co enquiry management — pipeline, contacts, analytics.

## Stack
- **Frontend:** Plain HTML/CSS/JS in `public/` — deployed to Cloudflare Pages
- **Backend:** Cloudflare Worker at `enquiries.incremento.co`
- **Database:** Cloudflare D1 (`incremento-admin`)
- **Email:** Resend API
- **Auth:** Built-in email OTP login (6-digit code via Resend → 7-day HS256 JWT signed with ADMIN_KEY, sent as `Authorization: Bearer`). Allowed emails in `ADMIN_EMAILS` var in wrangler.toml. Legacy `X-Admin-Key` header still accepted. Auth endpoints: `POST /auth/request-code`, `POST /auth/verify`. OTP codes in D1 `otp_codes` table (10-min expiry). Admin API also routed same-origin at `admin.incremento.co/api/*`.

## Design System
- Background: `#0e1820` (dark midnight)
- Accent lime: `#9EF54A`, Accent cyan: `#5EE7D8`
- Fonts: Space Grotesk (headings) + Inter (body)
- Shared stylesheet: `public/admin.css`
- Shared JS: `public/admin.js`

## Pages
```
public/
  index.html      → redirect to dashboard
  dashboard.html  → KPIs, pipeline funnel, won revenue, service breakdown, monthly chart (analytics merged in)
  triage.html     → smart triage queue: pending inbound submissions, each with a suggested match (email>company>domain). Actions: new deal / merge into existing / not-a-deal (dismiss)
  inbox.html      → (hidden from nav; still reachable by URL)
  enquiry.html    → full-page enquiry view/edit (?id=N)
  pipeline.html   → stage view + quick stage update
  proposals.html  → proposals HUB: overview stats + tabs (Proposals list · Proposal Structure · Components · Email template · Branding). Proposal Structure = ordered sections of a proposal doc, each with editable shared_content (feeds every proposal) + guidance; add/remove/reorder; a section can't be deleted while components are attached (warns). Components (formerly "Component Library") now each carry a section_id mapping them into a structure section.
  proposal.html   → full-page proposal editor (?id=N edit, ?enquiry=N AI-draft, blank=new)
  contacts.html   → deduplicated contact list
  settings.html   → admin key, editable proposal email template, quick links
  brand.html      → brand assets: logos (light/dark previews + SVG/PNG download), colour swatches (click-copy), guidelines
  brand/          → logo + favicon source files + incremento-brand-pack.zip
  admin.css       → shared dark theme stylesheet
  admin.js        → shared JS (auth, API helpers, utilities)
  favicon.svg     → Incremento icon
```

## Pipeline Stages
`new` → `meeting_booked` (labelled "Call Booked") → `call_complete` ("Call Complete") → `proposal_sent` → `accepted` → `in_progress` → `on_hold` → `completed` ("Complete") / `cancelled`
(Canonical labels/colours/order live in STAGE_LABELS/STAGE_CSS/STAGE_COLORS in admin.js, mirrored in pipeline.html arrays + enquiry.html `<select>`. NB the `meeting_booked` KEY is retained — only its label changed to "Call Booked" — so existing rows didn't need migrating. `completed` is still the won-revenue key.)

## Worker API (all require X-Admin-Key header)
- `POST /`                  → form submission (no auth required, public)
- `GET /api/enquiries`      → list all (supports ?limit=&offset=&stage=)
- `GET /api/enquiries/:id`  → single enquiry
- `PATCH /api/enquiries/:id`→ update stage/notes/value/project_title/company/company_url
- `DELETE /api/enquiries/:id`→ delete a deal + cascade (contacts, deal_assignees, messages, proposals via enquiry_id). UI: trash button per pipeline row + "Delete deal" button on enquiry.html
- `GET /api/stats`          → dashboard KPIs (total, new_count, stage_counts, potential_revenue)
- `POST /api/proposals/draft` → AI-draft proposal from enquiry (body: {enquiry_id, components?[]}; needs ANTHROPIC_API_KEY secret). New flow: `proposal.html?enquiry=N` shows a **scoping step** first — Robin picks library components in the picker ('draft' mode), then Claude assembles the proposal AROUND them (each component → tailored scope workstream + pricing line); "draft from enquiry alone" skips the picker. System prompt enforces no-em-dash.
- `GET/POST /api/proposals` (`?archived=1` lists archived; default excludes them) + `GET/PATCH/DELETE /api/proposals/:id` → proposal CRUD. **Archive** is soft (column `archived` 0/1 on proposals, set via PATCH {archived}); **Delete** is hard (cascade). Both surfaced on proposals.html rows (+ "Show archived" toggle with Restore) and on the proposal.html editor header.
- `POST /api/proposals/:id/send` → email proposal link to client via Resend
- `GET /p/:token` (public) → branded proposal page (records view). `?preview=1` = admin preview (works for drafts, banner, hides Accept, no view recorded). `?pdf=1` = auto-download. Download PDF button uses html2pdf to keep the dark theme; @media print also preserves dark colors.
- `POST /p/:token/accept` (public) → client accepts; emails Robin; enquiry stage → accepted; logs inbound message
- `GET /api/companies` → distinct company names (+ representative company_url) for the deal company picker
- `GET/POST /api/proposal-blocks` + `PATCH/DELETE /api/proposal-blocks/:id` → component library. Table `proposal_blocks` (kind, **service**, **section_id**, title, body, price, extra). `section_id` maps a component to a `proposal_sections` row.
- `GET/POST /api/proposal-sections` + `PATCH/DELETE /api/proposal-sections/:id` → proposal structure. Table `proposal_sections` (position, title, description, shared_content, created_at). DELETE returns 409 `{error:'has_components'}` if any component still references the section (UI warns + blocks the delete). Seeded with 7 default sections (worker/migrate-sections.sql). The library (Proposals hub → Component Library tab) is grouped by **service/speciality** (`service` ∈ web-design, web-development, seo, ai, analytics, infrastructure, email, ecommerce, ux, paid, general). Each row is a reusable client-facing component (e.g. seo→"On-page optimisation"). Seeded from `worker/seed-library.sql` (~58 components, on-brand, incl. the "free hosting up to 100k hits/day" line; AI components mirror the how-to-use-ai-in-web-projects blog). SERVICES list lives in proposals.html.
- `POST /api/send-email` → send a branded email to a contact from a chosen incremento.co address (signature varies by sender), logged outbound
- `GET /api/messages?email=` or `?enquiry_id=` → communication timeline. `enquiry_id` spans the whole deal (enquiry email + every contact on it); merges enquiries + logged messages
- `POST /api/messages` → manually log a message (direction, subject, body)
- `GET/PATCH /api/settings` → key/value settings. Email template is **per language**: keys `proposal_email_{greeting,intro,closing,signature}_{en,pt}` (tokens {name}=first name, {title}, {company}). Defaults in worker `EMAIL_TEMPLATE_DEFAULTS.{en,pt}` (also hold the localized `subject`). Edited in Proposals → Email with an EN/PT toggle (saves both).

## Communication language (en | pt)
Each **deal** (`enquiries.language`) and **proposal** (`proposals.language`) carries a communication language, default `en`. It flows through everything: the AI draft is written entirely in that language (`draftProposal` passes it to the system prompt + returns it), the proposal email picks the matching template set + localized subject, and the public proposal page renders its chrome via `PROP_LABELS[lang]` (`<html lang>`, headings, Why-Incremento copy, accept form, dates, JS strings). Set it with the EN/PT toggle on the deal page (auto-saves), the proposal editor (saved on persist, seeded from the deal/draft), or the send page (switches + re-renders the preview). Migration: worker/migrate-language.sql.

## Communication log (messages table)
Rendered newest-first in `mountComms`.

## Triage (inbound form submissions)
Form submissions are NOT auto-deals. `handleEnquiry` (POST /) inserts the enquiry with `triage_state='pending'` (column on `enquiries`: `pending`/`deal`/`dismissed`; NULL=legacy deal) and creates NO contact yet. Robin sorts them on `triage.html`:
- `GET /api/triage` → pending items, each with a suggested `match` against existing deals, ranked email > company > domain (free-mail domains in FREE_MAIL are ignored for domain matching).
- `POST /api/triage/:id/accept` → `triage_state='deal'` + creates primary contact (enters pipeline).
- `POST /api/triage/:id/merge` {target_enquiry_id} → adds sender as a contact on the target deal + logs the message into its comms, then marks the pending item `dismissed`.
- `POST /api/triage/:id/dismiss` → `triage_state='dismissed'` (sales/spam/not-a-deal).
`listEnquiries`, `getStats` only count `(triage_state IS NULL OR ='deal')`. `getStats` also returns `triage_count` (drives the sidebar Triage badge via `loadNavBadge`).
Auto-logged: enquiry (inbound), proposal email (outbound), acceptance (inbound). Manual: "Log a message" composer on enquiry.html + proposal.html (shared `mountComms()` in admin.js). Keyed by contact_email. Inbound capture: `POST /api/inbound-email?token=INBOUND_SECRET` (Mailgun webhook, multipart/form-data) — same pattern as lst-pp-admin. Detects direction (INC_SENDERS → outbound), matches contact by email, dedupes, logs kind='mail'. Robin BCCs log@mg.incremento.co / routes replies there via Mailgun. INBOUND_SECRET is a wrangler secret.

## Proposal statuses
`draft` → `sent` → `viewed` → `accepted` / `declined`
Content JSON: intro, scope[{title,description}], deliverables[], timeline[{phase,duration,description}], pricing[{item,description,price}], terms.
AI drafting calls the Claude API (claude-opus-4-8, structured JSON output) directly from the worker.

## Setup (first time)
```bash
# 1. Create D1 database
wrangler d1 create incremento-admin
# Copy the database_id into wrangler.toml

# 2. Run schema
wrangler d1 execute incremento-admin --file worker/schema.sql --remote

# 3. Set secrets
wrangler secret put RESEND_API_KEY
wrangler secret put ADMIN_KEY

# 4. Deploy worker
wrangler deploy

# 5. Deploy admin pages
wrangler pages deploy ./public --project-name incremento-admin --branch y --commit-dirty=true
```

## Deploy
```bash
wrangler deploy                                                              # worker
wrangler pages deploy ./public --project-name incremento-admin --branch y --commit-dirty=true  # pages (production branch is literally named "y" — without --branch y the deploy lands in Preview and admin.incremento.co won't update)
```

## Related
- `incremento-site` — the public site that submits to this worker
- `lst-pp-admin` — LST/PP admin (reference implementation)

## Admin URL
Set up `admin.incremento.co` as custom domain for the Pages project in Cloudflare dashboard.
Add `https://admin.incremento.co` to `ADMIN_ORIGINS` array in `worker/index.js` after setup.

---
> Source: [robskils/incremento-admin](https://github.com/robskils/incremento-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
