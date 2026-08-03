## rob

> Roblox game written in native Luau. This file is the canonical instruction set

# rob — agent instructions

Roblox game written in native Luau. This file is the canonical instruction set
for ALL coding agents (Claude Code reads it via CLAUDE.md; Codex reads it
directly). Keep it up to date: when you change tooling, structure, or policy,
update this file in the same commit.

## Project map

```
src/shared/    Pure Luau only. No Roblox globals (game, Instance, task, …).
               Runs under Lune. This is where game logic lives.
src/server/    Services/  — pure orchestration, all effects behind injected deps (Lune-testable)
               Adapters/  — thin wrappers around Roblox services (DataStore, …); NO logic
               init.server.luau — composition root; the only place adapters get wired in
src/client/    Thin rendering/input glue. No game logic.
tests/         Lune unit tests (*.spec.luau) + fakes in tests/helpers/
lune/          Task scripts (`lune run <name>`) + vendored libs in lune/lib/
docs/          Design notes, including the cloud integration test stub
```

Layer rules (enforced by review, not tooling — do not break them):

1. `src/shared/**` and `src/server/Services/**` must never touch a Roblox
   global. If code needs a Roblox service, define a minimal structural
   interface (see `StoreLike` in `CurrencyService.luau`), implement it in
   `src/server/Adapters/`, and inject it from the composition root.
2. Adapters contain zero logic — one Roblox API call per method, nothing else.
   Logic in an adapter is logic you can't test.
3. Composition roots (`init.server.luau`, `init.client.luau`) only wire things
   together. If a root grows logic, extract it into a service.

## Quality gates — always run them

**Before considering ANY work done, run `lune run check` from the repo root
and get every gate green.** No exceptions, including "trivial" changes.

| Gate | Command | What it catches |
|---|---|---|
| format | `stylua --check src tests lune` | style drift |
| lint | `selene src tests lune` | undefined globals, suspicious code |
| types | `lune run analyze` | strict Luau type errors vs real Roblox/Lune APIs |
| unit tests | `lune run test` | behavior regressions |
| build | `rojo build default.project.json --output build.rbxl` | invalid project/instance tree |

`stylua src tests lune` (no `--check`) auto-fixes formatting.
`lune run test -- --update-snapshots` refreshes tiniest snapshots.
CI (`.github/workflows/ci.yml`) runs the same gates — if it's red there, it
was runnable locally first.

## Guardrails

- **Never trust your memorized knowledge of library versions or APIs.** The
  Roblox toolchain moves fast and your training data is stale. Before adding,
  upgrading, or writing code against any tool or library: check the actual
  GitHub releases page / registry entry / `--help` output / bundled type
  definitions, and pin exact versions. This applies to Roblox engine APIs too
  — verify against the type definitions that `lune run analyze` downloads
  (`.cache/globalTypes.d.luau`) rather than assuming a method exists.
- **All tools are pinned in `rokit.toml`.** Install with `rokit install`;
  never install ad-hoc tool versions or invoke tools not listed there. To
  upgrade a tool: verify the new version's release notes, bump `rokit.toml`,
  run all gates, and note anything that changed behavior.
- **Do not commit generated artifacts**: `build.rbxl`, `sourcemap.json`,
  `.cache/` are gitignored — keep it that way. `roblox.yml` IS committed on
  purpose (pinned selene std; refresh with `selene update-roblox-std`).
- **Never edit `lune/lib/**` (vendored third-party code).** See
  `lune/lib/tiniest/VENDOR.md` for how to update it.
- **Secrets never enter this repo.** Roblox Open Cloud API keys live in
  GitHub Actions secrets / local env vars only.

## Require rules (important — two runtimes, one style)

- Inside `src/**`: **relative string requires only** — `require("./Sibling")`,
  `require("../../shared/Currency")`. In an `init.luau`, a child of the module
  is `require("@self/Child")`. These resolve identically in the Roblox engine
  (require-by-string) and Lune because `default.project.json` mounts
  `src/shared` as a sibling of `src/server` inside ServerScriptService.
  Do NOT use `.luaurc` aliases in `src/**` — engine support for aliases is not
  established.
- Inside `tests/**` and `lune/**` (Lune-only code): aliases from `.luaurc` are
  fine and preferred: `@shared/...`, `@server/...`, `@tiniest/...`, plus the
  Lune builtins `@lune/fs`, `@lune/process`, etc.
- Composition roots and adapters may use classic instance access
  (`game:GetService(...)`) — they are Roblox-only glue.

## Luau strict-mode notes

- All code is `--!strict` (set project-wide in `.luaurc`).
- Don't compare metatable-based types (e.g. `Wallet`) against `nil` with
  `==`/`~=` — strict Luau rejects it ("do not have the same metatable").
  Use truthiness: `if not wallet then`.
- Local runs of `lune run check` may skip the types gate if luau-lsp isn't
  installed; CI always runs it. Don't treat "passed locally" as green if the
  types gate was the one that failed to run.

## Testing policy

- Every module in `src/shared/**` gets a spec in `tests/**` mirroring its
  path (`src/shared/Currency/Wallet.luau` → `tests/Currency/Wallet.spec.luau`).
- Every service in `src/server/Services/**` gets a spec that constructs it
  with fakes (see `tests/helpers/FakeDataStore.luau`). If you can't construct
  a service without Roblox, its design is wrong — extract an adapter.
- Spec format: the file returns `function(t)` and registers
  `t.describe`/`t.test` blocks; the runner (`lune/test.luau`) discovers
  `tests/**/*.spec.luau` automatically — no registration list to maintain.
- Adapters and composition roots are NOT unit-tested. They stay thin and are
  covered by cloud integration tests later
  (see `docs/cloud-integration-tests.md` — not wired up yet).
- Tests must be deterministic: no timing dependence, no ordering dependence.
  Inject clocks/randomness as deps if a feature needs them.

## Dependency policy

There are currently **zero runtime dependencies** — do not add one casually.
When a real need appears (e.g. a persistence library with session locking):

1. Check the package exists and is maintained on the **pesde** registry
   (https://pesde.dev) — pesde is the active package manager for Luau
   (Wally is stale since 2024; pesde can also consume Wally packages).
2. Verify the version you're adding against its actual release notes.
3. Add pesde itself to `rokit.toml` (verify its current version first),
   commit the manifest + lockfile, and update this section.

Vendored code (currently only `tiniest`) lives in `lune/lib/` with a
`VENDOR.md` recording provenance.

## Toolchain reference

Pinned in `rokit.toml` (see that file for exact versions): Rojo (build/sync),
StyLua (format), selene (lint), luau-lsp (type analysis), Lune (Luau runtime
for tests/scripts). Bootstrap on a fresh machine:

```sh
# install rokit once: https://github.com/rojo-rbx/rokit
rokit install     # installs all pinned tools
lune setup        # installs @lune/* type definitions (once per Lune version)
lune run check    # everything should be green before you start work
```

## Roblox-facing work (out of local scope)

Anything that needs a real Roblox server — DataStore behavior, replication,
Player lifecycle — is validated via Roblox Open Cloud, not locally. The
planned harness is documented in `docs/cloud-integration-tests.md`. Until it
exists, keep Roblox-touching code inside adapters/roots so the untestable
surface stays minimal.

---
> Source: [dinoderek/rob](https://github.com/dinoderek/rob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
