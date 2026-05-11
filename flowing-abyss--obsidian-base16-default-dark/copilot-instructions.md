## obsidian-base16-default-dark

> Brutalist Obsidian theme built on the Base16 Default Dark color scheme. Single file: `theme.css`.

# Agent Instructions

Brutalist Obsidian theme built on the Base16 Default Dark color scheme. Single file: `theme.css`.

## Releasing a new version

1. Commit `theme.css` changes first (scripts don't touch it):
   ```bash
   git add theme.css && git commit -m "..."
   ```

2. Run the appropriate release script — it bumps `package.json`, `manifest.json`, `versions.json`, commits, tags, and pushes everything:
   ```bash
   npm run release:patch   # 3.1.0 → 3.1.1 (bug fixes, style tweaks)
   npm run release:minor   # 3.1.0 → 3.2.0 (new features)
   npm run release:major   # 3.1.0 → 4.0.0 (breaking changes)
   ```

---
> Source: [flowing-abyss/obsidian-base16-default-dark](https://github.com/flowing-abyss/obsidian-base16-default-dark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
