## emacs-proofread

> - Keep exactly two first-party packages: `lisp/proofread/` contains the core and

# README for Agents

## Architecture and Design

- Keep exactly two first-party packages: `lisp/proofread/` contains the core and
  every first-party backend feature, while `lisp/proofread-popup/` contains the
  optional frontend. Add first-party backends as `proofread-<backend>.el`
  features inside `proofread`; they share one distribution and release boundary.
- Keep dependencies directed toward the core. The core owns provider-neutral
  processing, orchestration, state, and aggregation; backends own options,
  identity, external I/O, parsing, and resources. Backends must not depend on
  one another, and the popup may depend only on the core. Keep shared invariants
  in the core instead of copying them, importing sibling private
  implementations, or storing derived state.
- Model a backend profile as an ordered collection of bindings. Each binding
  selects a registered backend and carries opaque backend-local options; an
  empty profile disables dispatch. Do not assume one binding per backend
  implementation. Snapshot the active profile per check, derive identity per
  binding, construct chunks once, and fan them out through the core scheduler.
- Give each scheduled item one binding owner and preserve that provenance in
  queues, caches, cancellation, results, diagnostics, and logs. Work owned by
  one binding must not cancel or supersede another binding's work. Failures are
  isolated, and core aggregation is deterministic regardless of completion
  order.
- Use registration and unregistration as the only dispatch seam; third-party
  backends require no core conditional or first-party loading metadata. Keep the
  shared contract public, and do not add cross-package `proofread--`
  dependencies. Submission is non-blocking and settles at most once; handles are
  opaque, cleanup is idempotent, and per-binding identities are stable,
  non-secret, and complete for cache compatibility.
- Test generic registration and multi-backend behavior with provider-neutral
  fake backends in the core suite; keep implementation tests with their backend.
  Update build, test, packaging, documentation, and release metadata together
  when source membership changes.

## Commits

When creating a commit, use the `commit` skill. Load
`.agents/skills/commit/SKILL.md`, follow its required staged-diff inspection
workflow, and write the commit message in the required GNU-style format.

## Emacs Lisp

When writing, editing, reviewing, or refactoring `.el` files, use the
`emacs-lisp-coding` skill first. Load
`.agents/skills/emacs-lisp-coding/SKILL.md` and its referenced style guide, then
follow that workflow for style, formatting, and validation.

## Emacs Evaluation and Tests

When Nix is available, evaluate and test this project through the packages
provided by this project's flake so Emacs runs with a completely clean init
directory. Prefer commands such as
`nix run .#emacs30-with-proofread -- --batch --eval ...` for evaluation and
`nix run .#emacs30-run-proofread-tests` or
`nix run .#emacs31-run-proofread-tests` for tests.

When Nix is not available, fall back to the system `emacs`, still using batch
mode and a temporary clean init directory where possible.

## Nix Code

When writing, editing, reviewing, or refactoring `.nix` files, use the
`nix-coding` skill first. Load `.agents/skills/nix-coding/SKILL.md` and its
referenced style guide, then follow that workflow for style, formatting, and
validation.

---
> Source: [brsvh/emacs-proofread](https://github.com/brsvh/emacs-proofread) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
