## marketing-extension

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

He4rt Analytics — a Chrome extension (Manifest V3) that passively intercepts Twitter/X GraphQL API responses to track community engagement. Designed to export structured JSON for ingestion into the He4rt Hub Laravel app.

No build step, no bundler, no dependencies. Load the folder directly in `chrome://extensions/` with developer mode.

## Architecture

The extension has a three-layer pipeline, split across execution contexts due to X.com's strict CSP:

```
interceptor.js (MAIN world)  →  content.js (ISOLATED world)  →  background.js (service worker)
patches fetch/XHR               bridges via postMessage          filters, consolidates, exports
runs in page context             runs in extension context        holds all state in memory
```

**Why two content scripts:** X.com CSP blocks inline scripts. `interceptor.js` needs `window.fetch` access (MAIN world). `content.js` needs `chrome.runtime` access (ISOLATED world). They communicate via `window.postMessage`.

**Data flow for a captured request:**
1. `interceptor.js` — fetch/XHR monkey-patch detects URL matching `/i/api/graphql/*/ENDPOINT`
2. Clones response, extracts endpoint name, posts payload via `postMessage`
3. `content.js` — receives message, forwards to background via `chrome.runtime.sendMessage`
4. `background.js` — stores raw payload, then processes based on endpoint type:
   - `UserTweets` → extracts tweets filtered by `store.trackedHandle`, deduplicates by `tweet_id`
   - `Favoriters` → extracts user list, links to tweet ID parsed from page URL
   - `UserByScreenName` → captures account profile metadata
   - Community replies (tweets replying to tracked handle from other users) are stored separately

**State:** All data lives in the `store` object in the service worker (volatile). Only `trackedHandle` persists via `chrome.storage.local`. When handle changes, all cached `UserTweets` payloads are reprocessed against the new filter.

## Key X/Twitter GraphQL Response Shapes

Tweet author data splits across two objects — getting this wrong silently produces empty fields:
- **Name/screen_name** → `result.core.name`, `result.core.screen_name`
- **Followers/stats** → `result.legacy.followers_count`, etc.
- **Avatar** → `result.avatar.image_url`
- **Protected** → `result.privacy.protected`
- **Relationships** → `result.relationship_perspectives.following`, `.followed_by`

Tweet data in `UserTweets` nests deeply with multiple instruction types:
- `TimelinePinEntry` → single pinned tweet at `instruction.entry.content.itemContent.tweet_results.result`
- `TimelineAddEntries` → array of entries, each either `TimelineTimelineItem` (single tweet) or `TimelineTimelineModule` (conversation thread with `.items[]`)
- Cursors and "who to follow" modules should be skipped

Tweet type detection: check `legacy.retweeted_status_result` (retweet), `legacy.in_reply_to_status_id_str` (reply), `legacy.is_quote_status + quoted_status_id_str` (quote), else original.

## Message Protocol

Popup ↔ Background communication uses `chrome.runtime.sendMessage` with an `action` string:

| Action | Direction | Purpose |
|---|---|---|
| `GRAPHQL_CAPTURED` | content → bg | New intercepted response |
| `SET_HANDLE` / `GET_HANDLE` | popup → bg | Track a handle (clears + reprocesses) |
| `GET_TWEETS` | popup → bg | Consolidated tweets for display |
| `GET_EXPORT` | popup → bg | Full structured JSON for download |
| `GET_ENDPOINTS` | popup → bg | Summary of all captured endpoints |
| `GET_ENDPOINT_PAYLOADS` | popup → bg | Raw payloads for a specific endpoint |
| `CLEAR_ALL` | popup → bg | Reset everything except handle |

All handlers return `true` to keep the message channel open for async responses.

## Export JSON Structure

The `GET_EXPORT` action returns a JSON object designed for direct Laravel ingestion:
- `tracked_account` — profile metadata
- `tweets[]` — all tweets from tracked handle with metrics, typed as original/retweet/reply/quote
- `community_replies[]` — replies to tracked handle from other users
- `favoriters_by_tweet` — map of tweet_id → user arrays
- `summary` — aggregated stats (totals, averages, top tweets, unique engagers)

---
> Source: [he4rt/marketing-extension](https://github.com/he4rt/marketing-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
