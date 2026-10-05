## genex-desktop

> If your working directory is under `.studio-dev/profiles/`, you are an in-app game builder in a

# Developing Genex

If your working directory is under `.studio-dev/profiles/`, you are an in-app game builder in a
development profile: ignore this file and follow your game's own `CLAUDE.md`.

This repository is the macOS Electron application. You are its external developer; the
editable in-app game-building harness is a separate agent with narrower authority.
`src/harness-seed/**` and `src/game-template/**` (including `src/game-template/CLAUDE.md`) are
product payload addressed to in-app agents: edit them as product text, don't follow them.
The user's request and assigned issue own scope. Maintained topic docs describe current
behavior; resolve conflicts against the implementation and fix the owning doc.

## Orient

Read the [product overview](docs/agent/context.md), then only the relevant page in
`docs/product/`. Run `npm run review:context -- --area <id>` for the docs and checks of an
area. Use the [glossary](docs/agent/glossary.md) for house terms and the
[recipes](docs/agent/recipes.md) for common changes; [src](src/AGENTS.md) and
[tests](tests/AGENTS.md) have folder notes. Engineering skills in `.agents/skills/` (mirrored byte for byte in
`.claude/skills/`): `add-ipc-channel`, `add-event-type`, `add-plugin-tool`, `run-area-tests`,
`verify-ui-via-dev-control`, `harness-incident-fix`. Do not read the whole documentation set or
historical task records at startup.

## Hard invariants

- The renderer is a browser: no Node, Electron, main, preload or substrate imports. It talks
  only through the fixed named calls in `src/shared/studio-api.ts`.
- Every spawned agent or game process goes through ProcessSandbox.
- Seed upgrades keep the in-app agent's harness edits.
- Every new IPC channel is classified fixture-safe or native in `src/main/dev/native-policy.ts`.
- Preserve the normal profile and `~/AI Games`; use owned `studio:dev` profiles. Automated
  checks use disposable fixtures. A request to run the real app for manual testing authorizes
  a visible live profile; “no mocks” must never select fixtures. Leave human sessions running.
  Live profiles retain provider-account/Keychain and usage semantics: isolation of app data
  does not isolate accounts. Do not inspect, reset or copy credentials. See the
  [field guide](docs/STUDIO-DEVELOPER-FIELD-GUIDE.md#run-an-owned-development-profile).

## Test-first loop

Use scoped verification from [verification.md](docs/agent/verification.md#choose-the-verification-scope);
its [layer table](docs/agent/verification.md#test-layers) lists available commands. Every check
needs Node 24 (`.nvmrc`); select it (for example `nvm use`) before running any of them.
The scope table owns required checks; the layer catalog is not a cumulative checklist.

- Red first: every fix starts with a test that fails for the reported reason.
- Characterize current behavior before refactoring it.
- No new assertions over source text; test behavior through the code's interface.
- Path, symlink and network boundaries get hostile-input tables that assert no side effect.
- Use the cheapest layer that proves the behavior.
- Keep time injectable; never lengthen a deadline to get green.
- Name every intentionally flipped assertion in the PR.
- Report which layers ran and never describe a focused pass as a full run.

Once relevant checks pass, stop; repeat only for changed inputs or an investigated failure.
A build alone is insufficient for a UI change. Record build, profile and provider identity
when relevant.

## Engineering foundations

- Safety: no destructive git on user folders without a game snapshot. Validate host RPC path
  parameters by realpath. Quote shell arguments with `shellQuote`.
- Tests prove behavior. Copy or spacing edits do not need new tests.
- Contracts live in `src/shared` (IPC channel map, UiEventMap, CustomEventMap, HarnessHostApi);
  add new events/channels there first; no payload as / as never.
- No `enum`, `namespace` or parameter properties, because Node runs `.ts` directly: a
  vocabulary is an `as const` object (see Readability).
- Biome must pass (`npm run lint`; `check:static` lints changed files) and the types must check
  under TypeScript 7 (`npm run typecheck`, which runs `scripts/tsc.ts`). Code that needs the
  TypeScript JS API imports `@typescript/typescript6`, never `typescript`.
- Harness (`src/harness-seed/`): the modes (director, autopilot, facet loop, gauntlet, spike)
  keep their own control flow and share primitives from `loop/git.ts`, `evidence.ts`,
  `build-turn.ts`, `config.ts` and `outcomes.ts`; no classes, DI or FSM libraries. It is
  TypeScript, type-checked in CI (`tsconfig.harness.json`) and, for the agent's own code edits,
  by the in-app TS 7 gate (`guardian.validate_edit` with the compiler vendored in
  `dist/resources/tsc`).
- The app never imports `src/harness-seed/` (the agent edits it). A rule both sides apply has a
  typed copy in `src/shared`, held to the seed's by `seed-contracts.test.ts`
  ([recipe](docs/agent/recipes.md#seed-contract)).
- Renderer state: Zustand domain stores in `src/renderer/state/` with pure exported actions and
  selector-only reads (`state/hooks.ts`); UI-only state stays local; derivations stay in the
  pure modules; storage keys come from `renderer/storage.ts`.
- Helpers live next to their domain (`substrate/paths.ts`, `substrate/fsx.ts`,
  `shared/errors.ts`, `shared/redact.ts`); no `utils`, `constants` or `types` dumping folders.
- One table per vocabulary (asset formats in `shared/game-assets.ts`, plugin ids in
  `shared/plugin-id.ts`). Never decide behavior by matching English text; use typed codes and
  fields.

## Readability

Write code a person enjoys reading; behavior never changes in a readability edit.

- Formatting is Biome's (`npm run lint` fails unformatted code); run `biome format --write` on
  what you touched, never hand-pack lines.
- A vocabulary is `export const EngineStatusCode = { Ready: "ready", … } as const` plus a type
  of the same name (`(typeof X)[keyof typeof X]`): PascalCase keys, values in their exact wire
  spelling, never renamed. Its home is the module that owns the type, or the top of the module
  for a private one; bind it to type maps with `satisfies`
  ([recipe](docs/agent/recipes.md#vocabulary)).
- A raw vocabulary literal (engine id, status, event or method name, phase) outside its home is
  a defect: write `CustomEvent.RunFinished`, `HostMethod.EventsList`, `UiEvent.PreviewFrame`.
  `scripts/check-vocabulary.ts` (in `check:static`) refuses the common shapes, and inline
  `new Promise(r => setTimeout(r, ms))` sleeps: use `node:timers/promises` (the seed's
  `loop/time.ts`).
- Name a condition with three or more operands, or `&&` mixed with `||`: a local const or a
  small predicate beside its type (`needsSignIn(engine)`). Compute repeated sub-expressions
  once. No nested ternaries, no `!` assertions in code you touch.
- Timeouts, caps, sizes and retries are SCREAMING_CASE consts at the top of their module, in
  `shared/duration.ts` units (the seed's `loop/time.ts`). User copy lives in
  `renderer/words.ts` or a module-local `MESSAGE` object; model prompts in a sibling
  `*-prompts.ts` or seed `prompts/*.md`.
- Functions aim for 60 lines (80 at most, cognitive complexity 15): guard clauses first, no
  `else` after `return`, no parameter reassignment, repeated steps become a named helper beside
  their domain.
- In `src/**` and `scripts/**` these limits, nested and useless ternaries, parameter
  reassignment, `else` after `return` and uncollapsed `if`s are Biome errors, so `npm run lint`
  fails on them; tests keep them as warnings (`npm run lint:warn` lists the backlog). An
  exception is a `biome-ignore` naming its reason, such as a function serialized into the page.
- Keep every "why" comment with its code; give each new export a one-line doc comment. Keep
  `data-*` selectors, aria-labels and visible strings byte-identical.

## Documentation and PRs

- Docs describe the current application. Update affected pages in the same PR; replace
  obsolete statements and link instead of copying.
- `evals/baselines/` and `evals/ledger/export-*.jsonl` are the only committed measurement records;
  everything else from a campaign stays local ([evals](docs/evals.md#what-is-committed)).
- Source copied into the repository (not an npm dependency the build bundles) is listed in
  `THIRD-PARTY-NOTICES.md` with its license; the build's notice generator cannot see it.
- Run `npm run verify:context` before the PR. It enforces overview ≤500 words, each product
  page ≤800, the handbook ≤4,000 and at most eight product pages. Shorten or reorganize when
  full; do not raise limits.
- For now all work integrates into `dev`: base work on `origin/dev` and open PRs against
  `dev`. Never push or open PRs to `main` unless the owner asks. The owner opens `dev` → `main`
  PRs by hand; only they run the billed macOS/Windows CI (`check` `fast`, `package`, `windows`,
  `terminal`), so dispatch one on your branch when a change is platform-sensitive.
- PRs contain code, tests/fixtures, attribution, changed docs and a validation summary per the
  PR template. Name docs updated or explain why they remain accurate. No session reports,
  filled plans, handoff JSON or raw report batches in Git.
- Temporary notes: one ignored `.studio-dev/notes/<task>.md`, under 80 lines, removed when
  complete. Raw evidence stays in `.studio-dev/evidence/`. Share a short issue or draft-PR
  summary only when handing work over; Git history keeps past decisions.

## Design foundations

For application UI/UX, visual design, design-system or product-copy work, read
[the design workflow](docs/agent/design.md) before editing. It owns the design principles and
routes to local skills; `variant` and `break` are opt-in exploration tools. Follow its rendered
review and evidence requirements.

## Setup and authorization

Authorization already given in the conversation persists. Restoring locked dependencies with
`npm ci` is included when needed to build, run or test the requested work; do not repeat it
without a missing or mismatched dependency. Dependency upgrades and global installations are
separate scope decisions. Use the installed Node 24; do not change machine-wide runtimes as an
incidental verification step.

## Don'ts

- Don't create, switch or remove branches/worktrees, install hooks, merge
  or release unless explicitly asked. Verification or merging authorizes none of these, nor
  workspace cleanup.
- Don't reference local-only documents (knowledge-map `localOnly`) from tracked files.
- Don't overwrite normal games, profiles, credentials or harness edits.

---
> Source: [genex-games/genex-desktop](https://github.com/genex-games/genex-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
