## gwmart

> **GWMart** is a specialized marketplace application for **Guild Wars 1**. It allows users to list and trade items with detailed metadata specific to the game (attributes, requirements, skins, mods).

# GWMart Project Context

## Overview
**GWMart** is a specialized marketplace application for **Guild Wars 1**. It allows users to list and trade items with detailed metadata specific to the game (attributes, requirements, skins, mods).

## Technology Stack
-   **Framework**: Next.js 14+ (App Router)
-   **Language**: TypeScript
-   **Database**: Supabase (PostgreSQL)
-   **Styling**: TailwindCSS
-   **Authentication**: Supabase Auth (Discord OAuth provider)
-   **Testing**: Vitest

## Architecture Highlights
### Database Schema
The database uses a **Normalized Class Table Inheritance** pattern to handle the complex variety of GW1 items:
-   **`listings`**: Parent table for all items (price, seller, type).
-   **`details_*`**: Child tables for specific attributes (e.g., `details_weapons`, `details_offhands`) linked by 1:1 relationship.
-   **`ref_skins`**: Source of truth for item definitions (names, icons), scraped from Guild Wars Wiki.
-   **`ref_mods`**: Upgrade components reference.

### Client-Side Filtering (Updated Dec 2025)
To address performance concerns with server-side round trips:
-   **Strategy**: "Fetch Once, Filter Instantly". The App fetches a batch of active listings (server-side) and passes them to the client.
-   **`ListingsClient.tsx`**: Handles all filtering logic (Search, Category, Price, Requirements, Perfect Checks) in memory.
-   **URL Sync**: Uses `window.history.replaceState` to keep the URL shareable without triggering page reloads.

### Key Logic Locations
-   **`src/lib/skinService.ts`**: Handles fetching and caching of item skins. Uses normalization to map Wiki categories to the project's 10-category taxonomy.
-   **`src/lib/utils/listingTransforms.ts`**: Utilities for transforming raw DB shapes into UI components.
-   **`src/lib/validation.ts`**: Zod schemas for validating listings.
-   **`src/components/listings/ListingsClient.tsx`**: Main entry point for the listings grid, handling state and logic.

## Developer Workflows
### Verification
**Command**: `npm run verify`
Runs `scripts/verify-consistency.ts` to ensure:
-   `types.ts` enums match official GW1 attributes.
-   `constants.ts` mappings match the scraper configuration.
-   Internal consistency of subcategory mappings.

### Scraping
**Command**: `npm run scrape`
Runs `scripts/scrapers/scrape.ts` to:
1.  Discover items from the Wiki.
2.  Populate `ref_skins` in Supabase.
3.  Fetch and cache icons via a proxy.

## Recent Updates (Dec 2025)
-   **Performance Refactor**: Moved from Server-Side Filtering (slow interactions) to Client-Side Filtering (instant).
-   **Layout**: Introduced `src/app/listings/layout.tsx` for persistent WTS/WTB toggles.
-   **Listing Freeze Fix**: Resolved an issue where `CreateListingModal` would hang indefinitely. Added timeout logic.
-   **Icon Loading**: Improved `skinService` and scraper to reliably load and cache icons, handling legacy category names.

## MCP Configuration
-   **Supabase MCP Server**: Configured in `.mcp.json` to run locally via `node` (bypassing `npx`).
-   **Requirement**: Requires `SUPABASE_ACCESS_TOKEN` environment variable to be set.
-   **Package**: `@supabase/mcp-server-supabase` (installed as dev dependency).

## 10-Category Taxonomy
The project uses a strict taxonomy defined in `Item-Categories.md` (and enforced in DB constraints):
1.  Martial Weapon
2.  Caster Weapon (Staff, Wand, Focus)
3.  Shield
4.  Collectible (Miniature, etc.)
5.  Consumable
6.  Material
7.  Dye
8.  Skill Tome
9.  Container
10. Upgrade

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/natsilent30)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/natsilent30)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
