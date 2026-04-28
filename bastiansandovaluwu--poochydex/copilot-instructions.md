## poochydex

> English in code; UI strings via language (es/en)


# Code language and UI (i18n)

## Source code in English

- **Identifiers:** English (`userName`, `loadPokemon`, not `nombreUsuario`).
- **Comments and JSDoc:** English.
- **String literals in code** that are **not** shown to the user: English if needed (logs, internal errors for developers).

## User-facing UI: Spanish and English

The app supports **Spanish** and **English**. User-visible text must **not** be fixed to a single language in templates without using the active locale.

- Use a **`language` (or `currentLanguage`) value** that is either **`'es'`** or **`'en'`**, aligned with **`LanguageService`** (`currentLanguage`, `currentLanguage$`, `setCurrentLanguage`).
- In components, subscribe to **`LanguageService.currentLanguage$`** (or read **`currentLanguage`**) and branch labels/messages with **`language === 'es'`** / **`'en'`**, or use existing **translate pipes** / helpers from `shared` where the project already does i18n.
- When adding new UI copy, provide **both** Spanish and English variants tied to that language variable (or extend the same i18n mechanism the feature already uses).

## Do not

- Hardcode only Spanish **or** only English for new user-visible strings.
- Add large duplicate blocks without a clear `language`-based split or shared translation layer consistent with the module.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BastianSandovalUwU) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
