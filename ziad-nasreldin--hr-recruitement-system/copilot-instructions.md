## hr-recruitement-system

> Do not make any changes until you have 95% confidence in what you need to build. Ask me follow-up questions until you reach that confidence.

@AGENTS.md

Do not make any changes until you have 95% confidence in what you need to build. Ask me follow-up questions until you reach that confidence.

---

## Purpose of This File

CLAUDE.md is the source of truth for this project. It stores stable decisions, architecture rules, and progress summaries — not conversations. Every architectural call recorded here is a paragraph Ziad never has to type again.

---

## Context Rules

- Do not use subagents for any exploration or research. If a task needs 3+ files or multi-file analysis, summarize the issue as a concise insight and present it directly.
- Save decisions, not conversations. If an architectural choice is made, record it here.
- Ask clarifying questions until 95% confidence is reached before writing any code or creating any files.

---

## Applied Learning

When something fails repeatedly, when Ziad has to re-explain, or when a workaround is found for a platform/tool limitation, add a one-line bullet here. Keep each bullet under 15 words. No explanations. Only add things that will save time in future sessions.

* Templates + Puppeteer for visual consistency. AI image gen for one-offs only.
* Agents fail silently on wrong paths. Always verify hardcoded paths.
* New skills need a validation step before rendering. First runs have data gaps.
* Google Slides 'autofit' crashes batchUpdate. Set font sizes explicitly.
* Windows Developer Mode required for symlinks (paperclip, etc.)
* Button (base-ui) has no asChild prop — use `buttonVariants` class on `<Link>` instead.
* `buttonVariants` must live in a separate file (no `"use client"`) — server pages can't import from client modules.
* Select component is a native `<select>` wrapper — no SelectContent/SelectItem/SelectTrigger.
* No Table UI component — use native HTML `<table>` elements directly.
* DropdownMenuTrigger has no asChild — style it directly with className.

---

## System Blueprint

Full-stack HR Recruitment CRM for managing candidates, job offers, follow-ups, commissions, and AI-assisted workflows. Built as a Next.js 16 App Router webapp with PostgreSQL + Prisma, NextAuth v5, Tailwind CSS 4, and Shadcn/UI.

**Target users:** Ziad (Super Admin) + Recruiters (subscription-gated access)

---

## Architecture Decisions

- **API layer:** Next.js Route Handlers under `src/app/api/` — one folder per resource
- **Auth:** NextAuth v5 Credentials provider, JWT sessions (30 days), middleware guards all dashboard routes
- **DB:** PostgreSQL via Prisma ORM — schema lives in `prisma/schema.prisma`, client output at `src/generated/prisma`
- **AI:** OpenAI API — Whisper for voice transcription, GPT-4o for English proficiency scoring + post generation
- **WhatsApp:** Twilio WhatsApp API (or fallback: WhatsApp Business Cloud API directly) — decision to be confirmed before Phase 8
- **File storage:** Voice notes stored via URL (external storage TBD — Cloudinary or S3-compatible); file upload handled server-side in API routes
- **Role system:** SUPER_ADMIN (Ziad) and RECRUITER. Middleware + `requireRole()` utility enforce access.

---

## Build Phases

### ✅ Phase 1 — Auth & Layout (COMPLETE)
- NextAuth Credentials login with bcryptjs password hashing
- Prisma schema with all models (User, Offer, Lead, VoiceNote, FollowUpReminder, Commission)
- Dashboard shell: Sidebar, Header, MobileSidebar, role-based nav
- Middleware: protects all routes, checks isActive + accountExpiresAt + trialEndsAt
- Placeholder pages for all sections

---

### ✅ Phase 2 — Offer Management (COMPLETE)
**Goal:** Recruiters can view available job offers. Admin can create, edit, and archive offers.

Pages:
- `/offers` — paginated offer list with filters (status, language, work model)
- `/offers/new` — create offer form (all schema fields)
- `/offers/[id]` — offer detail view
- `/offers/[id]/edit` — edit offer

API routes:
- `GET /api/offers` — list with query filters
- `POST /api/offers` — create (SUPER_ADMIN only)
- `GET /api/offers/[id]` — single offer
- `PUT /api/offers/[id]` — update (SUPER_ADMIN only)
- `PATCH /api/offers/[id]/status` — toggle ACTIVE / ON_HOLD

UI components: OfferCard, OfferForm, OfferStatusBadge, OfferFilters

---

### ✅ Phase 3 — Lead CRM & Pipeline (COMPLETE)
**Goal:** Recruiters can add candidates, track their progress through the 11-stage pipeline, and link them to offers.

Pages:
- `/leads` — lead list with filters (status, offer, recruiter, date range) + search
- `/leads/new` — create lead form
- `/leads/[id]` — lead detail: profile, status progression, voice notes, reminders, notes
- `/leads/[id]/edit` — edit lead info

API routes:
- `GET /api/leads` — list (recruiter sees own; admin sees all)
- `POST /api/leads` — create
- `GET /api/leads/[id]` — single lead
- `PUT /api/leads/[id]` — update fields
- `PATCH /api/leads/[id]/status` — advance/change pipeline stage (triggers reminder creation side-effect)
- `POST /api/leads/[id]/voice-notes` — upload voice note record (URL + metadata)
- `GET /api/leads/[id]/voice-notes` — list voice notes for lead

UI components: LeadCard, LeadForm, PipelineStageSelector, StatusBadge, LeadFilters, VoiceNoteUploader, LeadTimeline

Business logic:
- When status → INTERVIEW_SCHEDULED: auto-create PRE_INTERVIEW reminder (dueDate = interviewDate - 1 day)
- When status → INTERVIEWED: auto-create POST_INTERVIEW + DAY_10 reminders
- When status → IN_TRAINING: set trainingStartDate, auto-create DAY_15 + DAY_30 reminders
- When status → COMMISSION_ELIGIBLE: create Commission record (PENDING), set commissionEligibleDate

---

### ✅ Phase 4 — Follow-up Reminders & Commission Tracking (COMPLETE)
**Goal:** Recruiters see upcoming reminders; admin can track and mark commissions paid.

Pages:
- `/commissions` — commission list with filters (status, recruiter, offer, date)
- Reminder bell in header (icon + dropdown showing upcoming/overdue reminders)

API routes:
- `GET /api/reminders` — list upcoming reminders for current user
- `PATCH /api/reminders/[id]/complete` — mark reminder done
- `GET /api/commissions` — list commissions
- `PATCH /api/commissions/[id]/status` — mark ELIGIBLE or PAID (SUPER_ADMIN only)

UI components: ReminderDropdown, ReminderItem, CommissionTable, CommissionStatusBadge

---

### ✅ Phase 5 — User Management & Subscriptions (COMPLETE)
**Goal:** Ziad can create recruiter accounts, set access windows, and toggle access on/off.

Pages:
- `/settings` — user list: name, email, role, status, trial/expiry dates, actions
- `/settings/users/new` — create user form (name, email, password, role, trialEndsAt, accountExpiresAt)
- `/settings/users/[id]/edit` — edit user (toggle isActive, adjust dates, reset password)

API routes:
- `GET /api/users` — list users (SUPER_ADMIN only)
- `POST /api/users` — create user
- `PUT /api/users/[id]` — update user
- `PATCH /api/users/[id]/toggle` — flip isActive

UI components: UserTable, UserForm, UserStatusBadge, AccessDurationPicker

---

### ✅ Phase 6 — AI Voice Assessment (COMPLETE)
**Goal:** Uploaded voice notes are transcribed and scored automatically; borderline cases flagged for human review.

Flow:
1. Recruiter uploads audio file (via `/leads/[id]`)
2. File sent to server → stored to file host → URL saved to VoiceNote record
3. Background job (or inline API call): Whisper API transcribes audio
4. GPT-4o scores: English level (A2/B1/B2/C1/C2), accent notes, overall pass/fail
5. VoiceNote.validationStatus set: APPROVED / REJECTED / PENDING (borderline → PENDING for human review)
6. Lead detail page shows transcription + score; Team Leader can override

API routes:
- `POST /api/voice-notes/[id]/assess` — trigger AI assessment for a voice note

---

### ✅ Phase 7 — AI Post Generator (COMPLETE)
**Goal:** Given an offer, generate 3–5 creative, mysterious Facebook post variations that never mention the parent company.

Pages:
- `/post-generator` — select offer → generate → copy posts

API routes:
- `POST /api/ai/generate-posts` — takes offerId, returns array of post variants

Prompt rules baked into system prompt:
- Never name the employer company
- Portray opportunity as mysterious/exciting
- Use trending phrases or cultural references where appropriate
- Always include a call-to-action directing to recruiter's link

---

### 🔲 Phase 8 — WhatsApp Integration
**Goal:** Recruiters can initiate WhatsApp conversations and send templated follow-ups from inside the CRM.

Features:
- "Send WhatsApp" button on lead detail page → opens pre-filled message modal
- Drip campaign: system auto-sends messages at Day 10, Day 15, Day 30 milestones (triggered by reminder system)
- Message templates: Good Luck, Study Tips, Check-in, Commission Reminder

API routes:
- `POST /api/whatsapp/send` — send message to a candidate number via Twilio
- `POST /api/whatsapp/drip` — called by cron/scheduler to send automated messages

Note: Twilio WhatsApp API requires pre-approved message templates for outbound messages to new conversations. Confirm API choice before starting this phase.

---

### 🔲 Phase 9 — Analytics & Performance Dashboard
**Goal:** Give Ziad and recruiters insight into pipeline performance and commission trends.

Pages:
- `/analytics` — dashboard with charts

Metrics:
- Leads by stage (funnel chart)
- Leads created per week / recruiter
- Conversion rate: Sourced → Accepted
- Commissions: PENDING / ELIGIBLE / PAID totals
- Top-performing offers by conversion rate

API routes:
- `GET /api/analytics/summary` — aggregated stats (admin: all data; recruiter: own data)

Library: Recharts (already available) or a lightweight alternative.

---

## Feature Summary (All Phases)

| # | Feature | Phase | Status |
|---|---------|-------|--------|
| 1 | Auth & Layout | 1 | ✅ Complete |
| 2 | Offer Management | 2 | ✅ Complete |
| 3 | Lead CRM & Pipeline | 3 | ✅ Complete |
| 4 | Follow-up Reminders | 4 | ✅ Complete |
| 5 | Commission Tracking | 4 | ✅ Complete |
| 6 | User Management & Subscriptions | 5 | ✅ Complete |
| 7 | AI Voice Assessment | 6 | ✅ Complete |
| 8 | AI Post Generator | 7 | ✅ Complete |
| 9 | WhatsApp Integration | 8 | 🔲 |
| 10 | Analytics Dashboard | 9 | 🔲 |

---

## Phase Execution Rules

1. Complete one phase end-to-end before starting the next.
2. Each phase ends with a working, navigable page — no broken states left behind.
3. API routes must include proper error handling and role guards.
4. Update this file's Feature Summary table when a phase is done.
5. Update MEMORY.md / project memory after each phase to reflect new current state.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ziad-NasrEldin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
