## nixfied

> This is Nixfied’s active-context routing sheet. Before changing code, tests,

# Agent guide

This is Nixfied’s active-context routing sheet. Before changing code, tests,
docs, builds, or workflow:

1. Inspect `git status --short --untracked-files=all` and preserve unrelated work.
2. Read the authority routed below.
3. State the invariant, owner, rejection boundary, and proof before editing.

[`docs/CONTRACT.md`](docs/CONTRACT.md) is normative for product behavior and
the model/runtime ABI. [`capability.txt`](runtime/crates/nixfied-model/capability.txt)
is the authored wire inventory whose digest derives `runtimeAbi`, not generated
documentation.

## Core design

Nixfied uses correctness-by-construction. Correctness means invalid states are
unrepresentable or rejected at the owning boundary. Dexterity comes from fewer
owners and change sites. Simplicity means no duplicate authority or speculative
machinery.

- **Represent the domain.** Use product types for facts that coexist and sum
  types for alternatives. Use newtypes, private constructors, fallible
  conversions, and typestate for invariants. Match closed alternatives
  exhaustively. Use `Option` only for genuine absence, never for hidden
  alternatives or correlated state.
- **Close the boundary.** Parse untrusted input into domain types at the
  earliest boundary with enough information. Use fallible validation, lowering,
  or admission for cross-field, graph, host, time, and concurrency invariants
  that types cannot express. Preserve invariants through construction,
  conversion, defaults, and deserialization; reject incoherent input before
  side effects.
- **Give every fact one owner.** Derive repeated facts from one specification.
  Do not add duplicate registries, caches, seams, compatibility paths, or
  mutable authorities.
- **Fail closed and preserve phases.** Use typed redaction-safe errors and
  explicit checked outcomes. Keep admission, execution, lifecycle, output,
  containment, and cleanup failures distinct. Invalid admission starts no
  child process.
- **Prefer subtraction.** Choose the smallest design that preserves the
  invariant. Delete duplication, states, branches, and change sites rather
  than adding coordination or partial workarounds.
- **Prove the guarantee.** Test accepted construction, rejected inputs,
  invariant-preserving transformations, and runtime or relational behavior.
  Use compile-time proofs where the language supports them and runtime tests
  for runtime facts; tests support the design but do not replace it.

For every non-trivial change, answer:

1. What invariant must always hold?
2. Which invalid states are made unrepresentable?
3. Which owner rejects what the types cannot express, and when?
4. What tests or checks prove the invariant and cover coupled surfaces?

## Boundaries

- Nix evaluates, validates, builds, and realises. Rust admits, executes,
  reconciles, and cleans against `model.json` and OS reality.
- `model.json` is the only required semantic seam; `views/docs.md` is
  disposable. The runtime binary never invokes Nix or imports Nix expressions;
  declared child programs are a separate contract case.
- Keep runtime behavior domain-generic. The model remains tasks, services,
  inline invocations, static composite DAGs, and derived graph facts. Do not add
  runtime-owned caches, memberships, or dynamic orchestration.
- Keep runtime-owned host placement and secret values out of the model. Child
  environments stay hermetic. Admission, endpoint ownership, state cleanup,
  liveness, containment, and secret handling fail closed.
- Shell and gate scripts may orchestrate tests and assert results, but
  production graph, registry, validation, liveness, summary, and cleanup
  semantics stay in their owning layer. Nix does not supervise or reconcile;
  `nixfied-model` contains the shared typed contract, not runtime behavior.
- A new semantic kind, seam, mutable authority, compatibility path, daemon,
  or domain-specific runtime branch requires a contract-consistent design
  review before implementation.

## Authority

Read these before changing the corresponding area:

- User behavior, authoring, and recovery: [`README.md`](README.md),
  [`docs/GUIDE.md`](docs/GUIDE.md), [`docs/ADAPTERS.md`](docs/ADAPTERS.md).
- Model, ABI, lifecycle, output, and public surface: [`docs/CONTRACT.md`](docs/CONTRACT.md)
  and [`capability.txt`](runtime/crates/nixfied-model/capability.txt).
- Architecture and derived facts: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md),
  [`docs/DERIVATION_SPEC.md`](docs/DERIVATION_SPEC.md).
- Options and generated references: [`docs/OPTIONS.md`](docs/OPTIONS.md).
- Repository layout, tests, gates, and verification: [`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md).

## Change routing

| Change | Primary owner | Coupled surfaces |
| --- | --- | --- |
| Declarations, options, endpoints | `nix/modules/`, `nix/adapters/` | `README.md`, `docs/GUIDE.md`, generated `docs/OPTIONS.md` |
| Generated apps and help | `nix/project-apps.nix`, `nix/help-*.nix` | `docs/CONTRACT.md`, app and help tests |
| Resolution, validation, derivation, model/docs view | `nix/compiler/` | `docs/DERIVATION_SPEC.md`, Nix vectors and views |
| Model fields, ABI, serialization, structural validation | `nix/spec/`, `runtime/crates/nixfied-model/` | `capability.txt`, Nix/Rust constants, fixtures, contract tests |
| Admission, lifecycle, execution, and controls | `runtime/crates/nixfied-runtime/` | `docs/CONTRACT.md`, focused runtime tests |
| Install and upgrade behavior | `nix/install/`, `runtime/crates/nixfied-cli/` | `docs/GUIDE.md`, install and package checks |
| Builds, checks, gates, and CI | `nix/checks/`, `nix/gate*.nix`, `nix/dev.nix`, `.github/workflows/` | `docs/DEVELOPMENT.md` |

If a change names multiple participants, treat it as one atomic cutover rather
than separate implementations.

## Contract and delivery

- Treat model fields, enums, hidden commands, outputs, errors, and semantics as
  exact ABI. Update `capability.txt`, the ABI snapshot, Nix/Rust constants,
  producers, consumers, tests, fixtures, and docs together. Delete superseded
  readers, aliases, fallbacks, migrations, and dual paths; never reinterpret
  old bytes or rewrite append-only history.
- New model fields update `runtime/crates/nixfied-runtime/tests/capability_coverage.rs`.
  Derived facts update `docs/DERIVATION_SPEC.md`, both derivations, and both
  golden suites. Numeric versions change only when their semantics require it.
- Edit sources, not generated outputs. `nix/compiler/views.nix` renders
  `views/docs.md`; `nix/docs/options.nix` generates `docs/OPTIONS.md`.
- Use the narrowest proof first, then widen according to `docs/DEVELOPMENT.md`.
  `.#test` is fixture-backed, not raw Cargo; use `.#ci` for cross-layer changes
  and `--dirty` when a downstream gate must consume uncommitted changes. Report
  checks run and unverified platform, release, or integration coverage.
- Do not commit unless explicitly requested.

---
> Source: [willyrgf/nixfied](https://github.com/willyrgf/nixfied) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
