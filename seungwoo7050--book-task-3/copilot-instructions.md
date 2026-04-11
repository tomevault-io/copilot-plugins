## book-task-3

> This workspace is a study-first archive. It is not a single deployable product, and it is not

# AGENTS.md

## Mission

This workspace is a study-first archive. It is not a single deployable product, and it is not
an answer book.

When you work here, optimize for:

- runnable learning artifacts
- explicit reasoning and provenance
- stable navigation across many tracks
- curriculum quality, not just code quantity

## Workspace Reality

The root of `/Users/woopinbell/work/book-task-3` is a multi-track workspace, not a single
repository with one universal layout.

Top-level directories currently include separate study repositories or track groups such as:

- `cs-core`
- `algorithm`
- `network-atda`
- `database-systems`
- `backend-fastapi`
- `backend-spring`
- `bithumb`
- `front-react`
- `mobile`
- `cpp-server`
- `infobank`
- `guides`

Do not assume a root-level `legacy/` plus `study/` layout for the whole workspace.
Work at the target sub-repository level and follow that sub-repository's current conventions.

## Source Of Truth

This file is a workspace-level guide. It is intentionally lower priority than the actual
structure and documentation already established in mature directories.

When this file conflicts with the current shape of a completed area, prefer the completed area.

Good directories to compare before inventing new structure:

- `cs-core`
- `algorithm`
- `network-atda`
- `database-systems`
- `backend-fastapi`
- `front-react`

In practice, the nearest mature sibling project is a better template than an old generic rule.

## Common Current Patterns

There is no single required project shape for the whole workspace. Several patterns are already
valid and should be preserved where they fit.

| Pattern | Typical location | Current shape |
| --- | --- | --- |
| Track under `study/` | `cs-core`, `algorithm`, `network-atda`, `front-react`, `mobile` | `study/<track>/<project>/...` |
| Lab plus capstone repo | `backend-fastapi`, `backend-spring` | `labs/<lab>/...` and `capstone/<project>/...` |
| Language-split curriculum repo | `database-systems` | `go/...` and `python/...` with shared `docs/` |
| Numbered security/project sequence | `bithumb`, parts of `infobank` | ordered project directories plus roadmap docs |

Mirror the local pattern that already exists in the target area. Do not force one repo's layout
onto another repo just because both are study-oriented.

## Public Project Surface

Even though shapes differ, every good project here should still make these boundaries obvious:

- what the problem is
- what material was provided
- what implementation or analysis is user-authored
- how to build, run, and test it
- what was learned
- what is current, archived, or intentionally omitted

At minimum, preserve or create the nearest equivalent of:

- project-level `README.md`
- `problem/README.md` when there is an explicit problem statement or fixture set
- one or more implementation or analysis directories such as `python/`, `go/`, `c/`, `cpp/`,
  `react/`, `spring/`, `fastapi/`, or `analysis/`
- `docs/README.md` for durable tracked notes

## Language And Documentation Tone

Across the mature repositories in this workspace, the current default is:

- write explanatory prose in Korean
- keep commands, file paths, tool names, protocol names, code identifiers, and other technical
  tokens in their original English form
- prefer short, index-like public READMEs over long essay-style front matter

When adding or revising docs, match the tone of the target repository first.
Do not translate established technical identifiers just to make the prose more uniform.

## Documentation Depth By Layer

Keep the current layer separation clear:

- `README.md`: fast public index, scope, status, reading order, verification entrypoint
- `problem/README.md`: problem framing, provided material, constraints, canonical validation
- `docs/`: durable concept notes, references, curriculum context, and explanation that should
  remain useful after implementation details change
- `notion/`: longer public learning notes, decision rationale, debugging evidence, and rebuild
  guidance

Public READMEs should stay concise. Long explanations belong in `docs/` or `notion/`.
Do not let project READMEs collapse into answer dumps or raw work logs.

## Notion Convention

The old rule that `notion/` must always be local-only is no longer accurate for this workspace.

Current reality:

- many mature repositories in this workspace track `notion/`
- many also keep `notion-archive/` alongside the current `notion/`
- some projects use `05-development-timeline.md` as a reproducible rebuild log

Therefore:

- if a target repository already tracks `notion/`, keep following that repository's convention
- if a target repository already uses `notion-archive/`, preserve the two-tier pattern
- do not "clean this up" into a different policy unless the user explicitly asks for a
  workspace-wide normalization pass

Current common pattern in mature repos:

- `notion/` is the current public learning-note set
- `notion-archive/` preserves older long-form notes, earlier templates, or superseded drafts
- `notion/05-development-timeline.md` is often the primary rebuild and re-verification guide

When creating a new project inside a repository that already uses the expanded note set,
prefer the current neighboring convention:

- `00-problem-framing.md`
- `01-approach-log.md`
- `02-debug-log.md`
- `03-retrospective.md`
- `04-knowledge-index.md`
- `05-development-timeline.md`

The expected depth is not "dump every scratch note." The goal is a compressed, reusable current
edition:

- enough detail that another learner can rerun the build, test, or analysis flow
- enough debugging evidence that failure modes and fixes are understandable
- enough decision history that major tradeoffs are recoverable
- no meaningless chronological spam or copied terminal transcripts without commentary

When adding a new project inside an existing repository, match the neighboring project's
`notion/` pattern.

## Guides Rule

`guides/` is a shared primer and cross-track reference area. It is not limited to migration
rules.

Use `guides/` for:

- shared language, framework, tooling, systems, or security primers
- cross-track preparation material
- cross-track submission or readiness guidance

Do not use `guides/` for:

- repo-specific curriculum audit details that belong in the target repo
- per-project implementation notes
- duplicate copies of README content that already lives next to the project

When new projects introduce concepts that `guides/` does not yet cover:

1. add or extend the relevant guide documents
2. update `guides/README.md`
3. add cross-links from the affected repo README or roadmap where useful

Repo-specific curriculum maps, audits, and roadmaps should usually live in the affected
repository's own `docs/` directory or root README.

## Legacy And Migration Work

Some sub-repositories still contain legacy-style trees or historical source material.

Rules:

- treat any `legacy/` directory inside the target repository as read-only unless the user
  explicitly asks otherwise
- infer the intended curriculum from the current repository docs, roadmaps, and completed
  sibling projects, not from stale notes alone
- before scaling a large migration or expansion, shape one pilot inside the target repository
- do not apply a root-level fixed pilot order across the entire workspace

## Curriculum Design Standard

When you add or reshape study projects, evaluate:

- coverage gaps
- duplicates with low incremental learning value
- projects that are too small to deserve their own slot
- projects that are too broad and should be split
- missing bridge projects between theory-heavy and implementation-heavy areas
- whether the current ordering teaches the material in a defensible sequence

You may add, split, merge, rename, reorder, or replace projects when it improves the learning
program, but the pedagogical reason must be documented in the affected repo's root README,
roadmap, or curriculum docs.

## Evidence Standard

When you keep or write technical claims:

- tie commands to real files in the target repository
- prefer runnable commands over narrative claims
- flag unknowns instead of inventing them
- remove or fix broken links
- do not reference files that do not exist
- preserve current status markers such as `verified`, dates, or scope tables when the target
  repository already uses them

## Working Style

Keep changes scoped to the repository you are actively editing.

Do not:

- mass-normalize unrelated top-level repositories in one sweep
- rewrite neighboring repos just because they use a different valid convention
- modify generated artifacts unless the task requires it
- modify `legacy/` material unless the user asked for it

If the workspace is dirty, treat existing unrelated changes as user-owned unless proven
otherwise.

## Good Study Repository Test

A good study repository should let another reader answer these questions quickly:

1. What was the problem?
2. What constraints mattered?
3. What currently works?
4. How do I rerun the verification?
5. What concept does this project teach?
6. Why is this project in this curriculum at all?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seungwoo7050)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/seungwoo7050)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
