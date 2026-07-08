## reddivault

> Persistent project memory for Claude Code sessions. Read at session start; update when

# RedditVault — Claude Code Context

Persistent project memory for Claude Code sessions. Read at session start; update when
significant decisions are made or features added. Keep it **current-state** — decision
history lives in `docs/transcripts/` (see `journal.txt` for the index), not here.

RedditVault is a personal Reddit saved-items manager built to work around Reddit's
~1,000-item API limit. A static PWA (IndexedDB-first) syncs with Supabase; saves are
captured via a Chrome extension, a bookmarklet, or the Reddit RSS feed.

**Live URL:** https://reddivault.vercel.app · **Current version:** v0.9.19.0

---

## Hard Rules — every change must respect these

1. **Zero-build, static deploy.** Native ES modules loaded directly by the browser —
   no bundler, no package.json, no transpiler, no build step. Third-party libs (Dexie,
   PapaParse) come from CDN `<script>` tags in `pwa/index.html`. Vercel serves the files
   as-is.
2. **Version bumps on every deployed change.** Bump `APP_VERSION` in `pwa/js/state.js`
   AND `VERSION` in `pwa/sw.js` (format `major.minor.patch.hotfix`). The SW precaches a
   hand-maintained `ASSETS` array keyed off `VERSION` — **a new `js/*.js` file must be
   added to `ASSETS`** or it won't work offline.
3. **The `window` bridge.** Rendered HTML uses ~140 inline `onclick="fn(…)"` handlers
   that resolve against global scope. `app.js` does `Object.assign(window, ...modules)`,
   so any function referenced by an inline handler must be **`export`ed from its module**
   — the bridge picks it up automatically, zero template changes.
4. **Rating three-state.** `rating` is `null` (unrated) | `0` (thumbs-down) | `1`–`5`
   (stars). Always propagate with `?? null`, **never `|| null`** (which coerces a real
   `0` to unrated).
5. **Soft delete only.** "Permanently deleted" items stay in the DB with
   `isPermanentlyDeleted: true` + `deletedAt` (Supabase `deleted_at`) so feed sync can't
   resurrect them. Actual removal is only the explicit Purge action.
6. **Single ingestion path.** `mapRedditChild(child, source)` is the only mapper from a
   raw Reddit listing child to an item — feed sync and the bookmarklet drain both use it.
   The bookmarklet writes **only** to the `reddit_inbox` staging table, never to
   `reddit_saves`; all real writes go through the app's dedup-aware drain.
7. **`localEditAt` pull guard.** Every per-item local mutation stamps
   `item.localEditAt` with an ISO timestamp (client clock). `syncFromSupabase` and
   `deltaPullBeforePush` must skip overwriting any item whose `localEditAt` is newer
   than the pull's start time (both on the client clock), so an in-flight pull can't
   clobber a fresh edit.
8. **iOS quirks.** `font-size: 16px` on all inputs (prevents zoom-on-focus); viewport
   pinch-zoom lock is user-controllable (`state.disableZoom`). The bookmarklet string
   must stay quote-free (no double quotes, no backticks, no inline `onclick`) — banners
   use the `done()` helper with `textContent`.

---

## File Structure

```
/
├── CLAUDE.md                          ← this file
├── SETUP.md                           ← end-user setup guide
├── supabase-schema.sql                ← DB schema (fresh install + migration sections)
├── vercel.json                        ← Vercel config (auto-deploys from GitHub)
├── pwa/
│   ├── index.html                     ← shell: markup + CDN libs + module entry
│   ├── styles.css                     ← all app CSS
│   ├── sw.js                          ← service worker (VERSION + ASSETS list)
│   ├── manifest.json, icon-*.png
│   └── js/                            ← the app, native ES modules
│       ├── app.js                     ← entry: imports all, window bridge, bootstrap
│       ├── state.js                   ← APP_VERSION, Dexie db + version chain, state, syncLog
│       ├── util.js                    ← escHtml, fmtDate, showToast, renderMarkdown,
│       │                                openLink, fullUrl, ratingDisplay, applyZoomSetting
│       ├── core.js                    ← init, loadConfig/loadData, rebuildTagCache,
│       │                                rebuildFilterLists, reconcileDirtyState, markDirty
│       ├── enrich.js                  ← Arctic Shift + Reddit enrichment, rate limiting
│       ├── cloud.js                   ← Supabase REST (supabaseFetch), push/pull/delta sync,
│       │                                preferences push/pull, retry/dirty machinery
│       ├── feed.js                    ← RSS/JSON feed sync, _buildProxyUrl, config save
│       ├── bookmarklet.js             ← buildInboxBookmarklet, drainInbox, ingestChildren,
│       │                                mapRedditChild, applyScoreUpdates,
│       │                                importPastedBookmarklet, cspProbeBookmarklet
│       ├── dataio.js                  ← CSV import, JSON backup/restore, repairs, delete
│       ├── search.js                  ← parseSearchQuery, itemMatchesTokens, _parseWildcard,
│       │                                affinityScore, sortItems, filteredItems,
│       │                                list-options helpers (serialise/applyListOptions)
│       ├── items.js                   ← item/list mutations (favourite, rate, trash, delete,
│       │                                list CRUD), showPage, search/filter actions
│       ├── render.js                  ← barrel: `export *` over render/ submodules
│       └── render/
│           ├── shell.js               ← render dispatcher, header, error fallback,
│           │                            showModal/closeModal, showRatingMenu, sort control
│           ├── home.js                ← renderHome, renderRecent
│           ├── browse.js              ← renderBrowse, item list, search/filter handlers
│           ├── card.js                ← renderItemCard
│           ├── preview.js             ← showPreview, closePreview, showLinkPicker,
│           │                            refreshOpenPreviewMeta
│           ├── trash.js               ← renderTrashView
│           ├── lists.js               ← renderLists + list menus
│           └── settings.js            ← the whole settings page
├── extension/                         ← Chrome extension (manifest, popup, background)
├── cloudflare-worker/
│   ├── reddit-feed-proxy.js           ← CORS proxy worker
│   └── wrangler.toml                  ← wrangler deploy config
└── docs/transcripts/                  ← full session transcripts + journal.txt index
```

Other modules and the window bridge import rendering from `./render.js` (the barrel),
never from `render/` submodules directly.

---

## Data Flow

```
Reddit (session cookies) → Chrome Extension → Supabase → PWA (IndexedDB cache)
Reddit (session cookies) → Bookmarklet → Supabase reddit_inbox → PWA drains into library
Reddit (RSS feed)        → Cloudflare Worker (or corsfix proxy) → PWA feed sync
Reddit public JSON API   → PWA enrichment (no auth needed)
```

- **IndexedDB is the source of truth**; Supabase is the sync layer. The PWA reads local
  on every load; `syncFromSupabase` merges remote into local, `pushToSupabase` /
  `pushAllDirty` push local items up.
- **Session-based sync, not OAuth** — the extension and bookmarklet ride the user's
  existing reddit.com login cookies, avoiding Reddit's developer-app approval. Personal
  use only; the Supabase anon key embedded in the bookmarklet is RLS-protected and
  acceptable for this threat model.
- There are **no folders** — organisation is via Lists. (The old `folder` column became
  `deleted_at` in a Supabase migration; a legacy `folders` Dexie store lingers unused.)

---

## Supabase Schema

Five tables (see `supabase-schema.sql`):

| Table | Purpose |
|-------|---------|
| `reddit_saves` | items (posts + comments) |
| `reddit_inbox` | bookmarklet staging — drained into `reddit_saves`, then cleared |
| `reddit_lists` | user lists (static and smart) |
| `reddit_item_lists` | many-to-many item ↔ list, unique on `(reddit_id, list_name)` |
| `user_preferences` | key-value settings sync (`key, value, updated_at`) |

**`reddit_saves`:** `id, reddit_id, type, subreddit, title, url, permalink, body, author,
score, saved_at, post_created_at, enrich_status, is_favourite, rating, is_disliked,
deleted_at, created_at, updated_at`
- `enrich_status`: `'pending' | 'enriched' | 'dead'`
- `deleted_at`: null = active, timestamptz = permanently deleted
- `is_disliked`: soft trash (recoverable) · `rating`: nullable int (see Hard Rule 4)
- `updated_at`: maintained by moddatetime trigger — delta pulls key off it

**`reddit_lists`:** `id, name, type ('static'|'smart'), query, is_tag, tag_name,
options_json, created_at, updated_at`
- `options_json` is `text`, **not jsonb** — the app stores a JSON *string*
  (`serialiseListOptions`) and `JSON.parse`s it on read (`applyListOptions`); jsonb would
  round-trip as an object and the options would be silently dropped.

**`reddit_inbox`:** `id, reddit_id, payload jsonb, created_at` — transient staging.
- `reddit_id` is the `UNIQUE` upsert key (`on_conflict=reddit_id`,
  `resolution=ignore-duplicates`, so bookmarklet re-runs don't pile up).
- Rows are typed by `payload.op`: **no `op`** = capture (bare `{kind, data}` Reddit child,
  keyed by fullname `t3_…`/`t1_…`, fed to `mapRedditChild`); **`op:'score'`** =
  `{op:'score', reddit_id, score}`, keyed `'score:'+fullname` so the two op types can
  never collide under ignore-duplicates.

**`user_preferences` keys:** `redditFeedUrl`, `redditUsername`, `scoreRefreshLimit`,
`feedProxyUrl`, `feedProxyType`, `feedFormat`, `confirmDestructive`, `recentSearches`,
`last_modified`.

---

## IndexedDB Schema (Dexie, version 9)

```javascript
db.version(9).stores({
  items:      '++id, redditId, type, subreddit, title, url, savedAt, postCreatedAt, enriched, enrichStatus, enrichAttempts, syncedAt, isPermanentlyDeleted',
  folders:    '++id, name, icon, createdAt',   // legacy, unused
  lists:      '++id, name, type, createdAt, isTag',
  item_lists: '++id, itemId, listId',
  config:     'key',                            // local key-value config
});
```

Unindexed fields ride along on objects (e.g. `items.localEditAt`, `lists.optionsJson`).
The version chain in `state.js` carries upgrade migrations — never edit old versions,
add a new `db.version(n)`.

Field mapping (IndexedDB camelCase → Supabase snake_case): `redditId → reddit_id`,
`savedAt → saved_at`, `postCreatedAt → post_created_at`, `enrichStatus → enrich_status`,
`isFavourite → is_favourite`, `isDisliked → is_disliked`, `deletedAt → deleted_at`
(`isPermanentlyDeleted` ⇔ `deleted_at` non-null), `isTag → is_tag`,
`tagName → tag_name`, `optionsJson → options_json`.

---

## App State (`state` in state.js)

Key fields (see `state.js` for the full annotated object):

```javascript
state.page            // 'home' | 'browse' | 'recent' | 'lists' | 'settings'
state.items / lists / itemLists   // loaded from IndexedDB
state.search          // current search string
state.showFilters, filterType, filterRating, filterHasLinks,
state.filterSubreddit, filterAuthor, filterDateFrom/To,
state.filterDateField // 'postCreatedAt' | 'savedAt'
state.filterFavourite, searchBody
state.activeTagIds    // list ids active as tag filters
state.tagMode         // 'AND' | 'OR'
state.tagCache        // Map<listId, { count, itemIds: Set }> — see Lists
state.sortBy          // 'affinity' | 'savedAt' | 'postCreatedAt' | 'score' | 'rating' | 'subreddit' | 'title'
state.sortDir         // 'asc' | 'desc'
state.showTrash / showDeleted     // trash + deleted-items views (flags, not pages)
state.localDirty / cloudAhead     // sync dirty tracking
state.lastPushedAt    // delta cursor for push
state.supabaseUrl / supabaseKey
state.redditFeedUrl, feedProxyUrl, feedProxyType ('cloudflare'|'corsfix'),
state.feedFormat      // 'rss' | 'json' — Reddit currently WAF-blocks .json
state.redditUsername  // expected account for the bookmarklet check
state.scoreRefreshLimit  // saves checked by "Refresh scores" (0 = all, default 500)
state.listView / listSeparate / listSmartFirst
state.confirmDestructive, disableZoom
state.autoFeedSync, autoFeedSyncInterval
```

---

## Render Pattern

Manual render cycle, everything writes `innerHTML`:

- `render()` — full shell (nav + current page). Call on page/state changes.
- `renderBrowseList()` — just the item list. Call on search/filter/sort changes.
- `filteredItems()` (search.js) — the central filter: applies `isPermanentlyDeleted`,
  `isDisliked`, `enrichStatus !== 'dead'`, view context, active tags,
  subreddit/author/date filters, and search tokens.
- Error boundary: `window.onerror` + `unhandledrejection` show a recovery UI with version.

---

## Search Engine

`parseSearchQuery(raw)` → token objects; `itemMatchesTokens(item, tokens)` matches;
`_parseWildcard(raw)` → `{ value, exact, suffixWild, prefixWild }`.

| Input | Behaviour |
|-------|-----------|
| `python tutorial` | AND — whole-word matches |
| `react, vue` / `(react, vue)` | OR group |
| `"machine learning"` | exact substring |
| `witch*` / `*witch` / `*witch*` | prefix / suffix / contains |
| `-javascript` | exclude |
| `r:programming` / `u:name` | subreddit / author (partial match) |
| `type:post` | type filter |

- Bare words are **whole-word** matches (`app` ≠ `apple`); use `app*` for prefix.
- Wildcards work inside OR groups: `(run*, walk*)`.
- Field prefixes stay partial: `r:prog` matches r/programming.
- A negated term in a bare comma list (`-python, javascript`) falls through to AND parsing.
- Search input normalises smart quotes `' ' " "` (iOS autocorrect).

---

## Sync System

### Cloud push / pull (cloud.js)
- **Delta cursor:** `state.lastPushedAt` (historical name) means "everything on the
  cloud up to T has been merged locally". It advances **only when a pull completes**
  (`syncFromSupabase`, `deltaPullBeforePush`), never on push — advancing it on push
  would skip rows another device wrote between our last pull and the push. Pushes
  write `last_modified` (via `markClean`) so other devices know to pull; the cost is
  at most one redundant delta pull after a push-only session.
- `pushToSupabase(items)` — upsert to `reddit_saves` with `on_conflict=reddit_id`;
  stamps `syncedAt` on the pushed items on success, calls `markDirty` on failure.
  Items with no `syncedAt` are the retry queue: `pushAllDirty` re-pushes them (batched,
  skipping locally-deleted ones), and a completed pull keeps `localDirty` set while any
  remain.
- `pushListsToSupabase()` — upserts lists, rebuilds `reddit_item_lists` per list, and
  always touches the parent list's `updated_at` when memberships change (so delta sees it).
- `syncFromSupabase()` — delta pull via `updated_at=gt.{lastSync}`; full scan on first
  run. `lastSync` is recorded at pull-*start* so rows modified during a slow pull aren't
  missed. Static-list memberships reconcile per-list only for lists whose `updated_at`
  changed. Respects the `localEditAt` guard (Hard Rule 7).

### Feed sync (feed.js)
- `syncFromFeed()` — fetch Reddit RSS via CORS proxy, parse, upsert new items; skips
  known ids and permanently-deleted items. Auto-runs on interval if `state.autoFeedSync`.
- Proxy via `_buildProxyUrl(feedUrl)` — **always use it**, never build URLs by hand:
  `'cloudflare'` → `{workerUrl}?url={encoded}`; `'corsfix'` →
  `https://proxy.corsfix.com/?{feedUrl}` (unencoded, no key, no setup).

### Bookmarklet (bookmarklet.js) — mobile / non-Chrome capture
`buildInboxBookmarklet(url, key, expectUser)` generates a menu-driven `javascript:`
bookmarklet. **Must run on `old.reddit.com`** — new Reddit's CSP `connect-src` blocks the
cross-origin Supabase write (reads still work, so it looks like "found N items but
couldn't reach the inbox"). Off-reddit / new-reddit / logged-out states get overlays
telling the user to tap the bookmark again on the right page (a bookmarklet can't
survive navigation). Each menu button starts on a fresh user gesture (popup-blocker).

- **Account check:** resolves `/api/me.json` before the menu; header shows
  "Logged in as u/…". If `expectUser` (`state.redditUsername`) is set and mismatched,
  blocks behind a warning with a "Use anyway" escape. Empty = display only.
- **① Capture:** incremental — read-only GET of recent `reddit_id`s
  (`order=saved_at.desc&limit=500`), then pages `saved.json` newest-first, stopping at
  the first fully-known page; POSTs only new raw children to `reddit_inbox`. Falls back
  to a full scan if the read fails, and to a clipboard envelope
  (`{type:'rv-bookmarklet', children}` → "Paste captured items" /
  `importPastedBookmarklet`) if the inbox POST fails.
- **② Refresh scores:** reads `scoreRefreshLimit` live from `user_preferences` at run
  time (not baked in), pages that many active ids from `reddit_saves` via `Range`
  headers, batches `/api/info.json?id=…` (100/call, 1s pacing; 429 retries the batch),
  and flushes `{op:'score',…}` rows per batch so interruptions keep partial progress.

**Drain:** `drainInbox({manual})` runs on startup, on `visibilitychange` foreground, and
from the Settings button. Pages the inbox, dispatches by `payload.op`: capture rows →
`ingestChildren(children,'bookmarklet')`; score rows → `applyScoreUpdates`. Then DELETEs
drained ids (chunked). Result `{added, skipped, scoresUpdated, drained}`; quiet runs
toast only when something changed. Anti-resurrection: `ingestChildren` dedups against
all known ids **including permanently-deleted**; `applyScoreUpdates` only touches items
already in `state.items` and pushes via the app's own `pushToSupabase`.

---

## Lists, Tags, Ratings

- **Static lists** — manual membership, stored in the `item_lists` junction.
- **Smart lists** — a search query; membership computed at render time via
  `parseSearchQuery(list.query)`, never stored. `optionsJson` persists the list's
  filters/sort (`serialiseListOptions`/`applyListOptions`).
- **Tags** — smart lists with `isTag: true`; render as chip pills on matching cards and
  as filter buttons in browse. `rebuildTagCache()` precomputes
  `state.tagCache: Map<listId, { count, itemIds: Set }>` after `loadData()` and any list
  mutation, making per-item chip rendering O(1).
- **Affinity sort** — rating (0–50), favourite (+30 flat), author frequency (0–15
  log-scaled via `rebuildAuthorFreq()`), tag membership (0–20). Unrated (`null`) gets a
  neutral 10-pt baseline; thumbs-down (`0`) scores 0, below unrated.
- **Rating UI** — each card shows a compact rating chip (muted "☆ Rate" / 👎 / ★n) that
  opens `showRatingMenu(itemId)` (shell.js modal picker: 👎, ★1–5, Clear). Also reachable
  from the preview sheet, which refreshes its meta in place
  (`refreshOpenPreviewMeta()`). Read-only display always goes through
  `ratingDisplay(item)` in util.js.
- **Favourite ≠ rating** — `isFavourite` is a separate field, rendered as a **pink heart**
  (`#ec4899`, filled ♥ / hollow ♡) to stay distinct from amber rating stars and the red
  trash button. Card border tint pink when favourited; home stat reads "Favourited".

---

## Item Lifecycle

```
CSV import / feed sync / extension / bookmarklet drain
  → upsert IndexedDB (enrichStatus 'pending'|'enriched') → push to Supabase

Enrichment (manual): Arctic Shift bulk pass first, Reddit per-item fallback second
  → enrichStatus 'enriched' | 'dead' → push

Trash (isDisliked)      → hidden from browse, visible in Trash, recoverable
Permanent delete        → isPermanentlyDeleted + deletedAt / deleted_at; hidden everywhere
                          except Deleted Items; never resurfaces; restorable
Purge                   → actually removed from IndexedDB + Supabase; gone
```

**Deletion/restore sync rule** (both pull paths): a remote `deleted_at` always wins;
a remote row that is *active* while we hold a local permanent deletion un-deletes it
only if the remote row's `updated_at` is newer than our local `deletedAt` — so restores
propagate across devices, but a stale remote row can't resurrect a deletion.

**Enrichment detail:** phase 1 `enrichViaArcticShift` hits
`arctic-shift.photon-reddit.com` (`/api/posts/ids`, `/api/comments/ids`; up to 500
ids/request, 500ms between batches, no auth). Phase 2 `enrichItemFromReddit` is the slow
per-item fallback (configurable delay, default 7.5s) for whatever phase 1 missed.

---

## Settings Page — seven sections in order

1. **🔄 Sync New Saves** — sync button, auto-sync toggle/interval, feed connection (collapsible)
2. **🔖 Bookmarklet Sync** — bookmarklet copy, expected username, score-refresh limit, import-from-inbox
3. **☁️ Cloud Database** — status, manual push/pull, Supabase connection (collapsible)
4. **📊 Library** — stats (active items; deleted count if any), Backup & Restore
5. **📥 Import & Enrich** — CSV import, enrichment controls, About, speed settings
6. **🎛️ Behaviour** — confirm-destructive, disable pinch-to-zoom
7. **🔬 Diagnostics** (muted) — force reload, troubleshooting tools, sync log

---

## Conventions

- **CSS variables:** `--bg, --surface, --border, --text, --text-muted, --accent`
  (orange), `--accent2` (blue), `--danger` (red). Dark mode via
  `@media (prefers-color-scheme: dark)`.
- **Sync log:** `syncLog(msg, level)` — in-memory (session-only, cap 200), shown in
  Diagnostics. Levels: `'info' | 'ok' | 'warn' | 'error'`.
- **Markdown:** `renderMarkdown(text)` — bold, italic, inline code, links,
  strikethrough; extracts links before escaping. Used for comment bodies.
- **Toasts:** `showToast(msg)`; quiet background syncs only toast on change.

---

## Known Issues / Pending Work

1. **Cloudflare Worker `ALLOWED_ORIGIN`** — hardcoded to `https://reddivault.vercel.app`;
   new installs must change it before deploying the worker.
2. **Reddit CSV `saved_posts.csv`** — the saved-date field is usually empty, so `savedAt`
   is typically the import timestamp, not the real save date.
3. **Window-bridge migration** — a future pass can replace inline `onclick` handlers with
   a `data-action` delegated dispatcher and drop the bridge.

---
> Source: [narcolepticdoc/reddivault](https://github.com/narcolepticdoc/reddivault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
