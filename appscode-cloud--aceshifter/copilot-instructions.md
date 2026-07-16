## aceshifter

> This file provides guidance to coding agents (e.g. Claude Code, claude.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to coding agents (e.g. Claude Code, claude.ai/code) when working with code in this repository.

## Repository purpose

Go module `go.bytebuilders.dev/aceshifter` — a Kubernetes operator that installs and reconciles the AppsCode Container Engine (ACE) "feature set" charts via **FluxCD `HelmRelease`** resources. Drives the full ACE stack (CAPI, CAPA, OCM hub/managed/spoke, Crossplane, OpsCenter Backup, ACE installer, the main `ace` chart) onto a target cluster from a single set of `HelmRelease`s the operator owns.

Closely related to `kubeops.dev/fargocd` (which does a similar job for a smaller catalog).

The produced binary is `aceshifter`.

## Architecture

- `cmd/aceshifter/` — entry point.
- `pkg/cmds/` — Cobra root + run.
- `pkg/controller/`:
  - `helmrelease_controller.go` — main reconciler watching FluxCD `helm.toolkit.fluxcd.io/HelmRelease`.
  - `namespace_controller.go` — namespace-level setup.
  - `suite_test.go` — envtest harness.
- `pkg/featuresets/` — **embedded ACE feature-set definitions** that the controller materializes:
  - `ace.yaml`, `ace-installer.yaml` — top-level descriptors.
  - `capi-core/`, `capi-capa/`, `crossplane/`, `ocm-hub/`, `ocm-mc/`, `ocm-spoke/`, `opscenter-backup/` — per-stack descriptors.
  - `lib.go` — featureset loader.
- `pkg/tracker/tracker.go` — namespace skip-list / featureset tracking.
- `Dockerfile.in` (PROD, distroless), `Dockerfile.dbg` (debian), `Dockerfile.ubi` (Red Hat certified).
- `hack/`, `Makefile` — AppsCode build harness.
- `vendor/` — checked-in deps.

## Common commands

- `make ci` — full CI pipeline.
- `make build` / `make all-build` — host or all-platform build.
- `make fmt`, `make lint`, `make unit-tests` / `make test` — standard.
- `make verify` — codegen + module-tidy verification.
- `make container` / `make push` / `make release` — image build/publish flow.
- `make push-to-kind` / `make deploy-to-kind` — Kind dev loop.

## Conventions

- Module path is `go.bytebuilders.dev/aceshifter` (vanity URL); imports must use that.
- License: `LICENSE.md` (AppsCode). Sign off commits (`git commit -s`); contributions follow the DCO (`DCO`).
- Vendor directory is checked in; keep `go mod tidy && go mod vendor` clean.
- Driven by **FluxCD `HelmRelease`** primitives; don't introduce a parallel ArgoCD or Helm-direct path.
- New featureset: drop a `pkg/featuresets/<name>/` directory and a top-level YAML; register via `lib.go`. Don't hand-write `HelmRelease`s in the controller.
- Three Dockerfiles, one binary — keep `Dockerfile.in`, `Dockerfile.dbg`, and `Dockerfile.ubi` in sync.

---
> Source: [appscode-cloud/aceshifter](https://github.com/appscode-cloud/aceshifter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
