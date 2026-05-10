## demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Terraform modules and an Ansible role for configuring server-side infrastructure to integrate with [Vouch](https://github.com/vouch-sh), an OIDC-based workload identity provider. The repo is AWS-only — there is no GCP or Azure support.

## Commands

```bash
# Validate all Terraform (run from repo root)
terraform init -backend=false
terraform validate

# Validate the examples/complete configuration
cd examples/complete && terraform init -backend=false && terraform validate

# Run Ansible playbook (requires inventory)
ansible-playbook -i ansible/inventory/hosts ansible/playbooks/sshd-ca.yml
```

There are no tests, linters, or CI pipelines in this repo. Validation is done via `terraform validate`.

## Architecture

### Root Module (Composition Layer)

`main.tf` at the root is a composition module that wires together all sub-modules. It uses `*_enabled` boolean variables to toggle each module via `count`. A `locals` block auto-derives dependencies (e.g., `vpc_needed = var.ec2_enabled || var.eks_enabled`).

The root passes outputs between modules — for example, VPC outputs flow into EC2/EKS, the CodeCommit ARN flows into the AWS module for scoped IAM policies, and the Vouch IAM role ARN flows into EKS for Access Entries.

### Module Pattern

Every module under `modules/` follows the same 3-file convention:
- `main.tf` — resources and data sources
- `variables.tf` — input variables
- `outputs.tf` — output values

Modules are self-contained and don't reference other modules directly — the root module handles all inter-module wiring.

### Two Categories of Modules

**Identity plumbing** (always-on defaults):
- `modules/aws` — IAM OIDC provider + role with trust policy for Vouch tokens. Has a conditional `demo_services` IAM policy that is auto-enabled when any service module is on.
- `modules/k8s` — Namespace, ServiceAccount (IRSA-annotated), RBAC, ConfigMap.

**Demo service infrastructure** (all default to `false`):
- `modules/aws-vpc` — Shared VPC, auto-created when EC2 or EKS is enabled.
- `modules/aws-codecommit`, `aws-codeartifact`, `aws-ecr` — Standalone service resources.
- `modules/aws-ec2` — t2.nano with SSM, depends on VPC.
- `modules/aws-eks` — EKS Auto Mode cluster, depends on VPC. Creates Access Entries mapping the Vouch role to cluster admin.

### Ansible Role

`ansible/roles/vouch_sshd` configures SSH certificate auth on target hosts by fetching the Vouch CA public key and configuring `sshd` to trust it. Uses `AuthorizedPrincipalsFile` for access control. The role is independent of the Terraform modules.

## Key Design Decisions

- All demo service modules default to disabled to minimize cost. EKS is the expensive one (~$73/mo control plane).
- The `modules/aws` IAM policy for demo services is a single inline policy with all permissions rather than per-service policies, toggled by a single `demo_services_enabled` flag.
- EKS uses Auto Mode (no managed node groups) and API authentication mode (required for Access Entries).
- EC2 has zero inbound security group rules — access is SSM-only.
- VPC has no NAT gateway — nodes go in public subnets to keep costs at $0.
- The `tls` provider is declared explicitly in `versions.tf` because `modules/aws` uses `data.tls_certificate` to get the OIDC thumbprint.

---
> Source: [vouch-sh/demo](https://github.com/vouch-sh/demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
