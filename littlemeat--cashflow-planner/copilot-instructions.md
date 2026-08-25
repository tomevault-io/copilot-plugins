## cashflow-planner

> Personal long-horizon cashflow planner for a Czech household planning maternity leave and a mortgage. 30-year stress-test of finances with stackable life events, salary changes, inflation, mortgage rate changes.

# Cashflow Planner — Agent Onboarding

Personal long-horizon cashflow planner for a Czech household planning maternity leave and a mortgage. 30-year stress-test of finances with stackable life events, salary changes, inflation, mortgage rate changes.

## Stack

Vite + React 19 + TypeScript + Tailwind CSS v4 + Zustand + Recharts + LocalStorage. No backend, no auth.

## Project structure

```
src/
  types/index.ts          — all TypeScript interfaces (Plan, Baseline, CashflowEvent, Mortgage, Asset, MonthlySnapshot, Period)
  store/usePlanStore.ts   — Zustand store + localStorage persistence + migratePlan()
  lib/
    simulate/
      index.ts            — simulate(plan): MonthlySnapshot[], getMonthDetail(plan, month): MonthDetail,
                            stepMonth() (shared core step), applyEvents(), computeAssetsValue()
      mortgage.ts         — MortgageState, initMortgageState(), stepMortgage()
      __tests__/          — 50 tests across 4 files (simulate, presets, phase4, hidden)
    formatters.ts         — formatCZK, formatRunway, formatPercent, dateToMonthOffset, monthOffsetToDate,
                            addMonths, formatYearMonth, formatFrequency, formatCategory, computeMonthlyPayment
    formatters.test.ts    — 8 tests for formatting utilities (lives at src/lib/, not in __tests__/)
    constants.ts          — Czech 2026 social/tax constants (PPM cap, rodičovský příspěvek, sleva na dítě)
    seedData.ts           — default plan seeded on first run (uses growthSchedule, schemaVersion: 1)
  components/
    AmountInput.tsx       — controlled number input with space thousand separators (cs-CZ)
    CollapsiblePanel.tsx  — shared collapsible card wrapper (title, defaultOpen?, headerRight?, children)
    Modal.tsx             — shared backdrop modal with safe mouseDownTarget close pattern
    InfoTooltip.tsx       — shared SVG info icon + tooltip (text, width?, position?)
    EyeIcon.tsx           — shared eye / eye-slash SVG icon for visibility toggles
    KpiCard.tsx           — single KPI tile used in ResultsSummary
    BaselinePanel.tsx     — start date, cash, investments, yield, safety buffer, horizon
    EventsPanel.tsx       — CRUD + drag-and-drop sort for income/expense events; eye toggle for hidden
    EventForm.tsx         — add/edit event modal (growthSchedule editor, end date)
    MortgagePanel.tsx     — CRUD for mortgages; eye toggle for hidden
    MortgageForm.tsx      — add/edit mortgage modal with live amortization summary
    AssetsPanel.tsx       — CRUD for assets (property, investments); eye toggle for hidden
    AssetForm.tsx         — add/edit asset modal with linked expense/mortgage pickers
    MonthDetailModal.tsx  — per-event income/expense breakdown for a clicked month
    ResultsPanel.tsx      — thin wrapper: <ResultsSummary /> + <ResultsChart /> (kept for backward compat)
    ResultsSummary.tsx    — KPI bar, warning banner, date slider, Runway card
    ResultsChart.tsx      — line chart + yearly/monthly table toggle + CSV export button
    presets/
      PresetPicker.tsx        — dropdown triggering preset wizards
      MaterskaPreset.tsx      — PPM maternity benefit (calculate from income or manual for OSVČ)
      RodicovskaPreset.tsx    — parental allowance (rodičovský příspěvek)
      SlevaNaDitePreset.tsx   — child tax credit with optional scheduled increases
      ZvyseniPlatuPreset.tsx  — salary raise (% change on existing income event)
      InflacniSkok.tsx        — adds a new growthSchedule entry on selected expense events from a chosen date
      NavratDoPrice.tsx       — return-to-work income event
      PrevzetiHypoteky.tsx   — mortgage takeover wizard (one-time cash payment + mortgage, atomic batchUpdate)
  App.tsx — 3-row layout: KPI bar → (baseline | events+mortgages+assets) → chart
```

## Core architecture

### Month offsets
All event/mortgage/asset timing is stored as integer month offsets from `baseline.startDate` ("YYYY-MM"). Never store dates directly. Convert with `dateToMonthOffset(date, startDate)` and `monthOffsetToDate(offset, startDate)`. Use `<input type="month">` in forms and convert on change.

### simulate(plan): MonthlySnapshot[]
Pure function — no side effects, no React dependencies. Called on every plan change. Produces one snapshot per month for `horizonYears * 12 + 1` months.

Core loop: `stepMonth(plan, state, m): StepResult` — extracts one month's computation into a shared function used by both `simulate()` and `getMonthDetail()`. Do not duplicate this logic.

`applyEvents(plan, m)` uses `isActiveAt(period, m)` from `Period` type for half-open interval checks. Items with `hidden: true` are skipped entirely (no income/expense/mortgage/asset contribution).

### growthSchedule on events
Each `CashflowEvent` has `growthSchedule: Array<{ id, fromMonth, rateAnnual }>` — an ordered list of rate periods relative to the event's `startMonth`. Growth compounds across periods: `growthFactor *= (1 + rateAnnual/12)^monthsInPeriod`. This replaces the old single `annualGrowthPct` field. `migratePlan()` handles the conversion from old plans.

### Safety buffer sweep
Each month: surplus cash above `safetyBufferMonths × avgExpenses` auto-moves to investments. If cash goes negative, withdraws from investments first. `avgExpenses` = trailing **12-month** rolling average of (recurring expenses + mortgage payments + insurance). One-off events are excluded from the average.

### rebaseOffsets
When `startDate` changes, `rebaseOffsets()` in the store recalculates all event/mortgage/asset month offsets to preserve absolute calendar dates. Critical: without this, changing the start date shifts all events.

### Mortgage extra payments
Stored as `Mortgage.extraPayments: Array<{ id, month, amount, strategy }>`. `stepMortgage()` in `mortgage.ts` handles each lump sum — strategy `"shorten-term"` recalculates monthly payment from remaining principal + shorter term; `"lower-payment"` keeps original term length.

### Hidden items
`CashflowEvent`, `Mortgage`, and `Asset` all have `hidden?: boolean` (optional — stays `undefined` on old plans, treated as `false`). When `true`, the item is completely skipped in simulation (no cashflow, no mortgage balance, no asset value). Toggled via eye icon using dedicated store actions `toggleEventHidden(id)` / `toggleMortgageHidden(id)` / `toggleAssetHidden(id)` — these do **not** push to undo history (visibility is not undoable).

### Plan schema versioning + migration
`Plan.schemaVersion: number` tracks format version. `migratePlan()` in the store handles:
- Missing `assets: []` (pre-assets plans)
- `extraPayment` → `extraPayments[]` with UUID id
- `annualGrowthPct` → `growthSchedule` with single entry
- Missing `schemaVersion` → defaults to 1

## Key patterns

### AmountInput
Use for all CZK number inputs. Shows space-separated format on blur, raw number on focus. Enforces `min` on blur. Never use `<input type="number">` for CZK amounts.

### CollapsiblePanel
All major sections (BaselinePanel, EventsPanel, MortgagePanel, AssetsPanel) use `<CollapsiblePanel title="..." headerRight={<buttons />}>`. Never duplicate the collapse+chevron pattern inline.

### Modal
All dialogs (EventForm, MortgageForm, AssetForm, MonthDetailModal, presets) use `<Modal onClose={...} maxWidth="...">`. The component encapsulates the `mouseDownTarget` ref trick that prevents the modal closing when the user drags text inside to the backdrop.

### batchUpdate for atomic operations
Preset wizards that insert multiple events use `batchUpdate(recipe: (plan) => plan)` — produces a single undo history entry. Never call `addEvent` multiple times in a preset.

### setTimeout auto-close in preset wizards
Always use `useEffect` + `clearTimeout` cleanup, never bare `setTimeout`:
```tsx
useEffect(() => {
  if (!success) return;
  const tid = setTimeout(onClose, 1500);
  return () => clearTimeout(tid);
}, [success, onClose]);
```

### Preset events
Presets insert regular `CashflowEvent` entries with a `presetGroup: string` (UUID) to group related events. `deletePresetGroup(groupId)` removes all events in the group atomically. No hidden simulation logic in presets.

## CSV export
`ResultsChart` has an `exportCSV()` function triggered by the "↓ CSV" button (visible only in table mode). Exports the currently displayed table (monthly or yearly). Format: `;` separator (Czech Excel standard), UTF-8 BOM, raw integers (no "Kč"), filename `cashflow-YYYY-MM-DD.csv`.

## UI rules
- All UI strings in Czech. Code, comments, types in English.
- Tailwind only — no CSS-in-JS, no inline styles except dynamic chart colors.
- Number display: `formatCZK()` — space thousand separator, "Kč" suffix, no decimals for amounts ≥ 100.
- Chart x-axis: adaptive tick spacing — every year (≤10y), every 2y (≤20y), every 5y (>20y). Prevents Recharts hiding ticks.
- Hidden items: rendered at `opacity: 0.4` in their panel, eye-slash icon shown.

## Czech domain constants (2026)
```
PPM_MAX_MONTHLY_2026 = 49_020           // maternity benefit monthly cap
RODICOVSKY_PRISPEVEK_DEFAULT = 350_000  // parental allowance total
SLEVA_NA_DITE_2026 = [1_267, 1_860, 2_320]  // per child per month (1st, 2nd, 3rd+)
```

## Running the project
```
npm run dev     # start dev server (port 5173)
npm run build   # production build
npm test        # vitest (58 tests: 50 in simulate/__tests__/, 8 in formatters.test.ts)
npx tsc --noEmit  # type check
```

## What's been built

- **Phase 1**: Scaffolding, types, store, simulate, seed data, BaselinePanel, EventsPanel, MortgageForm with amortization summary, ResultsPanel with chart and table, JSON export/import.
- **Phase 2**: Safety buffer sweep, investments yield, chart series toggles, KPI summary bar, negative cash warnings, runway metric with tooltip.
- **Phase 3**: AmountInput, preset wizards (Mateřská, Rodičovská, Sleva na dítě, Zvýšení platu), annual growth rate on events with inflation suggestions.
- **Phase 4**: InflacniSkok preset, NavratDoPrice preset, MonthDetailModal (click any chart month or table row), accessibility improvements.
- **Architectural refactor**: CollapsiblePanel, Modal, InfoTooltip shared components (removed ~10× duplicated patterns); ResultsPanel split into ResultsSummary + ResultsChart + KpiCard; `stepMonth()` extracted as shared simulation core; `growthSchedule` replaces `annualGrowthPct`; `batchUpdate` + `deletePresetGroup` in store; `targetCash` in MonthlySnapshot; `schemaVersion` + `migratePlan()` for backward compat; AssetsPanel + AssetForm added.
- **CSV export**: Monthly/yearly table export from ResultsChart, Czech Excel format.
- **Hidden items**: `hidden?: boolean` on CashflowEvent, Mortgage, Asset — eye toggle in all three panels, items skipped entirely in simulation. Toggle calls `toggleEventHidden` / `toggleMortgageHidden` / `toggleAssetHidden` (no undo history entry). `EyeIcon` shared component in `EyeIcon.tsx`.
- **Audit fixes**: atomic `PrevzetiHypoteky` (batchUpdate), MortgagePanel multi-rate display fix, `formatRunway` in `formatters.ts`, growthSchedule pre-sorted once in `simulate()` + `getMonthDetail()`, `"﻿"` BOM escape in CSV, `revokeObjectURL` deferred via setTimeout, MaterskaPreset submit disabled at offset 0, all eye buttons have `aria-label`.

## Known decisions / non-obvious choices
- `growthSchedule[].id: string` / `rateSchedule[].id: string` — stable UUID keys for React list rendering, not array index.
- `getMonthDetail()` runs a full simulation up to `targetMonth` — it's `useMemo`'d in MonthDetailModal.
- Yearly events (`frequency: "yearly"`) fire on months where `(m - startMonth) % 12 === 0`, not on a fixed calendar month.
- `hidden?: boolean` is **optional** (not required) — `undefined` and `false` are both treated as visible. Keeps old JSON backups fully compatible without migration. Do not change to required.
- `hidden` mortgage: its `MortgageState` is carried through the loop untouched (principal stays frozen) and excluded from `mortgageBalance` in the snapshot. `simulate()` always starts fresh from `initMortgageState`, so there is no stale carry-over between user interactions — toggling hidden back on resumes from the correct remaining principal.
- Visibility toggles (`toggleEventHidden` etc.) intentionally do NOT push to undo history. 20 eye-clicks would fill the entire undo stack and bury meaningful edits. Undo only applies to structural changes (add, update, delete).
- `growthSchedule` entries are pre-sorted once at the top of `simulate()` and `getMonthDetail()` for events with multiple schedule entries. `applyEvents()` still contains its own sort call, which becomes a no-op on already-sorted data — this is intentional redundancy for safety, not a bug.
- `MortgagePanel` display calculation for multi-rate mortgages iterates through periods with a running `principal` variable — do not revert to the old `principalAtMonth()` helper which always used the first-period rate and gave wrong results for 3+ rate schedules.
- `InflacniSkok` does NOT split events — it appends a new entry to `growthSchedule` at `fromMonth = targetOffset - evt.startMonth`. This is atomic and undoable.
- Safety buffer uses a **12-month** trailing average of recurring expenses (excludes one-off events). This smooths out large annual payments (holidays, insurance) so they don't distort the buffer target.
- `rebaseOffsets` also rebases `asset.acquisitionMonth` — assets must stay at their original calendar dates when `startDate` changes.
- CSV BOM must be the escape `"﻿"` in source, not a literal invisible `U+FEFF` character. The literal gets silently stripped by some editors, minifiers, or copy-paste and is invisible during code review.
- CSV columns currently contain only numbers and `YYYY-MM` date strings — no user-supplied text. If a future column adds event names or notes, add a sanitisation step to strip leading `=`, `+`, `-`, `@` characters (CSV formula injection).
- All preset wizards must use `batchUpdate(recipe)` for any multi-entity operation. Never call `addEvent` + `addMortgage` separately — it produces two undo entries and leaves a dangling expense if the user immediately undoes.
- `PrevzetiHypoteky` creates the one-off expense event without a `presetGroup` (intentional — the mortgage has no group concept). This means `deletePresetGroup` cannot remove it; the user deletes the expense and mortgage separately. Do not add a "smazat preset" button to PrevzetiHypoteky entries.
- `deleteEvent` cascades: clears `linkedExpenseId` on any assets that referenced the deleted event. `deleteMortgage` cascades: clears `linkedMortgageId` on any assets that referenced the deleted mortgage. Both happen inside the same store mutation — no separate action needed.
- `formatPercent` is exported from `formatters.ts` but not currently used anywhere in the app. Do not remove it — it is available for future use.

## Keeping this file up to date
Update CLAUDE.md whenever: new components are added, simulation logic changes, a new pattern is established, or a feature is completed. The file is the primary onboarding document for new agent sessions — stale information causes mistakes.

---
> Source: [littlemeat/cashflow-planner](https://github.com/littlemeat/cashflow-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
