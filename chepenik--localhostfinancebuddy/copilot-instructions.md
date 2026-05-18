## localhostfinancebuddy

> validates each row's shape and rejects malformed data silently. This is the

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev         # Next.js dev server (Turbopack)
npm run build       # production build — runs typecheck as part of build
npm run typecheck   # tsc --noEmit (standalone)
npm run lint        # ESLint flat config; do NOT use `next lint` (removed in Next 16)
npm run validate    # Node 22.6+ assertion script over lib/finance.ts + lib/types.ts
```

There is no test runner. `npm run validate` is the project's only automated check
beyond the build — it imports the TypeScript modules directly via Node's
`--experimental-strip-types`, so any change to `lib/finance.ts` or `lib/types.ts`
should be re-validated. To add a single new assertion, edit
`scripts/validate-finance.mjs` and re-run; it's not a framework, just `node:assert`.

Node 22.6+ is required because of the `--experimental-strip-types` flag the
validator depends on. The build itself works on any Node ≥18.18.

## Architecture

### Local-first by design

The entire data layer is **one localStorage key (`lfb:state:v1`)** holding
`{ version: 1, entries: Entry[] }`. There is no backend, no auth, no database,
no API. Don't add any of those. If you need persistence beyond a single browser,
the export/import JSON path is the answer — see `app/settings/page.tsx`.

### The Entry model and `coerceEntry`

`lib/types.ts` defines a discriminated union `Entry = IncomeEntry | ExpenseEntry
| AssetEntry | LiabilityEntry`. **Every value entering the app from outside
memory** (localStorage parse, JSON import) must go through `coerceEntry`, which
validates each row's shape and rejects malformed data silently. This is the
seam that protects the rest of the codebase from corrupt/legacy/partial data —
do not bypass it.

### `toMonthly` is the core abstraction

Income and expenses can have any frequency (weekly/monthly/yearly/one-time),
but the dashboard speaks in **monthly run-rate**. Every flow number visible to
the user is normalized through `toMonthly` in `lib/finance.ts`. Notable
behaviors:

- `one-time` → 0 (intentional: doesn't recur, so doesn't belong in a run-rate).
- Negative / NaN / Infinity → 0 (defensive; `coerceEntry` would normally catch this).
- Assets and liabilities never go through `toMonthly` — they're point-in-time balances.

`summarize(entries)` is the single function that derives every dashboard number
(net worth, cash flow, savings rate, debt-to-asset, 12-month projection, health
rating). New metrics belong here.

### Reactive state via `useSyncExternalStore`

`lib/useEntries.ts` exposes `{ entries, hydrated }`. Two stores are involved:

1. An **entries store** that subscribes to both the cross-tab `storage` event
   and a same-tab `lfb:state-changed` `CustomEvent`. Mutations in
   `lib/storage.ts` dispatch this event after writing, so all open tabs/tab
   listeners stay in sync.
2. A **`useHydrated()` store** that flips to `true` after first subscribe via
   `queueMicrotask`. This exists because `eslint-config-next@16` enforces
   `react-hooks/set-state-in-effect`, which forbids the typical
   `useEffect(() => setMounted(true), [])` pattern.

`hydrated` is the right gate for SSR-mismatch-sensitive UI (e.g. "do you have
data?" branches). The dashboard renders the empty hero during SSR + first
client paint, then swaps to populated content once `hydrated` is true.

### Recharts gotchas

Recharts defaults the **tooltip item text and legend label color to the slice
color**, which can be invisible on dark backgrounds. `components/charts/CategoryPie.tsx`
explicitly overrides `itemStyle`, `labelStyle`, and uses a Legend `formatter`
to keep all chart text in `var(--color-fg)`. Chart palette tokens
(`--color-c1`–`c8`) are defined in **both** `:root` and `html.dark` in
`globals.css` — don't add a new palette token without a dark override.

### Theming

`components/ThemeScript.tsx` is inlined in `<head>` to set `html.dark` *before*
hydration based on `localStorage["lfb:theme"]` or the system preference. This
prevents a flash of the wrong theme. `ThemeToggle` mutates the class and
persists; both the entries store and the theme observer use
`useSyncExternalStore` rather than effects.

### Dashboard hierarchy is intentional

`app/page.tsx` is structured around **net worth as the hero** (`NetWorthHero`),
followed by three calm metric tiles, then the assets/liabilities split bar,
then category breakdowns. The previous "8 cramped tiles" treatment was
deliberately removed. New dashboard surface area should respect this hierarchy:
add to existing sections rather than introducing new top-level cards that
compete with the hero.

### Things that are deliberate, not omissions

- No state library, no UI kit, no test runner. The app is small enough that
  hooks + Tailwind + a single chart library are sufficient.
- No Vercel-specific config; the app is just a Next.js project that happens to
  deploy anywhere.
- `next.config.ts` is intentionally minimal — `vercel.ts`/`vercel.json` are not
  used.
- `formatCurrency` defaults to USD with the user's locale; multi-currency is
  out of scope for v1 (see README roadmap).

---
> Source: [Chepenik/localhostfinancebuddy](https://github.com/Chepenik/localhostfinancebuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
