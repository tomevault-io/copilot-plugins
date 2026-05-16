## capibarismo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Capibarismo** is a civic engagement platform for Peru's presidential elections that gamifies candidate comparison using a 90s fighting game aesthetic. The core experience is a **single-elimination tournament** where 36 candidates are narrowed down to a personal top 3 through pick-one-from-three and 1v1 matchups. Users also explore detailed candidate information, compare candidates side-by-side, and view a political compass.

**Core Philosophy: "Game Feel"**
- **The Punch** (<100ms): Voting must feel instant and visceral (requires optimistic UI updates)
- **The Flow** (<1s): Transitions must maintain user engagement without interruption
- **The Reach**: Optimized for 3G networks in rural Peru (high latency tolerance)

## Common Commands

### Development
```bash
npm install              # Install dependencies
npm run dev              # Start dev server at http://localhost:8080
npm run build            # Production build (validates data, generates sitemap)
npm run build:dev        # Development build (for testing)
npm run preview          # Preview production build locally
```

### Code Quality
```bash
npm run lint             # Run ESLint
npm run typecheck        # TypeScript type checking (no emit)
npm test                 # Run Vitest tests
npm test:watch           # Watch mode for tests
npm test:coverage        # Generate coverage report
npm test:ui              # Visual test UI
```

### Load Testing
```bash
npm run loadtest:smoke           # Quick test (5 users, 1 min)
npm run loadtest:baseline        # Normal load (10-50 users, 5 min)
npm run loadtest:peak            # Peak traffic simulation (1k users, 15 min)
npm run loadtest:peru            # Peru-specific network profiles

# Test specific network conditions
NETWORK_PROFILE=rural npm run loadtest:peru      # 3G rural
NETWORK_PROFILE=urban npm run loadtest:peru      # 4G urban
NETWORK_PROFILE=congested npm run loadtest:peru  # Peak hours
```

### Data & Validation
```bash
npm run generate:sitemap     # Generate sitemap.xml
tsx scripts/validate-data.ts # Validate candidate data structure
```

## Architecture Overview

### Frontend Stack
- **React 18** + **TypeScript** + **Vite** (with SWC for fast refresh)
- **Routing**: React Router with lazy-loaded pages
- **Animations**: Framer Motion for overlays, transitions, and bracket zoom
- **State Management**:
  - Zustand for tournament state (`useTournamentStore`) and UI state (`useGameUIStore`, `useCompareStore`)
  - TanStack Query for API calls (ranking page, with aggressive caching)
  - localStorage persistence via Zustand `persist` middleware (tournament state)
- **Styling**: Tailwind CSS + shadcn/ui components
- **Analytics**: PostHog (optional, graceful degradation if not configured)

### Backend (Vercel Serverless)
Located in `/api/` directory:
- **POST /api/game/vote** - Records user votes (fire-and-forget, optimized for write throughput)
- **GET /api/ranking/personal** - Calculates personalized rankings from vote history
- **Storage**: Dual adapter pattern (in-memory for dev, Vercel Blob for production)

### Key Directories
```
/api/                  # Vercel serverless functions
/src/
  /pages/              # Route-level components (lazy-loaded)
  /components/         # Feature-organized UI components
    /game/             # Core game UI (VSScreen, GameHUD, OnboardingModal, CandidateInfoOverlay)
    /tournament/       # Tournament bracket system (BracketTreePage, BracketTree, PickFromThree, PodiumScreen)
    /compare/          # Side-by-side comparison tool
    /political-compass/ # 2D political visualization
    /ui/               # shadcn/ui design system primitives
  /hooks/              # Custom React hooks (game API, legacy Elo hooks)
  /store/              # Zustand state stores (tournament, game UI, compare)
  /services/           # Business logic (tournament logic, session management)
  /data/               # Candidate data and type definitions
    /domains/          # Structured candidate information (education, income, etc.)
  /lib/                # Utilities, constants, types (tournamentTypes, tournamentConstants)
/scripts/              # Build and validation scripts
/load-tests/           # k6 performance test scenarios
```

## Critical Architectural Patterns

### 1. Tournament State Machine (Primary Game Flow)
**Location**: `src/services/tournamentService.ts`, `src/store/useTournamentStore.ts`, `src/lib/tournamentTypes.ts`

The core game is a single-elimination tournament with 36 candidates and 19 total decisions:

**Tournament format:**
| Round | Type | Matches | Description |
|-------|------|---------|-------------|
| R0 | pick-one-from-three | 12 | 36 → 12 candidates |
| R1 | pick-one-from-three | 4 | 12 → 4 candidates |
| R2 | 1v1 semifinal | 2 | 4 → 2 candidates |
| R3 | 1v1 final | 1 | 2 → 1 winner |

**Phase state machine:**
```
null → onboarding → bracket-preview → playing-pick-three/playing-1v1
  → round-transition → (next playing phase) → ... → podium
```

**Key design decisions:**
- `tournamentService.ts` contains **pure functions only** (no side effects). State flows in, new state flows out.
- `useTournamentStore` wraps the service with Zustand `persist` middleware (localStorage key: `tournament-state-v2`)
- Winner propagation: R0→R1 uses groups of 3, R1→R2 uses groups of 2, R2→R3 takes both winners
- Podium 3rd place: first semifinal loser

When modifying tournament logic, keep functions pure and test with plain state objects.

### 2. Bracket + Overlay UI Pattern
**Location**: `src/pages/JugarPage.tsx`, `src/components/tournament/`

During playing phases, the **bracket tree is always the base layer** and the match UI (PickFromThree or VSScreen) appears as a **full-screen animated overlay** (z-40):

1. Phase changes → bracket is visible immediately
2. After `AUTO_SHOW_DELAY` (1s), match overlay slides in with spring animation
3. User votes → overlay slides out, bracket briefly visible with updated state
4. "Ver Bracket" button hides overlay so user can inspect the bracket manually

This pattern keeps the bracket as persistent context while the match UI handles interaction.

### 3. Component Composition
Components follow shadcn/ui patterns:
```tsx
<Card>
  <CardHeader>
    <CardTitle>...</CardTitle>
  </CardHeader>
  <CardContent>...</CardContent>
</Card>
```

Prefer composition over prop drilling. Use Zustand stores for cross-component state rather than passing callbacks through multiple layers.

### 4. Dual Storage Adapters (API/Ranking)
**Location**: `api/storage.ts`

- **Development**: In-memory Map (fast, zero config, resets on restart)
- **Production**: Vercel Blob Storage (durable, one JSON file per vote)

Environment variables:
- `BLOB_READ_WRITE_TOKEN` (required for production)
- `VITE_USE_API` (toggle mock data in frontend)

### 5. Legacy Systems (Still in Codebase)

These files still exist and are used by `RankingPage` (`/ranking`) but are **not used by the tournament flow** in JugarPage:

| File | Purpose | Used by |
|------|---------|---------|
| `src/hooks/useOptimisticVote.ts` | Optimistic vote submission with rollback | Legacy (tests only) |
| `src/hooks/useGameAPI.ts` | Pair fetching, ranking queries | RankingPage |
| `src/services/pairGenerationService.ts` | Elo-based smart pairing | useGameAPI |
| `src/services/sessionService.ts` | Session/vote/Elo persistence | RankingPage, useGameAPI |
| `src/lib/gameConstants.ts` | ELO_K, INITIAL_ELO, vote goals | pairGenerationService, sessionService |
| `src/components/game/CompletionModal.tsx` | Vote milestone modal | Not rendered in tournament |

Do not remove these files — `RankingPage` depends on the session/API infrastructure.

## Development Workflows

### Adding New Candidates

1. Add candidate data to `/src/data/domains/base.ts`
2. Add supplementary data to domain files (education, income, etc.)
3. Add profile images to `/public/candidates/`
   - `{id}-headshot.webp` (profile photo)
   - `{id}-fullbody.webp` (optional, for fighting game view)
4. Run `tsx scripts/validate-data.ts` to ensure data integrity
5. Political compass positions in `/src/data/domains/compass.ts`

### Modifying Tournament Logic

**Critical files (tournament)**:
- `src/services/tournamentService.ts` - Pure tournament logic (create, advance, progress)
- `src/store/useTournamentStore.ts` - Zustand store with persist middleware
- `src/lib/tournamentTypes.ts` - TypeScript interfaces (TournamentState, TournamentPhase, etc.)
- `src/lib/tournamentConstants.ts` - Round configuration and constants
- `src/pages/JugarPage.tsx` - Phase-based rendering orchestrator

**Important constants** (in `tournamentConstants.ts`):
```typescript
TOTAL_CANDIDATES = 36       // Required candidate count
TOTAL_DECISIONS = 19        // Total user votes per tournament
TOTAL_MATCHES = 19          // Total matches across all rounds
TOURNAMENT_STORAGE_KEY = 'tournament-state-v2'  // localStorage key
ROUND_CONFIG = [...]        // 4-element array defining each round
```

**Critical files (legacy, used by RankingPage)**:
- `src/hooks/useGameAPI.ts` - Pair fetching, ranking queries
- `src/services/sessionService.ts` - Session/Elo persistence
- `src/lib/gameConstants.ts` - Elo and goal constants

### Working with State

**Tournament State** (Zustand + persist):
- `useTournamentStore` - Full tournament state machine (bracket, phase, matches, podium)
- Persisted to localStorage under key `tournament-state-v2`
- Actions: `startNewTournament`, `submitVote`, `advanceFromRoundTransition`, `goToBracketPreview`, `startPlaying`, `resetTournament`

**UI State** (Zustand, not persisted):
- `useGameUIStore` - CandidateInfoOverlay visibility, selected candidate, `reducedMotion` preference
- `useCompareStore` - Comparison tool candidate selection (left/right slots)

**Legacy Persisted State** (via sessionService, used by RankingPage):
- Vote count, seen pairs, local Elo ratings (localStorage)
- Session ID (nanoid-based)

**Server State** (TanStack Query):
- Ranking data (used by RankingPage)
- Aggressive caching (5min stale time)

### Testing Strategy

**Unit Tests**: Hooks, services, utilities
**Component Tests**: React Testing Library
**Load Tests**: k6 scripts in `/load-tests/`

Mock setup in `src/test/setup.ts` handles:
- `window.matchMedia()` for responsive behavior
- `IntersectionObserver` for lazy loading

When writing tests for session logic, use the factory pattern:
```typescript
const mockStorage = createMockStorage();
const service = createSessionService(mockStorage, mockStorage);
```

### Tournament Components Reference

| Component | Location | Purpose |
|-----------|----------|---------|
| `BracketTreePage` | `src/components/tournament/` | Full bracket view with mode-based rendering (preview/viewing/transition) |
| `BracketTree` | `src/components/tournament/` | SVG-based bracket visualization with mobile zoom animation |
| `PickFromThree` | `src/components/tournament/` | Pick-one-from-three UI for R0 and R1 |
| `PodiumScreen` | `src/components/tournament/` | Final results with confetti, share/screenshot functionality |
| `RoundTransition` | `src/components/tournament/` | Standalone round transition overlay (available but BracketTreePage handles this) |
| `BracketMatchBox` | `src/components/tournament/` | Compact match card for list-style bracket view |

| Component | Location | Purpose |
|-----------|----------|---------|
| `VSScreen` | `src/components/game/` | 1v1 fighting game screen (used in R2/R3), accepts `roundLabel` prop |
| `GameHUD` | `src/components/game/` | Top bar with round label, match progress, bracket/new-game buttons |
| `OnboardingModal` | `src/components/game/` | Tournament intro modal (describes the 4-round format) |
| `CandidateInfoOverlay` | `src/components/game/` | Slide-in candidate detail panel (available in all phases) |

### Performance Considerations

**Code Splitting**:
- Pages are lazy-loaded via React.lazy()
- Manual chunks configured in `vite.config.ts` for vendor libraries
- Political compass is split into separate chunk (large SVG library)

**Image Optimization**:
- Use WebP format for all candidate images
- Lazy load images (built-in browser loading="lazy")

**Bundle Analysis**:
```bash
npm run build
npx vite-bundle-analyzer dist/
```

## Environment Variables

See `ENVIRONMENT.md` for detailed configuration guide.

**Development** (`.env.local`):
```bash
VITE_USE_API=false           # Use mock data (no API calls)
VITE_POSTHOG_KEY=            # Optional: analytics
USE_PROD_API=true            # Optional: proxy to production API
```

**Production** (Vercel Dashboard):
```bash
VITE_POSTHOG_KEY=phc_xxx     # PostHog analytics key
BLOB_READ_WRITE_TOKEN=xxx    # Required: Vercel Blob storage
```

**Important**: `VITE_*` variables are embedded in client bundle (public). Never put secrets in them.

## Facts Protocol (Data Integrity)

This project follows strict fact verification standards:

1. **All candidate data must be traceable to primary sources**
2. **Sources documented in** `/src/data/candidateSources.ts`
3. **Official JNE data preferred** (Jurado Nacional de Elecciones)
4. When adding/modifying candidate information:
   - Include source URLs
   - Add verification timestamps
   - Cite official documents when available
   - Flag unverified claims clearly

## Code Style Conventions

### File Naming
- Components: `PascalCase.tsx`
- Hooks: `camelCase.ts` with `use` prefix
- Services: `camelCase.ts`
- Types: `types.ts` or `camelCase.ts`

### Import Order
1. External libraries (React, etc.)
2. Internal utilities (`@/lib/...`)
3. Components (`@/components/...`)
4. Types and data (`@/data/...`)

### TypeScript
- Use `@/` alias for absolute imports (configured in tsconfig)
- Strict mode disabled (`strictNullChecks: false`)
- Prefer interfaces for public APIs, types for unions/intersections

### Component Patterns
- Use `cn()` utility for conditional Tailwind classes
- Destructure props with defaults
- Export named components (not default exports for components)
- Colocate component-specific utilities in same file

## Git Workflow

**Branch naming**:
- `feature/description` for new features
- `fix/description` for bug fixes
- `refactor/description` for code improvements

**Commit messages**: Use conventional format
- `feat:` - New feature
- `fix:` - Bug fix
- `refactor:` - Code restructuring
- `docs:` - Documentation
- `test:` - Testing
- `perf:` - Performance improvements

**Pre-commit hooks** (Husky):
- Data validation runs on pre-commit
- Build must succeed before push

## Deployment

**Platform**: Vercel

**Build configuration** (`vercel.json`):
- Rewrites `/api/*` to serverless functions
- Single-page app fallback to `index.html`

**Build process**:
1. Data validation (`scripts/validate-data.ts`)
2. Sitemap generation (`scripts/generate-sitemap.js`)
3. Vite production build
4. Deploy to Vercel

**Environment setup**:
- Set `BLOB_READ_WRITE_TOKEN` in Vercel dashboard (required)
- Set `VITE_POSTHOG_KEY` for analytics (optional)
- Node.js 20.x required (specified in package.json engines)

## Troubleshooting Common Issues

### Development server won't start
- Clear node_modules: `rm -rf node_modules package-lock.json && npm install`
- Check Node version: `node -v` (should be 20.x)
- Port conflict: `npm run dev -- --port 3000`

### API routes failing locally
- Set `USE_PROD_API=true` in `.env.local` to proxy to production
- Or set `BLOB_READ_WRITE_TOKEN` for local Blob storage testing

### Tests failing
- Ensure `jsdom` environment is configured in `vitest.config.ts`
- Check that mocks in `src/test/setup.ts` are loaded
- Clear test cache: `npx vitest --clearCache`

### Build failing
- Run `tsx scripts/validate-data.ts` to check candidate data
- Check for TypeScript errors: `npm run typecheck`
- Verify all images referenced in data exist in `/public/candidates/`

### Performance degradation
- Run load tests: `npm run loadtest:smoke`
- Check bundle size: Build and analyze with `vite-bundle-analyzer`
- Verify lazy loading is working: Check Network tab in DevTools
- Ensure optimistic updates aren't awaiting API responses

## Additional Documentation

- `README.md` - Project overview and quick start
- `ENVIRONMENT.md` - Environment variables reference
- `dev.md` - Comprehensive developer documentation
- `CONTRIBUTING.md` - Contribution guidelines
- `docs/load-testing.md` - Load testing strategy
- `CODE_OF_CONDUCT.md` - Facts protocol and community guidelines

---
> Source: [SanGoku95/capibarismo](https://github.com/SanGoku95/capibarismo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
