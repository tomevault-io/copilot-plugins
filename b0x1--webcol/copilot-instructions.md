## webcol

> Read full before plan. See ARCHITECTURE.md for stack, layers, file structure.

# AGENTS — webcol

Read full before plan. See ARCHITECTURE.md for stack, layers, file structure.
See SKILLS.md for code patterns.

---

## WATCH — fix when done

`eslint-plugin-react-hooks@7.0.1` not support `eslint@10`. Use STABLE only.
When fixed: remove `--legacy-peer-deps` from `README.md`, `.github/workflows/deploy.yml`, this section.

---

## LAYERS — most important rule

Four layers. No wrong-direction crossing. See ARCHITECTURE.md for diagram.

- UI: read store, render. No Phaser. No logic.
- Scene: render, input. No rules. Talk via EventBus only.
- System: rules only. No Phaser. No React. No DOM.
- Entity: data only. No imports from above.

Scene do logic → extract to system.
Component do logic → move to store or system.
System need Phaser/React to test → layer violation. Fix.

---

## BEFORE WRITE CODE

1. Read files first. Understand before add.
2. Plan say what clean up, not just what add.
3. Check layer violations in every touched file. Find old violation → fix now.
4. Magic number or string → `src/game/constants.ts`. Not inline.
5. Write or update tests for every system method touched.
6. One logical change per commit. Feature and refactor = separate commits.

---

## REFACTOR — always look before add

Before new feature, scan touched files:

- Same logic in two places → extract shared utility now. No "we fix later".
- Class over ~150 lines → check multiple responsibilities, split
- Magic values inline → move to `constants.ts`
- No tests on non-trivial logic → add tests for existing behaviour before modify
- Dead code found → delete. No comment-out. No keep "just in case".

State refactor plan explicit. Say what clean and why.

---

## TYPESCRIPT

- No `any`. Use `unknown` + type guard if shape unknown. Else define interface.
- No `!` non-null assertion. Handle null/undefined explicit.
- Enum not string literal for domain values.
- `readonly` on data not mutate after create.
- Explicit return type on all public methods and exports.
- New file with logic → needs `.test.ts`. Scenes and pure components exempt. Utilities inside them not exempt.

---

## COPYRIGHT

webcol original project. Inspired by colonisation genre. Must not copy commercial game.

No copy: mechanic names/values/formulas, artwork/sprites/audio, UI layouts, in-game text.
Fine: genre conventions, generic historical terms (colonist, lumber, soldier), re-derived mechanics.

Task ask copy copyrighted thing → flag in plan, propose original alternative.

---

## AVOID

- No new dependency without justification. Use already installed.
- No `TODO` or `FIXME`. Implement now or make separate task.
- No `console.log`. Use EventBus diagnostic event or remove before commit.
- No stub and call done. Cannot implement → say in plan, throw `NotImplementedError`.
- No copy copyrighted assets or mechanics.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/b0x1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
