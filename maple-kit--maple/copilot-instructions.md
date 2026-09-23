## maple

> Maple is an open-source, vendor-agnostic tool for human-to-agent visual review

# Working in this repository

Maple is an open-source, vendor-agnostic tool for human-to-agent visual review
comments on deployed preview environments, with a CI merge gate and an agent
loop over MCP.

Everything below is enforced by lint, CI or a hook. Where a rule is not
automatable, it is here because breaking it is expensive to undo.

## Layout

| Path              | What lives there                                                   |
| ----------------- | ------------------------------------------------------------------ |
| `packages/core`   | Server SDK, overlay, connector contracts, logger. Subpath exports. |
| `packages/cli`    | The `maple` binary.                                                |
| `packages/mcp`    | The MCP server an agent talks to.                                  |
| `docs/`           | Design decisions and the owner checklist.                          |
| `evals/`          | Eval harness and conventions for every AI path.                    |
| `examples/`       | A Next and a Vite application Maple mounts into.                   |
| `tools/`          | Repository-local tooling, including the ESLint plugin.             |
| `.claude/skills/` | Skills that ship with the repository.                              |

## Rules

### Secrets

Never write a credential value into any file, including a test fixture, a
comment or an example. Inject them with `op run --env-file`.

`.env` and `.env.*` are gitignored; only `.env.example` is tracked, and it holds
names, never values. gitleaks runs as a pre-commit hook and as a CI job.

### Dependencies

Adding one is a decision, not a convenience.

- If fewer than about five functions of a library are needed, port them into
  `packages/core/src/lib/` with attribution and unit tests. That directory's
  README records what each file replaces and why.
- Before adding a package, check its stars, its last release, whether it is
  maintained, and whether its licence is compatible with Apache-2.0. **Record
  that reasoning in the commit body**, not in the pull request description,
  where it will not survive the squash.
- `.npmrc` pins exact versions, refuses install scripts and enforces engines —
  except against a Node so old that pnpm cannot start on it, which is a
  different error with the same cause. CONTRIBUTING.md names it.
  `pnpm-workspace.yaml` refuses anything published in the last three days. A
  package needing an install script goes in `onlyBuiltDependencies` with a
  reason.

### Breaking changes

Every package here is 0.x, and 0.x makes no compatibility promise. **Break a
public interface when breaking it is the right shape**: rename the export,
change the signature, move it to another entrypoint. Update every call site,
every test and every example in the same commit, and say what broke in the
changeset.

Do not write a deprecation alias, a compatibility shim or a re-export that
keeps an old name alive. There is nothing downstream to keep alive yet, and a
shim is dead code that has to be read, tested and eventually deleted anyway.

This stands until the packages ship 1.0, at which point this section is
replaced rather than edited.

### Effect

Effect v3 is used **inside** `packages/core`. It must not appear in any public
entrypoint: the contract a connector author implements is plain Promises and
structural types.

`no-restricted-imports` enforces this on `src/index.ts` and `src/*/index.ts`.
The rule is a backstop, not the design. The design is that everything crossing a
public boundary returns a Promise and throws a plain `Error` subclass — see how
`src/internal/effect/store.ts` unwraps an exit rather than letting
`Effect.runPromise` reject with a `FiberFailure`.

### Connectors

Capabilities are optional methods, detected by presence. Never add a parallel
field declaring what a connector supports; two sources of truth will disagree.

Every connector runs the shared contract suite in
`packages/core/src/testing/`. Use the `contribute-connector` skill.

### Logging

`createLogger({ sinks })` from `@maple-kit/core/logger`. Raw `console.*` is a
lint error everywhere except `src/logger/sinks/console.ts`, which is the one
sanctioned exit and says so at the top of the file.

The overlay logs through the same logger, so a noisy warning never lands in a
host application's monitoring.

### Comments

`maple/max-comment-lines` allows four lines per comment block, ten for a file
header, and exempts JSDoc on an exported symbol. Longer explanations belong in
`docs/`, where they are read and revised.

### Complexity

Errors, not warnings: cyclomatic ≤ 20, cognitive ≤ 15, nesting ≤ 4, parameters
≤ 4, 150 lines per function.

Suppressions live in `eslint-suppressions.json`, which is frozen. CI runs
`--prune-suppressions` and fails on a diff, so an entry that is no longer needed
cannot quietly stay.

### Tests

- Pure logic: plain vitest, table-driven where the cases are parallel.
- Components and hooks: vitest browser mode with `vitest-browser-react`, in real
  Chromium. Anything depending on constructed stylesheets, shadow DOM or CSS
  anchor positioning must be tested there, not in a DOM emulation.
- Every network call is mocked with msw, in `test/msw/`, including at least one
  error response.
- Every AI path ships with evals and a CI-enforced threshold. A threshold is
  raised when the code improves and never lowered to make a build green.

### Commits

Conventional commits, one concern per commit, and the message explains **why**.
Every commit is signed off (`git commit -s`); there is no CLA.

Write every commit as though the repository were already public, because it will
be, and the history is not rewritten before that happens. That includes the
author address: this repository requires a local `user.email` so the identity is
chosen rather than inherited from a global config.

`pnpm-lock.yaml` should contain no URLs. A registry mirror can write its own
tarball URLs into it, and they resolve on exactly one machine; a hook and a CI
job both check.

## Before opening a pull request

```
nvm use
pnpm lint && pnpm typecheck && pnpm test && pnpm build
```

`nvm use` first: on the wrong Node, pnpm fails with
`ERR_UNKNOWN_BUILTIN_MODULE: No such built-in module: node:sqlite`, which says
nothing about the version and is entirely about the version.

If the hooks are not running, `pnpm hooks`. They are not installed by
`pnpm install`, because `ignore-scripts=true` stops every install-time script,
this repository's own included.

Add a changeset when a published package changes.

---
> Source: [maple-kit/maple](https://github.com/maple-kit/maple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
