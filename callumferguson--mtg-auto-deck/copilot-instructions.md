## mtg-auto-deck

> - This app is dark mode only.

# Repository Guidelines

## UI Direction

- This app is dark mode only.
- Do not add light theme variants, light-mode toggles, or light surfaces that conflict with the existing dark presentation.
- New UI should use dark backgrounds, readable high-contrast text, and accent colors that fit the current visual style.
- When updating existing screens, preserve and extend the dark-mode look instead of introducing mixed light/dark sections.

## Product Context

- This project is an MTG deck goldfishing tool where an AI will automatically play your deck.


## Security

- Never read or change a .env file
- If there is a change you would like to make to it, notify the user and tell them what to change
- Treat `.env` as strictly hands-off, even when the user is asking about env vars, setup, API keys, or requests a full env template.
- Do not open `.env`, do not patch `.env`, do not rewrite `.env`, and do not include `.env` in scripted file edits.
- Instead, tell the user the exact lines to add, remove, or change, and keep the actual implementation work in code or docs outside `.env`.
- Before any file edit, sanity-check that `.env` is not one of the targets.
- Any time the app is updated to use a new env variable or is updated to no longer use an env variable, update the example env file .env.example

---
> Source: [CallumFerguson/mtg-auto-deck](https://github.com/CallumFerguson/mtg-auto-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
