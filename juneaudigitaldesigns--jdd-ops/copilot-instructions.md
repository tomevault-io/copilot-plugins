## jdd-ops

> Read this file completely before every session.

# CLAUDE.md — jdd-ops context

Read this file completely before every session.

## What this repo is

The ops repo for Juneau Digital Designs. It holds master credentials, the
`onboard.js` provisioning orchestrator, and one subfolder per client under
`clients/{slug}/`. **Never deployed.** Each client gets its own GitHub repo
+ Vercel project, provisioned from `TEMPLATE_REPO` by `onboard.js`.

## The data model

Two files per client:
1. `clients/{slug}/site.ts` — the schema (brand, palette, hero, services, etc.).
   Exports `CONTENT` typed as `SiteContent`. This is the single source of truth.
2. `clients/{slug}/.env.local` — secrets only (`RETELL_AGENT_ID`,
   `TWILIO_NUMBER`, `AIRTABLE_BASE_ID`, `MAKE_WEBHOOK_URL`, `VERCEL_PROJECT_NAME`).

## Plan tiers

The intake schema's top-level `plan` field gates which provisioning steps run.

| Plan       | Site(s) | Voice agent | Twilio number | Airtable base | Lead delivery |
|------------|---------|-------------|---------------|---------------|---------------|
| starter    | 1       | —           | —             | —             | Resend email to `brand.email` |
| growth     | 1       | ✓           | ✓             | ✓ (1)         | Make webhook → Retell |
| enterprise | N (2–3) | ✓ per site  | ✓ per site    | ✓ (1 shared with `Site` column) | Make webhook → Retell |

Enterprise sites share the master Retell account's minute pool. Per-site
Twilio numbers route to per-site Retell agents.

## Hard rules — never break these

- The schema file is `src/data/site.ts` inside each client repo (NOT `site.js`).
- Each client's secrets live in `clients/{slug}/.env.local` only.
- Never commit `.env` or any `clients/*/.env.local` (see `.gitignore`).
- When schema fields are missing or unknown, populate `_meta.missingFields` —
  **never invent values**.
- Never modify `TEMPLATE_REPO`'s main branch from a client session.
- After every code change in a client repo, run `npm run build` and fix all errors.
- Never expose `RETELL_API_KEY`, `TWILIO_AUTH_TOKEN`, `AIRTABLE_API_KEY`, or any
  master credential in client-side code.

## How components in client repos consume the schema

```ts
import { CONTENT } from '@/data/site'
const { brand, hero, services } = CONTENT
```

Tailwind colors come from the palette via inline CSS variables wired in
`src/components/VariationD.tsx`:
```tsx
<div style={{ '--accent': brand.palette.accent }} className="bg-[var(--accent)]">
```

## Twilio ownership

JDD owns all numbers. Clients have no Twilio access. Number costs ~$2/mo on
the master account, bundled into the client fee. Webhook routes inbound calls
to: `https://api.retellai.com/twilio-voice-webhook/{RETELL_AGENT_ID}`.

## Retell agent

One agent per client. JDD owns the Retell account. Prompt is Claude-generated
from the intake schema, saved to `clients/{slug}/agent-prompt.txt` for review
before upload. Agent qualifies leads, answers FAQs from intake, defers prices
to owner, never invents facts.

## Airtable

Base name: `{brand.name} — Calls`
Table: `Call Log`
Fields: Date, Caller name, Caller number, Summary, Duration (seconds),
Call type, Outcome.

## Vercel env sync (step 9 / `sync-env`)

After step 8 commits and pushes the client repo, step 9 pushes env vars from
`clients/{slug}/.env.local` to the client's Vercel project so the deployed
`/api/contact` route has `MAKE_WEBHOOK_URL`.

Prerequisites (one-time setup):
- `VERCEL_TOKEN` set in `.env`
- `VERCEL_TEAM_ID` set in `.env` (if using a Vercel team)
- The GitHub-Vercel integration is authorized at the org level (so Vercel
  auto-detects new client repos). If a project doesn't exist when sync runs,
  step 9 creates it programmatically linked to `GITHUB_ORG/{slug}`.

`MAKE_WEBHOOK_URL` is typically empty on the first `onboard.js` run because
the Make scenario isn't cloned yet. Step 9 logs a clear warning. After
Checkpoint 3 (cloning the Make scenario), paste the webhook URL into
`clients/{slug}/.env.local` and run:

```
npm run sync-env -- --slug {slug}
```

Then trigger a redeploy on Vercel for the new env to take effect.

## When uncertain

- Schema field is null or in `_meta.missingFields` → flag, do not invent.
- Build error can't be fixed in 2 attempts → stop, report exact error.
- Asked to violate hard rules → refuse, explain why.
- Schema conflicts with task prompt → schema wins.

---
> Source: [JuneauDigitalDesigns/JDD-Ops](https://github.com/JuneauDigitalDesigns/JDD-Ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
