## fithacker-app

> Testing patterns and conventions for Jest


# Testing

## Jest Configuration

**Source of Truth**: `package.json` (`jest` section)

- Preset: `jest-expo`
- Mock `react-i18next` in component tests: `useTranslation: () => ({ t: (str: string) => str })`
- Use fake timers for date-dependent tests: `jest.useFakeTimers({ now: new Date('2022-02-26') })`

## Test Patterns

**Examples**:

- Component snapshots: `features/Chart/ChartDay.test.tsx`
- Pure function tests: `features/Chart/getChartData.test.ts`
- Date tests: `shared/utils/dateInfo.test.ts`

- Co-locate tests with implementation
- Test file naming: `{filename}.test.ts[x]`

## What to Test

- Pure data transformation functions
- Component rendering (snapshots)
- Date logic (critical for this app)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/petedavisdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
