## command-center

> Private, single-user personal operations command center for Paul Pacardo. Complements the firm's Asana as their system of record; never replaces it. Full context lives in docs/Command_Center_Build_Plan.md and docs/Command_Center_Architecture.md. Read both before large tasks. When they agree, follow them. When something is missing or unclear, ask Paul rather than guessing.

# Command Center

Private, single-user personal operations command center for Paul Pacardo. Complements the firm's Asana as their system of record; never replaces it. Full context lives in docs/Command_Center_Build_Plan.md and docs/Command_Center_Architecture.md. Read both before large tasks. When they agree, follow them. When something is missing or unclear, ask Paul rather than guessing.

## How to work with Paul
- Direct and terse. Decisive recommendations, not menus of options. Push back when he is wrong.
- Never use em dashes. Not in chat, not in code comments, not in UI copy, not in commit messages.
- Do not wind the session down or suggest stopping points while there is a next thing to build. Keep going until Paul calls the stop.
- Show working software early. Build one module end to end before spreading wide.
- When you need something only Paul can provide (tokens, domain, account access), ask for exactly that and keep moving on everything else.

## Locked decisions (do not relitigate unless Paul raises it)
- Cloudflare everything: one Worker with Static Assets serving both the front end and the Hono API, D1 (database), R2 (files), KV (sessions and settings), Cron Triggers (digests), Resend (email). Not Pages. See D29 in docs/DECISIONS.md.
- Front end: SvelteKit with adapter-cloudflare targeting Workers. Boring, standard dependencies. No Next.js, no Astro.
- Auth: Cloudflare Access with One-Time PIN on the custom domain, whitelisting Paul's email only. Worker-based hashed-password auth is a documented fallback, not the build.
- Asana: one-way push in v1, store the Asana task gid on each action item. No two-way sync until v2.
- Single user. Read-only shareable reports come later.
- Free tier with a spend limit. Workers Paid only when Paul decides.

## Build order
1. Stage 1: DONE. Worker + Hono + D1 + KV, work.kabuhayan.app, Access OTP, Action Items module end to end. Paul logs in by emailed PIN and CRUDs action items persisting in D1.
2. MVP: Today cockpit, Projects (five PMI phases), SOP library with version history, Invoicing with aging, start-of-day and end-of-day digests via Cron + Resend.
3. v1: Meetings (transcript import, AI summary, action item extraction), Templates with AI drafting, Clients, Reports with PDF export, one-way Asana push.
4. v2: Partner-hours-saved dashboard, shareable read-only report links, two-way Asana sync, nightly D1 to R2 backups.

## Deployment
- Git-connected Workers Builds is the deploy model: commit, push, Cloudflare builds and deploys. Direct `wrangler deploy` only for quick throwaway tests.
- Schema changes go through D1 migrations only, applied with `npx wrangler d1 migrations apply command-center-db --remote`. Never hand-edit the live database.
- Secrets (ASANA_TOKEN, RESEND_API_KEY, AI keys) via `wrangler secret put` or the Worker's settings in the dashboard. Never in code, never in wrangler.toml.
- Ask Paul before running any command that deploys, applies remote migrations, or touches the Cloudflare account.
- The app is reachable only at work.kabuhayan.app, behind Cloudflare Access. `workers_dev` and `preview_urls` are both false and stay that way. Turning either on puts an unauthenticated copy of the app on the public internet.

## Conventions
- Data model, module specs, and site map are in docs/Command_Center_Architecture.md section E and B. Follow them.
- Design tokens: navy #102A4C, gold #C9A84C (accent only), cream #FAF6EC, ink #1B2433, muted #5B6470, green #2E7D5B. Fonts DM Sans and DM Mono. Clean, calm, low noise.
- UI copy: plain, clean, no em dashes.
- Wire relationships, not just screens. Every record links to its related records.
- Cron Triggers are UTC only. Convert Mountain Time and account for DST. Keep digests idempotent.

## What only Paul can provide (ask up front, all at once, when first needed)
Cloudflare account and wrangler login, a domain, GitHub repo access, the OTP whitelist email, Asana Personal Access Token, Resend API key, AI provider API key.

Added 2026-08-29, D61: a scoped Cloudflare API token with **Workers Observability Read** and nothing broader. Wrangler's own OAuth token cannot read Workers Logs, which is how a cron incident nearly closed on outcome evidence alone. Deliver it with `wrangler secret put`, never in chat.

---
> Source: [pacardopaul18/command-center](https://github.com/pacardopaul18/command-center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
