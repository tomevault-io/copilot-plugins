## go-harness

> @/Users/raezil/.codex/RTK.md

@/Users/raezil/.codex/RTK.md

# AGENTS.md

## Purpose

This repository uses agent-assisted development with a disciplined Go workflow.

`go-harness` is an agentic Go coding harness built on
`github.com/Protocol-Lattice/go-agent`. It provides a CLI/REPL that loads
markdown skills, persists session memory, discovers UTCP tools, and runs coding
tasks through filesystem, shell, and git provider binaries.

The goal is to make small, correct, maintainable changes without inventing
behavior, rewriting unrelated code, or weakening tests.

## Agent Stack

Use this stack together, in this priority order:

1. **Superpowers** - stay disciplined, verify assumptions, and avoid
   overengineering.
2. **RTK** - inspect, navigate, and execute repository changes safely.
3. **grill-me** - stress-test implementation plans before coding.
4. **Ponytail** - prevent over-engineering; prefer deleting, reusing,
   simplifying, and avoiding speculative abstractions before adding code.
5. **cc-golang-skills** - write idiomatic, tested, production-quality Go.
6. **Caveman** - compress communication without losing facts, commands, paths,
   errors, or test results.

`grill-me` is the plan-review layer. It must challenge weak assumptions, missing
edge cases, risky scope, vague requirements, and untested behavior before
non-trivial implementation starts.

Ponytail is the anti-overengineering layer. It must challenge whether new
code, new abstractions, new dependencies, new files, or broad rewrites are
actually necessary before implementation.

Caveman is the communication layer, not a shortcut. It must never override
Superpowers, RTK, grill-me, Ponytail, cc-golang-skills, tests, safety, or
correctness.

## Core Rules

1. Read this `AGENTS.md` before making changes.
2. Inspect the current repository before proposing implementation.
3. Do not invent files, APIs, packages, providers, commands, or behavior.
4. Prefer simple, boring, maintainable Go.
5. Make small, reviewable changes.
6. Keep public APIs stable unless the task explicitly requires breaking changes.
7. Add or update tests for every behavior change.
8. Run formatting and relevant tests before finishing.
9. Be honest about what changed, what was tested, and what was not tested.
10. Use `grill-me` for non-trivial plans before implementation.
11. Use Ponytail before coding to prefer the smallest correct change.
12. Use Caveman only to reduce output noise; never to reduce engineering rigor.

## Required Workflow

For every non-trivial task, follow this order:

1. **Read project instructions**

   - Read `AGENTS.md`.
   - Check relevant README sections, docs, examples, tests, and package layout.

2. **Inspect with RTK**

   - Use RTK for repository inspection, navigation, and command execution.
   - Identify relevant packages, commands, tests, examples, and existing
     conventions.
   - Do not assume architecture from memory.

3. **Plan**

   - Produce a short implementation plan before coding.
   - Include the problem, goal, constraints, risks, and acceptance criteria when
     useful.
   - Prefer minimal changes.
   - Avoid broad rewrites unless explicitly requested.

4. **Review with grill-me**

   - Stress-test the plan before coding.
   - Challenge unclear requirements, hidden assumptions, missing edge cases,
     risky scope, and weak tests.
   - Revise the plan if `grill-me` finds a real gap.
   - Do not use `grill-me` to expand scope beyond the user request.

5. **Simplify with Ponytail**

   - Ask whether the change needs new code at all.
   - Prefer using existing code, standard library, native platform features,
     smaller diffs, and deletion over new abstractions.
   - Reject speculative future-proofing, unnecessary dependencies, and broad
     rewrites unless the user explicitly asks for them.
   - Use `@ponytail-review` for diff review when available.
   - Use `@ponytail-audit` for whole-repository over-engineering audits when
     available.

6. **Implement**

   - Use idiomatic Go.
   - Keep interfaces small.
   - Use explicit error handling.
   - Use `context.Context` where cancellation, deadlines, I/O, or request scope
     matter.
   - Avoid unnecessary abstraction.
   - Avoid global mutable state unless clearly justified.
   - Preserve existing behavior unless the task requires changing it.

7. **Test**

   - Add or update tests for behavior changes.
   - Prefer table-driven tests where useful.
   - Cover success paths and failure paths.
   - Do not fake test results.

8. **Verify**

   Run relevant commands through RTK:

   ```sh
   rtk gofmt -w .
   rtk go test ./...
   rtk go vet ./...
   ```

   If the change touches build, providers, CLI behavior, or module state, also
   consider:

   ```sh
   rtk make build
   rtk make test
   rtk make tidy
   ```

9. **Report**

   Summarize what changed, why it changed, files changed, tests run, and known
   limitations or follow-ups.

## RTK Usage

Use RTK as the execution and repository-navigation layer.

Always prefix shell commands with `rtk`, including read-only commands:

```sh
rtk git status
rtk rg "pattern"
rtk sed -n '1,120p' README.md
rtk go test ./...
```

Use RTK to:

- Inspect the work tree.
- Read relevant files.
- Understand current architecture.
- Locate tests and examples.
- Apply focused changes.
- Avoid accidental unrelated edits.

RTK must not be used as an excuse to skip planning, tests, or verification.

## Superpowers Usage

Use Superpowers for disciplined behavior.

The agent must:

- Slow down before changing architecture.
- Verify assumptions from the repository.
- Prefer reversible changes.
- Avoid overengineering.
- Keep changes understandable.
- Stop and report uncertainty instead of inventing facts.
- Treat tests as part of the implementation, not as an afterthought.

## grill-me Usage

Use `grill-me` after repository inspection and planning, before non-trivial
implementation.

`grill-me` should ask:

- Is the plan too broad for the requested change?
- Are hidden assumptions verified from the repository?
- Are edge cases, failure paths, and compatibility concerns covered?
- Are tests specific enough to prove the behavior changed correctly?
- Could the same goal be achieved with a smaller diff?
- Does the plan preserve existing public APIs and behavior unless explicitly
  changed?

`grill-me` must not:

- Replace repository inspection.
- Delay simple fixes with unnecessary process.
- Add speculative features.
- Expand scope beyond the user request.
- Skip tests or verification.

Recommended activation language:

```text
Use grill-me to challenge the plan before coding.
Keep the critique focused on assumptions, edge cases, scope, and tests.
Do not expand scope beyond the task.
```

## Ponytail Usage

Use Ponytail as the anti-overengineering layer for coding and refactoring.

Ponytail should ask:

- Can this be solved by deleting code instead of adding code?
- Is there an existing function, package, helper, command, or test fixture to
  reuse?
- Is the standard library enough?
- Is this abstraction needed now, or is it speculative?
- Can the diff be smaller while preserving correctness?
- Does this dependency, config file, interface, cache, wrapper, or framework
  earn its place?

Use Ponytail commands when available in Codex:

```text
@ponytail-review
@ponytail-audit
@ponytail-debt
@ponytail-gain
@ponytail-help
```

`@ponytail-review` is preferred before finalizing non-trivial diffs. It should
identify unnecessary complexity, deletable code, duplicate helpers, needless
dependencies, and speculative abstractions.

Ponytail must not:

- Delete behavior the user asked to keep.
- Override tests, security, correctness, or compatibility.
- Excuse missing validation.
- Replace repository inspection.
- Block necessary code when a small, explicit implementation is the correct
  answer.

## cc-golang-skills Usage

Use cc-golang-skills for all Go implementation work.

Go code must be:

- Idiomatic.
- Formatted with `gofmt`.
- Tested.
- Explicit about errors.
- Minimal in dependencies.
- Clear in package boundaries.
- Safe for concurrent use where applicable.
- Easy to read and maintain.

Prefer the Go standard library unless a dependency is already present or clearly
necessary.

## Caveman Usage

Use Caveman as an output-compression and communication style layer.

Caveman is allowed when the task benefits from shorter agent messages, lower
token usage, or faster review. It must not weaken the engineering workflow.

Use Caveman to:

- Remove filler, repetition, and unnecessary explanations.
- Keep status updates short and action-focused.
- Prefer compact bullets, diffs, tables, and exact commands over long prose.
- Compress summaries while preserving important facts, file paths, commands,
  and test results.
- Keep code, tests, errors, and API names precise.

Caveman must not:

- Skip planning, repository inspection, tests, or verification.
- Hide uncertainty or omit important risks.
- Make code comments, public documentation, error messages, or user-facing API
  text unclear.
- Replace technical accuracy with jokes or vague shorthand.
- Compress away security, correctness, migration, or compatibility notes.

When Caveman conflicts with correctness, safety, tests, or maintainability,
correctness wins.

## Go Engineering Standards

### Package Design

- Keep package responsibilities clear.
- Avoid circular dependencies.
- Do not expose internals unnecessarily.
- Use `internal/` for private application or framework implementation.
- Keep `cmd/` packages thin.
- Do not add `pkg/` unless creating stable public packages intended for external
  use.

### Error Handling

- Return errors explicitly.
- Wrap errors with useful context.
- Do not leak unsafe internal errors to users.
- Do not use panic for normal control flow.

### Context

Use `context.Context` when functions involve:

- Network calls.
- I/O.
- Long-running work.
- Request-scoped operations.
- Cancellation.
- Deadlines.

Do not store context in structs unless there is a strong reason.

### Testing

- Prefer table-driven tests.
- Test public behavior.
- Include edge cases.
- Include error cases.
- Avoid brittle tests tied to implementation details.
- Use benchmarks only when performance is part of the task.

### Concurrency

- Avoid shared mutable state when possible.
- Protect shared state with synchronization.
- Use channels only when they simplify ownership or coordination.
- Avoid goroutine leaks.
- Respect context cancellation.

## go-harness Layout

Current top-level layout:

```text
cmd/go-harness/     CLI entry point and provider template
cmd/filesystem/     Filesystem UTCP CLI provider
cmd/shell/          Shell UTCP CLI provider
cmd/git/            Git UTCP CLI provider
internal/cli/       Command parsing and user-facing CLI commands
internal/harness/   Runtime, prompts, skills, memory, approval, normalization
bin/                Built binaries
```

Preserve this layout unless the task explicitly asks for architecture changes.

## Dependency Rules

- Do not add dependencies without justification.
- Prefer existing dependencies already used by the project.
- Prefer the standard library.
- Avoid large frameworks unless the project already depends on them.
- Update `go.mod` and `go.sum` only when necessary.

## Git and Change Hygiene

The agent must:

- Avoid unrelated formatting churn.
- Avoid rewriting files unnecessarily.
- Keep diffs focused.
- Preserve comments unless they are wrong or obsolete.
- Do not remove tests to make failures disappear.
- Do not silently delete features.
- Never revert user changes unless explicitly requested.

## Final Response Format

After completing a task, respond with:

````md
## Summary

- ...

## Files Changed

- ...

## Tests Run

```sh
...
```

## Notes

- ...
````

If tests were not run, say why. Do not claim tests passed unless they were
actually run.

## Forbidden Behavior

The agent must not:

- Invent missing APIs.
- Invent test results.
- Ignore failing tests.
- Rewrite unrelated code.
- Add unnecessary abstractions.
- Change module paths without instruction.
- Remove features silently.
- Hide uncertainty.
- Treat generated code as hand-written code unless required.
- Make large architecture changes without a short plan.

## Default Task Prompt

When asked to work on this repository, use this operating mode:

```text
Use AGENTS.md as the primary development instruction file.

Use Superpowers for disciplined agent behavior.
Use RTK for repository inspection and execution.
Use grill-me to challenge non-trivial implementation plans before coding.
Use Ponytail to prevent over-engineering, prefer deletion/reuse, and keep diffs
small.
Use cc-golang-skills for idiomatic, tested Go.
Use Caveman mode for concise, low-token output only; do not skip Superpowers,
RTK inspection, grill-me review, Ponytail simplification, cc-golang-skills
standards, tests, or precision.

Do not jump directly into coding.
Inspect the repository first.
Create a minimal implementation plan.
Run grill-me against non-trivial plans.
Run Ponytail review for non-trivial diffs when available.
Make small, reviewable changes.
Add or update tests for behavior changes.
Run relevant gofmt, go test, go vet, and Makefile commands through RTK.
Summarize changes honestly.
```

## Andrej Karpathy Skills

Use the guidance from:
~/.codex/skills/andrej-karpathy-skills/CLAUDE.md

Priority:
- Think before coding.
- Prefer simple solutions.
- Make surgical changes only.
- Define success criteria and verify with tests/builds.

---
> Source: [Protocol-Lattice/go-harness](https://github.com/Protocol-Lattice/go-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
