## tempo-analytics

> Every recharts chart must receive **pre-shaped data from a server function** in `src/lib/`. Never pivot or transform data inside a `'use client'` component — recharts silently renders invisible bars/lines when values are computed client-side.

# tempo-analytics — Dev Notes

## Chart data

Every recharts chart must receive **pre-shaped data from a server function** in `src/lib/`. Never pivot or transform data inside a `'use client'` component — recharts silently renders invisible bars/lines when values are computed client-side.

**Static dataKeys** — server returns `Array<{ day: string; my_metric: number }>`, component uses `<Bar dataKey="my_metric" />`.

**Dynamic/stacked dataKeys** — server returns `{ days: Array<Record<string, string | number>>, series: Array<{ id, label, total }> }`, component maps over `series` to render `<Bar key={s.id} dataKey={s.id} />`.

## Contract tests

`__tests__/lib/chart-data-contracts.test.ts` — one test per chart data function, using helpers from `__tests__/helpers/chart-contract.ts`:

- `expectRechartsRows(rows, ['field1', 'field2'])` — static dataKeys
- `expectPivotContract(data.days, data.series.map(s => ({ key: s.id })))` — dynamic dataKeys

Add a contract test for every new chart data function.

## CI

Run `npm test` and `npm run build` before committing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Evan-Kim2028)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Evan-Kim2028)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
