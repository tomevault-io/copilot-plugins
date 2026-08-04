## llmkube

> LLMKube is a Kubernetes operator that deploys and manages local LLM inference

# AGENTS.md

LLMKube is a Kubernetes operator that deploys and manages local LLM inference
workloads. Go 1.25, built with controller-runtime / Kubebuilder. This file tells
coding agents how to work in this repo. Humans: see `CONTRIBUTING.md`.

## Setup

- Go 1.25+ (pinned in `go.mod`).
- `make test` downloads envtest binaries on first run; no cluster needed for unit tests.
- `make test-e2e` requires Docker and Kind.

## Commands

| Task                              | Command          |
|-----------------------------------|------------------|
| Build the controller              | `make build`     |
| Build the `llmkube` CLI           | `make build-cli` |
| Unit tests (envtest)              | `make test`      |
| E2E tests (needs Kind)            | `make test-e2e`  |
| Format                            | `make fmt`       |
| Vet                               | `make vet`       |
| Lint (golangci-lint)              | `make lint`      |
| Regenerate CRDs / RBAC / webhooks | `make manifests` |
| Regenerate DeepCopy methods       | `make generate`  |

## Before you commit

A change is not done until all of these pass:

1. `make fmt`
2. `make vet`
3. `make lint`
4. `make test`

## Before pushing a PR

Run the cross-arch lint pass:

```sh
make lint-all
```

This catches `//go:build`-tagged files that the host's default `GOOS` would
silently skip. About 2× slower than `make lint`; intentional opt-in so the
inner loop stays fast.

If you edited CRD types in `api/v1alpha1/`, also run:

5. `make generate` — DeepCopy methods
6. `make chart-crds` — regenerates CRDs and syncs them into the Helm chart
7. `git status` must be clean afterward. Uncommitted generated files mean a
   step was skipped, and the CI CRD sync check will fail.

## Code style

- Match the surrounding code: naming, error handling, comment density, layout.
  Do not introduce a different style for new code.
- Idiomatic Go: `gofmt`, wrapped errors (`fmt.Errorf("...: %w", err)`), table-driven tests.
- Comments explain *why*, not *what*. Do not add docstrings or decorative
  comments to hit a quota.
- Controller code follows controller-runtime conventions: reconcilers are
  idempotent, garbage collection uses owner references, state is surfaced
  through status conditions.

## Testing

- Every behavior change needs a test. A bug fix gets a regression test that
  fails before the fix and passes after.
- Unit tests use `envtest` (controller-runtime); they run with `make test` and
  need no live cluster.
- Assert observable behavior, not internal implementation detail.
- Do not weaken or delete a test to make a change pass. If a test is genuinely
  wrong, fix it and say why.

## Commits

This repo uses conventional commit prefixes so `release-please` can generate
changelogs and version bumps. Every commit needs one:

| Prefix      | Use for                                   | Version bump |
|-------------|-------------------------------------------|--------------|
| `feat:`     | New feature, CRD field, CLI command       | minor        |
| `fix:`      | Bug fix, correctness improvement          | patch        |
| `perf:`     | Performance improvement                   | patch        |
| `docs:`     | Documentation only                        | patch        |
| `chore:`    | Deps, CI, tooling (hidden from changelog) | none         |
| `test:`     | Test-only change (hidden)                 | none         |
| `refactor:` | Refactor, no behavior change (hidden)     | none         |

Use `feat!:` / `fix!:` for breaking changes.

- Sign off every commit: `git commit -s`. A human is accountable for every
  commit, however it was produced (DCO is enforced by CI; bot-only sign-offs
  are not accepted).
- Subject says *what* changed; body says *why*. No implementation play-by-play.
- Keep commit messages free of attribution trailers (`Co-Authored-By`,
  `AI-Agent`, `Assisted-by`, etc.). Disclose AI assistance in the PR
  description instead, per [CONTRIBUTING.md](CONTRIBUTING.md) ("AI-Assisted and
  Agent Contributions").
- One logical change per commit.

## Branches and pull requests

- Contributions go through fork-based PRs. Branch from an up-to-date `main`.
- Branch names: `feat/<slug>`, `fix/<slug>`, `chore/<slug>`.
- PRs follow `.github/PULL_REQUEST_TEMPLATE.md` (What / Why / How / Checklist)
  and reference the issue with `Fixes #N`.
- Do not push to `main`. Do not force-push shared branches.

## Project layout

| Path                   | Contents                                        |
|------------------------|-------------------------------------------------|
| `cmd/`                 | Entry points (controller, metal-agent)          |
| `internal/controller/` | Model + InferenceService reconcilers            |
| `internal/metrics/`    | Prometheus metrics                              |
| `pkg/cli/`             | Cobra CLI commands                              |
| `api/v1alpha1/`        | CRD type definitions (regenerate after editing) |
| `charts/llmkube/`      | Helm chart (CRDs synced via `make chart-crds`)  |
| `config/`              | Kustomize bases                                 |

The two CRDs are **Model** (a model spec) and **InferenceService** (a
deployment config).

## Runtime-arg parity

Any new `InferenceServiceSpec` field that changes the llama-server command
line must be wired into **all three** of these places, with a test for each:

1. `LlamaCppBackend.BuildArgs` —
   `internal/controller/runtime_llamacpp.go` (and the helper in
   `runtime_llamacpp_args.go`). This is the controller-side arg builder that
   renders the args for the in-cluster container.
2. The metal-agent path:
   `MetalAgent.ensureProcess` → `buildExecutorConfig` (in
   `pkg/agent/agent.go`) → `MetalExecutor.StartProcess` /
   `buildLlamaServerArgs` (in `pkg/agent/executor.go`). The metal-agent is
   the out-of-cluster sibling that runs llama-server natively on Apple
   Silicon hosts; its flags must match the controller's.
3. `computeSpecHash` (in `pkg/agent/agent.go`) — any field that changes the
   command line must also be folded into the spec hash so a change triggers
   a respawn instead of a no-op.

When adding a new field, mirror the existing `resolveCacheTypes` comment in
`pkg/agent/agent.go`: add a one-line note in the new arg-builder site
explaining that it mirrors the other side, and add a single table-driven
Go test that feeds one representative `InferenceService` spec (with the
runtime-affecting fields set) through **both** `LlamaCppBackend.BuildArgs`
and the metal-agent's `buildLlamaServerArgs`, asserting the same flags
appear on both sides for each field. This is the parity guard — it
catches drift the moment someone wires a field into one side and forgets
the other.

Do **not** refactor the two arg builders into one shared function; they
serve different runtimes and have different defaults. The parity test is
the contract, not shared code.

## Do not

- Do not hand-edit generated files (`zz_generated.*`, CRD YAML under `config/`
  and `charts/`). Change the source and regenerate.
- Do not commit secrets, kubeconfigs, or `.env` files.
- Do not skip the pre-commit checks or the CRD sync step.
- Do not run an init/main container that performs a privileged syscall (`chown`,
  `chmod` of an unowned file, `mount`, etc.) non-root with `capabilities.add`
  when the syscall runs in an exec'd child (`command: ["sh", "-c", "chown ..."]`).
  A non-root process loses its capabilities across `execve` unless the runtime
  sets ambient capabilities, which is runtime-dependent (it works on kind but
  not on every cluster, so envtest and CI will pass while real users hit
  `EPERM`). Either run the container as root (uid 0, still `Drop: [ALL]` + the
  minimal caps + no privilege escalation), or do the syscall directly in the
  container's entrypoint (a small helper binary) so the caps are not cleared.
  This caused the 0.8.20 `model-cache-prep` regression (#887). Any change to an
  init/main container `securityContext`, `command`, `image`, or user that affects
  a privileged syscall must be reasoned about at the runtime level, not just by
  the rendered-spec envtest.

---
> Source: [defilantech/LLMKube](https://github.com/defilantech/LLMKube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
