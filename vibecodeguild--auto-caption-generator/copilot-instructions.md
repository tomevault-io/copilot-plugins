## auto-caption-generator

> This is a daily-use production application. Work in this repository must follow

# Engineering Collaboration Contract

This is a daily-use production application. Work in this repository must follow
the accepted product plan and preserve existing working behavior.

## Product decisions

- Treat user-approved designs and plans as implementation contracts.
- Do not replace an accepted design with an easier approximation.
- Do not promote an experiment, fallback, heuristic, or speculative idea into
  product behavior without explicit approval.
- Label proposed work as proposed. Do not describe it as approved until the
  user approves it.
- When a decision changes, record the old decision as superseded instead of
  silently rewriting history.

## Scope control

Before editing, state the authorized changes, excluded changes, and acceptance
criteria. Implement only the authorized scope.

If correct implementation requires a materially different design or additional
scope, stop and discuss it before editing. Do not work around the issue with a
parallel mechanism or UI.

## Root-cause engineering

- Diagnose failures from concrete evidence before implementing a fix.
- Fix the responsible layer rather than masking the symptom.
- Keep diagnostics in existing product surfaces unless a new surface is
  explicitly approved.
- Preserve independent safety checks even when removing an unrelated feature.

## Runtime ownership

The user starts and stops development servers from VS Code. Do not start, stop,
restart, or replace application server processes unless the user explicitly
asks for that operation.

## Handoff requirements

- Review the final diff for unauthorized behavior and documentation drift.
- Run relevant automated tests, TypeScript typecheck, and production build when
  the change affects those layers.
- Report what changed, what was deliberately not changed, and verification
  results accurately.

---
> Source: [vibecodeguild/Auto-Caption-Generator](https://github.com/vibecodeguild/Auto-Caption-Generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
