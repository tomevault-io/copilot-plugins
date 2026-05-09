## claude-prayer-statusline

> `CACHE_VERSION` in `prayer-time-addon.js` **must be bumped** whenever:

# Claude Code Instructions

## Cache versioning

`CACHE_VERSION` in `prayer-time-addon.js` **must be bumped** whenever:
- Prayer time calculation logic changes
- The cache data structure changes
- The location source changes (e.g., IP → config file)

Bumping the version auto-invalidates all users' stale caches on the next run.
Current version: `3`

---
> Source: [AfreenLikeCaffeine/claude-prayer-statusline](https://github.com/AfreenLikeCaffeine/claude-prayer-statusline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
