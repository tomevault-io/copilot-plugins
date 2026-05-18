## break-tracker

> A real-time decision signal tool for sports card breaks. Users are in live streams, emotional, and need fast trust signals — not data to interpret.

# CLAUDE.md — Break Tracker Development Rules

## What This Product Is

A real-time decision signal tool for sports card breaks. Users are in live streams, emotional, and need fast trust signals — not data to interpret.

Output is always one of: 🔴 Overpriced / 🟡 Fair / 🟢 Value / 🔥 Strong Buy Zone

Numbers are context, never the message.

---

## Non-Negotiable Rules

- No backend. No server. No API calls. All logic runs client-side.
- No database. No persistence. No localStorage for state (session-only).
- No authentication. No user accounts. No identity.
- No external state libraries (no Redux, Zustand, Jotai, etc.). React `useState` and `useReducer` only.
- No CSS-in-JS runtime libraries (no styled-components, Emotion). Use Tailwind or CSS modules.
- Signal output must always be computed in < 1ms. It must never be async.
- The primary signal (emoji + label) must be visible without scrolling on any device.
- Never render raw computed numbers as the primary UI element. Numbers are always secondary.

---

## Architecture Boundaries

**Allowed:**
- Next.js (App Router, static export)
- React hooks for all state
- Tailwind CSS for styling
- Plain TypeScript utility functions for computation
- `useMemo` for derived values

**Not Allowed:**
- Server components that fetch data
- API routes
- Any npm package that adds > 10KB gzipped to the bundle without clear justification
- Class components
- Context API for anything other than a single top-level config value (e.g., theme)

---

## State Model

Static template data is not dynamic state. Teams belong to `BreakTemplate`, not `BreakState`.

```ts
type Team = {
  id: string;
  name: string;
  weight: number;
};

type BreakTemplate = {
  id: string;
  name: string;
  teams: Team[];
};

type BreakState = {
  removedTeamIds: Set<string>;
  purchasedTeamIds: Set<string>;
  priceInput: number | null;
};
```

Derived values (never stored in state):
- `remainingTeams` — filter from `template.teams` minus `removedTeamIds`
- `remainingValue` — sum of weights of `remainingTeams`
- `fairValue` — `remainingValue / remainingTeams.length`
- `signal` — derived from `fairValue` vs `priceInput`

**Rules:**
- Derived values are computed inline or with `useMemo`. Never stored in state.
- State updates are synchronous. No async state mutations.
- Never nest state. Flat structures only.
- `removedTeamIds` and `purchasedTeamIds` use `Set<string>` keyed by team ID. No arrays of objects for these.

---

## Signal Computation

```ts
function getSignal(fairValue: number, price: number): Signal {
  const ratio = price / fairValue;
  if (ratio < 0.85) return 'STRONG_BUY';
  if (ratio < 0.98) return 'VALUE';
  if (ratio < 1.10) return 'FAIR';
  return 'OVERPRICED';
}
```

- Thresholds live in a single `constants.ts` file. Nowhere else.
- This function is pure. It takes numbers and returns a string. No side effects.
- Must have unit tests. This is the core of the product.

---

## Component Design Rules

**Split a component when:**
- It has its own local state unrelated to the parent
- It renders a repeated unit (e.g., each team in a list)
- It is reused in more than one place

**Do NOT split a component when:**
- The only reason is "it's getting long"
- The split would require threading props through a new intermediate component
- The extracted component would never be reused and has no independent state

**Max component depth for any feature: 3 levels.** If you're going deeper, the component model is wrong.

**Props:**
- No prop drilling past 2 levels. Lift state or rethink layout.
- No "god props" (objects with 10+ fields passed to a child). Pass what the child needs.

---

## Performance Rules

These must always be O(1) or trivially fast:

- Signal computation (`getSignal`)
- Rendering the signal display
- Any click handler on team removal

**Avoid:**
- `useEffect` for computing derived values — use `useMemo` or inline computation
- Sorting or filtering inside render without `useMemo`
- Re-rendering the full team list on every price keystroke — debounce price input at 150ms or derive signal only from the final committed value

---

## Validation Rules

- Validate numeric inputs at boundaries only — not deep inside compute functions
- Clamp impossible values instead of throwing errors
- Never block UI interaction because of a validation failure
- Prefer sane defaults over exceptions
- Pure compute functions must never return `NaN` — guard all division and coerce bad inputs at the edge
- Derived state must always have safe fallbacks (e.g., `fairValue` returns `0` when no teams remain, signal returns `null`)

---

## Testing Rules

**Must have unit tests:**
- All pure compute functions in `lib/`
- `getSignal` — every threshold boundary
- `computeFairValue` — including empty remaining set
- Edge cases: 0 remaining teams, `null` price input, duplicate toggles, empty templates

**Do NOT test:**
- UI snapshots
- Tailwind class output
- Click handlers in isolation
- E2E flows in v1

**Approach:**
- Vitest (or Jest if already present). No browser-based test runner.
- Tests live in a top-level `tests/` directory.
- No mocking — there are no network calls to isolate in v1.
- Each test file must be readable in under 2 minutes. If it isn't, it's too complex.

---

## Overengineering Smells

Stop and reconsider if you are:

- Introducing an abstraction before a second use case exists
- Adding global state without a measured need for it
- Building a generic component system when only one variant exists
- Optimizing render performance before profiling shows a problem
- Adding a configuration layer for logic that is fixed business rules
- Splitting a file or component because it "feels big"
- Writing a custom hook that is only called in one place

---

## Refactor Rules

- Refactor only after repeated pain — not after one instance of friction
- Duplication is acceptable early; remove it when it causes actual bugs or drift
- Prefer explicit, readable code over reusable abstractions in v1
- Optimize for readability over cleverness — the next reader is you in 3 weeks
- Small files are not automatically better files; splitting adds indirection
- A refactor that adds new files and lines without removing old ones is not a refactor — it is scope creep

---

## Iteration Philosophy

- Ship the simplest version that produces a correct signal
- Add configuration only when the absence causes real user pain, not theoretical pain
- Extend the state model only when a new user-facing behavior requires it — never pre-emptively
- Before adding any new component, file, or hook, ask: "Would removing this break a user-visible behavior?" If no, don't add it.
- V1 is a static frontend. V2 might add data. Design V1 as if V2 will never exist. Migrate if V2 happens.

---

## File Structure

```
/app
  page.tsx          # single page, root layout
/components
  SignalDisplay.tsx  # the primary output: signal + label
  TeamList.tsx       # list of teams with remove toggle
  PriceInput.tsx     # price entry
/lib
  signal.ts          # getSignal(), computeFairValue()
  constants.ts       # thresholds, team weights
  types.ts           # shared TypeScript types
/tests
  signal.test.ts
```

Adding a new top-level directory requires justification in the PR description.

---

## Git Workflow Rules

- One branch per logical change. Never push new work to a branch whose PR has already been merged.
- After a PR merges, cut a new branch from `main` for the next change.
- Before referencing a PR URL, confirm it is open — not closed or merged.
- Branch names must describe the change, not the session or agent that made it.

---

## Definition of Done

A feature is done when:
1. Signal output is correct for all threshold cases
2. Unit tests pass for any pure logic added
3. The UI renders the signal as the primary element
4. No new abstractions were added that aren't used
5. Bundle size did not increase by more than 5KB gzipped

---
> Source: [biversen21/break-tracker](https://github.com/biversen21/break-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
