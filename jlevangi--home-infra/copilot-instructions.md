## home-infra

> This repository contains a comprehensive Infrastructure-as-Code (IaC) solution for deploying and managing production-ready K3s Kubernetes clusters and LXC containers on Proxmox VE.

# Home Infrastructure Automation

This repository contains a comprehensive Infrastructure-as-Code (IaC) solution for deploying and managing production-ready K3s Kubernetes clusters and LXC containers on Proxmox VE.

## 🏗️ Architecture

- **Infrastructure**: Proxmox VE
- **Provisioning**: Terraform (VMs), Ansible (LXC containers)
- **Configuration**: Ansible (Cluster setup, App deployment)
- **Kubernetes**: K3s
- **Storage**: Longhorn (Distributed Block Storage), NFS (Backups/Shared)
- **Networking**: MetalLB (Load Balancer), Traefik (Ingress)
- **Secrets**: Ansible Vault

## 🚀 Key Workflows

### 1. Cluster Deployment

Deploy complete K3s clusters to specific environments:

```bash
# Production Cluster
./scripts/deploy-k3s-cluster.sh --prod

# Staging Cluster
./scripts/deploy-k3s-cluster.sh --stage

# Test Cluster
./scripts/deploy-k3s-cluster.sh --test
```

### 2. Component Management

Deploy or update specific infrastructure components or applications:

```bash
# Deploy/Update Infrastructure
./scripts/deploy-component.sh --prod traefik
./scripts/deploy-component.sh --prod metallb
./scripts/deploy-component.sh --prod longhorn

# Deploy/Update Applications
./scripts/deploy-component.sh --prod bookstack
./scripts/deploy-component.sh --prod vaultwarden
./scripts/deploy-component.sh --prod homepage

# Force Redeploy (useful for troubleshooting)
./scripts/deploy-component.sh --prod traefik --force

# List all available components
./scripts/deploy-component.sh --list
```

### 3. LXC Container Management

Manage standalone LXC containers (separate from K3s):

```bash
# Interactive deployment
./scripts/deploy-lxc.sh

# Deploy specific container
./scripts/deploy-lxc.sh nbn-srv

# List available container definitions
./scripts/deploy-lxc.sh --list
```

### 4. Context Management

Manage `kubectl` contexts for multiple clusters:

```bash
# Setup/Refresh all contexts
./scripts/helpers/k3s-context-manager.sh setup

# Switch Context
./scripts/helpers/k3s-context-manager.sh switch prod
./scripts/helpers/k3s-context-manager.sh switch test

# List Contexts
./scripts/helpers/k3s-context-manager.sh list
```

### 5. Disaster Recovery

Backup and restore operations powered by Longhorn and NFS:

```bash
# List available backups
./scripts/helpers/list-backups.sh

# Full Cluster Restore (Rebuild VMs + Data)
./scripts/restore-cluster.sh --prod

# Restore Data Only (to existing cluster)
./scripts/restore-cluster.sh --prod --restore-only

# Cross-Cluster Restore (Clone Prod Data to Stage)
./scripts/restore-cluster.sh --stage --from prod
```

## 📂 Key Configuration Files

| Type | Path | Description |
|------|------|-------------|
| **Ansible Vars** | `ansible/group_vars/k3s_cluster.yml` | Base cluster configuration |
| **Env Overrides** | `ansible/group_vars/k3s_cluster_prod.yml` | Production specific settings |
| **Secrets** | `ansible/group_vars/k3s_cluster_vault.yml` | Encrypted secrets (Vault) |
| **App Configs** | `ansible/group_vars/k3s_cluster.yml` | Application definitions (`app_definitions`) |
| **LXC Defs** | `ansible/lxc_definitions/containers/` | YAML definitions for LXC containers |
| **Terraform** | `terraform/k3_3node_cluster_prod/` | Production VM infrastructure |

## 🛠️ Development Conventions

1.  **Test First**: Always deploy changes to the `--test` environment first.
2.  **Vault Security**: Store all sensitive data in Ansible Vault. The password file is expected at `~/.ansible_vault_pass`.
3.  **Scripts over Manual**: Use the provided scripts in `scripts/` instead of running raw Ansible/Terraform commands when possible, as they handle environment variables and context correctness.
4.  **LXC Definitions**: Define new LXC containers in `ansible/lxc_definitions/containers/` before deploying.

## 📚 Documentation Resources

-   **`README.md`**: General project overview and prerequisites.
-   **`CLAUDE.md`**: Detailed operational guide and command reference (Primary Reference).
-   **`docs/operations/cluster-operations.md`**: Current cluster management, troubleshooting, and operator runbooks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jlevangi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
