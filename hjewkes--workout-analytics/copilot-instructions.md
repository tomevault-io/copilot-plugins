## typescript

> TypeScript conventions for the Workout Analytics library


# TypeScript Conventions

## Avoid `any`

- Use proper types or `unknown` with type guards
- Define types for all function parameters and returns

## File Naming Rules

- **NO logic in `index.ts` files** - barrel exports only
- Name files explicitly: `session-metrics.ts` not `session/index.ts`
- Use kebab-case for file names: `set-aggregator.ts`

```typescript
// index.ts - GOOD: barrel exports only
export { aggregatePhase, aggregateRep, aggregateSet } from './aggregators';
export type { SetMetrics, VelocityMetrics } from './models';

// index.ts - BAD: contains logic
export function computeMetrics() { ... }  // NEVER do this
```

## Export Patterns

### Public API Exports

Only export what library consumers need. Internal implementation details stay private.

```typescript
// GOOD: Focused public API
export {
  // Models
  type WorkoutSample,
  type Rep,
  type Set,
  type SetMetrics,
  // Analytics functions
  estimate1RM,
  computeFatigue,
  buildVelocityProfile,
  // Aggregators
  aggregateSet,
  aggregateRep,
} from './src';

// BAD: Exporting everything
export * from './src';  // Exposes internal implementation
```

### Internal Modules

Use named exports for internal utilities. Don't re-export from public index.

```typescript
// v0/analytics/internal-utils.ts - internal
export function calculateMean(values: number[]): number { ... }

// src/index.ts - DON'T export calculateMean, it's internal
```

## Import Patterns

### Absolute Imports with @ Alias

Use absolute imports with the `@/` path alias (mapped to `src/`) for all imports across modules.

```typescript
// GOOD: Absolute imports with @ alias
import { aggregatePhase } from '@/aggregators/phase-aggregator';
import type { WorkoutSample } from '@/models/sample';
import { estimate1RM, computeFatigue } from '@/analytics';

// GOOD: Import from barrel when available
import { aggregateSet, aggregateRep } from '@/aggregators';
import type { WorkoutSample, Rep, Set } from '@/models';
```

### When to Use Relative Imports

Only use relative imports (`./`) for imports within the same directory (e.g., barrel re-exports).

```typescript
// GOOD: Relative for same-directory barrel exports
// In src/analytics/index.ts
export { getRepMeanVelocity } from './rep-analytics';
export { getSetVelocityLossPct } from './set-analytics';

// BAD: Relative imports across directories
import { Rep } from '../models/rep';  // Use @/models/rep instead
```

## Interfaces vs Types

- Use `interface` for object shapes that may be extended
- Use `type` for unions, intersections, and primitives

```typescript
// Interface for extensible shapes
export interface WorkoutSample {
  sequence: number;
  timestamp: number;
  phase: MovementPhase;
  position: number;
  velocity: number;
  force: number;
}

// Type for unions
export type MovementPhase = 'concentric' | 'eccentric' | 'hold-top' | 'hold-bottom' | 'rest';
```

## Function Return Types

Prefer explicit return types for public API functions.

```typescript
// GOOD: Explicit return type
export function estimate1RM(weight: number, reps: number): number {
  return weight * (1 + reps / 30);
}

// ACCEPTABLE: Type inference for internal functions
function calculateMean(values: number[]) {
  return values.reduce((a, b) => a + b, 0) / values.length;
}
```

## Pure Functions

All analytics functions should be pure (no side effects, deterministic).

```typescript
// GOOD: Pure function
export function computeFatigue(sets: Set[]): FatigueEstimate {
  // No side effects, only computation
  return { level: 0.5, isJunkVolume: false };
}

// BAD: Side effects
export function computeFatigue(sets: Set[]): FatigueEstimate {
  console.log('Computing fatigue'); // Side effect
  return { level: 0.5, isJunkVolume: false };
}
```

---
> Source: [HJewkes/workout-analytics](https://github.com/HJewkes/workout-analytics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
