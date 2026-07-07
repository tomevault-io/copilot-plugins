## firefighterai

> - This file governs where code goes, how modules are split, and how changes must preserve structure over time.

## Purpose
- This file governs where code goes, how modules are split, and how changes must preserve structure over time.
- Apply these rules to all new files, refactors, and AI-generated changes.
- Treat the current root-level drift under `src/` as transitional. Move touched code toward the target structure when practical.

## Repo Skills
- Use `skills/game-design-maintainer/SKILL.md` for requests that scope or ship features, update `docs/GAME_DESIGN_REFERENCE.md`, mark obsolete work in `docs/deprecations.md`, maintain `work_queue.md`, answer "what should I work on next?", or turn an accepted plan into a queued task for later implementation.

## Core Principles
- Feature/domain-first organisation.
- High cohesion, low coupling.
- Small, focused modules.
- Readability for both humans and AI (optimised for context windows).
- Prefer local domain ownership over premature shared abstractions.

## Folder Structure
```text
src/
  core/
  systems/
  entities/
  ui/
  shared/
  config/
  generated/
```
- `core/`: app/runtime primitives and engine-level foundations. Keep it domain-agnostic.
- `systems/`: major gameplay or simulation domains such as fire, climate, terrain/mapgen, persistence, progression, and audio.
- `entities/`: world nouns with their own data and behaviour boundaries.
- `ui/`: presentation, input wiring, view state, and player-facing interaction code.
- `shared/`: truly cross-domain primitives with no domain ownership.
- `config/`: static configuration, tunables, and declarative setup.
- `generated/`: generated outputs and machine-produced artifacts that are not hand-authored.
- Existing top-level areas such as `render/`, `sim/`, `mapgen/`, `app/`, `audio/`, `persistence/`, and `progression/` are transitional. New work should move toward the target model.

```text
systems/<domain>/
  sim/
  rendering/
  controllers/
  types/
  utils/
  constants/
```
- `sim/`: pure simulation logic, rules, transforms, and state transitions.
- `rendering/`: visual adapters, scene construction, and rendering-only helpers.
- `controllers/`: orchestration, coordination, and runtime wiring.
- `types/`: domain interfaces, value objects, and public boundary types.
- `utils/`: small helpers owned by that domain only. Do not create repo-wide dumping grounds.
- `constants/`: domain-specific constants, tables, and static defaults.

## File Placement Rules
- If code belongs to a single domain, place it inside that domain.
- If code is rendering or visual presentation, place it under `rendering/` or `ui/`.
- If code is pure simulation logic, place it under `sim/`.
- If code orchestrates flows, runtime setup, or coordination between modules, place it under `controllers/`.
- If code defines types, interfaces, or schemas, place it under `types/`.
- If code is reused across multiple unrelated domains, place it under `shared/`.
- If code is generated, place it under `generated/` unless tooling requires co-location.
- Do not place new general-purpose modules directly under `src/`.
- Exception: true app entrypoints such as `main.ts` may remain at `src/` root. Do not add new non-entrypoint files there.

## Module Design Rules
- Keep files small and focused. Target roughly 200-300 lines unless there is a clear reason not to.
- Give each module one responsibility.
- Separate state, logic, and rendering into distinct modules.
- Extract helpers when a file stops scanning cleanly.
- Prefer composition over monolithic manager classes.
- Split large flows into collaborating modules instead of growing one controller or runtime file.

## Dependency Rules
- A domain may depend on its own subtree.
- A domain may depend on `core/`.
- A domain may depend on `shared/`.
- Domains must not directly depend on other domains unless the dependency goes through explicit interfaces or boundary types.
- `ui/` may depend on domain data and controllers. Domains must not depend on `ui/`.
- Rendering may consume prepared domain data. Rendering must not contain business logic or simulation rules.
- `shared/` and `core/` must stay domain-agnostic. They must not reach back into specific domains.

## Naming Conventions
- Use descriptive, domain-specific names.
- Name files for their actual responsibility, such as `fireSpreadSolver.ts`, `riverMeshBuilder.ts`, or `climateSeasonRules.ts`.
- Match names to the bounded context they live in.
- Do not create generic filenames such as `utils.ts`, `helpers.ts`, `common.ts`, or `misc.ts`.
- Do not hide important behaviour behind vague names like `manager`, `service`, or `processor` unless the role is truly specific and justified.

## Refactor Triggers
- Refactor when a file mixes multiple concerns.
- Refactor when a file becomes hard to scan or reason about quickly.
- Refactor when duplicate logic appears in the same domain or across domains.
- Refactor when a file grows beyond the normal size without a clear reason.
- Refactor when imports show cross-domain leakage or boundary violations.

## Anti-Patterns
- Dumping logic into the `src` root.
- Creating generic helper dumping grounds.
- Mixing rendering, simulation, and state mutation in one module.
- Adding cross-domain imports without explicit boundaries.
- Allowing oversized files to act as god objects.

## Required Pre-Change Reasoning
- Before making code changes, always state which files will be changed or created.
- Before making code changes, always state why each file belongs in that folder.
- Before making code changes, always state whether a new file is required or an existing file should be extended.
- Before making code changes, always state any impact on dependency direction.
- If the chosen location weakens structure, stop and pick a better boundary before editing.

## Output Expectations for AI Agents
- Preserve or improve structure with every change.
- Do not introduce new top-level folders without strong justification.
- Prefer refactoring over adding complexity.
- Keep modules easy to understand in isolation.
- Optimise for long-term maintainability, not short-term convenience.
- Leave the codebase more modular than you found it when the task allows.

---
> Source: [robopony2k/FIreFighterAI](https://github.com/robopony2k/FIreFighterAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
