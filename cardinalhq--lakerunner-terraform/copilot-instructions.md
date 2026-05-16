## lakerunner-terraform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## common rules

- no emoji in source code, comments, documentation, or scripts.  emoji are OK in tests when appropriate like testing for utf-8 support.
- no advertising for Claude or Claude's creator.

## Repository Purpose

This repository provides Terraform infrastructure-as-code for deploying Lakerunner in GCP, AWS, and Azure environments. It's designed primarily for customer POC (proof-of-concept) deployments, focusing on easy setup and excellent first impressions for potential customers evaluating the Lakerunner platform.

## Key Commands

- `make` or `make help` - Show available commands
- `make test` - Run full test suite (format check, validate all envs, plan gcp-poc)
- `make fmt` - Auto-format all Terraform files (recursive)
- `make validate` - Validate all environments (no credentials needed)
- `make plan` (alias `plan-gcp`) - Run terraform plan against gcp-poc with test project ID `lakerunner-terraform`
- `make clean` - Clean up temporary Terraform files

## Architecture Overview

### Directory Structure
```
terraform/
├── environments/
│   ├── gcp-poc/          # GCP POC environment
│   ├── aws-poc/          # AWS POC environment
│   └── azure-poc/        # Azure POC environment
└── providers/{gcp,aws,azure}/ # Provider configuration snippets
```

### Multi-Cloud Design
Each `*-poc` environment is a self-contained Terraform root module. They share a common pattern: dedicated VPC, object storage with notification queue, optional managed Postgres (default on), managed Kubernetes cluster with workload identity (default on). Workload identity is the only auth path for the Lakerunner workload across all three clouds; no static credentials are emitted.

### Customer Configuration Strategy
- `terraform.tfvars.example` - Template with documented options
- `terraform.tfvars` - Customer's actual config (git-ignored for safety)
- Customers only need to change `project_id` for basic setup
- Safe upgrade path: customer configs never conflict with repo updates

### Network Configuration
Each POC environment always provisions a dedicated VPC. Specifics vary per cloud (subnet layout, secondary ranges for pods/services on GCP, AZ count for AWS) but all three follow the same pattern: dedicated VPC, NAT for egress, no peering to anything else.

### Core Infrastructure Components

**Storage and notifications (all clouds):**
- One bucket per POC, with all object-create events routed to a queue (Pub/Sub on GCP, SQS on AWS, Storage Queue on Azure)
- The Lakerunner consumer filters out the `db/` prefix; the cloud-side notification is unfiltered

**Kubernetes (default on for all clouds):**
- Managed cluster (GKE / EKS / AKS) with autoscaling node pool, Spot capacity by default
- Workload identity (Workload Identity / IRSA / Workload Identity) wires a Kubernetes ServiceAccount to a cloud IAM identity scoped to the bucket and queue
- This is the *only* auth path for the Lakerunner workload — no long-lived access keys are emitted

**Security:**
- Dedicated service accounts / IAM roles with least-privilege access
- Auto-cleanup after 30 days on the bucket
- No hardcoded credentials anywhere

## Testing Strategy

Tests run without requiring real cloud credentials:
- `terraform fmt -check -recursive` across the entire `terraform/` tree
- `terraform validate` for each of the three environments (with `init -backend=false`)
- Targeted `terraform plan` for gcp-poc only, using synthetic project ID `lakerunner-terraform`
- AWS and Azure plans are not run by `make test` since they require real credentials

## Customer Experience Focus

This infrastructure prioritizes:
1. **5-minute setup** - Minimal configuration required
2. **Professional outputs** - Clear resource information and next steps
3. **Safety** - Auto-cleanup, upgrade-safe configuration management
4. **Flexibility** - Supports both greenfield and enterprise network constraints

The POC environment is designed to make an excellent first impression for potential customers evaluating Lakerunner.

- never use emoji in any docs, scripts, or other files except for tests needing to test for proper emoji handling

---
> Source: [cardinalhq/lakerunner-terraform](https://github.com/cardinalhq/lakerunner-terraform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
