## clair

> These instructions apply to the entire repository.

# Repository Instructions

## Scope

These instructions apply to the entire repository.

## Failure Model

Follow `docs/failure-model.md`.

Do not silently convert internal contract violations into successful control
flow. Report external system failures through the documented error path for the
API being implemented.

## Style

Follow the existing source style. Subprogram names are lowercase.

Do not add `use Package` clauses. `use type` clauses are allowed when they make
operators explicit without importing a whole package namespace.

Do not add `use Interfaces.C`.

## C ABI Boundary

`Clair.C` is the C ABI adapter boundary. Reserve `clair-c.ads` and
`clair-c.adb` for that Ada package, and do not create `clair-c.c`.

---
> Source: [hodong-kim/clair](https://github.com/hodong-kim/clair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
