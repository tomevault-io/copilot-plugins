## ai-leetcode

> Build an evidence-backed algorithm book from structured knowledge, executable solutions, deterministic traces, and reproducible benchmarks.

# Repository Instructions

## Mission

Build an evidence-backed algorithm book from structured knowledge, executable solutions, deterministic traces, and reproducible benchmarks.

## Source-of-truth boundaries

- `content/`: problem, pattern, data-structure, and relationship knowledge.
- `solutions/`: executable source code and test fixtures.
- `artifacts/`: machine-produced traces and benchmark reports.
- `book/src/generated/`: disposable renderer output; never edit it by hand.
- `book/src/`: hand-written reader onboarding and book-level navigation only.

## Required workflow

Every new problem moves through these states in order:

`draft -> tested -> verified -> benchmarked -> visualized -> editorial-reviewed -> published`

An agent must not advance a state unless the corresponding acceptance criteria in `specs/` pass.

## Non-negotiable rules

1. Do not scrape or reproduce complete LeetCode problem statements.
2. Never claim that one solution is faster without a committed benchmark report and environment metadata.
3. Every complexity claim must include its assumptions and derivation.
4. Every published solution must compile or execute against all checked-in cases.
5. Every animation event must map to an explicit algorithm state transition.
6. Prefer stable IDs over display names in cross-references.
7. Keep generated files out of manual reviews unless the task is specifically about rendering.
8. AI-authored prose must distinguish verified facts, analysis, and editorial judgment.
9. Every algorithm presented in the published book must have a visual complexity
   diagram showing both time complexity and auxiliary space complexity.

## Writing rules

- Problem pages explain one problem deeply; pattern pages synthesize at least three problems.
- Start from the invariant and state representation, not from memorized code.
- Compare approaches by constraints, complexity, memory, implementation risk, and measured results.
- Place a Mermaid complexity diagram next to every algorithm. Each diagram must
  define its variables, state whether bounds are worst-case, expected, or
  amortized, expose assumptions such as constant-time hashing or a fixed input
  domain, and account for implementation storage such as copied buffers.
- Keep asymptotic complexity diagrams separate from benchmark charts: the former
  explain growth, while the latter report measurements from a named environment.
- Avoid repeating the same proof or template across chapters; link to the canonical pattern page.
- Use the templates in `templates/` and record the page budget before drafting long chapters.

## Code conventions

- Keep platform adapters separate from the core algorithm.
- Tests should call the core algorithm directly.
- Every published LeetCode problem must provide a Rust submission snippet whose
  method name, parameter types, return type, and `impl Solution` wrapper exactly
  match the judge template.
- A submission snippet must compile in the judge environment by itself: do not
  reference workspace crates or private helper modules, and do not redeclare
  `struct Solution` when the platform supplies it.
- Store each copyable submission snippet under `solutions/<id>/`, include that
  exact file in the book, and compile it in a test harness with a mock
  `Solution`. Never maintain separate display and tested copies.
- Label reusable core functions, Rust Playground programs, and direct LeetCode
  submissions separately. Do not tell readers that a core function can be
  pasted into the judge unless it satisfies the platform signature.
- Trace instrumentation must be optional and must not change algorithm semantics.
- Benchmarks run optimized builds and use deterministic, documented datasets.

---
> Source: [ZhangHanDong/ai-leetcode](https://github.com/ZhangHanDong/ai-leetcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
