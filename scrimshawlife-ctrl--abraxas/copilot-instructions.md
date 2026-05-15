## abraxas

> 1. Canonical runtime and validator evidence

# ABRAXAS / AAL-Core Agent Contract

## Authority Order
1. Canonical runtime and validator evidence
2. Repository-enforced governance policy
3. Subsystem metadata constraints
4. Operator prompts, templates, and summaries

## Coding Eligibility
Code changes are eligible only when subsystem metadata indicates `code_authorized: true` and no stop condition is active.

## Mandatory Non-Negotiables
- Additive patch discipline only.
- Deterministic behavior; no hidden randomness in control logic.
- No prose-only claims of readiness, closure, promotion, or attestation.
- SHADOW and FORECAST lanes must remain separated.
- Derivative surfaces never confer authority.
- Missing evidence must remain explicit (`partial`, `blocked`, `attestation_pending`, `NOT_COMPUTABLE`).

## Default Assumptions
- If receipts are missing, completion defaults to non-closure.
- Governance memory is append-only and non-authoritative relative to runtime truth.
- Promotion is reversible and silence is not promotion.

## Resolve-by-Inspection-Before-Asking
Inspect subsystem YAML, registry, governance policy, and ledger entries before requesting clarification.

## Required Code Drop Envelope
Every substantive drop must include:
- objective
- scope and lane
- subsystem id
- change class
- proof targets
- tests and validation commands
- declared risks and stop conditions

## Standard Patch Order
1. Contract + policy updates
2. Subsystem metadata + registry alignment
3. Scripts and checks
4. Tests
5. Operator ergonomics (Makefile/CI/templates)

## Proof Rule
No readiness or closure claim is valid without required runtime/validator receipts.

## Stop Conditions
Stop if any of the following occur:
- Required receipts unavailable for a promoted/gated claim
- Subsystem registry mismatch
- Invalid governance record schema
- Lane violation (e.g., shadow surface influencing forecast-active authority path)

## Lane Discipline
- `shadow`: advisory only, non-authoritative.
- `forecast-active`: execution-affecting paths requiring strict proof and validator visibility.

## Registry Discipline
Every non-trivial subsystem must exist in `.abraxas/registries/expected_subsystems.yaml` and have a matching metadata file.

## Testing Discipline
Run deterministic checks tied to changed surfaces before completion (`pytest -q`, governance scripts, lint/check targets).

## Projection/UI Downstream Law
UI, summaries, and projections can report status but cannot mint authority, closure, or promotion.

## Preferred Pattern Sources
1. Existing repo governance scripts and deterministic tooling
2. `.abraxas/implementation_policy.md`
3. Explicit schemas and registry contracts

## Completion Labels
Use only: `blocked`, `partial`, `attestation_pending`, `candidate`, `promoted`, `revoked`.

## Fast-Path Goal
Accelerate delivery by enforcing proof-addressable structure early and failing closed when evidence is missing.

## Operator Output Style
Return: changed files, validation commands/results, unresolved risks/placeholders, and next manual steps.

---
> Source: [scrimshawlife-ctrl/Abraxas](https://github.com/scrimshawlife-ctrl/Abraxas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
