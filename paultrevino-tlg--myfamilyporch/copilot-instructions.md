## myfamilyporch

> > **On startup: read `TODO.md` and continue from the first unchecked task.**

# My Family Porch — Claude Code project brief

> **On startup: read `TODO.md` and continue from the first unchecked task.**
> Check tasks off as you finish them and keep the file current. Prefer small,
> reviewable commits that map to TODO items. Each TODO phase names the exact
> `docs/SPEC.md` section(s) and prototype to read before building — read those,
> not the whole spec, when you start a task.

## What this is
My Family Porch records a family elder's life stories as a short, AI-guided **voice
interview** in a family member's cloned voice, and turns them into a keepsake.
Multi-tenant: many unrelated families, each with multiple storytellers across
generations (parent, grandparent, aunt/uncle).

## References (read on demand, not by default)
- `docs/SPEC.md` — full product decisions, by section (TODO points you to the right one)
- `docs/prototypes/storyteller-flow.html` — elder's 7-screen session (EN/ES)
- `docs/prototypes/family-admin.html` — family/admin dashboard (3 signals, stories, topics, schedule, book, settings)
- `docs/ARCHITECTURE.md` — tenancy, auth, prompts, signals

## Stack
- Next.js 15 (App Router, TS, `src/`) on **Cloudflare Workers** via `@opennextjs/cloudflare`
- **Supabase** (Postgres + RLS + Storage). The Supabase MCP is attached — use it
  to apply migrations, run SQL, regenerate types. Don't hand-edit prod.
- shadcn/ui + Tailwind; Stripe + Resend later
- Anthropic API = interview brain (server-side only); ElevenLabs (TTS),
  Whisper/Deepgram (STT), Twilio (SMS)
- Deploy: GitHub → Cloudflare (Workers Builds / Git integration), or the included Action

## Repo map
- `src/app/(marketing)` — public site
- `src/app/(app)` — authenticated family/admin dashboard
- `src/app/(storyteller)/s/[token]` — token-scoped storyteller flow (no login)
- `src/app/api/ai/*` — all Anthropic calls (server-side only)
- `src/app/api/sessions` — session lifecycle (admin/cron)
- `src/app/api/storyteller/*` — storyteller writes; validates magic-link token, service role
- `src/app/api/cron/scheduler` — weekly Worker: nudges + the 3 signals
- `src/lib/supabase/{client,server,service}.ts` — anon / SSR-member / service-role
- `src/lib/{ai,sms,voice,i18n}` — interview brain, Twilio, TTS, UI strings
- `supabase/migrations/0001_init.sql` — multi-tenant schema + RLS
- `supabase/seed/*` — bilingual prompt library (en + es), seeded as global rows

## Non-negotiable rules (always apply)
- **Tenancy:** every domain row carries `family_id`. RLS is the security boundary —
  don't bypass it in app code. Members of family A must never read family B.
- **Two auth surfaces:** family members → Supabase Auth + `memberships`.
  Storytellers → signed magic-link tokens, never Supabase Auth; their writes go
  **only** through `api/storyteller/*` using the service role.
- **Service role key is server-only.** Never import `lib/supabase/service` into a
  client component or expose the key to the browser.
- **All Anthropic calls server-side** via `api/ai/*`. No API key in the client.
- **Sensitive media:** Storage buckets private; serve audio via short-lived signed
  URLs only after the same membership check.
- **Elder-facing UX:** storyteller surface stays large-target, single-tap,
  voice-first, forgiving. No scolding timeouts; never dead-end on an error.

## Commands
- `npm run dev` — local Next dev
- `npm run preview` — build + Cloudflare runtime locally
- `npm run deploy` — build + `wrangler deploy`
- `npm run db:push` — apply migrations (or use the Supabase MCP)
- `npm run db:types` — regenerate `src/lib/supabase/database.types.ts`
- `npm run seed:prompts` — load the bilingual global library

## Definition of done for a task
Type-checks; respects the non-negotiable rules; correct `family_id` scoping;
matches the SPEC section + prototype the task points to; `TODO.md` updated.

---
> Source: [paultrevino-tlg/myfamilyporch](https://github.com/paultrevino-tlg/myfamilyporch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
