## previews-centralized

> Enforce centralized preview support and colocated preview blocks.


# Centralized Previews

- Use `Brew/PreviewSupport/AppPreviewSupport.swift` for shared preview sample data and lightweight preview fakes/mocks.
- Do not create one-off inline preview mock repositories/services in individual view files.
- When preview data/mocks need updates, add/update them in `AppPreviewSupport.swift` so previews share one source of truth.
- Keep `#Preview` blocks at the bottom of the same file as their view type; do not use standalone `+Previews.swift` files.

---
> Source: [Homebrew/BrewUI](https://github.com/Homebrew/BrewUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
