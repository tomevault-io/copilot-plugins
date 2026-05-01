## hndigest

> AI-powered Hacker News newsletter builder. Users chat with an AI to design their newsletter (sections, colors, fonts, schedule), preview it live with real HN data, then activate it with a magic link. Digests are sent on a per-user QStash schedule using each user's own Resend API key.

# Project: HN Digest

AI-powered Hacker News newsletter builder. Users chat with an AI to design their newsletter (sections, colors, fonts, schedule), preview it live with real HN data, then activate it with a magic link. Digests are sent on a per-user QStash schedule using each user's own Resend API key.

## Stack

- **Next.js** (App Router) — read `node_modules/next/dist/docs/` before writing route or middleware code. This version may have breaking changes from your training data.
- **CopilotKit** — AI agent in the editor sidebar. Route at `app/api/copilotkit/route.ts`.
- **Resend + React Email** — transactional email. Templates in `emails/`.
- **Neon (Postgres)** — via `@neondatabase/serverless`. All DB logic in `lib/db.ts`.
- **Upstash QStash** — per-user HTTP scheduling. Logic in `lib/qstash.ts`.
- **Tailwind CSS v4** — no `tailwind.config`. Colors defined as CSS variables in `app/globals.css` under `@theme inline`. Use semantic names (`text-accent`, `bg-card`, `border-border`) — never hardcode hex.

## Key files

| File | Purpose |
|---|---|
| `lib/db.ts` | All Neon DB queries — schedules, test_sends table |
| `lib/qstash.ts` | QStash schedule create/cancel, cron conversion |
| `lib/auth.ts` | JWT magic tokens, session cookies |
| `lib/encrypt.ts` | AES-256-GCM encrypt/decrypt for Resend keys |
| `lib/types.ts` | Shared TypeScript types |
| `lib/section-meta.ts` | Section type metadata (label, icon, color, structural flag) |
| `lib/timezones.ts` | IANA timezone list + local↔UTC conversion utils |
| `app/api/send/route.tsx` | QStash callback — fetches schedule, sends digest |
| `app/api/send-test/route.tsx` | Test send during activation — creates draft record |
| `app/api/activate/route.ts` | Activates schedule, creates QStash job, sends magic link |
| `app/api/schedule/route.ts` | CRUD for schedule updates from dashboard |
| `app/api/unsubscribe/route.ts` | Cancels QStash + marks inactive |
| `app/api/auth/` | Magic link send + verify + session |
| `components/dashboard-view.tsx` | User dashboard — manage schedule, recipients, send history |
| `components/newsletter-editor.tsx` | AI editor with CopilotKit sidebar + live preview |
| `components/activate-modal.tsx` | 3-step activation flow (key → email → sent) |
| `app/page.tsx` | Landing page |
| `emails/newsletter.tsx` | React Email newsletter template |

## Architecture decisions

- **Per-user Resend keys**: Every user enters their own Resend API key at activation. It's encrypted with AES-256-GCM (`lib/encrypt.ts`) before DB storage. At send time, decrypted in memory only. Magic link emails also go through the user's key — the app's `RESEND_API_KEY` is only a fallback.
- **QStash for scheduling**: Vercel Hobby cron is limited to once/day. QStash allows per-user exact-time HTTP callbacks. Each activation creates a QStash schedule pointing to `/api/send?id=<scheduleId>` protected by `CRON_SECRET`.
- **Draft schedule records**: When a user sends a test email (`/api/send-test`), a draft record is created in the DB with `owner_email = "draft-<uuid>"`. On activation, the real email is set and the record goes active. Tracks test attempts in the `test_sends` table.
- **Stateless auth**: Magic link = signed JWT (30m expiry). Session = signed JWT in httpOnly cookie (30d). No server-side session store needed.
- **Section structural flag**: `SECTION_META[type].structural = true` for heading/divider/footer/intro — these don't count toward the section count shown in the dashboard.

## Patterns to follow

- All colors via CSS variables — never `text-[#hex]`. Add new colors to `:root` + `@theme inline` in `globals.css`.
- DB migrations via `ALTER TABLE IF NOT EXISTS` in `ensureTables()` — called before every query.
- QStash schedule updates = cancel old + create new (no update API).
- Never log decrypted keys. Never return encrypted keys to the browser.
- Rate limit magic link requests (in-memory Map, 1hr cooldown).

---
> Source: [Anmol-Baranwal/hndigest](https://github.com/Anmol-Baranwal/hndigest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
