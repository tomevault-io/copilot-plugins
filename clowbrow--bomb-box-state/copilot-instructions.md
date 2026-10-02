## bomb-box-state

> These instructions apply to the entire repository.

# Repository instructions

These instructions apply to the entire repository.

## Source of truth

- Treat `docs/rules.md` as the normative gameplay and state-transition
  specification.
- Do not silently choose behavior where the specification is unresolved. Call
  out the ambiguity and update the specification as part of the change when a
  decision is made.
- Keep `docs/architecture.md` and `docs/development.md` accurate when a change
  affects architecture, portability, build, or verification guidance.

## Tests are part of every feature

- Every feature addition or behavior change must include automated tests in the
  same change. A feature is not complete without its tests.
- For a bug fix, first add or identify a test that fails for the reported
  behavior, then make it pass.
- Prefer specification-level scenarios under `tests/behavior/`. Construct the
  complete input state, perform one public operation, and compare the complete
  ordered result: acceptance or rejection, ticks, events, resolved state,
  outcome, and history or lifecycle effects when relevant.
- Test normal behavior, meaningful boundary cases, invalid or rejected input,
  and interactions with existing rules. Features involving history or level
  lifetime must test repeated rewind, the beginning of history, branching after
  rewind, terminal states, and replacement-level isolation as applicable.
- Determinism is observable behavior. Add tests that catch dependence on entity
  IDs, container iteration order, platform details, wall-clock time, or other
  non-rule inputs whenever a feature could introduce such dependence.
- Keep tests platform-independent so the same behavior corpus can run against
  native, WebAssembly, and Unreal-facing adapters. Add boundary-specific tests
  for an adapter or ABI when changing that boundary.
- Do not delete, weaken, or over-specialize an existing test merely to make a
  change pass. If the specification intentionally changes, update the test and
  document the reason.
- At minimum, build and run the native test preset before declaring an
  implementation change complete:

  ```sh
  cmake --preset native-debug
  cmake --build --preset native-debug
  ctest --preset native-debug
  ```

- Run the sanitized preset for changes involving ownership, lifetime, memory,
  collections, serialization, or ABI code when the required toolchain is
  available. Run applicable WebAssembly or integration tests when changing
  those surfaces. Report any test suite that could not be run and why.
- Do not run `ctest --preset native-sanitized` on the current Apple
  Silicon/macOS development host, whether from the Codex workspace sandbox or
  a normal terminal. The Apple sanitizer runtime stalls during GoogleTest
  discovery before cases run and may leave test processes consuming CPU. The
  preset may be configured and built locally, but execute its tests through
  the Ubuntu sanitizer CI job or another known-working Linux environment;
  record this host-toolchain limitation in verification results.
- Documentation-only edits do not require a new executable test unless they
  introduce or change specified behavior that already has an implementation.

## Title-independent naming

- Treat every working or final game title as presentation metadata, not as a
  source-code identity.
- Do not derive namespaces, classes or other types, functions, C ABI symbols,
  build targets or options, integration modules, filenames, directory names,
  or serialized identifiers from the game's title.
- Name code and technical artifacts for their stable domain role instead, so a
  game-title change does not require an API, ABI, build, or data-format rename.

## Design for embedding in a game

- Treat `game_rules_state` as a headless rules library embedded inside a larger
  game, not as an application. The host owns presentation, animation, audio,
  input mapping, persistence, and scheduling.
- Keep the core portable C++20 with no platform or runtime dependencies. The
  one approved source dependency is the pinned, vendored C99 yyjson parser;
  keep it private, unmodified, license-attributed, and symbol-isolated as
  documented under `vendor/yyjson/`. The core must not depend on Unreal,
  Emscripten, a renderer, an input device, a filesystem, environment variables,
  wall-clock time, threads, or random-number sources. Platform-specific code
  belongs under `integrations/`.
- Preserve deterministic, replayable behavior. Given the same loaded level,
  resolved state, and command, the logical result and ordered events must be
  identical on every supported host.
- Keep mutable state owned by an explicit engine instance. Avoid global mutable
  state and singletons so a host can run, replace, destroy, or test multiple
  game sessions independently.
- Make lifecycle, ownership, and failure behavior explicit. Level replacement,
  rewind history, terminal state, invalid input, and engine destruction must
  not leak state across sessions or leave partially committed results.
- Commit rules transitions atomically from immutable pre-transition snapshots.
  Never expose a host callback or public state view into a partially mutated
  tick or turn.
- Return authoritative snapshots and semantic events; do not require the host
  to reconstruct state from animation events. Keep presentation timing and
  interpolation outside the rules engine.
- Keep public interfaces practical for both a primitive C ABI and thin Unreal
  and WebAssembly adapters. Use fixed-width values and explicit error handling
  at the C boundary, and never expose C++ standard-library objects across it.
- Prefer clear ownership, bounded per-operation work, and predictable
  allocations in hot paths. Do not trade correctness or determinism for a
  speculative optimization; measure first and preserve the behavior tests when
  optimizing.
- Avoid tying rule behavior to frame rate or asynchronous completion order. A
  host may animate results over many frames, but the rules result must not
  change with rendering cadence.

---
> Source: [CLowbrow/bomb-box-state](https://github.com/CLowbrow/bomb-box-state) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
