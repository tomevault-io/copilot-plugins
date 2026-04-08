## barka

> This repo uses k3s (local) and plain kubectl (all envs), with task entrypoints managed by mise.

# Agents Guide

This repo uses k3s (local) and plain kubectl (all envs), with task entrypoints managed by mise.

## Conventions

- Prefer mise tasks over ad-hoc commands.
- Use mise environments (`-E <name>`) to select a kubeconfig/cluster target.
- Keep tasks env-agnostic unless they inherently require local Docker/k3s lifecycle control.
- Use `.yaml` extension for chart template files

## Task Naming

- `kube:*`: environment-agnostic kubectl operations. These run against whatever cluster is selected via mise environment (`KUBECONFIG`).
- `local:*`: local-only tasks (k3s-in-Docker lifecycle).
- `prod:*`: production wrappers around `kube:*` tasks. These should add guardrails like `confirm` prompts.

## Local Environment

- Compose: `dev/k3s/docker-compose.yml`
- Kubeconfig output (generated): `dev/k3s/kubeconfig/kubeconfig.yaml`
- Local mise env config: `mise.local.toml` (loads `.env.local`)
- Local env file (gitignored): `.env.local` (must include `KUBECONFIG=dev/k3s/kubeconfig/kubeconfig.yaml`)

## Production Environment

- Prod mise env config: `mise.prod.toml`
- Provide `KUBECONFIG` via `.env.prod` or your shell environment (recommended: outside this repo)

Infisical (production secrets):

- `kube:tools:apply` will install the Infisical Secrets Operator and create/update the Universal Auth credentials secret.
- Provide `INFISICAL_CLIENT_ID` and `INFISICAL_CLIENT_SECRET` via `.env.prod` or your shell environment before running production tasks.

InfisicalSecret CRDs:

- OpenClaw's `InfisicalSecret` resource is rendered from `charts/openclaw/templates/infisical.yaml` and applied by `kube:tools:apply` in `prod`.

Common commands:

```bash
mise trust
mise install

mise -E local run local:up
mise -E local run kube:kubectl -- get pods -A
mise -E local run kube:headlamp:token

mise -E local run local:down
mise -E local run local:reset
```

## Tools Manifests

- Tools are installed by `kube:tools:apply`.
- Headlamp is managed by an in-repo Helm chart at `charts/headlamp/`.
- OpenClaw Operator is installed by `kube:tools:apply` (Helm chart from GHCR OCI).
- Avoid generating or storing credentials in the repo. Tokens are created on-demand via `kube:headlamp:token`.

## Apps

- Apps are applied by `kube:apps:apply`.
- OpenClaw instance is managed by the in-repo Helm chart at `charts/openclaw/`.

## Safety

- Use `local:reset` when you need a clean slate (wipes the Docker volume and removes the generated kubeconfig file).
- For production, prefer `prod:*` tasks for any mutating action.
- Do not run any production related commands yourself, instead show them in the output
- Do not commit anything instead of me
  

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TopHatCroat)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TopHatCroat)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
