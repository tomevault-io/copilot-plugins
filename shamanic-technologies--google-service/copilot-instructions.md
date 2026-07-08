## google-service

> Google Ads API v23 wrapper for MCC agency management, plus Google CRM bronze→silver ingestion (Gmail + People readonly) feeding the dashboard CRM at `/orgs/{orgId}/services/crm`.

# google-service

Google Ads API v23 wrapper for MCC agency management, plus Google CRM bronze→silver ingestion (Gmail + People readonly) feeding the dashboard CRM at `/orgs/{orgId}/services/crm`.

## Identity

All endpoints require `x-org-id` and `x-user-id` headers (UUIDs from client-service).
These are the internal org/user identifiers — never use Clerk IDs (clerkOrgId/clerkUserId).
The client-service is the source of truth for identity resolution.

### Tracking / cost-attribution headers

Inbound `x-run-id` (required), `x-feature-slug`, `x-brand-id`, **`x-audience-id`** (all optional) are the tracking block. They are read in `requireIdentityHeaders` (`src/middleware/validate.ts`) onto `req`, then forwarded to every **internal** sibling call (runs-service, billing-service, key-service) via the `trackingHeaders()` allowlist builder in `src/lib/tracking-headers.ts` — never cherry-picked per field. `x-audience-id` (the campaign's priority audience) tags `runs.audience_id` on `createRun` and the cost row on `addCosts`, which is how per-audience cost attribution works (`COALESCE(runs_costs.audience_id, runs.audience_id)` in runs-service). The only metered cost on the campaign path is `serper-dev-query` (`/search/*`). **Egress safety**: `trackingHeaders()` is imported ONLY by the internal clients; external vendor calls (Serper, Gmail/People, Google Ads, Google OAuth) build their own provider-auth headers and MUST never receive the tracking block.

## Stack

See global CLAUDE.md for shared stack details (TypeScript strict, Zod, Vitest+Supertest, Railway).

**Package manager: npm.** Lockfile is `package-lock.json`; the Dockerfile runs `npm ci`. Use `npm install` / `npm test` / `npm run build` locally. Do NOT run `pnpm install` here — it creates a stray `pnpm-lock.yaml` that diverges from the lockfile Railway actually reads.

## OAuth client credentials

The Google OAuth client (Client ID + Secret) is the **same** for the Google Ads Developer Console and the Gmail/People consent flow — one Google Cloud project, one OAuth client. It is registered as platform keys `google-client-id` / `google-client-secret` by the dashboard (`apps/dashboard/src/instrumentation.ts`), not by this service.

Business logic must call `getGoogleOAuthClient()` in `src/services/key-service.ts` to fetch the OAuth client at runtime; never read `GOOGLE_*` env vars directly. If `getGoogleOAuthClient()` returns 404, the dashboard side has not yet registered the providers — fix it there, not here.

## Migrations

`src/db/migrate.ts` exports `runMigrations()` which is awaited from `src/index.ts` **before** `app.listen()`. Every Railway deploy runs the migration; missing tables block startup so a bad migration triggers Railway's restart loop loudly instead of serving 500s.

Schema changes: edit the inline `migration` SQL in `src/db/migrate.ts`. Use `CREATE TABLE IF NOT EXISTS` / `DO $$ ... IF NOT EXISTS ... END $$` so the same migration runs cleanly on every boot.

Manual one-off run still available via `pnpm migrate`, which runs `src/db/migrate-cli.ts`. The CLI runner lives in a **separate file** from `migrate.ts` because `esbuild --bundle --format=cjs` inlines every imported file into `dist/index.js`, and at runtime `require.main === module` evaluates **true** for the bundled entry — so a CLI guard inside `migrate.ts` would fire at boot and call `pool.end()` after migrations, crashing every subsequent request with `Cannot use a pool after calling end on the pool`. Reference: hotfix v0.19.1.

## Data layering

This service owns **bronze** and **silver** for Google CRM data. Gold is served as an additive read projection over bronze+silver (no separate gold table yet).

**Silver tables** (`google_contacts_silver`, `gmail_messages_silver`) are typed projections of the bronze `*_raw` payloads, keyed on the SAME natural key as their bronze source (`(org_id, resource_name)` / `(org_id, gmail_message_id)`). They are:
- **Populated at ingest** — `people-ingest`/`gmail-ingest` parse the payload via `src/services/silver.ts` (`parseContactSilver` / `parseMessageSilver`) and upsert silver right after the bronze upsert (only on inserted/updated bronze rows; deletes cascade to silver via `deleteContactSilver`).
- **Backfilled from bronze on boot** — `src/services/backfill-silver.ts` runs AFTER `app.listen()` (never in the boot window) with `.catch(console.error)`, keyset-paginated + idempotent (upsert), so re-runs and silver schema changes are safe without re-fetching Google.

Bronze stays the source of truth; silver is a rebuildable view. Contact silver columns: `display_name, primary_email, emails[], phones[], organization, job_title, photo_url, updated_at, deleted`. Message silver columns: `from_email, from_name, to_emails[], subject, snippet, sent_at, labels[], history_id`.

### Read endpoints are ADDITIVE (gold)

`GET /orgs/google/messages` and `/contacts` LEFT JOIN silver onto bronze and return the typed fields **alongside** every legacy field (incl `payload`) — the change is non-breaking, so it ships to prod before the dashboard consumer switches. Locked byte-equal contract with distribute.you admin — do NOT rename: messages add `fromEmail, fromName, to[], subject, snippet, sentAt, labels[]`; contacts add `displayName, primaryEmail, emails[], phones[], organization, jobTitle, photoUrl, updatedAt, deleted`. Messages sorted `sent_at` desc (fallback `fetched_at`); contacts deduped by `primary_email` (rows without an email key on `resource_name`, never dropped).

### Cron auto-sync

`src/services/cron-sync.ts` `startAutoSync()` (scheduled from `index.ts` after listen) runs `syncOrg` for every distinct org in `google_oauth_tokens` every `GOOGLE_SYNC_INTERVAL_HOURS` (default 6). First tick fires after one interval (no boot-storm). Low-frequency by design so Neon scale-to-zero stays effective — no high-frequency DB-ping loop; a fresh pool connection per query. Google People/Gmail calls use the user's own OAuth token (no metered platform cost → no cost declaration). `src/services/sync.ts` `syncOrg` is the shared core used by both the async HTTP sync job and the cron.

### Bronze tables

| Table | Natural key | Source | Notes |
|-------|-------------|--------|-------|
| `google_oauth_pending` | `(org_id, state)` | OAuth start | 10 min TTL, stores PKCE verifier |
| `google_oauth_tokens` | `(org_id, google_account_email)` | OAuth callback | One row per (org, Gmail account). Stores refresh token, last access token, `gmail_history_id`, `people_sync_token`, `other_contacts_sync_token` |
| `gmail_messages_raw` | `(org_id, gmail_message_id)` | Gmail `messages.get format=full` | Full JSON payload in `payload jsonb` |
| `google_contacts_raw` | `(org_id, resource_name)` | People `connections.list` AND `otherContacts.list` | Full JSON payload in `payload jsonb`. `resource_name` namespace distinguishes sources: `people/c...` = address book, `otherContacts/c...` = Gmail-collected. |
| `google_sync_jobs` | `id` (UUID) | `POST /orgs/google/sync` | One row per sync request. `status` ∈ `running` \| `succeeded` \| `failed`; `summary` jsonb on success, `error` text on failure. Org-scoped lookups (`WHERE org_id = $1 AND id = $2`). |
| `google_contact_links` | `(org_id, resource_name)` | `PUT /orgs/google/contact-links` | Per-contact CRM tagging (NOT bronze — app state). `linked_org_ids`/`linked_brand_ids`/`linked_feature_slugs` TEXT[] default `'{}'`, `status` TEXT NULL (reserved). LEFT-JOINed onto `GET /orgs/google/contacts` as `links{}`; a contact with no row returns empty arrays + null status. `resource_name` lives in the request BODY, never the path (Google resourceNames contain `/`). |

All bronze tables (and `google_sync_jobs`, `google_contact_links`) are `org_id`-scoped. Every SQL query in `/orgs/google/*` includes `WHERE org_id = $N`.

### Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/orgs/google/auth/start` | Build authorize URL (PKCE), persist pending state |
| `GET` | `/orgs/google/auth/callback` | Exchange code, store tokens. Browser callback is proxied by the dashboard server-side so identity headers are present. |
| `POST` | `/orgs/google/sync` | Start an async sync. Inserts a `google_sync_jobs` row, fires ingest in a detached promise, returns `202 {jobId, status:"running"}` immediately. Backfill on first run (last `GOOGLE_GMAIL_BACKFILL_DAYS` for Gmail), delta thereafter (Gmail `historyId`, People `syncToken`). Fan-out per connected Google account. |
| `GET` | `/orgs/google/sync/{jobId}` | Poll job status. Returns `{jobId, status, summary, error, startedAt, finishedAt}`. Org-scoped: 404 if `jobId` belongs to another org. |
| `GET` | `/orgs/google/messages` | Cursor-paginated Gmail messages: bronze payload + typed silver fields, ordered by silver `sent_at` desc (fallback `fetched_at`). Optional `?participant=<email>` filters to one contact's thread (From/To/Cc participant via `payload::text ILIKE`), ordered by the message's own email date (`internalDate`) newest-first. |
| `GET` | `/orgs/google/contacts` | Cursor-paginated Google contacts: bronze payload + typed silver fields, deduped by `primary_email` (text `query` matches `payload::text ILIKE`). Each item also carries `links{orgIds,brandIds,featureSlugs,status}` from `google_contact_links` (LEFT JOIN, unconditional). |
| `PUT` | `/orgs/google/contact-links` | Upsert per-contact links on `(org, resourceName)`. Body `{resourceName, orgIds, brandIds, featureSlugs, status?}`; `resourceName` in the BODY (never path). Returns the persisted `{resourceName, orgIds, brandIds, featureSlugs, status}`. |

### Idempotency strategy: upsert-when-different

Sync re-runs produce no duplicate rows because each bronze table has a `UNIQUE` constraint on its natural key:

- `gmail_messages_raw`: `ON CONFLICT (org_id, gmail_message_id) DO UPDATE … WHERE history_id IS DISTINCT FROM EXCLUDED.history_id` — re-fetched messages with the same `historyId` are no-ops; mutations bump `payload` and `fetched_at`.
- `google_contacts_raw`: same pattern keyed on `etag`.

Append-only is preserved in spirit: we never mutate audit metadata; we only refresh `payload + fetched_at` when the upstream artefact changes.

### Sync model: fire-and-forget + status table

`POST /orgs/google/sync` is async. The handler:

1. Inserts a `google_sync_jobs` row with `status='running'`.
2. Calls `runSyncInBackground({ jobId, orgId, ... })` which kicks off a detached promise (`void runSync(...).catch(...)`) — the HTTP handler does NOT await it.
3. Returns `202 {jobId, status:"running"}` immediately.

The detached promise updates the row to `succeeded` (with `summary` jsonb) or `failed` (with `error` text) once Gmail + People (`connections.list` + `otherContacts.list`) ingest finishes. Callers poll `GET /orgs/google/sync/{jobId}` until `status != 'running'`. People connections (address book) and otherContacts (Gmail-collected) results are summed into a single `summary.contacts` accumulator — the UI does not distinguish the two sources. Tokens minted before the `contacts.other.readonly` scope was added skip the `otherContacts.list` call with a `console.warn`; the user must reauth to receive Gmail-collected contacts.

**Why async** — the dashboard's Vercel proxy caps function invocations at 300 s (Pro plan). First-sync backfills against busy mailboxes blew past that and surfaced as `FUNCTION_INVOCATION_TIMEOUT sin1::...`. Returning 202 keeps the proxy round-trip well under the cap regardless of mailbox size.

**Restart caveat (v1 trade-off)** — there is no queue and no worker. If the Railway service restarts mid-sync, the row stays `running` forever. Acceptable while sync is user-driven (the user can simply re-click sync); revisit when sync becomes scheduled or volume grows. The next iteration is `pgmq` with a reaper that flips long-stale `running` rows to `failed`.

**Large-mailbox reliability (do NOT regress) — a Gmail access token lives ~1h; a full backfill of a 15k+ message mailbox fetches one message at a time and EXCEEDS that window.** So the ingest loop MUST NOT mint one access token at the top and thread it through the loop (that was the original bug: every fetch past the hour 401'd → the whole job threw → `gmail_history_id` never persisted → next sync ran a full backfill from scratch → infinite full-rescan, 0 successful syncs ever). Invariants, all in `gmail-ingest.ts` / `people-ingest.ts` / `google-tokens.ts`:
- Thread an `AccessTokenProvider` (`createAccessTokenProvider`), never a bare token string. Every Google call runs through `withTokenRetry(provider, t => …)` which lazily re-mints on near-expiry and **force-refreshes + retries once on a 401**.
- Backfill is **resumable**: load the `gmail_message_id`s already in bronze and skip them before the per-message `getMessage`, so an interrupted run converges instead of re-scanning from zero. `gmail_history_id` is persisted only on successful completion (captured pre-loop) → next sync is a delta.
- Google People expires syncTokens: on `GoogleApiError` `400` with body `EXPIRED_SYNC_TOKEN` (`isExpiredSyncTokenError`), clear the stored sync token and retry a full list once. Applies to BOTH `people_sync_token` and `other_contacts_sync_token`.
- Match Google failures on `GoogleApiError.status`, never `.message.includes("…")` substrings.

### Future gold / canonical-Human trigger

Silver (`google_contacts_silver` / `gmail_messages_silver`) exists (see Data layering above). Promote further — a canonical `Human` entity or a materialized gold table — only when one of:

1. A second source (LinkedIn, Apollo, manual import) feeds the same canonical `Human` and cross-source merging is required (silver here is single-source per resource).
2. The read projection (bronze+silver LEFT JOIN + dedup) becomes too expensive per request and needs a materialized gold table.
3. Manual user edits must coexist with derived data (needs a `*_overrides` table winning over silver).

## OAuth flow

```
dashboard → POST /orgs/google/auth/start  → { url, state }
browser   → Google                         (user consents)
Google    → dashboard /services/crm/oauth/callback?code&state
dashboard → GET /orgs/google/auth/callback?code&state  (server-side, with identity headers)
google-service → google_oauth_tokens row (refresh + access)
```

The dashboard proxies the Google → service hop so the identity headers (`x-api-key`, `x-org-id`, `x-user-id`, `x-run-id`) can be attached.

---
> Source: [shamanic-technologies/google-service](https://github.com/shamanic-technologies/google-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
