## hoff-cluster

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a homelab Kubernetes cluster configuration (`hoff`) using **Talos Linux** for the OS and **Flux CD** for GitOps. It is not an application codebase — all "code" is YAML configuration for Kubernetes, Talos, and Helm charts.

## Cluster Nodes

Three control-plane-only nodes (scheduling allowed on all):

| Hostname | IP            | Install Disk  | Storage Disk |
|----------|---------------|---------------|--------------|
| sequoia  | 192.168.20.3  | /dev/nvme0n1  | /dev/sda → /var/mnt/data |
| cedar    | 192.168.20.4  | /dev/sdc      | /dev/sda → /var/mnt/storage |
| spruce   | 192.168.20.5  | /dev/sdc      | /dev/sda → /var/mnt/storage |

Virtual IP (kube-apiserver): `192.168.20.10:6443`

## Key Tools

- **talhelper** — generates Talos machine configs from `talos/talconfig.yaml`
- **talosctl** — applies configs and manages Talos nodes
- **sops** — encrypts/decrypts secrets (age key, config in `talos/.sops.yaml`)
- **flux** — GitOps controller watching this repo (`main` branch)

## Common Commands

```bash
# Generate Talos machine configs from talconfig.yaml
cd talos && talhelper genconfig

# Apply config to a node (e.g., after changes)
talosctl apply-config --nodes 192.168.20.3 --file talos/clusterconfig/hoff-sequoia.yaml

# Check cluster health
talosctl --nodes 192.168.20.3,192.168.20.4,192.168.20.5 health

# Interact with the cluster
export KUBECONFIG=./kubeconfig
kubectl get nodes

# Trigger Flux reconciliation
flux reconcile kustomization flux-system --with-source

# Encrypt a new secret with sops
sops --encrypt --in-place talos/newsecret.sops.yaml
```

## Repository Structure

```
talos/                    # Talos OS configuration (talhelper)
  talconfig.yaml          # Source of truth for node definitions
  patches/
    global.yaml           # Applied to all nodes
    controlplane.yaml     # Applied to all control-plane nodes
    sequoia.yaml          # Node-specific overrides
    cedar.yaml
    spruce.yaml
  talsecret.sops.yaml     # SOPS-encrypted cluster secrets
  clusterconfig/          # Generated output (git-ignored)

kubernetes/
  flux-system/            # Flux bootstrap manifests (managed by flux)
    flux-system/          # gotk-components, gotk-sync
    infrastructure.yaml   # Flux Kustomization pointing to ./kubernetes/infrastructure
  infrastructure/         # Cluster infrastructure (deployed by Flux)
    cilium/               # CNI — replaces kube-proxy, L2 LB
    longhorn/             # Distributed block storage (3 replicas)
```

## Architecture Notes

- **CNI**: Cilium with `kubeProxyReplacement: true` (kube-proxy is disabled in Talos)
- **Storage**: Longhorn with `defaultReplicaCount: 3`, kubelet root at `/var/lib/kubelet`
- **Secrets**: SOPS with age encryption; key fingerprint in `talos/.sops.yaml`
- **Flux sync**: Watches `./kubernetes/flux-system` (interval 10m), which in turn deploys `./kubernetes/infrastructure` (interval 1m)
- **Generated files**: `talos/clusterconfig/` is git-ignored; regenerate with `talhelper genconfig`
- The `kubeconfig` file is git-ignored (listed in root `.gitignore`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rjhoffmann)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rjhoffmann)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
