## typed-smol

> Canonical router-first policy for loading Effect skills


# Effect Skill Loading Policy

Use this policy whenever the active task is Effect-related.

## Effect-Related Detection

Treat work as Effect-related when any of the following is true:

- The task mentions Effect, effect-ts, modules, facets, layers, fibers, schemas, or runtimes in the Effect ecosystem.
- Code changes include imports from `effect`, `effect/*`, or `@effect/*`.
- Files under `packages/effect/` or `.cursor/skills/effect-*` are read or modified.

## Required Loading Order

1. Load `.cursor/skills/effect-skill-router/SKILL.md` first.
2. Use router references to identify the owning skill for the target module/facet.
3. Load the owning `effect-module-*` or `effect-facet-*` skill before making technical assertions or code edits.
4. For cross-module work, keep one primary owner and explicitly list secondary owners.

## Stage Expectations

- `brainstorming` / `planning`: route ownership before proposing implementation paths.
- `research` / `requirements` / `specification`: ground behavior claims in owning skill references.
- `execution`: resolve ownership before writing or editing Effect code.
- `review`: validate API use and ownership boundaries against owning skills.
- `finalization`: record whether router-first loading was followed and any deviations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TylorS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
