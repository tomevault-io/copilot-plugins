## lintcn

> lintcn uses `remorses/tsgolint` (forked from `oxc-project/tsgolint`).

# lintcn

## tsgolint fork

lintcn uses `remorses/tsgolint` (forked from `oxc-project/tsgolint`).

The fork adds on top of upstream:
1. `internal/runner/runner.go` — new file with `Run(rules, args)` entry point
2. `internal/rule_tester/snapshot.go` — added `TSGOLINT_SNAPSHOT_CWD` env var.
   When `TSGOLINT_SNAPSHOT_CWD=true`, snapshots are stored relative to
   `os.Getwd()` (the test package directory) instead of relative to snapshot.go.
   Default behavior is unchanged — tsgolint's own snapshots stay in
   `internal/rule_tester/__snapshots__/`.

User rules must set `TSGOLINT_SNAPSHOT_CWD=true` when running `go test` so
snapshots land in `.lintcn/<rule>/__snapshots__/` instead of the cached
tsgolint source directory.

When writing Go rule tests, prefer readable multi-line raw strings for TypeScript
snippets instead of dense one-line `Code` strings. Keep the test input shaped
like real source so snapshots and future edits stay easy to read.

After updating Go rule snapshots, always read the snapshot file back to verify
the diagnostic wording and highlighted ranges before finishing the task.

The `prefer-object-params` warning is meant for reusable definitions only:
function declarations, named/assigned lambdas, methods, and constructors with 3+
positional parameters should prefer a single object parameter, but inline
callbacks passed directly at the use site are allowed.

User rules import from `internal/rule`, `internal/utils` etc. — same paths
as tsgolint's own code. The Go workspace allows this because the user module
name is a child path: `github.com/typescript-eslint/tsgolint/lintcn-rules`.

## updating tsgolint

Two constants in `src/cache.ts`:

- `DEFAULT_TSGOLINT_VERSION` — commit hash from `remorses/tsgolint`
- `TYPESCRIPT_GO_COMMIT` — base commit from `microsoft/typescript-go`
  (before patches). Changes only when upstream updates its submodule.

To sync: merge upstream into fork, push, update both constants, clear
`~/.cache/lintcn`, rebuild, test.

typescript-go is managed by tsgolint — never fork it independently.

---
> Source: [remorses/lintcn](https://github.com/remorses/lintcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
