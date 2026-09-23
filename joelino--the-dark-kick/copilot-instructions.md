## the-dark-kick

> Guidance for coding agents working on **The Dark Kick**.

# AGENTS.md

Guidance for coding agents working on **The Dark Kick**.

## Product goal

Build a small, touch-first tactical web game where simple actions interact deeply with enemies, terrain, and environmental objects. The immediate goal is to test whether environmental combat feels satisfying on an iPhone.

Prioritize a playable vertical slice over breadth.

## Current stack

- Phaser 3
- TypeScript with strict checking
- Vite
- Vitest
- npm
- GitHub Actions
- GitHub Pages

Do not introduce React, Vue, a backend, native iOS code, a database, or external services without an explicit decision.

## Dependency safety

- Do not add a package when the existing stack or platform APIs can reasonably do the job.
- Direct dependencies are exact-pinned and allowlisted by `scripts/check-dependency-policy.mjs`.
- Before approving a new direct package, verify that it is widely adopted, actively maintained, and owned/published through an expected account.
- The project-level `.npmrc` rejects releases younger than 30 days and non-registry git/file/tarball dependency sources.
- Keep install scripts fail-closed. Review and pin any necessary script approval by exact package version in `allowScripts`.
- Use `npm ci` for reproducible installs from the committed lockfile. Do not bypass these controls with command-line overrides.

## Development principles

- Keep game rules independent from Phaser rendering whenever practical.
- Represent terrain, objects, and occupants separately.
- Prefer pure, deterministic rule functions with focused unit tests.
- Use data-driven definitions for actions, terrain, and objects, but avoid premature framework-building.
- Preserve touch usability, safe-area support, and a no-scroll phone layout.
- Use taps rather than drag gestures unless explicitly requested.
- Preview legal targets and deterministic consequences clearly.
- Prefer the locally included 32rogues sprites for the prototype's board, characters, terrain, and effects; keep the integration lightweight.

## Core action identities

- **Basic Strike:** reliable adjacent damage.
- **Power Strike:** heavy damage and destruction.
- **Kick:** low damage plus forced movement and collision interactions.
- **Fire Arrow:** ignition and battlefield transformation.

Do not blur these into interchangeable damage buttons.

## Shared world rules

Environmental interactions should emerge from common properties such as material, durability, weight, contents, and state.

Important current decisions:

- Fire spreads through wooden terrain and wooden objects.
- Kick on a brazier creates a narrow, longer fire trail.
- Power Strike on a brazier creates a wider, shorter fire area behind it.
- Force interactions may use walls, spikes, fire, ledges, units, and movable objects.
- Prefer deterministic outcomes and previews over random percentages.

## Scope discipline

Before adding a system, state the gameplay question it is meant to answer.

Near-term order:

1. Polish existing movement, Strike, Kick, wall, spike, exit, and phone UX.
2. Add Power Strike and a small set of destructible objects.
3. Add Fire Arrow and deterministic fire spread.
4. Add minimal enemy AI with visible intent.
5. Create one compact turn-limited exit encounter.

Defer:

- Deck building and card progression.
- Multiplayer and networking.
- Real-time action timers.
- Procedural generation.
- Inventory and loot systems.
- Meta-progression.
- Complex animation pipelines.
- Large abstraction layers, ECS architecture, or plugin systems.

## Quality bar

Before finishing a task, run:

```bash
npm ci
npm run typecheck
npm run test
npm run build
```

Use `npm ci` in CI when a lockfile is present.

Add or update tests for every rule change. Never claim a command passed unless it completed successfully. Environment-only failures, such as a blocked browser download, should be reported separately from project failures.

Keep README documentation accurate after each iteration.

## Post-implementation QA pass

After implementation and automated checks, perform a short adversarial QA pass
against the running game:

1. Turn each acceptance criterion into an observable behavior and verify it.
2. Exercise at least one phone-landscape viewport. For UI or animation changes,
   use an available browser-control tool to interact with the built result and
   inspect still frames during transitions.
3. Cover the happy path plus reasonable edge cases the spec may not spell out:
   skipping optional steps, repeating/illegal input, entering hazards normally,
   interrupted intents, defeat/victory, reset, and responsive layout.
4. Look for contradictions between numeric requirements and promised outcomes.
   Resolve only when there is a clear product-consistent interpretation;
   otherwise leave the behavior unchanged and flag the decision.
5. Create or update `QA_NOTES.md` with the behaviors checked, evidence available,
   assumptions made, environment limitations, and anything that still needs a
   human playtest.

If the environment has no controllable browser, say exactly that in
`QA_NOTES.md`, complete the strongest rule/build checks available, and do not
claim visual or interaction verification. Browser QA complements the physical
iPhone playtest; it does not replace the judgment of whether the game is fun.

---
> Source: [Joelino/the-dark-kick](https://github.com/Joelino/the-dark-kick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
