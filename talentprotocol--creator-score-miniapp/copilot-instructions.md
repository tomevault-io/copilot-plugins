## versioning

> every time we make a relevant/sizable GitHub commit

# Versioning

## Commands
```bash
npm run version:patch    # 1.0.0 → 1.0.1
npm run version:minor    # 1.0.0 → 1.1.0  
npm run version:major    # 1.0.0 → 2.0.0
```

## Auto-Generated
- Build date automatically set during `npm run build`
- Footer shows: `Creator Score App v1.0.0 (2025.01.13)`
- Changelog entries auto-created in Notion when version changes

## Files
- `lib/version.ts` - Version utilities
- `package.json` - Version source
- `scripts/version-bump.js` - Version bumping script # Versioning

---
> Source: [talentprotocol/creator-score-miniapp](https://github.com/talentprotocol/creator-score-miniapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
