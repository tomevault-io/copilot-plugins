## cnnews

> Daily AI-generated news digest emails, powered by Cloudflare Workers + Durable Objects.

# Nauseam News

Daily AI-generated news digest emails, powered by Cloudflare Workers + Durable Objects.

## Architecture

- **Worker** (`src/index.ts`): Stateless HTTP router, serves HTML pages, proxies API calls to Durable Objects
- **Durable Object** (`src/digest-object.ts`): Per-user stateful object storing email, topics, digest history. Uses DO alarms for daily scheduling at 8am UTC
- **Anthropic** (`src/anthropic.ts`): Claude claude-opus-4-6 with `web_search` + `web_fetch` tools (beta) to research topics and generate digests
- **Resend** (`src/resend.ts`): Email delivery via Resend API, from address `news@news.chadnauseam.com`
- **Frontend**: React + Tailwind - landing page + per-user dashboard with Yjs real-time sync

Deploy with `pnpm run deploy`.

## Key Patterns

- SQLite-backed DO with Yjs (y-partyserver) for real-time state sync to clients
- `idFromName(uuid)` maps user UUIDs to DOs - no auth, UUID unguessability is sufficient
- DO alarms (not cron) for self-scheduled daily digests
- `pause_turn` handling loop for Claude web search/fetch reliability
- Digest history capped at 30, last 3 passed as context to avoid repetition
- **Email confirmation**: First digest sends a confirmation email with an unguessable token (`/confirm/:uuid/:token`). Token is stored in DO KV storage only (never in Yjs doc). Future digests are blocked until confirmed. Prevents spam abuse.

## Security: Yjs doc is client-writable

The Yjs document is bidirectionally synced — clients can write to any field. **Never use Yjs as the source of truth for security-sensitive state.** Any data that gates access, controls email sending, or has security implications must be stored in DO KV storage (`this.ctx.storage`) and enforced server-side.

Currently server-authoritative fields (stored in KV, enforced via `enforceServerFields()`):
- `confirmed` / `confirmedAt` — email verification status
- `confirmToken` — verification token (KV-only, never in Yjs)

Client-writable fields (safe in Yjs):
- `email`, `enabled`, `frequency`, `timezone` — user preferences
- `topics` array — user's chosen topics
- `nextAlarmTime` — display-only, actual alarms use DO storage

If adding new security-sensitive fields, follow the `confirmed` pattern: store in KV, cache in an instance var, sync to Yjs for display, and revert unauthorized client mutations in `enforceServerFields()`.

## Commands

```bash
pnpm dev          # Local dev server (wrangler dev)
pnpm deploy       # Deploy to Cloudflare (builds CSS + JS first)
```

## Secrets

Local: `.dev.vars` file (gitignored)
Production: `wrangler secret put ANTHROPIC_API_KEY` / `wrangler secret put RESEND_API_KEY`

## Project Structure

```
src/
  index.ts           # Worker entry + routing
  digest-object.ts   # Durable Object class (YServer)
  anthropic.ts       # Claude web search/fetch + digest generation
  resend.ts          # Email sending + confirmation emails
  types.ts           # Shared types
  app.tsx            # React dashboard UI
  client.tsx         # Client entry point
  styles.css         # Tailwind source styles
  components/ui/     # UI components (button, card, input, etc.)
  html/
    landing.ts       # Landing page template
    dashboard.ts     # Dashboard page template
public/
  style.css          # Built CSS (generated)
  client.js          # Built JS (generated)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anchpop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
