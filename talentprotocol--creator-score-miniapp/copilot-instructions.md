## changelog

> When package.json version changes

# Generate Changelog

When package.json version changes:

1. **Extract version**: Get major.minor (e.g., "v2.1" from "2.1.0")
2. **Generate content**: Create summary + detailed changelog from git commits
3. **Create entry**: Add to Notion Product Changelog database: https://www.notion.so/talentprotocol/d8921ac6c484431dad428039279898aa
4. **Set properties**:
   - Title: "Creator Score vX.Y"
   - App: "Creator Score" 
   - Version: "vX.Y"
   - Release Date: today (UTC)
   - Summary: one-liner
   - Icon: 🎁 (default) or 🎠💎🏆

## Changelog Structure
- **New Features**: New functionality added
- **Improvements**: Enhanced existing features
- **Bug Fixes**: Issues resolved
- **Breaking Changes**: Incompatible changes

## Important Notes
- Audience of the changelog is non-technical 
- Never create standalone page; use database parent_id
- If duplicate vX.Y exists, update instead of creating
- Summary is page property only, not in content body
- Search existing entries before creating new ones

---
> Source: [talentprotocol/creator-score-miniapp](https://github.com/talentprotocol/creator-score-miniapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
