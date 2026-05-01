## mdtest-cli

> This repository is for `mdtest`, a test runner that uses AI agents as interpreters for Markdown test cases. The project dogfoods itself — mdtest's own integration tests are `.test.md` files run by mdtest.

# Repository Guidelines

This repository is for `mdtest`, a test runner that uses AI agents as interpreters for Markdown test cases. The project dogfoods itself — mdtest's own integration tests are `.test.md` files run by mdtest.

## Workflow

**Order of operations**

1. **Spec first**: update or add spec before changing production code.
2. **Tests next**: add/adjust `.test.md` files that encode the spec.
3. **Code last**: implement the spec fully.

**Drift rule**

- If implementation or tests reveal a missing/incorrect requirement, **update the spec first**.
- If code behavior changes, **spec must reflect it** before merge.

## Directory Structure

- `/docs/design/` — Design-level specs. Numbered starting from `001`. Describes what the system should do, in terms that anyone technical can follow without reading the codebase. Covers behavior, user-facing concepts, and architectural decisions.
- `/docs/implementation/` — Implementation-level specs. Numbered starting from `001`. Written for programmers and coding agents. Describes interfaces, constraints, invariants, and structural decisions. Should be declarative and durable — express intent and rationale, not one-time plans tied to specific lines of code.
- `/tests/` — Test cases as `.test.md` files. This is what mdtest runs.

**Referencing rules**: Design docs may reference other design docs. Implementation docs may reference design docs and other implementation docs. Design docs must not reference implementation docs — the dependency flows downward only.

**Spec language**: Specs must be definitive and reasonably concise. No "TBD", no "optionally", no unmade decisions. If something is unclear or undecided, the agent must discuss it with the user and resolve it before writing it into a spec. Specs should be short enough that a human can review them in full.

## Implementation Workflow

When implementing an implementation doc, the agent must first propose a phased breakdown to the user. Each phase must result in a coherent, compilable, runnable product (atomic) and be small enough for human review — ideally under 200 lines of code changed, excluding lockfiles and generated files. The agent checks back with the user after completing each phase before proceeding to the next.

## Tests

Test files end with `.test.md`. They are plain Markdown with optional YAML front matter for execution metadata:

```yaml
---
requires: [mcp:browser]
side-effects: true
---
```

No front matter means no requirements, runs everywhere.

A bootstrap layer of Go unit tests covers core runner logic (front matter parsing, capability matching, prompt construction). Integration and end-to-end flows are `.test.md` files.

## Misc Rules

- Commit messages follow `type(scope): summary`.
- Always use package manager commands to add or update dependencies. Never manually edit dependency files. Don't pin versions unless explicitly asked.
- When making changes to docs or code, refactor and restructure as needed. The result should read as if it was written that way from the start. Don't pile up patches or accumulate special cases.

---
> Source: [PeronGH/mdtest-cli](https://github.com/PeronGH/mdtest-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
