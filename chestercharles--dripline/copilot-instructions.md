## dripline

> Plant tracking app for a homeowner in Gilbert, AZ (USDA Zone 9b, Sonoran Desert). Catalogs plants by location, tracks health over time with photos, monitors irrigation, and provides region-aware care guidance.

# Dripline

Plant tracking app for a homeowner in Gilbert, AZ (USDA Zone 9b, Sonoran Desert). Catalogs plants by location, tracks health over time with photos, monitors irrigation, and provides region-aware care guidance.

## Tech Stack

- **Platform**: iOS, Expo SDK ~54, Expo Router v6
- **Database**: expo-sqlite + Drizzle ORM (local-first, no backend)
- **Photos**: expo-file-system (document directory, auto iCloud backup)
- **Secrets**: expo-secure-store (API keys)
- **State**: React Query v5
- **Weather**: Open-Meteo API (free, no key)
- **AI Vision**: OpenAI or Anthropic (user-provided key)
- **UI**: Reanimated, gesture-handler, @gorhom/bottom-sheet

## Project Structure

- `app/` — Expo Router routes (thin wrappers)
- `features/` — Feature modules (screens, hooks, components)
- `components/ui/` — Shared design system components
- `lib/db/` — Drizzle schema, migrations, provider
- `lib/theme/` — Colors, spacing, typography, shape, context
- `lib/query/` — React Query client and key factories
- `lib/constants/` — Block types, drip statuses, seasonal tips, etc.
- `lib/weather/` — Open-Meteo API integration
- `lib/ai/` — OpenAI + Anthropic vision clients
- `lib/photos/` — Photo storage, resize, capture utilities

## Conventions

- TypeScript strict mode with `@/*` path aliases
- Prettier: tabs, no semicolons, single quotes, trailing commas es5
- No comments unless truly necessary
- System fonts only, semantic color tokens, 4px spacing scale
- 44pt minimum touch targets, WCAG AA contrast
- Cards with elevated surface + lg radius
- Light/medium haptics for meaningful actions
- Spring physics animations via Reanimated

## Commands

- `npx expo start` — Start dev server
- `npx expo start --ios` — Start on iOS simulator

## Database

SQLite with Drizzle ORM. Schema in `lib/db/schema.ts`, migrations in `lib/db/migrations.ts`. Seeded with "Front Yard" and "Back Yard" on first launch.

## Key Patterns

- Features export screens from `features/*/` and are imported by `app/` routes
- All data access through React Query hooks in `features/*/hooks/`
- Database access via `useDrizzle()` hook from DatabaseProvider
- Theme access via `useTheme()` hook
- Photos stored at `{documentDirectory}/photos/{plantId}/{timestamp}.jpg`
- API keys stored in expo-secure-store, never in SQLite

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chestercharles)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chestercharles)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
