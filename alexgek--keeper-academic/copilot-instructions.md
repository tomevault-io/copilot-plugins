## keeper-academic

> Pocket-replacement app for saving/reading web articles and academic papers. Runs on Cloudflare Workers + D1 (serverless SQLite). AI features via local Ollama. Zotero integration for academic reading management.

# Keeper - CLAUDE.md

## What is this?
Pocket-replacement app for saving/reading web articles and academic papers. Runs on Cloudflare Workers + D1 (serverless SQLite). AI features via local Ollama. Zotero integration for academic reading management.

## Architecture

### Backend (Cloudflare Workers)
- **Runtime**: Cloudflare Workers with Hono router (`src/index.js`)
- **Database**: Cloudflare D1 (SQLite-compatible), database name `keeper-db`
- **Static assets**: Served from `public/` via Workers Sites (`assets = { directory = "public" }` in wrangler.toml)
- **Content extraction**: `linkedom` + `@mozilla/readability` (NOT jsdom - it doesn't work on Workers)
- **FTS**: Manual FTS5 sync required - D1 doesn't support triggers, so `database.js` manually inserts/deletes FTS rows

### Frontend (vanilla JS, no build step)
- `public/app.js` - Main UI logic, auth, article management
- `public/ollama.js` - Ollama client (browser-side, talks to localhost)
- `public/styles.css` - Dark/light theme via CSS variables, responsive breakpoints at 768px and 480px
- `public/mobile.html` - Standalone mobile save page with embedded CSS/JS
- No framework, no bundler. All files served as-is.
- **DOMPurify** and **pdf.js** are vendored in `public/vendor/` (unmodified npm dist builds), not CDN-loaded — so `script-src` is `'self' 'unsafe-inline'` with no third-party hosts. See `SECURITY.md` for how to refresh a version

### Auth
- Token-based: `KEEPER_TOKEN` Cloudflare secret
- Middleware in `src/index.js` checks `Authorization: Bearer <token>` header only (query param support removed for security)
- Frontend stores token in `localStorage` as `keeper_token`
- **Fails closed**: with no `KEEPER_TOKEN` set, `/api/*` returns `503`, not open access. Local dev opts in explicitly with `ALLOW_UNAUTHENTICATED = "true"` in `wrangler.toml`, or put a token in `.dev.vars`
- Failed attempts are throttled per-IP in isolate memory (10/min → `429`). A speed bump, not a guarantee — isolates are per-colo and ephemeral, so an attacker who reconnects gets a fresh budget; a Cloudflare Rate Limiting rule is the real control

### AI (Ollama - client-side only)
- All LLM calls go from browser directly to `localhost:11434` (Ollama)
- Server has zero AI dependencies - it only stores results in `article_insights` table
- Ollama settings stored in `localStorage` (`keeper_ollama_url`, `keeper_ollama_model`, `keeper_ollama_ctx`, `keeper_ollama_num_predict`)
- **Model fallback**: `resolveModel()` in `ollama.js` checks the configured model against `/api/tags` before generating. A model deleted from Ollama leaves a live `localStorage` key holding a dead name, so it falls back to another tag of the same family, then the default, then anything installed — persisting the correction and notifying via `onModelFallback`.
- Background generation: insight jobs continue even when user navigates away from article; results save to server and toast notification shown
- **Two insight modes**: web articles get standard summary/key_points/tags; Zotero items get academic prompt (methodology, key_findings, limitations, relevance). Selection is automatic based on `article.source`
- **Batch insights**: Sequential queue processes multiple articles one-at-a-time with progress bar and cancel support. Uses AbortController to cleanly abort in-flight Ollama requests.
- **Range Vibe enrichment**: When articles have existing AI summaries in `article_insights`, Range Vibe includes those summaries (truncated to 200 chars) in the prompt for richer pattern analysis
- **num_predict**: Configurable max output tokens in AI Settings modal. Stored in `localStorage`. Lower values = faster generation on constrained hardware. Defaults to `OLLAMA_DEFAULTS.numPredict` (1024) when unset; `0` means unlimited.
- **CORS requirement**: Ollama must be started with `OLLAMA_ORIGINS="*"` to accept requests from the Workers domain

### Zotero Integration
- **Server-side only**: `src/zotero.js` talks to Zotero Web API v3 from Workers. No browser-side Zotero calls.
- **Credentials**: Stored in D1 `settings` table (key-value), NOT in localStorage or env vars. API key + user ID.
- **No full-library sync**: By design. Three manual import modes: date range (by `dateAdded`), collection, search.
- **Preview-then-import flow**: Preview endpoints return lightweight item summaries; import endpoint re-fetches full items by key from Zotero API before saving. This avoids shape mismatches between preview and import data.
- **Early termination on date range**: `fetchItemsByDateRange` paginates items sorted by `dateAdded` desc and stops once past the start date, avoiding full-library downloads for large Zotero libraries.
- **Dedup**: Import checks both `zotero_key` and `url` to avoid duplicating items already in Keeper.
- **Full-text**: `fetchFullText(itemKey)` attempts to pull indexed PDF text from Zotero; falls back to abstract-only display if unavailable.
- **PDF attachments**: `fetchPdfFile(attachmentKey)` downloads PDF binary from Zotero Web API. Only works for cloud-synced PDFs — local-only files return 404.
- **Academic full-text retrieval** (`POST /api/articles/:id/fetch-fulltext`): Multi-strategy pipeline: Unpaywall OA locations → DOI direct resolution → Semantic Scholar → Zotero fulltext API → Zotero PDF attachment. Most publisher sites block server-side access (403, JS-rendered pages), so success rate for paywalled papers is low.
- **Client-side PDF upload** (`POST /api/articles/:id/upload-pdf`): Accepts pre-extracted text from browser-side pdf.js. This is the reliable path for academic full-text — user downloads PDF manually and uploads it. Server-side `unpdf` does NOT work in Workers (pdf.js needs browser APIs).
- **PDF text cleanup**: `cleanPdfText()` in `app.js` does conservative line-by-line filtering of page numbers, running headers, DOI lines, and publisher boilerplate. Does NOT cut content by section headers — only removes clear artifacts.

## Key Conventions

### First-time setup
See `SETUP.md`. In short: `npx wrangler login`, `npx wrangler d1 create keeper-db`,
`cp wrangler.toml.example wrangler.toml` and fill in its two placeholders, apply
migrations, set the `KEEPER_TOKEN` secret, deploy. The Worker URL is
`https://keeper.<your-subdomain>.workers.dev`.

### Deploying
Authenticate once with `npx wrangler login`; deploys then carry no token:
```bash
npx wrangler deploy
```

`wrangler.toml` is **gitignored** and holds your real values; the committed file
is `wrangler.toml.example`. A fresh clone has no `wrangler.toml` and wrangler
will not start without one — `cp wrangler.toml.example wrangler.toml` first.
Because the real file is never tracked, your `database_id` cannot be committed
by accident and survives every `git pull`.

After deploying, confirm the instance is both serving and still protected:

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://keeper.<your-subdomain>.workers.dev/api/articles
```

**`401` is the healthy answer** — it proves the `KEEPER_TOKEN` secret is set and
auth is live. A **`503` means the secret is missing**; the API refuses everything
rather than serving your library openly, so fix it with
`npx wrangler secret put KEEPER_TOKEN` and redeploy. A `200` should be impossible
unless `ALLOW_UNAUTHENTICATED` was set on a deployed Worker, which it never
should be. The root path returns `200` either way, so checking the homepage alone
tells you nothing about auth.
Check the connection with `npx wrangler whoami`. The OAuth session lives in
`~/.config/.wrangler` and is **per-machine** — a fresh clone, a CI runner, or a
cloud/remote coding session does not inherit it and reports "not authenticated"
until logged in there separately.

Never commit a Cloudflare API token — that includes `.claude/settings.local.json`,
where Claude Code stores approved commands verbatim, so a token typed inline as
`CLOUDFLARE_API_TOKEN=<token> npx wrangler ...` ends up saved in that file. It is
gitignored; keep it that way, and supply a token as an environment variable set
outside the repository if you need one.

### Tests

```bash
npm test    # node --test test/*.test.mjs — no dependencies, no build
```

Covers the SSRF guard, FTS5 sync ordering, list/search projections, the
server-side HTML escapers, and the auth middleware (fail-closed behaviour,
`Bearer` parsing, per-IP throttle — driven through the real Hono app). Frontend
escapers are not covered (they need a DOM). See `SECURITY.md`.

### D1 Schema Changes
Add a numbered file to `migrations/` and apply with
`npx wrangler d1 migrations apply keeper-db --remote`. For one-off queries:
```bash
npx wrangler d1 execute keeper-db --command "SQL_HERE" --remote
```
Tables: `articles`, `tags`, `article_tags`, `articles_fts` (FTS5), `article_insights`, `settings`

### API Pattern
All API routes under `/api/*` require auth (`503` if `KEEPER_TOKEN` is unset). Routes follow REST conventions:
- `GET/POST /api/articles` - list/create (POST accepts `source: 'vibe'` for Range Vibe entries)
- `GET /api/articles/search?q=` - FTS5 full-text search
- `GET/PATCH/DELETE /api/articles/:id` - read/update/delete
- `POST /api/articles/:id/tags` - add a tag; name goes in the JSON body (`{ tag }`), **not** the path
- `PATCH/DELETE /api/articles/:id/tags/:tag` - rename/remove a tag; name in the path
- `GET /api/tags` - list all tags
- `GET/PUT /api/articles/:id/insights` - AI insights
- `POST /api/articles/:id/fetch-fulltext` - multi-strategy academic full-text retrieval (Unpaywall, DOI, Semantic Scholar, Zotero)
- `POST /api/articles/:id/upload-pdf` - accept client-extracted PDF text
- `GET /api/articles/range?start=&end=` - date range query (for Range Vibe), LEFT JOINs `article_insights` for summaries
- `PATCH /api/articles/:id/reading-status` - update reading status (unread/reading/read/reviewed)
- `GET/POST/DELETE /api/zotero/settings` - Zotero credentials
- `POST /api/zotero/test` - test Zotero connection
- `GET /api/zotero/collections` - list Zotero collections
- `POST /api/zotero/preview/{daterange,collection,search}` - preview items before import
- `POST /api/zotero/import/{daterange,collection,search,selected}` - import items
- `POST /api/import/pocket` - import a Pocket **HTML** export (`{ html }` in body)
- `POST /api/import/pocket-csv` - import a Pocket **CSV** export (`{ csv }` in body)

### Security
See **`SECURITY.md`** for the full security review: all 21 findings, what each
fix changed, and the two things still open — a Cloudflare Rate Limiting rule
(account configuration, not repo content) and the inline-`onclick` refactor that
would let `script-src` drop `'unsafe-inline'`.
Shared SSRF logic lives in `src/urlguard.js` — one `isPublicHttpUrl()` and a
`safeFetch()` that re-validates every redirect hop. Don't reintroduce a local
copy of either check.

**Static assets never reach the Worker**, so the CSP middleware in `src/index.js`
covers `/api/*` only. `public/_headers` carries the same policy for `index.html`
and friends. The two files have no shared source — **change both or neither.**

- **CSP headers**: `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy` — from the middleware for `/api/*`, from `public/_headers` for static assets
- **CORS restricted**: Allowlist derived per-request from the Worker's own origin + `localhost:8787`, plus any `ALLOWED_ORIGINS` var (Hono `cors()` middleware). Extension schemes (`chrome-extension://`, `moz-extension://`) are accepted so extension popups can preflight. Not a security boundary — the API is bearer-token auth with no cookies, so it isn't CSRF-exposed
- **SSRF protection**: `isPublicHttpUrl()` + `safeFetch()` in `src/urlguard.js` block private/loopback/link-local/CGNAT v4 and v6, non-HTTP schemes, and re-check every redirect hop
- **XSS prevention**: DOMPurify sanitizes article HTML in reader; `escapeHtml()` for text, `escapeJsAttr()` for inline event handler string params. `escapeJsAttr` must escape for **both** JS-string and HTML-attribute context — JS escaping alone leaves a literal `"` that closes the attribute
- **Auth**: strict `Bearer ` prefix, constant-time token comparison, fail-closed when the secret is missing
- **Error sanitization**: `errorResponse()` logs real errors server-side but returns generic message to clients; `zoteroError()` maps config/validation failures to 400
- **Input validation**: `limit` param capped at 1000; article URLs validated before fetch (including in Pocket imports); Zotero keys must match `^[A-Z0-9]{8}$` and user IDs `^\d+$` before entering an API path; FTS queries quoted per-term; vibe URLs use `crypto.randomUUID()`
- **Unpaywall email**: Configured via `UNPAYWALL_EMAIL` env var in `wrangler.toml`, not hardcoded

### Mobile Responsive
- **Breakpoints**: 768px (tablet) and 480px (phone) in `public/styles.css`
- **Sidebar**: Hidden on mobile, slides in as overlay via hamburger menu button with backdrop
- **Reader**: Full-screen overlay on mobile (fixed position, fills viewport)
- **Grid**: Single column on phones, 200px min on tablets
- **Touch targets**: 44px minimum on buttons, nav items, tags
- **Safe areas**: `env(safe-area-inset-*)` for notched devices
- **PWA**: `manifest.json` with share target, `theme-color` meta tag for Android status bar

### Gotchas
1. **No jsdom on Workers** - Use `linkedom` for DOM parsing
2. **No triggers in D1** - FTS must be manually synced in `database.js` create/update/delete methods. **Order matters**: `articles_fts` is an external-content table (`content='articles'`), so `DELETE FROM articles_fts` reads the current row out of `articles` to work out which terms to drop. Always `removeFts` → write → `insertFts`; deleting *after* the write removes the new terms and orphans the old ones, and search then keeps matching text that no longer exists. Route body updates through `articleDb.updateContent()` rather than hand-rolling the SQL
3. **Fetch failures are normal** - Sites like NYT return 403; `fetcher.js` has a `buildFallback()` that saves URL/title/hostname anyway
4. **CORS for Ollama** - Must set `OLLAMA_ORIGINS="*"` or the Workers domain, otherwise browser blocks requests with 403. On macOS, the Ollama.app menu bar service reads env vars from the launchd session, not your shell — `export OLLAMA_ORIGINS` in a terminal has no effect on it. Set it with `launchctl setenv OLLAMA_ORIGINS "*"` and restart the app; for a fix that survives reboots, install a `RunAtLoad` LaunchAgent (e.g. `~/Library/LaunchAgents/com.<user>.ollama-origins.plist`) that runs that `launchctl setenv` command at login. If you see "address already in use" from `ollama serve`, that means the app is already running as this service — check `lsof -nP -iTCP:11434 -sTCP:LISTEN` rather than trying to start a second instance.
5. **Browser extensions** need the auth token configured in extension options (stored in `browser.storage.local`)
6. **wrangler version** - Project was built with wrangler 3.x; update to 4.x may require config changes
7. **Zotero API itemType filter** - Don't use `itemType: '-attachment || -note'` as a query param; Zotero rejects it with 400. Filter client-side with `filterImportable()` instead.
8. **Zotero preview vs import shape** - Preview endpoints return flattened summaries for display; import must re-fetch full items by key. Never pass preview-shaped items to `mapZoteroItem()`.
9. **Zotero date range = dateAdded** - The date range import filters by when items were added to Zotero, not by publication date. This is intentional for the "sync recent additions" use case.
10. **Settings table** - `settings` is a generic key-value store in D1. Currently stores `zotero_api_key` and `zotero_user_id`. Use `createSettingsDb(db)` from `database.js`.
11. **unpdf doesn't work in Workers** - pdf.js requires browser APIs (`_isSameOrigin` etc.). Use client-side pdf.js for PDF text extraction instead — vendored at `public/vendor/pdf.min.mjs`, not a CDN. The `unpdf` package has been removed from `package.json`.
12. **Publisher sites block server-side fetches** - Taylor & Francis, Elsevier, etc. return 403 or serve JS-rendered shells. `linkedom` + Readability extracts 0 words from these. Upload PDF is the reliable fallback.
13. **Zotero Web API can't serve local-only PDFs** - If a PDF exists only in the user's local Zotero data directory (not synced to Zotero cloud storage), the `/items/{key}/file` endpoint returns 404.
14. **Unpaywall requires a real email** - The API rejects `@example.com` addresses with 422. Set `UNPAYWALL_EMAIL` env var in wrangler.toml.
15. **Vibe entries use synthetic URLs** - Articles with `source: 'vibe'` use `keeper://vibe/{uuid}` as their URL since the `url` column is NOT NULL. These have no external link.
16. **Auth token header only** - Query parameter `?token=` support was removed for security. All clients must use `Authorization: Bearer` header.
17. **DOMPurify required** - Loaded from `public/vendor/purify.min.js` (vendored, not CDN). If it fails to load, `sanitizeHtml()` falls back to stripping all HTML tags, so the reader silently renders plain text — that flattening is the tell. pdf.js's main module and worker in the same directory must stay on matching versions.
18. **Source filter includes vibes** - Four source filter buttons: All / Web / Zotero / Vibes. Vibe entries hide reading status, "Open original" link, and AI insights section in the reader.
19. **CORS is derived, not hardcoded** - The allowlist is built per-request from the Worker's own origin plus `http://localhost:8787`. Set `ALLOWED_ORIGINS` (comma-separated) in `wrangler.toml` only for extra origins like a custom domain. Don't reintroduce a hardcoded deployment URL. Extension schemes are allowed in addition, since a `chrome-extension://` origin cannot be derived.
20. **Extensions ship with no server URL** - `DEFAULT_SERVER` is `''` in all extension scripts; the URL and token are entered in the extension's options page and stored in extension storage. Both Chrome and Firefox send `Authorization: Bearer`.
21. **The bookmarklet carries no token** - `public/bookmarklet.html` builds a snippet that opens `/mobile.html?url=…` on the Keeper origin, where the save page reads the token from localStorage itself. It survives token changes and needs re-dragging only if the Worker URL changes. It cannot POST to `/api/articles` directly: that is a cross-origin preflighted request from whatever site you are reading, which the CORS allowlist rejects by design.
22. **`think: false` is required** - Reasoning-capable models (gemma4, qwen3, deepseek-r1) emit a thinking trace that `ollama.js` does not render as output (the stream loop reads `chunk.response`). Without `think: false` the UI shows nothing while the model burns tokens and grows the KV cache — indistinguishable from a hang. `generate()` also passes `onThinking` so a model that ignores the flag surfaces as progress rather than silence, and `num_predict` defaults to 1024 rather than unlimited for the same reason.

---
> Source: [alexgek/keeper-academic](https://github.com/alexgek/keeper-academic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
