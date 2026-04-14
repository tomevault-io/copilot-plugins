## hpp-final-mvp

> Guidance for AI assistants working in this repository.

# CLAUDE.md — HPP Industry Feed

Guidance for AI assistants working in this repository.

---

## Project Overview

**HPP Industry Feed** is a static news aggregation dashboard for the High Pressure Processing (HPP) food-technology industry. It surfaces relevant RSS articles from 25+ sources and lets users copy pre-formatted LinkedIn posts for each article ("Approve" workflow).

The architecture is intentionally minimal:
- **No framework, no build step, no external dependencies** — pure HTML/CSS/JS on the frontend, Python standard library only on the backend.
- Data is refreshed server-side every 2 hours by a GitHub Actions job that writes `feeds.json` and patches `index.html` in-place.
- The entire repository root is served as a static site via GitHub Pages.

---

## Repository Layout

```
HPP_Final_MVP/
├── index.html                      # Main SPA — all HTML, CSS, and JS in one file
├── script.js                       # Standalone ROI calculator (separate page/widget)
├── feeds.json                      # Auto-generated; updated by GitHub Actions
├── scripts/
│   └── fetch_feeds.py              # Feed aggregation script (Python 3, stdlib only)
└── .github/
    └── workflows/
        ├── fetch-feeds.yml         # Runs fetch_feeds.py on a 2-hour cron schedule
        └── enable-pages.yml        # Deploys the repo root to GitHub Pages
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML5, CSS3, JavaScript (ES2020+) |
| Backend script | Python 3 (stdlib only — `urllib`, `xml.etree`, `json`, `re`, `datetime`) |
| CI/CD | GitHub Actions |
| Hosting | GitHub Pages (static) |
| Data storage | `feeds.json` (flat JSON file, no database) |

There are **no package managers, no `requirements.txt`, no `package.json`**. Do not add external dependencies.

---

## Data Flow

```
GitHub Actions (every 2 hours, on push to main, or manual dispatch)
  └─► python3 scripts/fetch_feeds.py
        ├─► Fetches 25+ RSS / Atom feeds over HTTPS
        ├─► Filters, deduplicates, and sorts up to 50 articles
        ├─► Writes  feeds.json
        └─► Patches inline JS block inside index.html
              │
              ▼
          GitHub commits both files → pushes to main [skip ci]
              │
              ▼
          GitHub Pages serves updated index.html
              │
              ▼
          Browser renders articles client-side (no further HTTP calls to own backend)
```

The browser also has a **client-side fallback**: if `__FEEDS__` data is stale or unavailable, `loadFeeds()` fetches the same Google News RSS URLs directly via three chained CORS proxy services (`corsproxy.io` → `api.allorigins.win` → `thingproxy.freeboard.io`). This path requires no server involvement.

---

## Key Files in Detail

### `scripts/fetch_feeds.py`

- **Entry point:** Run directly — `python3 scripts/fetch_feeds.py` — from the repo root.
- **FEEDS list** (line 43): Each entry is a dict:
  ```python
  {"label": "Human-readable name", "url": "RSS URL", "filtered": bool, "fetch_limit": int}
  ```
  - `filtered=False` — Google News searches (already topic-focused; no keyword filtering).
  - `filtered=True` — Broad publication feeds (Food Safety News, New Food Magazine, Food Dive); must pass `is_relevant()`.
- **RELEVANCE_KEYWORDS** (line 20): Controls what gets accepted from filtered feeds. Add/remove strings here to tune coverage. All checks are case-insensitive substring matches on `title + description`.
- **MAX_RELEVANT_PER_FILTERED_FEED = 5** (line 140): Cap on articles taken from each filtered feed.
- **MAX_TOTAL = 50** (line 141): Global article cap after deduplication and date-sort.
- **Deduplication key** (line 241): `link.split("?")[0].rstrip("/")` — strips query strings to catch identical articles from multiple sources.
- **Output 1 — `feeds.json`** (line 257): Written to the repo root. Schema:
  ```json
  {
    "updated": "<ISO 8601 UTC timestamp>",
    "articles": [
      {
        "source": "Feed Label",
        "title": "Article title",
        "link": "https://...",
        "description": "Truncated to 400 chars, HTML stripped",
        "pubDate": "RFC 2822 or ISO 8601 date string"
      }
    ]
  }
  ```
- **Output 2 — `index.html` inline patch** (line 265): Replaces the block between `/* __FEEDS_DATA_START__ */` and `/* __FEEDS_DATA_END__ */` with a JS variable assignment. **These marker comments must not be removed from `index.html`.**
- **Exit behaviour:** Always exits 0. Feed failures are best-effort — they warn to stderr but never block CI.

### `index.html`

Single-file SPA. Contains three distinct sections:

1. **`<style>` block** — All CSS. Mobile-responsive with Flexbox. Brand colour: `#0a3d62`. No external stylesheets.
2. **`<body>` HTML** — Minimal DOM:  `#status` (article count line), `#feed-container` (card grid), `#toast` (clipboard notification).
3. **`<script>` block** — All client-side JavaScript. Key functions:

| Function | Purpose |
|---|---|
| `loadFeeds()` | Main entry point; fetches all RSS feeds concurrently via `Promise.allSettled` |
| `fetchFeed(feed)` | Tries each CORS proxy in order; throws on total failure |
| `parseXmlItems(xmlStr, label)` | Parses RSS 2.0 / Atom XML into article objects |
| `renderArticle(article)` | Creates a `.article-card` DOM element |
| `renderSkeletons(n)` | Shows shimmer placeholders while loading |
| `buildLinkedInPost(...)` | Generates the clipboard text for the "Approve" button |
| `handleApprove(btn, post)` | Writes to clipboard; falls back to `execCommand` for older browsers |
| `showToast(msg)` | Displays the fixed-position toast notification |
| `stripHtml(raw)` | Uses `DOMParser` to safely strip tags and decode entities |
| `formatDate(dateStr)` | Formats a date string to "Mon DD, YYYY" |

**Critical constraint:** The `/* __FEEDS_DATA_START__ */` and `/* __FEEDS_DATA_END__ */` markers inside the `<script>` block must remain intact. `fetch_feeds.py` uses them as injection points.

**Client-side article limit:** `MAX_ARTICLES = 40` (line 241).

**CORS proxy chain** (line 265):
1. `corsproxy.io` — primary
2. `api.allorigins.win` — secondary; returns JSON wrapper, extracted via `.json().contents`
3. `thingproxy.freeboard.io` — tertiary

### `script.js`

Standalone ROI calculator for HPP machinery. Attaches to a form with id `roiForm`. Hard-coded assumptions:
- Machine cost: £850,000
- Waste recovery: 20% (shucking yield gain)
- Annual operating costs: £25,000
- Tier threshold: £15,000,000 revenue → Professional tier

This file is independent of the feed system. It is linked from a separate page (not `index.html`).

### `.github/workflows/fetch-feeds.yml`

- **Triggers:** `schedule` (every 2 hours), `workflow_dispatch` (manual), `push` to `main`.
- **Permissions:** `contents: write` (to commit updated files back to the repo).
- **Key step:** Runs `python3 scripts/fetch_feeds.py`, then stages `feeds.json` and `index.html`. Commits only if the diff is non-empty, with message `chore: refresh feed articles [skip ci]` — the `[skip ci]` tag prevents a re-trigger loop.

### `.github/workflows/enable-pages.yml`

- **Triggers:** `push` to `main`, `workflow_dispatch`.
- **Deploys** the entire repository root (`.`) as the GitHub Pages artifact.
- Uses the `pages` concurrency group with `cancel-in-progress: false` to avoid partial deployments.

---

## Running Locally

### Feed aggregation

```bash
# From the repo root (no dependencies to install):
python3 scripts/fetch_feeds.py
```

This updates `feeds.json` and patches `index.html`. Expect some ERR lines for feeds that block automated requests — this is normal.

### Viewing the site

Open `index.html` directly in a browser. The client-side JavaScript will attempt to fetch live RSS via CORS proxies if the inline `__FEEDS__` data is outdated. For a more accurate local preview, serve over HTTP:

```bash
python3 -m http.server 8080
# Then open http://localhost:8080
```

---

## GitHub Actions Workflow

| Trigger | Workflow | Effect |
|---|---|---|
| Push to `main` | `fetch-feeds.yml` | Refreshes feeds; commits if changed |
| Push to `main` | `enable-pages.yml` | Redeploys GitHub Pages |
| Every 2 hours | `fetch-feeds.yml` | Scheduled refresh |
| Manual dispatch | Either | On-demand trigger |

The automated feed commit uses `[skip ci]` to avoid a deployment loop.

---

## Adding or Modifying Feeds

Edit `FEEDS` in `scripts/fetch_feeds.py`:

- **Google News feed:** Append a dict with `filtered: False`. URL-encode the search query.
- **Direct publication feed:** Append a dict with `filtered: True`. The article must pass `is_relevant()`.
- **Relevance tuning:** Add/remove strings from `RELEVANCE_KEYWORDS`. Match is substring, case-insensitive.
- **Volume control:** Adjust `fetch_limit` per feed, `MAX_RELEVANT_PER_FILTERED_FEED`, or `MAX_TOTAL`.

Matching feed URLs and labels must also be kept in sync with the `RSS_FEEDS` array inside `index.html` if client-side fallback coverage matters.

---

## Conventions and Constraints

1. **Zero external dependencies.** `fetch_feeds.py` imports only stdlib. Do not add `pip install` steps or `requirements.txt`. The frontend uses no npm packages.
2. **Preserve the inline data markers.** `/* __FEEDS_DATA_START__ */` and `/* __FEEDS_DATA_END__ */` in `index.html` are required for the CI patch step. Never remove or rename them.
3. **Always exit 0 in `fetch_feeds.py`.** Feed errors are non-fatal. The script must not fail the GitHub Actions job.
4. **No build pipeline.** There is no transpilation, bundling, or minification. Edit source files directly. What you write is what ships.
5. **Single-file frontend.** HTML, CSS, and JS all live in `index.html`. Do not split them without a specific reason — the GitHub Pages deploy serves the root directory and keeping everything in one file avoids path issues.
6. **`script.js` is independent.** It is not loaded by `index.html`. Do not merge them unless explicitly requested.
7. **Brand colour: `#0a3d62`** (dark navy). Secondary accent: `#27ae60` (success/copied state).
8. **`feeds.json` is a generated file.** Never hand-edit it; the next Actions run will overwrite it.
9. **`[skip ci]` on automated commits** prevents GitHub Actions from triggering itself in a loop.

---

## No Tests

There is no test suite. The only validation is:
- Python `try/except` in `fetch_feeds.py` around each feed fetch (errors are logged, not raised).
- Browser console warnings for failed feeds in the client-side path.
- GitHub Actions logs (viewable in the Actions tab) for each scheduled run.

To validate a change to `fetch_feeds.py`, run it locally and inspect stdout/stderr output and the resulting `feeds.json`.

---

## Common Tasks

### Update the LinkedIn post template
Edit `buildLinkedInPost()` in `index.html` (around line 317). The hashtag string is at the end of the returned array.

### Change how many articles are shown
- Backend cap: `MAX_TOTAL` in `fetch_feeds.py` (line 141). Default: 50.
- Frontend cap: `MAX_ARTICLES` in `index.html` (line 241). Default: 40.

### Add a new food-safety keyword
Append to `RELEVANCE_KEYWORDS` in `fetch_feeds.py` (line 20). Lowercase, as matching is case-insensitive.

### Adjust the feed refresh schedule
Edit the `cron` expression in `.github/workflows/fetch-feeds.yml` (line 5). Current schedule: `0 */2 * * *` (top of every even hour, UTC).

### Change the ROI calculator assumptions
Edit the hard-coded constants at the top of the `submit` handler in `script.js` (lines 8–11, 16).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ghosty61)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ghosty61)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
