## architecture-essentials

> Essential architectural patterns and critical gotchas for RBIP duty list system


# Architecture Essentials

> **Purpose**: Critical architectural patterns and gotchas that new agents MUST know. For detailed changelog, see `CHANGELOG.md`.

## Critical Gotchas (Top 10)

1. **Database Type Safety (CRITICAL)**: Always use `lib/db/types.ts` utilities (`toDbLeaveType`, `programNamesToIds`, `normalizeFTE`) - TypeScript types are WIDER than database enums
2. **staffOverrides is Single Source of Truth**: All staff modifications must update `staffOverrides`; algorithms read from it
3. **`average_pca_per_team` / display Avg Can Change During Step 2**: The requirement-side target may change when Step 2 changes therapist distribution (Step 2.2/2.3) or special-program capacity (Step 2.0). Once Step 3/4 have been completed, any Step 2 change that alters this target must mark Steps 3/4 as **out of date** and require a rerun. For **what users see as “Avg PCA/team”** on the dashboard and Step 3.1, use the **unified projection** (`Step3ProjectionV2.displayTargetByTeam` / `getStep3AveragePcaDisplayTargets`) — do not mix ad-hoc `calculations` vs `step2Result` reads for that same label (see **Step 3 projection** below).
4. **Bed Relieving Calculations**: Use `totalBedsEffectiveAllTeams` (after SHS/students deductions) consistently - using raw `totalBedsAllTeams` creates impossible positive global sums
5. **Pending FTE Update Safety**: Always use `assignOneSlotAndUpdatePending()` for one-slot calls and `assignUpToPendingAndUpdatePending()` for global pending calls - never manually update `pendingFTE[team]` after calling wrappers
6. **TypeScript Strict Mode**: Use `createEmptyTeamRecord<T>()` for Record initialization, guard clauses instead of `!`, `PromiseLike<any>[]` for Supabase queries in `Promise.all()`
7. **Step 3 Wizard State**: Always clone state objects (`adjustedFTE`, `existingAllocations`) before modification; each mini-step progressively updates `currentPendingFTE` and `pcaAllocations`
8. **Adjacent Slot Logic (Step 3.3)**: Only considers slots actually assigned by special programs, NOT Step 3.2 assignments
9. **Bed Relieving Notes**: Stored in `staffOverrides.__bedRelieving` (within-day only, NOT copied across dates via copy schedule)
10. **Snapshot Envelope**: Always use `buildBaselineSnapshotEnvelope()` before saving; always validate with `validateAndRepairBaselineSnapshot()` on load

## State Management Architecture

### Three-Layer State Pattern
1. **Layer 1: Saved State** (from database) - `baseline_snapshot`, `staff_overrides`, `workflow_state`
2. **Layer 2: Algorithm State** (generated from snapshot + overrides) - allocations, calculations
3. **Layer 3: Override State** (user modifications) - `staffOverrides` (single source of truth)

### Key State Variables
- **`staffOverrides`**: Single source of truth for all staff modifications
  - `invalidSlots`: Array of `{ slot: number; timeRange: { start: string; end: string } }` (display-only)
  - `amPmSelection`: `'AM' | 'PM'` for therapist FTE = 0.5 or 0.25
  - `slotOverrides`: Manual slot transfers `{ slot1?, slot2?, slot3?, slot4? }`
  - `substitutionFor`: Non-floating PCA substitution override
  - `specialProgramOverrides`: Step 2.0 special program assignments
  - `__bedCounts`: Schedule-level bed count overrides per team
- **`pcaAllocations`**: `Record<Team, (PCAAllocation & { staff: Staff })[]>` - current PCA assignments
- **`calculations`**: `Record<Team, ScheduleCalculations | null>` - metrics per team
- **`baselineSnapshot`**: Frozen snapshot of dashboard state (per-date isolation)

## Allocation Workflow (5 Steps)

1. **Step 1**: Leave & FTE management → updates `staffOverrides`, calculates initial `average_pca_per_team` target
2. **Step 2**: Therapist & Non-Floating PCA
   - **2.0**: Special Program Overrides Dialog
   - **Algorithm**: Therapist → Non-floating PCA → Special program PCA (except DRM) → Non-floating substitution
   - **2.1**: Non-Floating PCA Substitution Dialog
3. **Step 3**: Floating PCA wizard (3.0 → 3.1 → 3.2 → 3.3 → 3.4)
   - **3.1**: Adjust pending FTE & team order
   - **3.2**: Preferred slot reservation
   - **3.3**: Adjacent slot assignment (from special program PCAs only)
   - **3.4**: Final floating PCA algorithm
4. **Step 4**: Bed relieving calculation
5. **Step 5**: Review and finalization

## Key Algorithm Rules

- **DRM Special Program**: DRM is NOT a special program requiring designated PCA staff. It only adds +0.4 FTE to DRO team's `average_pca_per_team`. Skip DRM during special program PCA allocation phase.
- **Floating PCA Substitution**: Step 2 handles non-floating substitution; Step 3 should NOT assign to slots already covered by Step 2 substitutions.
- **Rounding Consistency**: Use `roundToNearestQuarterWithMidpoint()` for pending FTE checks to prevent infinite loops.

## Step 3 projection: Avg vs floating chain (essential)

**Full glossary (floating vs non-floating):** `docs/glossary/step3-floating-nonfloating.md`

- **`Step3ProjectionV2` / one builder:** Built when Step 2 is settled for Step 3 entry (controller); pass **one** reference into schedule page, `PCABlock`, `FloatingPCAConfigDialogV2`, etc. Do not recompute `computeStep3BootstrapSummary` in parallel with different args for the **same** dashboard vs Step 3.1 display numbers.

- **Display Avg (source of truth for the label):** `displayTargetByTeam` — raw average PCA per team for **every** “Avg PCA/team” style display (dashboard + Step 3.1). It is **not** the surplus-adjusted operational team total; editing Step 3.1 **fixed floating** does **not** change this display Avg.

- **Non-floating FTE:** Reduces the floating gap; may use `nonFloatingFteBreakdownByTeam` / `computeStep3NonFloatingFteBreakdownByTeamFromAllocations` for engine truth while UI can still show friendly totals.

- **Raw floating:** Continuous `Avg − non-floating` **before** quarter rounding. Surplus and slack math stay in **raw** space first; then apply quarter rules to floating layers.

- **Rounded floating at Step 3 open:** `fixedRoundedFloatingTargetByTeam` = `roundToNearestQuarterWithMidpoint(pendingByTeam)` from the **surplus-aware** bootstrap — **not** `round(displayTarget)` alone and **not** `roundedAdjustedTeamTargets` as the Step 3.1 quarter seed. After 3.1, **pending floating** in dialog state drives the allocator.

- **Pending floating:** Set in Step 3.1 (order + ±); Steps 3.2–3.4 consume this until the user returns to 3.1.

- **Assigned floating / remaining pending:** Assigned = floating FTE actually placed in 3.2–3.4; remaining pending = pending floating − assigned (quarters, clamp at ≥ 0).

- **Extra coverage vs surplus:** Post-need **extra** assignments are a **separate** allocator/UI path; projection and surplus grants must not treat generic extra coverage as surplus authority.

## PCA Total Values (Critical Distinction)

- **`totalPCAOnDuty`**: STABLE - Sum of FTE from all on-duty PCAs (used for requirement calculations)
- **`totalPCAFromAllocations`**: UNSTABLE - Sum of FTE from current allocations (reference/debugging only, NOT used in calculations)

**Key Rule**: Always use `totalPCAOnDuty` for requirement calculations. Never use `totalPCAFromAllocations` for calculations.

## Snapshot System

- **Per-Date Isolation**: Each schedule date has isolated `baseline_snapshot` to prevent cross-date contamination
- **Versioned Envelope**: Always wrap snapshots in envelope before saving (`buildBaselineSnapshotEnvelope`)
- **Validation**: Always validate on load (`validateAndRepairBaselineSnapshot`) to handle corrupted data gracefully

---
> Source: [alvin19921008-arch/RBIP-web-app](https://github.com/alvin19921008-arch/RBIP-web-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
