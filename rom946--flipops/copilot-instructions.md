## flipops

> These rules are mandatory on every task. No exceptions.

# ⚡ COST EFFICIENCY RULES — READ FIRST, ALWAYS FOLLOW

These rules are mandatory on every task. No exceptions.

## @flipops-map anchor rule — apply before editing any file
Check every file you are about to edit for a `@flipops-map` block at L1.

**If the map has line numbers (old format):**
1. Read the existing map block
2. Place inline anchor comments in the code: Python `# ANCHOR:name` / JS/JSX `// ANCHOR:name`
3. Rewrite the map replacing line numbers with `ANCHOR:name` references
4. Remove any OFFSET line — no longer needed
5. Note "Uses named anchors" in the map header
Then proceed with the task.

**If no `@flipops-map` exists (non-test files only):**
1. Complete the task first
2. Add `@flipops-map` with named anchors (no line numbers)
3. Place `ANCHOR:` comments inline in the code

**After completing all changes:**
- Update the `@flipops-map` of every modified file
- Add/update `ANCHOR:` comments as needed
- Never use line numbers in any map

## Before reading any file
1. Check CLAUDE.md first — if the answer is here, do not read any other file
2. Only read files directly named in the task
3. If unsure which files are needed, ask — do not explore speculatively

## File reading rules
- NEVER read node_modules, dist, .vite, __pycache__
- NEVER read lock files (package-lock.json, yarn.lock)
- NEVER read .git folder
- NEVER read image/font/binary files
- NEVER read more than 5 files per task unless explicitly told to
- NEVER re-read a file you already read this session

## Prompt execution rules
- Implement directly — do not plan before implementing unless the task explicitly says "plan first"
- Do not summarize what you are about to do — just do it
- Do not explain what you just did — just confirm done
- Do not ask for confirmation mid-task
- Do not suggest alternatives unless asked
- Do not add features not explicitly requested

## Code editing rules
- Prefer minimal diffs over full file rewrites
- Edit only the specific lines that need changing
- Do not reformat or restyle untouched code
- Do not add comments unless asked
- Do not refactor unless asked
- Do not change variable names unless asked

## Output rules
- After completing a task, output only:
  1. Files modified (list)
  2. What was changed (one line per file)
  3. Any new env variables needed (if any)
- Nothing else — no explanations, no summaries, no suggestions for next steps unless asked

## Session hygiene
- If the conversation exceeds 15 messages, suggest /compact before continuing
- If asked to do more than 3 files in one task, flag it and suggest splitting into subtasks

## Project tracking files

After every batch of completed features, update both files before confirming done:

- **CHANGELOG.md** — append a versioned entry at the top under `[Unreleased]` or a new `[x.y.z]` block
- **STATUS.md** — update "What's working", "In progress", "Known bugs", and "Tech debt" sections to reflect current state

### Changelog entry format
```
## [x.y.z] - YYYY-MM-DD
### Added / Fixed / Changed / Removed
- One line per change, specific (feature name, file, behaviour)
```

### Rules
- Do not write vague descriptions ("improved UI") — name the component and what changed
- Bump patch version (0.x.**z**) for fixes; minor (0.**y**.0) for new features; major (**x**.0.0) for breaking changes
- Do not rewrite existing entries — only append new ones

---

# FlipOps — CLAUDE.md

## 1. Project Overview

FlipOps is a personal AI-powered second-hand market flipping assistant that helps find, analyze, negotiate, and track deals on Wallapop.

Monorepo layout:
```
FlipOps/
├── frontend/    # React + Vite (deployed to GitHub Pages)
├── backend/     # Python Flask (deployed to Render)
├── .github/workflows/deploy-frontend.yml
├── render.yaml
├── CLAUDE.md
└── .gitignore
```

---

## 2. Stack

**Frontend**
- Framework: React 18
- Build tool: Vite 5
- Styling: Tailwind CSS 3 (utility-first; custom component classes in `index.css`)
- Routing: React Router DOM v6 (`BrowserRouter` with `basename="/FlipOps"`)
- i18n: i18next + react-i18next + LanguageDetector (single file: `src/i18n.js`)
- Auth UI: Firebase Auth (Google provider)
- Icons: lucide-react

**Backend**
- Language: Python 3.11+
- Framework: Flask 3 with Flask-CORS
- AI: Anthropic Claude (model: `claude-haiku-4-5-20251001`)
- Scraping: curl_cffi (Chrome impersonation), BeautifulSoup4, ThreadPoolExecutor (8 workers)
- Search: DuckDuckGo HTML queries
- Auth verification: Firebase Admin SDK (ID token)
- Storage: Firebase Firestore
- Encryption: cryptography (Fernet) for API keys at rest

**Database & Auth**
- Firebase Firestore (database)
- Firebase Auth (Google sign-in, ID token verification)

**Deployment**
- Frontend: GitHub Pages (`gh-pages` branch, auto-deployed via GitHub Actions on push to `main`)
- Backend: Render (free tier, `gunicorn app:app`)
- Keep-alive: UptimeRobot pings `/api/health` every 5 min

## Token cost reminders
- `useApi` already handles auth headers — never rewrite auth logic
- Translation is already set up — never add a new i18n library, but always make sure translations are set up correctly on new pages/features you create
- Firestore access pattern is established — follow existing patterns in similar files

---

## 3. Key Files Map

### Frontend (`frontend/src/`)

| Role | Path |
|---|---|
| App entry | `frontend/src/main.jsx` |
| Routes | `frontend/src/App.jsx` |
| Auth context | `frontend/src/context/AuthContext.jsx` |
| API client | `frontend/src/hooks/useApi.js` |
| Firebase config | `frontend/src/firebase.js` |
| i18n config + all strings | `frontend/src/i18n.js` |
| Global CSS + Tailwind | `frontend/src/index.css` |
| Pipeline hook | `frontend/src/hooks/usePipeline.js` |
| Vite config | `frontend/vite.config.js` |

**Pages** (`frontend/src/pages/`)
| File | Route |
|---|---|
| `HomeView.jsx` | `/` |
| `SearchView.jsx` | `/search` |
| `DiscoveryView.jsx` | `/discovery` — See @flipops-map at L1 for internal anchors — ARCHITECTURE + GOTCHAS appended to map |
| `NegotiateView.jsx` | `/negotiate` |
| `ListingView.jsx` | `/listing` |
| `PipelineView.jsx` | `/pipeline` |
| `DashboardView.jsx` | `/dashboard` |
| `AdminDashboard.jsx` | `/management` — See @flipops-map at L1 for internal anchors |
| `HowToView.jsx` | `/howto` |
| `AppointmentsPage.jsx` | `/appointments` |
| `Login.jsx` | `/login` |

**Components** (`frontend/src/components/`)
| File | Purpose |
|---|---|
| `Layout.jsx` | Nav + page wrapper; renders `CounterOfferCalculator` globally |
| `ProtectedRoute.jsx` | Auth guard |
| `AppointmentModal.jsx` | Create/edit appointment modal |
| `CounterOfferCalculator.jsx` | Floating 💶 button (bottom-right, above mobile nav); slides up a margin/counter-offer calculator panel. No props, self-contained, rendered once in Layout. |
| `PlatformBadge.jsx` | Small colored pill badge showing the platform (wallapop/vinted/milanuncios/ebay_es). Props: `platform` (string). Used in DiscoveryView and PipelineView above item title. |

**Hooks** (`frontend/src/hooks/`)
| File | Purpose |
|---|---|
| `useApi.js` | All REST API calls to backend |
| `usePipeline.js` | Pipeline CRUD on localStorage |

**Utils** (`frontend/src/utils/`)
| File | Purpose |
|---|---|
| `extractWallapopUrl.js` | Extract + normalize Wallapop URL from arbitrary text |
| `expandKeywords.js` | Expand a keyword into search variants using user-saved groups; fallback to `[kw, kw + " segunda mano", ...]` |

### Backend (`backend/`)

| Role | Path |
|---|---|
| App entry | `backend/app.py` |
| Auth middleware | `backend/services/auth.py` |
| Encryption | `backend/services/encryption.py` |
| Key resolver | `backend/services/key_resolver.py` |
| Google Calendar | `backend/services/calendar_service.py` |
| Search providers | `backend/services/search_provider.py` — See @flipops-map at L1 for internal anchors (offset N=20) |
| Usage tracker | `backend/services/usage_tracker.py` — See @flipops-map at L1 for internal anchors (offset N=19) |
| Milanuncios scraper | `backend/services/scrapers/milanuncios.py` — See @flipops-map at L1 for internal anchors |

**Routes** (`backend/routes/`)
| File | Prefix |
|---|---|
| `search.py` | `/api/geocoder`, `/api/import`, `/api/discovery` — See @flipops-map at L1 for internal anchors (offset N=19) — ARCHITECTURE + GOTCHAS appended to map |
| `negotiate.py` | `/api/analyze-listing`, `/api/batch-analyze`, `/api/discussion/generate` — See @flipops-map at L1 for internal anchors (offset N=17) |
| `listing.py` | `/api/generate-listing` |
| `pipeline.py` | `/api/score-deals` |
| `user.py` | `/api/user/*` — See @flipops-map at L1 for internal anchors (offset N=26) |
| `admin.py` | `/api/admin/*` |
| `appointments.py` | `/api/appointments*` |

---

### DiscoveryView.jsx — Internal Map

Full path: `frontend/src/pages/DiscoveryView.jsx` (~1050 lines)
Props: `pipeline` (the `usePipeline()` hook result, passed down from App.jsx)

#### State variables block — L20–L50

| Variable | Holds |
|---|---|
| `keywords` | Current keyword input value |
| `location` | Current location input value |
| `loading` | true while `api.discover()` is in-flight |
| `analyzing` | true while `api.batchAnalyze()` is in-flight |
| `listings` | Array of available (non-sold) items after AI filter (score ≥ 50) |
| `discarded` | Items that are sold/reserved or scored < 50 |
| `filteredOut` | Count of items removed client-side (unused/reserved for future) |
| `error` | Inline error string shown inside the search card |
| `success` | Inline success string (e.g. "Added to pipeline") — auto-clears after 3s |
| `addedIds` | Set of item_ids already added to pipeline (disables the add button) |
| `expandedTextIds` | Set of item_ids with expanded AI reason text |
| `isDiscardedExpanded` | Whether the discarded/market-context accordion is open |
| `page` | Current pagination page |
| `hasMore` | Whether a "Load more" button should be shown |
| `recentSearches` | Array of last 5 searches (from Firestore via `api.getMe()`) |
| `savedSearches` | Array of user-starred searches (from Firestore) |
| `platformPrefs` | `{ wallapop, vinted, milanuncios, ebay_es }` booleans |
| `platformErrors` | Array of platform names that failed during the search |
| `platformCounts` | `{ wallapop: N, vinted: N, … }` per-platform result counts |
| `searchSource` | `"personal"` or `"admin"` — which key pool powered the last search |
| `activeProviders` | Array of provider names used (e.g. `["serpapi", "serper"]`) |
| `usageSummary` | `{ anyLowCredits, lowProviders }` from `api.getMe()` — drives low-credit banner |
| `bannerDismissed` | Whether the low-credit banner was dismissed this session (sessionStorage) |
| `userVariants` | Array of keyword variant groups from Firestore (loaded on mount) |
| `activeVariantLabel` | Trigger label of the variant group currently in use (or null) |
| `sessionChips` | Editable chip list for the active variant group (session-only edits) |
| `chipsEdited` | true if sessionChips differ from saved variants (shows "Save to Settings") |
| `chipInput` | Controlled input for adding a new chip inline |

#### useEffect hooks

- L62–96: **Mount effect** — calls `api.getMe()` + `api.getKeywordVariants()` in parallel; seeds `recentSearches`, `savedSearches`, `platformPrefs`, `usageSummary`, `userVariants`; also restores previous search state from `locationRouter.state.restoredState` (back button), OR handles `locationRouter.state.preloadedVariants` (from SearchView "Find similar" — sets `keywords`, `sessionChips`, `activeVariantLabel`, then calls `handleScan` after 300ms), OR restores from `sessionStorage` key `flipops_discovery_state`
- L99–111: **State persistence effect** — writes current listings/discarded/page/addedIds to `sessionStorage` key `flipops_discovery_state` whenever any of those change (only when results exist and not loading)

#### Handler functions

| Function | Line | Description |
|---|---|---|
| `restoreState` | L52 | useCallback; sets all result/pagination state from a snapshot object |
| `toggleText` | L113 | Toggles expanded/collapsed AI reason text for an item |
| `handleScan` | L122 | Main search: resolves variants, calls `api.discover()`, splits results into available/discarded, triggers `handleBatchAnalyze`; also saves recent search. Accepts optional 4th param `overrideChips` — bypasses variant/expand logic and uses those chips directly as search terms |
| `handleLoadMore` | L224 | Appends next page results; merges `platformCounts`; triggers `handleBatchAnalyze` for new items |
| `handleBatchAnalyze` | L287 | Calls `api.batchAnalyze()`; merges AI scores/verdicts back into `listings`; moves score < 50 items to `discarded`; sorts by score desc |
| `handleAddToPipeline` | L328 | Adds item to pipeline via `pipeline.addDeal()`; marks `addedIds`; shows success toast |
| `handleMoveToMain` | L356 | Moves item from `discarded` back into `listings`; marks item as `analyzed: true` |
| `handleAnalyzeItem` | L371 | Navigates to `/search` with `autoUrl` + full `discoveryState` snapshot for back-nav restore |
| `toggleSaveSearch` | L388 | Toggles current keyword+location in `savedSearches`; persists via `api.updateSettings()` |

#### Return JSX structure

```
L403  <div>                              — page root wrapper
L406    <div>                            — header (title + subtitle)
L416    <div.card>                       — search card (yellow border)
L417      <div>                          — variant chips row (only shown when activeVariantLabel set)
L459      <div>                          — platform chips row (Wallapop/Vinted/Milanuncios/eBay ES)
L493      <form onSubmit=handleScan>     — keyword input + location input + save-star + scan button
L540      <div>                          — saved searches + recent searches pills (conditional)
L590      <div>                          — inline error block (conditional on `error`)
L595      <div>                          — inline success block (conditional on `success`)
L600    </div>                           — end search card
L602    <div>                            — "AI is filtering…" analyzing banner (conditional on `analyzing`)
L611    <div>                            — per-platform counts bar (conditional, after search)
L623    <div>                            — platform error pills (conditional)
L632    <p>                              — search source badge (🔑 personal / shared) — conditional on results
L642    <div>                            — filtered-out notice (conditional on `filteredOut > 0`)
L649    <div>                            — LOW CREDIT BANNER (conditional on `usageSummary.anyLowCredits && !bannerDismissed`)
L671    <div>                            — results area (conditional on `listings.length > 0`)
L673      <div.card>                     — main results table card
L675        <table>                      — desktop results table (hidden on mobile)
L687          <tbody>                    — one <tr> per listing item
L811        <div>                        — mobile card layout (md:hidden)
L920      <div>                          — discarded / market context accordion
L921        <button>                     — accordion header toggle
L937        <div>                        — accordion body (desktop table + mobile cards)
L1024     <div>                          — Load More button (conditional on `hasMore`)
L1037   (else) <div.card>               — empty state (shown when no listings, not loading)
L1048 </div>                             — end page root
```

#### Conditional rendering blocks

- **L417**: Variant chips row — only when `activeVariantLabel` is set (user typed a recognized keyword)
- **L590**: Error block — `error` string is non-empty
- **L595**: Success block — `success` string is non-empty (auto-clears after 3s)
- **L603**: Analyzing banner — `analyzing === true` (batch AI in-flight)
- **L611**: Platform counts — `!loading && Object.keys(platformCounts).length > 0`
- **L623**: Platform error pills — `platformErrors.length > 0`
- **L632**: Search source badge — `!loading && listings.length > 0 && searchSource`
- **L642**: Filtered-out notice — `!analyzing && filteredOut > 0`
- **L649**: Low-credit banner — `usageSummary?.anyLowCredits && !bannerDismissed`
- **L671**: Results table — `listings.length > 0`; else empty state at L1037
- **L921**: Discarded accordion — `discarded.length > 0` (inside results block)
- **L1024**: Load More button — `hasMore` (inside results block)

#### How search results are fetched and stored

1. `handleScan` (L122) calls `api.discover(resolvedKeywords, location, 1, enabledPlatforms)` → sets `listings` (available) and `discarded` (sold/reserved)
2. Immediately after, `handleBatchAnalyze(uniqueAvailable, true)` (L215) calls `api.batchAnalyze()` → merges score/verdict/target/reason back into each item in `listings`; items with score < 50 move to `discarded`; remaining sorted by score desc
3. `handleLoadMore` (L224) repeats step 1 with `page + 1` and appends; then calls `handleBatchAnalyze` on the new batch only

#### Insertion points for new UI elements

- **Above search card** (before the yellow card): L404 — between header `</div>` and `<div className="card p-6 mb-8 ...`
- **Inside search card, above platform chips**: L417 — before `<div className="flex flex-wrap gap-2 mb-3">` (the platform chips row)
- **Inside search card, below the form**: L539 — after `</form>` and before the saved/recent searches block
- **Between search card and results** (status/notice zone): L601 — after `</div>` closing the search card; this is where the analyzing banner, platform counts, error pills, source badge, filtered-out notice, and low-credit banner all live
- **Top of results table**: L672 — inside `<div className="space-y-6">`, before the `<div className="card overflow-hidden ...`
- **Bottom of results, after table, before discarded**: L919 — after closing `</div>` of the results card (L918) and before the discarded accordion at L921
- **After Load More / bottom of results block**: L1035 — after the Load More `</div>` (L1035), still inside the results `<div className="space-y-6">`

---

## 4. Conventions in Use

**Translations**
- Single file: `frontend/src/i18n.js` — contains both `en` and `es` objects
- Usage in components: `const { t } = useTranslation()` then `t('namespace.key')`
- Namespaces: `nav`, `common`, `home`, `search`, `discovery`, `negotiate`, `listing`, `pipeline`, `dashboard`, `management`, `appointments`, `howto`
- Always add new keys to BOTH `en` and `es` sections

**API calls**
- All calls via `useApi()` hook (`frontend/src/hooks/useApi.js`)
- Auth token automatically attached from Firebase current user
- Pattern: `const api = useApi()` then `await api.methodName(params)`

**Auth access in components**
- `const { user, googleAccessToken } = useAuth()` from `frontend/src/context/AuthContext.jsx`
- Protected routes wrapped in `<ProtectedRoute>` component

**Firestore access**
- Frontend: only `AuthContext.jsx` reads/writes Firestore directly (user doc)
- Backend: all business logic hits Firestore via Firebase Admin SDK in route handlers
- Never add direct Firestore calls in page components

**CSS / Styling**
- Tailwind utility classes only in JSX
- Custom reusable classes defined in `frontend/src/index.css` under `@layer components`:
  `.card`, `.btn`, `.btn-primary`, `.btn-secondary`, `.btn-success`, `.btn-danger`, `.btn-ghost`, `.input`, `.label`, `.badge`, `.badge-deal`, `.badge-fair`, `.badge-pass`, `.metric-card`, `.section-header`, `.page-container`, `.gradient-text`, `.spinner`
- Mobile utilities: `.table-scroll-wrapper`, `.stack-mobile`, `.main-content`
- CSS variables in `:root`: `--color-bg`, `--color-surface`, `--color-border`, `--color-accent`, `--color-success`, `--color-warning`, `--color-danger`

**Component pattern**
- Functional components with hooks
- Local state with `useState`, side effects with `useEffect`
- Sub-components defined in same file when tightly coupled (e.g. `DealRow`, `AddDealModal` in `PipelineView.jsx`)

---

## 5. Environment Variables

### Frontend (`frontend/.env`)
| Variable | Description |
|---|---|
| `VITE_API_URL` | Backend base URL (empty = use Vite proxy in dev; full URL in prod) |
| `VITE_FIREBASE_API_KEY` | Firebase project API key |
| `VITE_FIREBASE_AUTH_DOMAIN` | Firebase auth domain |
| `VITE_FIREBASE_PROJECT_ID` | Firebase project ID |
| `VITE_FIREBASE_STORAGE_BUCKET` | Firebase storage bucket |
| `VITE_FIREBASE_MESSAGING_SENDER_ID` | Firebase messaging sender ID |
| `VITE_FIREBASE_APP_ID` | Firebase app ID |
| `VITE_ADMIN_UID` | Firebase UID of the admin user |

### Backend (`backend/.env`)
| Variable | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key (fallback if no user key) |
| `FLASK_ENV` | `development` enables debug mode |
| `PORT` | Port for Flask/gunicorn (default 5000) |
| `FIREBASE_SERVICE_ACCOUNT_PATH` | Path to Firebase service account JSON |
| `FIREBASE_PROJECT_ID` | Firebase project ID |
| `ADMIN_UID` | Firebase UID of the admin user |
| `ENCRYPTION_KEY` | Fernet key for encrypting API keys at rest |
| `FRONTEND_ORIGIN` | GitHub Pages URL for CORS whitelist |
| `SCRAPERAPI_KEY` | ScraperAPI key (optional scraping fallback) |

---

## 6. Current React Routes

| Path | Component | Auth |
|---|---|---|
| `/login` | `Login.jsx` | Public |
| `/` | `HomeView.jsx` | Protected |
| `/search` | `SearchView.jsx` | Protected |
| `/discovery` | `DiscoveryView.jsx` | Protected |
| `/negotiate` | `NegotiateView.jsx` | Protected |
| `/listing` | `ListingView.jsx` | Protected |
| `/pipeline` | `PipelineView.jsx` | Protected |
| `/dashboard` | `DashboardView.jsx` | Protected |
| `/management` | `AdminDashboard.jsx` | Protected |
| `/howto` | `HowToView.jsx` | Protected |
| `/appointments` | `AppointmentsPage.jsx` | Protected |
| `/settings` | Redirect → `/management` | Protected |
| `/admin` | Redirect → `/management` | Protected |

---

## 7. Backend API Routes

| Method | Path | Description |
|---|---|---|
| GET | `/api/health` | Health check |
| GET | `/api/geocoder?q=` | Geocode location string → lat/lon (Photon/Nominatim, 1h cache) |
| POST | `/api/import` | Scrape Wallapop listing URL → title, price, images, seller, condition |
| POST | `/api/discovery` | Multi-platform DDG search (Phase 1): accepts `platforms` array; runs one DDG search per platform in parallel; scrapes results with `__NEXT_DATA__` + OG meta fallback; deduplicates by normalized URL; location-filters; returns `{ results, platform_errors, platform_counts }` |
| POST | `/api/analyze-listing` | AI analysis: deal score, negotiation scripts, market data, red flags |
| POST | `/api/batch-analyze` | Score multiple listings for flip potential |
| POST | `/api/discussion/generate` | Negotiation assistant: analyze chat history, suggest next reply |
| POST | `/api/generate-listing` | Generate Wallapop resale listing (title, description, tags, pricing) |
| POST | `/api/score-deals` | Score batch of deals as deal/fair/pass |
| GET | `/api/user/me` | Get current user profile + settings |
| POST | `/api/user/api-key` | Store encrypted personal Anthropic API key |
| DELETE | `/api/user/api-key` | Remove personal API key |
| PATCH | `/api/user/settings` | Update user settings (tone, language, locations, custom prompts) |
| GET | `/api/user/history` | Get last 20 search history entries |
| POST | `/api/user/history` | Save search action to history |
| GET | `/api/appointments` | Get all appointments for user (sorted by date) |
| POST | `/api/appointments` | Create appointment (optionally sync to Google Calendar) |
| PATCH | `/api/appointments/<id>` | Update appointment fields |
| DELETE | `/api/appointments/<id>` | Delete appointment (removes from Google Calendar if synced) |
| PATCH | `/api/user/platform-preferences` | Update single platform enabled flag; seeds defaults if field missing; returns `{ success, platformPreferences }` |
| GET | `/api/user/keyword-variants` | Get user's keyword variant groups (seeds 9 AI defaults on first call) |
| POST | `/api/user/keyword-variants` | Create a new keyword variant group |
| PATCH | `/api/user/keyword-variants/<id>` | Update trigger, variants, or enabled; bumps lastEditedAt |
| DELETE | `/api/user/keyword-variants/<id>` | Remove a keyword variant group |
| GET | `/api/admin/preauthorized` | List all pre-authorized email docs (admin only) |
| POST | `/api/admin/preauthorized` | Add a pre-authorized email; validates format, lowercases, rejects duplicates (admin only) |
| DELETE | `/api/admin/preauthorized/<id>` | Remove a pre-authorized email doc (admin only) |
| GET | `/api/admin/stats` | System-wide stats (admin only) |
| GET | `/api/admin/users` | List all users (admin only) |
| PATCH | `/api/admin/users/<uid>` | Update user access/cap/role (admin only) |
| POST | `/api/admin/set-key` | Set shared Anthropic API key (admin only) |

---

## 8. Firestore Schema

### `users` collection
Document ID = Firebase UID

| Field | Type | Description |
|---|---|---|
| `uid` | string | Firebase user UID |
| `email` | string | User email |
| `displayName` | string | Display name |
| `photoURL` | string | Avatar URL |
| `role` | string | `"admin"` or `"user"` |
| `hasSharedAccess` | boolean | Can use shared API key |
| `dailyCap` | number | Max AI calls per day |
| `usageToday` | number | Calls used today |
| `totalUsage` | number | Lifetime API calls |
| `api_key` | string | Encrypted personal Anthropic key |
| `default_tone` | string | `friendly`, `firm`, `curious` |
| `preferred_language` | string | UI language (`en`, `es`) |
| `negotiation_language` | string | Language for AI messages |
| `locations` | array | Saved locations for distance calc |
| `saved_searches` | array | Saved discovery searches |
| `recent_searches` | array | Recent discovery searches |
| `custom_negotiation_prompt` | string | Custom AI prompt for negotiation |
| `custom_listing_prompt` | string | Custom AI prompt for listing generation |
| `custom_batch_prompt` | string | Custom AI prompt for batch analysis |
| `custom_discussion_prompt` | string | Custom AI prompt for discussion helper |
| `keywordVariants` | array | Keyword variant groups — see shape below |
| `platformPreferences` | map | `{ wallapop, vinted, milanuncios, ebay_es }` — all boolean, defaults to `true`; seeded on first `GET /api/user/me` if missing |

**`history` item `platform` field:** `platform: string` (default `"wallapop"`); treat missing as `"wallapop"` on read — no migration needed.
**Pipeline deal `platform` field:** `platform: string` (default `"wallapop"`); treat missing as `"wallapop"` on read — no migration needed.

**`keywordVariants` item shape:**
```json
{
  "id": "uuid",
  "trigger": "TV",
  "variants": ["TV 40\"", "TV 55\"", "smart TV"],
  "enabled": true,
  "createdAt": "ISO string",
  "lastEditedAt": "ISO string",
  "source": "user | ai"
}
```
9 AI defaults seeded on first `GET /api/user/keyword-variants` if array is empty.
| `createdAt` | timestamp | Account creation time |
| `lastLogin` | timestamp | Last login time |

### `users/{uid}/appointments` subcollection
Document ID = auto-generated

| Field | Type | Description |
|---|---|---|
| `title` | string | Appointment name |
| `start` | ISO string | Start datetime |
| `end` | ISO string | End datetime |
| `location` | string | Meeting location |
| `phone` | string | Seller phone |
| `description` | string | Notes |
| `deal_id` | string | Linked pipeline deal ID |
| `deal_title` | string | Linked deal name |
| `type` | string | `inspection`, `handover`, `meeting` |
| `google_event_id` | string | Google Calendar event ID (if synced) |
| `created_at` | ISO string | Creation timestamp |

### `users/{uid}/history` subcollection
Document ID = item_id or auto-generated

| Field | Type | Description |
|---|---|---|
| `item_id` | string | Wallapop item ID |
| `title` | string | Listing title |
| `price` | number | Listed price |
| `score` | number | AI deal score |
| `action` | string | `Discarded`, `Pipeline`, etc. |
| `images` | array | Image URLs |
| `url` | string | Wallapop listing URL |
| `timestamp` | ISO string | When saved |

### `preauthorized_emails` collection
Document ID = auto-generated

| Field | Type | Description |
|---|---|---|
| `email` | string | Lowercase email address |
| `sharedKeyEnabled` | boolean | Whether shared AI key access is granted |
| `dailyCap` | number | Daily AI call cap for the new user |
| `note` | string | Optional admin label |
| `addedAt` | ISO string | When the doc was created |
| `addedBy` | string | UID of the admin who added it |

Doc is **consumed on first sign-in**: `AuthContext` checks this collection after Google sign-in, applies fields to the new user doc, then deletes the preauth doc.

### `config` collection
Document ID = `"keys"`

| Field | Type | Description |
|---|---|---|
| `anthropic_shared` | string | Encrypted shared Anthropic API key |

---

## 9. Do Not Touch

- `frontend/src/firebase.js` — Firebase initialization, do not modify
- `frontend/src/context/AuthContext.jsx` — Auth flow, do not modify without explicit instruction
- `backend/services/auth.py` — Token verification + Firebase init, do not modify
- `backend/services/encryption.py` — Key encryption logic, do not modify
- `.github/workflows/deploy-frontend.yml` — CI/CD pipeline, do not modify without explicit instruction
- `render.yaml` — Render deployment config, do not modify without explicit instruction
- `frontend/vite.config.js` — Vite + base path config, do not modify
- Never modify `CLAUDE.md` itself unless explicitly asked to update it

---

## 10. Pipeline Data Storage

**Important:** The Pipeline is stored in `localStorage` (key: `flipops_pipeline`), NOT Firestore.
All pipeline CRUD goes through the `usePipeline()` hook (`frontend/src/hooks/usePipeline.js`).
Backend has no pipeline storage — `POST /api/score-deals` only scores, it does not persist.

Deal object shape:
```js
{
  id: string,          // "deal_{timestamp}_{random}"
  product: string,
  initial_price: number,
  target_buy: number,
  actual_buy: number,
  target_sell: number,
  actual_sell: number,
  status: 'Watching' | 'Negotiating' | 'Bought' | 'Listed' | 'Sold',
  url: string,
  slug: string,
  notes: string,
  analysis: object,    // full AI analysis result
  seller: object,      // seller info from Wallapop
  created_at: ISO string,
  updated_at: ISO string,
  sold_at: ISO string  // only when status = Sold
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rom946) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
