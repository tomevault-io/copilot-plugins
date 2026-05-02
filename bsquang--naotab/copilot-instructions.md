## naotab

> Chrome Extension (Manifest V3) that turns browser tabs into a **personal knowledge base**. No server, no backend — everything runs inside the extension, data stored in `chrome.storage.local`.

# naoTab — Chrome Extension

## Project overview

Chrome Extension (Manifest V3) that turns browser tabs into a **personal knowledge base**. No server, no backend — everything runs inside the extension, data stored in `chrome.storage.local`.

UI language: **English**. Codebase comments: mixed EN/VI.

---

## File structure

```
naoTab/
├── manifest.json        # Manifest V3, permissions: tabs + storage + unlimitedStorage + scripting
├── popup.html/css/js    # Popup — view tabs, save, export
├── app.html/js          # Knowledge Base full-page
├── settings.html/js     # AI provider config page
├── build.sh             # Build script → naotab-vX.Y.Z.zip
├── core/
│   ├── schema.js        # ⚠️ Single source of truth for data structure — never remove fields
│   ├── storage.js       # Bookmarks CRUD + settings (reads only, no business logic)
│   ├── ai.js            # callAI() + suggestTags()
│   └── export.js        # exportJSON / importJSON / exportObsidian / bookmarkToObsidianMd
│   └── sync/            # (future) drive.js, notion.js, etc.
├── vendor/
│   ├── d3.min.js        # D3.js v7.9.0 — bundled locally (CSP compliance)
│   └── jszip.min.js     # JSZip 3.10.1 — bundled locally (CSP compliance)
├── icons/               # icon16/48/128.png
├── README.md            # English README (links to VI version)
├── README.vi.md         # Vietnamese README (links to EN version)
└── CLAUDE.md            # This file
```

---

## Architecture & Data flow

```
popup.js  ──────────────────────┐
settings.js ────────────────────┤──► storage.js ──► chrome.storage.local
app.js  ────────────────────────┘         │
                                          └──► External AI API (optional)
```

### Data schema — bookmark object

```json
{
  "id": "1712345678901",
  "url": "https://...",
  "title": "Page title",
  "reason": "Why user saved this",
  "summary": "AI-generated or manual summary",
  "tags": ["rust", "async", "performance"],
  "favIconUrl": "https://...",
  "pageMeta": {
    "description": "og:description or meta description",
    "ogTitle": "og:title",
    "ogType": "article",
    "keywords": "meta keywords",
    "author": "meta author",
    "siteName": "og:site_name",
    "ogImage": "og:image URL",
    "lang": "html lang attribute",
    "canonical": "canonical URL"
  },
  "savedAt": "2024-01-01T00:00:00.000Z",
  "updatedAt": "2024-01-01T00:00:00.000Z"
}
```

Note: `status` field removed from UI (was: unread/reading/done/revisit). `pageMeta` is `null` if meta read failed or tab was a chrome:// page.

### Data schema — settings object

```json
{
  "aiEnabled": false,
  "aiBaseUrl": "https://api.openai.com/v1",
  "aiApiKey": "sk-...",
  "aiModel": "gpt-4o-mini",
  "featTags": true,
  "featSummary": true
}
```

---

## Key files and responsibilities

### `core/schema.js`
⚠️ **Never remove or rename existing fields.** Only ADD new fields with a default value.

- `SCHEMA_VERSION` — current version integer, increment when adding fields
- `BOOKMARK_DEFAULTS` — canonical bookmark shape with all fields and defaults
- `SETTINGS_DEFAULTS` — canonical settings shape
- `migrateBookmark(raw)` — heals old bookmark objects: fills missing fields, drops removed ones
- `createBookmark(fields)` — builds a new bookmark object with all required fields

### `core/storage.js`
CRUD only. Loaded into popup, app, settings pages.

- `getBookmarks()` — read all bookmarks, runs `migrateBookmark()` on each (safe for old data)
- `saveBookmark(fields)` — save new, dedup by URL, calls `createBookmark()`
- `updateBookmark(id, changes)` — partial update
- `deleteBookmark(id)` — delete by id
- `getSettings()` / `saveSettings(settings)` — read/write settings

### `core/ai.js`
- `callAI(title, url, pageMetaText)` — call AI API, returns `{tags, summary}` or throws
- `suggestTags(title, url)` — offline keyword + domain matching, returns tag array

### `core/export.js`
- `exportJSON()` / `importJSON(jsonString)` — full backup/restore
- `bookmarkToObsidianMd(bookmark)` / `exportObsidian()` — Obsidian .md export

### `popup.js`
- `loadTabs()` — queries all tabs + bookmarks, marks saved URLs
- `getPageMeta(tabId)` — uses `chrome.scripting.executeScript` to read SEO meta tags; returns structured object with `_aiText` field for AI prompt
- `openSaveModal(tab)` — async, reads meta in background, shows/hides AI row
- Window save: calls `getPageMeta()` for each tab, shows `⏳ N/total` progress, saves `pageMeta`
- Single tab save: passes `pageMeta` to `saveBookmark()`
- **"💾 Tab this"** button — saves active tab directly from toolbar

### `app.js`
State: `allBookmarks`, `activeTag`, `excludedTags` (Set), `searchQuery`, `currentView` (default: `'graph'`), `editingId`, `panelId`

Key features:
- **Graph view** (default): D3 force simulation, single-click → node panel + highlight connected nodes, double-click → open URL, click background → reset. Node color: blue (has summary) vs grey (no summary)
- **List view**: card per bookmark, click card → node panel (links/buttons still work normally)
- **Node panel**: slide-in right panel with full details, editable fields, AI Suggest button (shown only when AI enabled), Connected nodes section (only within current filter), save/delete/open-url actions
- **Sidebar**: split into Tags (top, scrollable) + Node list (bottom, realtime). Node list updates with filter/search; click item → open panel; hover item → tooltip with title + summary
- **Graph tooltip**: shows title, URL, summary (blue border), reason, tags on hover
- **AI Batch**: `✨ AI All` button in topbar — processes only visible nodes without a summary, shows confirm with counts, progress bar + notify in bottom-right, nodes update color live
- **Exclude tags**: tag pills in sidebar have `✕` button on hover → excluded tags shown as strikethrough red, not used for graph edges
- **Delete all**: button in topbar with confirmation
- **Refresh**: 🔄 button in topbar
- **Obsidian export**: JSZip → .zip of .md files with YAML frontmatter

### `settings.js`
- PRESETS: openai, claude, ollama, groq, openrouter, custom
- Detects active preset by matching saved URL
- Test connection: detects Anthropic vs OpenRouter vs standard; adds `HTTP-Referer` + `X-Title` headers for OpenRouter
- Save settings to `chrome.storage.local`

---

## AI Integration

`callAI()` in `storage.js` supports 2 formats:

**OpenAI-compatible** (OpenAI, Groq, Ollama, OpenRouter, etc.):
- Endpoint: `{baseUrl}/chat/completions`
- Header: `Authorization: Bearer {apiKey}`
- OpenRouter also needs: `HTTP-Referer: https://github.com/bsquang/naotab` + `X-Title: naoTab`

**Anthropic native**:
- Detect: `baseUrl.includes('anthropic.com')`
- Endpoint: `{baseUrl}/messages`
- Header: `x-api-key: {apiKey}` + `anthropic-version: 2023-06-01`

**Token efficiency**: AI receives page meta tags (~75 tokens) instead of body text (~750 tokens) — 10× cheaper. `_aiText` field is a preformatted string built from `pageMeta` fields.

---

## Permissions

| Permission | Reason |
|---|---|
| `tabs` | Read title, URL, favIconUrl of all tabs |
| `storage` | Save bookmarks and settings to chrome.storage.local |
| `unlimitedStorage` | Remove 10MB default cap |
| `scripting` | Execute script in tabs to read meta tags |
| `host_permissions: <all_urls>` | Allow meta reading on any tab |

---

## Conventions when working with Claude

- Claude edits files directly in this folder
- User reloads extension via 🔄 button in popup
- `storage.js` is shared — when adding functions, update popup.js and/or app.js as needed
- No ES modules (`import/export`) — Chrome Extension uses classic script loading
- No inline scripts in HTML — CSP requires all JS in external `.js` files
- No CDN scripts — bundle all libraries locally for CSP compliance
- Commit only when user explicitly asks
- All UI text in **English**

## Roadmap

- [ ] Google Drive sync (`core/sync/drive.js` — branch: feature/google-drive-sync)
- [ ] Semantic / AI-powered search
- [ ] AI Cluster Summary (summarize all visible nodes as a group)
- [ ] Dark mode
- [ ] Duplicate tab detector
- [ ] AI-powered bookmark grouping
- [ ] Browser history integration

---
> Source: [bsquang/naotab](https://github.com/bsquang/naotab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
