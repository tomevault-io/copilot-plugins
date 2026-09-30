## hivedispatch

> This file is the canonical instruction set for any coding agent working in this repository (Claude Code reads it through `CLAUDE.md`). It points at the conventions already written down for humans rather than restating them; when the two disagree, [`CONTRIBUTING.md`](CONTRIBUTING.md) wins and this file has a bug.

# Instructions for coding agents

This file is the canonical instruction set for any coding agent working in this repository (Claude Code reads it through `CLAUDE.md`). It points at the conventions already written down for humans rather than restating them; when the two disagree, [`CONTRIBUTING.md`](CONTRIBUTING.md) wins and this file has a bug.

## What this is

HiveDispatch turns tickets into pull requests: it polls an issue tracker, triages and claims a ticket, branches, runs a coding-agent CLI in an isolated worktree, commits, opens a PR, and reports back. Read [`README.md`](README.md) for the flow, [`docs/design-spec.md`](docs/design-spec.md) for the architecture, and [`docs/decisions.md`](docs/decisions.md) for every choice made so far and the alternative rejected.

- Go 1.27, stdlib only plus `gopkg.in/yaml.v3`. Do not add dependencies.
- `cmd/hivedispatch` is the CLI; everything else lives under `internal/`.
- Every external system (tracker, git host, executor, model) sits behind an interface in `internal/` with a fake next to it. Unit tests never touch the network.

## Dev setup

See [`CONTRIBUTING.md` — Dev setup](CONTRIBUTING.md#dev-setup): Go 1.27+ and `golangci-lint` v2 on `PATH`. If `golangci-lint` is not installed, run it without installing:

```sh
go run github.com/golangci/golangci-lint/v2/cmd/golangci-lint@latest run ./...
```

## Required before every commit and before every push

This is the protocol from [`CONTRIBUTING.md` — Before every commit](CONTRIBUTING.md#before-every-commit); CI runs the same three checks (`.github/workflows/ci.yml`), and a PR that fails any of them is not done.

1. **Vet, test, and lint the packages you touched** before each commit. Keep the loop fast by scoping to what changed:

   ```sh
   go vet ./internal/pkg/... && go test -race ./internal/pkg/... && golangci-lint run ./internal/pkg/...
   ```

   Note `go test -race` runs the tests with the race detector, and `golangci-lint` covers formatting too (`gofmt`, `goimports`), so unformatted code fails lint.

2. **Run the full set across the whole module** before pushing, and again before declaring the task done:

   ```sh
   go vet ./... && go test -race ./... && golangci-lint run ./...
   ```

3. **Do not commit or push with a failure.** Fix the code, not the check: do not skip tests, add `//nolint` without a reason, or loosen `.golangci.yml` to get green.

If an agent cannot run these (for example, `golangci-lint` is missing and `go run` fails offline), say so explicitly in the final report rather than claiming they passed.

## Rules

Carried over from [`CONTRIBUTING.md` — Rules](CONTRIBUTING.md#rules):

- **Fakes, not mocks of the network.** Any new external dependency gets an interface in `internal/` and a fake implementation; tests use the fake.
- **Tests first.** Write or extend the test for the behaviour you are changing before the implementation. A task is not done until `go test -race ./...` passes.
- **Errors are never discarded.** `errcheck` is on, including for `io.WriteString` and `Close`. If an error is truly irrelevant, write `_ = f()` so the choice is visible.
- **Decisions are append-only.** Record design decisions in `docs/decisions.md` with the alternative rejected and why. To change a decision, add a new entry that supersedes the old one; do not edit history.
- **Conventional commit messages:** `feat:`, `fix:`, `test:`, `docs:`, `chore:`, optionally scoped like `fix(ghissues): …`. Commit as you go with messages that describe the change, not the ticket number.
- **Never merge.** Agents branch and open PRs; a human merges.

---
> Source: [thomasmeadows/HiveDispatch](https://github.com/thomasmeadows/HiveDispatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
