## linuxdo-scripts

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LinuxDo Scripts is a browser extension built with WXT framework and Vue 3 that enhances the LinuxDo forum experience. It provides features like AI assistance, bookmark management, content filtering, theme customization, and various UI improvements.

## Development Commands

```bash
# Development (Chrome)
npm run dev

# Development (Firefox)
npm run dev:firefox

# Build for production
npm run build

# Build for Firefox
npm run build:firefox

# Create distribution zip files
npm run zip
npm run zip:firefox

# Full publish workflow (build + zip + python script)
npm run publish

# TypeScript type checking
npm run compile

# Clean install
npm run rf && npm install
```

## Project Architecture

### Core Technologies
- **Framework**: WXT (Web Extension Framework) v0.19.13
- **UI Framework**: Vue 3 with TypeScript
- **Component Library**: Arco Design Vue + Element Plus
- **Styling**: Less + Sass
- **Build Tool**: Vite (via WXT)

### Entry Points Structure

```
entrypoints/
├── background.ts          # Background service worker - handles messaging, API proxies, click behavior
├── content.js             # Content script - injects Vue app into LinuxDo pages via shadow DOM
├── App.vue                # Main content script UI (floating settings button + menu)
├── popup/                 # Browser action popup
│   ├── App.vue
│   └── main.js
├── sidepanel/             # Side panel UI (reuses PopupViews)
│   ├── App.vue
│   └── main.js
├── PopupViews/            # Shared UI components for popup/sidepanel
│   ├── index.vue          # Main view router
│   └── components/        # Feature components (Bookmark, HotPosts, NewsPosts, LDC, etc.)
├── SettingComponents/     # Settings menu components
│   ├── AIConfig/          # AI API configuration (GPTconfig.vue)
│   ├── BasicSettings/     # Toggle switches for features
│   ├── CustomCSS/         # Custom CSS editor
│   ├── DataBackup/        # Import/export settings
│   ├── ThemeStyle/        # Theme customization
│   └── UserTags/          # User tagging system
├── bookmark/              # Bookmark management interface
│   ├── App.vue
│   └── components/
├── share/                 # Share/post export functionality
└── utilities/             # Shared utilities
    ├── indexedDBStorage.js    # IndexedDB wrapper for settings/bookmarks
    ├── settingsManager.js     # Settings management with caching
    ├── storageCompat.js       # Storage migration/compatibility
    ├── connectApi.js          # Connect.linux.do API wrapper
    ├── componentUpdateExample.js
    ├── post.ts                # Post utilities
    ├── storageDebugger.js     # Storage debugging
    └── url.ts                 # URL utilities
```

### Key Architecture Patterns

1. **Shadow DOM Injection**: Content script uses WXT's `createShadowRootUi` to inject Vue app without conflicting with page styles
2. **Background Message Routing**: Background script acts as central hub for:
   - AI API proxy (CORS bypass)
   - WebDAV requests
   - Connect.linux.do API calls
   - Tab-to-tab messaging
   - Click behavior management (popup vs sidepanel)
3. **Storage Strategy**:
   - Primary: IndexedDB via `indexedDBStorage.js` for large data (bookmarks, history)
   - Secondary: chrome.storage.local for settings
4. **Shared Components**: Popup and sidepanel share the same `PopupViews` components
5. **Settings Management**: Centralized settings with caching and migration support

### Feature Modules

- **AI Features**: GPT integration, topic summarization, smart replies
- **Bookmark System**: Category-based bookmarking with IndexedDB
- **Content Filtering**: User blocking, keyword filtering, time-based filtering
- **UI Enhancements**: Themes, custom CSS, display optimizations
- **Utilities**: Export to image/PDF, quick actions, level lookup

## Configuration Files

- **wxt.config.ts**: WXT framework configuration, manifest settings, Vite CSS options
- **package.json**: Dependencies, scripts, version management
- **tsconfig.json**: TypeScript config extending WXT's base config
- **.prettierrc**: Code formatting rules (120 char width, tabs, single quotes)

## Build Output

- Development: `.output/chrome-mv3/` (or `firefox-mv2/`)
- Production zips: Created via `npm run zip` commands
- Python script (`scripts/build.py`) auto-updates `version-log.md` based on `package.json` version

## Common Development Tasks

### Adding a New Feature
1. Create component in appropriate `entrypoints/` subdirectory
2. Import and register in `App.vue` or `PopupViews/index.vue`
3. Add settings toggle in `SettingComponents/BasicSettings/`
4. Update storage schema if needed in `indexedDBStorage.js`

### Debugging Content Script
- Content script runs in shadow DOM, inspect via DevTools Elements panel
- Look for `#linuxdoscripts` shadow root
- Background script logs visible in extension service worker panel

### Testing AI Features
- Configure API endpoint in settings (AIConfig/GPTconfig.vue)
- Background script provides proxy endpoints: `ai_api_proxy`, `ai_api_stream_proxy`

### Storage Migration
- Check `storageCompat.js` for version handling
- `indexedDBStorage.js` has `getSettingsWithMigration()` for safe upgrades

## Important Notes

- **Node Version**: Requires Node.js v22.12.0 (see `.nvmrc`)
- **Browser Compatibility**: Chrome (Manifest V3), Firefox (Manifest V2)
- **Permissions**: storage, sidePanel, tabs, host_permissions for http/https
- **Side Panel**: Default entry point, can be switched to popup via settings
- **IDCflare Support**: Also compatible with idcflare.com domain

## Testing Checklist

When making changes, verify:
- [ ] Content script injection works on linux.do pages
- [ ] Background message handlers respond correctly
- [ ] Settings persist across sessions
- [ ] Bookmarks save/load from IndexedDB
- [ ] AI API proxy handles both regular and streaming responses
- [ ] Popup/sidepanel UI renders correctly
- [ ] Theme changes apply without conflicts

---
> Source: [anghunk/linuxdo-scripts](https://github.com/anghunk/linuxdo-scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
