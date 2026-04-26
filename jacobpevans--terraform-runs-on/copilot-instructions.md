## terraform-runs-on

> Self-hosted GitHub Actions runners on AWS EC2 spot instances via RunsOn.

# Terraform RunsOn - AI Agent Instructions

Self-hosted GitHub Actions runners on AWS EC2 spot instances via RunsOn.

## Technology Stack

- **Terraform/Terragrunt** - Infrastructure provisioning
- **GitHub Actions** - CI/CD (OIDC for AWS auth)
- **RunsOn** - Self-hosted runner orchestration
- **nix-devenv** - Dev shell via `shells/terraform` (includes aws-vault, awscli2, sops, tfsec, trivy)
- **aws-vault** - AWS credentials for S3 backend (profile: `tf-runs-on`)
- **Doppler** - Runtime secrets (RunsOn license key)

## Running Terraform Commands

**CRITICAL**: All Terragrunt commands require the complete toolchain wrapper.

### The Command (always this, always both)

```bash
aws-vault exec tf-runs-on -- doppler run -- terragrunt <COMMAND>
```

### Command Breakdown

1. **`aws-vault exec tf-runs-on`** - AWS credentials for S3 backend (profile: `tf-runs-on`)
2. **`doppler run --`** - Injects Doppler secrets as env vars (e.g., `RUNSON_LICENSE_KEY`)
3. **`terragrunt <COMMAND>`** - Runs Terraform

### Common Commands

```bash
aws-vault exec tf-runs-on -- doppler run -- terragrunt validate
aws-vault exec tf-runs-on -- doppler run -- terragrunt plan
aws-vault exec tf-runs-on -- doppler run -- terragrunt apply
```

### Claude Code Sessions

Start Claude inside aws-vault to get 1hr credential access without per-command popups:

```bash
cd ~/git/terraform-runs-on/main
aws-vault exec tf-runs-on -- claude
```

Then terragrunt commands only need Doppler (AWS credentials are inherited):

```bash
doppler run -- terragrunt plan
doppler run -- terragrunt apply
```

### Doppler Configuration

Doppler is configured at the `~/git/` scope (`iac-conf-mgmt/prd`), inherited by all repos.
No per-repo `doppler setup` needed. The `RUNSON_LICENSE_KEY` env var is mapped to
`license_key` via the `inputs` block in `terragrunt.hcl`.

## Dev Environment

Uses [nix-devenv](https://github.com/JacobPEvans/nix-devenv) terraform shell via direnv.
No local `flake.nix` — the remote shell provides all tooling with per-shell lock isolation.

```bash
direnv allow    # one-time per worktree, then automatic
```

## Architecture

- **VPC**: Dedicated VPC with 3 public subnets (us-east-2), no NAT Gateway
- **RunsOn**: Terraform module `runs-on/runs-on/aws` deploys App Runner + EC2 spot
- **OIDC**: GitHub Actions authenticates via OIDC (no stored AWS credentials in CI)
- **State**: S3 + DynamoDB via Terragrunt
- **Budget**: AWS Budget alarm at $10/month
- **Observability**: OTEL to Cribl.Cloud Free tier

## Secrets

| Secret | Source | Used By |
| ------ | ------ | ------- |
| `RUNSON_LICENSE_KEY` | Doppler (`iac-conf-mgmt/prd`) | Local: `doppler run`; CI: `ci-gate.yml` plan + `deploy.yml` apply |
| `GH_ACTION_DOPPLER_IAC_CONF_MGMT` | GitHub repo secret (secrets-sync Tier 2) | `ci-gate.yml` + `deploy.yml` Doppler fetch step |
| `AWS_OIDC_ROLE_ARN` | GitHub repo secret | `ci-gate.yml` plan + `deploy.yml` apply OIDC auth |
| AWS credentials | aws-vault profile `tf-runs-on` | Local terragrunt S3 backend auth |

Consumer repos using the deployed runner only need the RunsOn GitHub App
installed — they never need a license secret.

## Cost Target

~$5-8/month: App Runner (~$3) + EC2 spot (~$1-4) + CloudWatch ($0.50).
Budget alarm alerts at 50%, 80%, 100% of $10/month.

## Worktree Structure

```text
~/git/terraform-runs-on/
  .git/     # Bare repo
  main/     # Main branch worktree
  feat/     # Feature worktrees
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JacobPEvans) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
