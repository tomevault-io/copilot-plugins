## study-track-teacher

> > Global rules in root CLAUDE.md also apply.

# Claude Code — Teacher Module Rules

> Global rules in root CLAUDE.md also apply.

## Structure

- Follow the existing feature-based structure in this module.
- Place new pages/components/hooks inside the appropriate feature folder.

## Static data

- Do not hardcode reusable static UI data inside pages/components.
- Always move reusable static data into a dedicated adjacent `*.data.js` file and import it.
  - Examples: select options, filters, status labels/colors, table column configs.

## Dates

- Always use `formatDateUz(...)` for date formatting in UI.
- Do not use `toLocaleDateString()` / `toLocaleString()` directly in UI rendering.

## Data fetching & caching

- Use TanStack Query for all API calls and caching.
- Reuse the existing query key conventions and invalidation patterns already used in this module.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yaxyobekuz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yaxyobekuz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
