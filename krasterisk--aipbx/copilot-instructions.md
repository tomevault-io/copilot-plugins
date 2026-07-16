## frontend-fsd

> Frontend FSD and UI conventions


# Frontend FSD Rules

Architecture: Feature-Sliced Design (`app → pages → widgets → features → entities → shared`).

## UI layer

- **New UI only in** `src/shared/ui/redesign/`
- Do not add code to `shared/ui/deprecated/`
- Prefer `redesign-v3` components over MUI where equivalents exist

## API layer

- RTK Query endpoints in `entities/*/api/*Api.ts`, injected via `rtkApi`
- Types in `entities/*/model/types/`
- After backend DTO change, update matching entity types (see `intel/API-MAP.md`)

## i18n

- Keys in `public/locales/{en,ru}/`
- No hardcoded user-visible strings in components

## Build

- Production builds use Webpack (`npm run build:prod`), not Vite
- Env vars via `.env` / `.env.local` (see `.env.example`)

---
> Source: [krasterisk/aiPBX](https://github.com/krasterisk/aiPBX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
