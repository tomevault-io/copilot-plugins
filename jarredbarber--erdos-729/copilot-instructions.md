## erdos-729

> This project aims to prove an Erdős conjecture about factorial divisibility in both natural language and Lean 4 with Mathlib. See PROBLEM.md for the full statement and success criteria.

# CLAUDE.md

## Overview

This project aims to prove an Erdős conjecture about factorial divisibility in both natural language and Lean 4 with Mathlib. See PROBLEM.md for the full statement and success criteria.

## Commands

```bash
lake build          # Build the Lean project (THIS IS THE VALIDATOR)
```

If `lake build` succeeds with no errors, the proof compiles.

## Project Structure

```
Erdos/
  Basic.lean        # Main proof file(s) go here
Erdos.lean          # Root import file
lakefile.toml       # Lake project config (mathlib dependency already configured)
lean-toolchain      # Lean version pinning
PROBLEM.md          # Problem statement and success criteria
```

## Lean 4 Guidelines

- This is **Lean 4** (not Lean 3). Do not use Lean 3 syntax.
- Mathlib is available. Import what you need with `import Mathlib.X.Y.Z`
- Use `lake build` to verify compilation. This is the ground truth.
- **No `sorry`** allowed in any proof. If you can't prove something, escalate.
- Use tactic mode (`by`) for proofs.
- Key Mathlib references: https://leanprover-community.github.io/mathlib4_docs/

## Key Mathlib Modules (likely useful)

- `Mathlib.Data.Nat.Factorial.Basic` — factorial definitions
- `Mathlib.Data.Nat.Prime.Basic` — prime number basics
- `Mathlib.Data.Nat.GCDs` — gcd, divisibility
- `Mathlib.NumberTheory.Padics.PadicVal` — p-adic valuations (for Legendre's formula)
- `Mathlib.Order.Filter.Basic` — asymptotic notation
- `Mathlib.Analysis.Asymptotics.Asymptotics` — Big-O notation

## Success Criteria (from PROBLEM.md)

1. The main theorem *statement* in Lean corresponds to the natural language problem statement
2. `lake build` succeeds
3. No `sorry` in the codebase (upstream mathlib sorrys are acceptable)

## Validation

```bash
# Check for sorry
grep -rn "sorry" Erdos/ --include="*.lean"

# Build
lake build
```

Both must pass.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jarredbarber) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
