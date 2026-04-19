## wiroom

> This file provides guidance to Claude Code (claude.ai/code) when working with

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Development Commands

- `npm run dev` - Start Vite development server (usually http://localhost:5173)
- `npm run build` - Build production version
- `npm run preview` - Preview production build
- `npm test` - Run all tests with Vitest
- `npm run typecheck` - Run TypeScript type checking
- `npm run format` - Format code with Prettier

## Project Architecture

This is a React-based pension calculator that helps users determine if they're
financially prepared for retirement. The app uses Monte Carlo simulation to
model investment uncertainty and provides percentile-based projections.

### Core Mathematical Components

**`src/pension-calculations.ts`** - Contains the deterministic pension
projection logic:

- `calculatePensionProjection()` - Core function that projects pension pot value
  over time
- Handles two phases: Accumulation (before retirement) and Drawdown (after
  retirement)
- Accumulation: `pot = pot * (1 + growthRate) + annualContribution`
- Drawdown: `pot = pot * (1 + growthRate) - annualDrawdown + statePension`
- State pension income added at eligibility age (calculated from birth year)
- Stops simulation when pot depletes during drawdown phase

**`src/monte-carlo.ts`** - Monte Carlo simulation for modeling market
uncertainty:

- `runMonteCarloSimulation()` - Runs 1000+ simulations with random returns
- `pickFromNormalDistribution()` - Creates normally distributed returns based on
  expected return ± volatility
- `calculatePercentiles()` - Converts simulation results into 5th, 25th, 50th,
  75th, 95th percentiles

### UI Architecture

**Main Component**: `src/pension-calculator.tsx` - Root component with
two-column layout (controls + chart)

**Tab-Based Input System**:

- `CurrentSituationTab.tsx` - Age, current pot, contributions
- `MarketAssumptionsTab.tsx` - Expected returns and volatility
- `YourDecisionsTab.tsx` - Retirement age, drawdown amount, and state pension

**Visualization**: `src/components/PensionChart.tsx`

- Uses Recharts LineChart to display percentile projections
- **Important**: Implements data clamping to prevent 95th percentile from making
  other lines flat
- Charts scale to 110% of 75th percentile max, cropping extreme optimistic
  scenarios
- Custom tooltip shows unclamped values with "(cropped)" indicator

**Reusable Components**:

- `InputSlider.tsx` - Standardized slider with value display
- `CollapsibleSection.tsx` - Expandable content areas to save space
- `InfoButton.tsx` - Tooltip component for contextual help
- `ProjectedOutcomes.tsx` - Compact summary of key metrics

### Data Flow

1. User inputs → `PensionParams` interface in `types.ts`
2. Deterministic calculation via `calculatePensionProjection()`
3. Monte Carlo simulation via `runMonteCarloSimulation()`
4. Percentile calculation via `calculatePercentiles()`
5. Chart visualization with data clamping in `PensionChart`

### Key Implementation Details

**State Pension Integration**: `src/state-pension.ts` implements UK state
pension rules:

- Birth year-based eligibility age calculation (66-68 depending on birth year)
- `getStatePensionEligibilityAge()` - Calculates state pension age from current
  age
- `isEligibleForStatePension()` - Determines if state pension should be added
- Current full state pension: £11,973/year (2025-26 rate)
- State pension income added during retirement phase only

**Chart Scaling Solution**: The 95th percentile in optimistic scenarios can
reach extreme values, making median/quartile lines appear flat. PensionChart.tsx
solves this by:

- Setting yAxisMax to 110% of the 75th percentile maximum
- Clamping p95 data points above this threshold to `undefined`
- Preserving original unclamped values in custom tooltip

**Test Coverage**: `src/pension-calculator.test.ts` contains 36 tests across 7
categories:

- Basic functionality, edge cases, mathematical precision
- Realistic scenarios, stress tests, data integrity
- State pension integration tests
- Critical test: pot depletion only stops simulation in drawdown phase, not
  accumulation

**Retirement Logic**: Drawdown begins immediately at retirement age, not the
year after. This is the expected behavior tested throughout the test suite.

## TypeScript Interfaces

Key types in `src/types.ts`:

- `PensionParams` - All user input parameters (includes `statePensionAmount`)
- `PercentileDataPoint` - Chart data with p5, p25, p50, p75, p95 values
- `SurvivalRate` - Actuarial data for life expectancy calculations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tamlyn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
