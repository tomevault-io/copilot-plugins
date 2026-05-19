## domain

> Domain architecture patterns for the Workout Analytics library


# Domain Architecture

## Library Structure

```
src/
├── models/              # Core data structures
│   ├── sample.ts       # WorkoutSample interface
│   ├── phase.ts        # Phase, PhaseMetrics
│   ├── rep.ts          # Rep, RepMetrics
│   ├── set.ts          # Set, SetMetrics, VelocityMetrics, FatigueAnalysis, EffortEstimate
│   ├── session.ts      # ExerciseSession
│   └── index.ts
├── aggregators/        # Metric computation (pure functions)
│   ├── phase-aggregator.ts
│   ├── rep-aggregator.ts
│   ├── set-aggregator.ts  # Computes RIR/RPE from velocity
│   └── index.ts
├── detectors/          # Event detection (state machines)
│   ├── rep-detector.ts # Detects rep boundaries from WorkoutSamples
│   └── index.ts
├── analytics/          # High-level analytics and estimates
│   ├── strength.ts    # 1RM estimation
│   ├── fatigue.ts     # Fatigue estimation
│   ├── readiness.ts   # Readiness estimation
│   ├── session-metrics.ts # Combined session metrics
│   └── index.ts
├── vbt/               # Velocity-Based Training
│   ├── constants.ts   # VBT constants, velocity-%1RM mappings
│   ├── profile.ts     # Load-velocity profile builder
│   └── index.ts
└── index.ts           # Public API
```

## Hardware-Agnostic Design

The library is hardware-agnostic and uses `WorkoutSample` as the primary input format.

### WorkoutSample Format

```typescript
interface WorkoutSample {
  sequence: number;    // Incrementing sequence number (for drop detection)
  timestamp: number;    // Unix timestamp in ms
  phase: MovementPhase; // Movement phase
  position: number;     // Position in ROM (0-1 normalized)
  velocity: number;     // Instantaneous velocity (m/s, always positive)
  force: number;       // Force reading (lbs, absolute value)
}
```

**Key principles:**
- All values are normalized/standardized
- No device-specific data structures
- Adapters convert device-specific data to WorkoutSample format (outside this library)

## Model Patterns

### Pattern 1: Interface + Functions (Value Objects)

Use for **immutable data** and **computed results**.

```typescript
// models/sample.ts
export interface WorkoutSample {
  sequence: number;
  timestamp: number;
  phase: MovementPhase;
  position: number;
  velocity: number;
  force: number;
}

// Factory function
export function createSample(
  sequence: number,
  timestamp: number,
  phase: MovementPhase,
  position: number,
  velocity: number,
  force: number
): WorkoutSample {
  return { sequence, timestamp, phase, position, velocity, force };
}

// Pure functions operating on the data
export function isValidSample(sample: WorkoutSample): boolean {
  return sample.velocity >= 0 && sample.position >= 0 && sample.position <= 1;
}
```

**When to use Interface + Functions:**
- Value objects (WorkoutSample, Rep, Set)
- Computed/derived results (SetMetrics, SessionMetrics)
- Data structures that may be serialized

## Aggregator Patterns

Aggregators are pure functions that compute metrics from input data.

### Tiered Computation Pattern

SetMetrics uses nested sub-models with clear data flow:

```
Rep[] → VelocityMetrics → FatigueAnalysis → EffortEstimate
        (measurements)    (patterns)        (RIR/RPE)
```

```typescript
// aggregators/set-aggregator.ts
export function aggregateSet(
  reps: Rep[],
  targetTempo: TempoTarget | null,
  config: SetAggregatorConfig = DEFAULT_CONFIG
): SetMetrics {
  // Tier 1: Compute velocity metrics (raw measurements)
  const velocity = computeVelocityMetrics(reps, targetTempo, config);

  // Tier 2: Compute fatigue analysis (pattern detection from velocity)
  const fatigue = computeFatigueAnalysis(velocity, config);

  // Tier 3: Compute effort estimate (RIR/RPE from fatigue)
  const effort = computeEffortEstimate(fatigue);

  return {
    repCount: reps.length,
    velocity,
    fatigue,
    effort,
  };
}
```

**Aggregator Rules:**
- Pure functions (no side effects)
- Deterministic (same input = same output)
- No external dependencies
- Easy to test

## Detector Patterns

Detectors are state machines that detect events from sample streams.

```typescript
// detectors/rep-detector.ts
export class RepDetector {
  private state: RepDetectorState = 'idle';

  processSample(sample: WorkoutSample): RepBoundary | null {
    // State machine logic
    // Returns RepBoundary when rep is detected, null otherwise
  }

  reset(): void {
    this.state = 'idle';
  }
}
```

**Detector Rules:**
- State machines for event detection
- Process samples sequentially
- Return detected events (or null)
- Support reset for new sets

## Analytics Patterns

Analytics functions compute high-level estimates and metrics.

### Estimation Functions

Estimations have uncertainty (confidence levels).

```typescript
// analytics/strength.ts
export interface StrengthEstimate {
  estimated1RM: number;
  confidence: number;  // 0-1
  source: 'discovery' | 'historical' | 'session';
}

export function estimate1RM(
  weight: number,
  reps: number,
  velocity?: number
): StrengthEstimate {
  // Computation with confidence calculation
  return {
    estimated1RM: Math.round(weight * (1 + reps / 30)),
    confidence: calculateConfidence(reps, velocity),
    source: 'session',
  };
}
```

**Analytics Rules:**
- Estimates include confidence levels
- Calculations are deterministic (no uncertainty)
- Functions are pure (no side effects)
- Clear separation between estimates and calculations

## VBT Patterns

Velocity-Based Training (VBT) functions build load-velocity profiles and estimate 1RM.

```typescript
// vbt/profile.ts
export interface LoadVelocityProfile {
  exerciseId: string;
  dataPoints: LoadVelocityDataPoint[];
  slope: number;
  intercept: number;
  rSquared: number;
  estimated1RM: number;
  confidence: 'high' | 'medium' | 'low';
  mvt: number;  // Minimum velocity threshold
  createdAt: number;
}

export function buildLoadVelocityProfile(
  exerciseId: string,
  dataPoints: LoadVelocityDataPoint[]
): LoadVelocityProfile {
  // Linear regression to model load-velocity relationship
  // Estimate 1RM from profile
  // Calculate confidence
}
```

**VBT Rules:**
- Use linear regression for load-velocity relationship
- Estimate 1RM from velocity at minimum velocity threshold (MVT)
- Include confidence levels based on data quality
- Support profile updates with new data points

## Testing Patterns

```typescript
import { describe, it, expect } from 'vitest';

describe('aggregateSet', () => {
  it('should compute RIR/RPE from rep velocities', () => {
    const reps = createTestReps();
    const metrics = aggregateSet(reps);

    expect(metrics.effort.rir).toBeGreaterThanOrEqual(0);
    expect(metrics.effort.rpe).toBeGreaterThanOrEqual(4);
    expect(metrics.effort.rpe).toBeLessThanOrEqual(10);
  });
});
```

**Testing Rules:**
- Test pure functions with Vitest
- Use test data generators for realistic inputs
- Test edge cases (empty arrays, invalid data)
- Mock external dependencies (none in this library)

## Key Principles

1. **Hardware-agnostic** - Only knows about WorkoutSample, not device-specific formats
2. **Pure functions** - No side effects, deterministic
3. **Tiered computation** - Clear separation: measurements → patterns → predictions
4. **Confidence levels** - Estimates include uncertainty
5. **Easy to test** - Pure functions are straightforward to test
6. **No external dependencies** - Pure TypeScript, no runtime deps

## Adding New Features

1. **New metric:** Add to appropriate tier (velocity/fatigue/effort) in `set.ts`, compute in `set-aggregator.ts`
2. **New analytics function:** Add to `analytics/` directory, follow estimation pattern
3. **New detector:** Add to `detectors/` directory, implement state machine pattern
4. **New model:** Add to `models/` directory, use Interface + Functions pattern

---
> Source: [HJewkes/workout-analytics](https://github.com/HJewkes/workout-analytics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
