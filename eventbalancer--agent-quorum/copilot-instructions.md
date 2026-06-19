## agent-quorum

> `agent-quorum` is a standalone TypeScript CLI and library for the iterative

# agent-quorum — Agent Operating Rules

`agent-quorum` is a standalone TypeScript CLI and library for the iterative
plan -> critique -> update loop across Codex, Claude Code, and Cursor Agent
CLIs.

Global development conventions live in
[`docs/development/conventions.md`](docs/development/conventions.md). This file
keeps the operating rules that agents should load before changing this
repository.

## 1. Hard Constraints

Override all other project-level guidance.

- **English everywhere in committed artifacts.** Code, comments, commits, docs,
  tests, config, schemas, and prompts are written in English. User-facing
  conversation can be Russian when the user uses Russian.
- **Resolved paths in conversation.** When reporting paths to the user, use
  resolved absolute paths for this machine. Keep portable forms such as
  repo-relative paths and `$HOME` for committed files and reusable snippets.
- **Never commit or push without explicit user instruction.**
- **No destructive git or shell operations.** Do not run force pushes,
  `git reset --hard`, broad `git restore`, `git clean`, `rm -rf`, `sudo`, or
  permission-recursive commands unless the user explicitly asks and the risk is
  clear.
- **Use repo entry points.** Prefer `pnpm run <script>` and `pnpm exec <bin>`;
  never use `npx`.
- **Do not hand-edit generated artifacts.** `dist/`, `coverage/`, lockfiles, and
  package-manager output are generated through the project commands.
- **Preserve the public API.** Keep `src/index.ts`, `package.json` exports, and
  the `agent-quorum` bin stable unless a breaking change is explicit.
- **Respect architecture boundaries.** Provider calls go through
  `providerRun`; `core/` does not spawn provider CLIs directly, and lower layers
  do not import higher layers.
- **Treat role skills and schemas as contracts.** Files under `skills/` define
  provider I/O behavior. Changing a schema or role prompt changes the runtime
  contract and needs matching tests or documentation.
- **Source comments are exceptional.** Prefer names, types, tests, and structure
  over comments. Add comments only for critical non-obvious invariants, specific
  external bugs or provider quirks, or behavior that names and types cannot
  express. Do not restate what the code already says; describe the current task,
  branch, or PR; leave TODO/FIXME/HACK breadcrumbs; duplicate a function
  signature; or preserve commented-out code. If a block needs a comment to
  explain what it does, extract a named helper or boolean instead. Public API
  docblocks may explain invariants, units, failure modes, and external contracts;
  do not write parameter-by-parameter docblocks that repeat the type signature.
- **Dogfood through the real CLI.** For changes that should be designed by
  `agent-quorum` itself, drive the loop with `pnpm run plan:self -- --prompt …`
  (the `agent-quorum` bin from source, no build required); see
  [`examples/`](examples/).
- **Isolate sessions in worktrees.** Run nontrivial, multi-file, or potentially
  concurrent work in a session worktree
  (`pnpm run worktree:create <slug> --desc <text>`), not the shared checkout;
  integrate to `main` via `/ship` plus an explicit step. See
  [Session Worktrees](docs/development/conventions.md#session-worktrees).
- **No orphan background shells.** Do not leave long-running shell sessions or
  detached commands alive after moving on.

## 2. Sources of Truth

When facts conflict, trust in this order:

1. This file, mirrored by the `AGENTS.md` symlink.
2. [`docs/development/conventions.md`](docs/development/conventions.md) for
   code, git, verification, and style rules.
3. `eslint.config.ts`, `tsconfig.json`, and `vitest.config.ts` for enforced
   tooling behavior.
4. `package.json` for scripts, package exports, bin entries, engines, and direct
   dependencies.
5. [`docs/architecture.md`](docs/architecture.md),
   [`docs/configuration.md`](docs/configuration.md), [`docs/cli.md`](docs/cli.md),
   and [`docs/api.md`](docs/api.md) for runtime contracts.
6. `src/core/defaults.ts` (built-in orchestration defaults, mirrored by
   `config.example.json`) and `skills/**/*.schema.json` for default
   orchestration and role I/O contracts.

## 3. Required Entry Points

| Task                       | Use                                                                                                                        |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Install dependencies       | `pnpm install --frozen-lockfile`                                                                                           |
| Build                      | `pnpm run build`                                                                                                           |
| Typecheck                  | `pnpm run typecheck`                                                                                                       |
| Lint                       | `pnpm run lint`                                                                                                            |
| Format check               | `pnpm run format-check`                                                                                                    |
| Tests                      | `pnpm run test`                                                                                                            |
| Full verification          | `pnpm run check`                                                                                                           |
| Local CLI                  | `pnpm run dev -- <args>`                                                                                                   |
| Public API smoke           | `pnpm run build` then import from `agent-quorum`                                                                           |
| Self-planning dogfood      | `pnpm run plan:self -- --prompt <prompt.md>`                                                                               |
| Repo-local binaries        | `pnpm exec <bin>`                                                                                                          |
| Start a session worktree   | `pnpm run worktree:create <slug> --desc <text>`                                                                            |
| Session worktree lifecycle | `pnpm run worktree:list` / `worktree:touch <id>` / `worktree:done <id>` / `worktree:reopen <id>` / `worktree:release <id>` |

## 4. Git Boundaries

- Keep unrelated user changes intact. Work with them when they affect the task;
  otherwise ignore them.
- Use conventional commit headers only when the user asks for a commit:
  `type(scope): subject`, lowercase, at most 72 characters, no trailing period.
- When a GitHub issue is resolved, add `Closes #<n>` in the commit body. Use
  `Refs #<n>` for related work that does not close the issue.
- Do not stage, commit, push, create branches, or open PRs unless explicitly
  asked.

## 5. Implementation Rules

- Read the surrounding code before editing.
- Keep changes scoped to the requested behavior and existing ownership
  boundaries.
- Add abstractions only when they remove real duplication or match an existing
  local pattern.
- Use named exports and ESM `.js` extensions for relative TypeScript imports.
- Prefer structured parsers and existing helpers over ad hoc string handling.

## 6. Self-Planning Workflow

Use this when the repository should plan its own change before implementation:

```sh
pnpm run plan:self -- --prompt path/to/task.md
```

`plan:self` runs the `agent-quorum` bin from source (no build) and writes plan
artifacts under `.agents/plans/`. Requirements live under
`.agents/requirements/`; prompts live under `.agents/prompts/`. Read the
reported `summary.md` and `plan.final.md`, then implement and verify normally.

Canonical planning chains:

- `/requirements` -> `/solution-handoff` -> `/prompt-architect` -> confirmed
  self-planning run.
- `/solution-handoff` -> `/prompt-architect` -> confirmed self-planning run.
- `/prompt-architect` -> confirmed self-planning run.

Only `/prompt-architect` asks for launch confirmation and starts the
`plan:self` run; earlier steps prepare context and hand it downstream.

## 7. Verification Floor

Before claiming implementation is complete, run the narrowest relevant checks.
For broad or contract-touching changes, `pnpm run check` is the floor. If a
check cannot be run, report why and what risk remains.

## 8. Self-Improvement

If a task exposes a real gap in rules, scripts, docs, tests, or architecture,
report:

1. What happened.
2. What is missing.
3. Proposed fix.

Do not flag one-off situations.

---
> Source: [eventbalancer/agent-quorum](https://github.com/eventbalancer/agent-quorum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
