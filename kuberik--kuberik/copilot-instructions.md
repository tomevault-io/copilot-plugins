## kuberik

> Guidance for AI coding assistants working in `kuberik/kuberik`. Read this before making changes.

# AGENTS.md

Guidance for AI coding assistants working in `kuberik/kuberik`. Read this before making changes.

## What this repo is

`kuberik/kuberik` is the central hub for the Kuberik project. It contains:

1. The `kuberik` CLI - a Cobra-based binary that wraps common `kubectl` operations for installing and operating Kuberik.
2. The bundled install manifest (`config/install/kustomization.yaml`) that pins compatible versions of each Kuberik controller.
3. The lighthouse README, documentation, examples, and RFC process.
4. The GitHub Action (`action/`) that installs the CLI on GitHub runners.

It is **not** a controller repository. Controllers live in their own repos under the `kuberik/` GitHub org (`rollout-controller`, `datadog-controller`, etc.). Code changes that affect the controllers themselves should be opened against those repos, not this one.

## Contribution rules

- **Commit subject:** imperative mood, capitalized, no trailing period, ≤72 characters. Body wrapped at 72 columns, explaining what and why. Do not put `@mentions` or `#123` in the commit subject - put those in the PR description.
- **One concern per commit.** A bug fix and an unrelated refactor go in separate commits.
- **Disclose AI assistance** with a `Co-Authored-By` trailer in commits you make.
- **No marketing prose.** Drop adjectives like "comprehensive", "robust", "powerful" from docs and commit messages.
- **Trim the diff:** do not "tidy up" unrelated code as part of a feature change. Drive-by formatting changes belong in their own PR.

## Code quality

Before submitting:

- `go build ./...` and `go test ./...` must pass.
- `go vet ./...` must be clean.
- `go mod tidy` must not produce a diff.
- New CLI commands need at least a smoke test under `cmd/kuberik/*_test.go`.

CLI conventions:

- **Output discipline.** Human-facing status goes to stderr; data the user might pipe goes to stdout. The CLI shells out to `kubectl` for cluster ops - inherit kubectl's stdout/stderr behavior, do not buffer.
- **No silent failures.** Wrap errors with `%w` so callers can inspect; return actionable error messages.
- **Respect global flags.** Every command that touches the cluster must honor `--kubeconfig` and `-n/--namespace` (or `--all-namespaces` where applicable).

Documentation conventions:

- README and `docs/*.md` are read by people choosing whether to adopt Kuberik. Lead with the user problem, not the implementation. Show working YAML, not pseudocode.
- Link to deeper docs rather than expanding the README. The README is a poster, not a manual.

## Where things live

- `cmd/kuberik/` - CLI source, single `main` package, one file per command group.
- `action/` - the composite GitHub Action that downloads and installs the CLI.
- `config/install/` - kustomize bundle of pinned controller releases. Bumping a controller version is a one-line change here.
- `docs/` - user-facing documentation (Markdown).
- `examples/` - copy-pasteable manifests by use case. Keep these minimal and runnable.
- `rfcs/` - design proposals; see [rfcs/README.md](rfcs/README.md).
- `.github/` - issue templates, labels, workflows (CI, release, scorecard, codeql, stale, sync-labels).

## What is _not_ in scope here

- Controller source code (lives in per-controller repos).
- Per-controller release notes (those repos own their own changelogs).
- Helm chart implementation (planned, but not yet in this repo).

---
> Source: [kuberik/kuberik](https://github.com/kuberik/kuberik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
