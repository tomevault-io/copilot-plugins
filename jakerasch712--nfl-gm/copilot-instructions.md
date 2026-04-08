## nfl-gm

> AI assistant guidance for the **NFL Head Coach 2026** simulation game. Read this before making any changes.

# CLAUDE.md — NFL Head Coach 2026

AI assistant guidance for the **NFL Head Coach 2026** simulation game. Read this before making any changes.

---

## Project Overview

A client-side React/TypeScript NFL management simulation. Users pick a team and manage roster, salary cap, contracts, trades, the draft, scouting, coaching staff, and game-day strategy. Google Gemini AI powers two live features: real-roster syncing and draft strategy analysis.

**No backend, no database, no auth.** All state is in-memory; data resets on page reload.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | React 19 with hooks |
| Language | TypeScript 5.8 (strict) |
| Build | Vite 6, dev server on port 3000 |
| Styling | Tailwind CSS (dark theme, `slate-950` base) |
| Icons | Lucide React |
| Animations | Motion (Framer Motion v12) |
| Charts | Recharts |
| Markdown | react-markdown |
| AI | Google Gemini via `@google/genai` |

---

## Repository Structure

```
/
├── App.tsx               # Root component: global state, view routing
├── index.tsx             # React entry point
├── index.html            # HTML shell
├── types.ts              # ALL TypeScript interfaces and enums
├── constants.ts          # Static game data (teams, players, coaches, draft)
├── components/
│   ├── TeamSelection.tsx # Team picker grid (entry screen)
│   ├── Dashboard.tsx     # War Room HQ overview
│   ├── Navigation.tsx    # Sidebar nav, cap space, trust rating
│   ├── RosterView.tsx    # Roster management + AI roster sync
│   ├── FreeAgency.tsx    # Free agent marketplace
│   ├── TradeCenter.tsx   # Trade builder with Rich Hill values
│   ├── ContractNegotiation.tsx # Contract slider UI + agent feedback
│   ├── CapModals.tsx     # Restructure and release modals
│   ├── DraftRoom.tsx     # Draft simulator + AI strategy
│   ├── ScoutingView.tsx  # Scout management and prospect tracking
│   ├── StaffView.tsx     # Coaching staff management
│   ├── GamePlan.tsx      # Game week preparation
│   └── MatchSim.tsx      # Match simulation
├── services/
│   └── geminiService.ts  # Google Gemini AI integration
├── utils/
│   └── capUtils.ts       # NFL salary cap math (dead cap, restructures)
├── vite.config.ts
├── tsconfig.json
├── package.json
└── .env.example
```

---

## Key Files to Understand First

1. **`types.ts`** — The single source of truth for all data shapes. Read this before touching any component.
2. **`constants.ts`** — All static game data. New players, teams, coaches, and draft classes go here.
3. **`App.tsx`** — Global state lives here (prospects, scouts, picks, budget). View routing via `AppView` enum.
4. **`utils/capUtils.ts`** — Salary cap logic. Contains `calculateDeadCap()` and `executePlayerRelease()`.
5. **`services/geminiService.ts`** — AI calls. `syncTeamRoster()` and `getDraftStrategy()`.

---

## Development Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server at http://localhost:3000
npm run build        # TypeScript compile + Vite production build
npm run preview      # Preview production build
npm run lint         # TypeScript type checking (tsc --noEmit)
```

### Environment Setup

```bash
cp .env.example .env
# Set GEMINI_API_KEY=your_key_here
```

The Gemini API key is required for the "Sync Real Roster" and "AI Strategy" features. The app runs without it; those buttons will fail gracefully.

---

## Architecture Patterns

### State Management

All state uses React hooks. There is no Redux, Zustand, or Context API. Global state (team selection, draft picks, scouts, prospects) lives in `App.tsx` and is passed as props. Local UI state lives in each component.

**Adding new global state**: Add to `App.tsx` state and pass via props. Do not introduce external state libraries without discussion.

### View Routing

Navigation uses an `AppView` enum (defined in `types.ts`). The current view is a `useState` in `App.tsx`. To add a new view:
1. Add the value to the `AppView` enum in `types.ts`
2. Add a case in `App.tsx`'s render switch
3. Add a nav item in `Navigation.tsx`

### Data Flow

```
constants.ts (static data)
       ↓
App.tsx (useState, derives initial state)
       ↓
Component props (no drilling more than 2 levels)
       ↓
Services/Utils (pure functions, called from components)
```

### Salary Cap Math

Dead cap calculations follow real NFL rules. Key rules in `capUtils.ts`:

- **Standard cut**: All remaining signing bonus prorations accelerate to current year
- **Post-June 1 cut**: Current year gets year-1 proration; remaining prorations defer to next year
- **Proration** = total signing bonus ÷ contract length (integer division per year)
- Cap savings = base salary + current prorations relieved

Do not change cap math without understanding the NFL Collective Bargaining Agreement rules.

### AI Integration

`geminiService.ts` exports two async functions:

```typescript
syncTeamRoster(teamName: string): Promise<Player[]>
getDraftStrategy(picks: DraftPick[], board: DraftProspect[], needs: Position[]): Promise<string>
```

- `syncTeamRoster` uses `gemini-2.5-flash-preview` with Google Search grounding to fetch real rosters
- `getDraftStrategy` uses `gemini-2.5-pro-preview` with `thinkingConfig: { thinkingBudget: 8192 }` for deep analysis
- Both functions return parsed data or throw; callers handle loading/error state

**When modifying AI calls**: Keep prompts structured (JSON output for roster sync, Markdown for strategy). Test with the actual API before committing.

---

## TypeScript Conventions

- **Strict mode on.** No `any` unless absolutely unavoidable (and document why).
- All interfaces/enums are in `types.ts`. Do not define types inline in components.
- Use `type` for unions/aliases, `interface` for object shapes.
- Path alias `@/*` resolves to the project root (e.g., `import { Player } from '@/types'`).

### Key Types Reference

```typescript
// Player positions
enum Position { QB, RB, WR, TE, OL, DL, LB, CB, S, K }

// Application views
enum AppView { DASHBOARD, ROSTER, FREE_AGENCY, TRADE_CENTER, GAMEPLAN, MATCH, DRAFT, STAFF, SCOUTING }

// Core interfaces
interface Player { id, name, position, age, overall, contract, morale, fatigue, schemeFit, traits }
interface Contract { salary, bonus, years, deadCap, prorations, voidYears }
interface DraftProspect { id, name, position, grade, combineStats, scoutingReport }
interface Scout { id, name, region, specialization, level, experience }
interface Coach { id, name, role, traits, bonus, experience }
```

---

## Styling Conventions

- **Dark theme only.** Base color: `bg-slate-950`. Never use light backgrounds.
- Tailwind utility classes only. No inline `style` props except for dynamic values (e.g., bar widths).
- Color semantic conventions:
  - Green (`green-400/500`) → positive outcomes, good ratings, cap savings
  - Red (`red-400/500`) → negative outcomes, dead cap, warnings
  - Yellow/Amber (`yellow-400`) → caution, moderate risk
  - Blue (`blue-400/500`) → informational, selected states
  - Purple (`purple-400/500`) → AI features, premium actions
- Use `motion.div` (from the `motion` package) for meaningful transitions. Keep animations subtle.
- Lucide React for all icons. No other icon libraries.

---

## Game Data (constants.ts)

### Teams
`TEAMS_DB` contains all 32 NFL teams with `{ id, name, city, abbreviation, division, conference, offRating, defRating, stRating }`.

### Players
`MOCK_PLAYERS` contains ~15 player profiles, primarily Houston Texans. Players have full `Contract` objects with realistic salary figures.

### Draft
`DRAFT_CLASS` contains draft prospects with `grade` (A+ through D), `combineStats`, and `scoutingReports`.

**Rich Hill Draft Pick Values**: Trade values are calculated using the Rich Hill model (exponential decay by pick number). This is implemented inline in `TradeCenter.tsx` and `DraftRoom.tsx`.

---

## Common Tasks

### Adding a New Player to the Roster
1. Add a `Player` object to `MOCK_PLAYERS` in `constants.ts`
2. Follow existing contract structure; include all required fields from the `Player` interface

### Adding a New Team Feature/View
1. Add enum value to `AppView` in `types.ts`
2. Create component in `components/`
3. Import and render in `App.tsx` switch statement
4. Add nav item in `Navigation.tsx`

### Modifying Cap Logic
1. Read `capUtils.ts` fully before editing
2. Test both standard cut and post-June 1 cut paths
3. Verify dead cap + cap savings sum to the player's remaining cap hit

### Adding an AI Feature
1. Add a new export function to `geminiService.ts`
2. Use the appropriate model (`flash` for fast/simple, `pro` with thinking for deep analysis)
3. Always handle `GoogleGenAIError` gracefully in the calling component

---

## Known Limitations / In-Progress Areas

- **No persistence**: State resets on refresh. LocalStorage integration is not implemented.
- **No real opponent AI**: Match simulation uses static logic, not adaptive AI play-calling.
- **Partial team roster**: `MOCK_PLAYERS` only covers ~15 players; use "Sync Real Roster" to populate fully.
- **Single-team experience**: Multi-franchise/franchise mode is not implemented.
- **No unit tests**: The test infrastructure mentioned in early commits was not merged into the main codebase.

---

## What NOT to Do

- Do not add a backend server or database unless explicitly requested.
- Do not introduce state management libraries (Redux, Zustand, Jotai) without prior discussion.
- Do not use `any` type to silence TypeScript errors — fix the types.
- Do not hardcode player salaries or contract lengths in components — use `constants.ts`.
- Do not add light-mode styles — dark theme only.
- Do not change the Rich Hill trade value model without understanding it impacts all trade fairness logic.
- Do not make AI model calls directly from components — use `geminiService.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jakerasch712)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jakerasch712)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
