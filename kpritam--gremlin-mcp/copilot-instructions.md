## gremlin-mcp

> Authoritative operational standard for all AI coding agents (Copilot, Claude, GPT, others) contributing to this repository. Defines mandatory engineering rules, workflow protocol, architectural constraints, and decision heuristics to ensure consistent, safe, high‑quality, effect‑based TypeScript contributions.

# AI AGENTS GUIDELINES

## 1. Title & Purpose

Authoritative operational standard for all AI coding agents (Copilot, Claude, GPT, others) contributing to this repository. Defines mandatory engineering rules, workflow protocol, architectural constraints, and decision heuristics to ensure consistent, safe, high‑quality, effect‑based TypeScript contributions.

## 2. Scope (What this file governs / What it does not)

Governs: Code generation, refactors, tests, docs authored by AI; architectural changes; effect modeling; schema & Gremlin interactions; error & data modeling; PR structure; automation hygiene.
Does NOT govern: Human editorial style guides, product roadmap, infrastructure ops, non-code repository assets. If a conflict exists with LICENSE or SECURITY.md, those take precedence.

## 3. Core Principles (short list)

1. KISS – minimal viable abstraction.
2. YAGNI – add complexity only with a concrete second use.
3. DRY – no duplicated logic; prefer reuse/composition.
4. Purity – isolate side effects at explicit boundaries.
5. Explicitness – types, errors, contracts are declared, not inferred from context.
6. Determinism – schema generation & transformations must be reproducible.
7. Composability – build small effectful units combined via functional operators.
8. Safety – no unchecked external data, no silent failures.
9. Observability of intent – names reveal purpose; no hidden coupling.
10. Focus – each change addresses one coherent concern.

## 4. Operational Rules (MUST / MUST NOT)

### Architecture & Design

MUST:

- Preserve existing layered separation (config / gremlin services / schema / handlers / utils / models).
- Keep modules single‑purpose; split when a file exceeds clearly separated responsibilities (heuristic: > ~300 lines with 2+ conceptual domains).
- Encapsulate traversal construction behind utilities/services; expose intention, not query strings.
- Represent cross‑cutting concerns (config, schema cache, connection) via injectable services.
- Maintain deterministic schema assembly (stable ordering, idempotent recomputation, cache invalidation explicit).
- Introduce an abstraction only after a second concrete call site or clear complexity reduction.
- Ensure new layers or services declare explicit dependency requirements (no implicit global access).

MUST NOT:

- Mix query building logic with schema generation or result parsing.
- Create circular dependencies between modules.
- Leak internal traversal fragments to handlers or tests.
- Add global singletons outside controlled Layer/service patterns.
- Hide IO inside ostensibly pure helpers.

### TypeScript & Types

MUST:

- Use strict, explicit public function signatures (return + parameter types).
- Prefer `type` for unions/intersections/discriminated unions; `interface` for extensible object contracts.
- Use discriminated unions for state variants instead of boolean flags.
- Prefer readonly & immutable data structures unless a justified hotspot.
- Replace magic literals with named `const` declarations.
- Use `unknown` + type guards instead of `any` for external/unvalidated data.
- Export only stable public surface from barrel indexes; keep internals file‑scoped.

MUST NOT:

- Use `any` (except in a boundary shim with justification comment).
- Silence compiler warnings via assertions (`as`, `!`) unless after explicit validation.
- Re‑declare types already available through inference if local, unless clarity improves public API.
- Overuse generics where a concrete type is simpler.

### Functional & Effects

MUST:

- Model all side effects as typed Effects (or typed result constructs) rather than raw Promises.
- Keep effect execution (unsafe run / actual IO) at boundary layers (server entrypoints, handler adapters, CLI bootstrap).
- Compose behavior with `pipe`, `map`, `flatMap`, `mapError`, avoiding nested callbacks.
- Separate pure data transformation from effect orchestration.
- Provide explicit effect return type annotations for exported functions.

MUST NOT:

- Interleave imperative mutation with effect composition.
- Block or busy‑wait inside effects; use async composition.
- Partially execute effects during module top‑level evaluation.

### Error Handling

MUST:

- Represent failures using typed error/result constructs (Either/Option/Result style or domain error classes) – never raw throwing in core logic.
- Include structured context fields (e.g. code, operation, parameters) in error values.
- Distinguish operational (retryable) vs. domain (validation) vs. programmer errors via type shape or discriminant.
- Fail fast on invalid schema definitions; do not continue with partial state.
- Convert external/library exceptions into domain error representations at the boundary.

MUST NOT:

- Throw raw errors inside domain, parsing, schema, or transformation layers.
- Swallow or log‑only errors without returning a typed failure.
- Return opaque string errors.

### Data & Models

MUST:

- Validate external/untrusted data immediately (type guards or schema parsing) before internal use.
- Keep parsing & normalization isolated from business logic (e.g., dedicated parser utilities).
- Maintain canonical model definitions in `models` with single source of truth.
- Preserve immutability for model instances; derive new copies instead of mutating.
- Keep import/export data transformations reversible where feasible.

MUST NOT:

- Duplicate structural model definitions across modules.
- Mutate shared cached structures except via controlled service functions.
- Embed parsing logic deep inside unrelated transformations.

### Gremlin / Graph Layer

MUST:

- Build traversals through dedicated utilities/services that encapsulate raw string construction.
- Constrain dynamic segments; sanitize / validate inputs before interpolation.
- Keep connection management and schema caching responsibilities within their services.
- Provide typed representations for traversal results (never use untyped `any`).
- Model relationship patterns and schema derivation deterministically (stable sort / ordering).

MUST NOT:

- Inline ad‑hoc traversal strings inside handlers or tests (route via builder functions).
- Expose raw driver objects beyond the connection/service layer.
- Cache schema implicitly (all caching must be intentional and invalidation explicit).

### Performance & Efficiency

MUST:

- Measure before optimizing (add micro‑benchmark or profiling note if optimizing).
- Leverage caching layers already present (schema cache) instead of duplicating.
- Prefer data streaming / batching utilities for large import/export operations.
- Consider algorithmic clarity first; only introduce complexity for validated hotspots.

MUST NOT:

- Prematurely micro‑optimize straightforward pure functions.
- Introduce shared mutable state for performance without measurement and justification comment.

### Testing

MUST:

- Test public APIs, critical transformations, schema assembly logic, traversal building, and parsing paths (success + failure).
- Include edge cases: empty inputs, invalid schema definitions, unexpected external data shapes.
- Use focused fixtures; keep test data minimal & intention‑revealing.
- Assert on structured error variants, not string messages.
- Maintain deterministic snapshot or structural expectations (no brittle ordering unless guaranteed).

MUST NOT:

- Over‑mock internal pure functions (prefer real composition for confidence).
- Combine unrelated concerns in a single test case.
- Depend on network/external services in unit tests (integration tests only for real Gremlin).

### Documentation

MUST:

- Document only non‑trivial public APIs, boundary layers, architectural decisions, complex effect orchestration.
- Explain why over what; code already shows mechanics.
- Update docs (including this file) when introducing new architectural patterns.

MUST NOT:

- Document trivial helpers, obvious loops, or simple type wrappers.
- Include AI meta commentary or process narrative.
- Duplicate rule statements across files (this file is canonical for agent conduct).

### Security & Safety

MUST:

- Treat all external inputs (network, file import, environment variables) as untrusted until validated.
- Avoid embedding credentials or secrets in code or tests.
- Sanitize Gremlin dynamic fragments (labels, property names) before inclusion.
- Propagate only necessary error context (avoid leaking secrets in error values).

MUST NOT:

- Log sensitive configuration values.
- Introduce dynamic `eval` or equivalent reflection constructs.
- Hardcode endpoints or credentials outside configuration.

## 5. Workflow for AI Agents (Step-by-step contribution protocol)

1. Read relevant source modules & existing tests for the target area.
2. Identify change class: feature / fix / refactor / doc / test.
3. Enumerate impacts: types, effects, services, schema, tests.
4. Plan minimal set of edits (avoid speculative abstractions).
5. If adding behavior: write or update failing tests first (when feasible).
6. Implement pure transformations first; then wire effects; finally adapt handlers.
7. Model all new failure modes explicitly (typed errors or result variants).
8. Update schema or traversal builders if new graph constructs are required.
9. Run validation commands (lint, type-check, tests, coverage if relevant).
10. Ensure no duplication; refactor only within scope.
11. Update documentation only if architectural or boundary contract changed.
12. Prepare concise commit(s) & PR description summarizing intent and surface changes.
13. Re-run full validation (`npm run validate`).
14. Submit; do not auto-resolve review comments without applying changes.

## 6. Code Style & Formatting Conventions (only non-obvious items)

- Rely on existing lint + prettier; do not hand-format for alignment.
- Prefer early returns over deep nesting.
- Discriminant property name: `kind` or `type` (be consistent within a domain cluster).
- Group exports: types first, then constants, then functions, then service constructors.
- Avoid wildcard (`*`) exports; use explicit named exports for clarity.
- Keep function bodies < ~40 lines; extract helpers above that threshold.
- Name pure functions with indicative verbs (`buildX`, `deriveY`, `parseZ`).
- Async/effect functions use present tense action (`fetchSchema`, `executeTraversal`).
- Error types end with `Error` and include discriminant.
- Prefer object parameter objects for functions with > 3 related parameters (named options).

## 7. Decision-Making Heuristics

- Introduce abstraction only after two concrete usages (Rule of 2) unless complexity risk is imminent.
- If adding a conditional with more than 2 boolean flags, prefer a discriminated union.
- When a function handles more than one conceptual responsibility, split.
- If a test mocks more than one service, reconsider design boundaries.
- Prefer composition (pipe of small steps) over large monolith transformations.
- Stop refactoring when: duplication removed, intent clear, metrics stable, and no further simplification without speculation.
- Choose clarity over cleverness every time; avoid micro-optimizations early.

## 8. Anti-Patterns & Red Flags

- Raw `throw` in core logic.
- Hidden side effects inside utility modules.
- Catch-all `any` propagation.
- Ad-hoc inline Gremlin traversals in handlers.
- Copy-pasted schema derivation logic.
- Large god modules (> ~500 lines) spanning multiple domains.
- Boolean parameter proliferation (`doThing(x, true, false)`).
- Silent error suppression or logging without returning failure variants.
- Unvalidated external data flowing into transformations.
- Excessive test fixture complexity masking real invariants.

## 9. Examples

### Error Modeling (Good vs Bad)

Good:

```ts
type QueryError = { kind: 'QueryError'; query: string; cause: unknown };
return Effect.fail<QueryError>({ kind: 'QueryError', query, cause: e });
```

Bad:

```ts
throw new Error('query failed');
```

### Boundary Effect Execution

```ts
// handler adapter
export const run = (input: Input) => Effect.runPromise(program(input));
// program is pure Effect value composed elsewhere
```

### Gremlin Traversal Abstraction

```ts
export const buildVertexById = (id: string) => g => g.V(id).limit(1);
export const fetchVertex = (id: string) => executeTraversal(buildVertexById(id));
```

### Schema Generation Pipeline

```ts
const assemble = pipe(loadRawLabels, map(derivePatterns), map(applyOrdering), map(cacheResult));
```

## 10. Quick Reference Cheat Sheet

Top 10 DO:

1. Keep functions small & pure.
2. Model failures with typed results.
3. Encapsulate Gremlin traversals.
4. Validate external data immediately.
5. Use discriminated unions for variants.
6. Run full validation before PR.
7. Add/update tests with behavior change.
8. Maintain deterministic schema assembly.
9. Limit effect execution to boundaries.
10. Use explicit, intention‑revealing names.

Top 10 DON'T:

1. Don't throw in core logic.
2. Don't use `any` (except validated boundary shims).
3. Don't inline ad-hoc Gremlin strings.
4. Don't duplicate transformation logic.
5. Don't mutate shared state.
6. Don't mix unrelated concerns in one PR.
7. Don't silence or swallow errors.
8. Don't add speculative abstractions.
9. Don't document trivial code.
10. Don't introduce circular dependencies.

Common Failure Modes:

- Unmodeled new error paths.
- Overly broad test fixtures masking edge cases.
- Accidental side effects in pure utilities.
- Data parsed too late causing unsafe assumptions.
- Non-deterministic schema ordering leading to flaky tests.

Fast Submission Checklist:

- [ ] Change scoped & necessary (KISS/YAGNI respected)
- [ ] No duplication introduced
- [ ] Types & effects explicit
- [ ] Errors modeled (no raw throw)
- [ ] External data validated early
- [ ] Traversals encapsulated
- [ ] Tests added/updated (pass locally)
- [ ] Lint + type-check + format pass
- [ ] Docs updated only if needed
- [ ] Commit messages clear & imperative

---

---
> Source: [kpritam/gremlin-mcp](https://github.com/kpritam/gremlin-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
