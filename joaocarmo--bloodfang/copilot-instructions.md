## bloodfang

> Blood Fang is a strategic card game on a 3x5 grid. pnpm monorepo with two packages:

# CLAUDE.md — Blood Fang

## Project overview

Blood Fang is a strategic card game on a 3x5 grid. pnpm monorepo with two packages:

- **`packages/engine`** — Pure TypeScript game logic library. Zero dependencies. Exports pure functions (`createGame`, `playCard`, `pass`, `getValidMoves`, etc.).
- **`packages/client`** — React 19 + Vite web UI. Consumes the engine. Tailwind CSS v4, Zustand, TanStack Router, Lingui i18n, Motion.

The engine knows nothing about card names, art, or lore — only mechanics. The client maps engine card IDs to creative identity.

## Commands

```bash
# Build
pnpm -F @bloodfang/engine build      # tsdown (ESM + dts)
pnpm -F @bloodfang/client build      # tsc --noEmit && vite build

# Test
pnpm -F @bloodfang/engine test       # vitest (218 tests)
pnpm -F @bloodfang/client lint       # eslint src/

# Dev
pnpm -F @bloodfang/client dev        # vite dev server

# Format
pnpm format                           # prettier --write .
pnpm format:check                     # prettier --check .

# i18n
pnpm -F @bloodfang/client i18n:extract
pnpm -F @bloodfang/client i18n:compile
```

Always run `pnpm -F @bloodfang/client build` after client changes — it runs both `tsc --noEmit` (type-check) and `vite build`. Run `pnpm -F @bloodfang/client lint` to ensure zero ESLint errors/warnings. Both the engine and client have vitest test suites.

## Code style

- **Prettier**: single quotes, trailing commas, 100 char print width, semicolons, LF endings
- **TypeScript**: strict mode with `exactOptionalPropertyTypes`, `noUncheckedIndexedAccess`, `noUnusedLocals`, `noUnusedParameters`, `verbatimModuleSyntax`
- **ESLint**: flat config with `typescript-eslint`, `jsx-a11y` (accessibility), `react-hooks`
- Unused parameters must be prefixed with `_` (ESLint `@typescript-eslint/no-unused-vars` pattern `^_`)
- Never use `autoFocus` prop — use `useEffect` + `ref.current?.focus()` instead (jsx-a11y/no-autofocus)
- Never use `eslint-disable` comments unless absolutely necessary; remove stale ones
- Remove unused imports immediately — don't leave dead code

## Architecture patterns

### Engine (`packages/engine`)

- Pure functions: state in, new state out. No side effects.
- Discriminated unions for abilities (trigger + effect types) with exhaustive switch/match.
- `GamePhase` enum for game state machine (`Mulligan`, `Playing`, `Ended`).
- All types exported as type-only from `index.ts`.
- Tests in `*.test.ts` files adjacent to source.

### Client (`packages/client`)

- **State**: Zustand stores (`useGameStore`, `useDeckStore`, `useSettingsStore`). `useSettingsStore` uses `zustand/persist` with `localStorage` (key: `bloodfang-settings`). Use `useMemo` in selector helpers to prevent re-renders from new array refs.
- **Routing**: TanStack Router v1. Route paths defined as constants in `routes.ts`. Navigation via `useNavigate()`.
- **i18n**: Lingui with `t` tagged template from `@lingui/core/macro`. All user-visible strings must use `t` for translation.
- **Styling**: Tailwind CSS v4 with custom tokens in `app.css` `@theme` block. No separate `tailwind.config.ts`.
- **Animations**: Motion (`motion/react`). Always check `useReducedMotion()` — pass `false` to `initial` when reduced motion is preferred.
- **Accessibility**: aria labels on interactive elements, `sr-only` for screen-reader-only text, `role` attributes on custom widgets, roving tabindex for keyboard navigation in grids/lists.

### Component conventions

- Functional components only.
- Props interfaces defined above the component.
- `forwardRef` for UI primitives (e.g., `Button`) that need ref forwarding.
- Tailwind classes via template literals with conditional expressions.
- Focus management via `useRef` + `useEffect`, never `autoFocus`.
- Extract shared UI patterns into reusable components in `components/ui/`. Prefer composing existing primitives over duplicating markup.

## Semantic HTML and accessibility

Follow WCAG 2.1 AA. This is enforced by `eslint-plugin-jsx-a11y` but the linter only catches a subset — apply these rules by default:

- **Semantic elements first**: use `<main>`, `<section>`, `<aside>`, `<nav>`, `<h1>`-`<h6>`, `<dialog>`, `<table>`, `<button>` over generic `<div>`/`<span>`. Only reach for `<div>` when no semantic element fits.
- **Heading hierarchy**: one `<h1>` per page/screen, then `<h2>`, `<h3>` in order. Never skip levels.
- **Interactive elements**: use `<button>` for actions, `<a>` for navigation. Add `role`, `aria-label`, `aria-pressed`, `aria-disabled` when native semantics are insufficient.
- **Keyboard navigation**: all interactive elements must be reachable via Tab. Custom widgets (grids, listboxes) use roving tabindex with arrow key handlers.
- **Screen readers**: use `sr-only` class for visually hidden labels, `aria-live="polite"` for dynamic content (e.g., ActionLog), `role="status"` for live score regions.
- **Reduced motion**: always check `useReducedMotion()` from Motion and disable animations accordingly.
- **Color contrast**: never rely on color alone to convey meaning — pair with text labels, icons, or patterns.

## Tailwind responsive design

Mobile-first with breakpoints: base (< 640px), `sm:` (640px), `md:` (768px), `lg:` (1024px).

Pattern: start with the smallest value, scale up at each breakpoint:

```
text-2xl sm:text-3xl md:text-4xl
gap-2 p-2 sm:gap-3 sm:p-3 md:gap-4 md:p-4
```

All layout components should have responsive classes. No fixed widths/heights without responsive alternatives. No inline `style` props for sizing — use Tailwind arbitrary values (`min-w-[100px]`) instead.

## Custom design tokens

Defined as CSS custom properties in `packages/client/src/app.css`:

- **Players**: `text-p0` (blue), `text-p1` (red), `-light` variants
- **Card ranks**: `rank-1` (bronze), `rank-2` (silver), `rank-3` (gold), `rank-replacement` (purple)
- **Power states**: `power-buff` (green), `power-debuff` (red)
- **Surfaces**: `surface`, `surface-raised`, `surface-overlay`
- **Text**: `text-primary`, `text-secondary`, `text-muted`
- **Board**: `tile-empty`, `tile-p0`, `tile-p1`

## Monorepo

- **Package manager**: pnpm with workspaces. Node v24 (`.nvmrc`).
- **Filter syntax**: `pnpm -F @bloodfang/engine <cmd>`, `pnpm -F @bloodfang/client <cmd>`
- **Dependency direction**: client -> engine (never the reverse). Engine has zero npm dependencies.
- **Build tool**: tsdown for engine + server, Vite for client.

## File organization

```
packages/engine/src/         Game logic (board, game, abilities, effects, scoring, card-database)
packages/client/src/
  components/
    board/                   Board, Tile, LaneScores, PawnDots, PlacedCard
    card/                    Card, CardMini, CardDetail, CardArt, CardPreviewPopup, CardPreviewTrigger, RankIcon, PowerBadge, RangeGrid, AbilityText
    game/                    GameScreen, ActionLog, PassButton, MulliganScreen, ResultsScreen, TurnTransition, TurnIndicator
    hand/                    Hand, HandCard
    deck-builder/            DeckBuilder, CardCatalog, DeckSlots, FilterBar
    screens/                 HomeScreen, SetupScreen, RulesScreen, SettingsScreen, AboutScreen
    ui/                      Button, BackButton, ConfirmDialog, ContentDialog, DialogBase, LinkButton, ToggleButton, RouteAnnouncer
  store/                     Zustand stores (game-store, deck-store, settings-store)
  hooks/                     Custom hooks (use-valid-moves, use-lane-scores, use-small-screen, use-placement-preview, use-dialog)
  lib/                       Utilities (card-identity, player-color)
  locales/                   Lingui message catalogs
packages/client/public/art/  166 card artwork WebP files (DALL-E 3 watercolor style)
data/                        Reference data (flavor-text.json, card-mapping.json, art-prompts.json, seeds.json, expected_ranges.json)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joaocarmo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joaocarmo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
