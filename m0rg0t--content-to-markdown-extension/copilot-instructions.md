## content-to-markdown-extension

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
npm run build        # Production build (outputs to dist/)
npm run build:dev    # Development build with source maps
npm run watch        # Watch mode for development
```

To test the extension: load `dist/` as an unpacked extension in `chrome://extensions/` (Developer mode).

## Architecture

This is a Chrome Manifest V3 extension that converts webpage content to Markdown using the Turndown library.

### Component Communication Flow

```
┌─────────────┐    messages     ┌──────────────────┐
│   Popup     │ ───────────────→│  Background      │
│ popup.ts    │                 │  (Service Worker)│
└─────────────┘                 └────────┬─────────┘
                                         │
┌─────────────┐                          │ chrome.tabs.sendMessage
│  Options    │                          ▼
│ options.ts  │                 ┌──────────────────┐
└─────────────┘                 │  Content Script  │
       │                        │  content.ts      │
       └── chrome.storage.sync ─┴──────────────────┘
```

- **Background (service worker)**: Sets up context menus, routes messages to content script
- **Content script**: Injected into all pages; performs DOM manipulation, conversion, and clipboard operations
- **Popup**: UI for triggering conversions; sends messages via background script
- **Options**: Settings UI; reads/writes to `chrome.storage.sync`

### Key Files

- `src/utils/converter.ts` - Core HTML-to-Markdown logic using Turndown; handles element exclusion, site-specific content selectors, and output formatting
- `src/utils/settings.ts` - Settings interfaces, storage operations, domain matching with wildcard support
- `src/utils/messages.ts` - TypeScript types for inter-component messaging

### Patterns

- **DOM cloning**: Content script clones the page DOM before modifying it to avoid affecting the live page
- **Selector accumulation**: Exclusion settings build a CSS selector array applied via `querySelectorAll().forEach(el => el.remove())`
- **Wildcard domain matching**: Site rules support `*.domain.com` patterns with automatic `www.` normalization
- **Promise wrappers**: Chrome callback APIs wrapped in Promises for async/await usage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/m0rg0t)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/m0rg0t)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
