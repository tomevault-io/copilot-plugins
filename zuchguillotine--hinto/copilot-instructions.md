## hinto

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HINTO is a social relationship voting platform with an AI relationship coach, targeting women 16–35. Users manage ranked lists of romantic options, friends vote and comment, and an AI coach gives personalized guidance. Platforms: native iOS and web.

**This repo is in an active restart phase.** The legacy Expo/AWS Amplify architecture is still present but slated for retirement. New work targets Supabase + TypeScript HTTP API + native SwiftUI iOS + web. Read `docs/Restart_Unification_Plan.md` before making architecture-level decisions.

## Development Commands

### Legacy Expo App (apps/hnnt-app)
- `npm start` — Expo dev server
- `npm run ios` / `npm run android` / `npm run web` — Platform runners
- `npm run build` — Export web build

### New Backend API (services/api)
- `npm run api:build` — Compile TypeScript (`tsc -p services/api/tsconfig.json`)
- `npm run api:start` — Build and run (`node services/api/dist/server.js`)
- `npm run api:watch` — Watch mode for API TypeScript

### Code Quality
- `npm run lint` / `npm run lint:fix` — ESLint
- `npm run format` — Prettier
- Pre-commit hooks via Husky + lint-staged (auto-runs eslint --fix + prettier on staged files)

### Testing
- `npx jest` — Run tests (Jest is installed but no `npm test` script is wired up)

## Repository Structure

```
/apps
  /hnnt-app/         Legacy Expo/React Native app (still running, retiring)
  /ios/              New native SwiftUI app (in progress)
    HINTO/Sources/   App, Navigation, Design, Models, Services, Views

/services
  /api/              New TypeScript HTTP API (first vertical slice done)
    src/             server.ts, routes/, middleware/, config, logger, supabase client

/packages
  /contracts/        Vendor-neutral OpenAPI/JSON schema definitions
  /domain/           Business rules and validation (no vendor dependencies)

/supabase
  migrations/        PostgreSQL migration files

/amplify/            Legacy AWS backend — do NOT expand
/lambda/             Legacy reference only (Snapchat OAuth)
/scripts/            Legacy operational scripts (Cognito helpers)
/docs/               Restart architecture documentation (source of truth)
```

## Architecture: New (Target)

Defined in `docs/Canonical_Architecture.md`. First vertical slice (profile + situationships) is implemented.

- **Backend**: Node.js HTTP API under `/services/api/` with versioned routes (`/v1`), structured JSON error envelopes, request-context propagation
- **Database**: PostgreSQL via Supabase with RLS policies
- **Auth**: Supabase Auth + provider linkage tables (Apple, Meta/Facebook required for MVP; Snapchat, TikTok deferred). See `docs/Auth_Model.md`
- **iOS**: SwiftUI app under `/apps/ios/` with design system, models, API client, auth manager
- **Web**: Planned under `/apps/web/` (not yet scaffolded)
- **API Style**: REST/JSON — no GraphQL, no tRPC
- **Contracts**: OpenAPI definitions in `/packages/contracts/` consumed by Swift and web clients
- **Domain Logic**: Vendor-neutral rules in `/packages/domain/` (profile privacy, situationship ordering, viewer/audience modeling)

### Implemented API Routes (services/api)
- `GET /health`, `GET /v1/health`, `GET /v1` (discovery)
- `GET /v1/me`, `PATCH /v1/me` (profile)
- `GET|POST /v1/me/situationships`, `PATCH|DELETE /v1/me/situationships/:id`
- `PUT /v1/me/situationships/order` (reorder)

### Environment Variables (services/api)
`SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `OPENAI_API_KEY`

## Architecture: Legacy (Still Running)

- **Frontend**: Expo 53, React Native 0.79, React 19, React Navigation v7
- **Backend**: AWS Amplify — AppSync GraphQL, Cognito, DynamoDB, S3
- **Auth**: Cognito with federated OAuth (Google, Snapchat, Instagram)
- **State**: React Context providers (`SituationshipsProvider`, `UserProfileProvider`) + custom hooks
- **Entry**: `App.tsx` → hardcoded Amplify config → `index.ts` → Expo registration

### Legacy Coupling Hotspots (see `docs/Legacy_AWS_Audit.md`)
- `apps/hnnt-app/src/hooks/useAuth.tsx` — hard-coupled to Cognito
- `apps/hnnt-app/src/context/useSituationships.tsx` — uses AppSync GraphQL
- `apps/hnnt-app/src/context/useUserProfile.tsx` — uses AppSync GraphQL
- `App.tsx` — inline Amplify configuration

### GraphQL Schema
`amplify/backend/api/hinto/schema.graphql` — Models: User, Situationship, Vote, Report, InviteToken

## Core Domain Model

Defined in `docs/Canonical_Domain_Model.md`:
- **Profile**: User identity with privacy settings and social links
- **AuthIdentity**: Provider linkage per user (Apple, Meta, Snapchat, TikTok)
- **Situationship**: Ranked romantic options with sharing/access control
- **VotingSession / Vote**: Friend voting system (best/worst rankings)
- **AI Coach**: Personalized guidance (routes not yet implemented)

## Key Constraints

- **Do not widen the AWS Amplify/Cognito/AppSync footprint** unless explicitly asked
- Legacy `amplify/` directory: changes should be rare and intentional
- Prefer isolating and documenting legacy code over deepening coupling
- New structure follows: `/apps/ios`, `/apps/web`, `/services/api`, `/packages/domain`, `/packages/contracts`
- Keep changes scoped — do not mix restart rewrites with incidental refactors
- When touching legacy frontend code, follow existing patterns in `apps/hnnt-app/src/`

## Known Issues

- **Reanimated disabled**: Babel plugin permanently removed due to incompatibility with optional chaining and modern JS syntax. Standard FlatList used instead of DraggableFlatList.
- **No wired test script**: Jest is a dependency but `npm test` is not defined in package.json. Use `npx jest` directly.
- **Missing amplifyconfiguration.json import**: Legacy app startup risk documented in AWS audit.

## Key Documentation

| Document | Purpose |
|---|---|
| `docs/Restart_Unification_Plan.md` | Master restart plan — read first |
| `docs/Canonical_Architecture.md` | Target architecture decisions |
| `docs/Canonical_Domain_Model.md` | Core entities and business rules |
| `docs/Auth_Model.md` | Auth strategy (Supabase Auth + provider linking) |
| `docs/Execution_Backlog.md` | Current task queue for restart work |
| `docs/Legacy_AWS_Audit.md` | Coupling analysis of Amplify touchpoints |
| `docs/Schema_Entity_Mapping.md` | Legacy-to-new schema mapping |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZuchGuillotine)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ZuchGuillotine)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
