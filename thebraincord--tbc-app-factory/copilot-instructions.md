## tbc-app-factory

> High-velocity Expo monorepo for building premium Android-only apps under the TheBrainCord brand. Each app is a focused "simple-problem-solver". All apps share a single UI design system and common packages so app-level code is pure logic/functionality.

# TBC-APP-Factory — Claude Code Guide

## Project Overview
High-velocity Expo monorepo for building premium Android-only apps under the TheBrainCord brand. Each app is a focused "simple-problem-solver". All apps share a single UI design system and common packages so app-level code is pure logic/functionality.

## Repository Structure
```
TBC-APP-Factory/
├── apps/
│   ├── TBC-KDelivery/       # Delivery app
│   ├── TBC-Explorer/        # Discovery/browse app
│   └── TBC-*/               # Future apps (TBC-Calculator, TBC-Fitness, etc.)
├── packages/
│   ├── ui/                  # "Shining UI" — all visual components + theme tokens
│   ├── utils/               # Pure utility functions (format, validation)
│   ├── hooks/               # Shared React/RN hooks (useAsync, useDebounce, etc.)
│   ├── services/            # Supabase client, secure storage
│   └── config/              # Shared Babel, ESLint, tsconfig.base
├── templates/
│   └── TBC-AppTemplate/     # Copy this to create a new TBC app
├── .github/
│   └── workflows/
│       ├── build-apk.yml    # EAS build (manual + push to main)
│       └── pr-checks.yml    # TypeScript + ESLint on PRs
├── docs/
│   └── SETUP.md             # Token setup, EAS init, adding new apps
├── .env.example             # Environment variable template
├── eas.json                 # Build profiles (development / preview / production)
└── package.json             # Workspace root
```

## Stack
- **Runtime:** Expo SDK 52+ with Expo Router (file-based routing)
- **Platform:** Android only
- **Monorepo:** Yarn Workspaces
- **Build:** EAS Build — APK (preview) / AAB (production)
- **Database/Auth:** Supabase (configured in `packages/services`)
- **Language:** TypeScript strict mode

## Key Conventions
- App names use `TBC-` prefix: `TBC-Calculator`, `TBC-Fitness`, etc.
- App workspace package names: `@tbc/TBC-AppName`
- **Never duplicate** UI components or theme values across apps — everything lives in `packages/ui`
- App-level code should only contain screens, navigation, and business logic
- Env vars use `EXPO_PUBLIC_` prefix so they are accessible in Expo client code
- Each app has its own `.env` (never committed) — see `.env.example`

## Adding a New App
```bash
cp -r templates/TBC-AppTemplate apps/TBC-NewApp
# Then update: package.json (name), app.json (name/slug/scheme/package)
# Then: cd apps/TBC-NewApp && eas init
```

## Required GitHub Secrets
| Secret | Purpose |
|--------|---------|
| `EXPO_TOKEN` | EAS CLI auth in GitHub Actions |
| `SUPABASE_URL` | Passed as env var at build time |
| `SUPABASE_ANON_KEY` | Passed as env var at build time |

## Common Commands
```bash
yarn install                   # Install all workspace deps
yarn start:kdelivery           # Start TBC-KDelivery dev server
yarn typecheck                 # TypeScript check across all packages
yarn lint                      # ESLint across apps + packages
yarn build:kdelivery           # Trigger EAS preview build for KDelivery
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheBrainCord)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TheBrainCord)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
