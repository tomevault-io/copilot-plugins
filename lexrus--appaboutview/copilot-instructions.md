## localization

> - Simplified Chinese (zh-Hans)


# Localization Rules

## Supported Languages
- English (base)
- German (de)
- Spanish (es)
- French (fr)
- Italian (it)
- Japanese (ja)
- Korean (ko)
- Russian (ru)
- Simplified Chinese (zh-Hans)
- Traditional Chinese (zh-Hant)
- Finnish (fi)
- Hindi (hi)
- Indonesian (id)
- Dutch (nl)
- Norwegian (no)
- Polish (pl)
- Portuguese (pt)
- Swedish (sv)
- Thai (th)
- Turkish (tr)
- Ukrainian (uk)
- Vietnamese (vi)

## Localization Structure
- Language-specific `.lproj/` folders in `Sources/Resources/`
- Each folder contains `Localizable.strings` file
- Localized strings use bundle-specific lookup: `String(localized: "key", bundle: .module)`
- App data supports runtime locale-based string selection

## Implementation Guidelines
- Always use `String(localized: "key", bundle: .module)` for localized strings
- Maintain consistency across all language files
- Test localization with different system languages
- Ensure fallback logic works properly for missing translations

---
> Source: [lexrus/AppAboutView](https://github.com/lexrus/AppAboutView) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
