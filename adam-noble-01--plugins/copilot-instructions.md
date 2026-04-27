## plugins

> Standardize shared asset usage via Na__Common__PluginDependencies

# Common Assets Integration
- For shared icons/graphics/templates, load from sibling `Na__Common__PluginDependencies`.
- Use explicit `require_relative` or absolute path construction from plugin root.
- Keep asset source centralized; avoid copying shared assets into feature plugins.
- If a plugin has unique-only assets, keep them local and clearly separated from common assets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Adam-Noble-01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
