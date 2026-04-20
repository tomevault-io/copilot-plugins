## tabcycle

> Auto-generated from all feature plans. Last updated: 2026-02-12

# TabCycle Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-12

## Active Technologies
- JavaScript (ES2022+), no transpiler (same as feature 001) + Chrome Extension APIs — adds `chrome.bookmarks` to existing `chrome.tabs`, `chrome.tabGroups`, `chrome.storage`, `chrome.alarms`, `chrome.webNavigation` (002-bookmark-closed-tabs)
- `chrome.storage.local` — extends existing `v1_settings` with bookmark fields; adds new `v1_bookmarkState` key for folder ID tracking (002-bookmark-closed-tabs)

- JavaScript (ES2022+), no transpiler needed (Chrome supports modern JS natively) + Chrome Extension APIs (`chrome.tabs`, `chrome.tabGroups`, `chrome.storage`, `chrome.alarms`, `chrome.windows`, `chrome.webNavigation`). No third-party runtime dependencies. (001-manage-tab-lifecycle)

## Project Structure

```text
src/
tests/
```

## Commands

npm test && npm run lint

## Code Style

JavaScript (ES2022+), no transpiler needed (Chrome supports modern JS natively): Follow standard conventions

## Recent Changes
- 002-bookmark-closed-tabs: Added JavaScript (ES2022+), no transpiler (same as feature 001) + Chrome Extension APIs — adds `chrome.bookmarks` to existing `chrome.tabs`, `chrome.tabGroups`, `chrome.storage`, `chrome.alarms`, `chrome.webNavigation`

- 001-manage-tab-lifecycle: Added JavaScript (ES2022+), no transpiler needed (Chrome supports modern JS natively) + Chrome Extension APIs (`chrome.tabs`, `chrome.tabGroups`, `chrome.storage`, `chrome.alarms`, `chrome.windows`, `chrome.webNavigation`). No third-party runtime dependencies.

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/grothkopp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
