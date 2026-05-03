## mapyourhealth

> Instructions and patterns for AI agents working on this codebase.

# AGENTS.md - MapYourHealth

Instructions and patterns for AI agents working on this codebase.

## Project Overview

MapYourHealth is a mobile app that shows safety statistics (water quality, air quality, health hazards, disaster risks) for zip codes. Users can subscribe to zip codes and receive push notifications when safety updates are published.

## Repository Structure

```
MapYourHealth/
├── apps/
│   ├── mobile/                    # React Native (Expo) mobile app - Ignite template
│   │   └── app/
│   │       ├── components/        # Reusable UI components
│   │       ├── screens/           # Screen components
│   │       ├── navigators/        # Navigation configuration
│   │       ├── data/              # Data layer (NEW)
│   │       │   ├── types/         # TypeScript type definitions
│   │       │   ├── mock/          # Mock data for development
│   │       │   └── helpers.ts     # Data utility functions
│   │       └── services/          # External service integrations
│   │           └── amplify/       # Amplify service functions
│   └── admin/                     # Admin web portal (future - Phase 9)
├── packages/
│   └── backend/                   # AWS Amplify Gen2 backend
│       └── amplify/
│           ├── data/              # Data models (resource.ts)
│           ├── auth/              # Auth config (resource.ts)
│           ├── functions/         # Lambda functions
│           │   └── send-notifications/
│           └── backend.ts         # Main backend definition
├── scripts/
│   └── ralph/                     # Ralph autonomous agent loop
└── package.json                   # Monorepo root (yarn workspaces)
```

## Tech Stack

- **Mobile**: React Native with Expo, Ignite template patterns
- **Backend**: AWS Amplify Gen2 (TypeScript-first)
- **State Management**: Zustand (preferred) or MobX-State-Tree (Ignite default)
- **Navigation**: React Navigation
- **Styling**: Ignite's styling patterns with `$presets`

## Key Commands

```bash
# Install dependencies
yarn

# Mobile app
yarn mobile:start          # Start Expo dev server
yarn mobile:ios           # Run on iOS simulator
yarn mobile:android       # Run on Android emulator
yarn mobile:test          # Run tests

# Backend
yarn backend:sandbox      # Deploy Amplify sandbox
yarn backend:generate     # Generate TypeScript types

# Quality checks
yarn type-check           # TypeScript checking
yarn lint                 # ESLint
yarn test                 # All tests
```

## Coding Patterns

### Mobile App (apps/mobile/)

1. **Components**: Place in `app/components/`. Use Ignite's component patterns.
2. **Screens**: Place in `app/screens/`. Follow `*Screen.tsx` naming.
3. **Navigation**: Defined in `app/navigators/`.
4. **Styling**: Use Ignite's `$viewPreset` and `$textPreset` patterns.
5. **Types**: Place safety-related types in `app/data/types/safety.ts`.
6. **Mock Data**: Place in `app/data/mock/`. Each file exports typed mock data.
7. **Data Helpers**: Place utility functions in `app/data/helpers.ts`.
8. **Amplify Services**: Place in `app/services/amplify/`. Functions for backend interaction.

### File Path Convention

**IMPORTANT**: All mobile app source files use the `apps/mobile/app/` prefix:
- Components: `apps/mobile/app/components/ComponentName.tsx`
- Screens: `apps/mobile/app/screens/NameScreen.tsx`
- Data types: `apps/mobile/app/data/types/safety.ts`
- Mock data: `apps/mobile/app/data/mock/*.ts`
- Services: `apps/mobile/app/services/amplify/*.ts`

Do NOT use `src/` paths - this project uses the Ignite template structure.

### Backend (packages/backend/)

1. **Data Models**: Define in `amplify/data/resource.ts`
2. **Auth**: Configure in `amplify/auth/resource.ts`
3. **Functions**: Lambda functions in `amplify/functions/`

## Ralph Integration

This project uses the Ralph autonomous agent loop for AI-assisted development.

### Running Ralph

```bash
cd scripts/ralph
./ralph.sh [max_iterations]  # Default: 10 iterations
```

### Ralph Files

- `scripts/ralph/prd.json` - User stories with completion status
- `scripts/ralph/progress.txt` - Append-only log of progress and learnings
- `scripts/ralph/prompt.md` - Instructions for each Claude iteration

### Story Sizing

Stories must be small enough to complete in ONE iteration. If a story is too big:
- Split into smaller stories
- Update prd.json with new story IDs
- Maintain dependency order (priority numbers)

## GitHub Integration

Issues are tracked at: https://github.com/epiphanyapps/mapyourhealth

### Issue Structure

- **Epics** (Issues #1-11): High-level features marked with `[Epic]` prefix and `epic` label
- **Stories** (Issues #12-45): Small, implementable tasks linked to epics
- Each user story in prd.json has a `githubIssue` field linking to the corresponding GitHub issue

### Phase Labels

Stories are organized by phase:
- `phase-1-foundation` - Types and mock data (Issues #12-16)
- `phase-2-dashboard` - Dashboard UI components (Issues #17-23)
- `phase-3-details` - Detail view screens (Issues #24-28)
- `phase-4-report` - Hazard report feature (Issues #29-31)
- `phase-5-backend` - Backend infrastructure (Issues #32-36)
- `phase-6-auth` - Authentication UI (Issues #37-41)
- `phase-7-subscriptions` - Zip code subscriptions (Issues #42-45)

**Deferred Phases** (not in current prd.json):
- Phase 8: Push Notifications (Issue #10 epic)
- Phase 9: Admin Portal (Issue #11 epic)

## Testing

- Unit tests: Jest with React Native Testing Library
- Run before committing: `yarn test`
- Type check before committing: `yarn type-check`

## Common Gotchas

1. **Amplify outputs**: After `yarn backend:sandbox`, run `yarn sync:amplify` to copy outputs to mobile app
2. **iOS builds**: Run `npx pod-install` after adding native dependencies
3. **Type errors**: Run `yarn type-check` frequently, especially after changing types

## When to Update This File

Add patterns here when you discover:
- Conventions that aren't obvious from the code
- Gotchas that cost you time
- Dependencies between files/modules
- Testing approaches for specific areas

---
> Source: [epiphanyapps/mapyourhealth](https://github.com/epiphanyapps/mapyourhealth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
