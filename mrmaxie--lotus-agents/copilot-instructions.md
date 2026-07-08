## lotus-agents

> This repo defines a `.local` + `.docs` workflow for human-agent work.

# Lotus Agents - Repository Contract

## What This Repository Is

This repo defines a `.local` + `.docs` workflow for human-agent work.

The current product shape is:

- `README.md` is the main human entrypoint
- `lotus-local/` is the local-first installable skill package root
- `lotus-linear/` is the Linear-backed installable skill package root
- `.codex-plugin/plugin.json` bundles the skill collection for native Codex
  plugin installation

## What This Repository Is Not

Do not treat this repo as a consumer repository by default.

- root `.local/` is disposable private state, not the product contract
- do not keep `.local/examples` fixtures in this repository; remove them during
  cleanup unless a human explicitly asks to preserve a specific local example
- root `.docs/` may not exist
- do not bootstrap consumer `.local/` or `.docs/` in this repo unless a human
  explicitly asks

## Working Rules In This Repo

When changing the contract or adoption story:

1. keep `README.md` aligned with the relevant skills under `lotus-local/` and
   `lotus-linear/`
2. keep the model centered on `.local/AGENTS.md` and `.docs/AGENTS.md`
3. keep `.local/` private-first and `.docs/` required in the workflow but
   optional to commit
4. do not reintroduce `AGENTS_TO_COPY.md`, `AGENTS_ISSUE_FLOW.md`, `docs/`,
   `.local/context.md`, `.local/questions/`, or `.local/runs/`
5. do not reintroduce setup scripts, path configuration, or custom layout
   support unless a human explicitly asks
6. keep skills optional, task-focused, and copy-paste friendly
7. keep reusable templates in the relevant skill assets or in consumer
   `.docs/templates/`, not in one legacy copy artifact
8. never use live `.local/` session state, auth files, logs, databases, plugin
   caches, or sandbox files as examples

## Code Style

- Name files in camelCase by default.
- Name files that define classes or components in PascalCase.
- Prefer named exports; use default exports only when a tool or framework
  requires them.
- Prefer TypeScript `type` aliases over `interface` declarations unless an
  interface is required.
- Do not use TypeScript `function` declarations; use arrow function
  expressions. Use `function` only when a tool, framework, overload, generator,
  `this` binding, or hoisting requirement makes it necessary.
- Define reusable enum-like TypeScript value sets as TypeScript string enums
  and validate them through Zod schemas built from `enumValues(...)`, instead
  of duplicating string literal unions or inline `z.enum([...])` arrays.
- Parse enum-like values that cross CLI, file, prompt, config, or other
  untrusted runtime boundaries with Zod before using them.
- Use single quotes for strings outside JSX. Use double quotes for JSX and HTML
  attributes unless that is not practical.
- Use trailing commas wherever the syntax allows them to reduce future diffs.
- Keep line width at 140 columns, not 80.
- Always use semicolons.
- Use Bun for local dependency installation, lockfile updates, and package
  verification commands when working inside this repository. Keep `bun.lock` as
  the repository lockfile.
- Run Biome through the package scripts when editing JavaScript, TypeScript,
  JSON, or JSONC files.
- After local package verification, use `bun run clean` to remove generated
  package artifacts. Do not use it as a dependency reset, and do not remove
  `node_modules/` or private `.local/` state unless a human explicitly asks.

## Write For The Actual Reader

Keep each artifact scoped to the reader that will actually use it:

- `README.md` is for humans adopting or understanding the repo
- `AGENTS.md`, `lotus-local/`, and `lotus-linear/` are for agents executing
  work
- keep install and adoption guidance in `README.md`, not in machine-facing
  workflow files
- keep machine-facing files operational; avoid self-descriptions and historical
  narration
- if a file is meant to be executed or followed by an agent, write only the
  rules and expectations needed at execution time

## What To Update Together

If you change naming, paths, or adoption flow, review at least:

- `README.md`
- `.codex-plugin/plugin.json`
- `lotus-local/`
- `lotus-linear/`
- any affected skill directories under those package roots

## Branch And Pull Request Hygiene

- Branch names for implementation work must use exactly one of these patterns:
  `feat/<taskid>`, `fix/<taskid>`, `chore/<taskid>`, or
  `epic/<epictaskid>`
- `<taskid>` and `<epictaskid>` must be the lowercase Linear issue identifier,
  for example `feat/max-77`, `chore/max-82`, or `epic/max-72`
- reject other branch formats, including Linear-generated or personal-prefix
  names such as `maxie/max-77-...`, descriptive slug branches, and
  `feature/<taskid>`
- pull request titles must use
  `<TASKID>: <short Linear-aligned description>`, for example
  `MAX-77: Implement repair and forced reinstall guidance`
- use `.github/PULL_REQUEST_TEMPLATE.md` when opening GitHub pull requests and
  fill every applicable section
- write PR titles, descriptions, implementation notes, verification notes, and
  risks in English
- open implementation PRs ready for review by default; use draft PRs only when
  the human explicitly asks for a draft
- leave Linear issues, GitHub pull requests, and related workflow tasks
  unassigned unless the human explicitly asks for assignment or the next step is
  a clearly manual human action such as PR review, manual verification, or
  approval
- do not automatically assign the repository owner, current user, or any other
  person as a convenience default
- keep Linear current by updating issue status, progress comments, and PR links
  when the work state changes materially

## Linear Issue Implementation Workflow

Use this workflow when a human asks you to implement a Linear issue through
GitHub PRs. It is for implementation work, not review-only or test-only tasks.

1. Read the target Linear issue, parent issue or epic, existing PR links, and
   relevant comments before changing code.
2. Start from the current default branch. Fast-forward it from origin before
   creating implementation branches.
3. Create the epic integration branch from the default branch when the issue
   belongs to an epic. If an immediate epic PR is required and the branch has no
   code changes yet, use one empty conventional commit that opens the branch.
4. Open the epic PR as a regular, non-draft PR into the default branch and link it to the epic
   issue in Linear.
5. Create the implementation branch from the epic branch when the issue belongs
   to an epic; otherwise create it from the default branch. Name it with the
   exact branch patterns above.
6. Move the Linear issue to the active implementation status before coding.
7. Before committing, inspect the worktree with
   `git status --short --branch --untracked-files=all`, stage only intended
   files, and exclude test leftovers, generated scratch files, logs, traces, and
   debug artifacts.
8. Commit small, reviewable, coherent work ranges with English conventional
   commit messages without scopes, and push after each meaningful range.
9. Keep repository docs and agent-facing rules aligned with code changes when
   the change affects adoption flow, package usage, or workflow contracts.
10. Run the relevant checks locally before opening the implementation PR.
11. Open the implementation PR into the epic branch when one exists; otherwise
    open it into the default branch. Use the repository pull request template,
    the required PR title format, no automatic assignees, and ready-for-review
    status unless the human explicitly asked for a draft.
12. Link the PR to the Linear issue, add a concise implementation summary, and
    move the issue to the review status.

## Source Of Truth For This Repo

When working in this repository, use this order:

1. explicit human instruction in the current run
2. `.local/AGENTS.md` when present
3. this `AGENTS.md`
4. `README.md`
5. the current repository state and diff

## Restrictions

- do not fabricate missing root consumer structure unless a human asks
- do not add back removed legacy docs just for explanation
- do not turn optional skills into required repository contents
- do not create automation, CI, or packaging flow unless requested

---
> Source: [MrMaxie/lotus-agents](https://github.com/MrMaxie/lotus-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
