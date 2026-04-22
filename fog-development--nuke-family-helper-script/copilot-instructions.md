## nuke-family-helper-script

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a userscript for the Nuke Family faction in the online game Torn. The script enhances the game interface with faction management tools including shitlist management, payout distribution, and contract tracking. The script integrates with the Nuke.Family API and requires faction leadership permissions.

## Development Workflow

### Local Development Setup

For development, use the `dev.user.js` file which loads the main script from a local file path:
```javascript
// @require file://C:\Users\Justin\Documents\Coding\nuke-family-helper-script\nuke-family-helper.user.js
```

### Debug Mode
Enable debug mode by setting `const debug = true;` in `nuke-family-helper.user.js:22`

### Testing Environment
The dev script includes test match patterns:
- `https://nuke.family/auth/token-generation*`
- `http://nuke.test/auth/token-generation*`

## Code Architecture

### Core Components

**PageType Enum** (`nuke-family-helper.user.js:234`): Defines different page types (Profile, Faction, etc.)

**SettingsManager** (`nuke-family-helper.user.js:324`): Handles API token storage and retrieval using GM_getValue/GM_setValue

**Main Observer Pattern** (`nuke-family-helper.user.js:745`): Uses MutationObserver to detect page changes and inject appropriate UI elements

### Key Features

1. **Shitlist Management**
   - `getShitList()` - Fetches shitlist entries with caching
   - `getShitListCategories()` - Fetches available categories
   - Caching system with configurable expiration (default 60 minutes)

2. **Payout System**
   - `getPlayerPayoutList()` - Cash payouts to faction members
   - `getPlayerXanaxPayoutList()` - Xanax distribution system
   - Balance suggestion integration

3. **Contract Management**
   - `getContracts()` - Active contract display
   - `checkAndInsertActiveContract()` - UI injection for contract info

### Data Flow

1. **API Communication**: Uses `GM_xmlhttpRequest` with Bearer token authentication
2. **Caching**: localStorage with timestamp-based expiration
3. **UI Injection**: DOM manipulation with custom CSS classes (nfh-* prefix)
4. **Page Detection**: URL-based page type identification

### API Endpoints

Base URL determined by SettingsManager:
- `/shit-lists` - Shitlist entries
- `/shit-list-categories` - Category definitions  
- `/user/get-own-roles` - User permission roles
- `/payout/get-payout-table` - Payout information
- `/contracts/get_contracts` - Contract data

### TornPDA Compatibility

The script includes a polyfill for TornPDA mobile app (`nuke-family-helper.user.js:32-150`) that provides GM_* API compatibility when running in the mobile environment.

### Styling Convention

All custom elements use the `nfh-` CSS class prefix to avoid conflicts with the game's styling.

### Error Handling

Critical data operations are wrapped in try-catch blocks with user-facing alerts for cache corruption or API failures.

## Key Patterns

- **Async Data Fetching**: Promise-based with GM_xmlhttpRequest
- **DOM Waiting**: `waitForElm()` utility for dynamic content
- **Element Creation**: Dedicated builder functions for consistent UI components
- **Logging**: `LogInfo()` function with timestamp formatting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fog-Development) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
