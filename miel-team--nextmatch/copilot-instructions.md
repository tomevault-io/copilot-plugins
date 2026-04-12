## nextmatch

> Database is single source of truth


## Database Authority Rules

- Database is the single source of truth.
- Do not store persistent filters in localStorage.
- Search filters must be read from user_search_preferences.
- SmartMatches must always load preferences from DB.
- Never maintain multiple sources of filter state.
- All preference changes must invalidate smart_match_cache.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MIEL-TEAM)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MIEL-TEAM)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
