## timeline-hub

> - Prefer the smallest clear change that solves the current problem.

# Repository Agent Rules

## General defaults

- Prefer the smallest clear change that solves the current problem.
- Preserve existing behavior unless the request explicitly changes it.
- Avoid speculative architecture for hypothetical scale.
- If output expectations are unclear, ask before producing the final artifact.
- In plan/spec mode, prefer numbered clarification questions over assumptions.
- Keep recommendations scoped and actionable.
- Distinguish between:
  - now — required due to active impact
  - later — optional until a real trigger appears
- Do not introduce new abstractions or helpers unless required to solve the task correctly.
- Do not refactor unrelated code.
- Do not modify unrelated code or documentation.
- When editing docs, preserve existing structure and tone unless the task asks for a rewrite.
- When the response format is unconstrained, after completing a task, propose a short, high-signal next step or action that the user is likely to want next.

## Tooling

Environment and workflow:
- Use `uv` for dependency management and command execution.
- Add dependency ranges in `pyproject.toml`; rely on `uv.lock` for exact resolved versions.
- Install the dev environment with `uv sync --dev`.
- The repository uses `src/`; tests must import the installed package.
- Do not modify `sys.path` in tests or use pytest/pythonpath hacks.

Testing:
- Run tests with `uv run pytest`.
- Tests marked `live` hit external services and are skipped by default.
- Run live tests with `uv run pytest --live` when changing live integration coverage or when verifying external-service behavior.
- Keep existing unittest-style tests working under pytest.
- New tests should use pytest style unless there is a specific reason not to.

Test quality:
- Prefer fewer high-signal tests over broad low-value coverage.
- Add tests only when they protect a real invariant, failure mode, data contract, or behavior that would hurt if it silently changed.
- Avoid tests that merely assert default argument values, dataclass field defaults, trivial constructor wiring, or constant policy choices. Test explicit input -> behavior instead.
- Test defaults directly only when the default is an intentional stable public contract with meaningful user-facing behavior.
- Avoid tests that depend on exact checked-in config values or repo policy thresholds unless those exact values are an intentional stable public contract. Prefer explicit test fixtures that assert parsing, override precedence, validation, or resulting behavior.
- Avoid `__repr__` / formatting tests unless the exact representation is a stable public contract consumed by humans/tools. Do not test arbitrary visual output.
- Avoid tests that simply mirror implementation details, private helper structure, call order, or current refactor shape without validating observable behavior.
- Avoid dummy smoke tests that only instantiate objects or assert that code “does not crash” unless the absence of crashing is itself a meaningful regression boundary.
- Prefer tests for durable invariants: validation rules, conversion semantics, ordering guarantees, failure handling, idempotency, boundary behavior, persistence format, and externally visible API behavior.
- For data/domain code, prioritize edge cases that would corrupt stored data, hide invalid provider behavior, break replayability, or silently change model-facing features.
- If a change is purely mechanical and no durable behavior is affected, it is acceptable to add no tests rather than create weak tests.

Pre-commit:
- Pre-commit is the final enforcement gate.
- Do not bypass hooks.
- If hooks fail, fix the issues before committing.
- Ruff and Pyright must pass through the normal workflow.
- After completing any coding task, run `uv run pre-commit run --all-files`.
- Do not stop after tests, Ruff, or Pyright alone if pre-commit has not been run.
- Treat a failing pre-commit run as unfinished work and fix the issues before handing the result back for review.

## Code style

- Target modern Python as defined by `pyproject.toml`.
- Prefer current language features over legacy compatibility patterns.
- Do not add backward-compatibility patterns unless explicitly required.
- Prefer explicit code over clever abstractions.
- Prefer public documented APIs over private internals.
- Prefer fixing root causes over `Any`, `cast`, broad suppressions, or fragile hacks.
- Prefer explicit `None` checks and proper type narrowing.
- Use single quotes for normal strings.
- Use triple double quotes for triple-quoted strings and docstrings.
- Use Google-style docstrings.
- Do not add `from __future__ import annotations` unless there is a real need.
- Prefer snake_case naming.
- Use absolute imports from top-level packages.
- Do not rely on private attributes or methods unless explicitly required or there is no viable public alternative.
- Document intentional contract-level exceptions in `Raises:` sections.
- Do not document incidental internal exceptions unless they are part of intended API behavior.
- In behavioral and API-facing modules, prefer top-down organization:
  - public constants, types, and protocols
  - public classes and functions
  - private methods
  - private helper functions
- Readers should be able to understand the module's public surface before encountering implementation details.
- Avoid placing private helpers above the public APIs that use them unless there is a strong readability or dependency reason.
- Declaration, schema, model-definition, and infrastructure/support modules may keep dependency-first ordering when that structure is more natural.

Type checking:
- Do not distort architecture just to satisfy static analysis.
- If a library relies on dynamic runtime behavior that static analysis cannot model, prefer a narrow, well-commented suppression over architectural duplication or private API usage.

Markdown defaults:
- Do not use horizontal rules (`---`) to separate sections.
- Rely on headers (`#`, `##`, etc.) for structure and visual separation.
- Avoid redundant visual separators that add noise without improving readability.

## Operating assumptions

Audience and scale:
- primary user is the repository owner, possibly a few trusted users
- scheduled/background work may be unattended and reliability-sensitive
- API traffic may be bursty during collection jobs and must be throttled explicitly when needed
- developer time is the most constrained resource

Design principles:
- prefer simplicity and explicit assumptions over defensive completeness
- fail fast inside domain/data contracts
- long-running workers should log recoverable job failures and continue when safe
- keep logs concise and human-readable
- optimize for common paths and maintainability over exhaustive guards
- add stronger isolation or hard limits after real incidents, not speculatively
- keep ownership aligned with architectural layers: domain/lifecycle entities
  own domain facts, identity, and lifecycle-local state; services/orchestrators
  own runtime policy and cross-component coordination; transports own delivery
  mechanics; adapters own representation conversion; ML/runtime code owns model
  and feature-extraction concerns
- lifecycle convenience is not sufficient ownership justification
- do not place higher-level concerns into lower-level entities merely because
  cleanup, access, or lifecycle management is easier there
- prefer explicit cleanup over downward coupling

Documentation of invariants:
- Persist important architectural invariants and source-of-truth assumptions in docstrings near the owning class or function.
- Document non-obvious contracts when a reasonable reviewer might otherwise infer the wrong behavior.
- Prefer documenting the invariant once at the highest-value location.
- Especially document:
  - what state is authoritative
  - what is cache-like or derived
  - ordering and grouping guarantees
  - retry, throttling, and concurrency assumptions
  - single-writer or lifecycle assumptions
- Do not add generic or redundant docstrings.

## Review expectations
Reviews should evaluate:
- abstraction boundaries
- architecture fit by layer
- simplicity
- maintainability
- failure modes and cleanup
- API surface
- naming quality
- modern Python usage

Prefer comments only when they clarify non-obvious invariants, tradeoffs, or failure modes.

Review feedback should explicitly call out hidden coupling and implicit assumptions.

Infrastructure code must remain generic and domain-independent.

For internal packages:
- prefer empty `__init__.py`
- do not create package-level APIs unless explicitly requested
- do not re-export internal symbols for convenience

Review feedback should be grouped as:
- critical issues
- important improvements
- optional polish

## Commit messages

Use Conventional Commits.

Format:
- `type(scope): short description`

Allowed types:
- feat
- fix
- refactor
- test
- docs
- build
- chore

Type guidance:
- `feat` = new capability or new managed behavior
- `fix` = corrected existing runtime behavior without introducing a materially new capability
- `refactor` = restructuring without intended behavior change
- `test` = add or modify tests without changing application behavior
- `docs` = documentation-only changes
- `build` = dependency management, package/build configuration, lockfiles, runtime version constraints, Docker, and deployment build setup
- `chore` = repository maintenance that does not affect build, dependency resolution, runtime packaging, or application behavior

Core rule:
- describe the primary system change introduced by the full diff
- infer the commit message from the final staged diff first
- use conversation context only to disambiguate intent
- do not anchor on the last edited function, latest bug fix, or local implementation detail
- when a diff changes rules, policies, contracts, workflows, or instructions, describe the behavior being changed rather than the fact that text changed
- if the affected actor matters for understanding the change, name it plainly in the subject

Commit-intent pass:
1. Inspect the final diff before every commit.
   - Use `git diff --cached` if changes are staged.
   - Otherwise use `git diff`.
2. Summarize the complete semantic change set in 3-5 bullets.
3. Identify the dominant system-level change introduced by the full diff.
4. Choose the Conventional Commit message from that dominant change.
5. Do not anchor on:
   - the latest bug fix
   - the last edited function
   - the most recent review comment
   - a small validation or edge-case change that only supports a broader feature
6. If the diff contains a broad feature plus supporting fixes, tests, or docs, the subject must describe the broad feature. Put supporting details in the body when useful.
7. If the diff appears to mix unrelated intents, propose the preferred split, but still choose the best single commit message if the current diff must remain as-is.
8. For broad diffs, prefer a commit body that explains:
   - why the change exists
   - the main behavior, API, or entity changes
   - important invariants or guarantees
9. Use `refactor` for intentional presentation, wording, naming, metadata, layout, or formatting changes when they do not add a new capability and do not correct behavior that was explicitly wrong under the existing contract.

Type selection:
- if the diff introduces a new stored artifact, cache, lifecycle, workflow, API behavior, or managed derived state, use `feat`
- do not use `fix` just because the diff contains error handling, validation tightening, or retry logic
- if error handling or consistency logic exists mainly to support a new capability, the type is still `feat`
- use `fix` only when the dominant purpose is correcting wrong behavior in an existing intended feature
- use `build` for uv, dependency, lockfile, Docker, runtime, packaging, or deployment build changes

Breaking changes:
- use `type!:` or `type(scope)!:`
- include a `BREAKING CHANGE:` footer
- assume config, env-var, public API, persisted-format, and runtime-version changes are breaking unless clearly proven otherwise

Subject rules:
- lowercase by default
- imperative mood
- ≤72 characters
- system-level wording
- name the dominant capability or behavior changed
- do not use tools, checker complaints, or mechanical diff descriptions as the main subject
- avoid vague terms like `cleanup`, `tweaks`, or `guidance`
- avoid backticks in subjects unless they add clear value
- put implementation details in the body, not the subject

Examples:
- `build: migrate environment management to uv`
- `feat(api): add throttled async client`
- `fix(client): retry malformed responses`
- `refactor(storage): separate snapshot persistence from collection`
- `test(parser): cover retry exhaustion`
- `docs: clarify agent commit rules`
- Bad when the full diff adds convenience properties, repr output, and validation:
  `feat(api): update repr output`
- Better:
  `feat(api): add convenience properties and validation`

Commit bodies:
- optional for tiny obvious changes
- preferred for non-trivial refactors, behavior-preserving changes, broad diffs, or lifecycle/invariant changes
- explain why, summarize scope, and clarify important guarantees
- use normal sentence case
- keep secondary error-handling details in the body when they support a larger primary change

Scopes:
- use stable architectural subsystems
- use scope only when the change affects a clear, meaningful component
- omit scope when the change is cross-cutting or repository-level
- choose scope by system intent, not just touched file
- do not infer scope from filenames alone
- root-level files usually omit scope

Commit coherence:
- keep commits conceptually coherent by subsystem and intent
- split unrelated changes
- do not mix substantial runtime logic changes with unrelated tooling or formatting
- use kebab-case for scopes when a separator is needed; avoid snake_case scopes

---
> Source: [cayn-one/timeline-hub](https://github.com/cayn-one/timeline-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
