## home-operations

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A GitOps-managed home Kubernetes cluster running on Talos Linux, with 90+ applications across 18 namespaces. All cluster state is managed through Flux CD with automatic reconciliation from Git.

**Cluster:** 3-node control plane (k8s-0, k8s-1, k8s-2) at 10.88.0.20:6443 (HA VIP)

## Key Commands

All automation uses `task` (Taskfile runner). Run `task --list` to see all available tasks.

```bash
# Secrets — edit/view SOPS-encrypted files
sops path/to/secret.sops.yaml

# Encrypt all unencrypted SOPS files
task sops:encrypt

# Apply a Flux Kustomization manually (without waiting for reconcile)
task flux:apply cluster=main path=default/actual

# Force Flux reconciliation
flux reconcile source git flux-system
flux reconcile kustomization <name> --with-source

# Suspend/resume VolSync backups
task volsync:state-suspend
task volsync:state-resume

# Snapshot an app's PVC
task volsync:snapshot NS=default APP=actual

# Restore an app from backup
task volsync:restore NS=default APP=actual PREVIOUS=1

# Talos node management
task talos:apply-config CLUSTER=main HOSTNAME=k8s-0
task talos:upgrade-node CLUSTER=main HOSTNAME=k8s-0 VERSION=v1.12.4
task talos:upgrade-cluster CLUSTER=main VERSION=v1.12.4
task talos:upgrade-k8s CLUSTER=main VERSION=v1.35.2

# Browse a PVC's contents
task kubernetes:browse-pvc
```

## Repository Structure

```
kubernetes/
├── apps/                  # All application namespaces (not under main/)
│   ├── default/           # Most user-facing apps (33 apps)
│   ├── media/             # Plex, Jellyfin, Radarr, Sonarr, etc.
│   ├── home-automation/   # Home Assistant, ESPHome, Zigbee2MQTT, EMQX
│   ├── observability/     # Prometheus, Grafana, Gatus, Alertmanager
│   ├── cert-manager/      # TLS certificate management
│   ├── network/           # Ingress, DNS, Envoy Gateway
│   ├── database/          # CloudNative-PG, Dragonfly
│   ├── rook-ceph/         # Distributed storage cluster
│   └── ...                # Other system namespaces
├── components/            # Shared Flux components (volsync, externaldns, nfs-scaler)
├── flux/                  # Flux CD config (repositories, vars, apps.yaml)
└── main/                  # Cluster bootstrap only (Talos config, Flux bootstrap)
    ├── bootstrap/         # Talos clusterconfig, Flux bootstrap secrets
    └── talconfig.yaml     # Talos node configuration (via talhelper)
.taskfiles/                # Task automation modules
.sops.yaml                 # SOPS encryption rules
```

## App Structure Pattern

Each app under `kubernetes/apps/<namespace>/<app>/` follows this pattern:

```
<app>/
├── ks.yaml          # Flux Kustomization(s) — defines sync, dependencies, postBuild vars
└── app/
    ├── kustomization.yaml
    ├── helmrelease.yaml    # HelmRelease using bjw-s app-template or upstream chart
    ├── ocirepository.yaml  # OCI chart source (or HelmRepository)
    └── externalsecret.yaml # (optional) pulls secrets from external provider
```

Key patterns in HelmReleases:
- Most apps use the **bjw-s `app-template`** chart (`ghcr.io/bjw-s-labs/helm-charts`)
- Container images are pinned with SHA digest (e.g., `tag: 25.12.0@sha256:...`)
- Routes use `Gateway API` resources pointing to `envoy-internal` or `envoy-external` in the `network` namespace
- PVCs reference `existingClaim: <app-name>` — created by VolSync components
- Secrets pulled via `ExternalSecret` → `external-secrets` operator

## Secret Management

SOPS with Age encryption. The private key lives at `age.key` (outside the repo, set via `SOPS_AGE_KEY_FILE`).

- Files matching `*.sops.yaml` are encrypted; only `data`/`stringData` keys are encrypted in Kubernetes secrets
- Edit encrypted files with: `sops path/to/file.sops.yaml`
- New secrets must be named `*.sops.yaml` to be picked up by `.sops.yaml` rules

## Flux Components

Shared reusable Flux components in `kubernetes/components/`:
- **`volsync`** — adds VolSync `ReplicationSource` + PVC to any app that includes it
- **`externaldns`** — adds ExternalDNS annotations
- **`nfs-scaler`** — NFS scaling support

Apps opt into components via `ks.yaml`:
```yaml
spec:
  components:
    - ../../../../components/volsync
```

## Automated Dependency Updates

Renovate runs hourly and opens PRs for updates to Helm chart versions, container image tags (with digest pinning), and GitHub Actions. Managed by `.github/renovate/` config.

## Environment Setup

The `.envrc` (direnv) sets up:
- `KUBECONFIG` → `kubernetes/main/kubeconfig`
- `TALOSCONFIG` → `kubernetes/main/talosconfig`
- `SOPS_AGE_KEY_FILE` → `age.key`
- `VIRTUAL_ENV` → `.venv` (for Ansible)

Required CLI tools: `task`, `kubectl`, `talosctl`, `flux`, `helm`, `sops`, `age`, `minijinja-cli`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jalim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jalim)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
